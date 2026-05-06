# Healthcare Analytics & Operational Excellence - SQL / PySpark / Fabric

## Problem Statement
The clinic is experiencing unpredictable patient flow, leading to extended wait times during peak hours and underutilized staff during off-peak periods. Without visibility into appointment duration and wait time trends by provider specialty, the clinic cannot optimize scheduling, resulting in decreased patient satisfaction and staff burnout.


### 1. Clinical Performance and Trend Analysis
* **Objective:** Quantify healthcare delivery across various medical specialties and clinic locations.
* **KPIs:** Track operational metrics including No-Show Rates, Average Wait Times, and Total Patient Charges over rolling 30-day periods.
* **Impact:** Enables administrators to identify bottlenecks in specific departments (e.g., Cardiology vs. Pediatrics) and adjust staffing levels in real-time.


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
