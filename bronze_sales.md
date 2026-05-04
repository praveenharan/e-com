# Importing required library

### import random
```sql
import random
from pyspark.sql import SparkSession, Window
import pyspark.sql.functions as F
import pyspark.sql.types as T
```

### 1. Get the total number of students

#### PySpark Core & Windowing

```sql
# Initialize Spark Session (Optional: depending on your module structure)
def get_spark_session(app_name="AnalyticsApp"):
    return SparkSession.builder \
        .appName(app_name) \
        .getOrCreate()
```



