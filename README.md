Core Idea
Move billing records older than 3 months from Azure Cosmos DB to Azure Blob Storage (Archive Tier) while maintaining seamless access via existing APIs. Use Azure Functions to intercept read requests for archived data and serve them from Blob Storage.
