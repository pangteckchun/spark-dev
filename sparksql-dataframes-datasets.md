## DataFrame intro ##
1. Extended from RDD object and for working with structured data.

2. DataFrame behaves like a typical database with:
- Row objects
- Runs SQL queries
- Has a schema
- Read/write to JSON, Hive and parquet
- Comunicates via JDBC/ODBC, Tableau

## DataSets intro ##
1. DataSets differ from DataFrame primarily it can define the entity types for each row and can be checked at compile time, versus DataFrame schemea which is inferred a runtime; better optimization for DataSets.

1. RDD can be converted to DataSets with `to.DS()`; you can convert from a DataSet to RDD using `rdd()` function and you can perform more low level operations using RDD functions.

1. DataSets are the new hotness:
- More efficent for SQL queries and serialization
- Execution plans are optimized at compile time
- MLib and Spark Streaming moving to DataSets from RDD as their primary API
- Simplifes development with one-liner SQL operations

1. We use SparkSession instead of SparkContext when using DataSets and we will ge the context from the session object. Close the session when done.

## Spark SQL intro ##
1. A Spark module for structued data processing and provides DataFrames as an abstraction for interacting with it.

1. Brings SQL support to Spark ecosystem; Spark SQL conveniently blurs the lines between RDDs and relational tables. Unifying these powerful abstractions makes it easy for developers to intermix SQL commands querying external data with complex analytics.

1. Spark SQL also includes a cost-based optimizer, columnar storage, and code generation to make queries fast. At the same time, it scales to thousands of nodes and multi-hour queries using the Spark engine, which provides full mid-query fault tolerance.

1. Spark SQL exposes a JDBC/ODBC server (assuming Spark is setup with Hive support), for a SQL shell capability:
- Start it with `sbin/start-thriftserver.sh`
- Port 10000 by default
- Connect using `bin/beeline -u jdbc:hive2://localhost:10000
- Done! You now have access to a SQL shell to run Spark SQL; you can create new tables, query existing tables that were cached using `hiveCtx.cacheTable("tablename")`
---

## Working with DataSets ##
1. There are 2 ways of working with DataSets:
- One using `spark.sql ("REGULAR SQL STATEMENT")`
- One using functions provided by DataSets schema object such as `select()`, `filter()`, `groupBy()`

2. Choice depends on personal preference and familiarity with native SQLs and granularity of data manipulations desired.

1. Must always remember to define a schema upfront, e.g. using Scala statement for defining a class such as `case class Person(id:Int, name:String, age:Int, friends:Int)` then during loading of data, call `as[Person]` as the last function to return a DataSet.
- This will allow compile checks to catch data types mismatches;
- Also if a csv has header row, the names must match the column names exactly too avoid compile time errors.

4. Schema (in this case `Person`) will be checked at compile time.

1. Hence use DataSets to work with structured data and it is **not suitable** for unstructured data.

1. If a CSV has no specified header when loaded, when defining the case class, use column name `value: String` as the default.

1. Compared to RDDs, DataSets are cleaner, esp in the area of data loading. In RDD we have to parse every line of our CSV data to obtain the columns in a specific handcrafted function; whereas in DataSets, we simple load the CSV and use the defined schema class to load all the fields, with compile time checks. Also with RDD, in order to obtain counts for a specific grouping, we need to resort to techniques such as mapping a value of '1' with key being the target entity, then using `reduceByKey` to sum all the counts based on the keys.

1. For the DataSet functions such as `sort()`, `split()`, `filter()` etc use this syntax of example: `sort(`**$**"[column-name]"`)`, to reference the specific column directly.
- take note of the **$** sign followed by the column name in double quotes.

7. Equality operator in DataSet operators are of the syntax `=!=` (not equals) and `===` (equals)

1. For data which may not have schema or structure upfront, using an RDD to load the data then convert into DataSet using `toDS()` for analysis is a common hybrid approach.

1. For structured data without headers to guide us, we can also enforce schema compile time check by doing:
- Still define a case class as usual:
```
  case class CustomerSpend(cust_id:Int, amount_spent:Float)
```
- Define a `org.apache.spark.sql.types.StructType` to define the runtime schema:
```
    val CustomerSpendSchema = new StructType()
      .add("cust_id", IntegerType, nullable=true)
      .add("amount_spent", FloatType, nullable=true)
```
- Apply the same code when creating a spark session and apply the schema on reading from the loaded data source:
```
import sparkSess.implicits._
    val ds = sparkSess.read.schema(CustomerSpendSchema).csv("data/customer-orders.csv").as[CustomerSpend]
```
**Pro-tip:**  
- You can drop columns which are not needed too at this stage and do not need to import every column in the data source (e.g. CSVs);
- Case class column names must match your StructType column names; else runtime exception will occur.
---

## Advanced DataSets ##
1. One common problem - distributed sources of data, e.g. movieID in one source and movieDetails in another source.
- Joining distributed DataSets across a cluster has performance implications;
- Use **Broadcast Variables** to forward mapped data variables ONCE to each executor in the cluster as required;
- Use `SparkContext.broadcast()` to ship off to the cluster when required as a dictionary / map object;
- To get contents or value from the dictionary or map object, call `value(key)` on the map object;
- We can use **Broadcast Variables** in RDDs, DataSets or any user-defined funcions (`udf`):
```
import org.apache.spark.sql.functions.{col, udf}
...

  // We start by declaring an "anonymous function" in Scala
    val lookupName : Int => String = (movieID:Int)=>{
      nameDict.value(movieID)
    }

    // Then wrap it with a udf
    val lookupNameUDF = udf(lookupName)
```

2. DataFrame `agg()` function takes in different aggregeate methods such as `avg()`, `min()`, `max()`, `count()`, `sum()` etc of sql like behaviors to perform the intended functions. The `agg()` function is often preceded by `groupBy()` function as follows:
```
// Select the age of the oldest employee and highest expenses for each department - grouped by

import org.apache.spark.sql.functions

df.groupBy("department").agg(max("age"), sum("expense"))
```
Hence the different aggregate methods can operate independently of each other.  

3. Joins are expensive cluster operations so always filter down to smallest datasets before you join with other datasets together

1. Use an `Accumulator` to stored shared states across different executors in a Spark cluster, for example, using Long integer type accumulator to keep track of hits:
```
var hitCounter: Option[LongAccumulator] = None

...

// Initializing the hit counter accumulator
hitCounter = sparkContext.longAccumlator("Hit Counter")
```

5. Use Datasets to solve problems which can be tackled through SQL manipulations and has SQL optimization potentials; use RDDs to tackle non SQL solvable domains (such as Breadth-First search, BFS, for finding degress of separation between individuals). The speed of RDDs is 3 times faster compared to DataSets for BFS as an example.
SQL solvable domains such as finding similarities of movie ratings across a group users which might warrant some self.join operations are very suitable for DataSets and this will outperform RDDs.

1. Any time, if you will perform more than one action on a dataset, you should cache it. Use `.cache()` or `.persist()`.
`.persist()` optionally lets you cache to disk instead of memory in case a node fails.
---