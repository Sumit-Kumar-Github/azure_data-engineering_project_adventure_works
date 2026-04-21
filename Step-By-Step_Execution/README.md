# Step-by-Step Execution Guide: Adventure Works Data Pipeline

This document provides a comprehensive, visual walkthrough of the entire data engineering pipeline built on Microsoft Azure. It details the setup, configuration, and execution of resources across the Medallion Architecture (Bronze, Silver, Gold).

---

## 1. Project Architecture & Resource Provisioning

The pipeline is designed to ingest raw data, transform it using Spark, and serve it via serverless SQL for Power BI reporting.

**Project Architecture Diagram:**
![Architecture Diagram](1.jpeg)

**Resource Group Overview:**
All resources are centrally managed within the `adv_work_resource` group.
![Resource Group](2.jpg)

### Azure Data Lake Storage (ADLS Gen2) Setup
We configured an ADLS Gen2 storage account to act as our data lake, ensuring a hierarchical namespace is enabled for optimized big data processing.
![Create Storage Account](3.jpg)
![Enable Hierarchical Namespace](4.jpg)
![Storage Deployment Complete](5.jpg)
![Storage Account Overview](6.jpg)

We established three distinct containers to represent our Medallion architecture, plus a `parameters` container for ADF configuration:
![Containers Created](7.jpg)

---

## 2. Dynamic Data Ingestion (Azure Data Factory)

To ingest data dynamically from GitHub, we utilized Azure Data Factory.

**Data Factory Creation:**
![ADF Deployment](8.jpg)
![ADF Overview](9.jpg)

### Linked Services & Datasets
We created Linked Services for both the HTTP source (GitHub) and our destination (ADLS Gen2).
![HTTP Linked Service](10.jpg)
![ADLS Gen2 Linked Service](11.jpg)

We then defined datasets for our dynamic pipeline.
![ADF Pipeline Canvas](12.jpg)
![Source Dataset](13.jpg)
![Source Dataset Configuration](14.jpg)
![Sink Dataset](15.jpg)
![Sink Dataset Configuration](16.jpg)

### Parameterization
To make the pipeline dynamic, we uploaded a JSON parameter file (`p_git.json`) to the `parameters` container in ADLS. This file contains the relative URLs and sink folder names for each dataset.
![Parameter Container](17.jpg)
![JSON File Content](18.jpg)
![Parameter File Uploaded](19.jpg)

### Pipeline Execution
The `DynamicGitToRaw` pipeline uses a `Lookup` activity to read the JSON file, and a `ForEach` activity to iterate through each item, dynamically copying data into the Bronze layer.
![ForEach Activity](20.jpg)
![Lookup Activity Setup](21.jpg)
![Pipeline Debug & Lookup Output](22.jpg)
![JSON Array Output](23.jpg)
![Inside ForEach: Copy Data](24.jpg)
![Dynamic Content Configuration](25.jpg)
![Source Configuration inside ForEach](26.jpg)
![Sink Configuration inside ForEach](27.jpg)
![Pipeline Execution Success](28.jpg)

**Bronze Layer Output:** The `raw-data` container successfully populated with all Adventure Works tables.
![Raw Data Container](29.jpg)

---

## 3. Data Transformation & Security (Azure Databricks)

To process the data into the Silver layer, we spun up Azure Databricks.

**Databricks Provisioning:**
![Create Databricks](30.png)
![Databricks Deployment Complete](31.jpg)
![Databricks Overview](32.jpg)
![Cluster Configuration](33.jpg)
![Cluster Running](34.jpg)

### Security: Microsoft Entra ID (App Registration)
To securely authenticate Databricks with ADLS without hardcoding keys, we registered an application in Microsoft Entra ID.
![App Registrations Menu](35.jpg)
![Registering Application](36.jpg)
![App Overview & IDs](37.jpg)
![Generating Client Secret](38.jpg)
![Client Secret Created](39.jpg)
![Saving Credentials](40.png)

We then assigned the `Storage Blob Data Contributor` role to this newly registered app on our Storage Account.
![IAM Menu](41.jpg)
![Role Selection](42.jpg)
![Assign Access](43.png)
![Select App Registration](44.png)
![Role Assignment Complete](45.png)

### Silver Layer Processing
Using PySpark, we authenticated via OAuth, loaded the raw CSVs, enforced schemas, and wrote the transformed data back as optimized Parquet files.
![PySpark Transformation Script](46.jpg)

**Silver Layer Output:** Parquet files successfully written to the `transformed-data` container.
![Transformed Data Container](47.jpg)

---

## 4. Data Serving (Azure Synapse Analytics)

To expose our Silver data to Power BI, we created an Azure Synapse Analytics workspace utilizing a Serverless SQL pool.

**Synapse Provisioning:**
![Create Synapse Workspace](48.jpg)
![Synapse Security Config](49.jpg)
![Synapse Review & Create](50.jpg)
![Synapse Deployment Complete](51.jpg)

### Security: Managed Identity Role Assignment
Similar to Databricks, we granted the Synapse Workspace Managed Identity access to the ADLS account to seamlessly query the Parquet files.
![Role Assignment for Synapse](52.jpg)
![Select Managed Identity](53.jpg)
![Assigning Role](54.png)
![Role Assignment Complete](55.png)

### Gold Layer: External Tables & Views
Inside Synapse Studio, we structured our logical data warehouse.
![Synapse Studio](56.jpg)
![Creating Database](57.jpg)
![Database Overview](58.png)

We created a distinct `serving` schema.
![Create Schema Script](59.jpg)

We tested reading the Parquet files directly using `OPENROWSET` to create logical SQL views.
![Create Views Script](60.jpg)

To finalize the Gold layer, we established a Master Key, Database Scoped Credentials, and External Data Sources pointing to our ADLS paths, defining an External File Format for Parquet.
![Database Scoped Credentials](61.jpg)
![External Data Sources](62.jpg)

Finally, we executed queries to create explicit External Tables linked to the underlying data.
![Create External Tables](63.jpg)

**Gold Layer Output:** Data securely served and ready for downstream consumption.
![Serving Data Container](64.jpg)

---

## 5. Data Modeling & Visualization (Power BI)

With the pipeline complete, we connected Power BI directly to the Synapse Serverless SQL endpoint.
![Copying SQL Endpoint](65.jpg)
![Get Data -> Azure Synapse](66.jpg)

### Semantic Model
We imported the external tables and structured them into an optimized Star Schema, ensuring efficient query performance and DAX measure calculations.
![Data Modeling 1](67.jpg)
![Data Modeling 2](68.jpg)

### Final Dashboards
The culmination of the pipeline is a comprehensive, four-page interactive dashboard.

**The Executive Dashboard:**
![Executive Dashboard](adv_work_home-page_powerbi.jpg)

**Regional Sales & Map Analysis:**
![Regional Sales](adv_work_page-1_powerbi.jpg)

**Product & Inventory Deep-Dive:**
![Product Deep-Dive](adv_work_page-2_powerbi.jpg)

**Customer Intelligence:**
![Customer Intelligence](adv_work_page-3_powerbi.jpg)
