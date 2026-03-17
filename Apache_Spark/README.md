# 🔥 Analyze Data with Apache Spark in Microsoft Fabric

A hands-on implementation of **data analysis with Apache Spark** in Microsoft Fabric — ingesting CSV files into a Lakehouse, transforming and exploring data using PySpark DataFrames, querying with Spark SQL, and visualizing results with matplotlib and seaborn.

---

## 🏗️ Architecture

Before diving into the steps, here is the big picture of what we are building and how all the components connect to each other.

> ![Architecture Diagram](./screenshots/apache-architecture.png)
> *End-to-end architecture: CSV files uploaded to the Lakehouse, read into Spark DataFrames, transformed and saved as Parquet/Delta, then queried and visualized via a Fabric Notebook.*

### How it all fits together

This lab is built around four core components that work in sequence:

**1 — Lakehouse (OneLake)**
The central storage layer. We upload raw CSV files here and later write transformed data back as Parquet and Delta tables. Everything lives in OneLake, making it accessible to Spark, SQL, and Power BI without moving data.

**2 — Fabric Notebook (PySpark)**
The interactive workspace where all the data work happens. We use PySpark — Python optimized for Spark — to read files, define schemas, apply transformations, and run queries. The notebook is attached to the Lakehouse so it can read and write data directly.

**3 — Spark SQL & Delta Tables**
Once data is transformed, we register it as a managed Delta table. This gives us the ability to query it with standard SQL using the `%%sql` magic in the notebook — bridging the gap between a data lake and a relational warehouse.

**4 — Visualizations (matplotlib & seaborn)**
The final step is turning query results into charts. We use matplotlib for fine-grained control and seaborn for a cleaner, theme-based approach — both running directly inside the notebook.

```
Local CSV Files (orders.zip)
        │
        ▼
  [ Lakehouse Files ]       ← Upload 2019.csv, 2020.csv, 2021.csv
        │
        ▼
  [ PySpark Notebook ]      ← Read, explore, transform DataFrames
        │         │
        ▼         ▼
  [ Parquet /  [ Delta Table ]   ← Save transformed & partitioned data
  Partitioned ]  (salesorders)
                  │
                  ▼
          [ Spark SQL Queries ]   ← Query with %%sql magic
                  │
                  ▼
        [ matplotlib / seaborn ]  ← Visualize results as charts
```

---

## ⚙️ Prerequisites

- Access to a **Microsoft Fabric** tenant (Trial, Premium, or Fabric capacity)
- Permissions to create Workspaces, Lakehouses, and Notebooks
- A browser with internet access to reach `app.fabric.microsoft.com`

No local tooling or SDK installation is required. All steps are performed within the Fabric web interface.

---

## 🗂️ What Gets Created

By the end of this lab, your Fabric workspace will contain:

```
Fabric Workspace/
├── Lakehouse/
│   ├── Files/
│   │   ├── orders/                  # Uploaded raw CSV files (2019–2021)
│   │   ├── transformed_data/orders/ # Parquet output of transformed DataFrame
│   │   └── partitioned_data/        # Data partitioned by Year and Month
│   └── Tables/
│       └── salesorders              # Managed Delta table for SQL queries
└── Notebooks/
    └── (your notebook name)         # PySpark analysis notebook
```

---

## 🚀 Step-by-Step Guide

### 1. Create a Workspace

**What:** A Workspace is the top-level container in Microsoft Fabric. Every resource — lakehouses, notebooks, pipelines — lives inside one.

**Why:** Fabric features like Spark notebooks and Lakehouses require a workspace with Fabric capacity assigned. Without it, these capabilities are not available.

