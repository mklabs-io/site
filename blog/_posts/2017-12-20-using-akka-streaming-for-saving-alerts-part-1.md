---
layout: post
title: Using Akka Streaming "saving alerts" - part 1
date: 2017-12-20 17:49:30.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Akka
- AWS
- docker
- Kafka
- redis
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '12721149644'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2017/12/20/using-akka-streaming-for-saving-alerts-part-1/"
---
Full disclosure: this post was initially published at <a href="http://www.bonial.com/labs/2018/01/savingalerts/" target="_blank" rel="noopener">Bonial tech blog here</a>, one my favorite companies at the heart of Berlin, and where I have been fortunate enough to be working for 2+ years as a freelance Data Engineer. If you are looking for positions in tech, I can't help to recommend <a href="http://www.bonial.com/careers/" target="_blank" rel="noopener">checking their career page</a>.
<h3>Overview</h3>
Some months ago I was working on an internal project at <a href="http://www.bonial.com/" target="_blank" rel="noopener">Bonial</a> using Akka Streaming (in scala) to provide additional features to our current push notification system. The main goal of the project was to enhance the speed to which the client is able to notify its end users of available discount coupons. In this case, we wanted to notify the users in a real time fashion of available coupons on store, so that they could use them more effectively on the spot and save money. Hence our project code name: "saving alerts"!
After some architectural discussions where we compared several technical options, we decided to give akka streaming a go. It has been a fun ride, so we thought we might as well share some of the lessons learned.<!--more-->
This post has been divided into two parts:
<ol>
<li>Part 1 - we provide an overview about the main tech building blocks being used in this project (mainly focusing on Akka)</li>
<li>Part 2 - details how the system was designed and implemented</li>
</ol>
&nbsp;
Without further ado, let us start with an overview of our "saving-alerts" application:
<img class="alignnone size-full wp-image-3965" src="{{ site.baseurl }}/assets/2017/12/app-overview.jpg" alt="app-overview" width="1626" height="746" />
Logically speaking one can group the tasks executed by our application into three (3) main stages:
<ol>
<li>Read an event(s) from a given Kafka topic and perform basic validation (which are collected from our own tracking API); each event belongs to a unique user, and is triggered by our mobile App;</li>
<li>Querying an internal service - the so-called "Coupon API" - to check if there are any available coupons for that given user;</li>
<li>Apply of set of business logic rules - which at the moment are determined by our Product Managers - which determine whether, in the end, <em>to send or not to send</em> a push notification to that user mobile app.</li>
</ol>
Besides these main logical tasks, we still do some other "offline" housekeeping scheduled processes, namely: loading from an S3 bucket into memory updated versions of meta information about our retailers and the business logic rules, renew an Auth token to talk internally within the client's API, and obviously logging of app statistics for monitoring purposes.
In terms of tech stack, relevant for this project is simply the Akka actor system, a dedicated single node Redis instance, and some dedicated S3 buckets. All the rest - such as the tracking API, Kafka queue, Authentication API, Coupon API, Monitoring Service and Push Notification API, etc. - are all viewed as external services from the app point of view, even though most of them belong inside the company.
Though not particularly relevant for this project, the whole Akka application was deployed on AWS on ec2 instances. As we state in our final conclusion notes, a good fit for this application would also be to use some Docker container orchestration service such as Kubernetes.
Before we dive deep into how we implemented this system, let us first review the main technical building block concepts used in the project.
<h3 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-SystemArchitecture">System Architecture</h3>
The main building block of the current application is the Akka framework. Hopefully this section will guide you through some of the rational that we used to guide our decisions, and ideally why we choose to use Akka for this project.
<h3 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-AboutAkka">About Akka</h3>
<h4 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-Whyakka">Why akka</h4>
Let's start from the very basics: building concurrent and distributed applications is far from being a trivial task. In short, Akka comes to the rescue for this exact problem: it is an open source project that provides a simple and high level abstraction layer in the form of Actor model to greatly simplify dealing concurrent, distributed and fault tolerant applications on the JVM. Here is a summary of Akka's purpose:
<ul>
<li>provide concurrency (scale up) and remoting (scale out)</li>
<li>easily get rid of race conditions and multi-threading locking problems, such as deadlocks ("[...] when several participants are waiting on each other to reach a specific state to be able to progress"), starvation ("[...] when there are participants that can make progress, but there might be one or more that cannot"), and livelocks (when several participants are granting each other a given resource and none ends up using it) - please see <a class="external-link" href="http://doc.akka.io/docs/akka/2.4/general/terminology.html" rel="nofollow">akka documentation</a>, which does a fantastic job explaining it;</li>
<li>provide easy programming model to code a multitude of complex tasks</li>
<li>allow you to scale horizontally in your Application</li>
</ul>
&nbsp;
There are three main Building blocks we are using from Akka framework in this project:
<ul>
<li>Akka actors</li>
<li>Akka remote &amp; clustering</li>
<li>Akka streaming</li>
</ul>
<h4 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-AkkaActors">Akka Actors</h4>
The actor system provides asynchronous non-blocking highly performant message-driven programming model distributed environment. This is achieved via the Actor concept.
An Actor is sort of a safety container, a sort of light weight isolated computation units which encapsulate state and behaviour.
In fact, actor methods are private by default -  one cannot call methods on actors from the outside. The <u>only way to interact with actors is by message sending</u> - this holds also true for inter-actor communication. Moreover, and as stated in Akka documentation:  "<em>This method (the "receive" method, which has to be overriden by every actor implementation) is responsible for receiving and handling one message. Let's reiterate what we already said: each and every actor can handle at most one message at a time, thus receive method is <strong>never </strong>called concurrently. If the actor is already handling some message, the remaining messages are kept in a queue dedicated to each actor. Thanks to this rigorous rule, we can avoid any synchronization inside actor, which is always thread-safe.</em> "
Here is an example of a basic Actor class (written in scala), retrieved from Akka's documentation and changed on minor details:
https://gist.github.com/diogoaurelio/b469e5c6531e9b9f6b35cb8456292766
&nbsp;
<h4 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-Thereceivemethod">The receive method</h4>
In order to create an actor, one needs to extend the Actor Trait (sort of Java Interface), which mandates the implementation of the "receive" method - basically where all the action happens. In our example, in case the "MyActor" receives the message "test", the actor will log "received test", and if it receives the message "mutate", it will mutate its local variable by incrementing one (1). As each message is handled sequentially, and there is no other way to interact with an actor, it follows that you do not have to synchronize access to internal Actor state variables, as they are protected from state corruption via isolation - this is what is meant when one says that actors encapsulate state and behaviour.
As mentioned before, the receive method needs to be implemented by every actor. The receive Method is a PartialFunction, which accepts "Any" type and with void return. You can confirm this in Akka's source code, namely the Actor object implementation:
https://gist.github.com/diogoaurelio/376e40104ae660447aaa3201f10810d6
By the way, as a side note, the receive method being a PartialFunction is also one of Akka Streaming main proponents criticism, due to the lack of type safety.
In the provided example we are using simple strings as messages ("test", and "mutate"). However usually one uses <a class="external-link" href="http://docs.scala-lang.org/tour/case-classes.html" rel="nofollow">scala case classes</a> to send messages, since, as a best practice, messages should be immutable objects, which do not hold any object that is mutable. Finally, Akka will take care of serialization in the background. However, you can also implement your custom serializers, as is recommended speacially in the cases of remoting, in order to optimize performance or for complex cases. Here is an example how two actors can communicate with each other:
https://gist.github.com/diogoaurelio/1290126617b4042ba345496db4fc8c9b
If one wants to reply to a message sent, one can use the exclamation mark "!" notation to send a message. This is a "<em>fire and forget</em>" way of sending a message (which means there is no acknowledgement from the receiver that the message was indeed received). In order to have an acknowledgement one could use the ask pattern instead with the interrogation mark "?".
Also note that to retrieve the underlying message sender we call the "sender" method, which returns a so-called "ActorRef" - a reference of the underlying address of the sender actor.  Thus, if actor DudeB would receive message "hallo" from actor DudeA, it would be able to reply to it just by calling sender() method, which is provided in the Actor trait:
https://gist.github.com/diogoaurelio/67862899fe9e06ed2bbe143e6c6b3785
Finally, messages are stored in the recipients Mailbox. Though there are exceptions (such a routers, which we will see later), every actor can have a dedicated Mailbox. A Mailbox is a queue to store messages.
<h4 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-Messageordering">Message ordering</h4>
Important to note is that message <strong>order</strong> <strong>is not</strong> guaranteed. That is, if say Actor B has sent a message to Actor A at a given point in time, and then later Actor C sends a message to same Actor A, Akka does not provide any guarantee that the Actor's B message will be delivered before Actor's C message (event though Actor B sent it a message <em>before</em> Actor C). This would be fairly difficult for Akka to control especially in the case where actors are not co-located on the same server (as we will discuss later) - if Actor B is having high gitter on his network for example, <u>it might happen that Actor C gets his message passed through first</u>, for example.
Though order between different actors is not guaranteed, Akka does guarantee that messages from the same actor to another actor will be ordered. So, if Actor B sends one message to Actor A, and then later sends a second message again to Actor A, one has the guarantee that, assuming both messages are successfully delivered, <u>the first message will be processed before the second</u>.
&nbsp;
Besides being processed sequentially, it is also relevant to note that messages are also processed asynchronously to avoid blocking the current thread where the actor is residing. Each actor gets assigned a light weight thread - you can have several millions of actors per GB of heap memory - which are completely shielded from other actors. This is the first basic fundamental advantage of Akka - providing a lighting fast asynchronous paradigm/API for building applications.
As we will see next, akka provides many more building blocks which enhance its capabilities. We will focus on how akka benefits this application specifically, namely how it provides an optimized multi-threading scale-up (inside the same server) and scale-out (accross several remote servers) environment for free.
<h3 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-AkkaRouters(scale-up)">Akka Routers (scale-up)</h3>
Router actors are a special kind of actors, that make it easy to scale out. That is, with exactly the same code, one can simply launch an actor of a type Router, and it starts automatically child actors - so-called "routees" in akka terminology.
The child actors will have their own Mailboxes; however the <strong>Router itself will not</strong>. A router will serve as a fast proxy, which just forwards messages to it's own routees according to a given algorithm. For example, in our application, we are simply using round-robin policy. However, other (some more complex) algorithms could be used, such as load balancing by routee CPU and memory statistics, or scatter-gun-approach (for low latency requirements for example), or even simply to broadcast to all routees.
<img class="alignnone size-full wp-image-3966" src="{{ site.baseurl }}/assets/2017/12/akka_router_architecture.jpg" alt="akka_router_architecture" width="1471" height="851" />
The main advantage of Routers is that they provide a very easy way to scale-up the multi-threading environment. With the same Class code and simply changing the way we instantiate the Actor we can transform an actor to a Router.
<h3 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-AkkaRemote&amp;clusteringmodules(scale-out)">Akka Remote &amp; clustering modules (scale-out)</h3>
To distribute actors accross different servers one has two modules available: <a class="external-link" href="http://doc.akka.io/docs/akka/2.5/scala/common/cluster.html" rel="nofollow">Akka remoting</a>, and, dependent on the first, <a class="external-link" href="http://doc.akka.io/docs/akka/2.5/scala/common/cluster.html" rel="nofollow">Akka Clustering</a>. Akka remote provides location transparency to actors, so that the application code does not have to change (well, neglectable) if an actor is communicating with another actor on the same server or on a different one.
Akka Clustering on the other hand, goes on step further and builds on top of Akka Remoting, providing failure detection and potentially failover mechanisms. The way clustering works is by having a decentralized peer-to-peer membership service with no single-point-of-failure, nor single point of bottleneck. The membership is done via gossip protocol based on Amazon DynamoDB.
As we will see later, the way we scale in this our application, the clustering advantadges are not currently being used. That is, we are not extending specific actor system accross more than one node. However, the application is written in a way that it is completly prepared for it.
<h3 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-AkkaStreaming(backpressureforstreamingconsumption)">Akka Streaming (backpressure for  streaming consumption)</h3>
<a class="external-link" href="http://doc.akka.io/docs/akka/2.5.3/scala/stream/index.html" rel="nofollow">Akka streaming</a> is yet another module from Akka, relatively recently released. The main point of it is to hide away the complexities of creating a streaming consumption environment,  and providing back-pressure for free. <a class="external-link" href="http://doc.akka.io/docs/akka/2.5.3/scala/stream/stream-flows-and-basics.html#back-pressure-explained" rel="nofollow">Akka has a really good documentation explaining back-pressure in more detail</a>, but in short back-pressure ensures that producers halt down production speed in case consumers are lagging behind (for example, for some performance issue not being able to consume fast enough).
Last but not least, it is important to highlight that Akka Streaming works kind of a blackbox (in a good way), doing all the heavy lifting in the background reliefing you to focus on other business critial logic. The API is also intuitive to use, with a following nice functional programming paradigm style. However, we should warn that as your operations graph complexity grows, you will be forced to dive deep into more advanced topics.
<h4 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-AboutKafka">About Kafka</h4>
<div>Kafka is a complex system - more specifically in this case a distributed publish-subscribe messaging system - where one of the many uses-cases include messaging. This is provided to the Akka application as a service, thus from the application stand point, one does not need to care much about it. However, it is beneficial to understand how the application scales and how it ingests data. The following summary attempts to highlight some of the core differences that make Kafka different from other messaging systems, such as RabbitMQ:</div>
<ul>
<li>Kafka implements philosophy dumb broker, smart consumer; consumers are responsible for knowing from when they should consume data - kafka does not keep track; this is a major destinction compared to, for example, RabbitMQ, where many sophisticated delivery checks are available to deal with dead letter messages; in regards to our application, given Akka’ Streaming back-pressure mechanism, our application will halt consumption, in case consumers are not able to keep up with producers;</li>
</ul>
<ul>
<li>Persistent storage during X amount of time; many clients can read from same topic, for as long as Kafka is configured to persist data;</li>
<li>Topics can be partitioned for scalability - in practice this means that we can distribute and store for the same topic among several severs;</li>
<li>Data in each partition added in append-only mode, creating an immutable sequence of records - a structured commit log; records are stored in key value structure, and in any format, such as: String, JSON, Avro, etc.</li>
<li>It follows that order is only guaranteed on a partition basis; that is, inside the same partition if event A was appended before event B, it will be consumed before as well by the Consumer assigned to that partition. However, among partitions order is <em><u>not</u></em> guaranteed; <a class="external-link" href="https://kafka.apache.org/intro#intro_topics" rel="nofollow">the following illustration taken from kafka's own project page</a> illustrates this concept better:</li>
</ul>
&nbsp;
<img class="alignnone size-full wp-image-3968" src="{{ site.baseurl }}/assets/2017/12/log_anatomy.png" alt="log_anatomy" width="416" height="267" />
<ul>
<li>Besides possibly being partitioned, topics can also be replicated among several nodes, thus guaranteeing HA;</li>
<li>Consumers can be assigned to groups, thus scaling the amount of times topic pool can be consumed;</li>
</ul>
For more detail about Kafka, we recommend <a class="external-link" href="https://kafka.apache.org/intro" rel="nofollow">Kafka's own page</a>, which has really good intro. Finally, if you are indeed familiarized with RabbitMQ, we would recommend reading the <a class="external-link" href="https://content.pivotal.io/blog/understanding-when-to-use-rabbitmq-or-apache-kafka" rel="nofollow">following article, comparing Kafka with RabbitMQ, especially to which use cases each fits best</a>.
&nbsp;
