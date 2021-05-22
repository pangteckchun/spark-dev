
## Resilient Distributed Datasets (RDDs) ##
1. First API that Spark came out with in Spark v1 and though superseded still needed from time to time.

1. Still core and legacy Spark implementation may still use RDD. Newer approaches use DataFrames and DataSets.

1. RDD - a bunch of **data** arranged into rows that is **distributed** across different computers, **resiliently** managed by Spark to ensure they are being run.

1. Use the `SparkContext`(sc) object as the starting point of creating a RDD, which will care of the resiliency and distribution for you.

1. Create a sc object using these methods:
- using a `List` object
- load from a file storage, s3 or HDFS - sc.textFile ("file:///...") or s3n:// or hdfs://
- using Hive - `hiveCtx = HiveContext(sc)`
- from JDBC, Cassandra, HBase, ElasticSearch, any structued format sources - JSON, CSV and compressed formats

6. Many RDD methods accept a *function* as a parameter.

1. To perform useful stuff on your RDD, you need to perform certain actions on it, e.g. `collect`, `reduce`, `countByValue` etc. Nothing will happen in your driver script or program until an action is called!

1. High level steps to get a RDD to work are:
- import the right packages `org.apache.spark._`
- create the `SparkContext`
- create the RDD (in the example for **RatingsCounter.scala**, it loaded from a file)
- perform an necessary extract and transform from the RDD, e.g. map() to obtain the necessary intermediate RDD
- perform the necessary RDD actions, like `collect()`,`reduce()`, `count()` etc. NOTE: this action also makes us leave the Spark land and obtain the results back into our driver script / program; this step NO LONGER returns a RDD
- sort or print etc on the result as necessary

9. Under the hood, Spark looks at the code to be executed and derives an execution plan: parallelize the map() operations in its own executor thread; the countByValue() is prob hardly to parallelize hence this is in his own executor thread. The sort and print is in its own executor thread.

1. RDD can also be maps and not just confined to list. And there are associated methods / actions with manipulating RDD maps.

1. Often, we use this technique: `rdd.mapValues(x => (x,1))` to assign a value of 1 to every key. This allows us to sum the totals for a given key later easily using the reduceByKey() function.
- for a given key-value tuple like (33, 385), `mapValues(x => (x, 1))` gives (33, (385, 1))
---