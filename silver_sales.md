
### Importing required library

```sql
from pyspark.sql.functions import col, trim, lower, when, to_timestamp, initcap, upper, current_timestamp, lit, date_format
from delta.tables import DeltaTable 
from delta.tables import *
from pyspark.sql.window import Window
from pyspark.sql.functions import row_number
from pyspark.sql import functions as F
from pyspark.sql.types import StringType

from pyspark.sql.functions import from_json
from pyspark.sql.functions import col, from_json, get_json_object
from pyspark.sql.types import MapType, StringType
from pyspark.sql.types import MapType, StringType

```
