
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
