---
layout: post
title: Build a Data Pipeline with AWS Athena and Airflow (part 1)
date: 2018-06-07 18:01:38.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Airflow
- Athena
- AWS
- Big Data
- Data Pipelines
- Database
- Datawarehouse
- python
tags: []
meta:
  timeline_notification: '1528394502'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '18716129134'
  _thumbnail_id: '4046'
author:
  login: jrcferrao
  email: joao@mklabs.io
  display_name: João Ferrão
  first_name: ''
  last_name: ''
permalink: "/2018/06/07/build-a-data-pipeline-with-aws-athena-and-airflow-part-1/"
---
In this post, I build up on the knowledge shared in the post <a href="http://datacenternotes.com/2018/05/14/airflow-create-and-manage-data-pipelines-easily/">for creating Data Pipelines on Airflow</a> and introduce new technologies that help in the Extraction part of the process with cost and performance in mind. I'll go through the options available and then introduce to a specific solution using AWS Athena. First we'll establish the dataset and organize our data in S3 Buckets. Afterwards, you'll learn how to make it so that this information is queryable through AWS Athena, while making sure it is updated daily.
Data dump files of not so structured data are a common byproduct of Data Pipelines that include extraction. dumps of not-so-structured data. This happens by design: business-wise and as Data Engineers, <em>it's never too much data. </em>From an investment stand point, object-relational database systems can become increasingly costly to keep, especially if we aim at keeping performance while the data grows.
Having this said, this is not a new problem. Both Apache and Facebook have developed open source software that is extremely efficient in dealing with extreme amounts of data. While such softwares are written in Java, they maintain an abstracted interface to the data that relies on traditional SQL language to query data that is stored on filesystem storage, such as S3 for our example and in a wide range of different formats from HFDS to CSV.
Today we have many options to tackle this problem and I'm going to go through on how to welcome this problem in today's serverless world with <strong>AWS Athena</strong>. For this we need to quickly rewind back in time and go through the technology <!--more-->that marked a significant shift in the technology that helps us solve such problems while integrating these solutions with a popular Data Pipeline platform such as Airflow.
<h2>Popular solutions</h2>
<strong>Apache Hive</strong> was the first of the family,  it works with <strong>Apache Hadoop</strong> and relies heavily on MapReduce to the work behind the scenes. The SQL queries are translated to <strong>MapReduce</strong> stages and intermediate results are stored on filesystems, including S3 buckets. This made Hive extremely appealing as it was much faster then traditional alternatives and was also very reliable by taking a batch-approach to the processing of jobs - you know jobs are bound to fail at some point of development and production.
Apache Hive depends on something called the Hive Metastore. As the name suggests, this is a table that keeps the metadata of where the information is stored in your filesystem and the structure it has - it's essentially a pointer to data. We'll come back to this, as it is an essential component of this introduction, but for now it's important to know of its existence.
Fast forward to 2013, Facebook releases <strong>Presto</strong>, also for Apache Hadoop, which in turn relies on in memory processing to achieve the same goal resulting in much faster query execution and also with connectors to S3. This also meant, however, that your infrastructure needs to be specifically tuned to the jobs you are running so they don't run of memory constantly and, thus, giving you the despair of failed jobs. Of course this is a problem only when astronomical amounts of data are in place for being processed and still, Facebook was using it daily to scan around 1 Petabyte of data everyday (<em>was</em>, as the text back a Presto's official welcome excerpt looks like was last updated in 2016). Apart from this, Presto also relies on a Hive Metastore to understand how and where the data is stored.
There are several more differences between both but if I had <em>lightly </em>boil it down to the use cases that distinguishes both, I could say: you use Hive for heavy processing that has a flexible time schedule and Presto when you need more frequent interaction with the data.
<h2>AWS Athena</h2>
This context of differences (batch write to the filesystem vs. in memory) is extremely relevant if you are a IaaS provider with the capacity to direct <em>virtually</em> any amount of power at a designated service they deem worth. Combine this with the popularity of their storage service of S3 and the speed of Presto, you get the AWS Athena: a serverless service allows for queries to data stored in S3 buckets in several different formats, including CSV, JSON, ORC, Avro, and Parquet.
Moreover, the pricing strategy adopted by AWS means you only by per data scanned, letting you off he hook in regards to computational power. Not only you bypass the head aches of configuration and setup, you also bypass the costs of having a full blown EMR instance for running data intensive queries.
<h2>Establish a Dataset</h2>
For this tutorial, I there is a daily dump of .CSV files into an S3 bucket called <strong>s3://data</strong>. The first order of business is making sure our data will be organised in some way. A generic way of approaching this, which applies to most time-related data, is to organize it in a folder tree separated by Year, Month and Day.
Presto let's you query your data also based on this structure, transforming the directory tree in fields you can use in you SQL filters. For this to work, you specifically need to use a keyword=value relation in the literal name of each directory such as <strong>s3://data/year=2018/month=6/day=1/dump.csv. </strong>
For the sake of simplicity, I'll use this dataset of <a href="https://github.com/plotly/datasets/blob/master/2014_us_cities.csv">US Cities Population</a> in 2014, in this case, the file should be store in an S3 path like <strong>s3://data/year=2014/dump.csv</strong>. It really doesn't matter the name of the file. Athena will look for all of the formats you define at the Hive Metastore table level.
<blockquote><span style="color:#339966;"><strong>Top Tip</strong>:</span> If you go through the AWS Athena tutorial you notice that you could just use the base directory, e.g. <strong>s3://data</strong> and run a manual query for Athena to scan the files inside that directory tree. You would conclude you could do this the first time and then every time there is a new dump file in the file system. While this is true it is but highly disregarded<strong>. </strong>There are two main reasons: the first is that you will have Athena to crawl/refresh the Hive Metastore completely  every time you add a new file (e.g. daily with a manual command) which results in a scan. The second reason is that although AWS doesn't charge for partition detection, the process often times out and they <strong>do charge for S3 GET requests. </strong>And the direct consequence of calling the command equivalent to "refresh all" does generate GET Requests to S3 as clarified in this post on the<strong> <a href="https://forums.aws.amazon.com/message.jspa?messageID=773528">AWS forum</a></strong>. This is what we'll use Airflow for in the next tutorial as a Data Pipeline.</blockquote>
<h2>Create table</h2>
Now that your data is organised, head out AWS Athena to the query section and select the <strong>sampledb</strong> which is where we'll create our very first Hive Metastore table for this tutorial.
In line with our previous comment, we'll create the table pointing at the root folder <strong>but will add the file location (or partition as Hive will call it) manually for each file or set of files.</strong>
<h4>Creating an empty table:</h4>
Note how we define the specifics to our file in the SERDEPROPERTIES (specific to this SerDe). Also, note that we are creating an external table, which means that deleting this table at a later stage will not delete the underlying data.
```{sql}
CREATE EXTERNAL TABLE `sampledb.us_cities_pop`(
  `name` string,
  `pop` bigint,
  `lat` decimal,
  `long` decimal )
PARTITIONED BY (
  `year` int )
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
   'separatorChar' = ',',
   'quoteChar' = '\"',
   'escapeChar' = '\\' )
LOCATION's3://data/';
```
We could make the first query using the traditional
```{sql}SELECT * FROM sampledb.us_cities_pop;```
But the query will come back empty since we haven't added any partition or have explicitly told Athena to scan for files. Remember, you will be paying based on the amount of data scanned.
You could also check this by running the command:
```{sql}SHOW PARTITIONS sampledb.us_cities_pop;```
<h4>Let add the 2014 partition</h4>
Adding partitions is relatively simple, for the purposes of this simple directory tree and dataset:
```{sql}
ALTER TABLE sampledb.us_cities_pop
ADD IF NOT EXISTS PARTITION (
  year = 2014 )
LOCATION = 's3://data/year=2014/dump.csv;'
```
Now if you run the previous code to show partitions you'd see this very same one. The objective is that you would be doing this for every new dump file, e.g. every day. Adding manually a partition.
In the next post we'll let you know how to integrate Athena into an Airflow pipeline and make sure you add partitions to the Hive Metastore table with a specific routine without having to be charged just for updating the parititons. This way you make sure you only have costs when effectively running a query that aims and giving answers back to you.
Part 2 for this tutorial is now available <a href="http://datacenternotes.com/2018/07/21/build-a-data-pipeline-with-aws-athena-and-airflow-part-2/">here.</a>
<h1></h1>
&nbsp;
