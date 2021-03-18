---
layout: post
title: Careful with that dual-hop FCoE setup with HP Virtual Connect and Nexus 5500/5000
date: 2013-06-25 09:46:23.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Networking
tags:
- Cisco
- HP Netwroking
meta:
  _edit_last: '51216153'
  _publicize_pending: '1'
  tagazine-media: a:7:{s:7:"primary";s:0:"";s:6:"images";a:0:{}s:6:"videos";a:0:{}s:11:"image_count";i:0;s:6:"author";s:8:"51216153";s:7:"blog_id";s:8:"53483832";s:9:"mod_stamp";s:19:"2013-06-25
    09:48:57";}
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/06/25/carefull-with-that-dual-hop-fcoe-setup-with-hp-virtual-connect-and-nexus-55005000/"
---
HP just recently<a href="http://h18004.www1.hp.com/products/blades/virtualconnect/index.html" target="_blank"> launched</a> its new Firmware release, the <a href="http://h20000.www2.hp.com/bizsupport/TechSupport/SoftwareDescription.jsp?lang=en&amp;cc=us&amp;prodTypeId=3709945&amp;prodSeriesId=4144084&amp;swItem=MTX-3e137331d4634d7eb0c3acd6d8&amp;prodNameId=4144085&amp;swEnvOID=4064&amp;swLang=8&amp;taskId=135&amp;mode=3" target="_blank">4.01</a> - for its Blade Interconnect darling, the Virtual Connect. You can find the release notes <a href="http://bizsupport2.austin.hp.com/bc/docs/support/SupportManual/c03801912/c03801912.pdf" target="_blank">here</a>. Also noteworthy are the cookbooks, and HP just released its "<a href="http://bizsupport1.austin.hp.com/bc/docs/support/SupportManual/c03808925/c03808925.pdf" target="_blank"><em>Dual-Hop FCoE with HP Virtual Connect modules Cookbook</em></a>" .
<span style="font-style:inherit;line-height:1.625;">One of the new features is dual-hop FCoE. What this means is that until so far you had only one hop FCoE from Virtual Connect to the server Converged Network Adaptor (CNA). This facilitates quite a lot FCoE implementation, since you don't need any of the DCB protocols - its only one hop. Starting from version 4.01 you can now have an uplink connection where you pass FCoE traffic.</span>
<a href="http://datacenternotes.files.wordpress.com/2013/06/vcff-dual-hop-nexus.png"><img class="size-full wp-image-268 aligncenter" alt="VCFF-DUal-hop-Nexus" src="{{ site.baseurl }}/assets/2013/06/vcff-dual-hop-nexus.png" width="584" height="408" /></a>
The VC serves thus as a FIP-snooping device and the upstream switch as Fiber Channel Forwarder (FCF), in case you want to have retrocompatibility with former existing FCP environments. Please note that only a dual-hop topology is supported. You cannot continue hopping FCoE traffic accross your Network. Well you can, but its just not supported.<a style="font-style:inherit;line-height:1.625;text-decoration:underline;" href="http://datacenternotes.files.wordpress.com/2013/06/dual-hp-fip-snooping.png"><img class="size-full wp-image-269 aligncenter" style="border-color:#bbbbbb;background-color:#eeeeee;" alt="Dual-hp-FIP-snooping" src="{{ site.baseurl }}/assets/2013/06/dual-hp-fip-snooping.png" width="584" height="316" /></a>
Some important considerations:
<ul>
<li>On HP side, only VC FlexFabric (FF) and Flex10/10D are able to do dual-hop FCoE. On Cisco's side, only Nexus 5500 and 5000 (NX-OS version 5.2(1)N1(3)) are supported.</li>
<li>Only Nexus FCF mode is supported (NPV mode is not supported), and Nexus ports must be configured as trunk ports and STP edge ports;</li>
<li>VC-FF, only ports X1 to X4 support FCoE; VC Flex10/10D supports using all its Uplinks;</li>
<li>Only VC uplink Active/Active design is supported; Active/Standby design will not be supported due to potential extra-hop on VC stacking links due to misconfiguration on Server profile;</li>
<li>Cross-Connect links between VC uplinks to Nexus are <span style="text-decoration:underline;">not</span> supported. Even if you're using vPC, it will be a design violation of traffic mix in SAN A (say connected to Nexus 01) and SAN B (say connected to Nexus 02).</li>
</ul>
<a href="http://datacenternotes.files.wordpress.com/2013/06/vcff-dual-hp-constraints.png"><img class="size-full wp-image-271 aligncenter" alt="VCFF - Dual-hp-constraints" src="{{ site.baseurl }}/assets/2013/06/vcff-dual-hp-constraints.png" width="474" height="268" /></a>
<ul>
<li>Moreover, if you want to converge all traffic on the same uplink LAG ports, you will have to have a single Shared Uplink Set (SUS) per VC. Note that this SUS can only have one FCoE Network.</li>
</ul>
<a href="http://datacenternotes.files.wordpress.com/2013/06/dual-hop-sus.png"><img class="size-full wp-image-274 aligncenter" alt="Dual-hop SUS" src="{{ site.baseurl }}/assets/2013/06/dual-hop-sus.png" width="361" height="283" /></a>
<ul>
<li>However, if you have more uplink portson your VCs available, then consider seperating Ethernet traffic from Ethernet traffic, using SUS for Ethernet, and a vNet for the FCoE traffic. So in this scenario, and when using vPC on the Nexus it is still advisable to cross-connect VC SUS Uplinks from the same VC Module to different Nexus for pure Ethernet traffic.</li>
</ul>
<a href="http://datacenternotes.files.wordpress.com/2013/06/vcff-dual-hop-uplinks.png"><img class="size-full wp-image-272 aligncenter" alt="VCFF-Dual-hop-Uplinks" src="{{ site.baseurl }}/assets/2013/06/vcff-dual-hop-uplinks.png" width="584" height="311" /></a>
<ul>
<li>Note: Remember VC rules. You assign uplinks from one VC module to one SUS and uplinks from another<em>different</em> VC module to the <em>other</em> SUS. It is very important that you get this right, as VC modules are not capable of doing IRF withemselves (yet). More on how setup VC with IRF <a href="http://bizsupport1.austin.hp.com/bc/docs/support/SupportManual/c02843088/c02843088.pdf" target="_blank">here</a>.</li>
</ul>
<a href="http://datacenternotes.files.wordpress.com/2013/06/vc-uplinks.png"><img class="size-full wp-image-276 aligncenter" alt="VC-uplinks" src="{{ site.baseurl }}/assets/2013/06/vc-uplinks.png" width="261" height="220" /></a>
<ul>
<li>So when using port-channels on VC FCoE ports, you will need to configure ports on the Nexus to belong to the same Unified Port Controller (UPC) ASIC. Nexus 5010/5020 support a maximum of 4 ports when carrying FCoE traffic, 5548/5596 a maximum of 8.</li>
</ul>
<a href="http://datacenternotes.files.wordpress.com/2013/06/nuxus-upc-ports.png"><img class="size-full wp-image-270 aligncenter" alt="Nuxus-UPC-Ports" src="{{ site.baseurl }}/assets/2013/06/nuxus-upc-ports.png" width="493" height="361" /></a>
More on the VC setup and installation "<em>HP Virtual Connect for c-Class BladeSystem Version 4.01 User Guide</em>" <a href="http://bizsupport2.austin.hp.com/bc/docs/support/SupportManual/c03791917/c03791917.pdf" target="_blank">here</a>.
Disclamer: note that these are <em>my</em> own notes. HP is not responsable for any of the content here provided.
