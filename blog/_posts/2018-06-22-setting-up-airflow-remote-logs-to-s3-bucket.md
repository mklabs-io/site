---
layout: post
title: Setting up Airflow remote logs to S3 bucket
date: 2018-06-22 18:08:15.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Airflow
- devOps
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '19245603252'
  timeline_notification: '1529690896'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2018/06/22/setting-up-airflow-remote-logs-to-s3-bucket/"
---
Today is a short one, but hopefully a valuable devOps tip, if you are currently setting up remote logging integration to S3 of Airflow logs using Airflow version 1.9.0.
Basically <a href="https://stackoverflow.com/a/48194903/9979495" target="_blank" rel="noopener">this stackoverflow post</a> provides the main solution. However, the current template incubator-airflow/airflow/config_templates/airflow_local_settings.py present in master branch contains a reference to the class "airflow.utils.log.s3_task_handler.S3TaskHandler", which is not present in apache-airflow==1.9.0 python package. The fix is simple - use rather this base template: <a href="https://github.com/apache/incubator-airflow/blob/v1-9-stable/airflow/config_templates/airflow_local_settings.py" rel="nofollow noreferrer">https://github.com/apache/incubator-airflow/blob/v1-9-stable/airflow/config_templates/airflow_local_settings.py</a> (and follow all other instructions in the <a href="https://stackoverflow.com/a/48194903/9979495">mentioned answer</a>)
Hope this helps!
