# Copying Table to Warehouse to create Semantic Model

```sql
import random
from pyspark.sql import SparkSession, Window
import pyspark.sql.functions as F
import pyspark.sql.types as T
from pyspark.sql.functions import col
);

-- Tables from Silver
```sql
appointments = spark.read.table("Silver.silver_appointments")
patients = spark.read.table("Silver.silver_patients")
admissions = spark.read.table("Silver.silver_admission")
providers = spark.read.table("Silver.silver_providers")
# diagnosis = spark.read.table("Silver.silver_diagnosis")

```
### Creating Fact Sales Table
```sql
-- fact_sales table, JOIN between the orders and order_items

CREATE TABLE fact_sales AS
SELECT 
    -- Keys for the Star Schema
    oi.order_id,
    oi.line_num,
    o.customer_id,
    oi.product_id,
    
    -- Date formatting for a Date Dimension (YYYYMMDD)
    CAST(FORMAT(o.order_ts, 'yyyyMMdd') AS INT) AS order_date_key,
    
    -- Quantitative Measures
    oi.qty,
    oi.unit_price,
    (oi.qty * oi.unit_price) AS line_total_amount,
    o.order_status
FROM dim_order_items oi
JOIN dim_orders o ON oi.order_id = o.order_id;
```

### Moving dim_events to warehouse
```sql
SELECT
    event_id,
    customer_id,
    event_ts,
    event_type
INTO dim_events
FROM lh_Sales_Silver.dbo.silver_events
```

### Moving SCD2 to Warehouse
```sql
SELECT 
    customer_id,
    name,
    state,
    eff_start,
    eff_end,
    is_current
INTO dim_customers
FROM lh_Sales_Silver.dbo.silver_scd2
```
 ### Moving order_items to Warehouse
```sql
SELECT
    order_id, customer_id, order_ts, order_status, order_total
INTO fact_orders
FROM lh_Sales_Silver.dbo.silver_orders
```
### Moving customers to Warehouse
```sql
SELECT
    customer_id,
    name,
    state,
    is_current
FROM dim_customers
WHERE is_current = 'Yes'
```
