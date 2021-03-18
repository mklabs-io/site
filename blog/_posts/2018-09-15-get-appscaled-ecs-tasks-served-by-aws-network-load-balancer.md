---
layout: post
title: Get AppScaled ECS Tasks served by AWS Network Load Balancer
date: 2018-09-15 16:36:30.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- AWS
- devOps
- docker
- ECS
- Elastic Container Service
- Networking
- Terraform
tags: []
meta:
  timeline_notification: '1537029390'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '22178087391'
  _oembed_97534b07e5c9263f40d049502db70a8a: "{{unknown}}"
  _oembed_93b990e54bdd01615887049652e54a95: "{{unknown}}"
  _oembed_4dcb322af5bd48b541105a50ede23853: "{{unknown}}"
author:
  login: jrcferrao
  email: joao@mklabs.io
  display_name: João Ferrão
  first_name: ''
  last_name: ''
permalink: "/2018/09/15/get-appscaled-ecs-tasks-served-by-aws-network-load-balancer/"
---
This article is intended to be a quick and dirty snippet for anyone going to through the struggle of getting your ECS service, which might have one or more containers running the same App (being part of an Auto Scaling Group), with a Network Load Balancer (instead of the more common ELB or ALB).
<h3>ECS Service/Task Definition</h3>
Another particularity of this implementation is that I also decided to use the ECS task's network mode as <strong>awsvpc</strong>. In the case that you are not acquainted with this <em>new</em> option, this means that:
<ul>
<li>Your container will get its own network interface and its own IP address;</li>
<li>The Host port and the Container port need to be the same, since there is not <em>middleware</em> managing port match between the two entities.</li>
</ul>
The cherry on top is that the <strong>ECS Service</strong> now has the option of automatically registering and deregistering LB targets by their IP address, which fits perfectly on the intention described.
<h3>Network Load Balancer</h3>
This post isn't concretely about describing <a href="https://docs.aws.amazon.com/elasticloadbalancing/latest/network/introduction.html">the technical details</a> of what is a Network Load Balancer but about the caveats of using it in this scenario: because NLB is a layer 4 load balancer, <strong>you won't be able to define Security Groups at the NLB level</strong>. Instead, you'll have to make sure you make your tasks/containers secure by attaching the security groups to them - remember that with the <strong>awsvpc</strong> network mode, each container will get its own NIC.
<h3>Implementation</h3>
As for the actual code snippet to support what I'm trying to achieve:<!--more-->
https://gist.github.com/joaoferrao/d531160e38cf23821731d0d9881344d7
If you are familiar with Terraform and AWS services, the noteworthy parts part of the above code are:
<ul>
<li>The <strong>aws_lb</strong> is of type "network"</li>
<li>The host port and the container port are the same in the task definition</li>
<li><strong>awsvpc</strong> in the network mode of the tak</li>
<li>We make the ECS service responsible for registering targets with the "load_balancer" section.</li>
<li>The <strong>target_type</strong><strong> </strong>of the target group must be of type IP.</li>
<li>The security group to take into consideration here would be directly in the <strong>security_groups </strong>section of the <strong>aws_ecs_service</strong>.</li>
</ul>
And... that's it!
https://gist.github.com/joaoferrao/d531160e38cf23821731d0d9881344d7.js
&nbsp;
&nbsp;
&nbsp;
