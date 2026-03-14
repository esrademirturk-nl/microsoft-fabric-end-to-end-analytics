## 🏗️ Project Architecture
The project follows the **Medallion Architecture** principles by utilizing:
* **OneLake:** A unified, logical data lake for the entire organization.
* **Delta Lake:** High-performance storage layer with ACID transactions.
* **SQL Analytics Endpoint:** A read-only T-SQL interface for data exploration.

---

## 🛠️ Implementation Steps

### 1. Environment Setup
* Created a dedicated **Fabric Workspace** with Trial capacity enabled.
* Provisioned a **Lakehouse** artifact to manage both structured (Tables) and unstructured (Files) data.

### 2. Data Ingestion (Bronze Layer)
* **Upload:** Ingested raw `sales.csv` data directly into the OneLake storage.
* **Evidence:**
![Data Ingestion](/Microsoft_Fabric_Lakehouse/screenshots/Data_Ingestion.png)


### 3. Data Transformation (Gold Layer)
* **Delta Conversion:** Converted raw CSV files into **Managed Delta Tables** (Parquet format).
* **Evidence:**
![Data Transformation](Microsoft_Fabric_Lakehouse/screenshots/Data_Transformation.png)

### 4. Analytical Layer (SQL & Visual Insights)
The project utilized two primary methods to extract business value from the data:

#### A. T-SQL Mastery
Used the **SQL Analytics Endpoint** to perform high-performance aggregations.
```sql
SELECT Item, SUM(Quantity * UnitPrice) AS Revenue
FROM sales
GROUP BY Item
ORDER BY Revenue DESC;
```
Query Result:
![T-SQL Analysis](Microsoft_Fabric_Lakehouse/screenshots/T-SQL_Analysis.png)

#### B. Visual Query (Low-Code/No-Code)
Leveraged Power Query capabilities within Fabric to transform data visually, performing group-by operations to count distinct line items per order.
![Visual Query](Microsoft_Fabric_Lakehouse/screenshots/Visual_Query.png)

### 💡 Key Learnings & Takeaways
* Unification: Microsoft Fabric eliminates data silos via OneLake.

* Direct Lake Mode: Enables Power BI to analyze data directly from Parquet files without traditional refresh cycles.

* Versatility: Supports both SQL experts and low-code analysts seamlessly within a single SaaS environment.
