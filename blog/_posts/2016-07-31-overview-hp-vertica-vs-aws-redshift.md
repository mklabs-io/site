---
layout: post
title: Overview HP Vertica vs AWS Redshift
date: 2016-07-31 10:56:31.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- AWS
- Big Data
- Datawarehouse
- HP Vertica
- Redshift
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '25330011256'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2016/07/31/overview-hp-vertica-vs-aws-redshift/"
---
While working in HP some years ago, I was exposed to not only internal training materials, but also a demo environment. I still remember the excitement when <a href="https://techcrunch.com/2011/02/14/hp-acquires-data-management-and-real-time-analytics-company-vertica/" target="_blank" rel="noopener">HP acquired Vertica Systems in 2011</a>, and we had a new toy to play with... Come on, you can't blame me, distributed DBs was something only the cool kids were doing.
Bottom line is that it's been a while since I laid eyes on it... Well recently, while considering possible architectural solutions, I had the pleasure  to revisit Vertica. And since AWS Redshift has been gaining a lot of popularity and <a href="http://berlinsmartdata.de/" target="_blank" rel="noopener">we're also using it</a> at some of our clients, I thought I might give some easy summary to help others.
Now if you're expecting a smack down post, then I'm afraid I'll disappoint you - for that you have the <a href="http://stackoverflow.com/questions/8987727/advantages-of-databases-like-greenplum-or-vertica-compared-to-mongodb-or-cassand" target="_blank" rel="noopener">internet</a>. If experience has taught me  something is that - in the case of top-notch solutions there are only use cases, and one finds the best fitting one.<!--more-->
They share some properties in terms of architecture in internal engine:
<ul>
<li><strong>Massively Parallel Processing (MPP) architecture</strong>: data is distributed among distinct nodes in shared nothing architecture, leveraging scale out and; in case you're wondering how it compares to hadoop Hive, <a href="http://nerds.airbnb.com/redshift-performance-cost/" target="_blank" rel="noopener">AirBnB</a> did a smack-down comparison, concluding around 5x advantage of Redshift over Hive;</li>
<li><strong>High availability (HA)</strong>: this follows the first, thanks to to data replication mechanism; in the case of Vertica, they call it"k-safety" for measuring replication factor; and you may also want to check <a href="https://my.vertica.com/docs/7.1.x/HTML/index.htm#Authoring/ConceptsGuide/Components/HighAvailabilityWithFaultGroups.htm?Highlight=Fault Groups" target="_blank" rel="noopener">fault groups</a> to control how data is replicated according to physical distribution (such as server rack location, power circuits, etc); same <a href="http://www.allthingsdistributed.com/2013/02/amazon-redshift-resilience.html" target="_blank" rel="noopener">automatic data replication among nodes happens with Redshift</a> under the hood (besides more goodies such as backups)</li>
<li><strong>columnar oriented data store</strong>: for analytical applications/OLAP (where queries usually select only specific columns in opposition to OLTP), it is usually much faster mainly due to <strong>a)</strong> does not need to scan whole row and then discard the unnecessary content, <strong>b)</strong> higher efficiency in compression/encoding mechanisms due to similar data types; for more detailed explanation, I suggest <a href="http://www.advantisolutions.com/advantages-of-column-stores-over-row-stores-for-data-warehousing-and-analytical-applications/" target="_blank" rel="noopener">here</a>. Both <a href="https://my.vertica.com/get-started-vertica/architecture/" target="_blank" rel="noopener">Vertica</a> and <a href="http://docs.aws.amazon.com/redshift/latest/dg/c_columnar_storage_disk_mem_mgmnt.html" target="_blank" rel="noopener">Redshift</a> are built with this architecture;</li>
<li><strong>Data compression</strong>: Vertica mixes encoding strategies, depending on column data type, table cardinality, and sort order; they do distinguish a difference between encoding and compression, since it will operate directly on encoded data whenever possible, which does not hold true for compression; <a href="http://docs.aws.amazon.com/redshift/latest/dg/t_Compressing_data_on_disk.html" target="_blank" rel="noopener">Redshift</a> does not make such a distinction, and recommends leaving compression to auto mode, although you can choose encoding type;</li>
<li><strong>SQL standard interface</strong>: as always, minor differences are present, but bottom line you can use the SQL syntax that you're already accustomed to</li>
<li><strong>User Defined Functions</strong> (UDFs): both <a href="http://docs.aws.amazon.com/redshift/latest/dg/user-defined-functions.html" target="_blank" rel="noopener">Redshift</a> and <a href="https://my.vertica.com/docs/7.0.x/HTML/Content/Authoring/ProgrammersGuide/UserDefinedFunctions/TypesOfUDFs.htm" target="_blank" rel="noopener">Vertica</a> give you space for customization;</li>
<li><strong>In-memory DB</strong>: Nop, none of these are like SAP HANA, Oracle TimesTen or IBM's SolidDB</li>
</ul>
Where they differ:
<ul>
<li><strong>Architecture</strong>: in Vertica all nodes are "created equally", meaning they share similar functions; <a href="http://docs.aws.amazon.com/redshift/latest/dg/c_internal_arch_system_operation.html" target="_blank" rel="noopener">Redshift</a> has the concept of a leader node, a dedicated node which manages workload and query coordination among worker nodes;</li>
<li><strong>Management</strong>: (this is a key differentiator that most likely that the biggest weight in the final decision) with Vertica you have to do all the ops work (install, upgrade/update, configure nodes, etc.);  Redshift is a fully managed Cloud solution, where you only have pure Database related ops work; Note: yes, HP provides an AMI to easily kickstart projects in AWS cloud, but come on, this is still <em>not</em> the same thing;</li>
<li><strong>Freedom of environment</strong>: with Redshift you're locked in to AWS; with Vertica you can run it wherever you feel like;</li>
<li><strong>Schema Design</strong>: Vertica provides you with a Designer Tool to easily migrate from traditional RDBMS systems based on their schema (not saying this saves the world, but can be helpful); this is specially important in the beginning, since columnar data warehouses don't support indexes; so in Vertica you play with the projection concept, in Redshift with distribution and sort keys (and you better do it well, as it will be key for performance and keeping things balanced)</li>
<li><strong>Payment scheme</strong>: In Vertica (for data bigger than 1TB, which most certainly is the case) you pay upfront licensing, plus the cost of the machines where you're running them; with Redshift costs all diluted into an hourly cost, and that's it;</li>
<li><strong>Compiled code</strong>: Redshift claims that the leader node compiles the code for optimal performance on execution time, which <a href="http://techblog.getcake.com/redshift-vs-vertica/#sthash.vEYRh7lD.dpuf" target="_blank" rel="noopener">the guys at Cake also confirm with this excellent post</a>;</li>
<li><strong>Free Trial/usage</strong>: <a href="https://my.vertica.com/in-the-cloud/" target="_blank" rel="noopener">Vertica</a> lets you have up to 3 nodes and 1TB of data;  <a href="https://aws.amazon.com/redshift/free-trial/" target="_blank" rel="noopener">Redshift</a>, on the other hand, in case you're account is still electable for the free usage tier (in the first year), you can try for a total of 750 normalized instance hours per month, enough for running continuously one DC1.Large single node, with 160TB SSD storage</li>
<li><strong>Add-Ons</strong>: Vertica Pulse for sentiment analysis and Place for geospatial data analysis;</li>
</ul>
Finally, you might want to go deeper. Again, I really suggest <a href="http://techblog.getcake.com/redshift-vs-vertica/" target="_blank" rel="noopener">this excellent post by Cake, which provides performance benchmarks</a>. Benchmarks are always disputable; but still, it is always interesting and important comparison method.
