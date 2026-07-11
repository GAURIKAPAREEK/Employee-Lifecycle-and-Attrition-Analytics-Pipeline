Employee Lifecycle & Attrition Analytics Pipeline

End-to-end Data Engineering project that ingests daily HR snapshots, tracks every historical change using Slowly Changing Dimension (SCD) Type 2, and generates workforce and attrition analytics using Azure Data Factory, Azure Databricks (PySpark, Delta Lake), and a Medallion Architecture (Bronze → Silver → Gold).

🎯 Project Goal

Process daily HR snapshot data, maintain complete employee history using SCD Type 2, and generate workforce and attrition analytics through a Medallion Architecture (Bronze → Silver → Gold) to provide HR leadership with reliable insights into headcount, employee tenure, and attrition trends.

🏗 Architecture
🧰 Tech Stack
Layer	Technology	Purpose
Ingestion	Azure Storage Account	Store Raw, Bronze, Silver, and Gold data
Ingestion	Azure Data Factory	Automated ingestion from Raw to Bronze
Processing	PySpark	Distributed data processing and transformations
Workspace	Azure Databricks	Development and execution environment
Storage	Delta Lake	ACID transactions and SCD Type 2 MERGE
Data Modeling	Medallion Architecture	Bronze → Silver → Gold pipeline
Historical Tracking	SCD Type 2	Maintain complete employee history
Querying	SQL	CTEs, JOINs, CASE, GROUP BY, Window Functions
Programming	Python	Data generation and validation
📁 Repository Structure
Employee-Lifecycle-Attrition-Pipeline/
├── README.md
├── notebooks/
│   └── bronze_to_silver_to_gold.ipynb
├── outputs/
│   ├── employee_history_log.csv
│   ├── employee_current_snapshot.csv
│   ├── headcount_by_department.csv
│   ├── attrition_summary.csv
│   └── tenure_summary.csv
├── raw_datasets/
│   ├── day 1/
│   ├── day 2/
│   ├── day 3/
│   ├── day 4/
│   └── day 5/
└── Employee Lifecycle & Attrition Analytics Pipeline Report.pdf
    └── (Project documentation with screenshots and summary)
Step 1 — Data Source

Initially, a mock HR dataset was planned to be generated using Python scripts. Later, a real 5-day HR snapshot dataset became available in the form of Spark-partitioned CSV files.

The dataset included daily employee snapshots (Day 1 to Day 5) along with five reference output files:

employee_history_log
employee_current_snapshot
attrition_summary
headcount_by_department
tenure_summary

These reference files were used to validate the generated outputs.

Step 2 — Azure Storage Setup
Used an existing Azure Storage Account: storage12accountgp
Created the following containers:
raw-employee-data
bronze-layer
silver-layer
gold-layer
Uploaded the 5-day HR dataset into the raw-employee-data container with day-wise folders (Day 1–Day 5).
Step 3 — Azure Data Factory (Bronze Ingestion)
Created Azure Data Factory:
adf-hrpipeline-gp
Configured a Linked Service to connect to Azure Storage.
Created pipeline:
pl_copy_raw_employee_data
Used Copy Data Activity with:
Wildcard path (*.csv)
Recursive file traversal
Successfully executed the pipeline using Debug Run.
Published the pipeline.
Result:
Raw data was successfully copied into the Bronze Layer while preserving the folder structure.
Step 4 — Azure Databricks Setup
Created Databricks Workspace:
dbw-hr-analytics
Created a Single Node Cluster:
16 GB RAM
4 Cores
Auto Termination Enabled
Created notebook:
bronze_to_silver_to_gold
Connected Azure Storage using the Storage Account Access Key.
Later replaced the hardcoded key with Databricks Secret Scope for better security.
Step 5 — Bronze → Silver (PySpark + Delta Lake + SCD Type 2)
Read all daily CSV files from the Bronze Layer.
Combined all five daily snapshots into a single DataFrame.
Added:
batch_day
batch_date
Performed schema validation to detect schema drift.
Implemented SCD Type 2 using Delta Lake MERGE
Initialized the Silver History table using Day 1.
Processed Days 2–5 using Delta MERGE.

For each incoming record:

If department, salary, or employee status changed:
Closed the previous version (end_date, is_current = 0)
Inserted a new version (is_current = 1)
Inserted new employees.
Preserved complete employee history.

Generated:

employee_history_log
employee_current_snapshot

The current snapshot also included the calculated tenure_years column.

Step 6 — Silver → Gold (Spark SQL)

Created temporary views and generated business-ready analytical tables.

headcount_by_department
Department-wise active employee count
GROUP BY
JOIN
attrition_summary
Daily resignations
Attrition Rate %
CTEs
CASE statements
Aggregations
tenure_summary
Department-wise average employee tenure
AVG
GROUP BY
Step 7 — Output Validation
Saved all five analytical tables into the Gold Layer as CSV files.
Compared the generated outputs against the provided reference files.
Successfully validated that all record counts and values matched.
Step 8 — Security Improvement

Initially, the Azure Storage Access Key was hardcoded inside the notebook.

This was replaced by Databricks Secret Scope, and the key is now retrieved securely using:

dbutils.secrets.get(scope, key)

This prevents sensitive credentials from being exposed inside notebooks.

🚀 How to Run
Create four Azure Storage containers:
raw-employee-data
bronze-layer
silver-layer
gold-layer
Upload the dataset into the raw-employee-data container.
Execute the Azure Data Factory pipeline to copy data from Raw to Bronze.
Create a Databricks Secret Scope and store the Storage Account key.
databricks secrets create-scope hr-pipeline-scope
databricks secrets put-secret hr-pipeline-scope storage-account-key
Import the notebook into Azure Databricks.
Attach the notebook to the cluster.
Run all notebook cells.
Final output CSV files will be generated inside:
Gold Layer
outputs/ folder
📊 Sample Outputs
Output Table	Rows	Description
employee_history_log	594	Complete employee history using SCD Type 2
employee_current_snapshot	220	Latest version of every employee
headcount_by_department	25	Department-wise headcount across 5 days
attrition_summary	25	Department-wise daily attrition metrics
tenure_summary	5	Average tenure for each department
🛠 Challenges & Solutions
#	Challenge	Solution
1	Storage Account key was hardcoded	Replaced with Databricks Secret Scope
2	Attrition summary calculated cumulative resignations	Rewrote the SQL logic to count only daily resignations
3	SCD Type 2 used join_date as start_date	Updated logic to use the actual batch processing date
4	union() failed when schemas changed	Implemented unionByName(allowMissingColumns=True) with schema validation
5	SQL window functions were missing	Added LAG() and ROW_NUMBER() implementations
6	Spark outputs were generated as multiple part files	Combined all part files into a single CSV using Python
7	Duplicate output file caused by trailing whitespace	Removed the duplicate file and standardized file names
💡 Key Learnings
Implemented SCD Type 2 efficiently using Delta Lake MERGE operations.
Built an end-to-end Medallion Architecture (Bronze → Silver → Gold) for scalable data processing.
Automated ingestion of nested directory structures using Azure Data Factory with wildcard and recursive file handling.
Learned secure credential management using Databricks Secret Scope instead of hardcoded secrets.
Implemented schema drift handling to make the pipeline more robust and production-ready.
Applied advanced Spark SQL techniques including CTEs, Window Functions, JOINs, CASE statements, and Aggregations to generate business-ready HR analytics.
