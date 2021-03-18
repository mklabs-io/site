---
layout: post
title: Summary Terraform vs Spinakker
date: 2017-02-10 07:39:17.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- AWS
- Cloud
- Continuous Delivery
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '1700665495'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2017/02/10/summary-terraform-vs-spinakker/"
---
I recently needed to build this summary, so thought I'd rather share with more people as well. Please feel free to add any points you see fitting.
<h1>TL;DR</h1>
Rather then putting on versus another assuming mutual exclusivity, many companies are adopting both tools simultaneously.Terraform is usually used for static cloud Infrastructure setup and updates, such as networks/VLANs, Firewalls, Load Balancers, storage buckets, etc. Spinakker is used for setting up more complex deployment pipelines, mainly orchestration of software packages and application code to setup on servers.Though there is intersection (Spinakker can also deploy App environment), Terraform provides an easy and clean way to setup Infrastructure-as-Code.<!--more-->
<h1><a href="https://www.terraform.io/" target="_blank" rel="noopener noreferrer">Terraform</a></h1>
<ul>
<li>Hashicorp product focused on allowing you to cleanly describe and deploy infrastructure in Cloud and on premise environments;</li>
<li>Allows to describe your infrastructure as code, as well as to deploy it. Although complexer deployments replying on server images is possible, man-power effort starts growing exponentially</li>
<li>The <a href="https://www.terraform.io/docs/configuration/syntax.html" target="_blank" rel="noopener noreferrer">sintax</a> used to describe resources is own Hashicorp Configuration Language (HCL), which may be a turn off at first sight; however, after seeing how clear human readable it is, you won't regret it;</li>
<li>Deployment form varies; from immutable infrastructure style deployments (e.g. new update = complete new resource) to updates, depending on the nature of resources. For example, server instances require immutable approach, where firewall rules do not;</li>
<li>Failure to deploy resources does not roll back and destroy "zombie" resources; instead they are marked as tainted, and left as it is. On a next execution plan run, tainted resources will be removed, to give place to new intended resources. More detail <a href="https://www.terraform.io/intro/getting-started/provision.html" target="_blank" rel="noopener noreferrer">here</a>;</li>
<li><a href="https://www.terraform.io/docs/providers/" target="_blank" rel="noopener noreferrer">Multi-cloud deployments</a>, currently supporting AWS, GCP, Azure, OpenStack, Heroku, Atlas, DNSimple, CloudFlare.</li>
<li>Doesn't require server deployment, e.g. can be launched from your own local machine. However, Hashicorp provides Atlas, to provide remote central deployments.</li>
</ul>
<h1><a href="http://www.spinnaker.io/" target="_blank" rel="noopener noreferrer">Spinakker</a></h1>
<ul>
<li>Is a Netflix open source tool for managing deployment of complex workflows (Continuous Delivery). It is kind of a second generation/evolution of <a href="https://github.com/Netflix/asgard" target="_blank" rel="noopener noreferrer">Asgard</a>.</li>
<li>Orchestrator for deploying full Application environments, from sourrounding infrastructure environment (networks, firewalls and load balancers), along with server images;</li>
<li>Works hand-in-hand with tools such as <a href="https://jenkins.io/" target="_blank" rel="noopener noreferrer">Jenkins</a> and <a href="https://www.packer.io/" target="_blank" rel="noopener noreferrer">packer</a>.</li>
<li>As an orchestrator, it focuses on Pipelines, a sequence of stages. A stage is an atomic building block; <a href="http://www.spinnaker.io/docs/overview#section-deployment-management" target="_blank" rel="noopener noreferrer">examples of stages</a>:
<ul>
<li>Build an image (example AMI in case of AWS) in a specific Region</li>
<li>Deploy the image</li>
<li>Run a bash script</li>
<li>Resize a server group</li>
</ul>
</li>
<li>Multi-cloud deployments, currently supporting AWS, GCP, Azure, OpenStack, Kubernetes, and CloudFoundry.</li>
<li>It is by itself a product with several functionality, <a href="http://www.spinnaker.io/docs/creating-a-spinnaker-instance" target="_blank" rel="noopener noreferrer">requiring itself server deployemnt</a>. Images for easy deployment are available both in AWS, Azure, GCP, and Kubernetes;</li>
<li>Provides Web GUI for full pipeline configuration, as well as an API</li>
<li>Allows to setup webhooks for notifications via email, SMS and chat clients (HipChat/Slack)</li>
</ul>
&nbsp;
<h1>More Resources</h1>
<ul>
<li><a href="https://www.terraform.io/" target="_blank" rel="noopener noreferrer">Terraform.io</a></li>
<li><a href="http://www.spinnaker.io/v1.0" target="_blank" rel="noopener noreferrer">Spinakker.io</a></li>
<li>Armory Blog's contains several resources about Spinakker. Here is their own summary comparison: <a href="http://blog.armory.io/terraform-vs-spinnaker/" target="_blank" rel="noopener noreferrer">Spinakker vs Terraform</a></li>
<li>Kenzen labs provide a repo with sample code for <a href="https://github.com/kenzanlabs/spinnaker-terraform" target="_blank" rel="noopener noreferrer">Spinakker with Terraform</a></li>
</ul>
