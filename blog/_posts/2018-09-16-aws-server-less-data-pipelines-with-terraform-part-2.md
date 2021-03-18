---
layout: post
title: AWS Server-less data pipelines with Terraform to Redshift - Part 2
date: 2018-09-16 15:08:18.000000000 +02:00
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
  _publicize_job_id: '22206278020'
  timeline_notification: '1537110499'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2018/09/16/aws-server-less-data-pipelines-with-terraform-part-2/"
---
Alright, it's time for the second post of our sequence focusing on AWS options to setup pipelines in a server-less fashion. The topics that we are covering throughout this series are:
<ul>
<li><a href="https://datacenternotes.com/2018/09/01/aws-server-less-data-pipelines-with-terraform-part-1/" target="_blank" rel="noopener">Part 1: Python Lambda to load data into AWS Redshift datawarehouse</a></li>
<li>Part 2: Terraform setup of Lambda function for automatic trigger</li>
<li>Part 3: Example AWS Step function to schedule a cron pipeline with AWS Lambda</li>
</ul>
In this post we complement the previous one, by providing infrastructure-as-code with <a href="https://www.terraform.io/" target="_blank" rel="noopener">Terraform</a> for deployment purposes. We are strong believers of a DevOps approach also to Data Engineering, also known as "DataOps". Thus we thought it would make perfect sense to share a sample Terraform module along with Python code.
To recap, so far we have Python code that, if triggered by a AWS event on a new S3 object, will connect to Redshift, and issue SQL Copy command statement to load that data into a given table. Next we are going to show how to configure this with Terraform code.
As usual, all the code for this post is <a href="https://github.com/diogoaurelio/serverless-pipelines-blog-series" target="_blank" rel="noopener">available publicly in this github repository</a>. In case you haven't yet, you will need to install terraform in order follow along this post.
{:.lead}
<strong>Terraform setup</strong>
First lets start with our proposed structure. We have found it useful to distinguish and isolate dedicated Terraform code according to the environment it needs to be deployed in. An environment can mean different things to different companies. The most common environments are  dev/staging/production, which allow one to safely test code in a controlled setup. However other use cases exist, such as different VPCs or AWS accounts across different tech teams.
For this reason, we actually require one to export ENVIRONMENT variable in the beginning, which is used as a key for the Makefile to find the correct directory to deploy.
```
export ENVIRONMENT=dev
# create a file for terraform secret variables hosting: "$(ENVIRONMENT).tfvars", which our Makefile expects
touch terraform/environments/dev/dev.tfvars
# Verify what the terraform code plans to deploy
make plan
```
The make plan command will allow you to get started. Note that Terraform will use your default credentials that you have configured in "~/.aws/credentials". If you want to use different, one simple way (among several) is to export aws access key and secret values before running the command.
Whenever one runs "make plan", this will run three terraform commands: "terraform init", "terraform update" and "terraform plan". The terraform "init" and "update" are specially useful for the first time you initialize terraform, downloading all specific provider packages you require, and updating whenever you add new requirements.
At the time of writing this blog post, here is what I have got printed out when running make plan for the first time:
&nbsp;
```
Initializing modules...
- module.redshift_loader_lambda
  Getting source "github.com/diogoaurelio/terraform-aws-lambda-module"
Initializing provider plugins...
- Checking for available provider plugins on https://releases.hashicorp.com...
- Downloading plugin for provider "aws" (1.36.0)...
- Downloading plugin for provider "archive" (1.1.0)...
The following providers do not have any version constraints in configuration,
so the latest version was installed.
To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.
* provider.archive: version = "~&gt; 1.1"
* provider.aws: version = "~&gt; 1.36"
Terraform has been successfully initialized!
```
&nbsp;
Now, since we have not yet filled our "dev.tfvars", you will notice short after the following errors:
```
##### 1) VPC details: VPC ID and private subnets to use (where the lambda will run)
Error: Required variable not set: private_subnet_ids
Error: Required variable not set: vpc_id
##### 2) Redshift DB details
# The IAM Redshift Role used to issue COPY commands and load data into Redshift
Error: Required variable not set: redshift_data_loader_lambda_iam_role
# The intended Redshift dabase name
Error: Required variable not set: redshift_data_loader_lambda_db_name
# Redshift endpoint or Route53 record to use
Error: Required variable not set: redshift_data_loader_lambda_db_host
# Redshift DB user to use
Error: Required variable not set: redshift_data_loader_lambda_db_user
# Redshift User password to use
Error: Required variable not set: redshift_data_loader_lambda_db_password
# The ARN and name of the bucket where data is being stored (which should be loaded into Redshift)
Error: Required variable not set: s3_bucket_arn
Error: Required variable not set: s3_bucket_name
```
&nbsp;
The "*.tfvars" are gitignored. You should now add the details of your own setup into your "dev.tfvars", in case you want to test drive this code. After you do this, run again "make plan" to see what terraform plans to deploy.
&nbsp;
<strong>Core AWS Lambda function module </strong>
OK, before we deploy, let us have a look on the terraform code,  starting with the Lambda function. Here we are using a specific version of a generic module that we had previously created for Lambda functions. The beauty of this module is its flexibility. For example, you can launch the Lambda inside or outside a VPC, you can attach how many additional IAM policies you desire, and specify a SNS or SQS ARN for Dead Letter Queue (DLQ) for failure handling.
https://gist.github.com/diogoaurelio/4c4ef99dfa87dd7fb1d9b6381909f217
The first thing you can notice is the usage of a different <a href="https://github.com/diogoaurelio/terraform-aws-lambda-module" target="_blank" rel="noopener">dedicated git repository via the "source" keyword</a> for the Lambda module. Go ahead and have a look, this is also <a href="https://github.com/diogoaurelio/terraform-aws-lambda-module" target="_blank" rel="noopener">public, and we have been using it successfully in several projects</a>. Also note that one can pin point different versions of a given repository, which maps to repository tags. This reassures retro-compatibility and non breaking  progress on each module.
The second thing you might notice is that we are passing paths of the Lambda function. The reason behind it is that our terraform lambda module will proactively zip our lambda's "src" module; <a href="https://github.com/diogoaurelio/serverless-pipelines-blog-series" target="_blank" rel="noopener">in our repository located in</a>: "etl/lambda/redshift/src".
The second is that we are instantiating the lambda inside a VPC. As mentioned before,  this is optional - the module is flexible enough for you to chose if you want to have the lambda being executed inside a specific VPC or not. In our case, we are assuming that your <em><span style="text-decoration:underline;">Redshift cluster is indeed inside a private subnet</span></em> (as advisable), and thus we need to execute the Lambda function inside that VPC to be able to connect to Redshift.
Please remember to either open Redshift's Security Group either to the specified subnet CIDR, or to our Lambda's security group id.
<strong>Lambda environment variables</strong>
Next point up the list are environmental variables. The module accepts those via the variable "<em>lambda_env_vars</em>". We have defined those via terraform "<a href="https://www.terraform.io/docs/configuration/locals.html" target="_blank" rel="noopener">locals</a>", with <span class="pl-s">"</span><span class="pl-s1"><span class="pl-e">${</span><span class="pl-e">local.redshift_loader_lambda_env_vars</span><span class="pl-e">}</span></span><span class="pl-s">". Let us have a look at those briefly:</span>
https://gist.github.com/diogoaurelio/b0cb1bfb0ae86df671c7865f32bbb772
We are passing mainly all parameters which the lambda function uses to connect to Redshift DB. Note also that we are not passing the explicit Redshift DB password, but rather the name of the parameter stored in <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-paramstore.html" target="_blank" rel="noopener">SSM store</a>.
You may ask yourself why we are doing this, considering that this lambda module is already encrypting environmental variables by default. Yes, the lambda module creates automatically a KMS key for this purpose. We are doing it in the end more for illustration purposes, although one could argue that concealing these variables in the AWS GUI is also relevant so that not all users that have access to it can discover the password.
&nbsp;
<strong>Lambda permissions</strong>
You will verify that we simplified this code in several aspects. A good example are the broad permissions passed to the lambda function for all buckets, among others. A (much) better approach would be to clearly restrict to a small white list of resources. But, for the sake of the tutorial, we will relax on some of these aspects that you should consider when bringing this inside your company.
https://gist.github.com/diogoaurelio/5f8628684126230d1b72fac55685d76d
Note that we are passing permissions to use KMS keys, which is specially relevant if your buckets are encrypted.
Moreover, we are also passing permission to access the SSM parameter store specific DB password.
The final statement provides the Lambda function permissions to publish in a specific SNS topic, which leads us to the final point: failure handling.
<strong>Lambda Failure handling</strong>
Currently Lambda functions executed asynchronously - as is the case of S3 triggered Lambdas - will always<strong><span style="text-decoration:underline;"> retry twice</span></strong> on failure. the only thing you have to do is to configure the Dead Letter Queue (DLQ) configured, by passing a SNS topic to publish in case the maximum number of retries is reached. Note: Please make sure you always double check <a href="https://docs.aws.amazon.com/lambda/latest/dg/dlq.html" target="_blank" rel="noopener">in AWS documentation</a> for updates to find the current limits and default behaviors.
In our case, we have configured a SNS topic, so that subscribers can subscribe and be alerted of failure. For example, one can add an email subscriber, and be notified on Lambda failure. Unfortunately,  Terraform does not support the creation of SNS Email subscriptions as <a href="https://www.terraform.io/docs/providers/aws/r/sns_topic_subscription.html" target="_blank" rel="noopener">explained here</a>. Thus either subscribe manually, or use CloudFormation/custom script for this task.
&nbsp;
<strong>Deploying the code</strong>
Cool, we are ready to rock&amp;roll. Just open your terminal and type:
```
export ENVIRONMENT=dev
make apply
```
This will prompt for confirmation, before deploying.
Go ahead and create a table in Redshift, such as for example "kpi". Next upload a new CSV file into your S3 bucket with a prefix "kpi" (so that a key "kpi/your_file.csv" is saved in the bucket). You should be able to see data in your Redshift table in less than a minute. Moreover, you can also consult your Cloudwatch logs, and verify that the Lambda function has executed successfully.
Congratulations, you have deployed a asynchronous pipeline using infrastructure-as-code.
&nbsp;
As usual, let us summarize the sources used in this post:
<ul>
<li><a href="https://github.com/diogoaurelio/serverless-pipelines-blog-series" target="_blank" rel="noopener">Source repository for this blog post</a></li>
<li><a href="https://github.com/diogoaurelio/terraform-aws-lambda-module" target="_blank" rel="noopener">Source repository for the lambda module</a></li>
<li><a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-paramstore.html" target="_blank" rel="noopener">AWS SSM parameter store for storing secrets encrypted</a></li>
<li><a href="https://docs.aws.amazon.com/lambda/latest/dg/dlq.html" target="_blank" rel="noopener">AWS Lambda DLQ documentation</a></li>
</ul>
&nbsp;
We hope you found this tutorial useful. Please do not hesitate to me - diogo [at] gosmarten [dot] com - if you need any help on your project, be it cloud/devOps/Big Data/ML.
&nbsp;
&nbsp;
