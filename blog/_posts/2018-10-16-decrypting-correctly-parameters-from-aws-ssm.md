---
layout: post
title: Decrypting correctly parameters from AWS SSM
date: 2018-10-16 18:07:17.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- AWS
- AWS SSM
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '23260123319'
  timeline_notification: '1539713238'
  _oembed_6c5d0bc96d71a29d3e9ffd670008a28e: "{{unknown}}"
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2018/10/16/decrypting-correctly-parameters-from-aws-ssm/"
---
Today is yet short one, but ideally will already save a whole lot of headaches for some people.
Scenario: You have stored the contents of a string using AWS SSM parameter store (side note: if you are not using it yet, you should definitely have a look), but when retrieving it  decrypted via CLI, you notice that the string has new lines ('\n') substituted by spaces (' ').
In my case, I was storing a private SSH key encrypted to integrate with some Ansible scripts triggered via AWS CodePipeline + CodeBuild. CodeBuild makes it realy easy to access secrets stored in SSM store, however it was retrieving my key incorrectly, which in term domino-crashed my ansible scripts.
<a href="https://github.com/aws/aws-cli/issues/2596" target="_blank" rel="noopener">Here you can also confirm more people are facing this issue</a>. After following the suggestion of using AWS SDK - in my case with python boto3 - it <em>finally</em> worked. So here is a gist to overwrite an AWS SSM parameter, and then retrieving it back:
https://gist.github.com/diogoaurelio/372005295a4884456500bccaab3bf84b
Hope this helps!
