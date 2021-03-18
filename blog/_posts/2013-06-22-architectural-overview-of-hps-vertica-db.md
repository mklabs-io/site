---
layout: post
title: Architectural overview of HP's Vertica DB
date: 2013-06-22 17:01:52.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Big Data
- Datawarehouse
- HP Vertica
tags:
- DB
meta:
  _edit_last: '51216153'
  tagazine-media: a:7:{s:7:"primary";s:62:"http://datacenternotes.files.wordpress.com/2013/06/vertica.png";s:6:"images";a:1:{s:62:"http://datacenternotes.files.wordpress.com/2013/06/vertica.png";a:6:{s:8:"file_url";s:62:"http://datacenternotes.files.wordpress.com/2013/06/vertica.png";s:5:"width";i:527;s:6:"height";i:415;s:4:"type";s:5:"image";s:4:"area";i:218705;s:9:"file_path";b:0;}}s:6:"videos";a:0:{}s:11:"image_count";i:1;s:6:"author";s:8:"51216153";s:7:"blog_id";s:8:"53483832";s:9:"mod_stamp";s:19:"2013-06-22
    19:20:44";}
  _publicize_job_id: '25330274012'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/06/22/architectural-overview-of-hps-vertica-db/"
---
If you start digging for more info on HP Vertica, chances are high that you stumble uppon this image, which summarizes it's key selling points:<a style="font-style:inherit;line-height:1.625;" href="http://datacenternotes.files.wordpress.com/2013/06/vertica.png"><img class="alignnone size-full wp-image-206" src="{{ site.baseurl }}/assets/2013/06/vertica.png" alt="Vertica" width="527" height="415" /></a>
So the first thing one should start talking about should always be the origin of Vertica's name. Instead of horizontal data population, Vertica stores in a verticaL disposition - a columnar storage of data (imagine a 90 degree turn on you DB, where the properties of each row are now the main agents to consider). The goal is to benefit both in querry as in storage capabilities. Allocation in columnar fashion allows to sort data (no need for indexing, since data is already sorted), which is then followed by an encoding process that allows to store only one time each unique value, as well as how many times that value repeats itself.
Vertica prefers enconding instead of compressing data. You can still querry data while it is on its enconded state. Vertica applies different encoding mechanisms in the same table to different Columns, accordingly . You can also apply both encoding and compression. to the type of data stored. Besides using encoding to provide additional query speed, storage reduction is also in stake. <span style="font-style:inherit;line-height:1.625;">Vertica claims a 4:1 reduction of storage footprint. </span>
Another important differentiator is Vertica's distributed grid-like architecture. Likewise Big Data architectures, instead of having hercule muscled machine, you are able to gain the advantages of multiple smaller and, most importantly, less expensive computing nodes working in parralel. In other words, you use a Massively-parallel processing (MPP) system -grid-base architecture clustering a bunch of x86-64 servers. So it is a shared-nothing architecture - each node functions as if it were the only node, and uses its own processing capabilities to return part of the answer of a query that involves that data that it owns.
How to design such a system? Each node has a minimum hardware recommendation of two quad-core CPU, and 16GB of RAM, 1TB of disk space and two VLANs assigned - one for client connection, another for private inter-node communication. In the private inter-node communication VLAN is where nodes distribute queries along them, obtaining parallel processing.
You might be wondering by now, what would happen if you were to loose one or more of those commondity hardware servers. The Db provides all the HA mechanisms you need by spreading data repeatadidly across several nodes. We are talking about a RAID-like storage of the same data at least twice in different nodes, thus eliminating SPOFs and tolerating node/s failure. To do so, Vertica does table segmentation, distributing tables evenly across different nodes. The result: you get a clean HA, without any need for manual log-based recovery.
Queries are indeed run on all nodes, so if one node is done, you will suffer on performance (comparing to multi-node performance, not other DBs!), since at least one node will have to do double work on failure. If you lose enough nodes to loose at least one segment of data, Vertica goes automatically down so not to corrupt the DB.
Also quite neat is the auto-recover process. When a failing node comes back online, other nodes automatically sync data.
As you can see, the answer to the following question is <em>yes</em>, ou can forget about your centrallized SAN environment, and using your big-ass Storage Array. Each node uses local disk which also allows better performance, as redundancy is guaranteed at the DB-level.
Another important aspect is that you can load your DB schema into Vertica, and it uses what they call "Automatic DB designer" to facilitate the whole process of DB design. You can you the designer repeatadily in different points of time, as you evolve to adjust.
Finally Vertica has a standard SQL interface, and supports SQL, ODBC, JDBC and the majority of ETL and BI reporting products.
Disclamer: note that these are <em>my</em> own notes. HP is not responsable for any of the content here provided.
