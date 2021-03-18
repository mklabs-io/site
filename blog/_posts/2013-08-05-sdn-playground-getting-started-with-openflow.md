---
layout: post
title: SDN Playground - getting started with OpenFlow
date: 2013-08-05 11:35:42.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Networking
tags:
- OpenFlow
- SDN
meta:
  _edit_last: '51216153'
  geo_public: '0'
  _publicize_pending: '1'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/08/05/sdn-playground-getting-started-with-openflow/"
---
Most every big Networking company has announced something related to SDN. Whether simple marketing to concrete legit solutions, its a question of time until the market is filled with SDN-related products. It is thus essential to start getting familiar with it, and you know damn well there's nothing like getting your hands dirty. So here are some helper-notes on getting started with sandboxing <a href="http://www.openflow.org/" target="_blank">OpenFlow</a> (OF) environments.
To do so I'm using <a href="http://mininet.org/" target="_blank">Mininet</a> - a VM created part of an OpenSource Project to emulate a whole complete environment with a Switch, an OF Controller, and even three linux hosts. Also note I'm using my desktop as a Host, with VirtualBox.
So what you'll need:
<ul>
<li>If you don't have it yet, download <a href="https://www.virtualbox.org/wiki/Downloads" target="_blank">VirtualBox</a>, or another PC hyper-visor Software such as VMware Player. VirtualBox has the advantage of being free for Windows, Linux and Mac.</li>
<li>Download <a href="https://github.com/mininet/mininet/downloads/" target="_blank">Mininet VM</a> OVF image.</li>
<li>After decompressing the image, import the OVF.</li>
</ul>
<a href="http://datacenternotes.files.wordpress.com/2013/08/vb-import-applicance.png"><img class="alignnone size-full wp-image-480" alt="VB Import Applicance" src="{{ site.baseurl }}/assets/2013/08/vb-import-applicance.png" width="584" height="76" /></a>
<ul>
<li>In order to establish terminal session to your VM, you'll need to add a Host-only Adaptor on the Mininet VM. So <em><span style="text-decoration:underline;">first</span></em> (before adding the adaptor on the VM itself) go to VirtualBox &gt; Preferences. Then select the Networking tab, and add and adaptor.</li>
</ul>
<a href="http://datacenternotes.files.wordpress.com/2013/08/adaptor.png"><img class="alignnone size-full wp-image-481" alt="Adaptor" src="{{ site.baseurl }}/assets/2013/08/adaptor.png" width="584" height="443" /></a>
<ul>
<li>Next edit Vm Settings, and add an Host-only Adaptor. Save it and boot the VM.</li>
<li>User: <em>mininet</em>       Password: <em>mininet</em></li>
<li>Type <em>sudo dhclient eth1</em> (or if you haven't added another adaptor and simply changed the default Adaptor from NAT to Host-only adaptor then type <em>eth0</em> instead of <em>eth1</em>) to enable DHCP service on that interface.</li>
<li>Type <em>ifconfig eth1</em> to get the IP address of the adaptor.</li>
<li>Establish an SSH session to the Mininet VM. Open terminal, and type <em>ssh -X [<span style="text-decoration:underline;">user]@[IP-Address-Eth1],</span> </em>where the default mininet user is "mininet" and IP address is what you got after ifconfig. So in my case it was: <em>ssh -X mininet@192.168.56.101</em></li>
<li>Mininet has its own basics tutorial - the <a href="http://mininet.org/walkthrough/" target="_blank">Walkthrough</a>. Also interesting is the <a href="http://www.openflow.org/wk/index.php/OpenFlow_Tutorial" target="_blank">OpenFlow tutorial</a>.</li>
</ul>
The Mininet Walkthrough is designed for less than an hour tutorial. Here are some simple shortcuts to speedup your playing around:
<ul>
<li>Type <em>sudo mn --topo single,3 --mac --switch ovsk --controller remote. </em>This will fire up the emulated environment of the switch, OF controller, and 3 linux hosts.</li>
</ul>
<a href="http://www.openflow.org/wk/index.php/OpenFlow_Tutorial" target="_blank"><img class="size-full wp-image-488 aligncenter" alt="OF topology" src="{{ site.baseurl }}/assets/2013/08/of-topology.png" width="584" height="429" /></a>
<ul>
<li>Type <em>nodes</em> to confirm it. "h" stands for hosts, "s" for switch and "c" for controller. If you want, for instance, to now the addresses of a specific node such as Host2, type <em>h2 ifconfig</em>. If you want to establish a terminal session to the same host, type <em>xterm h2</em>. Note that xterm command only works if you first established ssh session by typing <em>ssh <span style="text-decoration:underline;">-X</span> </em>...</li>
</ul>
This should already get you started.
Have fun!
