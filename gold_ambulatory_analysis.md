# Copying Table to Warehouse to create Semantic Model
### Creating a Date Table

```sql
-- 1. Creating the Date table structure 
CREATE TABLE dim_date (
    date_key INT NOT NULL, -- Keep it NOT NULL for best practice
    full_date DATE,
    day_of_week_name VARCHAR(10),
    day_of_month INT,
    month_name VARCHAR(10),
    quarter INT,
    year INT,
    is_weekend BIT
);

-- 2. Logic to populate 
DECLARE @StartDate DATE = '2024-01-01', @EndDate DATE = '2026-12-31';

WHILE @StartDate <= @EndDate
BEGIN
    INSERT INTO dim_date (date_key, full_date, day_of_week_name, day_of_month, month_name, quarter, year, is_weekend)
    VALUES (
        CAST(FORMAT(@StartDate, 'yyyyMMdd') AS INT), 
        @StartDate,
        DATENAME(WEEKDAY, @StartDate),
        DAY(@StartDate),
        DATENAME(MONTH, @StartDate),
        DATEPART(QUARTER, @StartDate),
        YEAR(@StartDate),
        CASE WHEN DATEPART(WEEKDAY, @StartDate) IN (1, 7) THEN 1 ELSE 0 END
    );
    SET @StartDate = DATEADD(DAY, 1, @StartDate);
END;

SELECT *
FROM dim_date
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
