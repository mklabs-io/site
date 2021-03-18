---
layout: post
title: 'Spark - Redshift: AWS Roles to the rescue'
date: 2016-06-26 10:25:01.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- AWS
- Big Data
- Database
- Redshift
- Spark
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '24202546716'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2016/06/26/spark-redshift-aws-roles-to-the-rescue/"
---
If you are using AWS to host your applications, you probably heard that you can <a href="http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html">apply IAM Roles also to ec2 instances</a>. In a lot of cases this can be a really cool way to avoid passing AWS credentials to your applications, and having the pain of having to manage key distribution among servers, as well as ensuring key rotation mechanisms for security purposes.
This post is about a simple trick on how to take advantage of this feature when your Spark job needs to interact with AWS Redshift.
As can be read in <a href="https://github.com/databricks/spark-redshift#aws-credentials">Databricks repo for Spark-redshift library</a> the are three (3) strategies for setting up AWS credentials: either setup in hadoop configuration (how many people are used to so far with Cloudera or HortonWorks), encoding the keys in a tempdir (by far not the best option if you ask me), or using temporary keys. The last strategy is the one being discussed here, and its based on <a href="http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html">AWS own documentation how to use temporary keys</a>.<!--more-->
So let us start on Spark Redshift methods.
```{python}
from pyspark import SparkConf
from pyspark import SparkContext
from pyspark.sql import SQLContext
conf = SparkConf().setAppName(opts.get('app_name'))
sc = SparkContext(conf=conf)
ssc = SQLContext(sc)
rs_query = &amp;quot;select * from my_table limit 10&amp;quot;
rs_tmp_dir = 's3n://path/for/temp/data'
rs_url = 'jdbc:redshift://redshifthost:5439/database?user=username&amp;amp;amp;amp;amp;password=pass'
# Create spark dataframe through a Redshift query
df = ssc.read \
 .format('com.databricks.spark.redshift') \
 .option('url', rs_url) \
 .option('query', rs_query) \
 .option('tempdir', rs_tmp_dir) \
 .option('temporary_aws_access_key_id', sts_credentials.get('AccessKeyId')) \
 .option('temporary_aws_secret_access_key', sts_credentials.get('SecretAccessKey')) \
 .option('temporary_aws_session_token', sts_credentials.get('Token')) \
 .load()
```
&nbsp;
OK, until here almost only spark related code in Python.  Now the only thing you might be wondering is what the hell is that "sts_credentials" map?  Well spotted. The following snippet will reveal it.
```{python}
import requests
from requests.exceptions import HTTPError, Timeout
# Optionally define a default ec2 instance role for EMR instances
DEFAULT_INSTANCE_ROLE = 'STAGING_EMR_CLUSTER_ROLE'
def get_temp_credentials(role=DEFAULT_INSTANCE_ROLE):
  """ Retrieves temp AWS credentials """
  query_uri = 'http://169.254.169.254/latest/meta-data/iam/security-credentials/{}'.format(role)
  print('Querying AWS for credentials - {}'.format(query_uri))
  try:
sts_credentials = requests.get(query_uri).json()
    if isinstance(sts_credentials, dict) and \
sts_credentials.get('Code') == 'Success':
      print('Successfully retrieved temp AWS credentials.')
      return sts_credentials
print('There was a problem when retrieving temp credentials '
         'from AWS. Here\'s the response: \n{}'.format(sts_credentials))
   except (HTTPError, ConnectionError, Timeout) as err:
     msg = 'Unable to query AWS for credentials: {}'.format(err)
     print(msg)
   except ValueError as err:
     msg = 'Error: unable to decode json from \'None\' value - {} ' \
           '(hint: most likely the role you are using is wrong)'.format(err)
     print(msg)
   except Exception as err:
     msg = 'Failed to get AWS role temp credentials: {}'.format(err)
     print(msg)
# to use this function, you might do something like the following:
role = 'my-emr-cluster-role'
sts_credentials = get_temp_credentials(role=role)
aws_access_key_id = sts_credentials.get('AccessKeyId')
aws_secret_access_key = sts_credentials.get('SecretAccessKey')
token = sts_credentials.get('Token')
```
&nbsp;
Yes, essentially this peace of code is just doing an http request  to an AWS service. In case you're getting suspicious of the black magic looking address - "http://169.254.169.254/latest/meta-data" - let me tell you in advance that this is a handy service to <a href="http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html">provide meta-information about your instances</a>.
As a side note, and if you are indeed using pyspark jobs, you might want to give more flexibility whether your testing your code in local, or actually running the job in an AWS cluster. So, given the fact that the previous snippet will ONLY work if you run it on a AWS ec2 instance AND that instance is assigned the correct IAM role, here is a simple function which either passes credentials directly (if in local) or uses Role to ask for temp credentials to AWS.
```{python}
def get_redshift_credentials(role=DEFAULT_INSTANCE_ROLE,
                             local_aws_access_key_id=None,
                             local_aws_secret_access_key=None,
                             ):
    """ Returns temp AWS credentials present in an AWS instance
    Note: only works on AWS machines!
    :param role: (str) AWS instance role
    :param local_aws_access_key_id: (str) optional param for local testing/dev
    :param local_aws_secret_access_key: (str) optional param for local testing/dev
    :param opts: ()
    :return:
            (str) temp credentials to be used in query
    """
    if not role:
        role = DEFAULT_INSTANCE_ROLE
    sts_credentials = get_temp_credentials(role=role) or dict()
    aws_access_key_id = local_aws_access_key_id or sts_credentials.get('AccessKeyId')
    aws_secret_access_key = local_aws_secret_access_key or sts_credentials.get('SecretAccessKey')
    token = sts_credentials.get('Token')
    redshift_credentials = 'aws_access_key_id={0};aws_secret_access_key={1}' \
        .format(aws_access_key_id,aws_secret_access_key,token)
    if token and not all((local_aws_access_key_id, local_aws_secret_access_key)):
        redshift_credentials = '{0};token={1}'.format(redshift_credentials, token)
    return redshift_credentials
```
&nbsp;
<a href="https://gist.github.com/diogoaurelio/91e6086b53b5db7abd1cc5aac8a5de75" target="_blank" rel="noopener">Here is a complete gist of this code</a>:
https://gist.github.com/diogoaurelio/91e6086b53b5db7abd1cc5aac8a5de75
Finally some potential gotcha's: the temporary key have by default a 30 minutes limit, which you can extend initially when you request the temp keys.  In any case, you might want to take that into consideration on jobs that can potentially be long running.
&nbsp;
&nbsp;
&nbsp;
