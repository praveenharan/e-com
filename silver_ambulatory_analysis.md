
### Importing required library

```sql
from pyspark.sql import SparkSession, Window
from delta.tables import DeltaTable
import pyspark.sql.functions as F
import pyspark.sql.types as T

```
### Cleaning stg_customers
```sql
appointments_df = spark.read.table("stg_appointments")

appointments = appointments_df \
    .filter(F.col("AppointmentID").isNotNull()) \
    .withColumn("WaitTimeMinutes", F.col("WaitTimeMin")) \
    .withColumn("IsNoShow", F.when(F.col("StatusKey") == "No-Show", 1).otherwise(0)) \
    .withColumn("StandardizedStatus", F.upper(F.col("StatusKey"))) \
    .dropDuplicates(["AppointmentID"])

# Note: Keeping audit columns (UpdatedTimestamp) here, and I will exclude them when moving to the Gold layer.

# Write to Silver Lakehouse
appointments.write.format("delta").mode("overwrite").saveAsTable("Silver.silver_appointments")
```

### Cleaning Locations

```sql

location_df = spark.read.table('stg_location')

location = location_df.select(
    col("LocationKey").cast("string"),
    trim(col("ClinicName")).alias("ClinicName"),
    col("ZipCode").cast("string"), # Cast to string to keep leading zeros
    col("load_timestamp"),
    col("source_system"),
    current_timestamp().alias("silver_load_at") # Track when it hit Silver
).dropDuplicates(["LocationKey"])

# Write to Silver Lakehouse
location.write.format("delta").mode("overwrite").saveAsTable("Silver.silver_locations")
```
### Cleaning Patients
```sql
# Transform and Clean
silver_patients_df = patients.select(
    col("PatientKey"),
    col("Age"),
    upper(col("Gender")).alias("Gender"), # Standardizing case
    col("PrimaryInsurance"),
    col("load_timestamp"),
    col("source_system"),
    col("source_file")
).filter(col("Age") >= 0) \
 .dropDuplicates(["PatientKey"])

# Write to Silver Lakehouse as a Delta Table
silver_patients_df.write.format("delta") \
    .mode("overwrite") \
    .option("mergeSchema", "true") \
    .saveAsTable("Silver.silver_patients")
```

### Cleaning Providers
```python
# Clean and refine Provider data
silver_providers_df = providers.select(
    col("ProviderKey"),
    trim(regexp_replace(col("Name"), "Dr. ", "")).alias("ProviderName"), # Remove 'Dr.' prefix
    col("Specialty"),
    col("ExperienceYrs"),
    col("load_timestamp"),
    col("source_system"),
    col("source_file")
).dropDuplicates(["ProviderKey"])

# Write to Silver Lakehouse
silver_providers_df.write.format("delta") \
    .mode("overwrite") \
    .saveAsTable("Silver.silver_providers")
```
