---
layout: post
title: Getting started with Spark in Python/Scala
date: 2015-11-01 19:24:06.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Big Data
tags:
- Spark
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '16434588946'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2015/11/01/getting-started-with-spark-in-pythonscala-part-1/"
---
This is part of a series of introductory posts about Spark, meant to help beginners getting started with it. Hope it helps!
<strong>So what's that funky business people call Spark?</strong>
Essentially <a href="http://spark.apache.org/" target="_blank">Apache Spark</a> is a framework for distributing parallel computational (inherently iterative) work across many nodes in a cluster of servers maintaining high performance and High Availability (HA) while working with commodity servers. It abstracts core complexities that distributed computing activities are subject to (such as resource scheduling, job submission and execution, tracking, message passing between nodes, etc), and provides developers a higher level API - in Java, Python, R and Scala - to <a href="http://spark.apache.org/docs/latest/programming-guide.html" target="_blank">manipulate and work with data</a>.
Why is everyone going crazzy about it? Well, thanks to some cool features (such as in-memory caching, lazy evaluation of some operations, etc), it has been doing pretty well lately in <a href="https://databricks.com/blog/2014/11/05/spark-officially-sets-a-new-record-in-large-scale-sorting.html" target="_blank">performance benchmarks</a>.
In this article I want to avoid getting into more engineering details about Spark, and focus more on developer basic concepts; but it's always useful to have some general idea. Spark requires scala programming language to run, and (as you probably guessed) Java Runtime Environment (JRE) and Java Development Kit (JDK) installed.
A Spark cluster is made up of two main types of processes: a driver program, and worker program (where executors run). To its core, the programming model is that the driver passes functions to be executed on the worker nodes, which eventually return a value to the driver. The worker programs can run either on cluster nodes, or on local threads, and they perform compute operations on data.
<b><img class="alignnone size-full wp-image-991" src="{{ site.baseurl }}/assets/2015/11/spark-cluster-overview.png" alt="spark-cluster-overview" width="596" height="286" />Bootstrapping env</b>
OK, enough chit-chat, lets get our hands dirty. In case the procrastinator side of you is preparing to hit ctrl+w with the pseudo-sophisticated argument that you can't run spark except on a cluster, well I've got bad news.. Spark can be run using the built-in standalone cluster scheduler in the local mode (e.g. driver and executors processes running within the same JVM - single multithreaded instance).
The first thing a Spark program does is to create a SparkContext object (for Scala/Python, JavaSparkContext for Java, and SparkR.init for R), which tells Spark how to use the cluster resources.  Here is an example in Python:
```{python}
from pyspark import SparkConf, SparkContext
conf = (SparkConf()
.setMaster('local[4]')
.setAppName('My first app')
sc = SparkContext(conf = conf)
```
This example shows how to create a context running in local mode (using 4 threads). Lets exemplify in Scala, but now in cluster mode (e.g. connecting to an existing cluster) instead of local (usually, your pc):
```{scala}
import org.apache.spark.SparkContext
import org.apache.spark.SparkConf
val conf = new SparkConf()
.setMaster(&quot;spark://{MASTER_HOST_IP}:{PORT}&quot;)
.setAppName(&quot;My first app&quot;)
val sc = new SparkContext(conf)
```
Last, but not least, to acess Spark from the shell navigate to Spark base directory and in Scala:
```
./bin/spark-shell
Welcome to
____ __
/ __/__ ___ _____/ /__
_\ \/ _ \/ _ `/ __/ '_/
/___/ .__/\_,_/_/ /_/\_\ version 1.5.1
/_/
Using Scala version 2.10.4 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_51)
Type in expressions to have them evaluated.
Type :help for more information.
Spark context available as sc.
SQL context available as sqlContext.
scala&gt;
```
.. Or for PySpark (as of the time of writting of this post, I'm using locally Spark 1.5.1):
```
./bin/pyspark
Welcome to
____ __
/ __/__ ___ _____/ /__
_\ \/ _ \/ _ `/ __/ '_/
/__ / .__/\_,_/_/ /_/\_\ version 1.5.1
/_/
Using Python version 3.4.3 (default, Mar 6 2015 12:06:10)
SparkContext available as sc, HiveContext available as sqlContext.
```
Please note that when you access Spark from the shell (only available for Scala &amp; Python), the sparkContext is bootstrapped automatically for you (in local mode), and in Python and Scala you access this object by referencing "sc" (which, if you look carefully, the shell also instructs you).
<strong><em><a href="https://www.youtube.com/watch?v=010aaw1Ajo0" target="_blank">El Kabong</a></em> time</strong>
The first core concept to learn are Spark's own data collection structure, the Resilient Distributed Datasets (RDDs). RDDs are immutable collections of records that are split and spread across cluster nodes, and saved either in memory or disk. Another interesting property is that RDDs are fault-tolerant: data can be rebuilt even on node failure.
Please give a warm welcome to RDDs, because if you want to "talk parallel lingo", RDDs are your new best friends and currency when dealing with Spark.
<p style="text-align:center;"><a href="https://datacenternotes.files.wordpress.com/2015/11/buzzlight_rdds1.jpg"><img class="alignnone size-medium wp-image-803" src="{{ site.baseurl }}/assets/2015/11/buzzlight_rdds1.jpg?w=300" alt="buzzlight_rdds" width="300" height="300" /></a>
The first option to create an RDD in PySpark is from a list:
```{python}
my_app_name = 'hello spark';
conf = SparkConf().setAppName(my_app_name)
data = range(10) distData = sc.parallelize(data)
```
Please keep in mind that although the list in python is a mutable object, passing it the RDD creation will obviously have no reflection on RDDs immutable nature. Much like tuples in Python, operations you do on RDDs will create a new object. This means that our second option on how to create an RDD is.. from another already existing RDD.
This was obviously a silly example, but the important part to take into consideration is that depending on the number of nodes on your cluster, or if running locally on the number of threads specified initially in SparkContext constructor (..local[4] in our example), Spark will automatically take care of splitting RDD's data among workers (e.g. nodes or threads).
In case you are wondering if you can change per RDD how many partitions/splits are created, the answer is <strong>yes</strong>. Both methods parallelize() and textFile (as we will see next) accept an additional parameter, where you can specify an integer.
Lets look at the same example, but in scala and specifying parallelism:
```{scala}
val data = 0 to 9
val distData = sc.parallelize(data, 4)
```
Besides collections (created in the driver program) and transformations on other RDDs, you can also create RDDs by referencing external sources, such as local file system, distributed file systems (hdfs), cloud storage (S3),  Cassandra, HBase, etc.
Here are some simple examples in Scala:
```{scala}
val localFileDist = sc.textFile(&quot;myfile.txt&quot;)
val hdfsFileDist = sc.textFile(&quot;hdfs://{HOST_IP}:{PORT}/path/to/file&quot;)
val s3FileDist = sc.textFile(&quot;s3n://bucket/...&quot;)
```
Though you can also use the SparkContext textFile() method for grapping files from AWS S3, <a href="http://tech.kinja.com/how-not-to-pull-from-s3-using-apache-spark-1704509219" target="_blank">heads up for some possible issues</a>.
You should also keep in mind that RDDs are lazily evaluated. In other words, only when you actually perform an action (we will get into that in just a few seconds) computation on it. As a matter a fact even on the case of loading a file with the textFile() method, all you are doing is storing a pointer to the file location.
This becomes very clear if you are using the shell, and do for example the following action to return one element from the RDD to the driver:
```{python}
distData.take(1)
```
<strong>Transformations &amp; Actions</strong>
Now, <a href="https://www.youtube.com/watch?v=UUn-f5t53qc" target="_blank">before you get all excited to perform operations on RDDs</a>, there is yet another important aspect to grasp. There are two main types of operations that can be applied on RDDs, namely: actions and transformations.
The key distinctions between the two is that transformations are lazily evaluated and create new RDDs from existing ones, where actions are immediately evaluated and return values to the driver program. In other words, you can think of transformations as just recipes of data, and transformations the real cooking.
Yes, in our previous example, the take() method was indeed an action. Let's continue with our silly example to illustrate this aspect, starting with Python:
```{python}
derivateData = distData.map(lambda x: x**2).filter(lambda x: x%2 == 0).map(lambda x: x+1)
result = derivateData.reduce(lambda x,y: x+y)
result
```
..last, but not least in scala:
```{scala}
val derivateData = distData.map(x =&gt; x*x).filter(x =&gt; x%2 == 0).map(x =&gt; x+1)
val result = derivateData.reduce(_ + _)
result
```
On our first line we are simply performing transformations on our base RDD (distData), namely map and filter operations.
Furthermore, if you are following along with Spark shell, then it will be clear to you that only when you execute the second line - with the action reduce - that Spark will "wake up" and perform the desired transformations on each worker node plus the reduce action, and finally return the result to the driver.
As you can see, you can conveniently chain several transformations on RDDs, which will upon an action be computed in memory fashion on worker nodes. Moreover, all of these operations - both transformations (map +filter) and action (reduce) occurred in memory, without having to store intermediate results on disk. This is one of the main secrets that make benchmarks from Spark agains Hadoop's MapReduce jobs so disproportionately different.
However, after returning values to the driver, these RDDs will be cleared from memory for further computations. Now <a href="https://www.youtube.com/watch?v=nHc288IPFzk" target="_blank">before you panic</a>, the good folks at Berkley anticipated your wish, and there is a method you can call on an RDD to persist it in cache. In our example, to persist the derivateData in memory, we should call .cache() or .persist() method on it before calling any action.
In part 2, I will cover <a href="https://databricks.com/blog/2015/02/17/introducing-dataframes-in-spark-for-large-scale-data-science.html" target="_blank">Spark DataFrames</a>, also equally important.
In the meanwhile, where to go next? Here's a suggestion: <a href="http://ampcamp.berkeley.edu/3/exercises/" target="_blank">Berkley AMPcamp Introduction exercises</a>.
Hope this helped!
