---
layout: post
title: Comment Relative To former post "HP MSM Controllers Initial Setup Considerations"
date: 2013-09-21 18:37:52.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Networking
tags:
- HP MSM; WLAN
meta:
  _edit_last: '51216153'
  _publicize_pending: '1'
  tagazine-media: a:7:{s:7:"primary";s:88:"http://datacenternotes.files.wordpress.com/2013/09/access-controlled-flow-of-traffic.png";s:6:"images";a:1:{s:88:"http://datacenternotes.files.wordpress.com/2013/09/access-controlled-flow-of-traffic.png";a:6:{s:8:"file_url";s:88:"http://datacenternotes.files.wordpress.com/2013/09/access-controlled-flow-of-traffic.png";s:5:"width";i:679;s:6:"height";i:360;s:4:"type";s:5:"image";s:4:"area";i:244440;s:9:"file_path";b:0;}}s:6:"videos";a:0:{}s:11:"image_count";i:1;s:6:"author";s:8:"51216153";s:7:"blog_id";s:8:"53483832";s:9:"mod_stamp";s:19:"2013-09-21
    18:37:52";}
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/09/21/comment-relative-to-former-post-hp-msm-controllers-initial-setup-considerations/"
---
This post is an answer to a comment relative to this former post "<a href="http://datacenternotes.wordpress.com/2013/06/07/hp-msm-controllers-initial-setup-considerations/" rel="bookmark">HP MSM Controllers Initial Setup Considerations</a>". (I wanted to add some drawings to the answer to make it clearer, so ended up using another post.) Thank you for your comment. Please let me know if I understood your comment well, or got it all wrong.
George P Isaac's comment was:
"
<em>Hi,</em>
<em>As per my understanding..In option 2 we have to do following config.</em>
<em>1.We should tag particular VLAN in either internet port or access port</em>
<em>2.we should assign IP address to particular VLAN and gateway..</em>
<em>3.Nothing is required in VSC mapping in AP group.</em>
<em>then Where will I specify VLAN mapping..??</em>
<em>“extending the ingress interface to the egress interface “– is this option used when internet port and tunneled network in same VLAN??</em>
"
OK to be fair my post could have been clearer (probably related to the fact that I'm not an expert on HP's MSM solution). I took the next figure from a <a href="http://bizsupport2.austin.hp.com/bc/docs/support/SupportManual/c02566413/c02566413.pdf" target="_blank">MSM Controller Config manual</a> (section 4-30), which summarizes pretty well in my opinion what the options are for Access-Controlled traffic.
<a href="http://datacenternotes.files.wordpress.com/2013/09/access-controlled-flow-of-traffic.png"><img class="size-full wp-image-605 aligncenter" alt="Access-controlled Flow of traffic" src="{{ site.baseurl }}/assets/2013/09/access-controlled-flow-of-traffic.png" width="584" height="309" /></a>
So let me recap: "Option 2" should actually have been <em>Option 2 A) and B)</em>. Essentially these options are:
<strong>2. A)</strong> Having Access-Controlled Clients doing Web Auth on the HP MSM Controller, and then being ejected straight to the a default Gateway of a different interface that bypasses the Corporate Network. So in summary what the Egress VLAN is doing is defining a new default Gateway - for instance the routing device connected to the Internet - for the controller to use specifically for the clients assigned to that VLAN. The reasoning for this option might be that the Network Admin is worried that it may allow clients to access corporate resources, the Controller's default Gateway might not have the adjusted ACLs for handling the Client traffic, or you might prefer to simply save that router CPU. In any case - the main goal is to bypass the Corporate Network.
<span style="text-decoration:underline;">In this option Clients still have an IP address in a different subnet from the egress VLAN subnet</span>. This is the reason why you should enable NAT on that interface (to simplify): because clients will be placed on a subnet to which the gateway has no route to. Alternatively you can also configure a Route on that gateway to that subnet.
<strong>2. B)</strong> Having Access-Controlled Clients doing Web Auth on the HP MSM Controller, and then being ejected to a restricted VLAN, and <span style="text-decoration:underline;">receiving IP addresses on the egress VLAN</span>. Hence the term <em>extending</em> the Egress VLAN to clients. Sort of like in non-access-controlled scenarios, but in this case the controller is actually routing on the background, though clients can't notice it.
Please note that Option 2 A) and B) have essentially one thing in common: clients bypass Corporate resources. However in example B, clients actually receive an IP address in the VLAN where they are ejected either given by a DHCP Server resident on that VLAN, or by the Egress VLAN's Gateway, which is implementing DHCP Relay.
<strong>"<em>then Where will I specify VLAN mapping..??" </em></strong>
So in both cases, you have to alternatives to specify the VLAN Mapping: VSC level or user-based level. In VSC level you simply grab all authenticated users and forward them on the same VLAN. In user-based Egress VLANs you get more granularity by specifying customized VLAN IDs to user account profiles. Or you can implement both altogether, where user-based specifics override VSC level Egress VLAN definitions.
However in option B - where you extend the Egress VLAN IP addressing to the clients - you have some additional settings to configure, which I specified in the previous post:
<ul>
<li>In the global DHCP relay settings, select the checkbox for extending the ingress interface to the egress interface. After this is enabled, you will no longer be able to specify the IP address and subnet mask settings on VSCs.</li>
<li>Disable NAT on the egress VLAN IP interface.</li>
<li>Set the MSM controller’s IP address for the default gateway and DNS server (it will forward them to the correct ones).</li>
</ul>
<em><strong>“extending the ingress interface to the egress interface “– is this option used when internet port and tunneled network in same VLAN??</strong> </em>
<em></em>Well the Egress VLAN (whether extended or not to the client) might be implemented in either port (LAN or Internet). I would rather say that this option might make sense being used when you prefer having the "Bypassing main Corporate Network feature" + "the non-access-controlled alike behavior" all together. This solution might greatly simplify your manual setup, when you want certain clients to access some corporate resources, for instance. Having the same Subnet as those corporate resources might be advantageous for your setup.
Hope this helps. Cheers!
