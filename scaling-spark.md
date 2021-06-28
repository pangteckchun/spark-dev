## Scaling Spark ##
Real world spark applications need to run on a spark cluster and this knowledge here describes some these core know-hows.

### Packaging and deploying your application ##3
1. Pre-requisites:  
- Make sure there are no paths to your local filesystem used in our program / scripts. They would be loaded from HDFS or S3.
- Package your application in a jar file
- Use `spark-submit` to execute the driver script:  
`spark-submit -class <fully qualified class name that contains your main function> <path to the jar file containing the class referenced>`

2. Try a cluster on your development machine:
- Download from https://spark.apache.org/downloads.html
- Select a Spark release and pre-built for which version of Hadoop (as the HDFS file system); recommend to select spark 3.x and Hadoop 3.x too.  
Note that Spark 3.0+ is pre-built with Scala 2.12.
- Unzip tar file downloaded onto a path of your choice and there you have it - a standalone Spark environment.
- Jar your scala Spark code using `sbt pacakage` on the command line.

### Packaging with SBT ###
1. Get it from scala-sbt.org

1. Your build tool for Scala; like Maven for Java

1. Can package al of your dependencies into a self-contained jar and makes it easier for your `spark-submit` without the need to specify a ton of jar files dependencies on the command line.

1. Copy your scala source file to where you install `sbt` using the following directory structure.

1. Organize your directory as follows:
- project
    - assembly.sbt --> which contains one line: `addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.14.10")`
- src --> where your Scala source files are, grouped under sub-directories "main/scala"
- build.sbt --> make sure you get the scala ver and spark dependency version correct in the config. State "provided" for Spark library dependencies as you will use the cluster rather than packaging and duplicating spark jar files in your final jar.

5. Run `sbt assembly` from the root folder where src and project folder exists. Jar file will be found in target/[scala version] folder.  

1. Then use `spark-submt [jar file]` (without even the need to specify the main class name).

### Amazon Elastic MapReduce (EMR) cluster ###
1. EMR has setup your Hadoop and Spark optimally out of the box but you should be aware of these parameters if you need to tweak:
- use `--master yarn` to run your driver program in a Hadoop cluster
- use `--num-executors` to set the number of executors in the Hadoop cluster; must specify this if you use YARN and default is only 2
- use `--executor-memory` to allocate memory for each executor
- use `--total-executor-cores` to specify the number of vCores allocation for each executor

2. Make sure you can run locally first on a subset of data before you execute on EMR.

1. EMR sets up a default spark configuration on top of Hadoop's YARN manager. EMR uses m3.xlarge.instances so though not expensive but not cheap either; remember to shut down the clusters when you are done experimenting.

1. Use `repartition(numOfPartitions)`to the last line of your select/join operation, before you convert the dataframe to a dataset. Why?
- Because sometimes you need to give Spark some hints on how to partition your data to optimize execution. For example, self joins are very expensive and Spark wont distribute it on its own;
- If operating on a large data set, operations that will benefit from parition hint and will preserve your partitioning in their results are `join(), cogroup(), groupWith(), leftOuterJoin(), rightOuterJoin(), groupByKey(), reduceByKey(), combineByKey()` and `lookup()`;
- Use `repartition()` on a DataFrame and `paritionBy()`on a RDD.

5. **Best practices**:  
- Partitions  
-- Num of partitions === num of executors or number of cores that fit within your available memory  
-- Partition by 100 is a reasonable place for large operations  
-- Too few partitions wont take advantage of your Spark cluster as some executors will be idle;  
-- Too many will result in shuffling data overhead.  

- Configuration defaults  
-- Configurations precedence: Driver script --> Command line --> cluster config file;  
-- Do not specify config in driver script or command line, as they will overwrite the cluster setup defaults;
-- If you are using AWS EMR the cluster config is already defaulted for you.

- Executors failures  
-- Adjust memory each executor has.

- Always put your script and data files which can be accessed easily on a cluster; if you are using AWS EMR, S3 is a good option.

6. Troubleshooting cluster jobs
- Its a dark art;

- Using master:4040 will allow us to look into call stack exceptions; but you need to look hard which may not easy;  
-- master only fires up when a Spark job is running in the cluster!

- Watch out num of **stages** in the console tab; more stages is bad as it means data needs to be shuffled around;

- Look at the DAG visual of each stage as they may give you hints about how you should structure your driver scripts for the queries;

- Look at **executors** and compare that with the num of cores the cluster is running on; the number should be roughly same as the number of cores you have for the hardware / VM; look into Spark config;

- Getting logs from underlying Hadoop YARN manager is tough as the logs are distributed; use `yarn logs -applicationID [appID]` to collect them;

- Driver script will also log errors; look out for executors failing to issue heartbeats which means they are failing; possibly overworked executors, hence:  
-- Increase each executor memory;  
-- Increase more machine;  
-- Look at data partitioning in driver script.

- Use broadcast variables to share data outside of RDD or DataSets;

- Bundle dependent jar files with `sbt assembly`and remove packages you don't need in the 1st place so that you don't need spend time troubleshooting them; stick with core Spark packages as much as possble.
---


    
