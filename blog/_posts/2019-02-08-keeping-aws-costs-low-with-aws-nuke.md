---
layout: post
title: Keeping AWS costs low with AWS Nuke
date: 2019-02-08 17:19:06.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- AWS
- aws-nuke
tags: []
meta:
  timeline_notification: '1549646350'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '27423787914'
  _oembed_568b171c9062440a026623bbb98d5a95: "{{unknown}}"
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2019/02/08/keeping-aws-costs-low-with-aws-nuke/"
---
A common pattern in several companies using AWS services is having several distinct AWS accounts, partitioned not only by teams, but also by environments, such as develop, staging, production.
This can very easily explode your budget with not utilized resources. A classic example occurs when automated pipelines - think of terraform apply, or CI/CD procedures, etc - fail or time out, and all the resources created in the meanwhile are left behind.
Another frequent example happens in companies recently moving to the cloud. They create accounts for the sole purpose of familiarizing and educating developers on AWS and doing quick and dirty experiments. Understandably, after clicking around and creating multiple resources, it becomes hard to track exactly what was instantiated, and so unused zombie resources are left lingering around.
{:.lead}
<h2>AWS Nuke to the rescue</h2>
There are several options for tools to assist you with cleaning up AWS environment, such as from <a href="https://github.com/rebuy-de/aws-nuke" target="_blank" rel="noopener noreferrer">aws-nuke from rebuy-de</a> and <a href="https://github.com/gruntwork-io/cloud-nuke" target="_blank" rel="noopener noreferrer">cloud-nuke from gruntwork</a>. From the documentation aws-nuke supports destroying many more AWS resources compared to cloud-nuke, which was my intended use case. However cloud-nuke is being developed with a broader scope, with support for Azure and GCP in mind. When this post was released, this still remained a declaration of intentions though.
<a href="https://github.com/rebuy-de/aws-nuke" target="_blank" rel="noopener noreferrer">AWS Nuke</a> is quite easy and intuitive to work with. To install it:
https://gist.github.com/diogoaurelio/7d35868974e7acff613fe3ae8bdc2206
aws-nuke supports AWS CLI credentials, as well as profiles, something typical when managing several AWS accounts.  Here is how you can run a first scan of the resources to be deleted:
```
aws-nuke --config config.yaml --profile
```
&nbsp;
When you run it for the first time you might stumble upon the following message:
<em>aws-nuke version v2.7.0 - Fri Nov 23 10:28:30 UTC 2018 - 0b0806d56f85a329de6d1eedbf8559d46988a7f4</em>
<em>Error: The specified account doesn't have an alias. For safety reasons you need to specify an account alias. Your production account should contain the term 'prod'.</em>
<a href="https://stackoverflow.com/questions/54301200/unable-to-run-aws-nuke" target="_blank" rel="noopener noreferrer">As explained here</a>, the root cause might very likely be that you do not have a AWS account alias configured.
If things went as they should, you should be prompted with:
<em>Do you really want to nuke the account with the ID 12345678910 and the alias '&lt;your-aws-account-specific-profile&gt;'?</em>
<em>Do you want to continue? Enter account alias to continue.</em>
&nbsp;
At this point two things are important to be mentioned. The first is is that depending how many resources should be removed, this might take a while. The second is that no resource will be actually deleted at this point. You should get a message similar to the following:
<em>Scan complete: 436860 total, 436860 nukeable, 0 filtered. The above resources would be deleted with the supplied configuration. Provide --no-dry-run to actually destroy resources.</em>
Thus, to conduct the irreversible destruction:
```
aws-nuke --config config.yaml --profile  --no-dry-run
```
Here is an example of a config.yaml template to remove all resources except a given IAM:
https://gist.github.com/diogoaurelio/5d6e41f50d08b99546711c23993b5492
Alternatively you can also specify only some target resources to be removed. Here is an example:
https://gist.github.com/diogoaurelio/0422c3158d31926d9a4380d385fac837
That's all for today. For more information about aws-nuke, have a look in the <a href="https://github.com/rebuy-de/aws-nuke" target="_blank" rel="noopener noreferrer">repository</a>.
&nbsp;
&nbsp;