1. Navigate to the [Microsoft Fabric portal](https://app.fabric.microsoft.com/home?experience=fabric-developer) and sign in.
2. Select **Workspaces** from the left menu bar.
3. Select **New workspace**, provide a name, and choose a licensing mode that includes Fabric capacity (Trial, Premium, or Fabric).
4. Confirm creation — the workspace should open empty.

---

### 2. Create a Lakehouse and Upload Files

**What:** A Lakehouse is the unified storage layer of our solution. We upload raw CSV order files here so Spark can read them directly.

**Why:** Rather than connecting to an external database or HTTP source, we place the files inside the Lakehouse's `Files` section. This means Spark can access them via a simple file path (`Files/orders/*.csv`) without any additional connectors or credentials — keeping the setup simple and the data co-located with our compute.

1. From your workspace, select **New Item → Lakehouse** (under Store data).
2. Give it a unique name and ensure **Lakehouse schemas (Public Preview)** is **disabled**.

   > ⚠️ This setting cannot be changed after creation. If you miss this step, you will need to create a new Lakehouse.

3. Download the data files from: `https://github.com/MicrosoftLearning/dp-data/raw/main/orders.zip`
4. Extract the archive — you should have a folder named `orders` containing three files: `2019.csv`, `2020.csv`, and `2021.csv`.
5. In the Lakehouse Explorer pane, select the **...** menu next to **Files** and choose **Upload → Upload folder**.
6. Navigate to the extracted `orders` folder and upload it.
7. Verify the files appear under `Files/orders/`.

> **Screenshot placeholder**
> ![CSV Files Uploaded](./screenshots/apache-csv-files-uploaded.png)
> *The orders folder containing 2019.csv, 2020.csv and 2021.csv visible in the Lakehouse Explorer.*

---

### 3. Create a Notebook

**What:** A Fabric Notebook is an interactive environment where we write and run PySpark code, add markdown explanations, and view results inline.

**Why:** Notebooks are the standard tool for exploratory data analysis in Spark environments. They allow us to iterate quickly — running one cell at a time, inspecting results, and refining our code — without needing to deploy a full application.

1. From the left menu bar, select **... → Create → Notebook** (under Data Engineering).
2. Click the notebook name above the Home tab and rename it to something descriptive (e.g., `Sales Order Analysis`).
3. Select the first cell, click the **M↓** button in the top-right toolbar to convert it to a **markdown cell**, and add a title:

```markdown
# Sales order data exploration
Use this notebook to explore sales order data
```

4. Click outside the cell to stop editing.

> **Screenshot placeholder**
> ![New Notebook](./screenshots/apache-new-notebook.png)
> *A new Fabric notebook with a markdown title cell.*

---

### 4. Create a DataFrame

**What:** A DataFrame is Spark's core data structure — a distributed table of rows and columns. We use it to load the CSV files and work with the data programmatically.

**Why:** Loading data into a DataFrame gives us access to Spark's entire transformation and analysis API. We define a schema explicitly rather than relying on auto-detection — this ensures correct data types (especially for dates and numbers) and avoids silent type errors that would cause problems in downstream steps.

1. From the Lakehouse Explorer, attach your Lakehouse to the notebook: select **Open notebook → Existing notebook** and open your notebook. Expand **Files → orders** in the Explorer pane.
2. From the **...** menu for `2019.csv`, select **Load data → Spark** to auto-generate a code cell.
3. Replace the generated code with the following to define a proper schema and load all three years at once:

```python
from pyspark.sql.types import *

orderSchema = StructType([
    StructField("SalesOrderNumber", StringType()),
    StructField("SalesOrderLineNumber", IntegerType()),
    StructField("OrderDate", DateType()),
    StructField("CustomerName", StringType()),
    StructField("Email", StringType()),
    StructField("Item", StringType()),
    StructField("Quantity", IntegerType()),
    StructField("UnitPrice", FloatType()),
    StructField("Tax", FloatType())
])

df = spark.read.format("csv").schema(orderSchema).load("Files/orders/*.csv")

display(df)
```

4. Run the cell with **▷ Run cell**. The first run may take a minute as the Spark session starts.
5. Verify the output shows data with correctly typed columns across all three years.

> **Screenshot placeholder**
> ![DataFrame with Schema](./screenshots/apache-dataframe-schema.png)
> *DataFrame displaying order data with the defined schema — note the correct data types on each column.*

---

### 5. Explore the Data

**What:** This step uses DataFrame methods to filter, count, group, and aggregate data — answering basic business questions without writing SQL.

**Why:** Before transforming or saving data, it is good practice to understand its shape and content. DataFrame operations like `select`, `where`, `groupBy`, and `count` let us quickly validate data quality, spot anomalies, and confirm our schema is correct — all within the same notebook.

Add the following code cells one by one, running each to observe the results:

**Filter customers who bought a specific product:**
```python
customers = df.select("CustomerName", "Email").where(df['Item'] == 'Road-250 Red, 52')
print(customers.count())
print(customers.distinct().count())
display(customers.distinct())
```

**Total quantity sold per product:**
```python
productSales = df.select("Item", "Quantity").groupBy("Item").sum()
display(productSales)
```

**Number of orders per year:**
```python
from pyspark.sql.functions import *

yearlySales = df.select(year(col("OrderDate")).alias("Year")).groupBy("Year").count().orderBy("Year")
display(yearlySales)
```

> **Screenshot placeholder**
> ![Data Exploration Results](./screenshots/apache-data-exploration.png)
> *Yearly order counts showing aggregated results grouped by year.*

---

### 6. Transform and Save the Data

**What:** We apply a series of transformations to enrich the DataFrame — adding derived columns and reordering fields — then save the result in Parquet format, both as a flat file and as partitioned data.

**Why:** Raw CSV files are not ideal for large-scale analytics. Parquet is a columnar format that compresses data efficiently and enables predicate pushdown — meaning Spark only reads the columns and partitions it actually needs, dramatically improving query performance. Partitioning by Year and Month means that a query filtering for a single month only reads that month's file, not the entire dataset.

**Apply transformations:**
```python
from pyspark.sql.functions import *

# Add Year, Month, FirstName, LastName columns
transformed_df = df.withColumn("Year", year(col("OrderDate"))) \
                   .withColumn("Month", month(col("OrderDate"))) \
                   .withColumn("FirstName", split(col("CustomerName"), " ").getItem(0)) \
                   .withColumn("LastName", split(col("CustomerName"), " ").getItem(1))

# Select and reorder final columns
transformed_df = transformed_df[
    "SalesOrderNumber", "SalesOrderLineNumber", "OrderDate", "Year", "Month",
    "FirstName", "LastName", "Email", "Item", "Quantity", "UnitPrice", "Tax"
]

display(transformed_df.limit(5))
```

**Save as Parquet:**
```python
transformed_df.write.mode("overwrite").parquet('Files/transformed_data/orders')
print("Transformed data saved!")
```

**Save as partitioned Parquet (by Year and Month):**
```python
transformed_df.write.partitionBy("Year", "Month").mode("overwrite").parquet("Files/partitioned_data")
print("Partitioned data saved!")
```

Refresh the Explorer pane to verify the folder structure has been created under `Files/`.

> **Screenshot placeholder**
> ![Partitioned Data](./screenshots/apache-partitioned-data.png)
> *Explorer pane showing the partitioned folder hierarchy: Year=2021/Month=1, etc.*

---

### 7. Work with Tables and SQL

**What:** We register the DataFrame as a managed Delta table in Spark's metastore, then query it using standard SQL with the `%%sql` notebook magic.

**Why:** Not everyone is comfortable writing PySpark. Registering data as a Delta table means analysts can query it with familiar SQL syntax, BI tools like Power BI can connect to it directly, and we get all the benefits of Delta format — ACID transactions, schema enforcement, and time travel — for free.

**Create a Delta table:**
```python
df.write.format("delta").saveAsTable("salesorders")
spark.sql("DESCRIBE EXTENDED salesorders").show(truncate=False)
```

**Query with Spark SQL:**
```sql
%%sql
SELECT YEAR(OrderDate) AS OrderYear,
       SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue
FROM salesorders
GROUP BY YEAR(OrderDate)
ORDER BY OrderYear;
```

Refresh the **Tables** section in the Explorer pane to confirm the `salesorders` table is visible.

> **Screenshot placeholder**
> ![Delta Table Created](./screenshots/apache-delta-table.png)
> *The salesorders Delta table visible in the Lakehouse Explorer Tables section.*

> **Screenshot placeholder**
> ![SQL Query Results](./screenshots/apache-sql-query-results.png)
> *Spark SQL query results showing gross revenue grouped by year.*

---

### 8. Visualize Data

**What:** We use Python's matplotlib and seaborn libraries to create charts directly inside the notebook from Spark SQL query results.

**Why:** Tables of numbers are hard to interpret at a glance. Charts reveal trends, comparisons, and anomalies instantly. Running visualizations inside the notebook means there is no need to export data to a separate BI tool for basic analysis — the full story from raw data to insight lives in one place.

**Prepare the data:**
```python
sqlQuery = "SELECT CAST(YEAR(OrderDate) AS CHAR(4)) AS OrderYear, \
                SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue, \
                COUNT(DISTINCT SalesOrderNumber) AS YearlyCounts \
            FROM salesorders \
            GROUP BY CAST(YEAR(OrderDate) AS CHAR(4)) \
            ORDER BY OrderYear"
df_spark = spark.sql(sqlQuery)
df_sales = df_spark.toPandas()  # matplotlib requires a Pandas DataFrame
```

**Bar chart with matplotlib:**
```python
from matplotlib import pyplot as plt

plt.clf()
fig = plt.figure(figsize=(8, 3))
plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')
plt.title('Revenue by Year')
plt.xlabel('Year')
plt.ylabel('Revenue')
plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
plt.xticks(rotation=45)
plt.show()
```

**Side-by-side subplots (bar + pie):**
```python
from matplotlib import pyplot as plt

plt.clf()
fig, ax = plt.subplots(1, 2, figsize=(10, 4))

ax[0].bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')
ax[0].set_title('Revenue by Year')

ax[1].pie(df_sales['YearlyCounts'])
ax[1].set_title('Orders per Year')
ax[1].legend(df_sales['OrderYear'])

fig.suptitle('Sales Data')
plt.show()
```

**Line chart with seaborn:**
```python
import seaborn as sns

plt.clf()
sns.set_theme(style="whitegrid")
ax = sns.lineplot(x="OrderYear", y="GrossRevenue", data=df_sales)
plt.show()
```

> **Screenshot placeholder**
> ![matplotlib Bar Chart](./screenshots/apache-matplotlib-bar-chart.png)
> *Revenue by year displayed as a customized bar chart using matplotlib.*

> **Screenshot placeholder**
> ![seaborn Line Chart](./screenshots/apache-seaborn-line-chart.png)
> *Yearly revenue trend displayed as a line chart using seaborn.*

---

## 🔄 Transformations Applied

| Transformation | Input | Output | Why |
|---|---|---|---|
| Define schema | Raw CSV (no types) | Typed DataFrame | Prevents silent type errors |
| Add `Year` column | `OrderDate` | `Year` (Integer) | Enables year-based grouping |
| Add `Month` column | `OrderDate` | `Month` (Integer) | Enables month-based partitioning |
| Split customer name | `CustomerName` | `FirstName`, `LastName` | Normalises name data |
| Save as Parquet | DataFrame | `Files/transformed_data/` | Efficient columnar storage |
| Partition by Year/Month | DataFrame | `Files/partitioned_data/` | Improves query performance |
| Register Delta table | DataFrame | `salesorders` table | Enables SQL access |

---

## 🔑 Key Concepts

| Concept | Description |
|---|---|
| **PySpark** | Python API for Apache Spark — the default language for Fabric notebooks |
| **DataFrame** | Spark's distributed table structure for reading, transforming, and writing data |
| **Schema** | An explicit definition of column names and data types applied when reading files |
| **Parquet** | A columnar file format optimised for analytical queries — faster and smaller than CSV |
| **Partitioning** | Splitting data into subfolders by column value so Spark only reads relevant files |
| **Delta Lake** | Open-source table format adding ACID transactions and versioning over Parquet files |
| **Spark SQL** | SQL interface for querying Delta tables directly inside a notebook via `%%sql` |
| **matplotlib** | Core Python plotting library for fully customisable chart creation |
| **seaborn** | Higher-level plotting library built on matplotlib with built-in themes and simpler syntax |

---

## 📸 Screenshots

Create a `screenshots/` folder at the root of this repository and add the following images:

| File | Description |
|---|---|
| `screenshots/architecture.png` | Your architecture diagram showing the end-to-end flow |
| `screenshots/apache-csv-files-uploaded.png` | orders folder with CSV files in Lakehouse Explorer |
| `screenshots/apache-new-notebook.png` | New notebook with markdown title cell |
| `screenshots/apache-dataframe-schema.png` | DataFrame output with defined schema |
| `screenshots/apache-data-exploration.png` | Aggregation / groupBy results |
| `screenshots/apache-partitioned-data.png` | Partitioned folder hierarchy in Explorer |
| `screenshots/apache-delta-table.png` | salesorders table in Lakehouse Explorer |
| `screenshots/apache-sql-query-results.png` | Spark SQL revenue query results |
| `screenshots/apache-matplotlib-bar-chart.png` | Revenue bar chart from matplotlib |
| `screenshots/apache-seaborn-line-chart.png` | Revenue line chart from seaborn |

---

## 🧹 Clean Up

To remove all resources after completing the lab:

1. On the notebook menu, select **Stop session** to end the Spark session.
2. In the left navigation bar, select the icon for your workspace.
3. Select **Workspace settings → General**.
4. Scroll down and select **Remove this workspace → Delete**.

---

## 📚 Resources

- [Microsoft Fabric Documentation](https://learn.microsoft.com/fabric/)
- [Apache Spark in Microsoft Fabric](https://learn.microsoft.com/fabric/data-engineering/spark-overview)
- [PySpark DataFrame Documentation](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/dataframe.html)
- [Delta Lake Overview](https://delta.io/)
- [matplotlib Documentation](https://matplotlib.org/stable/index.html)
- [seaborn Documentation](https://seaborn.pydata.org/)
- [Source Lab (MS Learn)](https://microsoftlearning.github.io/mslearn-fabric/)

---

## 🪪 License

This project is based on lab content from the [Microsoft Learn Fabric repository](https://github.com/MicrosoftLearning/mslearn-fabric).  
© 2025 Microsoft — used for educational purposes.