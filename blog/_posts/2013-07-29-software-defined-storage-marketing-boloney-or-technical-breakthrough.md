---
layout: post
title: Software Defined Storage - marketing boloney or technical Breakthrough?
date: 2013-07-29 11:14:20.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Storage
tags:
- EMC
- Software Defined Storage
- VMware
meta:
  _edit_last: '51216153'
  geo_public: '0'
  _publicize_pending: '1'
  tagazine-media: a:7:{s:7:"primary";s:67:"http://datacenternotes.files.wordpress.com/2013/07/dataontap_vm.png";s:6:"images";a:1:{s:67:"http://datacenternotes.files.wordpress.com/2013/07/dataontap_vm.png";a:6:{s:8:"file_url";s:67:"http://datacenternotes.files.wordpress.com/2013/07/dataontap_vm.png";s:5:"width";i:450;s:6:"height";i:304;s:4:"type";s:5:"image";s:4:"area";i:136800;s:9:"file_path";b:0;}}s:6:"videos";a:0:{}s:11:"image_count";i:1;s:6:"author";s:8:"51216153";s:7:"blog_id";s:8:"53483832";s:9:"mod_stamp";s:19:"2013-07-29
    11:14:20";}
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/07/29/software-defined-storage-marketing-boloney-or-technical-breakthrough/"
---
If you're as suspicious as <a href="http://en.wikipedia.org/wiki/Software_defined_storage" target="_blank">Wikipedia</a> and I are about the new marketing buzzwords that have transcended the Networking world into Storage terminology, then this might be a post for you.
OK, so I tried a sort of reverse-engineering approach when investigating about Software Defined Storage (SDS). I tried to figure out how SDN would materialize in a Storage world, and only then did I checked what vendors are saying.
Here it goes. SDN's architecture decouples the operational control plane from a distributed architecture where each Networking box holds its own, and centralizes it in a single device (for the sake of simplicity, I will not consider HA concerns, nor scalablity details, are those are specifics of a solution, not a model), called the SDN Controller. The goal being to make it easier in terms of Northbound interface to have customized coding, whether from an Administrator or from the application's provider, and instantly change Networking's behavior. Thus allowing for swift changes to take place in Networking, and populating new forwarding rules "<em>on the fly</em>".
Now the way I would like to have sort of the same things map into the Storage world would be something arround the following basic characteristics:
<ol>
<li><span style="font-style:inherit;line-height:1.625;">Having a centralized Control Plane (either consisting of a single controller or several), which has an Northbound API against which I can run my own scripts to customize Storage configurations and behavior. The control is not comprised by a data-plane - that stays in Storage Arrays.</span></li>
<li>Applications being able to request customized Service Levels to the Control Plane, and being able to change those dinamically.</li>
<li>Automatic orchestration and Provisioning of Storage</li>
<li>Ability to react fast to storage changes, such as failures</li>
</ol>
Now when you talk about Networking devices, one of the advantages of decoupling Control Plane from all switchs in the Network is to have stupid or thin Switchs - and consequently cheaper ones. These minimalistic (dumb) switches would simply support populating their FIB table (whether using OpenFlow or another Protocol) by their Controller, and only a few more basic protocols related to link layer control and negotiation.
However when you try to the same with the Storage Arrays, the concept gets a little more complicated. You need to worry about data redundancy (not just the box redundancy for service), as well as performance. So the only way you can treat Storage Arrays as stupid devices is to add another layer between Arrays and Hosts, where you centralize IO - in other words, a Virtualization Layer. Otherwise, your SDS Controller would just be an orchestration layer for configuration, and we've already got a buzzword for that: Cloud.
By having a Virtualization layer in between you can now start mirroring data across different Arrays, locally or in a DR perspective, thus being able to control data redundancy outside your array. You also start having better control of your Storage Service level, being able to stripe a LUN accross different Tiers of Storage (SSD, 15k SAS, 10k SAS, 7,2k NL SAS) in different Arrays, transparently to the host. Please keep in mind that this is all theoratical babel so far; I'm not saying this should be implemented in production at real life scenarios. I'm justing wondering arround the concept.
So, besides having a centralized control plain, another necessity prompts: you need a virtualization layer in between your Storage Arrays and Hosts. You might (and correctly) be thinking: we already have that among various vendors, so the next question being: are we there yet? Meaning is this already an astonishing breakthrough? The answer must be no. This is the same vision of a Federated Storage environment which isn't new at all. Take Veritas Volume Manager, or VMware VMFS.
Wikipedia states that SDS could " <em>include any or all of the following non-compulsory features:<a href="http://en.wikipedia.org/wiki/Software_defined_storage#cite_note-7">
</a></em>
<ul>
<li><em>automation with policy-driven storage provisioning - with <a title="Service-level agreement" href="http://en.wikipedia.org/wiki/Service-level_agreement">SLAs</a> replacing technology details</em></li>
<li><em>virtual volumes - allowing a more transparent mapping between large volumes and the VM disk images within them, to allow better performance and data management optimizations</em></li>
<li><em>commodity hardware with storage logic abstracted into a software layer</em></li>
<li><em>programability - management interfaces that span traditional storage array products, as a particular definition of separating "control plane" from "data plane"</em></li>
<li><em>abstraction of the logical storage services and capabilities from the underlying physical storage systems, including techniques such as in-band storage virtualization</em></li>
<li><em><a title="Scale-out" href="http://en.wikipedia.org/wiki/Scale-out">scale-out</a> architecture</em> "</li>
</ul>
VMware had already pitched its Software Defined Datacenter vision in VMworld 2012, having bought Startups that help sustaining such marketing claims, such as <a href="http://datacenternotes.wordpress.com/2013/07/11/vmware-sds-virsto-vsa-architecture/" target="_blank">Virsto</a> for SDS, and Nicira for SDN.
But Hardware Vendors are also embracing the Marketing hype. NetApp <a href="http://www.forbes.com/sites/siliconangle/2013/05/29/software-defined-netapp-always-makes-the-right-moves-when-they-count-its-software-defined-storage-sds/" target="_blank">announced</a> SDS, with Data ONTAP Edge and <a href="http://www.netapp.com/us/system/pdf-reader.aspx?cc=us&amp;m=ds-3480.pdf&amp;pdfUri=tcm:10-110653" target="_blank">Clustered Data ONTAP</a>. The way I view it, both solutions consist on using a virtualization layer with common OS. One by using a simple VSA with NetApp's WAFL OS, that presents Storage back to VMs and Servers.
<a href="http://datacenternotes.files.wordpress.com/2013/07/dataontap.pdf"> <a href="http://datacenternotes.files.wordpress.com/2013/07/dataontap_vm.png"><img class="alignnone size-full wp-image-473" alt="DataOnTap_VM" src="{{ site.baseurl }}/assets/2013/07/dataontap_vm.png" width="450" height="304" /></a></a>
The other by using a Gateway (<a href="http://www.netapp.com/us/products/storage-systems/v-series/index.aspx" target="_blank">V-Series</a>) to virtualize third-party Arrays. This is simply virtualization, still quite faraway a truly SDS concept.
IBM announcing the same, with a <a href="http://public.dhe.ibm.com/common/ssi/ecm/en/til14072usen/TIL14072USEN.PDF" target="_blank">VSA</a>.
<a href="http://www8.hp.com/us/en/products/data-storage/data-storage-technology.html?compURI=1410844" target="_blank">HP</a> is also leveraging its LeftHand VSA for Block-Storage, as well as a new VSA announced for Backup to Disk - StoreOnce VM. Again, same drill.
<span style="font-style:inherit;line-height:1.625;">Now EMC looks to me (in terms of marketing at least) as the Storage Player who got the concept best. It was </span><a style="font-style:inherit;line-height:1.625;" href="http://www.emc.com/about/news/press/2013/20130506-03.htm" target="_blank">announced</a><span style="font-style:inherit;line-height:1.625;"> that EMC will launch soon its Software Defined Storage controller - ViPR. Here is its "<a href="http://www.emc.com/collateral/data-sheet/h11750-emc-vipr-software-defined-storage-ds.pdf" target="_blank">Datasheet</a>". </span>
To conclusion: in my oppinion SDS is still far far far away (technically speaking) from the SDN developments, so as usual, renew your ACLs for this new marketing hype.
