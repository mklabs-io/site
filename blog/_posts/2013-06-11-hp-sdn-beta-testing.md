---
layout: post
title: HP SDN beta-testing
date: 2013-06-11 09:46:34.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Networking
- Security
tags:
- SDN
meta:
  _publicize_pending: '1'
  _edit_last: '51216153'
  tagazine-media: a:7:{s:7:"primary";s:0:"";s:6:"images";a:0:{}s:6:"videos";a:0:{}s:11:"image_count";i:0;s:6:"author";s:8:"51216153";s:7:"blog_id";s:8:"53483832";s:9:"mod_stamp";s:19:"2013-06-11
    10:04:40";}
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/06/11/hp-sdn-beta-testing/"
---
HP is beta-testing its Security SDN-based solution: <a href="http://www.zdnet.com/au/case-study-ballarat-grammar-uses-sdn-to-fight-malware-7000015942/">Sentinel</a>.
The article describes a School implementing HP still to come SDN Security solution. The school implemented a hybrid OpenFlow solution - the most likely usual implementation in this initial SDN phase - where "intelligent" switchs are used running all usual old-school networking protocols simultaneously with OpenFlow enabled. OpenFlow is used to forward all DNS request to HP Security Controller - Sentinel. Sentinel uses HP's IPS DB - <a href="http://www8.hp.com/us/en/software-solutions/software.html?compURI=1343617#.UbbtPfm1Euc">Tipping Poin</a>t's Reputation DB - which is updated every 2 hours with spoted Internet suspicious threats.  Sentinel accepts or rejects traffic forwarding to a certain IP, based on what network administrator choose to do. The network admin can configure Sentinel to follow all Tipping Point recommendations, or rather specify his prefered alternatives. Thus when an OpenFlow switch requests what to do with a certain DNS querry, the controller simply "tells" what to do with related packets by populating its OpenFlow table.
This might be a very simplistic security implementation. However the most interesting is the promising margin for development. As this solution gains increasing intelligent, this may well start suiting as low-cost IPS/firewall solutions, using a distributed computing model with already existing OpenFlow switchs. I find this model very appealing for instance for ROBO sites.
Another alternative use-case is HP's Beta-testing example in the article: BYOD. Securing devices at the edge perimeter greatly simplifies network security.
SDN might be a simple reengineering of the way things are done. Still, it's a cool one in deed...
Disclamer: note that these are <em>my</em> own notes. HP is not responsable for any of the content here provided.
