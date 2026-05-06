### Importing required library

```sql
import random
from pyspark.sql import SparkSession, Window
import pyspark.sql.functions as F
import pyspark.sql.types as T
```

### Getting all the required files from Bronze/Files

```sql
# List of messy files and their intended table names
datasets = [
    ("appointments.csv", "stg_appointments"),
    ("location.csv", "stg_location"),
    ("patients.csv", "stg_patients"),
    ("providers.csv", "stg_providers")

]

for file_name, table_name in datasets:
    print(f"Processing {file_name}...")
    
    # 1. Read the raw CSV from the BRONZE Lakehouse explicitly
    df = spark.read.format("csv") \
        .option("header", "true") \
        .option("inferSchema", "true") \
        .load(f"abfss://Sales_Data_Platform@onelake.dfs.fabric.microsoft.com/Bronze.Lakehouse/Files/Ambulatory Analyst/{file_name}")
    
    # 2. Add Meta Data (Audit Columns)
    df_with_metadata = df \
        .withColumn("load_timestamp", F.current_timestamp()) \
        .withColumn("source_system", F.lit("Landing_Zone_CSV")) \
        .withColumn("source_file", F.lit(file_name))
    
    # 3. Write to the Bronze Lakehouse as a Delta Table
    # This will create the table automatically if it doesn't exist
    df_with_metadata.write.format("delta") \
        .mode("overwrite") \
        .option("overwriteSchema", "true") \
        .saveAsTable(f"{table_name}")

print("Success: All files moved to Bronze Delta Tables.")
```

print("Success: All files moved to Bronze Delta Tables.")
```



