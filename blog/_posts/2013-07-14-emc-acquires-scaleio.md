---
layout: post
title: EMC acquires ScaleIO
date: 2013-07-14 17:53:23.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Cloud
- Storage
tags:
- EMC
- ScaleIO
meta:
  _edit_last: '51216153'
  geo_public: '0'
  _publicize_pending: '1'
  tagazine-media: a:7:{s:7:"primary";s:72:"http://datacenternotes.files.wordpress.com/2013/07/scaleio-vs-amazon.png";s:6:"images";a:1:{s:72:"http://datacenternotes.files.wordpress.com/2013/07/scaleio-vs-amazon.png";a:6:{s:8:"file_url";s:72:"http://datacenternotes.files.wordpress.com/2013/07/scaleio-vs-amazon.png";s:5:"width";i:756;s:6:"height";i:445;s:4:"type";s:5:"image";s:4:"area";i:336420;s:9:"file_path";b:0;}}s:6:"videos";a:0:{}s:11:"image_count";i:1;s:6:"author";s:8:"51216153";s:7:"blog_id";s:8:"53483832";s:9:"mod_stamp";s:19:"2013-07-14
    17:53:23";}
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/07/14/emc-acquires-scaleio/"
---
<a href="http://techcrunch.com/2013/06/19/emc-acquires-israeli-storage-startup-scaleio-for-200m-300m-to-compete-better-with-the-cloud-kings/" target="_blank">EMC acquired the Storage Startup ScaleIO for $200M-$300M</a>.
ScaleIO is a Palo Alto based Startup that competes with Amazon AWS, more specifically with its Elastic Block Storage (EBS). They use an architecture of grid computing, where each computing node has local disks and ScaleIo Software. The Software creates a Virtual SAN with local disks, thus providing a highly parallel computing storage nodes SAN, while maintaining HA Enterprise requirements.
ScaleIO Software is allegedly a lightweight piece of Software, and runs alongside with other applications, such as DBs and hyper-visors. They work with all leading Linux distributions and hyper-visors, and offer additional features such as encryption at rest and quality of service (QoS) for performance.
Here's ScaleIO own competitive <a href="http://www.scaleio.com/images/scaleio/pdf/ScaleIO_Compete_EBS_Brief.pdf" target="_blank">smack down</a>:
<a href="http://datacenternotes.files.wordpress.com/2013/07/scaleio-vs-amazon.png"><img class="alignnone size-full wp-image-370" alt="ScaleIO vs Amazon" src="{{ site.baseurl }}/assets/2013/07/scaleio-vs-amazon.png" width="584" height="343" /></a>
