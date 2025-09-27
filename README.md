# Deloitte Case Study – Execution Steps & Rationale

This document explains the steps I followed to complete the case study, the tools I used, and the reasons behind my design choices. It also includes my key observations from the dataset.

---

## Tools and Technologies

- **Python**  
  I used Python to build the ETL pipeline because it is flexible, has powerful libraries for data cleaning (`pandas`, `numpy`), and integrates well with PostgreSQL.  
  Python also allows me to handle data quality rules in a very transparent way (e.g., flagging negative sales, ambiguous dates, etc.).

- **PostgreSQL**  
  I chose PostgreSQL as the database because it is reliable, open-source, and has strong support for relational models and constraints.  
  Using PostgreSQL allowed me to create schemas (`raw`, `stg`, `dwh`) and enforce data quality checks directly at the database level.

- **Windows Task Scheduler**  
  For now, I used Task Scheduler to automate pipeline execution on my local machine.  
  This is a simple solution to trigger Python scripts daily until a more advanced orchestrator like Apache Airflow can be introduced.

- **Power BI**  
  Power BI is the BI tool I planned for the dashboard because it integrates easily with PostgreSQL and can handle fact/dimension models well.  
  It is also suitable for building KPI reports required in the case study.

---

## Data Warehouse Design

- **Star Schema with 3 Layers**  
  I used a star schema approach with separate schemas:  
  - **RAW**: Stores the raw source files exactly as received. This ensures traceability.  
  - **STG (Staging)**: Stores cleaned and validated data. This is where all data quality rules are applied.  
  - **DWH (Data Warehouse)**: Stores the dimensional model (fact + dimension tables). This layer is used for analysis and reporting.

- **Why star schema?**  
  It reduces redundancy, makes queries faster for analytics, and separates master data (dimensions) from transactional data (fact).  
  It also matches Power BI’s strengths for building dashboards.

---

## ETL Process (Extract – Transform – Load)

1. **Extract**  
   - Files were read directly from the given CSVs.  
   - All files were first loaded into the `raw.orders_landing` table without modifications.

2. **Transform (Staging)**  
   - Applied data quality checks:  
     - Date parsing with MM/DD vs DD/MM ambiguity handling.  
     - Negative values in latest month = excluded (returns/credit memos).  
     - Missing required fields = excluded.  
     - Profit missing = allowed, flagged.  
     - "Same Day" ship mode but different dates = flagged as soft warning.  
   - Results stored in:  
     - `stg.orders_clean` (valid data).  
     - `stg.orders_exceptions` (all issues, soft + hard flags).

3. **Load (Warehouse)**  
   - Built fact and dimension tables:  
     - `fact_orders` for transaction-level measures.  
     - Dimensions for Customer, Product, Geo, Date, Segment, Ship Mode.  
   - Ensured surrogate keys and relationships (ERD based on star schema).  
   - Only latest clean data loaded into `dwh.fact_orders`.

---

## Observations on Dataset

- **Dates**: Some files contained ambiguous date formats (could be read as DD/MM or MM/DD). I solved this by validating against the file’s month window.  
- **Negative values**: Found negative `Quantity` and `Sales`. For older months, I kept and flagged them (likely returns). For the latest month, I excluded them.  
- **Profit field**: In some rows `Profit` was missing. I allowed NULL but flagged it.  
- **Same Day Ship Mode**: Found rows where `Ship Mode = Same Day` but `Order Date` ≠ `Ship Date`. These were flagged as soft inconsistencies.  
- **Duplicates**: Found duplicate `(Order ID + Product ID)` rows in the same snapshot. Kept only the first and flagged duplicates.  

---

## Suggestions for Improvement

- **Automation**: Replace Windows Task Scheduler with Apache Airflow for better scheduling, retries, and monitoring.  
- **Data Quality**: Share the Data Quality report (`Task_5_Inconsistencies_Analysis.xlsx`) with SMEs for feedback and corrections from the business side.  
- **Scalability**: For larger datasets, move from local PostgreSQL to cloud-based warehouses (Snowflake, BigQuery, etc.).  
- **Versioning**: Keep a record of each raw file version in storage (e.g., S3 bucket) to support historical reprocessing.

---

## Deliverables Generated

- **Task 1**: KPI Glossary (`Task_1_KPI_Glossary.csv`)  
- **Task 2**: High-Level Design PPT (`Task_2_HLD.pptx`)  
- **Task 3**: ERD (`Task_3_ERD.jpg`)  
- **Task 4**: DDL scripts (`Task_4_DDL.txt`)  
- **Task 5**: Data Quality Report (`Task_5_Inconsistencies_Analysis.xlsx`)  
- **Task 6**: Exported marts (`Task_6_1_Data_Marts.zip`, `Task_6_2_Data_Marts_Rows.csv`)  
- **Task 7**: This README.md

---


**Prereqs**
- Python 3.x
- PostgreSQL running locally
- Create database: `retail_dwh`
- Update DSN in code if needed:
  `postgresql://postgres:<password>@localhost:5432/retail_dwh`

**Install packages**
```bash
pip install -r requirements.txt

**requirements.txt**
```txt
pandas
numpy
psycopg2-binary
openpyxl
python-dateutil


Order to execute

DDL (Task 4): create schemas/tables

Run the SQL in ddl/Task_4_DDL.sql (or the DDL cell in the DWH.ipynb)

RAW load (Block 1): land files as-is into raw.orders_landing

Notebook: Raw Ingestion.ipynb

STG clean (Block 2): window-aware date parsing, soft/hard rules

Notebook: stg.ipynb

Soft flags include: ambiguous dates (MM/DD vs DD/MM) and “Same Day” ship mode when dates differ

DWH build (Block 3): populate dimensions + fact

Notebook: DWH.ipynb

DQ Excel (Task 5): generate Task_5_Inconcistencies_Analysis.xlsx

Notebook: Inconcistencies.ipynb

Exports (Task 6): export marts to CSV + ZIP and produce counts CSV

Script/notebook step produces:

Task_6_1_Data_Marts.zip

Task_6_2_Data_Marts_Rows.csv

Optional: schedule with Windows Task Scheduler to run daily.

Data model (short)

RAW (schema raw): landing table orders_landing (files as-is)

STG (schema stg):

orders_clean (typed/validated, delivery_days)

orders_exceptions (all DQ issues, soft + hard)

DWH (schema dwh):

Dims: dim_date, dim_customer, dim_product, dim_geo, dim_segment, dim_shipmode

Fact: fact_orders (measures: sales, quantity, discount, profit, delivery_days)

Key data quality rules (short)

Dates: parse Order Date & Ship Date pairwise; enforce file-month windows

Ambiguity: if both MM/DD and DD/MM fit → keep MM/DD + soft-warn

Same Day: if Ship Mode = "Same Day" but dates differ → soft-warn

Discount: must be between 0 and 1

Numbers: Sales/Quantity required; negative in latest month = hard reject

Duplicates: same (Order ID, Product ID) within a snapshot = hard reject

Profit: may be NULL; Sales>0 & Profit<0 = soft-warn

All soft/hard issues logged to stg.orders_exceptions and reported in Excel.
