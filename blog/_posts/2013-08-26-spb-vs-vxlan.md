---
layout: post
title: SPB VS VXLAN
date: 2013-08-26 11:42:38.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Networking
- Virtualization
tags:
- Avaya
- Overlay Virtual Networks
- SPB
- VMware
- VXLAN
meta:
  _edit_last: '51216153'
  _publicize_pending: '1'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/08/26/spb-vs-vxlan/"
---
I found this Packet-Pusher's Podcast from last week about <a href="http://packetpushers.net/show-158-avaya-software-defined-data-centre-fabric-connect/" target="_blank">Avaya's Software Defined Datacenter &amp; Fabric Connect</a> with Paul Unbehagen really interesting. He points out some of the differences in VMware's Overlay Network <a href="http://datacenternotes.wordpress.com/2013/08/12/overlay-virtual-networks-vxlan/" target="_blank">VXLAN</a> approach against Physical Routing Switches Overlay Network's SPB approach (which naturally Avaya is using). Some of these were different encapsulation methods - where with VXLAN the number of headers is quite more numerous - ability to support Multicasting environments (such as PIM), and most importantly, raises the central question: where do you want to control your routing and switching - the Virtual Layer or the Physical Layer. Even though I'm theoretically favorable to Virtual, arguments to keep some functions on the Physical layer still do make a lot of sense in a lot of scenarios.
The Podcast also features a lot of interesting Avaya Automation related features result of a healthy promiscuous relationship between VMware and OpenStack. Also if you want to get in more detail about SPB, Paul Unbehagen covers lots of tech details in his <a href="http://paul.unbehagen.net/" target="_blank">blog</a>.
