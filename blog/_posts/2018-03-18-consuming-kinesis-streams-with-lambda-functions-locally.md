---
layout: post
title: Consuming Kinesis Streams with Lambda functions locally
date: 2018-03-18 10:47:03.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- AWS
- AWS kinesis
- docker
- java
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '15846455587'
  timeline_notification: '1521370024'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2018/03/18/consuming-kinesis-streams-with-lambda-functions-locally/"
---
This blog post was originally published at <a href="https://gosmarten.com/kinesis-lambda-docker.html" target="_blank" rel="noopener">GoSmarten website</a>. As the number of projects where we use it was increasing, we thought we might as well share it. Let me know if it was helpful!
<h2>Motivation</h2>
AWS offers the cool possibility to consume from Kinesis streams in real time in a serverless fashion via AWS Lambda. However in can become extremely annoying to have to deploy a Lambda function in AWS just to test it. Moreover, it is also expensive to hold a Kinesis stream (e.g. queue) up and running just to test code.
Thus, by combining <a href="https://docs.aws.amazon.com/streams/latest/dev/developing-consumers-with-kcl.html">Kinesis Client Library (KCL)</a> with AWS Kinesis and DynamoDB docker containers we were able to recreate locally everything that happens on the background when you plug a Lambda function to a Kinesis stream on AWS. Besides saving costs, this allows developers to substantially <b>reduce development time</b>, as well as develop <b>higher quality code </b>due to the ease and flexibility of testing different scenarios locally.
Feel free to checkout the <a href="https://github.com/gosmarten/aws-kinesis-lambda-mock-cloud">code supporting this blog post on our repository</a>.
<h2>Context: Event Sourcing/CQRS</h2>
&nbsp;
Event sourcing is not a new concept, but as available streaming technologies have evolved, its widespread use has gained the attention it deserves. Thanks to “publish-subscribe” type of queues, it has become much easier to build streams of events available to multiple consumers at the same time. This democratization of access to an immutable, append-only stream of events is essential, as it separates the responsibility of modelling an event schema to a particular logic. It is also the reason why so many people either argue CQRS and event sourcing are the same thing, or have a symbiotic relationship.<!--more-->
The consumer on his side can then choose how to represent a given fact and ingest it in light of his own business framing. Now, it is important to note that though consumers use events to constantly mutate state representation and store data in a database - that does not mean that they are locked to that interpretation forever in time. As raw events are stored in an immutable fashion and decoupled from consumers interpretation, state can always be replayed and reconstructed to any given point in time.
This is where AWS Kinesis shines, offering a queue as a service, offloading a lot of the maintenance effort and complexities from your development and/or operations teams.
<img class="alignnone size-full wp-image-3983" src="{{ site.baseurl }}/assets/2018/03/kinesis-architecture.png" alt="kinesis-architecture" width="1102" height="474" />
Furthermore, having serverless streaming consumption can further offload a substantial chunk of work of streaming projects. Lambda functions obviously have limitations, and cannot stand by themselves compared to proper streaming engines, such as Apache Flink, Apache Spark, etc. The first limitation would be that each execution is stateless and independent of the previous one. The implications are that, unless you query some external data source, you are left alone in the party with a collection of events, completely blind to what happened in the previous execution. To add up, at the time of writing this blog post, lambda functions can only run up to 5 minutes maximum.
However, depending on your requirements, there are potential ways around their limitations. One example would be using materialized views, supported in all main relational databases. Depending on your write load, some people even consider database triggers, though this might be a ticking bomb in the medium/long term. Another strategy could be leveraging further AWS goodies, using DynamoDB as a stateful layer and DynamoDB streams to progressively evolve as potential out of order events arrive.
Thus, considering plugging AWS Lambda functions to your streaming execution can be a viable solution, depending on the complexity of the application you are building. Next, we dive into more details on how AWS is actually implementing the plugging of Lambda functions to Kinesis streams.
<div class="title-bordered border__dashed">
<h2>AWS Lambda for streaming</h2>
</div>
AWS Lambda service has come a long way since it was launched, and it integrates with numerous other services. One of the ways it integrates with other services is by allowing you to specify other services as triggers for lambda execution. In our case, the service is Kinesis streams. The way this works is by having the AWS Lambda service constantly polling the stream and invoking a particular lambda function.
When using AWS services as a trigger for lambda invocation, that invocation is obviously predetermined by the service. As stated in the <a href="https://docs.aws.amazon.com/lambda/latest/dg/java-invocation-options.html">AWS documentation</a>, in the case of stream-based services - at the time of writing this blog post that means Kinesis streams and DynamoDB streams - the invocation will always be synchronous. However, the polling from the stream is done in parallel, where the parallelism level is determined by the number of shards a given stream has. In practice, that will potentially result in having X amount of lambda functions simultaneously running. If this represents a potential issue for you, one way to minimize ordering issues is to adapt the partition key in your producer to group events conveniently.
Last but not least, one of the cool things about this solution is that the polling from the stream is done in the background by AWS. Every second AWS will poll from the stream, and if there are records, it will pass that collection of records to your lambda function. However, don’t fear: you can and should customize this batch size of records, given that, as previously mentioned, lambda functions can only run up to 5 minutes.
<div class="title-bordered border__dashed">
<h2>Running locally</h2>
The following steps assume that you have clone locally <a href="https://github.com/gosmarten/aws-kinesis-lambda-mock-cloud" target="_blank" rel="noopener">our public repo</a>.
<h3>1) start docker env</h3>
</div>
Although we are big fans of docker compose, we have rather chosen to implement bootstrapping docker containers via bash scripts for two main reasons:
a) We wanted to give developers the flexibility to choose which Dockers to start. For example, Java consumer implementation requires using a local DynamoDB, whereas Python doesn't;
b) We wanted to have the flexibility to automate additional functionality, such as creating Kinesis streams and DynamoDB tables, listing them after boot, creating local AWS CLI profile, etc.
To bootstrap all containers:
<code class="code-block">cd docker/bootstrapEnv &amp;&amp; bash +x runDocker.sh
If you check the runDocker.sh bash script, you will see that it will:
<b>a)</b> start DynamoDB docker
<b>b)</b> locally setup a fake AWS profile to interact with local Kinesis
<b>c)</b> start Kinesis container
<b>d)</b> create a new kinesis stream Note that we are not persisting any data from these containers, so every time you start any Docker it will be "brand new".
Also relevant to point out is that we are creating the stream with three shards. In WS Kinesis terminology, this means the queue partitioning, and also how one would improve read/write capacity of a given stream. However, in reality this is completely mocked, since we are running a single docker container, which "pretends" to have X amount of shards/partitions.
<div class="title-bordered border__dashed">
<h3>2) Publish fake data to Kinesis stream</h3>
</div>
To help you get started, we provided yet another bash script that pushes mock data (json encoding) to the same Kinesis stream previously created.
To start producing events:
<code class="code-block">cd docker/common &amp;&amp; bash +x generateRecordsKinesis.sh
This will continuously publish events to the Kinesis stream until you decide to stop it (for example by hitting Ctrl+C). Note also that we are randomly publishing to any of the 3 available shards. For future reference, besides adapting our mock event data towards your requirements, the specification of partition key might also be something you want to enforce depending on how your producers are publishing data into Kinesis.
<div class="title-bordered border__dashed">
<h3>3) Start consuming from kinesis stream</h3>
</div>
At this point, you have everything to test the execution of a Lambda function. We have provided an example of a basic Lambda - com.gosmarten.mockcloud.localrunner.TestLambda - that just prints each event. To actually test it running, you need to run com.gosmarten.mockcloud.localrunner.RawEventsProcessorLambdaRunner class. This class continuously iterates over each stream shard and pulls for data, which it will then pass to our lambda as a collection of Records.
<div class="title-bordered border__dashed">
<h3>4) How to test your own Lambda functions</h3>
</div>
<code class="code-block">final KinesisConsumerProcessorManager recordProcessorFactory = new KinesisConsumerProcessorManager&lt;&gt;(TestLambda::new);<code class="code-block">recordProcessorFactory.setAwsProfile(AWS_PROFILE).runWorker(STREAM_NAME, APP_NAME);
Instead of "TestLambda", specify your own. And ... that's it!
Last but not least, stay tuned as we plan to update <a href="https://github.com/gosmarten/aws-kinesis-lambda-mock-cloud">the original repo</a> with the python version. Happy coding!
&nbsp;
&nbsp;
&nbsp;
