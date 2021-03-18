---
layout: post
title: Faust, your Python-based streaming library
date: 2019-05-29 11:36:04.000000000 +02:00
type: post
parent_id: "0"
published: true
password: ""
status: publish
categories:
  - async
  - IoT
  - python
  - streaming
tags:
  - backend
  - faust
meta:
  _thumbnail_id: "4251"
  timeline_notification: "1559129774"
  _rest_api_published: "1"
  _rest_api_client_id: "-1"
  _publicize_job_id: "31295823788"
  _oembed_e7056704db51174d42bef513c3e457a1: "{{unknown}}"
author:
  login: jrcferrao
  email: joao@mklabs.io
  display_name: João Ferrão
  first_name: ""
  last_name: ""
permalink: "/2019/05/29/faust-your-python-based-streaming-library/"
---

Robinhood is a very popular California based FinTech, included <a href="https://www.forbes.com/companies/robinhood/#63d753c06076">by Forbes</a> in the top 50 FinTechs to watch in 2019. Their primary mission is to bring down stock trading fees for the common Joe/Jane, although their roadmap also includes cryptocurrency trading.
Due to the nature of the bidding market, their data stack probably includes a lot of stream tooling. Also (probably) due to the lack of quick and easy tooling for streaming with Python, supported with the <a href="https://insights.stackoverflow.com/survey/2019#technology-_-most-loved-dreaded-and-wanted-languages">growing demand for Python in the developer community</a>, they launched their own port of Kafka Streams, called <a href="https://faust.readthedocs.io/">Faust</a>.
In this post, we're going to show how to easy it is to bootstrap a project with Faust to put your stream related business logic needs in practice very quickly. The demo we prepared is of an app which filters words from a Kafka topic and then keeps a count of how many times it has seen the colors "red", "green" and "blue".</
In a nutshell, Faust is:
{:.lead}

<ul>
<li class="topic-title first">easy to bootstrap streaming library for Python to use (mostly) Apache Kafka as the source;</li>
<li class="topic-title first">relies on Python's asyncio to split the tasks into agents</li>
<li class="topic-title first">distributing the workload between agents living in the same cluster</li>
<li class="topic-title first">provides stateful​processing with support for rocksdb (same as Apache Flink)</li>
<li>the agents can use whatever Python code/modules/libraries you are accustomed to use to the transformation or sinking of data (numpy, pandas, boto3, whatever you so desire).</li>
</ul>
<p class="topic-title first">The original developers of Faust describe it as falling under the use case of any of the following domains:
<table class="hlist">
<tbody>
<tr>
<td>
<ul class="simple">
<li><strong>Event Processing</strong></li>
<li><strong>Distributed Joins &amp; Aggregations</strong></li>
<li><strong>Machine Learning</strong></li>
<li><strong>Asynchronous Tasks</strong></li>
<li><strong>Distributed Computing</strong></li>
</ul>
</td>
<td>
<ul class="simple">
<li><strong>Data Denormalization</strong></li>
<li><strong>Intrusion Detection</strong></li>
<li><strong>Realtime Web &amp; Web Sockets.</strong></li>
<li><strong>and much more…</strong></li>
</ul>
</td>
</tr>
</tbody>
</table>
Also, luckily, Faust's documentation is already at a mature stage, even including instructions on <a href="https://faust.readthedocs.io/en/latest/userguide/testing.html">how to test your applications</a>, in case you are not familiarised with Python's asyncio.
<h2>Distributed</h2>
It's also worth noticing that Faust is tightly coupled to core Kafka concepts. More concretely, it relies on the way Kafka manages consumer groups to identify whether any partition/topic is not being attended to and launching an agent in an instance where  the application is reporting as functioning. Or, more correctly, it assigns that partition to an agent living on an instance/app that is still identified by Kafka as still being active in the consumer group.
<img class="alignnone size-full wp-image-4252" src="{{ site.baseurl }}/assets/2019/05/basic_faust-e1558701705450.png" alt="basic_faust.png" width="1602" height="762" />
On the image above, we have a single Kafka topic which consists of four partitions. We also have two instances running simultaneously of the same Faust app. On Diagram 1 both are working, so the elected Faust leader for that topic will decide which topics it will get and which topics the other consumers in the same consumer group will get the remaining partitions. Since there is a 4 by 2 distribution, it will attribute 2 partitions to each instance.
On Diagram 2, the first instance goes down. Based on a certain timeout (which is can also be parameterised in faust settings), the customer group will inform the leader that one of the consumers is down and will force a re-distribution of topic/partition attribution. In this case, it will assign all four partitions to the same instance.
The reason this is important to mention is that the <strong>maximum parallelisation you are able to get out of any given topic is as high as its number of partitions</strong>. A topic with eight partitions will, at maximum, be assigned to eight instances of the app.
<h2>It's all about the topics</h2>
<h3>Streams</h3>
When developing Faust, you are creating Agents which are basically async functions that look into every event passing through a stream and apply some logic to it. A stream is the basic abstraction for the communication link between source/agents/sinks and you can, usually, look at them as a Kafka topic.
If you have a simple workflow, like counting words, you would have
<ul>
<li>a single agent that reads from a Kafka topic (not managed by Faust)</li>
<li>updates the word count table</li>
<li>sends the result to another stream/topic.</li>
</ul>
However, you could have an agent filtering the words filtering words, selecting only words for the 3 prime colors and sending the "filtered" results to another Faust agent that keeps the actual relevant word count.
So how do the agents communicate to each other? We'll illustrate later the example, but it's important to know that if we want agents to communicate with each other we can declare "internal" topics which are managed by Faust. It will take care of creating them with the default (8) or expressed number of partitions.
<h3>Stateful / Managed state</h3>
Because managed state is a whole different discussion, in order to keep this article focused, it's important to mention that Faust supports it in a very peculiar way. The concept is that we declare a table for each object that want want to keep track of. Each table is basically a special Faust class that behaves very similarly to a Python Dictionary.
<ul>
<li>for persistence: supports both Rocksdb used for Production (same as Flink, not that they tackle the same problems at the same level) and Memory (useful for development).</li>
<li>for recovery: uses topics to keep track of the table's changelog of each table. This is done for you.</li>
</ul>
<h2>Basis for demo</h2>
In order to create sharable and demonstrable code, let's look at the repository's structure, which you <a href="https://github.com/joaoferrao/datacenternotes-faust-python-streaming">can find here</a>.
├── Makefile
├── README.md
├── requirements.txt
├── docker
│  ├── docker-compose-local.yml
│  └── kafka-docker
└── src
    └── app.py
