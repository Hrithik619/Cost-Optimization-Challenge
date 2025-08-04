Core Idea
Move billing records older than 3 months from Azure Cosmos DB to Azure Blob Storage (Archive Tier) while maintaining seamless access via existing APIs. Use Azure Functions to intercept read requests for archived data and serve them from Blob Storage.
+---------------------+       +---------------------+
|   Client/API Layer  | <---> | Azure API Management|
+---------------------+       +---------------------+
                                      |
                                      v
                           +----------------------+
                           | Azure Function Proxy |
                           +----------------------+
                            /           \
                           /             \
         +----------------+               +----------------+
         | Cosmos DB (Hot)|               | Blob Storage   |
         |  < 3 months    |               |  > 3 months    |
         +----------------+               +----------------+

