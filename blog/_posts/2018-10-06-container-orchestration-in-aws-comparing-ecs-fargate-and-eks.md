---
layout: post
title: 'Container orchestration in AWS: comparing ECS, Fargate and EKS'
date: 2018-10-06 17:23:15.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- AWS ECS
- AWS EKS
- AWS Fargate
- devOps
- Kubernetes
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '22907326074'
  timeline_notification: '1538846596'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2018/10/06/container-orchestration-in-aws-comparing-ecs-fargate-and-eks/"
---
Before rushing into the new cool kid, namely AWS EKS, AWS hosted offering of Kubernetes, you might want to understand how it works underneath and compares to the already existing offerings. In this post we focus on distinguishing between the different AWS container orchestration solutions out there, namely AWS ECS, Fargate, and EKS, as well as comparing their pros and cons.
<h3><strong>Introduction</strong></h3>
Before we dive into comparisons, let us summarize what each product is about.
<strong>ECS</strong> was the first container orchestration tool offering by AWS. It essentially consists of EC2 instances which have docker already installed, and run a Docker container which talks with AWS backend. Via ECS service you can either launch tasks - unmonitored containers suited usually for short lived operations - and services, containers which AWS monitors and guarantees to restart if they go down by any reason. Compared to Kubernetes, it is quite simpler, which has advantageous and disadvantages.
<strong>Fargate</strong> was the second service offering to come, and is intended to abstract all everything bellow the container (EC2 instances where they run) from you. In other words, a pure Container-as-a-Service, where you do not care where that container runs. Fargate followed two core technical advancements made in ECS: possibility to assign ENI directly and dedicated to a Container and integration of IAM on a container level. We will get into more detail on this later.
The following image sourced from <a href="https://aws.amazon.com/blogs/compute/aws-fargate-a-product-overview/" target="_blank" rel="noopener">AWS blog here</a> illustrates the difference between ECS and Fargate services.
<img class="alignnone size-full wp-image-4194" src="{{ site.baseurl }}/assets/2018/10/fargate-1.png" alt="fargate-1" width="975" height="294" />
<strong>EKS</strong> is the latest offering, and still only available on some Regions. With EKS you can abstract some of the complexities of launching a Kubernetes Cluster, since AWS will now manage the Master Nodes - the control plane. Kubernetes is a much richer container orchestrator, providing features such as network overlay, allowing you to isolate container communication, and storage provisioning. Needless to say, it is also much more more complex to manage, with a bigger investment in terms of DevOps effort.
Like Kubernetes, you can also use kubectl to communicate with EKS cluster. You will need to configure the <a href="https://github.com/kubernetes-sigs/aws-iam-authenticator" target="_blank" rel="noopener">AWS IAM authenticator locally to communicate with your EKS cluster</a>.
{:.lead}
<h3><strong>Cost</strong></h3>
Let us start with costs, the number one usual favorite suspect that Software Developers like to neglect.
<strong>ECS</strong>, is a simple service, where the control plane is abstracted away by AWS. You simply launch Worker nodes, which consist of EC2 instance with Docker installed, and a Container daemon running which contains the minimum metadata to know to which cluster it belongs to, and to receive instructions from AWS ECS control plane. Thus, with ECS you only pay per Worker node, read EC2 instance.
<strong>Fargate</strong> abstracts from you the hosting platform of containers - the EC2 instances. This obviously means that you pay a premium per container, compared to ECS. So yes, the cost per Container in Fargate will naturally be higher. On the other hand, and to perform a fair comparison with pure ECS, you should also take into consideration maintenance costs associated with ECS. The hours invested troubleshooting, upgrading ECS instance agents, updating EC2 instance packages, etc., is translated in the form of salaries. Thus, should not be forgotten.
<strong>EKS</strong>, being Kubernetes under the hood,  requires Master nodes always running to monitor the cluster, the EKS control plane. This does not come for free, and will cost you minimum pro cluster 0.20 US Dollars/ hour, which amounts to 144$ pro month. This just to start playing around, without any Worker node. From here on, you only pay the normal price for EC2 instances you launch for Worker nodes, just like the ECS offering.
&nbsp;
<h3><strong>Auto-scaling</strong></h3>
<strong>ECS</strong> allows you to scale at the task level. Remember: a task maybe one or more containers. Much like EC2, you can configure rules to scale the number of tasks running. However, these are containers running on EC2 instances, which may not have available CPU or RAM resources. So in practice, the trick here is to define another Auto-scaling rule at the ECS cluster level, where you define scale up or down based on the CPU or RAM reservation on the cluster.
<strong>Fargate</strong>, as expected, makes the auto-scaling much easier. Here you simply need to define it at the task level, and you are good to go.
More on <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html" target="_blank" rel="noopener">ECS/Fargate auto-scaling can be found here</a>.
Just recently <a href="https://aws.amazon.com/blogs/opensource/horizontal-pod-autoscaling-eks/" target="_blank" rel="noopener">AWS introduced integration with Kubernetes Horizontal Pod Autoscaler (HPA) for EKS</a>. Much like ECS, this will  allow you to scale on the pod level, which means again that you will also need to scale on the EC2 level to support pod growth. More on getting started with <a href="https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/" target="_blank" rel="noopener">HPA here</a>.
Also, if you already have an EKS cluster running, make sure your <a href="https://docs.aws.amazon.com/eks/latest/userguide/platform-versions.html" target="_blank" rel="noopener">EKS version supports HPA</a>. All new EKS clusters will be launched with platform version eks.2+, which means they already support HPA at the time of writing this blog post.
Let us now dive deeper technically in the three main Networking aspects that distinguish the three services.
<h3><strong>Networking: Traffic Load Balancing</strong></h3>
Until AWS integrates EKS with their more sophisticated load balancers, namely Application LB and Network LB, traffic flow from a networking perspective will remain very inefficient. Since EKS only supports classic AWS ELB, traffic is randomly and blindly distributed among EC2 instances, and be prone to multi-hop traveling.
Both ECS and Fargate cleanly integrate with AWS ALB and Network LB, allowing you directly point at the intended containers. Cloudnout provide very good illustrations on <a href="https://cloudonaut.io/eks-vs-ecs-orchestrating-containers-on-aws/" target="_blank" rel="noopener">their blog post</a> on this topic.
<h3><strong>Networking: ENI</strong></h3>
Being able to assign a network card directly to a container might not seem like that big of a deal at first sight. However, effectively this means a higher level of security. This way you are able to assign a Security Group dedicated for that individual container, rather that simply opening all ports for the whole cluster.
With <strong>ECS</strong>, you have the option to associate an ENI directly with a container, by choosing to launch a task in "awsvpc" mode. However - there is always a but, isn't there ? - the maximum number of ENIs (read Virtual NICs) that can be assigned to each EC2 instance varies according to EC2 type; and one could argue that it is rather limited from 2 to 15 at the current time being. So if you are running a lot of small containers, this scenario effectively would mean creating a very big EC2 cluster of small EC2 instances. <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#AvailableIpPerENI" target="_blank" rel="noopener">The following list exemplifies this statement</a>.
With EKS, on the other hand, you have the option to assign a dedicated network interface a a container group, what Kubernetes calls a pod. So that means all containers inside that pod will share the same internal network. However, here come the good news, you can place up to 750 pods per instance. This is a much bigger advantage for EKS.
<h3><strong>Networking: Service Discovery</strong></h3>
AWS has <a href="https://aws.amazon.com/about-aws/whats-new/2018/03/introducing-service-discovery-for-amazon-ecs/" target="_blank" rel="noopener">recently</a> launched service discovery for ECS/Fargate. This was a key differentiator that the ECS was lagging behind compared to Kubernetes, which uses network overlay to allow essentially container level DNS.
However it needs to be noted that only Containers launched in "awsvpc" network mode support this feature.
Moreover, it might also be interesting to mention that AWS recently introduced transparent integration cross all their docker services, a thus unified service discovery for ECS and Kubernetes. <a href="https://aws.amazon.com/blogs/opensource/unified-service-discovery-ecs-kubernetes/" target="_blank" rel="noopener">See more about it in this blog post</a>.
&nbsp;
<h3><strong>Security Policy Control (AWS IAM integration)</strong></h3>
Returning to the security topic, another pro in favor of ECS/Fargate is the possibility to assign IAM instance profiles dedicated to the Container. What do these policies specify? Well basically access to all other AWS resources, such as S3 buckets, databases/datawarehouses such as DynamoDB, Aurora, Redshift, queing systems such as SQS and Kinesis, etc. You get the picture.
Again, the argument here is similar to the ENI: assigning dedicated IAM roles on a container level is order of magnitudes safer than on the EC2 instance level.
ECS and Fargate allow you to specify permissions at the container level. Currently, EKS does not allow you to so. This means you are limited to constrain permissions at the Workers/EC2 level.
<h3><strong>Container Registry - AWS ECR</strong></h3>
Nothing special here: as expected, all services integrate with AWS container registry, allowing you to pull from your internal AWS registry.
<h3><strong>Summary</strong></h3>
To conclusion, ECS and Fargate have a much tighter integration with AWS ecosystem. Kubernetes, on the other hand, has a much tighter integration with generic open source ecosystem, allowing you also to be more Cloud vendor agnostic.
As usual, here is a compilation of some of the useful sources used to write this post which might serve you as well:
<ul>
<li><a href="https://medium.com/containers-on-aws/choosing-your-container-environment-on-aws-with-ecs-eks-and-fargate-cfbe416ab1a" target="_blank" rel="noopener">Introduction comparison of AWS docker orchestration tools from Nathan Peck </a></li>
<li><a href="https://cloudonaut.io/eks-vs-ecs-orchestrating-containers-on-aws/" target="_blank" rel="noopener">Great technical summary from Cloudnaut</a></li>
<li><a href="https://medium.com/@remy.dewolf/aws-fargate-first-hands-on-experience-and-review-1b52fca2148e" target="_blank" rel="noopener">Fargate introduction with detailed GUI walkthrough</a></li>
<li><a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html" target="_blank" rel="noopener">AWS ECS/Fargate auto-scaling configuration documentation</a></li>
<li><a href="https://aws.amazon.com/blogs/compute/introducing-cloud-native-networking-for-ecs-containers/" target="_blank" rel="noopener">AWS new networking mode "awsvpc" for container level direct ENI mapping</a></li>
<li><a href="https://aws.amazon.com/blogs/compute/aws-fargate-a-product-overview/" target="_blank" rel="noopener">AWS blog post about Fargate</a></li>
<li><a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html" target="_blank" rel="noopener">AWS documentation for ECS task roles (container level)</a></li>
<li><a href="https://hackernoon.com/quickly-spin-up-an-aws-eks-kubernetes-cluster-using-cloudformation-3d59c56b292e" target="_blank" rel="noopener">Tutorial walk-through with AWS EKS and ClodFormation</a></li>
<li><a href="https://aws.amazon.com/blogs/aws/amazon-ecs-service-discovery/" target="_blank" rel="noopener">ECS supports service discovery</a></li>
<li><a href="https://aws.amazon.com/blogs/opensource/horizontal-pod-autoscaling-eks/" target="_blank" rel="noopener">EKS Horizontal auto-scaling</a></li>
<li><a href="https://aws.amazon.com/blogs/opensource/unified-service-discovery-ecs-kubernetes/" target="_blank" rel="noopener">Unified service discovery between ECS and EKS</a></li>
</ul>
