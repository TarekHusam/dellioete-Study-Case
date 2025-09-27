
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
