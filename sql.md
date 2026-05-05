## sql

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
