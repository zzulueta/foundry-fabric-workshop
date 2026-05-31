---
  title: 'Foundry Fabric Workshop'
  module: 'Analyzing data with Microsoft Fabric'
---

# Analyzing data with Microsoft Fabric and Azure Content Understanding

## Step 1: Set Up Content Understanding for Audio Analysis

### 1.1 Create a Microsoft Foundry Resource (for Content Understanding)
1. Go to https://portal.azure.com click **Create a resource**.

2. Search for **Microsoft Foundry** and select it
3. Click **Create**
4. Configure the resource:
   - **Subscription:** Select your subscription
   - **Resource group(create new):** rg-documents`
   - **Name:** `ai-content-<yourname>` (must be globally unique)
   - **Region:** `Australia East`
   - **Default project name:** project-cu
5. Click **Review + Create**, then **Create**
6. Wait for deployment to complete (typically 2-3 minutes)
7. Once deployed, click **Go to resource**
8. Navigate to **Resource Management > Keys and Endpoint** in the left menu
9. Copy and save (you will need these later):
   - **KEY 1** 
   - **Foundry > API endpoint** URL

### 1.2 Create Storage Account
Azure Blob Storage provides scalable, cost‑effective storage for unstructured data such as audio files

1. In the **Azure portal**, use the **top search bar**
2. Type **Storage accounts** and select **Storage accounts.**
3. Select **+ Create**
4. Configure the storage account with the following settings:
   - **Subscription:** Select your Azure subscription
   - **Resource group name:** `rg-documents`
   - **Storage account name:** `aistr<yourname>` (must be globally unique, lowercase letters and numbers only)
   - **Storage account name:** `Australia East`
   - **Performance:** Standard
   - **Redundancy:** Locally-redundant storafe (LRS)

5. Leave remaining options as default
6. Select **Review + Create**
7. Select **Create.** Wait for deployment to complete, then select **Go to resource.**
8. In the left navigation bar, go to **Data storage > Containers**
9. Click **Add Container**
   - Enter Name: `callrecordings` 
   - Click **Create**
10. Select the **callrecordings** container
11. Select **Upload,** then select **browse for files**
12. Upload the Call Recordings from your local drive.
   > Note: You can download the recordings from the callrecordings folder of this repository
13. Then select **Upload.** Wait for all the recordings to be uploaded
14. Select the first recording and copy the URL into your Notepad
15. Do the rest for the remaining call recordings. We will need those URLs later in the lab

### 1.3 Access Content Understanding Studio
1. Navigate to [Content Understanding Studio](https://contentunderstanding.ai.azure.com/)
2. Sign in with your Azure account credentials
3. Select **Get started with Content Understanding**

### 1.4 Select Your Resource
1. Click on the settings icon (cog icon) near your profile icon in the top right
2. Select **Add resource**
3. Configure your resource:
   - **Subscription:** Select your subscription
   - **Resource group name:** Select `rg-documents`
   - **Resource name:** Select `ai-content-<yourname>`
   - Enable auto-deployment for required models if no default deployment available
   - Click **Next**
4. Click **Save** to confirm your resource selection
5. Verify that models are deployed successfully for the selected resource

### 1.5 Create a New Content Understanding Project
1. Select **Build** in the navigation bar
2. Click **+ Create**
3. Configure the project:
   - **Project name:** `call-center-analysis`
   - **Description:** `Analyze customer service call recordings to extract insights`
   - **Project Type:** Select **Extract content and field with custom schema**
4. Configure Advanced settings:
   - **Connected resource:** Select `ai-content-<yourname>`
   - **Subscription:** Select your subscription
   - **Resource Group:** Select `rg-documents`
   - **Storage Account:** Select `aistr<yourname>`
   - **Blob Container:** Select `call-recordings`
   - **Model for analysis:** Select the default model
5. Click **Create**

### 1.6 Upload Call Recordings
1. In your new project, click **Browse for files**
2. Upload one of the call recordings from your local drive
3. Select Audio Analysis
4. Click **Save**

### 1.7 Define Schema for Extraction
You'll now define what information to extract from the call recordings.

1. In your project, navigate to **Schema**
2. Click **+ Add new field** to create each extraction field
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
         - Select + **Add category:** then enter the following names and description;
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
          - Select **Add category:** then enter the following names and description;
               - Category Name: Positive 
               - Category Description: The customer expresses positive sentiment during the call
               - Category Name: Neutral
               - Category Description: The customer expresses neutral sentiment during the call
               - Category Name: Negative
               - Category Description: The customer expresses negative sentiment during the call
          - Click **Save**

4. Click **Save**

### 1.8 Process Audio Files and Review Results
1. Click **Run analysis** at the upper left
2. Wait for the processing to complete. This may take several minutes depending on the length of the audio and the complexity of the analysis
3. Once processing is complete, you can review the extracted information for the call recording in the **Fields** tab
4. Review each field's value
5. Select the **Result** tab to see the structured JSON output of the analysis, which includes all extracted fields and their values
---

### 1.9 Analyze Remaining Recordings
1. Select **Browse for files** again to upload the next call recording from your local machine
2. Click **Run analysis** 
3. Review extracted information for each call and the JSON output.

### 1.10 Build the Analyzer
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
backed by a Fabric capacity (F SKU) to enable Fabric-specific features

> **Design note:** Fabric capacities are billed based on compute usage (CU seconds),
> not storage. OneLake storage is billed separately per GB stored. This lab uses an
> F2 capacity, which provides a good balance of performance and cost for learning
> and development scenarios.

### 2.1 Deploy a Fabric Capacity (F2) in Azure Portal

Before creating a Fabric workspace, you need to provision a Fabric capacity in Azure
This capacity provides the compute resources for all Fabric workloads

1. Sign in to the [Azure Portal](https://portal.azure.com)

2. In the search bar at the top, type **Microsoft Fabric** and select **Microsoft Fabric** 
   from the results.

3. Select **+ Create** to create a new Fabric capacity

4. On the **Basics** tab, configure the following settings:

   | Setting | Value |
   | --- | --- |
   | Subscription | Select your Azure subscription |
   | Resource group | `rg-documents` |
   | Capacity name | Enter `fabriccapacityworkshop` |
   | Region | Australia East |
   | Size | Select **F2** (2 capacity units) |

   > **Note:** F2 is the smallest Fabric capacity SKU and is suitable for development 
   > and learning scenarios. Production workloads typically require F4 or higher

5. Select **Review + create**.

6. Review the configuration and estimated cost. Fabric capacities are billed per hour 
   while running.

7. Select **Create** to deploy the capacity

8. Wait for the deployment to complete (typically 2-5 minutes). You can monitor progress 
   in the **Notifications** panel.

9. Once deployed, select **Go to resource** to view your Fabric capacity.

> **Cost Management Tip:** Fabric capacities can be paused when not in use to avoid 
> charges. To pause a capacity, navigate to the capacity resource in Azure Portal and 
> select **Pause** from the toolbar. Remember to resume it before starting the lab

### 2.2 Create a Fabric Workspace

1. Go to [Fabric portal](https://app.fabric.microsoft.com), select **Workspaces** from the left navigation pane.

2. Select **+ New workspace**.

3. Configure the workspace:

   | Setting | Value |
   | --- | --- |
   | Name | `FabricWorkspace-Workshop` |
   | Description | `Foundry Fabric Workshop` |
   | Workspace type | **Fabric** |
   | Details | Select your **F2** capacity from the dropdown |

4. Select **Apply** to create the workspace.

5. You are now in your workspace. The workspace is empty and ready for Fabric items

---

## Step 3: Create a Lakehouse and Understand OneLake Storage

A Lakehouse in Fabric combines the flexibility of a data lake (file storage, open formats)
with the query performance of a data warehouse (SQL analytics endpoint, automatic schema
discovery). Every Lakehouse is backed by OneLake, which stores data in Delta Lake format.

### 3.1 Create a Lakehouse

1. In your workspace, select **+ New item**.

2. Under **Store data**, select **Lakehouse**.

3. Name the Lakehouse `LakehouseWorkshop`, enable Lakehouse schemas and select **Create**.

4. After a few moments, your Lakehouse opens. You will see two main sections:
   - **Tables** — structured Delta tables with schema and indexing
   - **Files** — unstructured or semi-structured files (Parquet, CSV, JSON)

5. On the upper left, notice the **Notebook** and **SQL analytics endpoint** selections. The SQL
   analytics endpoint is automatically created and allows you to query tables using T-SQL
   without any ETL.

---
## Step 4: Create a Fabric Notebook

The Fabric notebook is used to execute all sentiment analysis logic, including calling Microsoft Foundry, parsing results, and saving outputs.

### 4.1 Create a Notebook

1. Go to your workspace, click **+ New item.** Under **Get data** select **Notebook**
2. Name your **Notebook**
   - Notebook name: AnalyzeCalls
   - Location: FabricWorkspace-Workshop
3. Click **Create**
4. In the **Data items** tab under Explorer, select **Add data items > From OneLake Catalog** 
5. Then select your Lakehouse
6. Then click **Add**

### 4.2 Notebook Implementation
1. Click **+ Code** and paste the following Python code:

Replace the endpoint, key, and file_path form the values you saved in your notepad
```
import requests
from datetime import datetime, timezone
from pyspark.sql import Row
import time
 
 
# Set variables
endpoint = "YOUR_API_ENDPOINT"
key = "YOUR_API_KEY"
analyzer_id = "callanalyzer"
file_path = "YOUR_FILE_PATH"
api_version = "2025-11-01"
utc_time = datetime.now(timezone.utc)
processed_rows = []
```
Click the play button to run the cell.

2. Click **+ Code** and paste the following Python code:
```
def analyze_file(analyzer_id, file_url):
    """
    Submit a file for analysis using Azure Content Understanding API.
   
    Args:
        analyzer_id (str): The ID of the analyzer to use (e.g., "callanalyzer")
        file_url (str): The URL or path to the file to be analyzed
       
    Returns:
        str: The Operation-Location URL for polling the analysis status
       
    Raises:
        Exception: If the API returns a non-success status code (not 200 or 202)
    """
    # Construct the API endpoint URL, ensuring no double slashes
    url = f"{endpoint.rstrip('/')}/contentunderstanding/analyzers/{analyzer_id}:analyze"
   
    # Set up authentication and content type headers
    headers = {
        "Ocp-Apim-Subscription-Key": key,
        "Content-Type": "application/json"
    }
   
    # Prepare the request payload with the file URL
    payload = {
        "inputs": [{"url": file_path}]
    }
 
    # Submit the analysis request with API version as query parameter
    response = requests.post(
        url,
        headers=headers,
        params={"api-version": api_version},
        json=payload
    )
 
    # Check if the request was successful (200 OK or 202 Accepted)
    if response.status_code not in (200, 202):
        raise Exception(response.text)
 
    # Return the operation location URL for status polling
    return response.headers["Operation-Location"]
 
 
