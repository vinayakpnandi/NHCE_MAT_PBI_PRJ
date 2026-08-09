# 🌊 GreenWave Data Pipeline

### Snowflake ETL/ELT Pipeline & Power BI Analytics

A complete data engineering and business intelligence project that ingests customer and transactional data from **Amazon S3 and Excel**, processes and transforms it using **Snowflake**, calculates key business metrics such as **Revenue and Profit**, and prepares an analytical dataset for **Power BI reporting and visualization**.

---

## 📌 Project Overview

The **GreenWave Data Pipeline** is designed to demonstrate an end-to-end data analytics workflow, starting from raw data ingestion and ending with business-ready insights.

The project integrates multiple data sources, performs data transformation and cleaning, joins customer, order, and product information, calculates financial metrics, and aggregates the results based on customers, products, and time periods.

### End-to-End Flow

```text
Amazon S3 + Excel
       ↓
   Data Ingestion
       ↓
     Snowflake
       ↓
Data Transformation
       ↓
Table Joins & Cleaning
       ↓
Revenue & Profit Calculation
       ↓
Aggregation
       ↓
Business-Ready Dataset
       ↓
     Power BI
       ↓
Business Insights
```

---

## 🎯 Objectives

The main objectives of this project are to:

* Build an end-to-end data pipeline
* Ingest data from multiple sources
* Load structured and semi-structured data into Snowflake
* Process JSON data using Snowflake
* Dynamically process multiple Excel worksheets
* Clean and standardize data types
* Join customer, order, and product datasets
* Calculate Revenue and Profit
* Create time-based analytical attributes
* Aggregate transactional data
* Create a business-ready analytical table
* Use the processed data for Power BI reporting

---

## 🏗️ Architecture

```text
                    ┌───────────────────┐
                    │    Amazon S3      │
                    │                   │
                    │ customer_accounts │
                    │      .json        │
                    └─────────┬─────────┘
                              │
                              │
                    ┌─────────▼─────────┐
                    │                   │
                    │     Snowflake     │
                    │                   │
                    │  DEV_DB           │
                    │  DEV_SCHEMA       │
                    └─────────┬─────────┘
                              │
                              │
              ┌───────────────┴───────────────┐
              │                               │
       ┌──────▼──────┐                 ┌──────▼──────┐
       │ JSON Data   │                 │ Excel Data  │
       │ Processing  │                 │ Processing  │
       └──────┬──────┘                 └──────┬──────┘
              │                               │
              │                         ┌─────▼─────┐
              │                         │  ITEMS    │
              │                         │  ORDERS   │
              │                         │ORDER_ITEMS │
              │                         └─────┬─────┘
              │                               │
              └──────────────┬────────────────┘
                             ▼
                  ┌─────────────────────┐
                  │ Data Transformation │
                  │                     │
                  │ • Flatten JSON      │
                  │ • Type Casting      │
                  │ • Joins             │
                  │ • Calculations      │
                  └──────────┬──────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │ Business Metrics    │
                  │                     │
                  │ Revenue             │
                  │ Profit              │
                  └──────────┬──────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │ Aggregation         │
                  │                     │
                  │ Account             │
                  │ Product             │
                  │ Year / Quarter      │
                  │ Month               │
                  └──────────┬──────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │ GW_PROFIT_BY_ACCOUNT│
                  └──────────┬──────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │     Power BI        │
                  │                     │
                  │ Reports & Insights  │
                  └─────────────────────┘
```

---

# 📂 Data Sources

The project uses two primary data sources.

## 1. Amazon S3

Customer account information is stored as a JSON file:

```text
customer_accounts.json
```

The JSON data contains customer/account information that is loaded into Snowflake and processed using semi-structured data capabilities.

---

## 2. Excel Workbook

Transactional data is stored in:

```text
store_lite.xlsx
```

The workbook contains multiple worksheets:

```text
ITEMS
ORDERS
ORDER_ITEMS
```

These worksheets contain information about:

