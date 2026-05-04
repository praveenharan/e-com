### Importing required library

```sql
import random
from pyspark.sql import SparkSession, Window
import pyspark.sql.functions as F
import pyspark.sql.types as T
```

### Getting all the files from Bronze/Files

```sql
# List of messy files and their intended table names
datasets = [
    ("customers.csv", "stg_customers"),
    ("orders_raw.csv", "stg_orders"),
    ("order_items_raw.csv", "stg_order_items"),
    ("products_raw.csv", "stg_products"),
    ("events_raw.csv", "stg_events"),
    ("scd2_customer_dimensions.csv", "stg_customer_scd")
]

for file_name, table_name in datasets:
    print(f"Processing {file_name}...")
    
    # 1. Read the raw CSV from the BRONZE Lakehouse explicitly
    df = spark.read.format("csv") \
        .option("header", "true") \
        .option("inferSchema", "true") \
        .load(f"abfss://Sales_Data_Platform@onelake.dfs.fabric.microsoft.com/lh_Sales_Bronze.Lakehouse/Files/sales/{file_name}")
    
    # 2. Add Meta Data (Audit Columns)
    df_with_metadata = df \
        .withColumn("load_timestamp", current_timestamp()) \
        .withColumn("source_system", lit("Landing_Zone_CSV")) \
        .withColumn("source_file", lit(file_name))
    
    # 3. Write to the Bronze Lakehouse as a Delta Table
    # This will create the table automatically if it doesn't exist
    df_with_metadata.write.format("delta") \
        .mode("overwrite") \
        .option("overwriteSchema", "true") \
        .saveAsTable(f"{table_name}")

print("Success: All files moved to Bronze Delta Tables.")
```



