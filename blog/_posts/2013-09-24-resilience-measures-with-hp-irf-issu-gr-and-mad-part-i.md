---
layout: post
title: 'Resilience measures with HP IRF: ISSU, GR and MAD (Part I)'
date: 2013-09-24 14:40:53.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Networking
tags:
- GR
- HP IRF
- IRF MAD
- ISSU
meta:
  _edit_last: '51216153'
  _publicize_pending: '1'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/09/24/resilience-measures-with-hp-irf-issu-gr-and-mad-part-i/"
---
In an effort to come down to earth and cover a topic which can be useful for the majority of now-a-days Enterprises that have HP gear, I will cover resilience features one can/should use in a HP Networking environment along with IRF.
Though I don't argue that this is a Best Practice for all cases, clustering HP (former 3Com) switches with IRF can be a great solution to a lot of problems. How? Basically using a simple and thus effective ingredient that speaks by itself: drastic topology complexity reduction. Aggregation of devices to function as one can be an early Christmas for many cases: You get to do so many more LAGs (MLAGs preferably), your Spanning Tree is much simpler (I'm not that big of a fan of the HPN marketing papers issuing STPs death certificate by IRF, you gotta admit human mistake!), your linkstate DB gets much simpler, and you have less devices to config and manage (after you get centralized control &amp; management planes).
<strong>Creating a Non-stop network environment</strong>
I'm not going to focus on HP's claim on convergence time above HPs claim. That should be left for a demo by HP guys. What I want to focus on are the Software features that can be used to create an even more resilient network along with IRF, and which require human config.
These features are In-Service-Software-Upgrade (ISSU) - so yes, you get this bonus feature on standalone switches (where this feature is not natively present) when you setup an IRF cluster - Graceful Restart, and Multi-Active detection - commonly know as split brain detection.
<strong>ISSU</strong>
First you have to take into consideration that the way IRF works when you do virtual clustering with standalone switches, is exactly the way chassis-based switches work: Master MPU controls Management &amp; Control planes, synchronizes real time with standby MPU, and forwarding plane is active on all LPUs. The difference being that in standalone switches one of the switch acts as a Master and the all the rest of the nodes as standby "MPUs".
So when your start a Software upgrade on a IRF stack, the first members to be upgraded are the standby nodes. After this job is completed, one of the standby nodes gets elected as the new master, and failover occurs. While the new elected node acts as Master, the former Master is upgraded. After this job is done, preemption-alike behavior occurs, and the former master gets reelected to the master role.
Note that the virtual cluster runs with a virtual Bridge MAC address, so L2 destination remains remains the same, and the only changes that might occur are on the link forwarding inside MLAGs. This should be neglectable if you have a solid config. Note also that the routing process (if any) running on the master will not be restarted during the service upgrade.
<strong> </strong>
