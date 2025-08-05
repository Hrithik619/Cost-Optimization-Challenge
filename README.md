Azure Serverless Cost Optimization for Billing Records

📘 Overview

This solution aims to reduce storage costs in an Azure-based serverless architecture by offloading infrequently accessed billing records (older than 3 months) from Cosmos DB to a cheaper storage tier (Azure Blob Storage), while preserving seamless access and maintaining API contracts.

🧩 Problem Summary

Storage: Azure Cosmos DB

Record Size: Up to 300 KB

Volume: 2M+ records

Traffic: Read-heavy

Access Pattern: Records older than 3 months are rarely accessed

🎯 Goals

Reduce Cosmos DB storage costs

Retain access to old records with latency within a few seconds

No changes to existing API contracts

No downtime or data loss

Serverless and simple to maintain

🏗️ Architecture Overview

<img width="1536" height="1024" alt="Image" src="https://github.com/user-attachments/assets/2da92897-42ce-44d9-823a-078f710dde45" />

⚙️ Implementation Strategy
1. Data Archival Process
Use a scheduled Azure Function (Timer Trigger) to move old records to Blob Storage.

Pseudocode:

def archive_old_records():
    cutoff_date = datetime.utcnow() - timedelta(days=90)
    query = f"SELECT * FROM c WHERE c.timestamp < '{cutoff_date.isoformat()}'"
    old_records = cosmos_db.query_items(query)

    for record in old_records:
        blob_name = f"{record['id']}.json"
        blob_storage.upload(blob_name, json.dumps(record))
        cosmos_db.delete_item(record['id'])

2. Read Interceptor Logic
Modify the read function to check Cosmos DB first, then fallback to Blob Storage.

Pseudocode:
def get_billing_record(record_id):
    try:
        return cosmos_db.read_item(record_id)
    except ItemNotFound:
        blob_name = f"{record_id}.json"
        return blob_storage.download(blob_name)

3. Blob Storage Configuration
Use Archive Tier for cost savings.

Enable Lifecycle Management to auto-transition blobs to Archive after upload.

4. API Compatibility
Wrap Cosmos DB and Blob logic inside Azure Functions.

Keep API contracts unchanged by abstracting storage logic.

⚙️ Folder Structure for GitHub




