---
layout: post
title: 'Spark 2.0: From quasi to proper-streaming?'
date: 2016-09-04 18:31:57.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Big Data
- Hadoop
- Spark
tags: []
meta:
  _publicize_job_id: '26482613417'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2016/09/04/spark-2-0-from-quasi-to-proper-streaming/"
---
This post attempts to follow the relatively recent new Spark release - <a href="https://spark.apache.org/releases/spark-release-2-0-0.html" target="_blank" rel="noopener">Spark 2.0</a> - and understand the differences regarding Streaming applications. Why is streaming in particular?, you may ask. Well, Streaming is the ideal scenario most companies would like to use, and the competition landscape is definitely heating up, specially with <a href="https://flink.apache.org/" target="_blank" rel="noopener">Apache Flink</a> and Google's <a href="http://incubator.apache.org/projects/beam.html" target="_blank" rel="noopener">Apache Beam</a>.
<strong>Why is streaming so difficult</strong>
There are three main problems when it comes to building real time applications based on streaming data:
<ul>
<li><span style="text-decoration:underline;">Data consistency</span>:  due to the distributed architecture nature it is possible that at any given point in time some events have been processed in some nodes and <em>not</em>  in other nodes, even though these events might actually have occurred before than others. For example, you may have a Logout event without a Login event, close app event without open app, etc.</li>
<li><span style="text-decoration:underline;">Fault tolerance (FT)</span>: related to the first problem, on node failure processing engines may try to reprocess an event that had actually already been processed by the failed node, leading to duplicated records and/or inaccurate metrics</li>
<li><span style="text-decoration:underline;">Dealing with late data/out-of-order data</span>: either because you are reading from a message bus system such as Kafka or Kinesis, or simply because mobile app might only be able to sync data hours later, developers simply must tackle this challenge in application code.</li>
</ul>
See this <a href="https://databricks.com/blog/2016/07/28/structured-streaming-in-apache-spark.html?utm_campaign=Databricks+newsletter&amp;utm_source=hs_email&amp;utm_medium=email&amp;utm_content=32442368&amp;_hsenc=p2ANqtz-931r_2I49QFGSgPZAaegXtkjBD7_35CsZVlHRxIDkiDdiD5vfIVlFPNsKQGAhX4mvNbN3uI-YAU9ArIOk6_eJCajD4Ww&amp;_hsmi=32442368" target="_blank" rel="noopener">post</a> for an excellent detailed explanation.<!--more-->
<strong>Rewind to Spark - 1.6.2</strong>
Unlike Apache Flink, where batch jobs are the exception and Streaming the default, Apache Spark uses the exact opposite logic: Streaming is the exception of batch. To make this more clear, the way Spark Streaming works is by <a href="https://spark.apache.org/docs/1.6.2/streaming-programming-guide.html" target="_blank" rel="noopener">dividing live data streams of data into batches (sometimes called microbatches). </a>The continuous streams are represented as a high level abstraction called discretized stream or DStream, which internally is represented as a sequence of RDDs.
Some things in common between RDDs and DStreams are that DStreams are executed lazily by the <span style="text-decoration:underline;"><a href="https://spark.apache.org/docs/1.6.2/streaming-programming-guide.html#output-operations-on-dstreams" target="_blank" rel="noopener">output operations</a></span> - the equivalent to Actions in RDDs. For example, in the classical word count example, the print method would be the output operation.
https://gist.github.com/diogoaurelio/e948547ed23001281805b4463bc05872
Nonetheless, even though the main abstraction to work around was DStreams on top of RDDs, it was already possible to work with <a href="https://spark.apache.org/docs/1.6.2/streaming-programming-guide.html#dataframe-and-sql-operations" target="_blank" rel="noopener">Dataframes (and thus SQL) on streaming data before</a>.
&nbsp;
<strong>Changes in Spark 2.0 - introducing "Structured Streaming"</strong>
So  the new kid on the block is structured streaming, a feature still in alpha for Spark 2.0. It's marketing slogan: "<em>Fast, fault-tolerant, exactly-once stateful stream processing without having to reason about streaming</em>". In other words, this is an attempt to simplify the life of all developers, struggling with the three main general problems in streaming, while bringing a streaming API for DataFrames and DataSets.
Note that now the streaming API has been unified with Batch API, which means that you should expect (almost always) to use same method calls exactly as you would in normal batch API. Of course, some minor exceptions, such as reading from file and writing to file,  require extra method call. Here is a copy paste from the examples shown at Spark Submit in the "<a href="https://www.youtube.com/watch?v=rl8dIzTpxrI&amp;feature=youtu.be" target="_blank" rel="noopener">A deep dive into structured streaming</a>" presentation (highly recommendable).
Syntax for batch more:
```{scala}
input = spark.read.format("json").load("source-path")
result = input.select("device", "signal").where("signal &gt; 15")
result.write.format("parquet").save("dest-path")
```
Now for structured streaming:
```{scala}
input = spark.read.format("json").stream("source-path")
result = input.select("device", "signal").where("signal &gt; 15")
result.write.format("parquet").startStream("dest-path")
```
In case you didn't notice the difference (that's good in this case), what changed is instead of load() of a file, you continuously stream(), and same logic to write with startStream(). Pretty cool, right?
Also note that the base premise of  spark's streaming discussed before still stays; that is, micro-batching still stays.
<strong>Repeated Queries on input table</strong>
So besides unifying the API, where's the magic sauce? The main concept is this new abstraction where streaming data is viewed as if it were much like a table in a relational DB. In other words new incoming data is treated as a new row inserted on an <em>unbounded input table</em>.
<strong>The developer has three tasks in the middle of this</strong> <strong>whole streaming party</strong>.
<ol>
<li>One is to define a<span style="text-decoration:underline;"> query that acts repeatedly</span> on this input table, and produces a new table called <span style="text-decoration:underline;">result table</span> that will be written to an <span style="text-decoration:underline;">output sink</span>. This query will be analyzed under the hood by spark (as explained bellow) by the query planner into an execution plan, and Spark will figure out on its own what needs to be maintained in the input table to update the result table.</li>
<li>The second is to specify the <span style="text-decoration:underline;">triggers</span> that control when to update the results.</li>
<li>Define an <span style="text-decoration:underline;">output mode</span>, so that spark knows how it should write to an external system (such as S3, HDFS or database). Three alternatives: <span style="text-decoration:underline;">append</span> (only new rows of result table will be written), <span style="text-decoration:underline;">complete</span> (full result table will be flushed), <span style="text-decoration:underline;">update</span> (only rows that were updated since last trigger will be changed)</li>
</ol>
<strong>Structured Streaming under the hood</strong>
Under the hood (and when I say "under the hood" I mean what is done for you that you don't need implement in your code - but yes, you should nonetheless care about it) structured streaming are possible due to some considerable changes. The first is that the Query planner (the same for batch mode) generates a continuous series of multiple <strong>incremental</strong> execution plans <strong>for each</strong> processing of the next chunk of streaming data (for example, when pooling from Kafka this would mean the new range of offsets).
Now this can get particularly tricky because each execution plan needs to take into consideration the previous aggregations that were held by previous executions. So these continuous aggregations' state is maintained internally as an in-memory stated, and backed by  Write Ahead Log (WAL) in the file system (for Fault Tolerance). So yes, the Query Planner is FT by storing tracks of offsets of each execution into a distributed file system such as HDFS.
Moreover on FT, the sinks are also idempotent, which means that one should expect that they handle re-executions to avoid double committing the output.
Finally, before you get all excited, I do want to emphasize that at the moment, this is still in alpha mode, and is explicitly not recommended for production environments.
Not enough?, bring on more resources:
<ul>
<li><a href="http://hortonworks.com/hadoop-tutorial/introduction-spark-streaming/" target="_blank" rel="noopener">HortonWorks basic tutorial</a></li>
<li>Databricks blog post <a href="https://databricks.com/blog/2016/07/28/continuous-applications-evolving-streaming-in-apache-spark-2-0.html" target="_blank" rel="noopener">1</a> and <a href="https://databricks.com/blog/2016/07/28/structured-streaming-in-apache-spark.html" target="_blank" rel="noopener">2</a></li>
<li>Presentation "<a href="https://spark-summit.org/2016/events/a-deep-dive-into-structured-streaming/" target="_blank" rel="noopener">A deep dive into Structured Streaming</a>"</li>
</ul>
&nbsp;
