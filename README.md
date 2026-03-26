# 🏗️ Microsoft Fabric — End-to-End Analytics Journey

![Microsoft Fabric](https://img.shields.io/badge/Microsoft%20Fabric-Analytics-0078D4?style=flat&logo=microsoft&logoColor=white)
![Labs](https://img.shields.io/badge/Labs-9-107C10?style=flat)
![Status](https://img.shields.io/badge/Status-In%20Progress-F2C811?style=flat)
![License](https://img.shields.io/badge/License-MIT-green?style=flat)

A structured, end-to-end documentation of my Microsoft Fabric learning journey — from building a first Lakehouse to advanced data warehousing, security, and streaming. Each lab is documented with architecture diagrams, narrative explanations, and real screenshots captured during implementation.

---

## 🗺️ The Learning Path

The labs follow the official **Microsoft Fabric curriculum** sequence, building on each other from foundational concepts to advanced engineering patterns.

```
01 — Lakehouse fundamentals
        │
        ▼
02 — Ingest data with Pipelines
        │
        ▼
03 — Dataflows Gen2 (no-code ETL)
        │
        ▼
04 — Analyze data with Apache Spark
        │
        ▼
05 — Advanced Delta Lake (streaming, time travel)
        │
        ▼
06 — Load data into a Warehouse with T-SQL
        │
        ▼
07 — Monitor a Data Warehouse
        │
        ▼
08 — Secure Data in a Warehouse
        │
        ▼
09 — Real-Time Intelligence
```

---

## 📚 Labs

| # | Folder | Topic | Key Technologies |
|---|---|---|---|
| 01 | [01-Microsoft_Fabric_Lakehouse](./01-Microsoft_Fabric_Lakehouse/README.md) | Create a Lakehouse, upload files, load Delta tables, SQL & visual queries | OneLake · Delta Lake · SQL Analytics Endpoint |
| 02 | [02-Pipelines](./02-Pipelines/README.md) | ELT pipeline with Copy Data, PySpark notebook, parameter cells | Data Factory · PySpark · Delta Lake |
| 03 | [03-DataFlows_Gen2](./03-DataFlows_Gen2/README.md) | No-code ETL with Power Query Online, Lakehouse destination, pipeline orchestration | Dataflows Gen2 · Power Query · Delta Lake |
| 04 | [04-Apache_Spark](./04-Apache_Spark/README.md) | DataFrames, schema definition, Parquet partitioning, Spark SQL, matplotlib, seaborn | PySpark · Spark SQL · matplotlib · seaborn |
| 05 | [05-Delta_Table_Logic](./05-Delta_Table_Logic/README.md) | Schema enforcement, managed vs external tables, time travel, structured streaming | Delta Lake · Spark Streaming · Checkpoint |
| 06 | [06-T-SQL_Data_Warehouse](./06-T-SQL_Data_Warehouse/README.md) | Star schema, stored procedures, cross-database views, analytical T-SQL queries | T-SQL · Star Schema · Stored Procedures |
| 07 | [07-Monitor_Data_Warehouse](./07-Monitor_Data_Warehouse/README.md) | DMVs, live query monitoring, Query Insights, historical performance analysis | DMVs · sys schema · queryinsights |
| 08 | [08-Secure_Data_Warehouse](./08-Secure_Data_Warehouse/README.md) | Dynamic Data Masking, Row-Level Security, Column-Level Security, GRANT/DENY | DDM · RLS · CLS · T-SQL DCL |
| 09 | [09-Real-Time_Intelligence](./09-Real-Time_Intelligence/README.md) | Eventstream, Eventhouse, KQL Queries, Real-Time Dashboard, Activator Alert | Eventstream · KQL · Eventhouse · Real-Time Dashboard · Activator |

---

## 🗂️ Repository Structure

| Folder | README |
|---|---|
| `01-Microsoft_Fabric_Lakehouse/` | [→ README](./01-Microsoft_Fabric_Lakehouse/README.md) |
| `02-Pipelines/` | [→ README](./02-Pipelines/README.md) |
| `03-DataFlows_Gen2/` | [→ README](./03-DataFlows_Gen2/README.md) |
| `04-Apache_Spark/` | [→ README](./04-Apache_Spark/README.md) |
| `05-Delta_Table_Logic/` | [→ README](./05-Delta_Table_Logic/README.md) |
| `06-T-SQL_Data_Warehouse/` | [→ README](./06-T-SQL_Data_Warehouse/README.md) |
| `07-Monitor_Data_Warehouse/` | [→ README](./07-Monitor_Data_Warehouse/README.md) |
| `08-Secure_Data_Warehouse/` | [→ README](./08-Secure_Data_Warehouse/README.md) |
| `09-Real-Time_Intelligence/` | [→ README](./09-Real-Time_Intelligence/README.md) |

---

## 🔄 Folder Rename Guide

To rename your existing folders to match the structure above, run these commands in your terminal from the repo root:

```bash
git mv Microsoft_Fabric_Lakehouse  01-Microsoft_Fabric_Lakehouse
git mv Pipelines                   02-Pipelines
git mv DataFlows_Gen2              03-DataFlows_Gen2
git mv Apache_Spark                04-Apache_Spark
git mv Delta_Table_Logic           05-Delta_Table_Logic
git mv T-SQL_Data_Warehouse        06-T-SQL_Data_Warehouse
git mv Monitor_Data_Warehouse      07-Monitor_Data_Warehouse
git mv Secure_Data_Warehouse       08-Secure_Data_Warehouse
git mv Real-Time_Intelligence      09-Real-Time_Intelligence

git commit -m "refactor: rename folders to MS curriculum order"
git push origin main
```

> ⚠️ `git mv` preserves the full commit history of each folder — do not use regular `mv` or the history will be lost.

---

## ⚙️ Prerequisites

- Access to a **Microsoft Fabric** tenant (Trial, Premium, or Fabric capacity)
- A browser with internet access to reach `app.fabric.microsoft.com`
- No local tooling or SDK installation required for most labs

---

## 🎓 Curriculum

This repository follows the official **Microsoft Fabric Data Engineering** learning path on Microsoft Learn.

- [Microsoft Fabric Documentation](https://learn.microsoft.com/fabric/)
- [Microsoft Learn — Fabric Labs](https://microsoftlearning.github.io/mslearn-fabric/)
- [Delta Lake Overview](https://delta.io/)
- [Apache Spark Documentation](https://spark.apache.org/docs/latest/)

---

## 🪪 License

Lab content is based on the [Microsoft Learn Fabric repository](https://github.com/MicrosoftLearning/mslearn-fabric).  
© 2025 Microsoft — used for educational purposes.
