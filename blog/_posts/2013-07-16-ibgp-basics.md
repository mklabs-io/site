---
layout: post
title: iBGP basics
date: 2013-07-16 22:13:14.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Networking
tags: []
meta:
  _edit_last: '51216153'
  geo_public: '0'
  _publicize_pending: '1'
  tagazine-media: a:7:{s:7:"primary";s:0:"";s:6:"images";a:0:{}s:6:"videos";a:0:{}s:11:"image_count";i:0;s:6:"author";s:8:"51216153";s:7:"blog_id";s:8:"53483832";s:9:"mod_stamp";s:19:"2013-07-16
    22:13:14";}
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/07/16/ibgp-basics/"
---
BGP peers that belong to the same Autonomous System (AS) are considered iBGP peers. Why is this important? Because iBGP behavior is different from eBGP, even though the commands might be quite similar. Here is a summary of the differences:
<ul>
<li>AS-Path is only pre-pended at eBGP border, <em><span style="text-decoration:underline;">not</span></em> in iBGP. iBGP has thus its own loop prevention mechanism, which consists of prohibiting the advertising of iBGP prefixes from other iBGP peers amongst themselves</li>
<li>BGP sets the TTL in its messages' IP packet equal to one (1), so that it is restricted to one hop. In iBGP TTL is set to the maximum value of 255, as connections between iBGP peers may be multiple hops away</li>
<li>BGP attributes are not changed within iBGP communications. Next-hop remains the eBGP next-hop. Moreover, Local preference attribute will only remain within iBGP peers and will not traverse to neighboring AS.</li>
<li>Route selection process will prefer an eBGP route over iBGP route when AS-Path is the same.</li>
</ul>
It is the IGP's responsibility to find a path to the loopback's interface. However, if the IGP process fails, iBGP will also fail. So the first troubleshooting task should be confirming if both routers can reach the peer's loopback interface.
Since best practices recommend using loopback interfaces for establishing connection with iBGP peers redundancy reasons, remember to advertise each peer's loopback interface into the IGP.
On the other hand loop prevention mechanism is different as well between eBGP and iBGP. eBGP uses AS Path attribute to guarantee loop free behavior, where iBGP uses almost sacred rules in terms of prevention:
<ul>
<li>Prefixes received from an eBGP peer will always be advertised to all other BGP peers (in other words, directly connected)</li>
<li>Prefixes received from iBGP peers are only sent to eBGP peers.</li>
</ul>
So by not advertising iBGP prefixes from other iBGP peers amongst themselves, iBGP is able to prevent loops. However the problem being is that such functioning mechanism <em>requires full-mesh topology between iBGP peers</em>. If not in full-mesh, a second hop away iBGP peer router may not receive certain eBGP routes, breaking reachability.
However, since full-mesh topology requirement makes it quite hard to scale, another mechanism was created, which explicitly breaks the stated above rules about iBGP loop prevention mechanism: <em>Route Reflectors </em>(RR). RR allow one iBGP Router to send prefixes to another iBGP peer (client). So the RR iBGP client does not need to be fully meshed, it only needs to maintain a session with the RR Router.
RR iBGP routers should mirror prefixes to its iBGP RR client with BGP attributes unchanged. Note that you can configure 1:N relationship in terms of iBGP RR router and several iBGP RR clients. Also noteworthy in terms of scalability is the fact that a RR client can also be a RR router for other RR clients, and of the same routes. However the more you cascade, the bigger the risk, since RR Routers represent a Single Point of Failure (SPOF). It is always a best practice to configure a redundant Route reflector, acting as a cluster (so that updates are not duplicated by the reflectors).
Moreover, you can have an hibrid iBGP AS, where you use Route Reflectors for some non fully meshed iBGP routers, and fully meshed another set of routers that follow traditional iBGP rules.
Finally Originator ID attribute and Cluster List attribute can prevent from misconfigurations when using RR.
