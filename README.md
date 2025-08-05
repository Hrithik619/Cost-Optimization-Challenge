# Cost-Optimized Azure Serverless Architecture for Billing Records

## 🔧 Problem Statement
- **Storage**: Billing records in Azure Cosmos DB
- **Data size**: 2M+ records, up to 300KB each
- **Access pattern**: Read-heavy, but older records (3+ months) rarely accessed
- **Constraints**:
  - No downtime or data loss
  - No changes to existing API contracts
  - Easy to implement and maintain

## ✅ Solution Overview
Introduce a **Hot-Cold Data Architecture**:

- **Hot Data** (recent, <3 months): Remain in **Cosmos DB** for low-latency access.
- **Cold Data** (older, >3 months): Moved to **Azure Blob Storage (Hot tier)** as compressed JSON.
- **Access Layer**: Middleware (e.g., Azure Function/Logic App) checks Cosmos DB first, then Blob if not found.
- **Archival**: Azure Data Factory or Durable Functions perform scheduled archival.


## 📊 Architecture Diagram

<img width="1536" height="1024" alt="Image" src="https://github.com/user-attachments/assets/329857b4-3ef8-4440-bc2a-58ec5b352223" />

## 💾 Storage Tiering Strategy
| Data Type | Storage Location | Access Time | Cost            |
|-----------|------------------|-------------|-----------------|
| < 3 months | Cosmos DB        | <100ms      | High (hot)      |
| > 3 months | Blob Storage     | <2 seconds  | Low (compressed) |

---

## 🧠 Archival Pseudocode
```python
# Scheduled via Azure Data Factory or Timer Triggered Durable Function
from azure.cosmos import CosmosClient
from azure.storage.blob import BlobServiceClient
import json, gzip, datetime

COSMOS_CONN = "..."
BLOB_CONN = "..."

client = CosmosClient.from_connection_string(COSMOS_CONN)
database = client.get_database_client("billing")
container = database.get_container_client("records")

blob_service = BlobServiceClient.from_connection_string(BLOB_CONN)
container_client = blob_service.get_container_client("archived-billing")

cutoff_date = datetime.datetime.utcnow() - datetime.timedelta(days=90)

for item in container.query_items("SELECT * FROM c WHERE c.timestamp < @cutoff", parameters=[{"name": "@cutoff", "value": cutoff_date.isoformat()}], enable_cross_partition_query=True):
    blob_name = f"{item['id']}.json.gz"
    data = gzip.compress(json.dumps(item).encode())
    container_client.upload_blob(blob_name, data)
    container.delete_item(item, partition_key=item['partitionKey'])
```

---

## 🔍 Retrieval Logic Pseudocode (Azure Function)
```python
# HTTP Triggered Azure Function (Node.js / Python)
# Unified access for both hot and cold records

def get_billing_record(record_id):
    try:
        # First try Cosmos DB (hot)
        item = cosmos_container.read_item(record_id, partition_key=...)
        return item
    except Exception:
        # If not found, try Blob (cold)
        blob = blob_client.get_blob_client(f"{record_id}.json.gz")
        data = gzip.decompress(blob.download_blob().readall())
        return json.loads(data)
```

---

## 🧪 Benefits
- ~70-90% Cosmos DB cost reduction
- No API changes
- Few seconds access latency for cold data
- Blob Storage is durable, scalable, and cheap




