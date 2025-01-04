Lakehouse-NS gives you a simple framework to implement your lakehouse based on the Medallion Architecture. 

- Currently, the frameworks supports the Bronze and Silver layer
- It currently, supports Spark. (Tested with Spark 4.0) More engines like Daft or Polars are in the backlog
- Currently, it supports Delta Lake as lakehouse format
- The framework will also be extended step by step with more baseline logic

Some import links:

- "Homepage" = "https://github.com/datanikkthegreek/lakehouse-docu"
- "API Referance" = "https://nikkthegreek.codeberg.page/"
- "Samples" = "https://github.com/datanikkthegreek/lakehouse-docu/tree/main/samples"
- "Source" = "https://github.com/datanikkthegreek/lakehouse"
- "Issues" = "https://github.com/datanikkthegreek/lakehouse-docu/issues"
- "Project Planning" = "https://github.com/users/datanikkthegreek/projects/1/views/1"
- "Get in touch" = "https://www.linkedin.com/in/dr-nikolaos-servos-nikk-the-greek-a29137b3/"

# 1. Set-Up

Requires to have installed one of the following:
- pyspark and delta-spark
- Databricks Connect
- Spark and Delta Connect
- default spark session on Databricks or Fabric

`pip install lakehouse-ns`

Also you need to have a catalog set-up and create your bronze, silver and gold schema(s)

That's already it!

# 2. Get Started

Just import the Bronze, Silver and Gold classes and overwrite the load or transform functions. That's it.

```
from lakehouse import bronze, silver, gold

spark = <Your Spark Session>

#Create your schemas
spark.sql(f"CREATE SCHEMA IF NOT EXISTS <catalog>.<schema>")

options = {
    "catalog": "<catalog>",
    "target_schema": "<schema>" 
}


class StarWarsBronze(bronze.Bronze):
    def custom_load(self, table):
        results = []
        query = f"https://swapi.tech/api/{table}"
        json_request = requests.get(query).json()
        results.extend(json_request["results"])

        while json_request["next"]:
            json_request = requests.get(json_request["next"]).json()
            results.extend(json_request["results"])
        return spark.createDataFrame(results)
    
bronze_instance = StarWarsBronze(spark, **options)
bronze_instance.load().write(mode="overwrite").execute("people", "planets")
```

See detailed samples here: https://github.com/datanikkthegreek/lakehouse-docu/tree/main/samples

In the examples you will find also an notebook showing you how easy debugging and testing is despite using this framework.

# 3. Options

You can/must pass in the Bronze, Silver and Gold class the following options. Besides you can specifiy any custom options which you can access via self.options in your class.

| Option Type | Option              | Description                                                                                                                                                                                                                                   | Type    | Default       | Accepted Values                           | Bronze   | Silver   | Gold     |
| ----------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------- | ------------- | ----------------------------------------- | -------- | -------- | -------- |
| class       | catalog             | The name of the catalog, e.g. spark_catalog, hive_metastore or any other custom catalog                                                                                                                                                       | String  | To be defined | Any String                                | Required | Required | Required |
| class       | source_schema       | The schema from which the data is loaded                                                                                                                                                                                                      | String  | To be defined | Any String                                | Optional | Required | Required |
| class       | target_schema       | The schema to which the data are written                                                                                                                                                                                                      | String  | To be defined | Any String                                | Required | Required | Required |
| class       | any                 | Pass any other options you can use within the class                                                                                                                                                                                           | Any     | To be defined | Any                                       | Optional | Optional | Optional |
| load        | mode                | Mode of loading loads either the table from source_schema.table as default or as defined in the custom_load function. In Bronze always a custom_load function is needed meaning the default is custom                                         | String  | default       | default, custom                           | N/A      | Optional | Optional |
| load        | filter              | Allows applying directly filters on the loaded data for predicate pushdown using "custom", otherwise "all" data is loaded. In Bronze always all data is loaded based on the custom_load function                                              | String  | all           | all, custom                               | N/A      | Optional | Optional |
| load        | source_tbl          | Name of the source table,  If provided the source table is used instead of the provided target table to load data.                                                                                                                            | String  | None          | None, Any String                          | N/A      | N/A      | Optional |
| transform   | ignore_defaults     | Ignores executing default transformations as defined in default_transform function. Usually used during debugging                                                                                                                             | Boolean | FALSE         | TRUE, FALSE                               | Optional | Optional | Optional |
| transform   | tbl_transformations | Allows to define custom transformations per table by specifying the table name as key and the function name as value. For the tables it is not defined custom_transform is used.                                                              | Dict    | {}            | Dict[str, str]                            | Optional | Optional | Optional |
| write       | mode                | Defines the mode of writing the data as overwrite, append, replace (define replace filter with function get_replace_condition(), merge (define merge builder with function get_delta_merge_builder(), custom (define custom_write() function) | String  | append        | overwrite, append, replace, merge, custom | Optional | Optional | Optional |
| write       | merge_schema        | If the schema should be automatically envolved/merged                                                                                                                                                                                         | Boolean | FALSE         | TRUE, FALSE                               | Optional | Optional | Optional |

# 4. Functions

The following functions can be overwritten in the Bronze, Silver and Gold class

| Function                                                                                    | Description                                                                                                                                                                                                                          | Bronze                            | Silver                                          |
| ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------- | ----------------------------------------------- |
| custom_load(self, table: str) -> DataFrame                                                  | Function to customize the way or the source data is loaded                                                                                                                                                                           | required                          | required, if load(mode="custom") else ignored   |
| custom_filter(self, sdf: DataFrame, table: str) -> DataFrame                                | Function to apply a custom filter after data loading for predicate pushdown                                                                                                                                                          | N/A                               | required, if load(filter="custom") else ignored |
| custom_transform(self, sdf: DataFrame, table: str) -> DataFrame                             | Can be overwritten to add custom transformations. Only executed if transform() is defined                                                                                                                                            | Optional                          | Optional                                        |
| default_transform(self, sdf: DataFrame, table: str) -> DataFrame                            | Can be overwritten to add default transformations executed after the the custom transformations. Defaults create a timestamp column with the current timestamp of transformations. Only executed if transform(ignore_defaults=False) | Optional                          | Optional                                        |
| get_replace_condition(self, sdf: DataFrame, table: str) -> str                              | Allows you to define the filter used for the replace where overwrite operation                                                                                                                                                       | required if write(mode="replace") | required if write(mode="replace")               |
| get_delta_merge_builder(self, sdf: DataFrame, delta_table: DeltaTable) -> DeltaMergeBuilder | Allows you to define the merge builder for the merge write into delta                                                                                                                                                                | required if write(mode="merge")   | required if write(mode="merge")                 |
| custom_write(self, sdf: DataFrame, table: str) -> None                                      | Allows to define a custom write operation                                                                                                                                                                                            | required if write(mode="custom")  | required if write(mode="custom")                |
