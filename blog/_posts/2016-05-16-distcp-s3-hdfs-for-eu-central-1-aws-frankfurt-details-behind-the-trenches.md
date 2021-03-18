---
layout: post
title: Distcp S3-hdfs for eu-central-1 (AWS Frankfurt) - details "behind the trenches"
date: 2016-05-16 14:30:02.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- AWS
- Big Data
- Hadoop
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '22865925243'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2016/05/16/distcp-s3-hdfs-for-eu-central-1-aws-frankfurt-details-behind-the-trenches/"
---
&nbsp;
Distcp (distributed copy) is a fairly old tool used to move a large quantity of files usually within hdfs, using MapReduce job to do so where mappers list the source files and reducers do the copy heavy lifting. Another useful integration is that it can also deal with file migrations between hdfs and AWS S3. In the meanwhile S3Distcp came along as an specific enhancement for speeding up these last kind of migrations.
However, what might be new for you is the requirement to use either one of these tools to move data from/to a bucket located in S3 Frankfurt region.   So this post intends to save you that half an hour of painful google research trying to figure out why your Oozie job is failing.
Without further ado, before you dive in to your Cloudera or Hortonworks Manager to edit hadoop properties in core-site.xml, I would suggest that you SSH to one of your hadop nodes, and start by testing running distcp command. Here is the sintax for moving data from S3 to HDFS:
```
hadoop distcp -Dfs.s3a.awsAccessKeyId=XXX \
-Dfs.s3a.awsSecretAccessKey=XX \
-Dfs.s3a.endpoint=s3-eu-central-1.amazonaws.com s3a://YOUR-BUCKET-NAME/ \
hdfs:///data/
```
So yeah, in case you haven't noticed, the black magic pieces in this command that makes things work just like in other regions are: the "fs.s3a.endpoint" property, as well as the "a" after "s3".
Also in case you are moving data from hdfs to S3, besides write access, make sure you also give remove permissions on your AWS policy attaches to  the given user account on for that S3 bucket (or specific path) . This is because Distcp produces temporary files adjacent to your bucket target file migration, which you want to allow the reducers to remove in the end. Here is an example of an AWS policy for that effect:
[code language="javascript"]
{
    &quot;Version&quot;: &quot;2012-10-17&quot;,
    &quot;Statement&quot;: [
        {
            &quot;Effect&quot;: &quot;Allow&quot;,
            &quot;Action&quot;: [
                &quot;s3:ListAllMyBuckets&quot;,
                &quot;s3:GetBucketLocation&quot;
            ],
            &quot;Resource&quot;: &quot;arn:aws:s3:::*&quot;
        },
        {
            &quot;Effect&quot;: &quot;Allow&quot;,
            &quot;Action&quot;: [
                &quot;s3:ListBucket&quot;,
                &quot;s3:GetBucketLocation&quot;,
                &quot;s3:GetObject&quot;,
                &quot;s3:PutObject&quot;,
                &quot;s3:DeleteObject&quot;
            ],
            &quot;Resource&quot;: [
                &quot;arn:aws:s3:::YOUR-BUCKET/*&quot;,
                &quot;arn:aws:s3:::YOUR-BUCKET&quot; ]
}
]
}
```
Alternatively you might also want to test it using S3Distcp tool. Here is another example:
``` hadoop jar s3distcp.jar --src /data/ \
--dest s3a://YOUR-BUCKET-NAME/ \
--s3Endpoint s3-eu-central-1.amazonaws.com
```
Note that the s3distcp jar needs to be locally on the host file system from which you are running the command. If you have the jar in hdfs, here is an example how you can fetch it:
```
hadoop fs -copyToLocal /YOUR-JARS-LOCATION-IN-HDFS/s3distcp.jar \
/var/lib/hadoop-hdfs/
```
Also there is also the possibility you might stumble uppon some headaches with S3Distcp tool. <a href="http://stackoverflow.com/questions/14631152/copy-files-from-amazon-s3-to-hdfs-using-s3distcp-fails">This</a> might prove itself to be handy.
Hope this helped! Where to go next? AWS has a handy best practice guide listing different strategies for data migration <a href="https://media.amazonwebservices.com/AWS_Amazon_EMR_Best_Practices.pdf">here</a>.
&nbsp;
&nbsp;
