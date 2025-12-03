

# 📘 Projeto de Extensão 3: Conversão de Dataset Excel para Banco de Dados PostgreSQL com Análises de Vendas (Granito & Solid Surface – 2024)

Este projeto documenta todo o processo de **conversão de um dataset Excel** contendo dados de vendas de **granito e solid surface** para um **banco de dados relacional PostgreSQL** totalmente normalizado, acompanhado de **relatórios analíticos**, consultas SQL e preparação para ferramentas de BI.

O objetivo é fornecer uma estrutura profissional para análise das vendas do ano de **2024**, atendendo às necessidades da empresa do segmento de superfícies (Granito, Corian, Quartzo etc.).

---

## 📂 Conteúdo do Projeto

* Importação do arquivo `excel_base.csv`
* Criação do schema e banco de dados no PostgreSQL
* Modelagem normalizada em **3ª Forma Normal (3FN)**
* Criação de tabelas dimensionais e tabela fato
* Correções aplicadas durante a carga (ex.: chaves duplicadas)
* Scripts SQL completos (DDL + ETL)
* Relatórios de vendas, clientes, regiões, lucro e materiais
* Base pronta para dashboards em Power BI / Looker Studio / Metabase

---

# 🏗️ 1. Introdução

A empresa forneceu um dataset no formato Excel contendo todas as vendas de **2024**. O arquivo passou por revisão, padronização e posterior migração para o PostgreSQL.

A estrutura final permite análises como:

* principais materiais vendidos
* margem de lucro por material
* vendas por cliente
* desempenho por estado/cidade
* comparativo trimestral
* ticket médio
* tendência de vendas ao longo do ano

Toda a modelagem foi pensada para **alto desempenho analítico** e fácil extensão futura.

---

# 📥 2. Importação do Dataset Excel

O arquivo `excel_base.csv` foi importado para o PostgreSQL via PGAdmin utilizando uma tabela de **staging**, responsável por armazenar o conteúdo bruto do dataset antes da normalização.

### ▶️ Processo de Importação no PGAdmin

1. Criar o schema e a tabela staging
2. Abrir:
   `sales_data → Tables → staging_sales_raw`
3. Botão direito → **Import/Export**
4. Configurações:

   * Import
   * CSV
   * Header: Yes
   * Delimiter: ,
   * Encoding: UTF-8

Ao finalizar, todos os registros foram carregados corretamente na staging.

---

# 🗃️ 3. Estrutura do Banco de Dados

O banco segue uma modelagem relacional **normalizada em 3FN**, com separação em dimensões e tabela fato. Isso evita redundância, reduz inconsistências e melhora a performance das análises.

### **Diagramas disponíveis**

* ER clássico (entidade–relacionamento)
* Star Schema (modelo dimensional)

---

# 🧱 4. Criação do Schema e Tabelas (DDL)

### Schema

```sql
CREATE SCHEMA IF NOT EXISTS sales_data;
SET search_path TO sales_data;
```

### Tabela de Staging

```sql
CREATE TABLE sales_data.staging_sales_raw (
    MATERIAL_ID      VARCHAR(50),
    MATERIAL_NAME    VARCHAR(100),
    MATERIAL_TYPE    VARCHAR(50),
    MATERIAL_COLOR   VARCHAR(100),
    PRICE_SQFT       NUMERIC(10,2),
    SELL_DATE        DATE,
    SQFT_SOLD        NUMERIC(10,4),
    SALES_TOTAL      NUMERIC(12,2),
    EXPENSES         NUMERIC(12,2),
    PROFIT           NUMERIC(12,2),
    CUSTOMER         VARCHAR(100),
    CITY_NAME        VARCHAR(100),
    COUNTY_NAME      VARCHAR(100),
    STATE            CHAR(2),
    TYPE             VARCHAR(50),
    TRIMESTRE        VARCHAR(10)
);
```

### Tabelas Normalizadas

**materials, customers, locations, sales (fato)**
*(todas incluídas no projeto, usando chaves primárias e estrangeiras)*

---

# 🔄 5. Transformação e Carga (ETL)

Após carregar os dados brutos na staging, foram populadas as tabelas normalizadas.

### Materiais

```sql
INSERT INTO sales_data.materials (...)
SELECT DISTINCT ... FROM sales_data.staging_sales_raw;
```

### Clientes

```sql
INSERT INTO sales_data.customers (...)
SELECT DISTINCT CUSTOMER, TYPE ...
```

### Localizações

```sql
INSERT INTO sales_data.locations (...)
SELECT DISTINCT CITY_NAME, COUNTY_NAME, STATE ...
```

### Tabela Fato – Vendas

```sql
INSERT INTO sales_data.sales (...)
SELECT ... FROM sales_data.staging_sales_raw
JOIN customers ...
JOIN locations ...
```

---

# 🛠️ 6. Problemas Encontrados e Correções

### ✔️ Duplicidade em MATERIAL_ID

Durante a tentativa de inserir materiais, ocorreu:

```
ERROR: duplicate key value violates unique constraint "materials_pkey"
```

**Solução aplicada:**

* Uso de `SELECT DISTINCT` antes da carga
* Limpeza final sem necessidade de ajustes na chave primária

---

# 📊 7. Relatórios e Consultas SQL Criadas

Esta seção contém as principais análises desenvolvidas no projeto.

### 📌 1. Ranking de Materiais Mais Vendidos

```sql
SELECT m.name, SUM(s.sales_total) AS total_sales
FROM sales_data.sales s
JOIN sales_data.materials m ON m.material_id = s.material_id
GROUP BY m.name
ORDER BY total_sales DESC;
```

### 📌 2. Vendas por Cliente

```sql
SELECT c.customer_name, SUM(s.sales_total)
FROM sales_data.sales s
JOIN sales_data.customers c ON c.customer_id = s.customer_id
GROUP BY c.customer_name;
```

### 📌 3. Análise por Localização

```sql
SELECT state, county_name, city_name, SUM(s.sales_total)
FROM sales_data.sales
JOIN sales_data.locations USING(location_id)
GROUP BY 1,2,3;
```

### 📌 4. Faturamento por Trimestre

```sql
SELECT trimestre, SUM(s.sales_total)
FROM sales_data.sales
GROUP BY trimestre;
```

### 📌 5. Margem de Lucro por Material

```sql
SELECT 
    m.name, 
    SUM(s.sales_total), 
    SUM(s.profit),
    ROUND((SUM(s.profit)/SUM(s.sales_total))*100,2) AS margin
FROM sales_data.sales s
JOIN sales_data.materials m ON m.material_id = s.material_id
GROUP BY m.name;
```

---

# 📈 8. Possíveis Extensões do Projeto

* Criação de dashboards em Power BI ou Looker Studio
* Previsão de vendas com modelos estatísticos
* Detecção de anomalias em vendas e margens
* Automação do ETL com Airflow ou dbt
* Publicação via APIs (Flask, FastAPI)

---

# ✔️ 9. Conclusão

O projeto entregou um processo completo de:

* **ingestão de dados Excel**,
* **modelagem normalizada em PostgreSQL**,
* **padronização e limpeza**
* **carga em um banco robusto**,
* **consultas analíticas avançadas** focadas nas vendas de 2024 da empresa.

A estrutura final serve como base sólida para BI, análises financeiras, auditoria e acompanhamento operacional.

---

