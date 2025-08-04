💡 Proposed Solution: Tiered Storage with Azure Functions & Blob Archive

🧠 Core Idea
Move billing records older than 3 months from Azure Cosmos DB to Azure Blob Storage (Archive Tier) while maintaining seamless access via existing APIs. Use Azure Functions to intercept read requests for archived data and serve them from Blob Storage.

🏗️ Architecture Overview

<img width="734" height="377" alt="image" src="https://github.com/user-attachments/assets/2a964f77-1f15-4e17-9bdf-10fc13390edd" />

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

🧪 Edge Cases & Reliability

<img width="862" height="250" alt="image" src="https://github.com/user-attachments/assets/8825b29c-ae1b-46d4-8516-a464d3b73d7a" />


🛠️ Deployment Commands

Cosmos DB Query:

SELECT * FROM c WHERE c.timestamp < "2023-05-01T00:00:00Z"


Azure CLI for Blob Upload:

az storage blob upload \
  --account-name <storage_account> \
  --container-name billing-archive \
  --name <record_id>.json \
  --file <local_file_path> \
  --tier Archive





