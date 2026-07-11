# Employee Lifecycle & Attrition Analytics Pipeline

End-to-end **Data Engineering** project that ingests daily HR snapshots, tracks every historical change using **Slowly Changing Dimension (SCD) Type 2**, and generates workforce and attrition analytics using **Azure Data Factory**, **Azure Databricks (PySpark, Delta Lake)**, and a **Medallion Architecture (Bronze → Silver → Gold)**.

---

# 🎯 Project Goal

Process daily HR snapshot data, maintain complete employee history using **SCD Type 2**, and generate workforce and attrition analytics through a **Medallion Architecture (Bronze → Silver → Gold)** to provide HR leadership with reliable insights into headcount, employee tenure, and attrition trends.

---

# 📁 Repository Structure

```text
Employee-Lifecycle-Attrition-Pipeline/
├── README.md
├── databricks_notebook/
│   └── bronze_to_silver_to_gold.ipynb
├── outputs/
│   ├── employee_history_log.csv
│   ├── employee_current_snapshot.csv
│   ├── headcount_by_department.csv
│   ├── attrition_summary.csv
│   └── tenure_summary.csv
├── Raw_Datasets/
│   ├── day 1/
│   ├── day 2/
│   ├── day 3/
│   ├── day 4/
│   └── day 5/
└── Employee_Lifecycle_Attrition_Pipeline_Report.pdf
    └── (Project documentation with screenshots and summary)
```
# Tech Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| Ingestion | Azure Storage Account | Store Raw, Bronze, Silver, and Gold data |
| Ingestion | Azure Data Factory | Automated ingestion from Raw to Bronze |
| Processing | PySpark | Distributed data processing and transformations |
| Workspace | Azure Databricks | Development and execution environment |
| Storage | Delta Lake | ACID transactions and SCD Type 2 MERGE |
| Data Modeling | Medallion Architecture | Bronze → Silver → Gold pipeline |
| Historical Tracking | SCD Type 2 | Maintain complete employee history |
| Querying | SQL | CTEs, JOINs, CASE, GROUP BY, Window Functions |
| Programming | Python | Data generation and validation |



---

# Step 1 — Data Source

A real **5-day HR snapshot dataset** became available in the form of Spark-partitioned CSV files.

The dataset included daily employee snapshots (Day 1 to Day 5) along with five reference output files:

- employee_history_log
- employee_current_snapshot
- attrition_summary
- headcount_by_department
- tenure_summary

These reference files were used to validate the generated outputs.

---

# Step 2 — Azure Storage Setup

- Used an existing Azure Storage Account
- Created the following containers:
  - raw-employee-data
  - bronze-layer
  - silver-layer
  - gold-layer
- Uploaded the 5-day HR dataset into the **raw-employee-data** container with day-wise folders (Day 1–Day 5).

---

# Step 3 — Azure Data Factory (Bronze Ingestion)

- Created Azure Data Factory
- Configured a **Linked Service** to connect to Azure Storage.
- Created pipeline
- Used **Copy Data Activity** with:
  - Wildcard path (`*.csv`)
  - Recursive file traversal
- Successfully executed the pipeline using **Debug Run**
- Published the pipeline
- Raw data was successfully copied into the **Bronze Layer** while preserving the folder structure

---

# Step 4 — Azure Databricks Setup

- Created Databricks Workspace
- Created a Single Node Cluster
- Created notebook
- Connected Azure Storage using the Storage Account Access Key


---

# Step 5 — Bronze → Silver (PySpark + Delta Lake + SCD Type 2)

- Read all daily CSV files from the Bronze Layer.
- Combined all five daily snapshots into a single DataFrame.
- Added:
  - `batch_day`
  - `batch_date`
- Performed schema validation to detect schema drift.

### Implemented SCD Type 2 using Delta Lake MERGE

- Initialized the Silver History table using Day 1.
- Processed Days 2–5 using Delta MERGE.

For each incoming record:

- If department, salary, or employee status changed:
  - Closed the previous version (`end_date`, `is_current = 0`)
  - Inserted a new version (`is_current = 1`)
- Inserted new employees.
- Preserved complete employee history.

Generated:

- `employee_history_log`
- `employee_current_snapshot`

The current snapshot also included the calculated **tenure_years** column.

---

# Step 6 — Silver → Gold (Spark SQL)

Created temporary views and generated business-ready analytical tables.

### `headcount_by_department`

- Department-wise active employee count
- GROUP BY
- JOIN

### `attrition_summary`

- Daily resignations
- Attrition Rate %
- CTEs
- CASE statements
- Aggregations

### `tenure_summary`

- Department-wise average employee tenure
- AVG
- GROUP BY

---

# Step 7 — Output Validation

- Saved all five analytical tables into the Gold Layer as CSV files.
- Compared the generated outputs against the provided reference files.
- Successfully validated that all record counts and values matched.

---


# 🚀 How to Run

1. Create four Azure Storage containers:
   - raw-employee-data
   - bronze-layer
   - silver-layer
   - gold-layer

2. Upload the dataset into the **raw-employee-data** container.

3. Execute the Azure Data Factory pipeline to copy data from Raw to Bronze.

4. Create a Databricks Secret Scope and store the Storage Account key.

5. Import the notebook into Azure Databricks.

6. Attach the notebook to the cluster.

7. Run all notebook cells.

8. Final output CSV files will be generated in:
   - Gold Layer
   - `outputs/` folder

---

# 📊 Sample Outputs

| Output Table | Rows | Description |
|--------------|-----:|-------------|
| employee_history_log | 594 | Complete employee history using SCD Type 2 |
| employee_current_snapshot | 220 | Latest version of every employee |
| headcount_by_department | 25 | Department-wise headcount across 5 days |
| attrition_summary | 25 | Department-wise daily attrition metrics |
| tenure_summary | 5 | Average tenure for each department |

---

# Conclusion- 

- Implemented **SCD Type 2** efficiently using **Delta Lake MERGE** operations.
- Built an end-to-end **Medallion Architecture (Bronze → Silver → Gold)** for scalable data processing.
- Automated ingestion of nested directory structures using **Azure Data Factory** with wildcard and recursive file handling.
- Implemented schema drift handling to make the pipeline more robust and production-ready.
- Applied advanced Spark SQL techniques including **CTEs, Window Functions, JOINs, CASE statements, and Aggregations** to generate business-ready HR analytics.
