# Copying Table to Warehouse to create Semantic Model

```sql
import random
from pyspark.sql import SparkSession, Window
import pyspark.sql.functions as F
import pyspark.sql.types as T
from pyspark.sql.functions import col
);
```
### Tables from Silver

```sql
appointments = spark.read.table("Silver.silver_appointments")
patients = spark.read.table("Silver.silver_patients")
admissions = spark.read.table("Silver.silver_admission")
providers = spark.read.table("Silver.silver_providers")

```
### Transforming Silver to Gold
```sql
# Transforming Silver to Gold
fact_appointments_final = appointments.\
    select(
        col("AppointmentID"),
        col("PatientKey"),
        col("ProviderKey"),
        col("LocationKey"),
        col("DateKey"),
        col("DurationMin"),
        col("WaitTimeMin"),
        col("WaitTimeMinutes"),
        col("Charges"),
        col("IsNoShow"),
        col("StandardizedStatus").alias("Status")
)

# Write to the Gold Lakehouse
# Fabric automatically applies V-Order when writing to Delta tables in a Notebook
fact_appointments_final.\
    write.\
    format("delta").\
    mode("overwrite").\
    saveAsTable("gold_lh.dbo.fact_appointments")
```

### Moving dim_patients to gold
```sql
patients_final = patients.\
    select("PatientKey",
    "Age",
    "Gender",
    "PrimaryInsurance")

patients_final.\
    write.\
    format("delta").\
    mode("overwrite").\
    saveAsTable("gold_lh.dbo.dim_patients")
```

 ### Moving admissions to Warehouse
```sql
admissions_final = admissions.\
    select(
        "AdmissionID",
        "PatientID",
        "PhysicianID",
        "AdmissionDate",
        "DateKey",
        "DiagnosisCode",
        "LengthOfStay"
    )

admissions_final.\
    write.\
    mode("overwrite").\
    format("delta").\
    saveAsTable("gold_lh.dbo.dim_admissions")
```
```sql
providers_final = providers.select("ProviderKey", "ProviderName", "Specialty", "ExperienceYrs")

providers_final.\
    write.\
    mode("overwrite").\
    format("delta").\
    saveAsTable("gold_lh.dbo.dim_providers")
```
