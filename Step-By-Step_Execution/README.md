# Step-by-Step Execution Guide: Adventure Works Data Pipeline

This document provides a comprehensive, visual walkthrough of the entire data engineering pipeline built on Microsoft Azure. It details the setup, configuration, and execution of resources across the Medallion Architecture (Bronze, Silver, Gold).

---

## 1. Project Architecture & Resource Provisioning

The pipeline is designed to ingest raw data, transform it using Spark, and serve it via serverless SQL for Power BI reporting.

**Project Architecture Diagram:**
![Architecture Diagram](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/1.jpeg)

**Resource Group Overview:**
All resources are centrally managed within the `adv_work_resource` group.
![Resource Group](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/2.png)

### Azure Data Lake Storage (ADLS Gen2) Setup
We configured an ADLS Gen2 storage account to act as our data lake, ensuring a hierarchical namespace is enabled for optimized big data processing.
![Create Storage Account](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/3.png)
![Enable Hierarchical Namespace](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/4.png)
![Storage Deployment Complete](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/5.png)
![Storage Account Overview](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/6.png)

We established three distinct containers to represent our Medallion architecture, plus a `parameters` container for ADF configuration:
![Containers Created](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/7.png)

---

## 2. Dynamic Data Ingestion (Azure Data Factory)

To ingest data dynamically from GitHub, we utilized Azure Data Factory.

**Data Factory Creation:**
![ADF Deployment](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/8.png)
![ADF Overview](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/9.png)

### Linked Services & Datasets
We created Linked Services for both the HTTP source (GitHub) and our destination (ADLS Gen2).
![HTTP Linked Service](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/10.png)
![ADLS Gen2 Linked Service](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/11.png)

We then defined datasets for our dynamic pipeline.
![ADF Pipeline Canvas](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/12.png)
![Source Dataset](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/13.png)
![Source Dataset Configuration](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/14.png)
![Sink Dataset](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/15.png)
![Sink Dataset Configuration](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/16.png)

### Parameterization
To make the pipeline dynamic, we uploaded a JSON parameter file (`p_git.json`) to the `parameters` container in ADLS. This file contains the relative URLs and sink folder names for each dataset.
![Parameter Container](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/17.png)
![JSON File Content](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/18.png)
![Parameter File Uploaded](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/19.png)

### Pipeline Execution
The `DynamicGitToRaw` pipeline uses a `Lookup` activity to read the JSON file, and a `ForEach` activity to iterate through each item, dynamically copying data into the Bronze layer.
![ForEach Activity](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/20.png)
![Lookup Activity Setup](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/21.png)
![Pipeline Debug & Lookup Output](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/22.png)
![JSON Array Output](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/23.png)
![Inside ForEach: Copy Data](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/24.png)
![Dynamic Content Configuration](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/25.png)
![Source Configuration inside ForEach](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/26.png)
![Sink Configuration inside ForEach](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/27.png)
![Pipeline Execution Success](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/28.png)

**Bronze Layer Output:** The `raw-data` container successfully populated with all Adventure Works tables.
![Raw Data Container](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/29.png)

---

## 3. Data Transformation & Security (Azure Databricks)

To process the data into the Silver layer, we spun up Azure Databricks.

**Databricks Provisioning:**
![Create Databricks](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/30.png)
![Databricks Deployment Complete](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/31.png)
![Databricks Overview](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/32.png)
![Cluster Configuration](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/33.png)
![Cluster Running](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/34.png)

### Security: Microsoft Entra ID (App Registration)
To securely authenticate Databricks with ADLS without hardcoding keys, we registered an application in Microsoft Entra ID.
![App Registrations Menu](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/35.png)
![Registering Application](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/36.png)
![App Overview & IDs](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/37.png)
![Generating Client Secret](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/38.png)
![Client Secret Created](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/39.png)
![Saving Credentials](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/40.png)

We then assigned the `Storage Blob Data Contributor` role to this newly registered app on our Storage Account.
![IAM Menu](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/41.png)
![Role Selection](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/42.png)
![Assign Access](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/43.png)
![Select App Registration](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/44.png)
![Role Assignment Complete](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/45.png)

### Silver Layer Processing
Using PySpark, we authenticated via OAuth, loaded the raw CSVs, enforced schemas, and wrote the transformed data back as optimized Parquet files.
![PySpark Transformation Script](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/46.png)

**Silver Layer Output:** Parquet files successfully written to the `transformed-data` container.
![Transformed Data Container](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/47.png)

---

## 4. Data Serving (Azure Synapse Analytics)

To expose our Silver data to Power BI, we created an Azure Synapse Analytics workspace utilizing a Serverless SQL pool.

**Synapse Provisioning:**
![Create Synapse Workspace](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/48.png)
![Synapse Security Config](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/49.png)
![Synapse Review & Create](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/50.png)
![Synapse Deployment Complete](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/51.png)

### Security: Managed Identity Role Assignment
Similar to Databricks, we granted the Synapse Workspace Managed Identity access to the ADLS account to seamlessly query the Parquet files.
![Role Assignment for Synapse](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/52.png)
![Select Managed Identity](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/53.png)
![Assigning Role](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/54.png)
![Role Assignment Complete](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/55.png)

### Gold Layer: External Tables & Views
Inside Synapse Studio, we structured our logical data warehouse.
![Synapse Studio](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/56.png)
![Creating Database](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/57.png)
![Database Overview](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/58.png)

We created a distinct `serving` schema.
![Create Schema Script](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/59.png)

We tested reading the Parquet files directly using `OPENROWSET` to create logical SQL views.
![Create Views Script](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/60.png)

To finalize the Gold layer, we established a Master Key, Database Scoped Credentials, and External Data Sources pointing to our ADLS paths, defining an External File Format for Parquet.
![Database Scoped Credentials](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/61.png)
![External Data Sources](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/62.png)

Finally, we executed queries to create explicit External Tables linked to the underlying data.
![Create External Tables](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/63.png)

**Gold Layer Output:** Data securely served and ready for downstream consumption.
![Serving Data Container](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/64.png)

---

## 5. Data Modeling & Visualization (Power BI)

With the pipeline complete, we connected Power BI directly to the Synapse Serverless SQL endpoint.
![Copying SQL Endpoint](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/65.png)
![Get Data -> Azure Synapse](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/66.png)

### Semantic Model
We imported the external tables and structured them into an optimized Star Schema, ensuring efficient query performance and DAX measure calculations.
![Data Modeling 1](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/67.jpg)
![Data Modeling 2](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/68.jpg)

### Final Dashboards
The culmination of the pipeline is a comprehensive, four-page interactive dashboard.

**The Executive Dashboard:**
![Executive Dashboard](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/adv_work_home-page_powerbi.jpg)

**Regional Sales & Map Analysis:**
![Regional Sales](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/adv_work_page-1_powerbi.jpg)

**Product & Inventory Deep-Dive:**
![Product Deep-Dive](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/adv_work_page-2_powerbi.jpg)

**Customer Intelligence:**
![Customer Intelligence](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/adv_work_page-3_powerbi.jpg)
