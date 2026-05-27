---
  title: 'Foundry Fabric Workshop'
  module: 'Analyzing data with Microsoft Fabric'
---

# Analyzing data with Microsoft Fabric and Azure Content Understanding

## Step 1: Set Up Content Understanding for Audio Analysis

### 1.1 Create a Microsoft Foundry Resource (for Content Understanding)
1. Go to Azure Portal, click **Create a resource**....

2. Search for **Microsoft Foundry** and select it
3. Click **Create**
4. Configure the resource:
   - **Subscription:** Select your subscription
   - **Resource group:** `rg-documents`
   - **Name:** `ai-content-<yourname>` (must be globally unique)
   - **Region:** `Australia East`
   - **Default project name:** project-cu
5. Click **Review + Create**, then **Create**
6. Wait for deployment to complete (typically 2-3 minutes)
7. Once deployed, click **Go to resource**
8. Navigate to **Resource Management > Keys and Endpoint** in the left menu
9. Copy and save (you'll need this later):
   - **KEY 1** 
   - **Foundry > API endpoint** URL

### 1.2 Access Content Understanding Studio
1. Navigate to [Content Understanding Studio](https://contentunderstanding.ai.azure.com/)
2. Sign in with your Azure account credentials
3. Select **Explore Content Understanding**

### 1.3 Select Your Resource
1. Click on the settings icon near your profile icon in the top right
2. Select **Add resource**
3. Configure your resource:
   - **Subscription:** Select your subscription
   - **Resource group name:** Select `rg-documents`
   - **Resource name:** Select `ai-content-<yourname>`
   - Enable auto-deployment for required models if no default deployment available
   - Click **Next**
4. Click **Save** to confirm your resource selection
5. Verify that models are deployed successfully for the selected resource

### 1.4 Create a New Content Understanding Project
1. Select **Build**
2. Click **+ Create**
3. Configure the project:
   - **Project name:** `call-center-analysis`
   - **Description:** `Analyze customer service call recordings to extract insights`
   - **Project Type:** Select **Extract content and field with custom schema**
4. Configure Advanced settings:
   - **Connected resource:** Select `ai-content-<yourname>`
   - **Subscription:** Select your subscription
   - **Resource Group:** Select `rg-documents`
   - **Storage Account:** Select `stdocumentai<yourname>`
   - **Blob Container:** Select `call-recordings`
   - **Model for analysis:** Select the default model
5. Click **Create**

### 1.5 Upload Call Recordings
1. In your new project, click **Browse for files**
2. Upload one of the call recordings from the `Recordings` folder
3. Select Audio Analysis
4. Click **Save**

### 1.6 Define Schema for Extraction
You'll now define what information to extract from the call recordings.

1. In your project, navigate to **Schema**
2. Click **+ Add field** to create each extraction field
3. Add the following fields:

   - **Field name:** `CustomerName`
     - **Description:** `The name of the customer calling`
     - **Type:** String
     - **Method:** Generate
   
   - **Field name:** `AgentName`
     - **Description:** `The name of the customer service agent`
     - **Type:** String
     - **Method:** Generate
     
   - **Field name:** `CallSummary`
     - **Description:** `A brief summary of what the call was about`
     - **Type:** String
     - **Method:** Generate
     
   - **Field name:** `CallResolution`
     - **Description:** `Determine if the agent was able to resolve the customer's issue (e.g., Resolved, Unresolved, Escalated)`
     - **Type:** String
     - **Method:** Classify
     - **Categories:**
         - Add category: 
            - Category Name: Resolved 
            - Category Description: The customer's issue was resolved during the call
            - Category Name: Unresolved
            - Category Description: The customer's issue was not resolved during the call
            - Category Name: Escalated
            - Category Description: The customer's issue was escalated to a higher level of support
         - Click **Save**
   
   - **Field name:** `ProductName`
     - **Description:** `Identify any product names mentioned during the call`
     - **Type:** String
     - **Method:** Generate
   
   - **Field name:** `CallSentiment`
     - **Description:** `Identify the overall sentiment of the call (Positive, Neutral, Negative)`
     - **Type:** String
     - **Method:** Classify
       - **Categories:**
          - Add category: 
               - Category Name: Positive 
               - Category Description: The customer expresses positive sentiment during the call
               - Category Name: Neutral
               - Category Description: The customer expresses neutral sentiment during the call
               - Category Name: Negative
               - Category Description: The customer expresses negative sentiment during the call
          - Click **Save**

4. Click **Save**

### 1.7 Process Audio Files and Review Results
1. Click **Run analysis** at the upper left.
2. Wait for the processing to complete. This may take several minutes depending on the length of the audio and the complexity of the analysis.
3. Once processing is complete, you can review the extracted information for the call recording in the **Fields** tab.
4. Review each field's value
5. Select the **Result** tab to see the structured JSON output of the analysis, which includes all extracted fields and their values.
---

### 1.8 Analyze Remaining Recordings
1. Select **Browse for files** again to upload the next call recording from your local machine
2. Click **Run analysis** 
3. Review extracted information for each call and the JSON output.

### 1.9 Build the Analyzer
1. Select **Build analyzer** from the top beside **Run analysis**
2. Configure the analyzer:
   - **Name:** `callanalyzer`
   - **Description:** `Analyzer for processing call center recordings and extracting insights`
   - Click **Build**
3. Select **Jump to analyzer list**
4. Select the `callanalyzer` you just built and review the details
5. View the Code Examples for how to call this analyzer via code

---

## Step 2: Provision a Fabric Workspace and Configure Capacity

Microsoft Fabric workspaces are the foundational containers for all Fabric items
(Lakehouses, Warehouses, Notebooks, Semantic Models, Reports). A workspace must be
backed by a Fabric capacity (F SKU) to enable Fabric-specific features.

> **Design note:** Fabric capacities are billed based on compute usage (CU seconds),
> not storage. OneLake storage is billed separately per GB stored. This lab uses an
> F2 capacity, which provides a good balance of performance and cost for learning
> and development scenarios.

### 2.1 Deploy a Fabric Capacity (F2) in Azure Portal

Before creating a Fabric workspace, you need to provision a Fabric capacity in Azure. 
This capacity provides the compute resources for all Fabric workloads.

1. Sign in to the [Azure Portal](https://portal.azure.com).

2. In the search bar at the top, type **Microsoft Fabric** and select **Microsoft Fabric** 
   from the results.

3. Select **+ Create** to create a new Fabric capacity.

4. On the **Basics** tab, configure the following settings:

   | Setting | Value |
   | --- | --- |
   | Subscription | Select your Azure subscription |
   | Resource group | Select **Create new** and type `rg-fabric-workshop` |
   | Capacity name | Enter `fabriccapacityworkshop` |
   | Region | Select a region close to your location (e.g., Australia East or West Europe) |
   | Size | Select **F2** (2 capacity units) |

   > **Note:** F2 is the smallest Fabric capacity SKU and is suitable for development 
   > and learning scenarios. Production workloads typically require F4 or higher.

5. Select **Review + create**.

6. Review the configuration and estimated cost. Fabric capacities are billed per hour 
   while running.

7. Select **Create** to deploy the capacity.

8. Wait for the deployment to complete (typically 2-5 minutes). You can monitor progress 
   in the **Notifications** panel.

9. Once deployed, select **Go to resource** to view your Fabric capacity.

> **Cost Management Tip:** Fabric capacities can be paused when not in use to avoid 
> charges. To pause a capacity, navigate to the capacity resource in Azure Portal and 
> select **Pause** from the toolbar. Remember to resume it before starting the lab.

### 2.2 Create a Fabric Workspace

1. Go to [Fabric portal](https://app.fabric.microsoft.com), select **Workspaces** from the left navigation pane.

2. Select **+ New workspace**.

3. Configure the workspace:

   | Setting | Value |
   | --- | --- |
   | Name | `FabricWorkspace-Intermediate` |
   | Description | `Intermediate lab for Lakehouse, Warehouse, and Direct Lake analytics` |
   | Workspace type | **Fabric** |
   | Details | Select your **F2** capacity from the dropdown |

4. Select **Apply** to create the workspace.

5. You are now in your workspace. The workspace is empty and ready for Fabric items.

---

## Step 3: Create a Lakehouse and Understand OneLake Storage

A Lakehouse in Fabric combines the flexibility of a data lake (file storage, open formats)
with the query performance of a data warehouse (SQL analytics endpoint, automatic schema
discovery). Every Lakehouse is backed by OneLake, which stores data in Delta Lake format.

### 3.1 Create a Lakehouse

1. In your workspace (**FabricWorkspace-Intermediate**), select **+ New item**.

2. Under **Store data**, select **Lakehouse**.

3. Name the Lakehouse `LakehouseIntermediate`, enable Lakehouse schemas and select **Create**.

4. After a few moments, your Lakehouse opens. You will see two main sections:
   - **Tables** — structured Delta tables with schema and indexing
   - **Files** — unstructured or semi-structured files (Parquet, CSV, JSON)

5. On the upper left, notice the **Notebook** and **SQL analytics endpoint** selections. The SQL
   analytics endpoint is automatically created and allows you to query tables using T-SQL
   without any ETL.

### 3.2 Understand OneLake and Delta Lake

1. In the Lakehouse explorer, select **Files**.

2. OneLake automatically organizes files and tables under your workspace. All data is
   stored in **Delta Lake format** by default, which provides:
   - ACID transactions
   - Schema enforcement and evolution
   - Time travel (query historical versions)
   - Efficient upserts and deletes

3. OneLake is accessible via ABFS paths (Azure Blob File System) and supports shortcuts
   to external storage accounts (ADLS Gen2, AWS S3, etc.) without copying data.

---
## Step 4: Create a Fabric Notebook

The Fabric notebook is used to execute all sentiment analysis logic, including calling Microsoft Foundry, parsing results, and saving outputs.

### 4.1 Create a Notebook

1. Inside the same workspace, click **New item** select notebook
2. Name your Notebook
   - Notebook name: Notebook_Intermediate
   - Location: FabricWorkspace-Intermediate
3. Click Create

### 4.2 Configure Secure Access to Azure AI Services

The notebook retrieves Microsoft Foundry credentials securely from Azure Key Vault to avoid hard‑coding secrets.
This configuration allows the notebook to authenticate when calling the Content Understanding analyzer.

1. A fabric notebook is composed of cells. Each cell can be either:
   - Code for Python execution
   - Markdown for explanation and step labels

2. Click + code
3. A new empty code cell appear

### 4.3 Notebook Implementation

Below shows exactly how your code should be organized inside the notebook

1. Click + Markdown and paste: 
2. Code Cell 1: Import Libraries & Initialize Variables

```
file_url = ""

from notebookutils import credentials
import requests
import time
from pyspark.sql import Row
from datetime import datetime, timezone
```
 This cell sets up all required libraries.

3. Add code cell 
4. Code Cell 2: Retrieve Secrets from Azure Key Vault
 
```
# Retrieve Secret from the Key Vault
azure_ai_services_key = credentials.getSecret(
    "https://telavikeyvault1.vault.azure.net",
    "cukey"
)

azure_ai_services_endpoint = credentials.getSecret(
    "https://telavikeyvault1.vault.azure.net",
    "cuendpoint"
)
```
5. Code Cell 3: Analyzer Configuration
```
ANALYZER_ID = "telavianalyzer"
API_VERSION = "2024-10-01"

processed_rows = []
utc_time = datetime.now(timezone.utc)
```

6. Code Cell 5: Submit File to Content Understanding Analyzer
```
def analyze_file(analyzer_id, file_url):
    url = f"{azure_ai_services_endpoint.rstrip('/')}/contentunderstanding/analyzers/{analyzer_id}:analyze"
    headers = {
        "Ocp-Apim-Subscription-Key": azure_ai_services_key,
        "Content-Type": "application/json"
    }
    payload = {
        "inputs": [{"url": file_url}]
    }

    response = requests.post(
        url,
        headers=headers,
        params={"api-version": API_VERSION},
        json=payload
    )

    if response.status_code not in (200, 202):
        raise Exception(response.text)

    return response.headers["Operation-Location"]
```

7. Code Cell 6: Poll Analyzer Status
```
def poll_status(operation_location):
    headers = {"Ocp-Apim-Subscription-Key": azure_ai_services_key}
    response = requests.get(operation_location, headers=headers)
    response.raise_for_status()
    return response.json()
```

8. Code Cell 7: Run End-to-End Analysis
```
operation_location = analyze_file(ANALYZER_ID, file_url)

while True:
    result = poll_status(operation_location)
    status = result.get("status")

    if status == "Succeeded":
        fields = result["result"]["contents"][0]["fields"]

        processed_rows.append(Row(
            Customername=fields.get("customername", {}).get("valueString"),
            Agentname=fields.get("agentname", {}).get("valueString"),
            Callsentiment=fields.get("callsentiment", {}).get("valueString"),
            Product=fields.get("product", {}).get("valueString"),
            Resolution=fields.get("resolution", {}).get("valueString"),
            Emotion=fields.get("emotion", {}).get("valueString"),
            Callcategory=fields.get("callcategory", {}).get("valueString"),
            DateTime=utc_time,
            File=file_url
        ))
        break

    elif status in ["Failed", "Cancelled"]:
        raise Exception(f"Analyzer failed: {status}")

    time.sleep(10)
```

9. Code Cell 8: Create DataFrame and Display Output
```
processed_rows = [
    {k: (v if v is not None else "no data") for k, v in row.asDict().items()}
    for row in processed_rows
]

dataframe = spark.createDataFrame(processed_rows)
display(dataframe)
```

10. Code Cell 9: Save Results to Fabric Table
```
dataframe.write.mode("append").saveAsTable("analyzed_calls")
```

### Step 6: Verify the Output Table in the Lakehouse

1. Navigate to Onelake
2. Expanding OneLake
3. Checking tables > dbo > analyzed_calls
4. Previewing data
5. Running SQL validation queries

### Step 7: Configure Pipeline with Blob Storage Trigger

Automatically trigger the Fabric pipeline when a new file is uploaded to Azure Blob Storage and pass the file URL to the notebook.

## 7.1 Create or Open the Fabric Pipeline
1. Pipeline Name
2. Activity: Notebook - AnalyzeCalls

This notebook activity will execute your sentiment analysis notebook.

## 7.2 Define a Pipeline Parameter
1. Click on the ***blank canvas*** not the notebook activity
2. Open the ***Settings*** panel at the bottom
3. Under ***Base parameters***, click + ***New***

Create:
   - Name file_url
   - Type: String

This parameter will receive the blob file path from the trigger.

## Step 7.3: Correct concat() Expression for Blob File URL
Use this exact expression in the file_url parameter ***Value*** field:
```
@concat(
  'https://telavistorage.blob.core.windows.net/call-recordings/',
  pipeline().TriggerEvent.FileName
)
```
This builds the full public blob URL:
```
https://telavistorage.blob.core.windows.net/call-recordings/<uploaded-file>
```
This syntax follows Fabric Data Factory expression rules.

## Step 7.5 Map Pipeline Parameter to Notebook Parameter
1. Click the ***AnalyzeCalls*** notebook activity
2. Open ***Settings**

file_url → @pipeline().parameters.file_url
This passes the blob URL into your notebook variable:
```
file_url = ""







