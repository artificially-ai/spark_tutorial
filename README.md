# Spark for Dummies [like me]

To keep learning and improving my skills, I decided to dig deep into Scala and Spark. The former has already been covered in my *scala_tutorial* Github [repository](https://github.com/ekholabs/scala_tutorial) and the latter is live now.

#### Differences compared to Hadoop’s MapReduce?

* Operations are intuitive, instead of expressed as a chain of map/reduce calls.
* Interactive (REPL) environment.
* In-memory, instead of writing to disk after every iteration.
* Stream Processing, instead of batch.

#### How does it do it?

* It uses Resilient Distributed Datasets, or RDDs:
  * RDDs are the main programming abstraction in Spark.
  * RDDs are in-memory objects and all data is processed using them.
  * RDDs are fault-tolerant.

#### How to overcome complexities?

1. Distributed data access a cluster
2. Fault tolerance (if in-memory data is lost)
3. Efficiently processing billions of rows of data

Spark’s RDDs takes care of the complexities above.

#### Built-in Libraries

* Spark Core is a computing engine:
  * It contains the basic functionality of Spark.
  * It provides an API for working with RDDs.

In order to be useful, Spark needs additional components, like a Storage System and a Cluster Manager.

* Storage System
  * HDFS, Hive, HBase, Cassandra, etc.
* Cluster Manager
  * Schedules tasks and manages resources across the cluster.
  * Spark comes with a built-in Cluster Manager, but you can plug others in:
    * Apache Mesos, Hadoop, Hadoop’s YARN.

Spark comes with additional packages that makes it truly general-purpose:

* Spark SQL
  * Provides a SQL interface for Spark.
* Spark Streaming
  * Enables processing stream of data in (nearly) real time.
  * Can process, for instance, las for reporting, monitoring and react to them in real time.
* MLlib
  * Provides a built-in Machine Learning functionality in Spark.
  * Contains built-in methods for classification, regression and clustering.
  * It takes care of running the algorithms across a cluster.
  * ML algorithms are iterative, demanding passes over the same data many times. Thanks to RDDs, those passes are done without data having to be written to disk
* GraphX
  * It’s a library for graph algorithms.
  * Represent and perform computations across graph datasets (e.g. social media, linked webpages, etc.).

#### Installing Spark

To install Spark on your MacBook you need Home Brew. After installing Home Brew, do the following:

* brew install apache-spark
  * This will install the latest Apache Spark, which is version 2.3.0.

To be able to use Jupyter Notebooks as development environment, we need Apache Spark 1.6.3, which uses Scala 2.0.x. So, unless you want to do this, follow the steps below. Otherwise, just use IntelliJ IDEA or the spark-sell.

Thanks to the Apache Tree project, we can run Scala code within Jupyter. Install Toree and the Jupyter plugin with the following commands:

* ```pip install toree```
* Download Apache Spark 1.6.3 and unpack it somewhere
  * https://spark.apache.org/downloads.html
* ```jupyter toree install --spark_home  apache-spark-1.6.3-PATH```
* ```jupyter-notebook```

#### Spark Shell

The Spark Shell (REPL) is a convenient too to try out some code to get familiar with Spark. By default, the Spark Shell doesn’t run with distributed mode. To change this, you have to inform how many threads you want Spark to use. The amount of threads is related to the amount of cores in your machine.

* ```spark-shell —master local[2]```
    * 2 => number of threads.

This configuration can also be added to the Jupyter kernel:

* ```vim /usr/local/share/jupyter/kernels/apache_toree_scala/kernel.json```
* Make the property ```“__TOREE_SPARK_OPTS__”: “”``` look like this:
    * ```"__TOREE_SPARK_OPTS__": "--master=local[2]"```

To make the editor use Scala highlight style, add the following line to the ```kennel.json``` configuration file:

* ```“codemirror_mode”: “scala”```

#### Data Analysis with Spark

This tutorial uses the USA Bureau of Transportation Statistics data. To download the same dataset, please use the link below:

* [Bureau of Transportation Statistics]  (https://www.transtats.bts.gov/Tables.asp?DB_ID=110&DB_Name=Air%20Carrier%20Statistics%20%28Form%2041%20Traffic%29-%20%20U.S.%20Carriers&DB_Short_Name=Air%20Carriers)

All the code is available in the spark_tutorial notebook. Hence, I will avoid typing code here, but only make a few comments on how to proceed to perform certain tasks.

* Loading a dataset
    * You can inform the local path of the file or another destiny (e.g. HDFS path):
      ```scala
      val flightsCSV = "datasets/usa_carrier_only.csv"
      ```
    * Spark Shell starts a SparkContext object, which can be used to load the dataset into memory. The context is responsible for naming spark communicate with the Cluster Manager.
      ```scala
      val flightsDS = sc.textFile(flightsCSV)
      ```
    * That will make flightsDS a Resilient Distributed Dataset, our first RDD.

For more details about the operations executed on top of this dataset, please refer to the notebook ```spark_tutorial```.

#### Resilient Distributed Datasets

RDDs are kept in memory, and distributed amongst nodes. How does it work? The answer for this question is **Partitions**.

To understand how it is done, let’s have a look at HDFS (Hadoop Distributed File System):

* Hadoop keeps the data in disk.
* The data is distributed in block sizes in disks from different nodes.
* The data is also replicated amongst the nodes so if a node dies, the data can still be accessible.
* The Node Name keeps track of where certain data is and where it is replicated.

When Spark uses HDFS, it will automatically distributed the data in memory in the nodes where the data is stored in disk, that will avoid network latency. When a node doesn’t have enough memory, then another one is chosen.

#### RDDs Lineage

When created, a RDD holds metadata about the transformation that was used to create it and also about the parent RDD, the one the transformation was executed upon. For example, a RDD created out of a ```filter``` operation will hold in its metadata that that operation caused its creation. Knowing about its lineage allows RDDs to trace back to its source, the file that was used to create the RDD.

Lineage is responsible for two important concepts:

1. It makes RDDs resilient.
2. It makes lazy evaluation possible.

But how does lineage make those things possible?

#### Resilience

In distributed computing, resilience is a very important concept because data can, and will be, lost. The way Hadoop fixes this, for instance, is by a) replicating the data in the HDFS and b) by writing data to disk.

