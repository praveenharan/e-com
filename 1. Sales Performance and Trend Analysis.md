# Sales Performance and Trend Analysis
### Using SQL

```sql
SELECT 
    c.state,
    p.category,
    d.full_date,
    SUM(f.line_total_amount) AS total_revenue,
    COUNT(DISTINCT f.order_id) AS daily_order_count
FROM fact_sales f
JOIN dim_customers c ON f.customer_id = c.customer_id
JOIN dim_products p ON f.product_id = p.product_id
JOIN dim_date d ON f.order_date_key = d.date_key
WHERE c.is_current = true  -- Ensures we use the most recent customer info
  AND d.full_date >= DATEADD(day, -7, CURRENT_DATE())
GROUP BY 1, 2, 3
ORDER BY d.full_date DESC, total_revenue DESC;

```
### Using PySpark
```sql
from pyspark.sql import functions as F

# 1. Filter dimensions for current records and products
current_customers = spark.table("dim_customers").filter(F.col("is_current") == True)
products = spark.table("dim_products")
dates = spark.table("dim_date").filter(F.col("full_date") >= F.date_sub(F.current_date(), 7))
sales = spark.table("fact_sales")

# 2. Join and Aggregate
performance_df = sales.join(current_customers, "customer_id") \
    .join(products, "product_id") \
    .join(dates, sales.order_date_key == dates.date_key) \
    .groupBy("state", "category", "full_date") \
    .agg(
        F.sum("line_total_amount").alias("total_revenue"),
        F.countDistinct("order_id").alias("daily_order_count")
    ) \
    .orderBy(F.col("full_date").desc(), F.col("total_revenue").desc())

performance_df.show()
```
