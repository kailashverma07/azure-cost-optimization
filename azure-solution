# Azure Cost Optimization for Billing Records

## Problem Statement

We have a serverless Azure architecture storing billing records in Cosmos DB. The system is read-heavy, but records older than 3 months are rarely accessed.

Over time, the Cosmos DB size has grown to over 2 million records (up to 300 KB each), causing high costs. We need a cost-optimized solution that preserves availability and API contracts, with no downtime or data loss.

---

## Proposed Solution Overview

- Keep **recent records (≤ 90 days)** in Azure Cosmos DB (hot storage) for fast access.
- Archive **older records (> 90 days)** in **Azure Blob Storage** using cool/archive tiers to reduce costs.
- Implement an **Azure Function middleware** to route read/write requests:
  - Writes always go to Cosmos DB.
  - Reads first try Cosmos DB; if record is not found and is older than 90 days, fallback to Blob Storage.
- This approach maintains existing APIs without changes.
- No downtime or data loss during the migration.
- Simple and maintainable.

---

## Architecture Diagram

![Architecture Diagram](./architecture.svg)

---

## Summary: Gradual Cleanup of Cosmos DB Records Older Than 90 Days

| Feature / Constraint     | Met? | Solution                                                   |
|-------------------------|-------|------------------------------------------------------------|
| Simplicity               | Yes   | Serverless Azure Function, native Azure storage solution  |
| No Data Loss / Downtime  | Yes   | Gradual migration with zero user impact                    |
| API Contracts Unchanged  | Yes   | Access logic hidden inside middleware; API remains same    |
| Cost Optimization        | Yes   | Blob Storage is 90%+ cheaper than Cosmos DB                |
| Access Latency (Old Data)| Yes   | Blob retrieval within seconds                               |
| Maintenance Overhead     | Yes   | Low — uses native Azure services                            |
| Security / Compliance    | Yes   | Blob versioning, managed identity, RBAC, logging           |

---

## 💰 Cost Optimization Benefits

| Service          | Optimization Strategy                      | Savings Type       |
|------------------|-------------------------------------------|--------------------|
| Cosmos DB        | Reduce size with TTL + archive             | Throughput & RU/s  |
| Blob Storage     | Archive as JSON with Cool/Archive Tier     | Cheap per GB       |
| Azure Functions  | Serverless — runs only on access            | Minimal Cost       |

---

## 🧠 Why This Works

- **No downtime:** Seamless migration using background tasks.  
- **No data loss:** Every record is moved, not deleted.  
- **No API changes:** Read logic handles tiered lookups.  
- **Simplicity:** Serverless + native Azure services only.  

---

## 📝 Optional Enhancements

| Feature             | Description                                  |
|---------------------|----------------------------------------------|
| Index Blob metadata  | For faster lookups (filename = record ID)   |
| Use ADLS Gen2       | For hierarchical file structure, advanced logging |
| Cache old records   | Frequently accessed older data can be cached |

---

## ✅ Final Summary

| Requirement      | Solution                                         |
|------------------|-------------------------------------------------|
| Simplicity       | Native Azure services only                        |
| No Downtime      | Background archival + fallback reads             |
| No Data Loss     | Full export to Blob before TTL deletion          |
| No API Changes   | Abstracted in backend logic                       |
| Cost Reduction   | Up to 70%+ storage savings on cold data          |

---

## Implementation Details

### Data Archival Process (scheduled Azure Function)

- Query Cosmos DB for records older than 90 days.
- Serialize and upload these records as blobs to Azure Blob Storage (cool/archive tier).
- Mark or delete archived records from Cosmos DB.

### Read Flow (Azure Function Middleware)

```python
def get_billing_record(record_id):
    record = cosmosdb_read(record_id)
    if record:
        return record
    elif is_older_than_90_days(record_id):
        return blob_storage_read(record_id)
    else:
        return None
Write Flow (Azure Function Middleware)
python
Copy
Edit
def write_billing_record(record):
    cosmosdb_write(record)
