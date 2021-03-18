---
layout: post
title: New HP VC Cookbook for iSCSI
date: 2013-07-11 10:31:52.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Networking
tags:
- HP
- HP Virtual Connect
meta:
  _edit_last: '51216153'
  _publicize_pending: '1'
  tagazine-media: a:7:{s:7:"primary";s:63:"http://datacenternotes.files.wordpress.com/2013/06/vc-iscsi.png";s:6:"images";a:1:{s:63:"http://datacenternotes.files.wordpress.com/2013/06/vc-iscsi.png";a:6:{s:8:"file_url";s:63:"http://datacenternotes.files.wordpress.com/2013/06/vc-iscsi.png";s:5:"width";i:941;s:6:"height";i:425;s:4:"type";s:5:"image";s:4:"area";i:399925;s:9:"file_path";b:0;}}s:6:"videos";a:0:{}s:11:"image_count";i:1;s:6:"author";s:8:"51216153";s:7:"blog_id";s:8:"53483832";s:9:"mod_stamp";s:19:"2013-07-11
    10:31:52";}
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/07/11/new-hp-vc-cookbook-for-iscsi/"
---
Even though its been here for such a long time, iSCSI SANs still have not convinced everyone. Though I do not want to go through that discussion, every help one can get on recommendations how to set it up is helpful. HP just released its "<a href="http://h20628.www2.hp.com/km-ext/kmcsdirect/emr_na-c02533991-7.pdf" target="_blank">HP Virtual Connect with iSCSI</a>", so here are some notes about it.
<ul>
<li>Array supportability: Any Array with iSCSI ports.</li>
<li>You can use Accelerated iSCSI (and NIC behaves like an iSCSI HBA), if you choose Blade NICs that support so. Max MTU size in such scenario is limited to 8342 bytes, unchangeable. Both on Windows and VMware you have no control, as this is automatically negotiated (I am still refering on Accelerated iSCSI option). On Windows Box, when TCPMSS displays 1436, the MTU negotiated is 1514, and when it displays 8260 the MTU negotiated is 8342.</li>
<li>Note that though you can directly connect Storage Host ports to Virtual Connect (except with HP P4000 or any other IP-Clustered Storage), I would definetely not do so, since it limits your options. <em><span style="text-decoration:underline;">VC does not behave as a Switch</span></em> in order to prevent loops and simplify Network management, so traffic will not be forwarded out of the Enclosure (say for instance if you had another Rack Server Host wanting iSCSI). This is also the reason you cannot setup more than one box of HP P4000 to it, since you do require inter box communication for metadata and cluster formation.</li>
<li>Do not forget iSCSI basics: Keep it L2, enable 802.3x Flow control everywhere (Storage Ports, Switch ports, Virtual Connect uplink ports (enabled by default on downlink ports, and Host ports if using Software iSCSI) enable Jumbo Frame support on OS level if you are using Software iSCSI (instead of Accelerated iSCSI), configure iSCSI multipathing when possible (either by using the Storage vendor DSM, or if possible the OS own MPIO), dedicate at least an isolated VLAN or with dedicated physical devices (Switches), and finally isolate at least a FlexNIC on the Host for iSCSI traffic.</li>
</ul>
<a href="http://datacenternotes.files.wordpress.com/2013/06/vc-iscsi.png"><img class="alignnone size-full wp-image-261" alt="VC iSCSI" src="{{ site.baseurl }}/assets/2013/07/vc-iscsi.png" width="584" height="263" /></a>
This figure illustrates a mapping of VC config from the Blade's FlexNICs, to interior of VC (two SUS with corporate VLANs plus dedicted vNet for iSCSI VLAN, and exterior VC uplinks. For redundancy reasons and to limit contention between Downlinks and Uplinks, one would recommend using at least two Uplink Ports on each VC module dedicated to the iSCSI vNet. Whenever a VC module has a one-to-one relationship towards the upstream ToR switch (or several ToR switches when using MLAG, such as HP IRF, Cisco Catalyst VSS or Nexus vPC, etc). When configuring the iSCSI vNets, make sure you enable "Smart Link" feature, to ensure for faster failovers.
You might also want to take advantage of new VC firmware 4.01 features, which include two helpful ones. The first is flexible overprovision of Tx Bandwidth. Ensure a minimum pipeline for that iSCSI Host port, and let it fly (by assigning higher Max Bandwidth) if other pipes are empty on other FlexNICs.
Finally, the second concerns QoS. You might want to use a dedicated 802.1p class for your iSCSI traffic, which is mapped for Egress traffic.
