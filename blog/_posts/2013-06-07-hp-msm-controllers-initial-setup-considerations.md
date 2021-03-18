---
layout: post
title: HP MSM Controllers Initial Setup Considerations
date: 2013-06-07 19:36:11.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Networking
tags:
- HP Wireless
- Wireless
meta:
  _wp_page_template: default
  _edit_last: '51216153'
  _publicize_pending: '1'
  tagazine-media: a:7:{s:7:"primary";s:82:"http://datacenternotes.files.wordpress.com/2013/06/hp-msm-controllers-services.png";s:6:"images";a:2:{s:82:"http://datacenternotes.files.wordpress.com/2013/06/hp-msm-controllers-services.png";a:6:{s:8:"file_url";s:82:"http://datacenternotes.files.wordpress.com/2013/06/hp-msm-controllers-services.png";s:5:"width";i:960;s:6:"height";i:540;s:4:"type";s:5:"image";s:4:"area";i:518400;s:9:"file_path";b:0;}s:61:"http://datacenternotes.files.wordpress.com/2013/06/msm720.png";a:6:{s:8:"file_url";s:61:"http://datacenternotes.files.wordpress.com/2013/06/msm720.png";s:5:"width";i:821;s:6:"height";i:267;s:4:"type";s:5:"image";s:4:"area";i:219207;s:9:"file_path";b:0;}}s:6:"videos";a:0:{}s:11:"image_count";i:2;s:6:"author";s:8:"51216153";s:7:"blog_id";s:8:"53483832";s:9:"mod_stamp";s:19:"2013-06-07
    21:12:22";}
  _format_link_url: ''
  _format_quote_source_url: ''
  _format_url: ''
  _format_quote_source_name: ''
  _format_image: ''
  _format_gallery: ''
  _format_audio_embed: ''
  _format_video_embed: ''
  _post_restored_from: a:3:{s:20:"restored_revision_id";i:50;s:16:"restored_by_user";i:51216153;s:13:"restored_time";i:1371126076;}
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/06/07/hp-msm-controllers-initial-setup-considerations/"
---
HP MSM Controllers have the following services: DHCP (tied to the LAN portal essentially), Access Controller, RADIUS (client and server), Router (tied to access controller), VPN (tied to the Internet port/IP Interfaces except the LAN port), Bandwidth control (tied to the Internet port/IP Interfaces except the LAN port), NAT (tied to the Internet port/IP Interfaces except the LAN port), and stateful Firewall (tied to the Internet port/IP Interfaces except the LAN port). The fact that the services are tied to physical ports, or have some dependencies, has some consequences that you should be aware of:
<ul>
<li><strong>About the Route</strong>r: it is tied to the Access Controller (AC), meaning it <em>only</em> routes traffic to or from access-controlled clients, and does not route traffic for other devices. Also note that it does not do any routing on its own behalf, so you must guarantee it has a default gateway,</li>
<li><strong>About the AC:</strong> Responsible for redirecting access-controlled users to a login portal and for controlling the user's traffic before and after Auth.</li>
<li><strong>About the DHCP Server:</strong> though the DHCP server service is tied to the LAN port, it is also intended to provide IP addresses to guest traffic tunneled to the Access Controller. Thus, you may be using the Internet Port to receive tunneled traffic and still use the internal DHCP service to provide IP to guests. None the less, you can also relay DHCP requests from client tunneled traffic to your Network DHCP, if you do not want to use the controller's DHCP server. You might also be wondering how to choose which port is used to tunnel access controlled client traffic? Easy - the MSM solution always tunnels client traffic to the same interface where AP management traffic tunnels are established.</li>
<li><strong>About the Internet Port:</strong> when traffic is routed out the Internet port the controller can apply bandwidth control, NAT services, and stateful firewall services.</li>
<li><strong>About the LAN Port: </strong> <em>untagged</em> LAN port interface supports DHCP services. When DHCP is enabled, Controller responds to all untaggedDHCP discovery requests on the LAN port. AC traffic can also be routed out the LAN port, however you loose bandwidth control, NAT and FW services. For this reason, one should not design solutions to route access controlled traffic out this interface. The controller treats devices on the untagged LAN port as access-controlled clients.</li>
</ul>
<strong>Architecture Planning</strong>
HP MSM solution allows you to distribute data forwarding from APs directly to destination, avoiding hitting all traffic on Controllers. This way you can enhance considerably performance of traffic forwarding. You basically do this decision when creating what HP calls a Virtual Service Community (VSC), which is kind of a virtual profile of the service for traffic forwarding (most of the times it is simply the settings associated with a SSID).
In the creation of a VSC you choose if you want to use the controller for Authentication, and if you want additionally to use the controller for Access Control. When you enable use the controller for Auth., only Authentication traffic is tunneled to the controller, whether using pre-shared keys or 802.1X; however data traffic is forwarded directly to its destination. You might centralize Authentication on the controller even when you are using an external RADIUS, in order to simply the setup by having a single entity talking to the RADIUS server.
Only when using the Access Controller is all traffic forwarded to the controller.
<img class="alignnone size-medium wp-image-125" style="border-color:#bbbbbb;background-color:#eeeeee;" alt="VSC" src="{{ site.baseurl }}/assets/2013/06/vsc.gif?w=300" width="300" height="188" />
This solution is best indicated for providing guests a portal for Web Auth, while simultaneously using Bandwidth control and Firewall for example for such clients. However, when you use  a NAC solution, such as HP IMC-UAM 5.2, you should not enable Access Controll on the controller - you rather use the NAC solution as the controller's RADIUS server.
Also note that the egress VLAN (VLAN in which traffic is forwarded to) in this case is different than in a distributed architecture. In a distributed architecture the egress VLAN is simply the VLAN where the AP forwards traffic to. So in the case of a distributed architecture (read non-access-controlled traffic) you can do this by specifying a VSC binding:
<ul>
<li>Select the group of APs to which you want to apply the settings.</li>
<li>Select "<em>VSC bindings</em>", and "<em>Add a new binding...</em>"</li>
<li>Associate the configured VSC with an egress network.</li>
</ul>
<a href="http://datacenternotes.files.wordpress.com/2013/06/vsc-binding.jpg"><img class="alignnone size-medium wp-image-129" alt="VSC binding" src="{{ site.baseurl }}/assets/2013/06/vsc-binding.jpg?w=300" width="300" height="162" /></a>
If you do not specify a binding, the VSC simply forwards traffic to the APs default VLAN, if there is not any user-defined VLAN to forward to.
However Access-controlled VSCs are a little bit more tricky. As far as I understand you have two main options. <span style="text-decoration:underline;">In the first option</span> clients in access-controlled VSC can be routed essentially to any subnet that the controller, and also furthermore by using the controller's gateway.
You can achieve this by either specifying a subnet for the clients using the controller's DHCP server on the LAN port or from tunneled traffic (thus the recommendation to always tunnel traffic to simplify setup on the rest of the network infrastructure), or you use DHCP relay for clients to acquire IP. For the same type of reasoning you are advised to check the tunnel traffic box, you are also advised to activate NAT service here, so that after controller has routed traffic out to its Gateway, the gateway knows how to return back the traffic. Alternatively, you can configure that gateway and add a route.
<span style="font-style:inherit;line-height:1.625;"><span style="text-decoration:underline;">Option two</span> is focused on restricting access-controlled clients reachability. You do it by using the the egress VLAN. This makes sense for instance when you want guests to be routed directly to the Internet, without accessing any corporate resources. Considering that the Controller only routes traffic, the guest will not have an IP on the egress VLAN, because the controller does not simply bridge traffic as APs do for VSC bindings. Instead it will still have an IP on the specified subnet for these clients using internal DHCP, but the traffic will be routed to the controller's Gateway.</span> So you must be aware that in  it is mandatory for you to have the following conditions guaranteed when implementing an Egress network for access-controlled VSCs:
<ul>
<li>specify a tagged VLAN to be the egress VLAN. It cannot be any of the untagged VLAN. If you want traffic bandwidth control, then you must specify a tagged VLAN on the Internet port.</li>
<li>The VLAN must have an IP interface, as well as a gateway.</li>
<li>It is recommended to specify NAT for the IP interface. This is because you have to guarantee that the default gateway for that VLAN can route traffic back to the clients subnet, which can be greatly simplified if you use NAT.</li>
</ul>
<span style="font-style:inherit;line-height:1.625;">Finally, there is another varient in access-controlled VSCs, which includs adjusting the solution so that clients receive their IP in the egress VLAN they are ejected to (thus resembling the egress VLAN concept in non-access-controlled traffic, where APs bridge traffic to a certain VLAN). This solution involves using DHCP relay. Here are the guidelines:</span>
<ul>
<li>Apply the egress VLAN for unauthenticated and authenticated clients in the VSC</li>
</ul>
<a style="font-style:inherit;line-height:1.625;" href="http://datacenternotes.files.wordpress.com/2013/06/vsc-egress-mapping-ac-traffic.jpg"><img class="alignnone size-medium wp-image-130" alt="VSC egress mapping AC traffic" src="{{ site.baseurl }}/assets/2013/06/vsc-egress-mapping-ac-traffic.jpg?w=300" width="300" height="145" /></a>
<ul>
<li>In the global DHCP relay settings, select the checkbox for extending the ingress interface to the egress interface. After this is enabled, you will no longer be able to specify the IP address and subnet mask settings on VSCs.</li>
<li>Disable NAT on the egress VLAN IP interface.</li>
<li>Set the MSM controller's IP address for the default gateway and DNS server (it will forward them to the correct ones).</li>
</ul>
In sum, the egress VLAN here simply specifies a specific default route for the controller to use for the intended clients, thus limiting client's access to corporate resources.
<strong>Basic info about MSM Controllers</strong>:
<ul>
<li>HTTP/HTTPS supported to access Web Page, by default on standard ports 80 and 443; can be later on changed. If changed, then:  <span style="font-style:inherit;line-height:1.625;">http: //&lt;controller IP </span><span style="font-style:inherit;line-height:1.625;">address&gt;:port or https: //&lt;controller IP address&gt;:port</span></li>
<li>Default Controller IP Address: 192.168.1.1/24.</li>
<li>Default Credentials: User: admin  Password: admin</li>
<li>SNMP (UDP 161) and SOAP (TCP448) only on LAN port.</li>
<li>AP discovery (UDP 38212): only on Access Network (for MSM720), and LAN port and IP interfaces on the LAN port (for the rest of MSM Controllers)</li>
<li>Internal Login Pages access from any interface, except if you use NOC-based Auth., in which case you should enable the Internet Port</li>
<li>For security reasons, you can and should restrict the management access to a single interface. This operation can be done by the Web Management Interface.</li>
</ul>
To add VLANs to Controller ports - configure "Network Profile", which entails a VLAN ID and can entail an IP interface when assigned to a port. A network profile does not have any effect until you apply it to a port. Furthermore you can only apply an IP interface to a VLAN ID after you have associated the network profile to a specific interface. Note also that though you can apply several profiles to each physical interface, the services tight to the particular interface do not change.
A Network profile assigned to a port with an IP interface is called a non-default IP interface to distinguish from the (untagged) Internet port. Like the (untagged) Internet port, non-default IP interfaces do not map to the Access Controller - unless special tunneling is involved. However, the Controller can route traffic to and from access-controlled clients and any IP interface (independently from which physical port this interface is assigned to). Again, note that the controller will only route traffic related to access-controlled devices.
A port can only have one untagged profile, and supports more tagged profiles. On MSM760 and 765 you can only add new profiles for tagged traffic. Have in consideration that<span style="font-style:inherit;line-height:1.625;"> the physical ports are only routed ports, and thus you cannot switch traffic in a VLAN between ports. You cannot assign the same network profile to two different ports, but you can assign the same VLAN ID in two different ports. So be aware of what this means, and avoid doing so unless you have different subnets on the two ports.</span>
By default management (of the controller) is enabled on the Internet Network Profile, and is thus usually used for such purpose to make setup easier. You can eanble it on a different interface; but just make sure that for security reasons you only enable it on one interface. Be sure to enable protocols like SOAP (essential for HP NAC solution with IMC-UAM module), SNMP, DNS and SNTP on the appropriate management interface.
<strong>MSM720 technicalities</strong>
Though until so far I have been treating all MSM Controllers equally, the MSM720 does not function exactly like other controllers. To start with it has a total of six ports 10/100/1000 Mbps (unlike others that have only two), which act like switch ports (as opposed to other controllers which act as routing ports).
<a href="http://datacenternotes.files.wordpress.com/2013/06/msm720.png"><img class="alignnone size-medium wp-image-53" alt="MSM720" src="{{ site.baseurl }}/assets/2013/06/msm720.png?w=300" width="300" height="97" /></a>
You can aggregate ports with static port trunk or dynamic LACP, and assign network profiles as untagged and tagged to multiple ports or trunks.
Like other Controllers it uses global Network profiles. However, every profile that is mapped to a port requires a VLAN ID. When you assign the profile to a port, you select whether the associated VLAN is tagged or untagged on that port. The MSM720 only accepts tagged traffic for which the ID is configured as a tagged VLAN on the port.
The MSM720 has two network profiles at factory defaults. Called the Access network
and the Internet network, these profiles roughly correspond to other controllers’ LAN
port network (untagged) and Internet port network (untagged) profiles, respectively.
However, the profiles have associated VLAN IDs, the Access network’s default ID
being 1 and the Internet network’s being 10.
Similar to the other controllers, you can also associate more network profiles to the controller's ports. However, in MSM720 these can be either tagged or untagged. Note that the basec functioning rule still applies: a port can only have one untagged VLAN. So by assigning a untagged profile, you will consequently be removing the former untagged profile from that port.
<strong>AP Deployment Considerations</strong>
There are essentially 3 main ways for deploying APs on the network: two where APs discover the Controller by L2 broadcast, one by L3 discovery.
The first deployment is when you deploy a dedicated subnet or VLAN for AP's management, and different and dedicated for Controller management, for security reasons. It also allows you to isolate the MSM APs management traffic from other network traffic. To make things easier, ensure that the Controller also supports the APs dedicated VLAN, tagged on one (or more in case of MSM720) of its ports, in order to support L2 discovery. This is the best option to consider.
The second option consists of deploying both Controller and APs on the same VLAN. Usually this happens when companies do not want to change their network infrastructure by adding a new VLAN.
Though normally companies prefered to use their own DHCP server to contralize all scope management, you can also use the Controller's own DHCP service to provide IP for the APs. In option 1, the DHCP server admin should create a new scope (or pool) for the MSM APs. The network admin should also configure DHCP server relay on the routing switch that acts as gateway for the new VLAN.
Note: DHCP service can only be used when controllers are not teamed (in other words, in clustering mode for HA). Also you should be careful not to enable the DHCP service on the   same subnet where another DHCP server is working on.
The third and last option is usually deployed when APs are deployed in remote sites. Communication between APs and the Controller is done in L3, so you must either use DHCP option 43 attribute, by DNS resolution (which implies having the default controller name used in your DNS), or configure manually on the AP the IP address of the controller.
Please note that these options can be combined.
Also for security reasons you might prefer having MSM APs connect to tagged ports on the switch, aswell as implementing 802.1X on the switch port, since MSM APs support acting as supplicants.
Disclamer: note that these are <em>my</em> own notes. HP is not responsable for any of the content here provided.
