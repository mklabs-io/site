---
layout: post
title: List all your AWS resources with Go
date: 2019-03-12 08:12:46.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- AWS
- Cloud
- devOps
- Go
tags: []
meta:
  _rest_api_client_id: "-1"
  _rest_api_published: '1'
  _thumbnail_id: '4242'
  timeline_notification: '1552378369'
  _publicize_job_id: '28542778304'
  _oembed_67ed653e8d3ab51cb0fa96ebdf87d222: "{{unknown}}"
  _oembed_bd638a0501a12d2bc9c5d5ef464a7381: "{{unknown}}"
  _oembed_213ed2ebdf09e8a97df4d90a244bed08: "{{unknown}}"
  _oembed_6b1fab65fdef92c52e2ac7cefe59ea16: "{{unknown}}"
  _oembed_2107f4cf498e1c4865d34ddebe0ef690: "{{unknown}}"
  _oembed_d17073622ca4be3748cf7dfc7732c2ff: "{{unknown}}"
author:
  login: jrcferrao
  email: joao@mklabs.io
  display_name: João Ferrão
  first_name: ''
  last_name: ''
permalink: "/2019/03/12/list-all-your-aws-resources-with-go/"
---
Picking up on <a href="http://datacenternotes.com/2019/02/08/keeping-aws-costs-low-with-aws-nuke/">Diogo's last post on how to obliterate all resources on your AWS Account</a>, I thought it could also be useful to, instead, list all you have running.
Since I'm long overdue on a Go post, I'm going to share a one file app that uses the Go AWS SDK for to crawl each region for all <strong>taggable</strong> resources and pretty printing it on stdout, organised by Service type (e.g. EC2, ECS, ELB, etc.), Product Type (e.g. Instance, NAT, subnet, cluster, etc.).
The AWS SDK allows to retrieve all ARNs for taggable resources, so that's all the info I'll use for our little app.
Note: If you prefer jumping to full code code, please scroll until the end and read the running instructions before.
The objective
The main goal is to get structured information from the ARNs retrieved, so the first thing is to create a type that serves as a blue print for what I'm trying to achieve. Because I want to keep it simple, let's call this type SingleResource.
Also, since we are taking care of the basics, we can also define the TraceableRegions that we want the app to crawl through.
Finally, to focus the objective, let's also create a function that accepts a slice of []*SingleResource and will convert will print it out as a table to stdout:
https://gist.github.com/joaoferrao/610482a2394b17546e12d1c172f5198e
https://gist.github.com/joaoferrao/610482a2394b17546e12d1c172f5198e.js
{:.lead}

<h3>Start parsing the ARN</h3>
All ARNs are composed of, at least, something like arn:partition:service:. Having this in mind, we can find a function to extract the Service Name portion of the SingleResource.
Because AWS SDK will allow to crawl only 1 region at a time, we will know which region each resource belongs to. We will also know which account, so... this means we can remove the four first sections of the ARN: <del datetime="2019-03-11T23:25:23+01:00">arn:partition:service:region:account-id</del>:resourcetype/resource:qualifier, so we can strike that information by creating our own version of a "shortened ARN".
This will become useful down the line.
https://gist.github.com/joaoferrao/8d677df0ba39b5327e9598d1075039dd
<h3>AWS Resource Types</h3>
Since not all ARNs are created equal, it's useful to create a specific type for each of the services we are expecting to parse. While it's easy to deduce the name of the service via any ARN name, independently of the resource, it's not easy to treat all ARNs the same way to get the product of that service.
Let's take these ARNs for example:
[code language="bash" gutter="false"]
arn:aws:rds:eu-west-1:123456789012:db:mysql-db
arn:aws:elasticbeanstalk:us-east-1:123456789012:environment/My App/MyEnvironment
arn:aws:s3:::my_corporate_bucket/exampleobject
```
As you can see, there isn't an obvious way of retrieving the service name (s3, Elasticbeanstalk, rds), but an obvious way of of stating one is a bucket or another is a db or an environment. This can be further illustrated by the possibilities for naming that AWS provides for ARNs:
[code language="bash" gutter="false"]
arn:partition:service:region:account-id:resource
arn:partition:service:region:account-id:resourcetype/resource
arn:partition:service:region:account-id:resourcetype/resource/qualifier
arn:partition:service:region:account-id:resourcetype/resource:qualifier
arn:partition:service:region:account-id:resourcetype:resource
arn:partition:service:region:account-id:resourcetype:resource:qualifier
```
To overcome this, if we really want a specific report at a Product level, we could do so by specifying a method for each type of service that interprets the "shortArn" resulting from the second function we already created. We we want to tell our application how to parse EC2 and ECS ARNs and a Generic method for everything else, we could create a type for them, giving them a method "ConvertToRow".
Finally, at the bottom of the following snippet, I use a function that accepts the several different bits and pieces of information we were <strong>are able</strong><strong> to generate</strong> so far (service name, ARN, region) and converts that into our intended SingleResource type.
https://gist.github.com/joaoferrao/d275a5cd5002fb5f9d4fc0008c87bf4d
<h3>So far..</h3>
If you are still following along, we are now able to get the service with ServiceNameFromARN() and a shortened ARN with ShortArn() funcs, we are now able to pass them into ConvertArnToSingleResource() and get the correct SingleResource.
<h3>Finally, get the resources</h3>
All that's left is to find a way to get all the ARNs in ones account so our code can feed on them. I left this part to the main() function and use AWS SDK for Resource Groups Tagging API to retrieve such info:
Because this code could be heavily improved, I walk you through the steps first:
<ol>
<li>AWS SDK will crawl one region at a time, so I create an aws.Config for each one.</li>
<li>The calls to  AWS' GetResources method will be paginated with a max of 50 results per request, so I create an empty paginationToken outside the most nested for loop to control whether it should keep going or break so the app can create a new config for the next region and a client to call GetResources on.</li>
</ol>
https://gist.github.com/joaoferrao/4a44eb1630bfcb670249862731ee256e
<h3>Running Instructions</h3>
The <a href="https://gist.github.com/joaoferrao/59321c83c36fd4a6a8821ef6c7e9aa0d">full code is available in this Gist</a> if you want to copy to a one .go file, but before running <strong>remember to activate your AWS_PROFILE</strong> environment variable so that the AWS SDK knows which account and credentials to use. export AWS_PROFILE=aws_named_profile.
In summary, copy everything into a main.go, open the terminal in the same dir and:
```
export AWS_PROFILE=aws_named_profile
go run main.go
```
Voilá, you should see something similar to the following:
<img class="alignnone size-full wp-image-4242" src="{{ site.baseurl }}/assets/2019/03/caws-1.png" alt="caws" width="1762" height="528" />
