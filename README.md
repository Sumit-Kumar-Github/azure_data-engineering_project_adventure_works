# Enterprise Data Pipeline: Adventure Works Analytics

## 📝 Project Overview
This project demonstrates an end-to-end, enterprise-grade Data Engineering pipeline on Microsoft Azure. Utilizing the **Medallion Architecture** (Bronze, Silver, Gold layers), the pipeline orchestrates the ingestion, transformation, and serving of the Adventure Works dataset. The final output is a comprehensive, multi-page PowerBI dashboard designed for executive decision-making, regional sales tracking, and customer intelligence.

## 🏗️ Architecture Diagram
![Architecture Diagram](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/1.jpeg)

The architecture follows a standard data lakehouse pattern:
1. **Raw Data Store (Bronze):** Ingestion of source data into Azure Data Lake Storage Gen2.
2. **Transformation (Silver):** Data cleaning and processing using Azure Databricks (PySpark).
3. **Serving (Gold):** Optimized views and external tables created via Azure Synapse Analytics (Serverless SQL).
4. **Reporting:** Interactive dashboards built in Power BI.

## 🛠️ Technology Stack
* **Azure Data Factory (ADF):** Metadata-driven data ingestion and pipeline orchestration.
* **Azure Data Lake Storage (ADLS Gen2):** Hierarchical cloud storage.
* **Azure Databricks:** Distributed data processing and transformation using PySpark.
* **Azure Synapse Analytics:** Serverless SQL pools for defining external tables and logical data warehousing.
* **Power BI:** Data modeling, DAX, and interactive data visualization.
---

## 🚀 Step-by-Step Implementation

### Phase 1: Environment & Security Setup
* Provisioned a central Resource Group (`adv_work_resource`) to house all services.
* Deployed **Azure Data Lake Storage Gen2** (`advworkstorage`) with hierarchical namespace enabled.
* Created three distinct containers to enforce the Medallion architecture: `raw-data`, `transformed-data`, and `serving-data`.
* **Security configuration:** Registered an application (`adv-work-app`) in Microsoft Entra ID. Assigned `Storage Blob Data Contributor` roles to both the App Registration and the Synapse Workspace Managed Identity to ensure secure, credential-less data access across the pipeline.

### Phase 2: Dynamic Data Ingestion (ADF)
* Engineered a dynamic, metadata-driven pipeline (`DynamicGitToRaw`) in Azure Data Factory.
* Utilized an HTTP Linked Service to connect to external CSV data files.
* Leveraged a JSON parameter file (`p_git.json`) alongside `Lookup` and `ForEach` activities to dynamically iterate through source URLs and sink folders.
* Ingested multiple tables (Calendar, Customers, Product Categories, Subcategories, Products, Returns, Sales) into the Bronze layer (`raw-data`).

![ADF Pipeline](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/28.png)

### Phase 3: Data Transformation (Databricks)
* Deployed an Azure Databricks workspace (`adv-work-db`) and configured a standard compute cluster.
* Established a secure connection to ADLS Gen2 using **OAuth 2.0** and the App Registration's Client ID/Secret.
* Developed PySpark notebooks to read the raw CSV data, enforce schemas, perform necessary data cleansing, and write the optimized data back to the Silver layer (`transformed-data`) in **Parquet** format.

![Databricks Script](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/46.png)

### Phase 4: Data Serving (Synapse Analytics)
* Deployed an Azure Synapse Analytics workspace (`adv-work-sa`) utilizing a Serverless SQL pool.
* Created a logical data warehouse by defining a custom schema (`serving`).
* Generated Database Scoped Credentials and External Data Sources pointing to the transformed ADLS directories.
* Created **SQL Views** and **External Tables** (`ext_calendar`, `ext_customers`, etc.) over the Parquet files to prepare the data for optimized BI consumption in the Gold layer (`serving-data`).

![Synapse External Tables](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/63.png)

### Phase 5: Data Modeling & Reporting (Power BI)
Connected Power BI to the Synapse Serverless SQL endpoint to build a robust semantic model. The relationships were defined in a **Star Schema** to ensure high-performance filtering and calculations.

**Dashboard Highlights:**
1. **The Executive Dashboard:** High-level KPIs (Revenue: ₹24.91M, Profit Margin: 41.97%), year-over-year revenue trends, and top 10 products by profit.
   ![Executive Dashboard](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/adv_work_page-1_powerbi.png)
2. **Regional Sales & Map Analysis:** Geographic performance visualization and detailed territory scorecards tracking revenue, profit, and order volume.
   ![Regional Sales](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/adv_work_page-1_powerbi.png)
3. **Product & Inventory Deep-Dive:** Decomposition trees mapping the revenue path from Category to specific Products, and risk analysis scatter plots correlating revenue against return rates.
   ![Product Deep-Dive](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/adv_work_page-2_powerbi.png)
4. **Customer Intelligence:** Granular analysis of loyal customers, demographic filtering (Gender, Marital Status, Education), and revenue distribution by annual income levels.
   ![Customer Intelligence](https://github.com/Sumit-Kumar-Github/zz_Raw-Data/blob/744c992f7aff25934987df83a1813824e1dbecc2/adventure-work-data/images/adv_work_page-3_powerbi.png)

---

## 📂 Repository Structure
* `/Databricks_notebook/`: Contains the `.ipynb` files used for Silver layer transformations.
* `/PowerBi_dashboard/`: Contains the `.pbix` file.
* `/ADF_parameters/`: Contains the `p_git.json` used for dynamic pipeline routing.
* `/Step-By-Step_Execution/`: Contains a dedicated README with a comprehensive, image-backed walkthrough detailing every configuration step across the Azure pipeline.
