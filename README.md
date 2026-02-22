# DE Assignment | True Digital Group
## 📂 Part 1). SQL Technical Assessment: SF Bikeshare Data
The answers are in the file: `notebooks/part_1_sf_bike_share.ipynb`
I answer 5 questions from the assignment.  
I also create an ER Diagram and design data modeling for this project.

For each question, I provide
- Data Quality insight  
- Data Engineering (DE) insight  
- Business insight from the query result

## 📂 Part 2). Engineer Skills
### 📊 1. Data Reconciliation:

**Row Count Check** 
- compare row count between source and target after load.

**Checksum** 
- compare sum, min, max, and average of important columns for example `duration_sec` between source and target.

**Schema Check** 
- compare column name and data type between source and target. Use dltHub to handle schema change like new column or type change.

**Freshness Check:** 
- check latest `start_date` in source and target to make sure data is up to date.

**Join Validation:** 
- join fact and dimension tables and check that unmatched record = 0.

**Automation:** 
- run all reconciliation checks in Airflow as part of the pipeline and fail the task when result not match rule.

### 📊 2. Pipeline Orchestration: 
<img width="1637" height="663" alt="Screenshot 2569-02-22 at 20 50 55" src="https://github.com/user-attachments/assets/a65f78ff-a1b4-46cd-b3eb-611571e4b449" />
This data pipeline follows the Medallion Architecture with the flow below.

#### Architecture Overview

The pipeline follows this flow **Data Source → Bronze → Silver → Gold → Data Consumers**

- Orchestrated by **Airflow**
- Deployed with **Docker** and **Kubernetes**
- Transformed with **dbt**
- Ingested with **dltHub**


#### 1. Orchestration & Deployment

- **Airflow** controls all data pipelines in this project.  
- **Docker** and **Kubernetes** deploy the pipeline as containers so it can run in a real environment.


#### 2. Data Source

- Source data comes from BigQuery dataset:  `bigquery-public-data.san_francisco_bikeshare`
- **dltHub** pulls data by API request
- Handles **schema evolution** when the source schema changes for example new columns, change data type, create index.


#### 3. Bronze Layer 

- Raw data is stored in **Google Cloud Storage (GCS)** as **Parquet files**.  
- Data loads in:
  - Incremental mode  
  - Snapshot mode  
- Data can duplicate but is separated by load date (`dt`).  
- **External tables** are created in BigQuery for the Silver layer to read.


#### 4. Silver Layer 

- Incremental data is merged and **deduplicated**, keeping only the latest record.  

##### Transformations:
- Standardize data format  
- Clean null values  
- Cast data types (Date, Int, Float)  
- Join multiple tables  

- Data is stored in GCS and created as **native tables** in BigQuery.  
- Tables use **SCD Type 2 (Slowly Changing Dimension Type 2)** to keep history and not overwrite old records.


#### 5. Gold Layer 

#### 5.1 dbt Semantic Layer

This layer defines business meaning of data.

- **Semantic Models**
  - Entities  
  - Measures  
  - Dimensions  

- **Metrics**
  - Simple  
  - Derived  
  - Cumulative  
  - Conversion  

This ensures all users use the same business logic.


#### 5.2 Data Modeling

- Data is modeled as a **Star Schema**:
  - Fact tables  
  - Dimension tables  

This design supports fast analytics and BI tools.

#### 6. Data Consumers

- **Power BI** for dashboards and reports  
- **Vertex AI** for machine learning models  
- **Users** for ad-hoc analysis


#### 7. Supporting Components

- **Data Catalog**  
  - Shows what data exists and where it is  
  - Helps users find and understand data  

- **Data Quality**  
  - Ensures data is reliablesee details in Part 3: Data Quality Assurance


#### Tech Stack

- Airflow  
- Docker  
- Kubernetes  
- dltHub  
- dbt  
- Google Cloud Platform (BigQuery, GCS)

---

### 📊 3. Data Quality Assurance:

| Test Category | Columns | Purpose |
| :--- | :--- | :--- |
| **Null Check** | `station_id`, `name` | Ensure critical identifiers are never missing. |
| **Unique Check** | `station_id` | Enforce primary key integrity. |
| **Referential** | `region_id` | Validate that all stations map to a valid region. |
| **Range Check** | `lat`, `lon` | Geospatial validation (Lat: -90 to 90). |
| **Consistency** | `station_id` vs `name` | Ensure 1:1 mapping between IDs and Names. |
| **Schema Evolution** | All columns | Detect new or changed columns and keep schema in sync. |

- Run data quality checks in Airflow, fail task when rule break, and send alert when condition fail.

THX for review kub :)
I really appreciate your time and feedback.
