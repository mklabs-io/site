---
layout: post
title: Some thoughts on HP MSM765zl Wireless Controller
date: 2013-06-07 19:53:24.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Networking
tags:
- HP Wireless
- Wireless
meta:
  _edit_last: '51216153'
  _publicize_pending: '1'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/06/07/some-thoughts-on-hp-msm765zl/"
---
MSM765zl controller has its ports facing the internal Switch Fabric of 5400zl and 8200zl Chassis Switchs.
<a href="http://datacenternotes.files.wordpress.com/2013/06/msm765zl-ports.png"><img class="alignnone size-medium wp-image-45" alt="MSM765zl Ports" src="{{ site.baseurl }}/assets/2013/06/msm765zl-ports.png?w=300" width="300" height="169" /></a>
Here are my bullets on the consequences of this fact to consider:
<ul>
<li>Has two ports  - LAN and Internet Port - each 10GbE;</li>
<li>"Slot 1" is the Internet Port</li>
<li>"Slot 2" is the LAN Port</li>
<li>If MSM765zl is installed, for instance, on slot "D", the Slot 1 is "d1" CLI-wise and Slot 2 is "d2" CLI-wise.</li>
<li>Tagging (VLAN trunking Cisco) a VLAN on slot 1 on the CLI example: Switch(config)#   VLAN 5 tagged d1</li>
<li>Since ports are permanently connected, to simulate disconnecting a port disable the port.</li>
<li>Note: LAN Port <em>should not</em> be disabled on the MSM765zl Controller, since it carries Services Managent Agent (SMA) communications between the Controller and the switch, which delivers the clock to the controller. If you plan to only use the Internet port, then you can assign slot 2 port an unused VLAN ID untagged.</li>
<li>Configuring Controller's IP address on the CLI if managing through the Internet Port: Switch(config)# ip interface wan     Switch(config)# ip address mode [static | dhcp]   Switch(config)# ip address &lt; IP address/prefix length&gt;</li>
</ul>
Disclamer: note that these are <em>my</em> own notes. HP is not responsable for any of the content here provided.
