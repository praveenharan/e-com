
### Importing required library

```sql
from pyspark.sql import SparkSession, Window
from delta.tables import DeltaTable
import pyspark.sql.functions as F
import pyspark.sql.types as T

```
### Cleaning stg_customers
```sql
customer_df = spark.read.table("lh_Sales_Bronze.dbo.stg_customers")

# 1. Standardize Casing and Trim whitespace
# This handles 'ca' vs 'CA' and 'ny' vs 'NY'
df_cleaned = customer_df.\
    withColumn("name", trim(initcap(col("name")))).\
    withColumn("state", upper(trim(col("state"))))

# 2. Fix State Inconsistencies (Mapping 'TEXAS' to 'TX')
# You can expand this mapping as needed for your Medallion architecture
state_map = {"TEXAS": "TX", "INVALIDSTATE": "UNKNOWN"}
def map_states(state):
    return state_map.get(state, state)

# 3. Handle Nulls and Invalid Data
# Providing a default for missing names and filtering invalid states
df_cleaned = df_cleaned.\
    fillna({"name": "Unknown Customer"}).\
    filter(col("state") != "UNKNOWN")


# 4. Handle Timestamps
# Convert 'created_at' to a standard timestamp format for SCD logic
df = df_cleaned \
    .withColumn("created_at", to_timestamp(col("created_at"), "M/d/yy H:mm")) \
    .withColumn("created_at", 
        when(col("created_at").isNull(), 
             to_timestamp(date_format(current_timestamp(), "yyyy-MM-dd HH:mm:ss"))
        ).otherwise(col("created_at")))

window_spec = Window.\
    partitionBy("customer_id").\
    orderBy(col("created_at").desc())

customer_df = df.\
    withColumn("rank", row_number().over(window_spec)).\
    filter(col("rank") == 1).\
    drop("rank")

customer_df.\
    write.\
    mode("overwrite").\
    option("overwriteScema", "true").\
    format("delta").\
    saveAsTable("silver_customers")

display(customer_df)
```

### Cleaning Events

```sql
# 1. Read your staging table
df_events = spark.read.table("lh_Sales_Bronze.dbo.stg_events")

# 2. Apply robust cleaning
df_cleaned_events = df_events \
    .dropDuplicates(["event_id"]) \
    .withColumn("event_ts", try_to_timestamp(col("event_ts"))) \
    .filter(col("payload").contains("{"))\
    .dropna()

# 3. Write to a new 'silver_events_cleaned' table
df_cleaned_events.write.format("delta") \
    .mode("overwrite") \
    .option("overwriteSchema", "true") \
    .saveAsTable("silver_events")

# 4. Display
display(df_cleaned_events)
```
### Cleaning Orders
```sql
# 1. Read the raw bronze table
df_orders = spark.read.table("lh_Sales_Bronze.dbo.stg_orders")

# 2. Apply Silver-level cleaning logic
df_silver_orders = df_orders \
    .dropDuplicates(["order_id"]) \
    .withColumn("order_status", upper(col("order_status"))) \
    .withColumn("order_ts", try_to_timestamp(col("order_ts"), lit("M/d/yy H:mm"))) \
    .withColumn("order_total", when(col("order_total") < 0, abs(col("order_total"))).otherwise(col("order_total"))) \
    .filter(col("order_ts").isNotNull()) 

# 3. Write to Silver Lakehouse
df_silver_orders.write.format("delta") \
    .mode("overwrite") \
    .saveAsTable("lh_Sales_Silver.dbo.silver_orders")

display(df_silver_orders)
```

### Cleaning Items
```python
# 1. Read the raw bronze order items
df_items = spark.read.table("lh_Sales_Bronze.dbo.stg_order_items")

# 2. Apply cleaning and enrichment
df_silver_items = df_items \
    .dropDuplicates(["order_id", "line_num"]) \
    .withColumn("qty", col("qty").cast("int")) \
    .withColumn("unit_price", col("unit_price").cast("double")) \
    .withColumn("line_total", round(col("qty") * col("unit_price"), 2)) \
    .filter(col("qty") > 0) # Remove any zero-quantity rows if they exist

# 3. Write to Silver Lakehouse
df_silver_items.write.format("delta") \
    .mode("overwrite") \
    .saveAsTable("lh_Sales_Silver.dbo.silver_order_items")

display(df_silver_items)
```

### Cleaning Products
```sql
# 1. Read the raw bronze products table
df_products = spark.read.table("lh_Sales_Bronze.dbo.stg_products")

# 2. Apply cleaning logic
df_silver_products = df_products \
    .dropDuplicates(["product_id"]) \
    .withColumn("category", initcap(col("category"))) \
    .withColumn("category", coalesce(col("category"), lit("Uncategorized"))) \
    .withColumn("price", when(col("price") <= 0, lit(None))
                        .when(col("price") > 5000, lit(None)) # Handling outliers
                        .otherwise(col("price"))) \
    .filter(col("price").isNotNull()) \
    .filter(~col("sku").contains("B00")) # Filtering non-standard SKUs if required

# 3. Write to Silver Lakehouse
df_silver_products.write.format("delta") \
    .mode("overwrite") \
    .saveAsTable("lh_Sales_Silver.dbo.silver_products")

display(df_silver_products)
```
