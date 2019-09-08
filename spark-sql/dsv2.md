## Datasource V2

```
public interface DataSourceV2 {}

// Read APIS 
public interface ReadSupport extends DataSourceV2 {
  DataSourceReader createReader(DataSourceOptions options);
}

public interface DataSourceReader {
  StructType readSchema();
  List<InputPartition<InternalRow>> planInputPartitions();
}

```

Spark Data source V2 Api introduced a nice clean Java based interface for custom 
data sources. This allows an easy extension for any new data sources. In order to 
integrate with Spark, we need to provide an implementation of above mentioned methods
 and we will be able to query using Spark. 
 
Let's build our custom csv data source named turtle. We will use the employee table created earlier.
We wil store that table in csv form at /tmp/turtlefs/data/1.csv

### Data source registration
First we will create an registration class. This class inform spark about turtle data source
Spark will load this class and invokes `createReader` method to read data

```
class DefaultSource extends DataSourceV2
  with DataSourceRegister
  with ReadSupport {

  override def createReader(dataSourceOptions: DataSourceOptions): DataSourceReader =
    new TurtleDSReader()

  override def shortName(): String = "turtle"
}
```

### Data source reader
Data source reader class enable spark to read data.

[Source Code](link to github repo)

### Execution
Start a spark shell and load turtle data source jar

1. load data frame using custom datasource 
```
scala> val df = spark.read.format("turtlefs").load()
df: org.apache.spark.sql.DataFrame = [name: string, salary: string]
``` 

Notice that spark have successfully create a dataframe and identified schema of underlying data.
This schema is visible in loaded data frame.

Lets observe query execution plan that spark have built for us
```
scala> df.queryExecution
res0: org.apache.spark.sql.execution.QueryExecution =
== Parsed Logical Plan ==
RelationV2 turtle[name#0, salary#1] (Options: [paths=[]])

== Analyzed Logical Plan ==
name: string, salary: string
RelationV2 turtle[name#0, salary#1] (Options: [paths=[]])

== Optimized Logical Plan ==
RelationV2 turtle[name#0, salary#1] (Options: [paths=[]])

== Physical Plan ==
*(1) Project [name#0, salary#1]
+- *(1) ScanV2 turtle[name#0, salary#1] (Options: [paths=[]])
```

Notice, spark loaded custom data source turtle and generated a `RelationV2`.
Finally, let see content of data frame, this will trigger spark physical plan execution
```
scala> df.show()
+-----+------+
| name|salary|
+-----+------+
|  tom| 80000|
|  bob| 50000|
|alice|180000|
|james|160050|
|steve|250000|
+-----+------+
```