def poll_status(operation_location):
    """
    Check the status of a content analysis operation.
   
    Args:
        operation_location (str): The Operation-Location URL returned from analyze_file
       
    Returns:
        dict: JSON response containing the operation status and results
       
    Raises:
        requests.exceptions.HTTPError: If the request fails
    """
    # Set up authentication header for the status check
    headers = {"Ocp-Apim-Subscription-Key": key}
   
    # Query the operation location endpoint
    response = requests.get(operation_location, headers=headers)
   
    # Raise an exception if the request failed
    response.raise_for_status()
   
    # Return the status information as a dictionary
    return response.json()
```
 > The code above defines two functions: `analyze_file` to submit a file for analysis and `poll_status` to check the status of the analysis operation. The `analyze_file` function sends a POST request to the Content Understanding API with the file URL and returns an operation location URL for polling. The `poll_status` function sends a GET request to the operation location URL to retrieve the current status and results of the analysis.

 Click the play button to run the cell.

3. Click **+ Code** and paste the following Python code:
```
# Submit the file for analysis and get the operation location URL for polling
operation_location = analyze_file(analyzer_id, file_path)
 
# Poll the analysis operation until it completes (succeeds or fails)
while True:
    # Check the current status of the analysis operation
    result = poll_status(operation_location)
    status = result.get("status")
 
    # If analysis completed successfully, extract and process the results
    if status == "Succeeded":
        # Navigate to the fields in the analysis result
        fields = result["result"]["contents"][0]["fields"]
 
        # Create a Row object with extracted field values
        # Use .get() with default empty dict to safely handle missing fields
        processed_rows.append(Row(
            Customername=fields.get("CustomerName", {}).get("valueString"),
            Agentname=fields.get("AgentName", {}).get("valueString"),
            Callsummary=fields.get("CallSummary", {}).get("valueString"),
            Callresolution=fields.get("CallResolution", {}).get("valueString"),
            Productname=fields.get("ProductName", {}).get("valueString"),
            Callsentiment=fields.get("CallSentiment", {}).get("valueString"),  # Note: Using CallSummary, may be a bug
            DateTime=utc_time,
            File=file_path
        ))
        # Exit the polling loop after successful processing
        break
 
    # If the analysis failed or was cancelled, raise an exception
    elif status in ["Failed", "Cancelled"]:
        raise Exception(f"Analyzer failed: {status}")
 
    # Wait 10 seconds before checking the status again to avoid overwhelming the API
    time.sleep(10)
