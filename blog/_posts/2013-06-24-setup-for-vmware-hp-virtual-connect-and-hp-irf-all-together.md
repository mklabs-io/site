---
layout: post
title: Setup for VMware, HP Virtual Connect, and HP IRF all together
date: 2013-06-24 08:32:45.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Networking
tags:
- HP
- VMware
meta:
  _edit_last: '51216153'
  _publicize_pending: '1'
  tagazine-media: a:7:{s:7:"primary";s:61:"http://datacenternotes.files.wordpress.com/2013/06/vc-irf.png";s:6:"images";a:2:{s:61:"http://datacenternotes.files.wordpress.com/2013/06/vc-irf.png";a:6:{s:8:"file_url";s:61:"http://datacenternotes.files.wordpress.com/2013/06/vc-irf.png";s:5:"width";i:532;s:6:"height";i:512;s:4:"type";s:5:"image";s:4:"area";i:272384;s:9:"file_path";b:0;}s:69:"http://datacenternotes.files.wordpress.com/2013/06/vswitch-vmware.png";a:6:{s:8:"file_url";s:69:"http://datacenternotes.files.wordpress.com/2013/06/vswitch-vmware.png";s:5:"width";i:594;s:6:"height";i:137;s:4:"type";s:5:"image";s:4:"area";i:81378;s:9:"file_path";b:0;}}s:6:"videos";a:0:{}s:11:"image_count";i:2;s:6:"author";s:8:"51216153";s:7:"blog_id";s:8:"53483832";s:9:"mod_stamp";s:19:"2013-06-24
    08:32:45";}
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/06/24/setup-for-vmware-hp-virtual-connect-and-hp-irf-all-together/"
---
Trying to setup your ESXi hosts networking with HP Virtual Connect (VC) to work with your HP's IRF-Clustered ToR switchs in Active-Active mode?
<a href="http://bizsupport1.austin.hp.com/bc/docs/support/SupportManual/c02843088/c02843088.pdf" target="_blank"><img class="alignnone size-full wp-image-227" alt="VC-IRF" src="{{ site.baseurl }}/assets/2013/06/vc-irf.png" width="532" height="512" /></a>
This <a href="http://bizsupport1.austin.hp.com/bc/docs/support/SupportManual/c02843088/c02843088.pdf" target="_blank">setup</a> improves performance, as you are able to load balance traffic from several VMs across both NICs, across both Virtual Connect modules, and across both ToR switchs, while in the meanwhile gaining more resilience. Convergence on your network will be much faster, either if a VC uplink fails, a whole VC module fails, or a whole ToR fails.
Here's a couple of docs that might help you get started. First take a look on the IRF config manual of your corresponding HP's ToR switches. Just as an example, <a href="http://bizsupport1.austin.hp.com/bc/docs/support/SupportManual/c02648772/c02648772.pdf" target="_blank">here's</a> the config guide for HP's 58Xo series. Here are some important things you should consider when setting up an IRF cluster:
<ol>
<li>If you have more than two switches, choose the cluster topology.  HP IRF allows you to setup a daisy-chained cluster, or in ring-topology. Naturally the second one is the recommended for HA reasons.</li>
<li>Choose which ports to assign to the cluster. IRF does not need specific Stacking modules and interfaces to build the cluster. For the majority of HP switchs, the only requirement is to use 10GbE ports, so you can flexibly choose whether to use cheaper CX4 connections, or native SFP+ ports (or modules) with copper Direct Attach Cables or optical transcevers. Your choice. You can assign one or more physical ports into one single IRF port. Note that the maximum ports you can assign to an IRF port varies according to the switch model.</li>
<li>Specify IRF unique Domain for that cluster, and take into consideration that nodes will reboot in the meanwhile during the process, so this should be an offline intervention.</li>
<li>Prevent from Split-brain Condition potential events. IRF functions by electing a single switch in the cluster as the master node (which controls Management and Control plane, while data plain works Active-Active on all nodes for performance). So if by any reason stacking links were to fail, slave nodes by default initiate new master election, which could lead into a split-brain condition. There are several mechanisms to prevent this front happening (called Multi-Active Detection (MAD) protection), such as LACP MAD or BFD MAD.</li>
</ol>
Next hop is your VC. Whether you have a Flex-Fabric or a Flex-10, in terms of Ethernet configuration everything's pretty much the same. You setup two logically identical Shared Uplink Sets (SUS) - note that VLAN naming needs to be different though inside the VC. Next you assign uplinks from one VC module to one SUS and uplinks from another <span style="text-decoration:underline;"><em>different</em> </span>VC module to the <em><span style="text-decoration:underline;">other</span></em> SUS. It is very important that you get this right, as VC modules are not capable of doing IRF withemselves (yet).
If you have doubts on how VC works, well nothing like the <a href="http://h20000.www2.hp.com/bc/docs/support/SupportManual/c02616817/c02616817.pdf" target="_blank">VC Cookbook</a>. As with any other documentation that I provide with direct links, <span style="text-decoration:underline;"><em>I strongly recommend that you google them up just to make sure you use the latest version available online.</em></span>
If you click on the figure above with the physical diagram you will be redirected to HP's Virtual Connect and IRF Integration guide (where I got the figure in the first place).
Last hop is your VMware hosts. Though kind of outdated document (as Vmware already launched new ESX releases after 4.0), for configuration purposes the "<a href="http://h20195.www2.hp.com/V2/GetPDF.aspx/4AA1-1907ENW.pdf" target="_blank">Deploying a VMware vSphere HA Cluster with HP Virtual Connect FlexFabric</a>" guide already allows you to get started. What you should take into consideration is that you will not be able to use the <a href="http://blogs.vmware.com/vsphere/2013/01/vsphere-5-1-vds-new-features-link-aggregation-control-protocol-lacp.html" target="_blank">Distributed vSwitch environment with new vmware's LACP capabilities</a>, which I'll cover in just a while. This guide provides indications on how to configure in a standard vSwitch or distributed setup on your ESX boxes (without using VMware's new dynamic LAG feature).
Note that I am skipping some steps on the actual configuration, such as the Profile configuration on the VC that you need to apply to the ESX hosts. Manuals are always your best friends in such cases, and the provided ones should provide for the majority of steps.
You should make sure that you use redundant Flex-NICs if you want to configure your vSwitchs with redundant uplinks for HA reasons. Flex-NICs are present themselves to the OS (in this case ESXi kernel) as several distinct NICs, when in reallity these are simply Virtual NICs of a single physical port. Usually each half-height HP Blade comes with a single card (the LOM), whcih has two physical ports, each of which can virtualize (up to date) four virtual NICs. So just make sure that you don't choose randomly which Flex-NIC is used for each vSwitch.
<span style="font-style:inherit;line-height:1.625;">Now comes the tricky part. So you should not that you are configuring Active-Active NICs for one vSwitch.</span>
<a href="http://datacenternotes.files.wordpress.com/2013/06/vswitch-vmware.png"><img class="alignnone size-full wp-image-228" alt="vSwitch-Vmware" src="{{ site.baseurl }}/assets/2013/06/vswitch-vmware.png" width="584" height="134" /></a>
<span style="font-style:inherit;line-height:1.625;">However, in reality these NICs are not touching the same device. Each of these NICs is mapped statically to a different Virtual Connect module (port 01 to VC module 01, and port 02 to VC module 02). Therefore in a Virtual Connect environment you should note that you cannot choose the load balancing algorithm on the NICs based on IP-hash. </span><span style="font-style:inherit;line-height:1.625;">Though you can choose any other algorithmn, HP's recommendation is using Virtual Port ID, as you can see on the guide (this is the little piece of information missing on VC Cookbook on Active-Active config with Vmware).</span>
Finally, so why the heck can't you use IP-hash, nor VMware's new LACP capabilities? Well  IP-Hash is intended when using link aggregation mechanisms to a single switch (or Switch cluster like IRF, that behaves and is seen as a single switch). As a matter a fact, IP-SRC-DST is the <a href="http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&amp;cmd=displayKC&amp;externalId=1004048" target="_blank">only algorithm supported</a> for NIC teaming on the ESX box as well for the Switch when using port trunks/EtherChannels:
" [...]
<ul>
<li>S<em>upported switch Aggregation algorithm: IP-SRC-DST (short for IP-Source-Destination)</em></li>
<li><em>Supported Virtual Switch NIC Teaming mode: IP HASH</em></li>
<li><em><strong>Note</strong>: The only load balancing option for vSwitches or vDistributed Switches that can be used with EtherChannel is IP HASH:</em>
<ul>
<li><em>Do not use beacon probing with IP HASH load balancing.</em></li>
<li><em>Do not configure standby or unused uplinks with IP HASH load balancing.</em></li>
<li><em>VMware only supports one EtherChannel per vSwitch or vNetwork Distributed Switch (vDS).</em>"</li>
</ul>
</li>
</ul>
Moreover, VMware explains in another <a href="http://blogs.vmware.com/vsphere/2013/01/vsphere-5-1-vds-new-features-link-aggregation-control-protocol-lacp.html" target="_blank">article</a>: " <em>What does this IP hash configuration has to do in this Link aggregation setup? IP hash algorithm is used to decide which packets get sent over which physical NICs of the link aggregation group (LAG). For example, if you have two NICs in the LAG then the IP hash output of Source/Destination IP and TCP port fields (in the packet) will decide if NIC1 or NIC2 will be used to send the traffic. Thus, we are relying on the variations in the source/destination IP address field of the packets and the hashing algorithm to provide us better distribution of the packets and hence better utilization of the links.</em>
<em>It is clear that if you don’t have any variations in the packet headers you won’t get better distribution. An example of this would be storage access to an nfs server from the virtual machines. On the other hand if you have web servers as virtual machines, you might get better distribution across the LAG. Understanding the workloads or type of traffic in your environment is an important factor to consider while you use link aggregation approach either through static or through LACP feature.</em>"
So given the fact that you are connecting your NICs to different VC modules (in a HP c7000 Enclosure), the only way you will be able to have active-active teaming is with static etherchannel, and without IP-Hash.
None-the-less please note that if you were configuring for instance a non-blade server, such as a HP Dl360, and connecting it to an IRF ToR cluster, then in that case you would be able to use vDS with LACP feature, and thus use IP-Hashing. Or if you were using a HP c3000 Enclosure, which is usually positioned for smaller enviroments. So it does not make that much sense to be using a c3000 and in the meanwhile having the required vSphere Enterprise Plus licensing.
That's it, I suppose. Hopefully this overview should already get your ESX farm up-and-running.
Cheers.
Disclamer: note that these are <em>my</em> own notes. HP is not responsable for any of the content here provided.
