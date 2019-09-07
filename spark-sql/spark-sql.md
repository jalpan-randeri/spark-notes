## Spark SQL

## Spark SQL Pipeline
Spark SQL pipeline mainly categories into 4 parts
1. [Query Parsing](#query-parsing)
2. [Logical Planning](#logical-planning)
3. [Query Optimization](#query-optimization)
4. [Physical Planning & Execution](#physical-planning-&-execution)

![Spark SQL Query Planning](./assets/query-execution.png)

Spark perform various operations to convert given SQL query into RDD (Resilient Distributed Dataset).

Lets prepare our table. We will query against this table

| name   |  salary  | age |
|------  |--------  |---- |
| tom    | 80,000   | 23  |
| bob    | 50,000   | 23  |
| alice  | 180,000  | 39  |
| james  | 160,050  | 31  | 
| steve  | 250,000  | 45  |

Lets say, we want to find all people who earns more than 100,000.
This translate to following query.
```
select * from people where salary > 100,000
```
Lets run this query through spark and will observe what spark does on each step.

Before spark perform any operation it simply convert sql code into data frame.

### Query Parsing
During this stage, spark will parse the query and create a tree like structure for us. Spark uses Catalyst parser
 internally. This enable spark to validate the syntax of query and extract various part of query such projected
  fields, table, database, filter conditions and joins. This way spark generates Unresolved Plan
```dtd
add query parsing tree image here
```

### Logical Planning
In this phase, spark will read parsed query and with the help of Catalog it resolve attributes of the query such as
identifying database of required table in query. All the projected field types and filter condition types. During
this phase spark also determines how to read certain underlying files and their location. If this table is managed
by external catalog such as Hive then spark will determine all these information from Hive metastore. 
(Note: AWS Glue is an example of external Hive meta store) This metastore persists metadata about tables.
Spark uses Catalyst Analyzer during this phase. Its job is to simply resolve names of attributes in SQL query using
information present in table. In simple  At the end of this stage spark generates resolved logical plan. 

```
Add logical plan info here
```

### Query Optimization
During this phase, spark will applies set of rules and generates optimized plan. Analyzed Logical plan is represented
as tree internally in spark. Spark Optimizer read through tree and run a batches of rules. Each rule will try to
optimize and generate a new tree. At the end we will have optimized logical plan.

```dtd
Add logical plan info here
``` 

### Physical Planning & Execution
Spark now reads optimized logical plan and generate set of physical plans. [SparkStrategies]()
Physical plan actually translate the requested operations into RDD operations and execute them on executors. 
After aset of physical plans are generate spark evaluate them based on cost and pick the cheapest one. These cost are
calculated using various statistics information available from meta store. Some of these statistics include total
size of table, max, min value of columns, number of rows etc. Spark uses these information to improvise runtime.
For example if our query is performing a join on two tables A and B.
Size of table A is 10 mb and table B is 1 TB. Since we can easily store table A in memory and broadcast it to all
executors. This will speed up our query execution due to map side join over sort merge joins.
Once cheapest Physical plan is identified spark begins the execution and generate output.

```dtd
Add physical plan info here
```


Now we have high level idea and what happens under the hood on each stage. 
Lets dive little deeper into physical plan execution

## Physical plan execution
During the physical plan execution spark perform 2 basic operations. 
1. File System Listing (Find all the candidate files where data is located)
2. Record Reading (Read all files and generate output)

Spark provide 3 different ways of physical plan execution
1. [Hive Mode (a.k.a. File Input Format API)](../spark-sql/hive.md)
2. [Datasource Mode (a.k.a Datasource v1 API)](../spark-sql/datasourcev1.md)
3. [Datasource V2 Mode](../spark-sql/dsv2.md)

We will look into each of this mode 