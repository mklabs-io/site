---
layout: post
title: HP 10500 Campus-Core switch now supports new OS
date: 2013-06-18 17:40:21.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Networking
tags:
- HP Networking
meta:
  _edit_last: '51216153'
  _publicize_pending: '1'
  tagazine-media: a:7:{s:7:"primary";s:67:"http://datacenternotes.files.wordpress.com/2013/06/10500-mpu-v2.png";s:6:"images";a:1:{s:67:"http://datacenternotes.files.wordpress.com/2013/06/10500-mpu-v2.png";a:6:{s:8:"file_url";s:67:"http://datacenternotes.files.wordpress.com/2013/06/10500-mpu-v2.png";s:5:"width";i:1025;s:6:"height";i:456;s:4:"type";s:5:"image";s:4:"area";i:467400;s:9:"file_path";b:0;}}s:6:"videos";a:0:{}s:11:"image_count";i:1;s:6:"author";s:8:"51216153";s:7:"blog_id";s:8:"53483832";s:9:"mod_stamp";s:19:"2013-06-18
    17:40:21";}
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/06/18/hp-10500-campus-core-switch-now-supports-new-os/"
---
HP just released a new hardware model of Management Processing Units (MPUs) for their  10500 Core switch platforms. (For those of you more Cisco-oriented, we are talking about sort of the equivalent of the Supervisors on a Nexus 7k platform, since the 10500 has a CLOS architecture, with dedicated Switching Fabric modules that offload all the switching between the line cards.) Here are some specs of the new MPU on the right side, along with a (for example) 10508 mapping where they fit:
<a href="http://datacenternotes.files.wordpress.com/2013/06/10500-mpu-v2.png"><img class="alignnone size-full wp-image-190" alt="10500 MPU v2" src="{{ site.baseurl }}/assets/2013/06/10500-mpu-v2.png" width="584" height="259" /></a>
Now though the hardware specs show a some hardware enhancements (comparing with the previous one, which had a 1GHz single-core CPU, and 128MB of Flash), the most interesting part is definitely the support for the new version of OS: Comware 7.
Until so far the only platforms that supported this OS were the muscled ToR 5900 (yes 5920 as well, but for the sake of simplicity I will try not making it complicated) and the 12500 high-end Switching Core platform. Here are some of the enhanced and exclusive features that the boxes running CW7 were blessed with:
<ul>
<li>ISSU (In-Service Software Update, which means online Software update without forwarding interruption)</li>
<li>RBAC</li>
<li>DCB Protocols (PFC, DCBX, and ETS) + FCoE (exclusive to the 5900 on this first release)</li>
<li><span style="font-style:inherit;line-height:1.625;">EVB (naturally exclusive to the 5900, as this is a feature intended for ToR)</span></li>
<li>MDC (Multi-Device Context - supports completely isolated Contexts/vSwitches, exclusive to 12500)</li>
<li>TRILL (until so far was exclusive to 5900)</li>
</ul>
<span style="font-style:inherit;line-height:1.625;">So what this means is that HP is bringing some of this juice to its platform positioned for Campus-Core. So if you upgrade your existing MPUs with the new ones, you will be able to add <a href="http://bizsupport1.austin.hp.com/bc/docs/support/SupportManual/c03796249/c03796249.pdf" target="_blank">ISSU</a> (without needing a second box clustered with IRF), as well as <a href="http://bizsupport1.austin.hp.com/bc/docs/support/SupportManual/c03796285/c03796285.pdf" target="_blank">TRILL</a>. </span>
And, as you can imagine, there might be some updates in the near future on which Comware 7 features are supported on this box. Just be aware that 10500 is and will remain (at least for quite a while) a platform positioned for Campus-core, and as such, it does not make sense to add Datacenter funcionalities, such as EVB and DCB/FCoE.
Cheers.
