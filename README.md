# Ambulatory Performance & Clinical Insights Dashboard - SQL / PySpark / Fabric

## Problem Statement
> Ambulatory leadership currently lacks a unified view of clinical and operational metrics, leading to delayed decision-making and difficulties in identifying performance gaps across multi-disciplinary workgroups. This project aims to bridge the gap between complex raw data sources (medical records, financial, and telecom) and actionable insights by developing a validated, end-to-end data pipeline and visualization suite. The goal is to ensure data integrity and provide clinic leaders with the clarity needed to optimize patient care and operational efficiency.

> The clinic is experiencing unpredictable patient flow, leading to extended wait times during peak hours and underutilized staff during off-peak periods. Without visibility into appointment duration and wait time trends by provider specialty, the clinic cannot optimize scheduling, resulting in decreased patient satisfaction and staff burnout.


### 1. Clinical Performance and Trend Analysis
* **Objective:** Quantify healthcare delivery across various medical specialties and clinic locations.
* **KPIs:** Track operational metrics including No-Show Rates, Average Wait Times, and Total Patient Charges over rolling 30-day periods.
* **Impact:** Enables administrators to identify bottlenecks in specific departments (e.g., Cardiology vs. Pediatrics) and adjust staffing levels in real-time.


### 2. Patient Behavioral & Admission Insights
* **Objective:** Analyze patient journey patterns using dim_patients and dim_admissions.
* **Engagement:** Implement logic to track Length of Stay (LOS) and readmission patterns by diagnosis code.
* **Impact:** Provides a foundation for proactive patient care and identifying "high-frequency" individuals who may benefit from specialized care management.

### 3. Data Integrity and Revenue Assurance
* **Objective:** Perform rigorous audit checks on billing data within the Silver and Gold layers.
* **Logic:** Validate that Total Charges in the FactAppointments table align with insurance provider fee schedules and duration-based billing logic.
* **Medallion Strategy:** Specifically excluded technical metadata cleaning flags and silver table audit columns from the final Gold reporting layer to ensure a clean, business-ready schema.

### 4. Advanced Data Orchestration
* **Star Schema Implementation:**  Developed a robust dimensional model, featuring one-to-many relationships between Patients, Providers, and Admissions.
* **SCD Type 2:** Implemented Slowly Changing Dimensions for dim_providers to track changes in specialty or seniority over time, ensuring historical appointment data remains contextually accurate.
* **Direct Lake Mode:** Leveraged Microsoft Fabric’s Direct Lake technology to provide Power BI with sub-second performance by reading Delta tables directly from OneLake without the need for import or Refresh.
---

## Technical Stack
* **Data Warehouse:** Microsoft Fabric (OneLake, Lakehouse, Data Warehouse, SQL Analytics Endpoint)
* **Transformation Layer:** Spark (PySpark via Fabric Notebooks) & SQL (T-SQL for Warehouse/Lakehouse Endpoint)
* **Languages:** PySpark (Spark 3.4) for Medallion Layer transformations and T-SQL for Gold layer views.
* **Orchestration:** Fabric Pipelines (Data Factory) for end-to-end ELT scheduling.
* **Visualization:** Power BI (Direct Lake) for real-time operational dashboards.


### Analysis Framework (What / When / How / So What)

| Framework | Application |
| :--- | :--- |
| **What** | Identified that **Cardiology** accounts for 40% of total revenue but 60% of total patient wait time. |
| **When** | Discovered a 22% spike in **No-Shows** on Monday mornings compared to mid-week averages. |
| **How** | Correlated `ExperienceYrs` with `DurationMin` to determine that senior providers handle complex cases 15% faster. |
| **So What** | Recommended a dynamic overbooking strategy for Monday mornings to recover an estimated **$12k/month** in lost revenue. |