* Products
* Orders
* Customers
* Order quantities
* Product prices
* Product costs

---

# ❄️ Snowflake Data Warehouse

Snowflake is used as the central data warehouse for the project.

The pipeline works within:

```text
NHCE_WAREHOUSE
      │
      └── DEV_DB
            │
            └── DEV_SCHEMA
```

The pipeline creates and populates several tables.

### Customer Table

```text
GW_CUSTOMER_ACCOUNTS
```

Stores customer/account information extracted from the JSON source.

### Product Table

```text
GW_ITEMS
```

Stores product information such as:

* Item ID
* Item name
* Cost
* Price

### Orders Table

```text
GW_ORDERS
```

Stores order-level information such as:

* Order ID
* Customer ID
* Order date

### Order Items Table

```text
GW_ORDER_ITEMS
```

Stores the relationship between orders and products, including:

* Order ID
* Item ID
* Quantity

---

# 🔄 Data Pipeline

## Step 1 — Load Customer JSON

Customer information is ingested from Amazon S3.

The JSON data is initially handled as semi-structured data using Snowflake's `VARIANT` data type.

```text
S3
 ↓
customer_accounts.json
 ↓
Snowflake
 ↓
GW_CUSTOMER_ACCOUNTS
```

---

## Step 2 — Flatten JSON

The nested JSON structure is transformed into relational columns.

Important customer fields are extracted, such as:

```text
CUSTOMER_RID
ACCOUNT_NAME
```

This allows customer data to be joined with transactional data.

---

## Step 3 — Identify Excel Worksheets

Instead of creating separate pipelines for every Excel worksheet, the project dynamically generates the worksheet names:

```text
ITEMS
ORDERS
ORDER_ITEMS
```

These names are stored in:

```text
GW_SHEET_NAMES
```

This enables dynamic processing of multiple worksheets.

---

## Step 4 — Dynamic Table Processing

A **Table Iterator** processes each worksheet dynamically.

Conceptually, the pipeline executes:

```sql
SELECT *
FROM ${worksheet_name}
```

for each worksheet.

This allows the pipeline to process:

```text
ITEMS
ORDERS
ORDER_ITEMS
```

without creating completely separate workflows for each one.

---

# 🧹 Data Transformation

The pipeline performs several transformations before creating the final analytical dataset.

### Data Type Standardization

Examples include:

```text
CUSTOMER_RID → NUMBER
RID          → NUMBER
ORDER_DATE   → DATE
QUANTITY     → NUMBER
PRICE        → NUMBER(38,2)
COST         → NUMBER(38,2)
```

Proper data types ensure that joins, calculations, and aggregations work correctly.

---

# 🔗 Data Integration & Joins

The project combines information from multiple datasets.

The main relationships are:

```text
CUSTOMER
   │
   │ CUSTOMER_RID
   ▼
ORDERS
   │
   │ ORDER_RID
   ▼
ORDER_ITEMS
   │
   │ ITEM_RID
   ▼
ITEMS
```

The final dataset combines:

```text
Customer
+
Order
+
Product
+
Quantity
+
Price
+
Cost
+
Order Date
```

This creates a complete transactional view for business analysis.

---

# 💰 Revenue Calculation

Revenue is calculated at the order-item level using:

```text
Revenue = Price × Quantity
```

In the pipeline:

```text
PRICE * QUANTITY
```

Example:

```text
Price    = ₹500
Quantity = 3

Revenue = ₹500 × 3
        = ₹1,500
```

The calculated value is stored as:

```text
REVENUE
```

---

# 📈 Profit Calculation

Profit is calculated using:

```text
Profit = (Price - Cost) × Quantity
```

Example:

```text
Price    = ₹500
Cost     = ₹300
Quantity = 3

Profit = (₹500 - ₹300) × 3
       = ₹600
```

The calculated value is stored as:

```text
PROFIT
```

---

# 📅 Time-Based Analysis

The pipeline extracts useful time attributes from the order date.

