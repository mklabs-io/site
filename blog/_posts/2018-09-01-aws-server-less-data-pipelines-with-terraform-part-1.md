---
layout: post
title: AWS Server-less data pipelines with Terraform to Redshift - Part 1
date: 2018-09-01 14:43:51.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- AWS
- AWS Step Functions
- Data Pipelines
- Lambda Functions
- Redshift
- Terraform
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '21688480964'
  timeline_notification: '1535813032'
  _oembed_7ef71d0584ed273321ae884ca7146173: "{{unknown}}"
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2018/09/01/aws-server-less-data-pipelines-with-terraform-part-1/"
---
This post is the first of sequence of posts focusing on AWS options to setup pipelines in a serverless fashion. The topics that we all cover throughout the whole series are:
<ul>
<li>Part 1: Python Lambda to load data into AWS Redshift datawarehouse</li>
<li><a href="https://datacenternotes.com/2018/09/16/aws-server-less-data-pipelines-with-terraform-part-2/" target="_blank" rel="noopener">Part 2: Terraform setup of Lambda function for automatic trigger</a></li>
<li>Part 3: Example AWS Step function to schedule a cron pipeline with AWS Lambda</li>
</ul>
In this post we lean towards another strategy to setup data pipelines, namely event triggered. That is, rather than being scheduled to execute with a given frequency, our traditional pipeline code is executed immediately triggered by a given event. Our example consists of a demo scenario for immediately and automatically loading data that is stored in S3 into Redshift tutorial.<!--more-->
Note that triggers are not something new exclusive from a Cloud paradigm. Most databases, for example, have triggers that one can setup to execute User Defined Functions. However, the multitude of integrations that AWS Lambda functions have with its own services is quite impressive and useful. In this case, it is AWS S3 that can trigger the execution of the Lambda function.
The typical first use-case that people think for integrating S3 with Lambda functions via triggers is for image processing, such as thumbnails. In this post we are using it for a second use case, namely near real time data imports to AWS Redshift. From the developers perspective, the effort is quite minimal, since AWS does all the lifting in the background.
Without further ado, let us dive into the code.
<strong>The Lambda Function</strong>
The Python code is a very simple Redshift loading code. We will later show in the terraform code, but important to note is that an S3 bucket will be configured to trigger/invoke our Lambda function whenever a new object/key is saved on that S3 bucket. This trigger is active for the whole Bucket, which means any new file copied into the bucket will trigger our Lambda function.
AWS will pass to the lambda function's parameters the Bucket which triggered the function, as well as the new S3 key. Now there is a very important assumption that you need to know, namely that we assume that this key is created with a given prefix, which is the same as the target Redshift table you want it to be imported. For example, if you copy a new file into "kpis/new_file.csv", the lambda will split on "/", and use "kpis" as the table name in Redshift. And there you go, you are ready to see the code.
https://gist.github.com/diogoaurelio/cb289a70e580b087c422471f2d598044
The lambda function code is quite simple. It simply builds a string with the Redshift Copy command, implicitly assuming a CSV file as the underlying format. Why use "Copy command"? This is one the <a href="https://docs.aws.amazon.com/redshift/latest/dg/c_loading-data-best-practices.html" target="_blank" rel="noopener">Redshift best practices main recommendations</a>, and provides two main benefits: speed (data is loaded in parallel) and data storage optimizations.
As a side comment, please do not assume we are recommending using CSV file format. We just chose it due to the easiness that one can take the code in this tutorial and test it live, rather then using more optimized binary formats, such as Avro or Parquet.
<strong>Secrets</strong>
Note that the lambda function is receiving all relevant parameters and secrets via environment variables. So, besides the simple bureaucracies for the copy command, there is just one thing more we would just like to highlight. For Redshift's user password, we are using an additional AWS service, namely <strong>AWS SSM parameter store</strong>. This is a relatively recent service (from 2016 if I am not mistaken) that allows you to store secrets encrypted using KMS key. Now the cool thing about it is that now you can control granularly exactly which instances and roles have access to what.
This means that we need to make sure we provide via IAM policy permissions for this Lambda function to use the specified SSM paramter. We will show how to do this later in the terraform code.
<strong>Last thoughts</strong>
Before we end our first post, we would like to add that there are a lot of things to be improved in this lambda. One suggestion would be to guarantee deduplication, namely that the same data does not get loaded twice into Redshift. One simple way to do this is by using a database, to guarantee synchronization. This would also allow better error handling.
We hope you found this tutorial useful. Also please do not hesitate to contact us at <a href="https://gosmarten.com/" target="_blank" rel="noopener">gosmarten</a>, or me directly - diogo [at] gosmarten [dot] com - if you need any help on your project, be it cloud/devOps/Big Data/ML.
