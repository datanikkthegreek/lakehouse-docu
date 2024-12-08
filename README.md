Lakehouse-NS gives you a simple framework to implement your lakehouse based on the Medallion Architecture. 

- Currently, the frameworks supports the Bronze and Silver layer
- It currently, supports Spark. (Tested with Spark 4.0) More engines like Daft or Polars are in the backlog
- Currently, it supports Delta Lake as lakehouse format
- The framework will also be extended step by step with more baseline logic

# 1. Set-Up

Requires to have installed one of the following:
- pyspark and delta-spark
- Databricks Connect
- Spark and Delta Connect
- default spark session on Databricks or Fabric

`pip install lakehouse-ns`

Also you need to have a catalog set-up and your bronze and silver schema(s)

That's already it!

# 2. Get Started

Just import the Bronze and Silver classes and overwrite the load or transform functions. That's it.

```
from lakehouse import bronze, silver

spark = <Your Spark Session>

#Create your schemas
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

See detailed samples here: https://github.com/datanikkthegreek/lakehouse-docu/tree/main/samples

# 3. Options

You can/must pass in the Bronze and Silver class the following options. Besides you can specifiy any custom options which you can access via self.options in your class.

| Option        | Description                                                                             | Type    | Default       | Bronze       | Silver   |
| ------------- | --------------------------------------------------------------------------------------- | ------- | ------------- | ------------ | -------- |
| catalog       | The name of the catalog, e.g. spark_catalog, hive_metastore or any other custom catalog | String  | To be defined | Required     | Required |
| source_schema | The schema from which the data is loaded                                                | String  | To be defined | Not Required | Required |
| target_schema | The schema to which the data are written                                                | String  | To be defined | Required     | Required |
| merge_schema  | If the schema should be automatically envolved/merged                                   | Boolean | FALSE         | Optional     | Optional |
