# 📊 Dataflows Gen2 in Microsoft Fabric

A hands-on implementation of **Dataflows Gen2** in Microsoft Fabric — connecting to an external CSV source, applying no-code transformations in Power Query Online, and loading the results into a Lakehouse Delta table via a Data Pipeline.

---

## 🏗️ Architecture

Before diving into the steps, here is the big picture of what we are building and how all the components connect to each other.

> ![Architecture Diagram](/screenshots/gen2-architecture.png)
> *End-to-end architecture: from the HTTP data source through Power Query transformations, into the Lakehouse via a pipeline.*

### How it all fits together

This lab is built around three core Fabric components that work in sequence:

**1 — Dataflow Gen2 (Power Query Online)**
This is where the actual data work happens. The dataflow connects to an external CSV file over HTTP, applies transformations visually without writing any code, and defines where the output should land. Think of it as the brain of the ETL process.

**2 — Lakehouse (OneLake)**
The destination for our data. The Lakehouse stores everything in Delta format on top of OneLake, giving us ACID transactions, time travel, and the ability to query data with SQL or Spark later. We point the dataflow directly at a Lakehouse table so the data lands there automatically.

**3 — Data Pipeline**
The dataflow on its own can be run manually, but wrapping it inside a pipeline makes it schedulable, monitorable, and composable with other activities. This is the orchestration layer — it is what makes the whole process production-ready.

```
HTTP Source (orders.csv)
        │
        ▼
  [ Power Query Online ]   ← Transform: add MonthNo column, enforce data types
        │
        ▼
  [ Pipeline: Dataflow Activity ]   ← Orchestrate and schedule the dataflow
        │
        ▼
  Lakehouse Delta Table (orders)   ← Query-ready, ACID-compliant storage
```

---

## ⚙️ Prerequisites

- Access to a **Microsoft Fabric** tenant (Trial, Premium, or Fabric capacity)
- Permissions to create Workspaces, Lakehouses, Pipelines, and Dataflows
- A browser with internet access to reach `app.fabric.microsoft.com`

No local tooling or SDK installation is required. All steps are performed within the Fabric web interface.

---

## 🗂️ What Gets Created

By the end of this lab, your Fabric workspace will contain:

```
Fabric Workspace/
├── Lakehouse/
│   └── Tables/
│       └── orders               # Final Delta table loaded by the Dataflow
├── Pipelines/
│   └── Load Data                # Pipeline that runs the Dataflow on demand
└── Dataflows/
    └── Dataflow 1               # Power Query Online ETL definition
```

---

## 🚀 Step-by-Step Guide

### 1. Create a Workspace

**What:** A Workspace is the top-level container in Microsoft Fabric. Every resource — lakehouses, pipelines, dataflows — lives inside one.

**Why:** Before we can create any Fabric resources, we need a workspace with the right capacity assigned. Without Fabric capacity, features like Dataflows Gen2 and Lakehouses are not available.

