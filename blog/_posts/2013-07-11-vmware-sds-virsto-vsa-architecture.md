---
layout: post
title: VMware SDS (?) - Virsto VSA Architecture
date: 2013-07-11 00:13:31.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Storage
tags:
- SDS
- VMware
meta:
  _edit_last: '51216153'
  _publicize_pending: '1'
  tagazine-media: a:7:{s:7:"primary";s:75:"http://datacenternotes.files.wordpress.com/2013/07/vm-io-blender-virsto.png";s:6:"images";a:3:{s:75:"http://datacenternotes.files.wordpress.com/2013/07/vm-io-blender-virsto.png";a:6:{s:8:"file_url";s:75:"http://datacenternotes.files.wordpress.com/2013/07/vm-io-blender-virsto.png";s:5:"width";i:747;s:6:"height";i:370;s:4:"type";s:5:"image";s:4:"area";i:276390;s:9:"file_path";b:0;}s:81:"http://datacenternotes.files.wordpress.com/2013/06/virsto-vmware-architecture.png";a:6:{s:8:"file_url";s:81:"http://datacenternotes.files.wordpress.com/2013/06/virsto-vmware-architecture.png";s:5:"width";i:495;s:6:"height";i:347;s:4:"type";s:5:"image";s:4:"area";i:171765;s:9:"file_path";b:0;}s:74:"http://datacenternotes.files.wordpress.com/2013/07/virsto-architecture.png";a:6:{s:8:"file_url";s:74:"http://datacenternotes.files.wordpress.com/2013/07/virsto-architecture.png";s:5:"width";i:619;s:6:"height";i:318;s:4:"type";s:5:"image";s:4:"area";i:196842;s:9:"file_path";b:0;}}s:6:"videos";a:0:{}s:11:"image_count";i:3;s:6:"author";s:8:"51216153";s:7:"blog_id";s:8:"53483832";s:9:"mod_stamp";s:19:"2013-07-11
    15:42:13";}
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/07/11/vmware-sds-virsto-vsa-architecture/"
---
Though still having limited technical resources available for an indepth deep dive, the <a href="http://virsto.com/resources/presentations/" target="_blank">available resources</a> already provide an overall picture.
Virsto positions itself as a Software Defined Storage (SDS) product, using Storage virtualization tecnology. Well, besides being 100% Software, I see a huge gap from the Networking Software-Defined concept to the SDS side. I do imagine VMware's marketing pushing them to step up the SDS marketing, with their Software Defined Datacenter vision. However Virsto failed to make me disagree on <a href="http://en.wikipedia.org/wiki/Software_defined_storage" target="_blank">Wikipedia's SDS definition</a>.
Living the SDS story aside, there's still a story to tell. Not surprisingly, <a href="http://www.youtube.com/embed/qBpzSQAulxs?rel=0&amp;autoplay=1" target="_blank">their main selling point is not different from the usual Storage vendors: performance</a>. They constructed their technical marketing focused on solving what they call the "<a href="http://virsto.com/blog/the-missing-link-in-software-defined-storage" target="_blank"><em>VM I/O blender</em></a>" issue (well choosen term, have to recognize that). The VM I/O blender effect derivates from having of several VMs in concurrency disputing IOPS from the same Array, thus creating a large randomized pattern. (By the way: this randomized pattern is one of the reasons why you should always lookout for Storage Vendors claiming large Cache-Hit percentages, and always double-check the<em> theoretical IO capacity on the disk side</em>.)
<a href="http://datacenternotes.files.wordpress.com/2013/07/vm-io-blender-virsto.png"><img class="alignnone size-full wp-image-349 aligncenter" alt="VM IO Blender Virsto" src="{{ site.baseurl }}/assets/2013/07/vm-io-blender-virsto.png" width="584" height="289" /></a>
<strong>How do you <a href="http://www.youtube.com/watch?v=qg2pn3Mugfg" target="_blank">Architect Virsto's Solution</a></strong>
Virsto uses a distributed Clustered Virtual Storage Appliance (VSA), with a parallel computing architecture (up to 32 nodes on the same VMware cluster latest from what I could check). You install a Virtual Storage Appliance (VSA) on each host, (and from what I could understand) serving as a target for you physical storage Arrays. Virsto VSA then presents storage back to your VMs just like NFS datastores, allowing it to control all VM IO while supporting HA, FT, VM and storage vMotion, DRS. Here is an <a href="http://virsto.com/downloads/data-sheets/Virsto_for_vSphere_Data_Sheet_(Nov_12).pdf" target="_blank">overview of the architecture</a>.
<a href="http://datacenternotes.files.wordpress.com/2013/06/virsto-vmware-architecture.png"><img class="size-full wp-image-283 aligncenter" alt="Virsto VMware Architecture" src="{{ site.baseurl }}/assets/2013/07/virsto-vmware-architecture.png" width="495" height="347" /></a>
As usual, one of the VSA's serves as a Master, and the other VSA on different nodes (i.e. VMware Hosts) as slaves. Each VSA has its own Log file (the vLog file), that serves as the central piece in the whole architecture, as we will see next. They support heterogeneous Block Storage, so yes, you are able to virtualize different Vendor Arrays with it (although in practice that might not be the best practical solution at all).
<a href="http://datacenternotes.files.wordpress.com/2013/07/virsto-architecture.png"><img class="size-full wp-image-351 aligncenter" alt="Virsto Architecture" src="{{ site.baseurl }}/assets/2013/07/virsto-architecture.png" width="584" height="300" /></a>
You can use different Arrays from different providers, and tier those volumes up to four (4) storage tiers (such as SSD, 15k rpm, 10k rpm, and 7.2k rpm). After aggregation on the VSA, Virsto vDisks can be presented in many formats, namely: iSCSI LUN, vmdk, VHD, vVol, or KVM file. So yes, Virsto VSA also works on top of Hyper-V the same way. Here is an <a href="http://www.youtube.com/watch?v=1wbyi4uGjUc" target="_blank">interesting Hyper-V demo</a>.
So to solve the VM I/O blender effect, every Write IO from each VM is captured and logged into a local log file (Virsto vLog). Virsto aknowledges the write back to the VM immediately after the commit on the log file, and then asynchronously writes in a sequential manner back to your Storage Array. Having data written in a sequential manner has the advantage of having blocks organized in a contiguous fashion, thus enhancing performance on reads. Their claim is a better 20-30% read performance.
So as a performance sizing best practice, they do recommend using SSD storage for the vLog. Not mandatory, but serious performance booster. Yes please.
The claim is that through the logging architecture they are able to do both thin provisioning and Snapshots without any performance degredation.
As a result, they compare Virsto vmdks to Thick vmdks in terms of performance, and to linked-clones in terms of space efficiency. If you check Virsto's demo on their site, you'll see that they claim having better performance than thick eagor-zeroed vmdks, even with Thin Provisioned Virsto-vmdk disks. Note that all Virsto-vmdk are thin.
Finally how do they guarantee HA of the vLog? Well from what I could understand, one major difference from VMware's own VSA is that Virsto's VSA will simply leverage already existing shared storage from your SAN. So it will not create the shared storage environment, from what I understood it stands on Shared Storage. When I say it, please take only into consideration the vLog. I see no requirements to have the VSA as well on top of Shared Storage.
Data consistency is achieved by redoing the log file of the VSA that was on the failing Host. VMware's VSA on the contrary, allows you to aggregate local non-shared disks of each of your VMware Hosts, and present them back via NFS to your VMs or Physical Hosts. It does this while still providing HA (of of the main purposes) by coping data across Clustered nodes in a "Network RAID", providing continuous operations even on Host failure.
Some doubts that I was not able to clarify:
<ul>
<li>What are the minimum vHardware recommendations for each VSA?</li>
<li>What is the expected Virsto VSA's performance hit on the Host?</li>
<li>What is the limit maximum recommended number of VSAs clustered together?</li>
</ul>
<strong>VMware's VSA vs Virsto VSA</strong>
So as side note, I do <span style="text-decoration:underline;"><em>not</em></span> think it is fair to claim to near death of VMware's own VSA, even if Virsto is indeed able to sustain all its technical claims. Virsto has a different architecture, and can only be positioned in more complexed IT environments  where you already have different Array technologies and struggle for performance vs cost.
VMware's VSA is positioned for SMB customers with limited number of Hosts, and is mainly intended to provide a Shared Storage environment without shared storage. So different stories, different ends.