The following columns are generated:

```text
ORDER_YEAR
ORDER_QTR
ORDER_MONTH
```

For example:

```text
ORDER_DATE = 2026-08-10

ORDER_YEAR  = 2026
ORDER_QTR   = 3
ORDER_MONTH = 8
```

These fields make it easier to perform:

* Monthly analysis
* Quarterly analysis
* Yearly analysis
* Trend analysis

in Power BI.

---

# 📊 Data Aggregation

After calculating Revenue and Profit, the pipeline aggregates the data.

The data is grouped by:

```text
ACCOUNT_NAME
ITEM_NAME
ORDER_YEAR
ORDER_QTR
ORDER_MONTH
```

The following metrics are then calculated:

```text
SUM(PROFIT)
SUM(REVENUE)
```

The final columns are standardized as:

```text
TOT_PROFIT
TOT_REVENUE
```

---

# 🏁 Final Analytical Table

The final business-ready dataset is stored as:

```text
GW_PROFIT_BY_ACCOUNT
```

This table contains aggregated information that can directly support business intelligence and reporting.

Example structure:

| Account   | Item      | Year | Quarter | Month | Total Revenue | Total Profit |
| --------- | --------- | ---: | ------: | ----: | ------------: | -----------: |
| Account A | Product X | 2026 |       3 |     8 |       ₹50,000 |      ₹12,000 |
| Account A | Product Y | 2026 |       3 |     8 |       ₹30,000 |       ₹8,000 |
| Account B | Product X | 2026 |       3 |     8 |       ₹75,000 |      ₹18,000 |

---

# ⚙️ Pipeline Orchestration

The project includes an orchestration workflow that controls the execution order of different pipeline components.

The orchestration ensures that:

1. Customer data is prepared
2. Worksheet names are generated
3. Excel worksheets are processed
4. Required tables are populated
5. Transformations are completed
6. Profit and Revenue calculations are performed
7. The final analytical table is generated

Simplified workflow:

```text
START
  │
  ├───────────────┐
  │               │
  ▼               ▼
Customer       Create
Accounts       Sheet Names
  │               │
  │               ▼
  │         Table Iterator
  │               │
  │               ▼
  │          Excel Tables
  │               │
  └───────┬───────┘
          │
          ▼
     Synchronization
          │
          ▼
Calculate Profit
& Revenue
          │
          ▼
Final Analytical Table
          │
          ▼
        Power BI
```

---

# 📁 Repository Structure

```text
greenwave-data-pipeline/
│
├── GreenWave Pipelines/
│   │
│   ├── GreenWave Technologies Demo.orch.yaml
│   │
│   ├── Calculate Profit and Revenue.tran.yaml
│   │
│   └── Create SHEET_NAMES.tran.yaml
│
├── README.md
│
└── ...
```

### Pipeline Files

| File                                     | Purpose                                                       |
| ---------------------------------------- | ------------------------------------------------------------- |
| `GreenWave Technologies Demo.orch.yaml`  | Main orchestration workflow                                   |
| `Create SHEET_NAMES.tran.yaml`           | Dynamically generates Excel worksheet names                   |
| `Calculate Profit and Revenue.tran.yaml` | Performs joins, transformations, calculations and aggregation |

---

# 🛠️ Technologies Used

| Technology    | Purpose                                 |
| ------------- | --------------------------------------- |
| **Snowflake** | Cloud data warehouse                    |
| **Amazon S3** | Cloud object storage / data source      |
| **JSON**      | Customer data source                    |
| **Excel**     | Transactional data source               |
| **SQL**       | Data querying and transformation        |
| **ETL/ELT**   | Data integration and processing         |
| **Power BI**  | Business intelligence and visualization |
| **YAML**      | Pipeline/orchestration definitions      |

---

# 📌 Key Features

### 🔹 Multi-Source Data Integration

Combines:

```text
Amazon S3
+
JSON
+
Excel
```

into a centralized Snowflake environment.