1. Navigate to the [Microsoft Fabric portal](https://app.fabric.microsoft.com/home?experience=fabric) and sign in.
2. Select **Workspaces** from the left menu bar.
3. Select **New workspace**, provide a name, and choose a licensing mode that includes Fabric capacity (Trial, Premium, or Fabric).
4. Confirm creation — the workspace should open empty.

---

### 2. Create a Lakehouse

**What:** A Lakehouse is a unified analytical store that combines the scalability of a data lake with the query performance of a data warehouse. Data is stored in Delta format on top of OneLake.

**Why:** We need a destination for our transformed data. Rather than writing to a raw file, we write directly to a structured Delta table inside the Lakehouse — this makes the data immediately queryable with SQL or Spark, and gives us features like ACID transactions and schema enforcement for free.

1. From your workspace, select **Create → Lakehouse** (under Data Engineering).
2. Give it a unique name and ensure **Lakehouse schemas (Public Preview)** is **disabled**.
3. Wait for the empty Lakehouse to be created.

---

### 3. Create a Dataflow Gen2

**What:** A Dataflow Gen2 is a Power Query-based ETL component. You connect it to a data source, apply transformations using a visual editor, and point it at a destination — all without writing code.

**Why:** Instead of writing PySpark or SQL, Dataflows let us use the familiar Power Query interface to clean and enrich data. This is especially useful for analysts or engineers who want fast, repeatable transformations without managing a Spark cluster. The `MonthNo` column we add here is a good example — extracting the month number from a date field is a common business requirement that would otherwise require code.

1. From the Lakehouse home page, select **Get data → New Dataflow Gen2**.
2. The **Power Query Online** editor will open.
3. Select **Import from a Text/CSV file** and configure the connection:

   | Setting | Value |
   |---|---|
   | File path or URL | `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/orders.csv` |
   | Connection | Create new connection |
   | Data gateway | (none) |
   | Authentication kind | Anonymous |

4. Select **Next** to preview the data, then select **Create**.
5. On the **Add column** tab, select **Custom column** and add a derived month field:

   | Setting | Value |
   |---|---|
   | New column name | `MonthNo` |
   | Data type | Whole Number |
   | Formula | `Date.Month([OrderDate])` |

6. Select **OK** and confirm the column appears in the data preview.
7. Verify the following data types are correctly assigned — incorrect types here will cause errors downstream:

   | Column | Required Type |
   |---|---|
   | `OrderDate` | Date |
   | `MonthNo` | Whole Number |

> **Screenshot placeholder**
> ![Power Query Editor](/screenshots/gen2-power-query-custom-column.png)
> *Power Query Online showing the MonthNo custom column formula and the Applied Steps panel on the right, which records every transformation as a replayable step.*

---

### 4. Configure the Data Destination

**What:** This step tells the Dataflow where to write its output — in our case, a new table inside the Lakehouse we created earlier.

**Why:** Dataflows are not useful unless their output lands somewhere structured and queryable. By pointing the destination at a Lakehouse table with **Append** mode, we ensure that every time the dataflow runs, new records are added without overwriting existing ones — which is the correct pattern for incremental data ingestion.

1. On the **Home** tab of the Power Query editor, select **Add data destination → Lakehouse**.
2. Sign in with your Power BI organisational account when prompted, then select **Next**.
3. Find your workspace, select your Lakehouse, and specify a new table named `orders`.
4. Select **Next**, then on the **Choose destination settings** page:
   - Disable **Use automatic settings**
   - Set the update method to **Append**
   - Select **Save settings**
5. Open **View → Diagram view** to visually confirm the Lakehouse destination is connected to the query.
6. Select **Save & run** and wait for the dataflow to finish executing.

> **Screenshot placeholder**
> ![Dataflow Diagram View](/screenshots/gen2-architecture.png)
> *Diagram view in Power Query Online — the Lakehouse icon on the right confirms the destination is correctly wired up.*

---

### 5. Add the Dataflow to a Pipeline

**What:** A Data Pipeline wraps the Dataflow inside an orchestration layer, giving us scheduling, monitoring, retry logic, and the ability to chain it with other activities.

**Why:** A standalone Dataflow can only be triggered manually or on a simple schedule. By embedding it in a pipeline, we gain full control over the execution context — we can trigger it on a schedule, chain it after other steps, monitor run history, and handle failures gracefully. This is what separates a proof-of-concept from a production-grade data flow.

1. From your workspace, select **+ New item → Data pipeline** and name it `Load Data`.
2. If the Copy Data wizard opens automatically, close it.
3. On the **Activities** tab, select **Pipeline activity → Dataflow** to add a Dataflow activity to the canvas.
4. With the Dataflow activity selected, open the **Settings** tab and configure:

   | Property | Value |
   |---|---|
   | Dataflow | `Dataflow 1` |

5. Save the pipeline using the **🖫** icon.
6. Select **▷ Run** and wait for the pipeline to complete. This may take a few minutes.
7. Navigate to your Lakehouse, open the **...** menu on **Tables**, select **Refresh**, and confirm the `orders` table has been created and populated.

> **Screenshot placeholder**
> ![Dataflow Pipeline Completed](/screenshots/gen2-dataflow-pipeline-completed.png)
> *The pipeline run completed successfully — the green checkmark on the Dataflow activity confirms the data was written to the Lakehouse.*

> **Screenshot placeholder**
> ![Orders Table Preview](/screenshots/gen2-orders-table-preview.png)
> *The `orders` Delta table in the Lakehouse Explorer, populated with the transformed data including the new MonthNo column.*

---

## 🔄 Transformations Applied

| Transformation | Input | Output | Why |
|---|---|---|---|
| Import CSV | HTTP URL | Raw table in Power Query | Connects to the source data |
| Add `MonthNo` column | `OrderDate` | `MonthNo` (Whole Number) | Enables month-based filtering and aggregation |
| Enforce data types | `OrderDate`, `MonthNo` | Date, Whole Number | Prevents type mismatch errors on write |
| Write to Lakehouse | Transformed table | `orders` Delta table (Append) | Persists data for downstream querying |

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **Dataflow Gen2** | A Fabric component using Power Query Online to define ETL logic without writing code |
| **Power Query Online** | Browser-based transformation tool with a visual, step-by-step editor and 300+ connectors |
| **Applied Steps** | The ordered, replayable list of transformations recorded automatically in Power Query |
| **Lakehouse** | A unified store combining data lake flexibility with warehouse query performance |
| **Delta Lake** | Open-source table format enabling ACID transactions, versioning, and schema enforcement |
| **Append mode** | Write strategy that adds new records without overwriting existing data |
| **Data Pipeline** | Fabric's orchestration layer for scheduling, chaining, and monitoring data activities |

---

## 📸 Screenshots

| File | Description |
|---|---|
| `screenshots/architecture.png` | Your architecture diagram showing the end-to-end flow |
| `screenshots/gen2-power-query-custom-column.png` | Power Query editor with MonthNo column and Applied Steps |
| `screenshots/gen2-dataflow-diagram-view.png` | Diagram view with Lakehouse destination linked |
| `screenshots/gen2-dataflow-pipeline-completed.png` | Completed Dataflow pipeline run |
| `screenshots/gen2-orders-table-preview.png` | `orders` table preview in the Lakehouse |

---

## 🧹 Clean Up

To remove all resources after completing the lab:

1. In the left navigation bar, select the icon for your workspace.
2. Select **Workspace settings → General**.
3. Scroll down and select **Remove this workspace → Delete**.

---

## 📚 Resources

- [Microsoft Fabric Documentation](https://learn.microsoft.com/fabric/)
- [Dataflows Gen2 in Microsoft Fabric](https://learn.microsoft.com/fabric/data-factory/dataflows-gen2-overview)
- [Power Query Documentation](https://learn.microsoft.com/power-query/)
- [Delta Lake Overview](https://delta.io/)
- [Source Lab (MS Learn)](https://microsoftlearning.github.io/mslearn-fabric/)

---

## 🪪 License

This project is based on lab content from the [Microsoft Learn Fabric repository](https://github.com/MicrosoftLearning/mslearn-fabric).  
© 2025 Microsoft — used for educational purposes.