```
> The code above submits the specified file for analysis and continuously polls the operation status until it completes. If the analysis succeeds, it extracts the relevant fields from the result and appends them as a Row object to the `processed_rows` list. If the analysis fails or is cancelled, it raises an exception. The polling interval is set to 10 seconds to balance timely updates with API rate limits.

Click the play button to run the cell. Wait for the cell to complete approximately 30-60 seconds

4. Click **+ Code** and paste the following Python code:
```
# Convert Row objects to dictionaries and replace None values with "no data"
# This ensures all fields have displayable values in the final output
processed_rows = [
    {k: (v if v is not None else "no data") for k, v in row.asDict().items()}
    for row in processed_rows
]
 
# Create a Spark DataFrame from the processed row dictionaries
dataframe = spark.createDataFrame(processed_rows)
 
# Display the DataFrame (typically used in Databricks/Jupyter notebooks)
display(dataframe)
 
```
> The code above processes the list of Row objects by converting them to dictionaries and replacing any None values with the string "no data". This ensures that all fields have displayable values when shown in the DataFrame. Then, it creates a Spark DataFrame from the list of processed row dictionaries and displays it. The `display` function is commonly used in notebook environments to render the DataFrame in a readable format.

Click the play button to run the cell.

5. Click **+ Code** and paste the following Python code:
```
dataframe.write.mode("append").saveAsTable("analyzed_calls")
```
> The code above takes the Spark DataFrame containing the processed analysis results and writes it to a table named "analyzed_calls" in the Lakehouse. The `mode("append")` option ensures that if the table already exists, the new data will be added to it rather than overwriting existing data. This allows for incremental updates to the "analyzed_calls" table as new call recordings are processed.

Click the play button to run the cell. Wait for the cell to complete approximately 10-20 seconds

6. Run the Notebook with the other call recordings to populate more data in the Lakehouse

### 4.3 Verify the output table in the Lakehouse

1. Go to **Workspaces** 
2. Select your **Lakehouse** 
3. Navigate to Tables then select the ellipsis (...) then click Refresh
4. Expand the Tables > dbo > analyzed_calls table
5.Verify:
   - Table exists
   - Columns are populated
   - Rows reflect analyzed calls

> **Notes:**
> 1. We manually ran the notebook to analyze the calls. In a production setting, a Fabric Pipeline 
> will automate the process whenever a new recording comes into the Azure Blob Storage Container.
> 2. The endpoint and key values are currently hardcoded in the Notebook. In a production setting,
> you will need to use Azure Key Vault to protect these values.
> We did not include the Pipeline and Key Vault in this lab in the interest of time.

## Step 5: Create a Data Agent

A Data Agent in Microsoft Fabric provides a managed way to interact with data sources using natural language and AI‑assisted querying.
In this workshop, the Data Agent is used to explore and validate data stored in the Lakehouse after sentiment analysis results are generated.

### 5.1 Create Data Agents in Microsoft Fabric
1. Select **Workspaces** from the left navigation pane.
2. Open your Workspace
3. Select **+ New item.**
4. Select **Data agent.** under Analyze and train data

### 5.2 Configure the Data Agent
1. The **Create data agent** screen opens. Configure the Data Agent with the following settings:
   - **Name:** calldataagent

2. Select **Create.**
The Data Agent is now created inside the workspace.

### 5.3 Connect the Data Agent to the Lakehouse
1. Under the Data tab inside the Explorer pane select **Add data > Data source**
2. Select your **Lakehouse**
3. Click **Add**
4. Expand the dbo > Tables folder and select the analyzed_calls table

The Data Agent now automatically has access to all Lakehouse tables that the current user has permission to read.

### 5.4 Set Up the Data Agent
1. In the Setup tab under the Explorer pane select **Agent instruction**
2. Enter the following under the Agent instructions:

   ```markdown
   You are an AI agent that analyzes customer call recordings.
   ```

3. Select **Data source instructions** 
4. Enter the following under **Data source description**:
   ```
   The analyze_call table contains the call recordings analysis
   ```
5. Enter the following under **Data source instructions:**
   ```
   Agentname contains the name of the agent taking the call.
   Customername contains the name of the customer calling.
   Callsummary contains a brief summary of what the call was about.
   Callsentiment identifies the overall sentiment of the call either positive, neutral, or negative.
   Callresolution determines if the agent was able to resolve the customer issues either resolves, unresolved, or escalated.
   Productname identifies any product names mentioned during the call.
   ```

### 5.5 Validate the Data Agent
1. In the Data Agent chat input box, enter a sample question such as:
   - `How many calls have we received so far?`
   - `How many calls have negative sentiment?`
   - `What is the nature of the calls received by Ava`
2. Submit the question for each question.

### 5.6 Improving Agent Performance with Example Queries
1. For each the replies you have received, look for the Expand response button.
2. Select the **Expand response** and go to the **Steps completed** tab.
3. Click the drop down arrow to see the SQL query used to answer your question.
4. Copy this SQL query along with the question you asked and save it in a Notepad.
5. Go back to the Setup tab under the Explorer pane and select **Example queries**
6. Click **Add example**
7. Copy and paste the respective Question and SQL query pairs.
8. Do this for every question you have asked.
9. Select Clear chat and ask the same questions again.

### 5.7 Publish your Data Agent
1. Click **Publish** in the menu bar
2. Under the description enter the following:
   ```
   This agent analyzes customer calls.
   
   ```
3. Select publish

## Step 6: Power BI Report from Sentiment Analysis Results

6.1 Create a Power BI Semantic Model
1. In **Microsoft Fabric,** open **FabricWorkspace‑Intermediate.**
2. Select **LakehouseIntermediate**
3. In the Lakehouse ribbon, select **SQL analytics endpoint.**
4. In the SQL analytics endpoint view, select **New semantic model.**

Configure the semantic model:
| Setting | Value |
   | --- | --- |
   | Name | `SentimentSemanticModel` |
   | Workspace | `FabricWorkspace-Intermediate` |
5. Select analyzed_calls
6. Select **Confirm**

### 6.1 Open the Semantic Model

1. Return to **FabricWorkspace-Intermediate**
2. Locate **SentimentSemanticModel**
3. Select the semantic model to open it
4. With SentimentSemanticModel open, select **Open** tab in data pane
5. The Power BI report designer opens in your browser

### 6.2 Build the Sentiment Analysis Report
Create a Sentiment Distribution Visual
1. In the **Data** pane, expand analyzed_calls
2. Drag **Callsentiment** to the canvas
3. Drag **Customername** (or any field) to **Values**
4. Change aggregation to **Count**
5. Select a **Column chart** or **Pie chart** visualization

Create a Time-Based Sentiment Trend 
1. Add a new visual
2. Drag **DateTime** to the **X-axis**
3. Drag **Callsentiment** to **Legend**
4. Drag **Customername** to **Values** (Count)
5. Select a **Line chart**
6. Select **Save**
7. Enter report name








