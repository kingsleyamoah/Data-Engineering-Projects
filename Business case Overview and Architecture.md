# Earthquake Azure Data Engineering Pipeline:

## Overview and Architecture

### Business Case

Earthquake data is incredibly valuable for understanding seismic events and mitigating risks. Government agencies, research institutions, and insurance companies rely on up-to-date information to plan emergency responses and assess risks. With this automated pipeline, we ensure these stakeholders get the latest data in a way that’s easy to understand and ready to use, saving time and improving decision-making.

### Architecture Overview

This pipeline follows a modular architecture, integrating Azure’s powerful data engineering tools to ensure scalability, reliability, and efficiency. The architecture includes:

1. **Data Ingestion**: Azure Data Factory orchestrates the daily ingestion of earthquake data from the USGS Earthquake API.
2. **Data Processing**: Databricks processes raw data into structured formats (bronze, silver, gold tiers).
3. **Data Storage**: Azure Data Lake Storage serves as the backbone for storing and managing data at different stages.
4. **Data Analysis**: Synapse Analytics enables querying and aggregating data for reporting.
5. **Optional Visualization**: Power BI can be used to create interactive dashboards for stakeholders.

### Data Modeling

I implement a **medallion architecture** to structure and organize data effectively:

1. **Bronze Layer**: Raw data ingested directly from the API, stored in Parquet format for future reprocessing if needed.
2. **Silver Layer**: Cleaned and normalized data, removing duplicates and handling missing values, ensuring it’s ready for analytics.
3. **Gold Layer**: Aggregated and enriched data tailored to specific business needs, such as adding in country codes.


### Key Benefits

- **Automation**: Eliminates manual data fetching and processing, reducing operational overhead.
- **Scalability**: Handles large volumes of data seamlessly using Azure services.
- **Actionable Insights**: Provides stakeholders with ready-to-use data for informed decision-making.

---

## Step 1: Create an Azure Account
1. I used an Azure account I previously created.

---

## Step 2: Create a Databricks Resource
1. Signed into  a Databricks resource in Azure.
2. Selected the **Standard LTS (Long Term Support)** tier.

---

## Step 3: Set Up a Storage Account
1. I Created a Storage Account and enable **hierarchical namespaces** in the advanced settings.
2. Creating the Medallion Architecture Storage Account resource:
   - Go to **Data Storage > Containers > + Containers**.
   - Create three containers: `bronze`, `silver`, and `gold`.
3. Configuring access:
   - Go to **IAM > Add role assignment > Storage Blob Data Contributor**.
   - Click **Next > Managed Identity > Select Members**.
   - Select **Access Connector for Azure Databricks** as the managed identity.
   - Click **Review + Assign**.

---

## Step 4: Configure Databricks
1. I opened the Databricks resource and click **Launch Workspace**.
2. Started a compute instance (this may take a few minutes).
3. Set up external data access:
4. Defined external locations:
   - Navigate to **External Data > External Locations**.
   - Assign a name, select the storage credential, and specify the URL (use the container name and storage account name for `bronze`, `silver`, and `gold`).


---

## Step 5: Create and Execute Notebooks
1. In the Databricks workspace, I created a notebook for each layer (`bronze`, `silver`, `gold`).
   - Executed the notebook and refresh the Storage Account containers to verify updates.
   - Repeated the process for `silver` and `gold` notebooks, adding the corresponding code.
     
---

## Step 6: Setting Up Azure Data Factory (ADF)
1. Created a new Azure Data Factory instance (in a new Resource Group).
2. Launched the ADF studio and create a pipeline:
   - Drag the **Notebook** activity into the pipeline and configure it to run Databricks notebooks.
   - Add a **Databricks Linked Service**:
     - Use the **AutoResolveIntegrationRuntime**.
     - Authenticate with an Access Token (recommended to store the token in a Key Vault for security).
3. Pass parameters to the pipeline:
   - For example, add parameters `start_date` and `end_date` with dynamic values using `@formatDateTime` expressions.
4. Chain notebooks (`bronze`, `silver`, `gold`) to create a pipeline with success dependencies.
5. Validate, publish, and run the pipeline.
6. Schedule the pipeline to run at desired intervals (e.g., daily).

---

## Step 7: Integrate Azure Synapse Analytics
1. **Created a Synapse Workspace**:
   - Linked it to the existing Storage Account.
   - Configured a file system and assign necessary permissions.
2. **Queryed Data Using Serverless SQL**:
   - Use `OPENROWSET` to query Parquet files stored in `bronze`, `silver`, and `gold` containers.
   - Example query:
     ```sql
     SELECT
         country_code,
         COUNT(CASE WHEN LOWER(sig_class) = 'low' THEN 1 END) AS low_count,
         COUNT(CASE WHEN LOWER(sig_class) IN ('medium', 'moderate') THEN 1 END) AS medium_count,
         COUNT(CASE WHEN LOWER(sig_class) = 'high' THEN 1 END) AS high_count
     FROM
         OPENROWSET(
             BULK 'https://<storage_account>.dfs.core.windows.net/gold/earthquake_events_gold/**',
             FORMAT = 'PARQUET'
         ) AS [result]
     GROUP BY
         country_code;
     ```
3. **Created External Tables** for structured access:
   - Defined external tables linked to the `gold` container for better organization and performance.
4. **Optimize Performance**:
   - Use indexing, partitioning, and caching as required.

---


## Key Considerations
- **Linked Services**: Ensure reusable and secure connections between Azure services.
- **Scalability**: Use Synapse for querying large datasets efficiently.
- **Data Engineering Focus**: Maintain an emphasis on structured pipelines and optimized workflows.


