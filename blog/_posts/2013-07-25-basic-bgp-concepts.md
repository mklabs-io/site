---
layout: post
title: Basic BGP Concepts
date: 2013-07-25 08:30:40.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Networking
tags:
- BGP
meta:
  _edit_last: '51216153'
  geo_public: '0'
  _publicize_pending: '1'
  tagazine-media: a:7:{s:7:"primary";s:0:"";s:6:"images";a:0:{}s:6:"videos";a:0:{}s:11:"image_count";i:0;s:6:"author";s:8:"51216153";s:7:"blog_id";s:8:"53483832";s:9:"mod_stamp";s:19:"2013-07-25
    08:30:40";}
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/07/25/basic-bgp-concepts/"
---
Here is a very short introduction to Border Gatway Protocol (BGP) - or Bloody Good Protocol as some like to call it. BGP is a routing Protocol, which is used mainly for:
<ul>
<li>Sharing prefixes (networks) between ISPs, thus enabling the Internet to scale;</li>
<li>Multi-home an organization to several ISPs (whereby Internet prefixes from ISPs are learned, and its own networks are advertised)</li>
<li>Scaling internally in very large organizations</li>
</ul>
BGP is an Exterior Gateway Protocol (EGP), which differentiates from IGPs - such as RIP, OSPF, IS-IS, EIGRP - mainly for:
<ul>
<li>Uses TCP (port 179) for transport ensuring reliable delivery of BGP messages between peers (Routers)</li>
<li>Can scale hundreds of thousands of Routes (without crashing like IGPs would)</li>
<li>Peers are manually configured - there is no automatic peer discovery, all peers must be manually added</li>
<li>Besides prefix, mask and metric, BGP carries several additional attributes. Though being a major advantage against other protocols, attributes also have the disadvantage of making BGP more complex to configure</li>
<li>BGP is "political in nature" when it comes to finding best paths, meaning Best Paths can be flexibly changed (using attributes). IGPs on the contrary have fixed best path algorithm, namely Short Path First, and chose it by metric. It is much harder to manually influence the Best path choosen in an IGP (for instance you can change the cost of an interface, but it is not possible to set a different cost on the same interface for different destinations), whereas in BGP it is much easier.</li>
<li>BGP may converge more slowly when failures occur, whereas IGPs usually converge faster</li>
<li>Since BGP is not a link state protocol, BGP does not share every prefix in its BGP table with every peer. Instead, it only shares the best routes with peers (even though it might know several paths to the same destination).</li>
</ul>
BGP carries several attributes with each prefix. Since there is no space in the routing table to hold all those attributes, BGP has its own table where it stores prefixes with all its attributes. However, BGP table is not used directly to route IP packets. Instead BGP places only the best prefixes in the routing table with administrative distance of 255 (so that ii the prefix is learned by both BGP and IGP, the IGP route will always be preferred), while maintaining all prefixes in its table. This allows for redundancy as well as load balancing capabilities.
Attributes are what makes BGP so flexible and thus interesting. Most are optional, only the first three are mandatory. Here is a list of BGPs attributes:
<ul>
<li><strong>Origin</strong> (mandatory) - indicates how the attribute was originally created into BGP; in other words, it indicates if a certain prefix was imported from another Routing protocol or static routes, or if it was specifically originated by the administrator manually, or even if it was originated by the EGP (obsolete)</li>
<li><strong>AS Path</strong>  (mandatory) - allows eBGP to be loop free. Subsequent AS can distinguish how the route was created, be being able to see a ordered list of AS between local (first AS Path number) and destination prefix (last AS Path number).</li>
<li><strong>Next hop</strong>  (mandatory) -is an IP address, that should be used for packets destined to a certain prefix. It allows a peer to deduce the interface to use to send packets to the appropriate border router.</li>
<li><strong>Multi-Exit Descriminator (MED) </strong>(optional) - used to influence inbound traffic from a neighboring AS. It <em>only</em> influences direct neighbor peers. It is a low-power attribute (comes late in the decision process), but can be useful in organizations that are multi-homed to the same ISP. The <em>lowest</em> MED value wins</li>
<li><strong>Local preference </strong>(optional) - used to influences outbound traffic, also in organizations multi-homed to the same ISP. Local preference value is <em>only</em> advertised within iBGP peers, and is not advertised to a neighboring AS. Prefix with highest local preference value wins decision process, with the advantage of being able to load balance traffic while maintaining redundancy.</li>
<li><strong>Atomic Aggregate</strong> (optional) - Used in prefix summarization to warn throughout the Internet that a certain prefix is an aggregate</li>
<li><strong>Aggregator </strong>(optional) -Also used in prefix summarization, shared throughout the Internet and it includes the router-id and AS number of the router that performed the summarization</li>
<li><strong>AS 4 Path</strong> (optional) - used to support the longer 32 bit AS numbers through AS that support only 16 bit AS numbers</li>
<li><strong>Communities</strong>  (optional) - it is a special marking for policies usually deployed by ISPs. It allows to group prefixes together in order to give special and common treatment for a set of prefixes</li>
<li><strong>Extended communities</strong>  (optional) - as the name indicates, it extends the Communities attribute length, allowing for additional provider offerings such as MPLS VPNs, etc.</li>
<li><strong>Originator ID</strong> (optional) - attribute intended for iBGP environments where Route Reflectors (RR) are used. It prevents from misconfiguration of RR, by ignoring  duplicate prefixes that a client has advertised and received back.</li>
<li><strong>Cluster List</strong> (optional) - helps preventing loops when using multiple clusters of Route Reflectors (in redundant HA mode). Cluster List operates much like AS Path does, collecting the sequence of Cluster IDs through which the update has traversed. This attribute is also exclusive for iBGP environments, and will not traverse to eBGP peers</li>
</ul>
Finally the BGP decision process hierarchy, from highest to lowest. BGP will chose the best path considering the many attributes associated with the multiple copies of one prefix, instead of the cost or metric like IGPs. Since the attributes can be changed by the administrator, the best path configuration is indeed based on the preferences of the administrator. BGP will also maintain several paths in its table, so that when a prefix is no longer available (for example due to a link failure which BGP monitors through its keep-alive messages in the TCP session) a new best path is populated in the routing table.
Whenever a tie, move to next lower level in order to choose the best path chosen to populate the routing table:
<ul>
<li><strong>Next hop reachable</strong> - a route must exist to next hop IP address, and will not be considered if not reachable</li>
<li><strong>Preferred Value</strong> - the highest preferred value will be chosen. It is a proprietary parameter and local to the router.</li>
<li><strong>Local preference</strong> - the highest local preference value will be chosen. The policy is local to the AS</li>
<li><strong>Locally originated</strong> - prefix originated by the local router</li>
<li><strong>Shortest AS Path</strong> - shared throughout between local and destination.</li>
<li><strong>Origin</strong> - "i" preferred over "?"</li>
<li><strong>Multi-Exit Discriminator</strong> - influences neighboring AS only</li>
<li><strong>External BGP versus internal BGP</strong> - eBGP preferred over iBGP</li>
<li><strong>Router-ID</strong> - the lowest value will be chosen. It is the final tiebraker</li>
</ul>
