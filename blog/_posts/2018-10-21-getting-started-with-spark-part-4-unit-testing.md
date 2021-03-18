---
layout: post
title: Getting Started with Spark (part 4) -  Unit Testing
date: 2018-10-21 15:17:11.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Spark
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '23429040190'
  timeline_notification: '1540135031'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2018/10/21/getting-started-with-spark-part-4-unit-testing/"
---
Alright quite a while ago (already counting years), I published a tutorial series focused on helping people getting started with Spark. Here is an outline of the previous posts:
<ul>
<li><a href="https://datacenternotes.com/2015/11/01/getting-started-with-spark-in-pythonscala-part-1/" target="_blank" rel="noopener">Part 1 Getting Started - covers basics on distributed Spark architecture, along with Data structures (including the old good RDD collections (!), whose use has been kind of deprecated by Dataframes)</a></li>
<li><a href="https://datacenternotes.com/2015/11/01/getting-started-with-the-spark-part-2-sparksql/" target="_blank" rel="noopener">Part 2 intro to Dataframes</a></li>
<li><a href="https://datacenternotes.com/2016/10/03/getting-started-with-spark-part-3-udfs-window-functions/" target="_blank" rel="noopener">Part 3 intro to UDFs and Window Functions </a></li>
</ul>
In the meanwhile Spark has not decreased popularity, so I thought I continued updating the same series. In this post we cover an essential part of any ETL project, namely Unit testing.
<a href="https://github.com/diogoaurelio/pyspark-bootstrap-repo" target="_blank" rel="noopener">For that I created a sample repository</a>, which is meant to serve as boiler plate code for any new Python Spark project.
{:.lead}
Let us browse through the main job script. Please note that the repository might contain updated version, which might defer in details with the next gist.
https://gist.github.com/diogoaurelio/9b6338d257e51cddb2f3015ea2ee17ae
The previous gist recovers the same example used in the previous post on UDFs and Window Functions.
Here is an example how we could test our "amount_spent_udf" function:
https://gist.github.com/diogoaurelio/f1a1f0f79f235d14868cb6dfe2015c93
Now note the first line on the unit tests script, which is the secret sauce to load a spark context for your unit tests. Bellow is the code that creates the "spark_session" object passed as an argument to the "test_amount_spent_udf" function.
https://gist.github.com/diogoaurelio/709b1d6096da0ce9fdd713d3be4073d8
And that is it. We strongly encourage you to have a look on the <a href="https://github.com/diogoaurelio/pyspark-bootstrap-repo" target="_blank" rel="noopener">correspondent git repository, where we specify detailed instructions how to run it locally</a>.
And that is it for today, hope it helped!
