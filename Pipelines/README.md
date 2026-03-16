# 🏭 Data Ingestion with Pipelines in Microsoft Fabric

A hands-on implementation of an **ETL pipeline** using Microsoft Fabric — covering data ingestion from an external HTTP source, transformation with Apache Spark, and loading into a Delta Lake table via a Lakehouse.

---

## 📋 Overview

This project demonstrates how to build a complete data ingestion solution in Microsoft Fabric by combining:

- **Data Pipelines** — to extract and copy raw data from an external source
- **Apache Spark Notebooks** — to transform and enrich the data
- **Lakehouse (OneLake)** — as the central analytical data store

The pipeline follows an **ELT (Extract → Load → Transform)** pattern:

```
HTTP Source (sales.csv)
        │
        ▼
  [ Delete old files ]   ← Cleans staging area before each run
        │
        ▼
  [ Copy Data ]          ← Ingests raw CSV into Lakehouse Files
        │
        ▼
  [ Spark Notebook ]     ← Transforms data and writes Delta table
        │
        ▼
  Lakehouse Delta Table (new_sales)
```

---

## 🗂️ Project Structure

```
Fabric Workspace/
├── Lakehouse/
│   ├── Files/
│   │   └── new_data/            # Staging folder for raw CSV ingestion
│   └── Tables/
│       └── new_sales            # Final Delta Lake table (output)
├── Pipelines/
│   └── Ingest Sales Data        # Main orchestration pipeline
└── Notebooks/
    └── Load Sales               # PySpark transformation notebook
```

---

## ⚙️ Prerequisites

- Access to a **Microsoft Fabric** tenant (Trial, Premium, or Fabric capacity)
- Permissions to create Workspaces, Lakehouses, Pipelines, and Notebooks
- A browser with internet access to reach `app.fabric.microsoft.com`

---

## 🚀 Step-by-Step Guide

### 1. Create a Workspace
1. Navigate to the [Microsoft Fabric home page](https://app.fabric.microsoft.com/home?experience=fabric-developer)
2. Select **Workspaces** from the left menu bar
3. Create a new workspace with **Fabric capacity** enabled (Trial, Premium, or Fabric)

---

### 2. Create a Lakehouse
1. From the workspace, select **Create → Lakehouse** (under Data Engineering)
2. Give it a unique name — make sure **Lakehouse schemas (Public Preview)** is **disabled**
3. In the Explorer pane, create a subfolder under **Files** named `new_data`

---

### 3. Create the Ingestion Pipeline

1. From the Lakehouse home page, select **Get data → New data pipeline**
2. Name it `Ingest Sales Data`
3. Use the **Copy Data assistant** and configure the source:

   | Setting | Value |
   |---|---|
   | Source type | HTTP |
   | URL | `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv` |
   | Authentication | Anonymous |
   | File format | DelimitedText (CSV) |
   | First row as header | ✅ Selected |

4. Set the destination:

   | Setting | Value |
   |---|---|
   | Root folder | Files |
   | Folder path | `new_data` |
   | File name | `sales.csv` |

5. Select **Save + Run** and wait for the pipeline to complete successfully

---

### 4. Create the Transformation Notebook

1. From the Lakehouse, select **Open notebook → New notebook**
2. In the first cell, add this and **toggle it as a Parameter cell**:

```python
table_name = "sales"
```

3. Add a second code cell with the transformation logic:

```python
from pyspark.sql.functions import *

# Read the new sales data
df = spark.read.format("csv").option("header","true").load("Files/new_data/*.csv")

# Add month and year columns
df = df.withColumn("Year", year(col("OrderDate"))).withColumn("Month", month(col("OrderDate")))

# Derive FirstName and LastName columns
df = df.withColumn("FirstName", split(col("CustomerName"), " ").getItem(0)) \
       .withColumn("LastName", split(col("CustomerName"), " ").getItem(1))

# Filter and reorder columns
df = df["SalesOrderNumber", "SalesOrderLineNumber", "OrderDate", "Year", "Month",
        "FirstName", "LastName", "EmailAddress", "Item", "Quantity", "UnitPrice", "TaxAmount"]

# Load the data into a table
df.write.format("delta").mode("append").saveAsTable(table_name)
```

4. Run all cells and verify the `sales` table is created
5. Rename the notebook to `Load Sales`

---

### 5. Modify the Pipeline (Full ETL)

Enhance the pipeline with three sequential activities:

| Activity | Purpose |
|---|---|
| **Delete Data** | Remove any existing `.csv` files from `new_data/` |
| **Copy Data** | Download fresh `sales.csv` from the HTTP source |
| **Notebook** | Run `Load Sales` notebook to transform and load data |

**Notebook activity parameter:**

| Name | Type | Value |
|---|---|---|
| `table_name` | String | `new_sales` |

This overrides the default table name, writing to a `new_sales` table.

6. Run the full pipeline and verify the **new_sales** table in your Lakehouse

---
## Screenshots
 
Replace each placeholder reference above with actual screenshots captured during your implementation. The recommended files are:

| File | Description |
|---|---|
| `screenshots/pipelines-copy_data.png` | First pipeline activity for copying data |
| `screenshots/pipelines-showing_file.png` | Pipeline designer after initial Copy Data run |
| `screenshots/notebook-load_modify_data` | Notebook showing parameter and transformation cells |
| `screenshots/pipelines-delete_copy_load_data.png` | Final pipeline with all three activities |
| `screenshots/pipelines-showing_table.png` | `new_sales` table preview in Lakehouse Explorer |
 
 
---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **Lakehouse** | Unified storage combining data lake flexibility with warehouse query performance |
| **OneLake** | Fabric's single logical data lake underpinning all Lakehouse storage |
| **Delta Lake** | Open-source table format enabling ACID transactions on the Lakehouse |
| **Parameter cell** | Notebook cell whose variables can be overridden by pipeline parameters |
| **ELT vs ETL** | ELT loads raw data first, then transforms in-place — Fabric supports both |

---

## 🧹 Clean Up

To avoid consuming capacity after completing the lab:

1. Go to your workspace
2. Select **Workspace settings → General**
3. Scroll down and select **Remove this workspace → Delete**

---

## 📚 Resources

- [Microsoft Fabric Documentation](https://learn.microsoft.com/fabric/)
- [Apache Spark in Microsoft Fabric](https://learn.microsoft.com/fabric/data-engineering/spark-overview)
- [Delta Lake Overview](https://delta.io/)
- [Source Lab (MS Learn)](https://microsoftlearning.github.io/mslearn-fabric/)

---

## 🪪 License

This project is based on the [Microsoft Learn Fabric lab](https://github.com/MicrosoftLearning/mslearn-fabric) content.  
© 2025 Microsoft — used for educational purposes.