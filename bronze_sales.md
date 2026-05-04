## import random

# Standard PySpark components
from pyspark.sql import SparkSession, Window

# Use aliases to avoid redundant imports and namespace conflicts
import pyspark.sql.functions as F
import pyspark.sql.types as T

# For custom logic
from pyspark.sql.functions import udf

def init_spark(app_name="DataProcessor"):
    """Initializes a Spark Session."""
    return SparkSession.builder \
        .appName(app_name) \
        .getOrCreate()