So, how does RDDs do it then? How can in-memory based storage provide fault-tolerance? It uses **Lineage**. Since the RDDs know about their lineage, they can be reconstructed from source. Let’s say that data in node A is gone, but we still have data in node B, which was created by partitioning the RDD. From node B the original RDD can be reconstructed since it holds metadata about how it came to “life”, or, where its original file is stored.

#### Lazy Evaluation

When created, a RDD contains only metadata about how it was crated, which transformation was used. They are not materialised, which means they hold no data at all. The lazy evaluation is very efficient for computation of huge amounts of data.

From the example in the notebook, we loaded the RDD with the ```textFile``` function (RDD one), then we filtered the header (RDD two) and the last call was to split the lines by comma (RDD three). Three RDDs were created, but nothing happened until we called ```take(10)``` - for the first 10 rows. So, the RDDs are only materialised by Spark when an Action is called upon them.

#### Transformations on Rows

In the notebook, we used the following transformations:

* filter
* map
* flatMap

Exception for flatMap, the other two seem quite straight-forward to understand. The **filter** takes a function that will be applied on each row of the RDD, filtering it depending on the boolean evaluation depicted by the function. Concerning **map**, it takes a function that will convert, or transform, each row into a new object. In the notebook we used map to split rows by comma.

Each row in the RDD was a comma-separated value and we converted it to a collection of values. If you recall, after calling take(10) we got as output an Array of Array.

So, how does flatMap comes into the game? It flattens things. If we would have used flatMap instead, we would end up with just one Array. You can see it clearly in the notebook example under Transformation. You can also try use flatMap in the newFlightsDS and see how it behaves.

#### Transformation on RDDs

Besides the ones above, there are also transformations that require two RDDs. For example:

* union
    * It creates a RDD that is a union of two RDDs.
* intersection
    * It creates a RDD that is an intersection of two RDDs.
* subtract
    * It removes the content of one RDD from the other.
* cartesian
    * It return an RDD which contains the Cartesian product of the two, tuples of pairs.

To see more Transformations and Actions, check the Spark documentation: https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.UnionRDD

And remember, those operations are lazy evaluated.

#### Actions

We will look into actions with more detail in the notebook, but here is a list of some commonly used ones:

* collect
* first
* take
* reduce
* aggregate
* count
* countByValue
