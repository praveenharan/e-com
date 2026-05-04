# E-Commerce Data Modeling and Revenue Integrity - SQL / PySpark

## Problem Statement
The goal of this project is to develop a robust analytics solution that transforms raw transactional data into actionable business intelligence. The solution addresses the following four key pillars:

### 1. Sales Performance and Trend Analysis
* **Objective:** Quantify revenue across geographic regions and product categories.
* **KPIs:** Track time-bound metrics including the last 7 days of revenue and daily order counts.
* **Impact:** Enables stakeholders to identify high-growth regions and inventory demands in real-time.

### 2. Customer Behavioral Insights
* **Objective:** Identify unique active customers and implement sessionization logic.
* **Engagement:** Analyze user engagement patterns to understand the "path to purchase" and session duration.
* **Impact:** Provides a foundation for targeted marketing and improved user experience.

### 3. Data Integrity and Accuracy
* **Objective:** Perform rigorous audit checks on financial data.
* **Logic:** Ensure that the reported `order_total` aligns with granular line-item calculations.
* **Thresholds:** Implement automated flags for discrepancies that exceed defined variance thresholds.

### 4. Advanced Data Orchestration
* **SCD Type 2:** Manage complex data states by implementing **Slowly Changing Dimensions** to maintain a full history of customer attribute changes.
* **Localization:** Handle time-zone-specific reporting requirements to ensure daily metrics are accurate to the local region of operation.

---

## Technical Stack
* **Data Warehouse:** Microsoft Fabric (OneLake, Lakehouse, Data Warehouse)
* **Transformation Layer:** Spark (PySpark via Fabric Notebooks) & SQL (T-SQL for Warehouse/Lakehouse Endpoint)
* **Orchestration:** Data Factory (Fabric Pipelines)
* **Visualization:** Power BI
