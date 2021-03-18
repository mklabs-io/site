---
layout: post
title: Using Akka Streaming for "saving alerts" - part 2
date: 2018-02-19 06:18:24.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Akka
- async
- Big Data
- Kafka
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '14893550479'
  timeline_notification: '1519021105'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2018/02/19/using-akka-streaming-for-saving-alerts-part-2/"
---
This blog post is the second and final part of the post <em>Using akka streaming for "saving alerts"</em>. <a href="https://datacenternotes.wordpress.com/2017/12/20/using-akka-streaming-for-saving-alerts-part-1/" target="_blank" rel="noopener">Part 1 is available here</a>.  In this part we enter the details on how the application was designed.
Full disclosure: this post was initially published at <a href="http://www.bonial.com/labs/2018/01/using-akka-streaming-for-saving-alerts-part-2/" target="_blank" rel="noopener">Bonial tech blog here.</a> If you are looking for positions in tech, I would definitely recommend <a href="http://www.bonial.com/careers/" target="_blank" rel="noopener">checking their career page</a>.
<h2 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-ApplicationActorSystem">Application Actor System</h2>
The following illustration gives you a schematic view of all the actors used in our application, and (hopefully) some of the mechanics of their interaction:
<img class="alignnone size-full wp-image-3972" src="{{ site.baseurl }}/assets/2018/02/akka-streaming-pipeline.jpg" alt="akka-streaming-pipeline" width="1815" height="884" />
As previously mentioned, one could divide the application lifecycle logically into three main stages. We will get into more detail about each one of them next, but for now let us walk through them narrowly and try to map to our application<!--more-->
<h5 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-MainActors">Main Actors</h5>
The main actors in our application are: <em>KafkaConsumerActor</em>, <em>CouponRouterActor</em> and <em>PushNotificationRouterActor</em>. They perform the core business tasks of this application:
<ul>
<li><strong>Consume events from Kafka and validate them</strong> - this is done by <em>KafkaConsumerActor</em>. This is also the actor who controls the whole Akka Streaming Pipeline/flow. The flow is controled so that we can be assured to not overflow the main downstram Akka actors - <em>CouponRouterActor</em> and <em>PushNotificationRouterActor</em> - with too many events such that they cannot handle.</li>
<li><strong>Query Coupon API for results</strong> - for available coupons for a given merchant and for a given user, we query coupon API for results. Those results are sent back to Akka Streaming Pipeline.</li>
<li><strong>Apply Business Rules &amp; fire or not a Push notification</strong> - the last key stage involves sending returned results to <em>PushNotificationRouterActor</em> for it to apply a given set of business rules. In case those rules consider the event valid, a push notification may be fired, in case none has been sent in the last X amount of hours.</li>
</ul>
Not mentioned yet is <em>MetaInfoRouterActor</em>. It is used with sole purpose of monitoring statistics throughout the whole Akka Streaming pipeline flow. As written on the illustration, given that it is not a core feature of the application itself, and thus we send all messages to our monitoring service in a "fire and forget" manner - that is, we do not wait for acknowledgement. This of course implies that there is the possibility of messages not being delivered, and ultimately not landing in our monitoring system. However, this was considered as a minor and neglectable risk.
<h5 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-SecondaryActors">Secondary Actors</h5>
In the sidelines, and as a secondary service that serves the main actors we have three actors: <em>AppMasterActor</em>, <em>MetaInfoActor</em> and <em>RulesActor</em>.
<u>AppMasterActor</u> actor has two main functions: <u>control the discovery protocol</u> that we implemented, and host <u>healthcheck endpoint</u> used for outside application monitoring.
The so called discovery protocol basically makes sure that all actors know where - on which servers - other actors are, so that theoretically speaking we could separate each actor into different servers in a scale-out fashion. As a side note, we would like to highlight that this discovery protocol could have been implemented using Distributed PubSub modules from Akka - which would be definetely more advisable in case our application would grow in the number of actors. Full disclosure: at the time, due to the simplicity of our current App, it seemed simpler to implement it ourselves to keep the project simpler and smaller, which might be a questionable technical architecture decision.
Technically speaking, <em>MetaInfoActor</em> and <em>RulesActor</em> are almost identical actors in their implementation: they basically have a scheduled timer to remind them to check in a S3 bucket for a given key, stream-load it into memory, and broadcast it to their respective client actors.
As explained in the previous section, routers host many workers (so called "routees") behind them, serving as a ... well yes, router in front that directs traffic to them. All the actors that are Routers have it explicitely referenced in their name. Thus, when we say the <em>MetaInfoActor</em> or the <em>RulesActor</em> broadcast a message, in fact we are just sending one single message to the respective Router wrapped in a Broadcast() case class; the router then knows that it should broadcast the intended message to all it's routees.
<h5 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-Scalability&amp;HA">Scalability &amp; HA</h5>
All the actors depict in the illustration live in the same server. As a matter a fact, for the time being we are scaling out the application kind of in a "schizofrenic manner" - we deploy the application in different servers, and each server runs a completely isolated application <em><u>unaware of the existance of other twin applications</u></em>. In other words, actors living inside server 1 do not cumunicate with any actor living in server 2. Thus we like to call our current deployment "Pod mode". We are able to achieve this because all the application "Pods" are consuming events from Kafka using the same consumer group. Kafka intelligently assigns partitions Ids to the several consumers. In other words, Kafka controls the distribution of partitions to each POD, thus allowing us to scale out in a very simple manner:
<img class="alignnone size-full wp-image-3973" src="{{ site.baseurl }}/assets/2018/02/kafka-actor.jpg" alt="kafka-actor" width="1774" height="779" />
To increase performance, we can scale out the number of <em>KafkaConsumerActors</em> up to the same number of Kafka partitions. So, for example, if we had a topic with three (3) partitions, we could improve consumption performance by scaling up to three (3) KafkaConsumerActors.
To address High Availability (HA), we could, theoretically speaking, add N+1 <em>KafkaConsumerActors</em>, where N is the number of paritions for HA purposes, if our application was mission critical and very sensitive to performance. This would, however, only potentially improve HA of the application, as this additional <em>KafkaConsumerActor</em> would sit iddle (e.g. not consuming anything from Kafka) until another <em>KafkaConsumerActor</em> node failed.
Moreover, in case you are wondering, not having N+1 <em>KafkaConsumerActor</em> does not severely harm the HA of our application, as Kafka will reassign partition Ids among remaining Consumers in the event of Consumer failure. However, this obviously means that consumers will be unbalanced, as one them will be simultaneously consuming from two partitions.
Now, you may ask what happens in the case of failure of a given node that was processing a given event? Well at the end of the Akka Streaming Pipeline each <em>KafkaConsumerActor</em> commits back the message offset to Kafka - thus ackowledging consumption. So in this case, after the default TTL of message consumption that is configured in Kafka passes, we know that a message was not successfully processed (no acknowledgement), and so another <em>KafkaConsumerActor</em> will actually read again from Kafka that same message, and thereby reprocessing it.
As mentioned previously, when an event processing was processed by the system <em>KafkaConsumerActor</em> will commit back the to Kafka that event's offset, thereby acknowledging to Kafka that a given message has been successfully consumed for it's Kafka Consumer Group. We can't stress this enough (and thus repeating ourselves): this is how we are able to guarantee at <strong>at-least <em>once</em> semantics</strong> when processing a given message. Note that in our case, since we are storing in Kafka the offsets, in our implementation <strong>we cannot guarantee <em>exactly once</em> semantics</strong>. Nonetheless, this does not constitute a problem, as we are later using Redis cache to assure event. For <a class="external-link" href="http://doc.akka.io/docs/akka-stream-kafka/current/consumer.html" rel="nofollow">more information about Akka Kafka consumer, please check here</a>.
Let us address scalabilty in the rest of the application, by taking the <em>CouponRouterActor</em> architecture into consideration.
<img class="alignnone size-full wp-image-3974" src="{{ site.baseurl }}/assets/2018/02/coupon-actor-scalability.jpg" alt="coupon-actor-scalability" width="1848" height="955" />
&nbsp;
As shown in the previous illustration, performance is scaled by using Akka "routees" behind CouponRouterActor (as well as behind PushNotificationRouterActor). On of the beauties of Akka is that it allows us to code the CouponRouterActor 99% the same as if it was not operating as Akka Router. Simply on Actor class instantiation we mention its Router nature, and the rest is handled by Akka.
<h5 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-Finalremarks">Final remarks</h5>
We will dive into more detail into each stage next. However, we would like to highlight the importance of Akka Streaming Pipeline. It is able to control how many messages should be read from Kafka, because it sends messages to <em>CouponRouterActor</em> and <em>PushNotificationRouterActor</em> using the <a class="external-link" href="https://alvinalexander.com/scala/scala-akka-actors-ask-examples-future-await-timeout-result" rel="nofollow">Ask Pattern</a> - which waits for a response (up to a given time-to-live (TTL)).
Also note that no matter how far an event may go down the flow (an event may be, for example, filtered right in the beginning in case it is considered invalid), we always log to Datadog that a given message was read from Kafka, and was successfully processed. Note that "successfully processed" can have different meanings - either considered Invalid event right in the beginning of the streaming pipeline, or no available coupons returned from Coupon API, or even business rules considered that the system should not send push notification to Kepler API, as business rules define it is unfit.
Moreover, please note that when an event processing is finished - <em>again, no matter how far it goes down the stream pipeline</em> - <em>KafkaConsumerActor</em> has the final task of committing back the to Kafka for that event's offset. In other words, we acknowledge back to Kafka that a given message has been processed. This is an important detail -  since in case of failure of processing a given event (let's say one of the application servers crashes), after the default TTL of message consumption tha tis configured in Kafka passes, another <em>KafkaConsumerActor</em> will actually read again from Kafka that same message, thus reprocessing it.
&nbsp;
<h2 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-Dockerenvironment">Docker environment</h2>
Currently we are only using Docker for local development, although this application would fit quite well in, say, Kubernettes cluster, for example.
We have setup a complete emulation of the production env in local via docker:
<ul>
<li>Akka Application cluster - image customized by ourselves</li>
<li>Kafka Cluster - used <a class="external-link" href="https://hub.docker.com/r/wurstmeister/kafka/" rel="nofollow">wurstmeister image</a>.</li>
<li>Redis Docker - used <a class="external-link" href="https://hub.docker.com/_/redis/" rel="nofollow">redis official image</a>.</li>
<li>S3 mock Docker - used <a class="external-link" href="https://hub.docker.com/r/scality/s3server/" rel="nofollow">scality image</a>.</li>
</ul>
This is (extremely) useful not only to get a better grip of how the system works in day to day development, but also to do harder to emulate behavioral tests, such as High Availability (HA) tests.
<h1 id="Blogpost-Usingakkastreamingfor&quot;savingalerts&quot;-FinalNotes">Final Notes</h1>
Like any application, there are a number of things that could have be done better, but due to practical constraints (mainly time), were not.  Let us start with some of the things we do not regret:
<ul>
<li><strong>Using Akka: </strong>there are many ways we could have implemented this application. Overall akka is a mature full-fledge framework - contains modules for almost anything you may require while building distributed highly available asyncronous applications - and with very satisfactory performance.</li>
<li><strong>Using Akka streaming</strong>: there are many blogs out there with horror stories on constant performance issues with pure Akka implementations. Akka Streaming module, not only increases stability via back-pressure, it also provides a very intuitive and fun to work with API</li>
<li><strong>Using Docker in local</strong>: this allowed us to test very easily and especially rapidly in our local machines, more rare scenarious, such as simulating failures on all points in the application: Kafka nodes, Redis, S3, and of course, the Akka application itself.</li>
</ul>
Some open topics for further reflection:
<ul>
<li>Using our own discovery protocol ended was a questionable technical decision. One possible alternative could have been using akka module "DistributedPubSub"</li>
<li>Ideally, this application would be a very nice initial use case to start using Container orchestration tools, such as Kubernetes</li>
</ul>
&nbsp;
And ... that's all folks. We hope that this post was useful to you.