<ul>
<li>We'll need a handy Kafka cluster running locally. For ease of reproducibility, we're using the popular <a href="https://github.com/wurstmeister/kafka-docker">wurstmeister</a>'s <em>kafka-docker. </em></li>
<li>There is a Makefile which eases launching individual components (Kafka+Zookeeper or the sample faust app)</li>
<li>Faust app is stored in src/app.py for simplicity, but <a href="https://faust.readthedocs.io/en/latest/userguide/application.html#id23">they have instructions on how to organize larger projects</a></li>
</ul>
We will be running our Python code from outside Docker, so we need a virtual environment. We suggest using conda, so... after cloning the repository, you can follow the below to setup your proper environment
conda create -n faust-test python=3.6
conda activate faust-test
pip install -r requirements.txt
make kafka
The last command will launch Zookeeper and Kafka in different containers and will create 2 topics with 1 partition each: input and output. This is the source and sink topics which we'll use in our Faust app.
Finally, because we will be sending stuff to and from Kafka topics, we need the Kafka executables to achieve that purpose. You can download them <a href="https://kafka.apache.org/quickstart">here</a>. The bash scripts, inside the bin directory you get after decompressing the tarball, will have the files we need to interact with Kafka as producers or consumers of a certain topic (the input and output mentioned earlier).
<h2>Quick code review</h2>
https://gist.github.com/joaoferrao/8c63bc3ade919b7f6c8e3fba738996a1
This Faust app is relatively simple. It's purpose is to read all words into the input topic and keep a count of how many times the colors red, green and blue have been sent. Each time either of them has been said, it sends an update to the output topic.
After creating an app object from the faust.App class:
<ul>
<li>we declare two external topics with 1 partition each: input and output (which are created automatically by the provided docker code when running make kafka</li>
<li>we declare an internal topic colors_topic which is the means of communications between our 2 agents</li>
<li>we also declare a Table object so Faust knows to keep the running count of the colors in a managed state, in case it fails.</li>
<li>we create 2 agents: one filters colors from all the words and sends the color to the internal topic. The other agent keeps a count of the filtered colors that come in from the internal topic and reports its running count.</li>
</ul>
<h2>Running the sample code</h2>
Assuming you already ran the code in the <strong>Basis for demo</strong> above, then Kafka should already be running in the background and the <strong>conda environment</strong> <strong>should be activated</strong>.  You should also have already <strong>downloaded Kafka.</strong>
Now, I suggest you split your terminal in 2:
<ol>
<li>One table located at the repo's root</li>
<li>Another repo at the bin directory with Kafka's command line utility scripts.</li>
</ol>
On the repo tab, run make run-app which will start the app we described above.
On Kafka's bin directory run the following to start a Kafka terminal producer:  ./kafka-console-producer.sh --broker-list localhost:9092 --topic input
Now you can start writing individual words, including colors. A word of advice is that, since we haven't defined a concrete type (with a schema) for the values expected in the input topic, we need to include double quotes that we send to the output topic.
You should now have something like this:
<img class="alignnone size-full wp-image-4254" src="{{ site.baseurl }}/assets/2019/05/running_faust.png" alt="running_faust" width="809" height="805" />
<h2>Where to go from here</h2>
You can now start bootstrapping your own streaming project with Python and Kafka. My suggestion for your further exploration of Faust would be to try and see how state is managed when using topics with more than 1 partition and when using more than 1 Faust app instance. You'll understand how the Table gets sharded across Faust/Kafka consumers.
Another neat feature we did not explore was the type safety you can input on topics and tables: when declaring a topic <a href="https://faust.readthedocs.io/en/latest/userguide/agents.html#the-stream">you can specify the</a> valye_type (and even the key_type) you are expecting to exist in that topic.
Found an error or need clarification from some topic? Leave a comment down below.
