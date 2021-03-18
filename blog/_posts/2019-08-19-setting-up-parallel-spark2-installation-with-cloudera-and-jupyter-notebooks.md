---
layout: post
title: Setting up parallel Spark2 installation with Cloudera and Jupyter Notebooks
date: 2019-08-19 20:43:05.000000000 +02:00
type: post
categories:
- cloudera
- Hadoop
- Spark
tags: []
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
---

Yes, this topic is far from new material. Especially if you consider Cloud tech stack evolution/change speed, it has been a long time since Apache Spark version 2 was introduced (26-07-2016, to be more precise). But moving into the cloud is not an easy solution for all companies, where data volumes can make such a move prohibitive. And in on-premises contexts, the speed of operational change is significantly slower.
{:.lead}

This post summarizes the steps for deploying Apache Spark 2 alongside Spark 1 with <a href="https://www.cloudera.com/" target="_blank" rel="noopener">Cloudera</a>, and install python jupyter notebooks that can switch between Spark versions via kernels. Given that this is a very frequent setup in big data environments, thought I would make the life easier for "on-premise engineers", and, hopefully, speed up things just a little bit.

That is exactly the case of one of the current projects where I am working on, where a volume of 10 TB/day is the minimum norm. It is in these scenarios where vendors such as <a href="https://www.cloudera.com/" target="_blank" rel="noopener">Cloudera</a> (now merged with <a href="https://hortonworks.com/" target="_blank" rel="noopener">HortonWorks</a>) and <a href="https://mapr.com/" target="_blank" rel="noopener">MapR</a> (now acquired by Hewlett and Packard) have their sweet spot.
<h3>Preparation</h3>
So let's start with Cloudera setup. First, make sure <a href="https://www.cloudera.com/documentation/spark2/latest/topics/spark2_requirements.html" target="_blank" rel="noopener">you meet all requirements</a> before you proceed. Note that some might be optional, such as scala. In my case only pyspark was required, so scala was not required to be setup on the cluster.
Just to be on the safe side have also a look on the <a href="https://www.cloudera.com/documentation/spark2/latest/topics/spark2_new_features.html" target="_blank" rel="noopener">release notes</a> and <a href="https://www.cloudera.com/documentation/spark2/latest/topics/spark2_incompatible_changes.html" target="_blank" rel="noopener">incompatible changes pages</a>. This should also help you choosing the Spark 2 version you intend to setup, and make sure all features you required are there.
Last but not least, do note that, although Cloudera allows you to set Spark1 and Spark2 in parallel on the same cluster, it does not support having multiple Spark2 versions. This is understandable, as default paths could become a mess.
<h3>Install add-on service &amp; parcel</h3>
If everything looks good so far, it is time to familiarize yourself with the <a href="https://www.cloudera.com/documentation/spark2/latest/topics/spark2_installing.html" target="_blank" rel="noopener">installation process.</a> Whenever ready, the first step is to download to your cloudera manager instance/server the respective <a href="https://www.cloudera.com/documentation/spark2/latest/topics/spark2_packaging.html" target="_blank" rel="noopener">Custom Service Descriptor</a> (CSD). CSDs are metadata files that describe a product for use with Cloudera (for more information about CSDs: <a href="https://www.cloudera.com/documentation/director/2-6-x/topics/director_non-cdh_products_custom_descriptors.html" target="_blank" rel="noopener">check here)</a>.  Put the CSD jar on cloudera local configured location, which by default is /opt/cloudera/csd.  Now follow the instructions provided by cloudera to install an <a href="https://www.cloudera.com/documentation/enterprise/latest/topics/cm_mc_addon_services.html#concept_qbv_3jk_bn__section_xvc_yqj_bn" target="_blank" rel="noopener">add-on service</a>.
If all went well, it is time to download Spark2 parcel. This can be done directly on Cloudera Manager GUI. In parcels configuration, add the respective Spark2 URL. In my case, I was using 2.4.0, so the URL is: http://archive.cloudera.com/spark2/parcels/2.4.0.cloudera2/. Naturally you can also specify: http://archive.cloudera.com/spark2/parcels/latest/ . You should download, activate and distribute the parcel on all hosts of your cluster where you want to setup Spark.
In cloudera home page you should now already see Spark2 listed among the different services, similarly as follows:
<img class="  wp-image-4289 aligncenter" src="{{ site.baseurl }}/assets/2019/08/cloudera_spark2.jpg" alt="cloudera_spark2" width="560" height="224" />
Click on Spark2 service. On the service page, click on "Actions" button, and "Add Role Instances". The first and mandatory role you will need to setup is Spark History server. If you already have a Spark History server (for Spark1) on the same host installed, make sure you install it in a non-conflicting port, which will be 18089 instead of the usual 18088. Also note that Spark history server will attempt to write application history into HDFS, by default in this localtion: "/user/spark/spark2ApplicationHistory". Make sure permissions are correctly assigned for the respective user. For example:
[code language="shell"]
hdfs dfs -chown spark:spark /user/spark/spark2ApplicationHistory
hdfs dfs -chmod 755 /user/spark/spark2ApplicationHistory
```
Finally deploy Spark2 gateway role on all hosts you wish to run Spark 2. <a href="https://www.cloudera.com/documentation/enterprise/latest/topics/cm_mc_managing_roles.html#managing_roles__section_scv_ywt_cn" target="_blank" rel="noopener">Gateways</a> are a special type of role that just indicates Cloudera that the respective cluster configurations should be installed and updated.  Deploy the Configuration update on clients, and restart stale services. Congratulations, you have Spark2 setup on you Cloudera cluster.
<h3>Jupyter Notebooks environment</h3>
The ability to quickly test and analyse your data is essential to develop any decent machine learning algorithm. Jupyter notebooks are the current <em>de facto standard</em> for this purpose. So, the following script is intended to be set in any server that data scientists/engineers use as development environment, which is part of your cloudera cluster. Naturally, make sure this server also has Spark2 gateway installed.
Data scientists/engineers can run the script to install a local python virtual environment, along with a jupyter kernel that has Cloudera Spark2 path correctly setup.
https://gist.github.com/diogoaurelio/ef249b28afb5401d6bd9bf2dfb497e8d
To run the script:
[code language="shell"]
# ./setup_jupyter_notebook.sh
./setup_jupyter_notebook.sh mySuperSecretNotebookPassword 3.7
```
By default the script will install in the current's user home folder issuing the command. So to run the jupyter notebook:
[code language="shell"]
cd &amp;&amp; . venv_notebooks/bin/activate &amp;&amp; jupyter notebook
```
In the kernels tab you should be able to see a kernel with the name "PySpark2" listed, to which you should switch to. And that is it.
Make sure you do give a closer look to the script, and adjust it to your particular needs. For example, you might want to have a closer look on the paths set in the ipykernel.
As usual, here a summary of the key sources listed in this blog post:
<ul>
<li><a href="https://www.cloudera.com/documentation/spark2/latest/topics/spark2_requirements.html" target="_blank" rel="noopener">Cloudera Spark requirements</a></li>
<li><a href="https://www.cloudera.com/documentation/spark2/latest/topics/spark2_packaging.html" target="_blank" rel="noopener">Cloudera Spark2 list of Custom Service Descriptors</a></li>
<li><a href="https://www.cloudera.com/documentation/spark2/latest/topics/spark2_installing.html" target="_blank" rel="noopener">Cloudera Spark2 installation process</a></li>
</ul>
&nbsp;
Hope this helps!
