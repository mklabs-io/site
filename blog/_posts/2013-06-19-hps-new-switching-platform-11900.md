---
layout: post
title: HP's new Switching platform 11900
date: 2013-06-19 14:23:28.000000000 +02:00
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
  tagazine-media: a:7:{s:7:"primary";s:0:"";s:6:"images";a:0:{}s:6:"videos";a:0:{}s:11:"image_count";i:0;s:6:"author";s:8:"51216153";s:7:"blog_id";s:8:"53483832";s:9:"mod_stamp";s:19:"2013-06-19
    14:57:03";}
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/06/19/hps-new-switching-platform-11900/"
---
HP showcased one of its soon to release new big&amp;mean switching platform in InterOp 2013, the <a href="http://h17007.www1.hp.com/docs/interop/2013/4AA4-6497ENW.pdf" target="_blank">11900</a>, and today you can already get prices on this peace. Although having been considered <a href="http://h30507.www3.hp.com/t5/HP-Networking/HP-Networking-included-in-Hottest-products-at-Interop-2013-by/ba-p/138885#.UcG-gPm1Euc" target="_blank">hottest products in InterOp 2013</a>, if you ask me, this might not be a game changer for HP's portfolio as I believe the <a href="http://h17007.www1.hp.com/docs/interop/2013/4AA4-6499ENW.pdf" target="_blank">12900</a> might be when it releases. But it is interesting enough to be noted.
This box is positioned for aggregation layer in the Datacenter (HP's FlexFabric Portfolio), right next to today's main Datacenter core platform, the 12500. Although probably a lot of work like me in countries qith "slightly" different companies as the US does. In countries where big companies considering substituting their Cisco 6500 with a 11900 might be more than enough, and therefore surely suffice as a core.
So we are talking about another platform with CLOS architecture designed to provide a non-blocking architecture with 4 Fabric Modules (each with 3x 40Gbps <em><span style="text-decoration:underline;">per</span></em> I/O line card), with performance figures of 7,7Tbps, 5.7 Mpps, and achieving latency levels of 3 μs.
As for scalability, we are talking about being able to scale up to 64x 40GbE ports (non-blocking), and 384x 10GbE (also non-blocking) now-a-days. As a matter a fact, technically each I/O line card has 480Gbps (4x3x40) of total backend bandwidth,
Anyway, as you may already suspect, it does indeed run HP's new version of Network OS, the Comware 7 (Cw7), and will therefore also contain almost all of the cool "Next-Generation" Datacenter features, namely:
<ul>
<li>TRILL</li>
<li>DCB and FCoE</li>
<li>EBV/VEPA</li>
<li>OpenFlow 1.3 (thus standards-based SDN-ready)</li>
<li>Though not available in this first release, MDC (context/ Virtual Switchs inside your chassis box) and SPB (sort of alternative to TRILL) will also become available</li>
</ul>
You might be wondering why the hell have a new Box, when HP has just released <a href="http://datacenternotes.wordpress.com/2013/06/18/hp-10500-campus-core-switch-now-supports-new-os/" target="_blank">new management modules for the 10500</a>, which also bring Comware 7 to it, and some of its features. Note that the 10500 is positioned for Campus-Core, not DC. As such, it will not support features as DCD/FCoE and EVB/VEPA, and those the major differences in the mid-term that you can count on.
As for it being positioned as a Core platform, please note that when it comes to scalability numbers in terms table size of FIB, ARP, MAC, etc 11900 platform does not even scale as near as HP's 12500. Along with high buffering capability, these might be important features in DC environments with high density and number of VMs.
Networktest also publiced its<a href="http://h20195.www2.hp.com/V2/GetPDF.aspx/c03743130.pdf" target="_blank"> test results</a> on the 11900. The tests consist essentially of a Spirent TestCenter hitting a L2&amp;3 bombardment simultaneously on all 384 10Gbps ports (all 8 I/O slots filled with 48 10GbE line cards), and another bombardment simultaneously on all 64 40Gbps ports (all 8 I/O slots filled with 8 40GbE line cards).
Disclamer: note that these are <em>my</em> own notes. HP is not responsable for any of the content here provided.
