
import random

# PySpark Core & Windowing
from pyspark.sql import SparkSession, Window

# PySpark Functions & Types (Aliased for clarity and namespace safety)
import pyspark.sql.functions as F
import pyspark.sql.types as T

# Initialize Spark Session (Optional: depending on your module structure)
def get_spark_session(app_name="AnalyticsApp"):
    return SparkSession.builder \
        .appName(app_name) \
        .getOrCreate()

# Usage Examples:
# Schema: T.StructType([T.StructField("id", T.IntegerType())])
# Logic:  df.withColumn("new_col", F.col("old_col"))
# Window: Window.partitionBy("id").orderBy("date")
