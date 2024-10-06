# 1. Get started

Requires to have installed one of the following:
- pyspark and delta-spark
- Databricks Connect
- Spark and Delta Connect

`pip install lakehouse-ns`

```
from lakehouse import bronze, silver

spark = <Your Spark Session>

spark.sql(f"CREATE SCHEMA IF NOT EXISTS <catalog>.<schema>")

options = {
    "catalog": "<catalog>",
    "target_schema": "<schema>" 
}


class StarWarsBronze(bronze.BronzeOverwrite):
    def load(self, table):
        return spark.read.format("SWAPI").load(table)
    
bronze_instance = StarWarsBronze(spark, **options)
bronze_instance.execute_one("people")
```

See samples here: https://github.com/datanikkthegreek/lakehouse-docu/tree/main/samples