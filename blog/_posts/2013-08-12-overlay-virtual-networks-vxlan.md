---
layout: post
title: Overlay Virtual Networks - VXLAN
date: 2013-08-12 16:29:28.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Networking
- Virtualization
tags:
- VMware
- VXLAN
meta:
  _edit_last: '51216153'
  geo_public: '0'
  _publicize_pending: '1'
  tagazine-media: a:7:{s:7:"primary";s:77:"http://datacenternotes.files.wordpress.com/2013/08/network-virtualization.png";s:6:"images";a:1:{s:77:"http://datacenternotes.files.wordpress.com/2013/08/network-virtualization.png";a:6:{s:8:"file_url";s:77:"http://datacenternotes.files.wordpress.com/2013/08/network-virtualization.png";s:5:"width";i:554;s:6:"height";i:321;s:4:"type";s:5:"image";s:4:"area";i:177834;s:9:"file_path";b:0;}}s:6:"videos";a:0:{}s:11:"image_count";i:1;s:6:"author";s:8:"51216153";s:7:"blog_id";s:8:"53483832";s:9:"mod_stamp";s:19:"2013-08-12
    16:29:28";}
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/08/12/overlay-virtual-networks-vxlan/"
---
Overlay Virtual Networks (OVN) are increasingly gaining a lot of attention, whether from Virtual Networking providers, as for Physical Networking providers. Here are my notes on a specific VMware Solution: Virtual eXtensible LAN (VXLAN).
<strong>Motivation</strong>
<strong></strong>This would never be an issue if not large/huge datacenters didn't come into place. We're talking about some big-ass companies as well as Cloud providers, enterprises where the number of VMs scales beyond thousands. This is when the ability to scale within the Network, to rapidly change the network, and the ability to isolate different tenants is crucial. So the main motivators are:
<ul>
<li>First L2 communication requirement is an intransigent pusher, which drags 4k 802.1Q VLAN-tagging limitation plus L2 flooding with it</li>
<li>Ability to change configurations without burining the rest of the network, and doing it quickly - e.g. easy isolation</li>
<li>Ability to change without being "physically" constrained by Hardware limitations</li>
<li>Ability to scale large number of VMs, and being able to isolate different tenants.</li>
<li>Unlimmited Workload mobility.</li>
</ul>
These requirements demand for a change in the architecture. They demand that one is not bound to physical hardware constraints, and as such, demand an abstraction layer run by Software and which can be mutable - a virtualization layer in other words.
<a href="http://www.vmware.com/files/pdf/techpaper/Virtual-Network-Design-Guide.pdf" target="_blank"><img class="size-full wp-image-523 aligncenter" alt="Network Virtualization" src="{{ site.baseurl }}/assets/2013/08/network-virtualization.png" width="554" height="321" /></a>
<a href="http://www.cc.gatech.edu/~feamster/" target="_blank">Professor Nick Feamster</a> - who ended today his SDN MOOC course on <a href="https://www.coursera.org/" target="_blank">Coursera</a> - goes further and describes Network Virtualization as being the "killer app" for SDN. As a side note, <a href="http://packetpushers.net/a-review-of-the-recent-coursera-sdn-mooc/" target="_blank">here</a> is an interesting comment from a student of this course.
Thus it is no surprise that Hyper-visor vendors were the first to push such technologies.
It is also no wonder that their approach was to treat the physical Network as a dumb Network, unaware of the Virtual Segmentation that is done within the Hyper-visor.
To conclusion, the main goal being really moving away from the dumb VLAN-aware L2 vSwitch to building a Smart Edge (Internal VM Host Networking) without having to rely on smart Datacenter Fabrics (supporting for instance EVB, etc).
<strong>Overall solutions</strong>
There is more than vendor using a OVN approach to solve the stated problems. VMware was probably one of the first Hyper-visor vendors who started with their vCloud Director Networking Infrastructure (vCDNI). MAC in MAC solution, so L2 Networks over L2. Unfortunately this wasn't a successful attempt, and so VMware changed quickly the its solution landscape. VMware currently has two Network Virtualization solutions, namely VXLAN and more advanced Nicira NVP. Though I present these two as OVN solutions, this is actually quit an abuse, as these are quit different from each other. In this post I will restrict myself to VXLAN.
As for Microsoft, shortly after VXLAN was introduced Microsoft proposed its own Network Virtualization Solution called NVGRE. Finally Amazon uses L3 Core, which uses IP-over-IP communications.
<strong>VMware VXLAN</strong>
Virtual eXtensible LAN (VXLAN) was developed by in conjunction of Cisco and VMware, and IETF launched a draft. It is <a href="http://blog.ipspace.net/2012/08/vxlan-and-otv-ive-been-suckered.html" target="_blank">supposed to have a similar encapsulation Header as in Nexus 7k OTV/LISP</a>, allowing for Nexus 7k to act as a VXLAN Gateway.
VXLAN introduces an additional kernel Software layer between ESX vSwitches and Physical Network Card, which can either be VMware Distributed vSwitch or Cisco's Nexus 1000v. This kernel code is able to introduce additional L2 Virtual Segments beyond the 4k 802.1Q limitation over standard IP Networks. Note that these segments run solely within the hyper-visor, which means that in order to have a physical server communicating with these VMs you will need a <a href="http://searchsdn.techtarget.com/definition/VXLAN-gateway-Virtual-Extensible-VLAN-gateway" target="_blank">VXLAN Gateway</a>.
So the VXLAN kernel is aware of Port-Groups on VM-side and intriduces a VX-segment ID (VNI), and introduces an adaptor on the NIC-side for IP communications- the VXLAN Termination Point (VTEP). VTEP has an IP address and performs encapsulation/decapsulation from L2 traffic generated by a VM and inserts VXLAN header and an UDP header plus traditional IP envelop to talk to the physical NIC. The receiving host where the destination VM resides will do the exact same reverse process.
Also note that it transforms broadcast traffic into multicast traffic for segmentation.
It is thus a transparent layer between VMs and the Network. However, since there is no centralized control plane, VXLAN used to need IP multicast on the DC core for L2 flooding. However this has <a href="http://blog.ipspace.net/2013/07/unicast-only-vxlan-finally-shipping.html" target="_blank">changed with recent enhancements on Nexus OS</a>.
Here's VMware's VXLAN Deployment <a href="http://www.vmware.com/files/pdf/techpaper/VMware-VXLAN-Deployment-Guide.pdf" target="_blank">Guide</a>, and Design <a href="http://www.vmware.com/files/pdf/techpaper/Virtual-Network-Design-Guide.pdf" target="_blank">Guide</a>.
Finally please do note that <a href="http://etherealmind.com/top-5-things-vxlan-fail/" target="_blank">not everyone is pleased with VXLAN solution</a>.
<strong> </strong>
