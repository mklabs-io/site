---
layout: post
title: VMware NSX
date: 2013-08-26 22:41:09.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Networking
tags:
- Network Virtualization
- VMware
meta:
  _edit_last: '51216153'
  _publicize_pending: '1'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/08/26/vmware-nsx/"
---
Its VMworld show time, which means awesome stuff being announced. VMware's Overlay Network and SDN Sagas are clearly just starting, and so VMware announced today its Network Virtualization Solution: <a href="http://blogs.vmware.com/networkvirtualization/2013/08/vmware-nsx-network-operations.html" target="_blank">NSX</a>.
I haven't been able to grasp any VMware own technical material yet, however from what I could until so far understand we are talking about the first true signs of a Nicira's integration/incorporation into current Vmware Networking Portfolio. VMware's Overlay Networking technology - VXLAN -  has been integrated with Nicira's NVP platform. However, this is not mandatory, so you do get 2 additional flavors of encapsulation to choose among VXLAN: GRE and STT.
As it can be seen from <a href="http://www.vmware.com/files/pdf/products/nsx/VMware-NSX-Datasheet.pdf" target="_blank">NSX datasheet</a>, key features are similar to Nicira's NVP plataform:
"
<em>• Logical Switching – Reproduce the complete L2 and L3 </em>
<em>switching functionality in a virtual environment, decoupled </em>
<em>from underlying hardware</em>
<em>• NSX Gateway – L2 gateway for seamless connection to </em>
<em>physical workloads and legacy VLANs</em>
<em>• Logical Routing – Routing between logical switches, </em>
<em>providing dynamic routing within different virtual networks. </em>
<em>• Logical Firewall – Distributed firewall, kernel enabled line </em>
<em>rate performance, virtualization and identity aware, with </em>
<em>activity monitoring </em>
<em>• Logical Load Balancer – Full featured load balancer with </em>
<em>SSL termination. </em>
<em>• Logical VPN – Site-to-Site &amp; Remote Access VPN in software </em>
<em>• NSX API – RESTful API for integration into any cloud </em>
<em>management platform"</em>
Ivan Pepelnjak gives very interesting preview-insight on <a href="http://blog.ipspace.net/2013/08/what-is-vmware-nsx.html" target="_blank">what's under the hood</a>. Though new, integration with Networking partners is also being leveraged. Examples are: <a href="http://www.vmware.com/files/pdf/products/nsx/vmw-nsx-arista-networks.pdf" target="_blank">Arista</a>,  <a href="http://www.vmware.com/files/pdf/products/nsx/vmw-nsx-brocade-networks.pdf" target="_blank">Brocade</a>, <a href="http://www.vmware.com/files/pdf/products/nsx/vmw-nsx-cumulus-networks.pdf" target="_blank">Cumulus</a>, <a href="http://www.vmware.com/files/pdf/products/nsx/vmw-nsx-dell-systems.pdf" target="_blank">Dell</a>, and <a href="http://www.vmware.com/files/pdf/products/nsx/vmw-nsx-juniper-networks.pdf" target="_blank">Juniper</a> on the pure Networking side, and <a href="http://www.vmware.com/files/pdf/products/nsx/vmw-nsx-palo-alto-networks.pdf" target="_blank">Palo Alto Networks</a> on the security side.
Exciting time for the Networking community!
