# 🌊 Advanced Data Engineering with Spark & Delta Lake

This project focuses on the advanced capabilities of **Delta Lake** within the Microsoft Fabric ecosystem. It demonstrates how to handle complex data scenarios, including schema enforcement, table versioning (Time Travel), and real-time data streaming using **Apache Spark**.

---

## 🎯 Project Objectives
The core goal of this project was to implement a robust relational layer over a Data Lake. Key technical milestones include:
* **Managed vs. External Tables:** Understanding metadata ownership and data persistence.
* **Time Travel:** Implementing data recovery and auditing via the transaction log.
* **Structured Streaming:** Building a real-time IoT data pipeline.
* **Spark SQL Integration:** Performing complex aggregations and visualizations.

---

## 🛠️ Technical Implementation & "The Why"

### 1. Schema Enforcement & Data Loading
Instead of relying on "Inference," I manually defined a `StructType` schema for the `products.csv` dataset.
* **Why?** In production, manual schema definition prevents "Schema Drift" and ensures that data types (like `ListPrice` as Double) are strictly maintained for downstream calculations.

### 2. Managed vs. External Tables
I implemented two distinct table architectures:
* **Managed Tables:** Created in the `/Tables` folder. Fabric manages both data and metadata.
* **External Tables:** Created in the `/Files` folder. I maintain ownership of the data files.
* **Key Finding:** I proved that dropping an **External Table** only deletes the metadata, keeping the actual Parquet files safe—critical for multi-engine data access.

> **📸 Evidence: Table Properties**
> ![Table Location Analysis](/screenshots/Managed&External_Tables.png)

---

### 3. Data Versioning & Time Travel
Using the Delta Lake **Transaction Log**, I performed a 10% price update on 'Mountain Bikes' and successfully retrieved the "pre-update" state of the data.
* **The Logic:** `spark.read.format("delta").option("versionAsOf", 0).load(path)`
* **Why?** This feature is a lifesaver for data auditing and recovering from accidental "Mass Updates" in a production environment.

> **📸 Evidence: Data Versioning Results**
> ![Data Versioning Output](/screenshots/Data_Versioning.png)


---

### 4. Real-time IoT Streaming
I simulated a live **Internet of Things (IoT)** stream where device status data was written to a Delta sink.
* **Streaming Logic:** Used `readStream` to monitor a folder and `writeStream` with a **Checkpoint** location.
* **Why?** Checkpointing ensures **fault tolerance**; if the stream fails, it resumes exactly where it left off, ensuring no data loss.

> **📸 Evidence: Streaming Results**
> ![Streaming Output](/screenshots/Streaming.png)
> 
> ![Streaming Output](/screenshots/Streaming_2.png)

---

### 5. Advanced Analytics & Visualization
I created **Temporary Views** to transform raw product data into business insights, calculating Min/Max/Avg prices per category.
* **Visualization:** Utilized Fabric's native charting tools to convert SQL results into actionable bar charts.

> **📸 Evidence: Advanced Analytics & Visualization Results**
> ![Visualization Output](/screenshots/Visualization.png)
---

## 💡 Key Takeaways & Skills
* **ACID Compliance:** Brought reliability to the Data Lake with Spark-based transactions.
* **Direct Lake Integration:** Tables created here are immediately available for Power BI without data movement.
* **Scalability:** Managed streaming and batch data within a single unified platform.

---

*Developed as part of the Microsoft Fabric Data Engineering Path.*