### 🔹 Semi-Structured Data Processing

Processes nested JSON data using Snowflake's semi-structured data capabilities.

### 🔹 Dynamic Excel Processing

Uses worksheet names and iteration to dynamically process multiple Excel sheets.

### 🔹 Data Transformation

Includes:

* Data type conversion
* JSON flattening
* Joins
* Column transformations
* Date extraction
* Aggregations

### 🔹 Business Metric Calculation

Calculates:

```text
Revenue
Profit
```

from transactional data.

### 🔹 BI-Ready Dataset

Creates:

```text
GW_PROFIT_BY_ACCOUNT
```

for downstream Power BI analysis.

---

# 📊 Business Questions This Dataset Can Answer

The final dataset can be used to answer questions such as:

* What is the total revenue?
* What is the total profit?
* Which customers generate the most revenue?
* Which customers generate the highest profit?
* Which products generate the most revenue?
* Which products are the most profitable?
* How does revenue change month-to-month?
* How does profit change over time?
* Which quarter performs best?
* Which accounts contribute most to overall profitability?
* What are the revenue and profit trends by product?

---

# 💡 Key Data Engineering Concepts Demonstrated

This project demonstrates practical understanding of:

* Data ingestion
* ETL/ELT
* Cloud data warehousing
* Snowflake architecture
* Semi-structured data
* JSON flattening
* Data type conversion
* Dynamic processing
* Iterative workflows
* Table joins
* Data aggregation
* Business metric engineering
* Pipeline orchestration
* BI data preparation

---

# 🚀 End-to-End Pipeline Summary

```text
                 RAW DATA
                    │
       ┌────────────┴────────────┐
       │                         │
    Amazon S3                 Excel
       │                         │
   Customer JSON        ITEMS / ORDERS /
                        ORDER_ITEMS
       │                         │
       └────────────┬────────────┘
                    ▼
                SNOWFLAKE
                    │
                    ▼
              DATA CLEANING
                    │
                    ▼
              DATA TRANSFORM
                    │
          ┌─────────┴─────────┐
          │                   │
       Flatten              Cast
        JSON              Data Types
          │                   │
          └─────────┬─────────┘
                    ▼
                  JOINS
                    │
                    ▼
          REVENUE & PROFIT
             CALCULATION
                    │
                    ▼
               AGGREGATION
                    │
                    ▼
         GW_PROFIT_BY_ACCOUNT
                    │
                    ▼
                POWER BI
                    │
                    ▼
             BUSINESS INSIGHTS
```

---

# 🎓 What I Learned

Through this project, I gained hands-on experience with:

* Building data pipelines
* Working with Snowflake
* Loading data from different sources
* Handling semi-structured JSON data
* Performing data transformations
* Creating dynamic data workflows
* Joining multiple datasets
* Creating business metrics
* Preparing datasets for Power BI
* Thinking about data from both an engineering and business perspective

---

# 🔮 Future Improvements

Potential improvements to the project include:

* Add automated data quality checks
* Implement incremental data loading
* Add error handling and pipeline monitoring
* Add data validation and duplicate detection
* Introduce dimensional modeling
* Create fact and dimension tables
* Add automated pipeline scheduling
* Implement Snowflake Streams and Tasks
* Add more advanced Power BI dashboards
* Implement role-based access control
* Add automated documentation and data lineage

---

# 👨‍💻 Author

**Vinayak Prakash Nandi**

Data Science Engineering Student
New Horizon College of Engineering

### Areas of Interest

* Data Analytics
* Data Engineering
* Business Intelligence
* SQL
* Python
* Snowflake
* Power BI
* Artificial Intelligence

---

## ⭐ Project Highlights

```text
☁️ Amazon S3
        +
❄️ Snowflake
        +
🔄 ETL/ELT
        +
📊 Power BI
        =
🚀 End-to-End Data Analytics Pipeline
```

---

## 📜 License

This project is intended for educational and portfolio purposes.
