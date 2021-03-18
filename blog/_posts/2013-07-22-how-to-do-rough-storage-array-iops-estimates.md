---
layout: post
title: How to do rough Storage Array IOPS estimates
date: 2013-07-22 09:24:20.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Storage
tags:
- Storage Arrays
meta:
  _edit_last: '51216153'
  geo_public: '0'
  _oembed_215c9772260dd2e341380775a686cabf: "{{unknown}}"
  tagazine-media: a:7:{s:7:"primary";s:0:"";s:6:"images";a:0:{}s:6:"videos";a:0:{}s:11:"image_count";i:0;s:6:"author";s:8:"51216153";s:7:"blog_id";s:8:"53483832";s:9:"mod_stamp";s:19:"2013-07-22
    09:24:20";}
  _oembed_3ea0ff139127bf70a48d3250c771aa24: "{{unknown}}"
  _oembed_9944c885593ca893a7683c522b755ddc: "{{unknown}}"
  _oembed_b29fc941679675b74b6ea41ccf65ca47: "{{unknown}}"
  _oembed_21bf7b466953703f392fc5dbcbb94f97: "{{unknown}}"
  _oembed_25df8127e908a9ddcd5ba88312189d25: "{{unknown}}"
  _oembed_6ee2e60824c8e72a7859ecaccc2c28bc: "{{unknown}}"
  _oembed_ea6f361f20c72f2e5c41de60def3d0e9: "{{unknown}}"
  _oembed_7afac42386fe32b762de754c4856d215: "{{unknown}}"
  _publicize_job_id: '14895295003'
  _oembed_7ccb785d9a0975d6254e29924fcce687: "{{unknown}}"
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/07/22/how-to-do-rough-storage-array-iops-estimates/"
---
This post is dedicated to trying to get around the mystery factor that Cache and Storage Array algorithms have, and helping you calculating how many disks you should have inside your Storage Array to produce a stable average number of disk operations per second - IOPS.
Face it, it is very hard to estimate performance - more specifically IOPS. Though throughput may be high in sequential patterns, Storage Array face a different challenge when it comes to random IOPS. And from my personal experience, Array-vendor Sales people tend to be over-optimistic when it comes to the maximum IOPS their Array can produce. And even though their Array might actually be able to achieve a certain high maximum value of IOPS with 8KB blocks, <em>that does not mean you will in your environment</em>.
<strong>Why IOPS matter</strong>
A lot of factors can affect your Storage Array performance. The first typical factor are the very random traffic and high output patterns of databases. It is no wonder  this is the usual first use-case for SSD. Online Transaction Processing (OLTP) workloads, which by having verified writes (write and read back the data) double IOPS, and since it has a high speed demand, can be source of stress for Arrays.
Server Virtualization is also a big contender, which produces the "<em>IO blender effect</em>". Finally Exchange is also a mainstream contender for high IOPS, though the architecture since Microsoft version 2010 changed the paradigm for storing data in Arrays.
These are just some simple and common of the many examples where IOPS can be even more critical than throughput. This is where your disk count can become a critical factor, and to the rescue when that terabyte of Storage Array cache is lost and desperately crying out for help.
<strong>Monkey-math </strong>
So here are some very simplistic grocery-style type of math, which can be very useful to quickly estimate how disks you need in that new EMC/NetApp/Hitachi/HP/... Array.
First of all IOPS variate according to the disk technology you use. So in terms of Back-end these are the average numbers I consider:
<ul>
<li>SSD - 2500 IOPS</li>
<li>15k HDD - 200 IOPS</li>
<li>10k HDD - 150 IOPS</li>
<li>7.2k HDD - 75 IOPS</li>
</ul>
Total Front-End IOPS = C + B , where:
C stands for total number of successful Cache Hit IOPS on reads, and B for total IOPS you can extract from your disk backend (reads + writes). Their formula is:
C = %Cache-hit * %Read-pattern
B = (Theoretical Raw Sum of Back-end Disk IOPS) * %Read-pattern + (Theoretical Raw Sum of Back-end Disk IOPS)/(RAID-factor) * %Write-pattern
C is the big exclamation mark on every array. It depends essentially on the amount of Cache the Array has, on the efficiency of its Algorithms and code, and in some cases such as in EMC VNX the usage of helping technologies such as FAST Cache. This is where your biggest margin of error lies. I personally use values between 10% up to 50% maximum efficiency, which is quite a big difference, I know.
As for B, you have to take into consideration the penalty that RAID introduces:
<ul>
<li>RAID 10 has a 2 IO front-end penalty: for every write operation you will have one additional Write for data copy. Thus you have to halve all Back-End IOPS, in order to have the true Front-End IOPS</li>
<li>RAID 5 has a 4 IO back-end penalty: for every write operation, you have 2 reads (read old data + parity) plus 2 writes (new data and parity)</li>
<li>RAID 6 has a 6 IO Back-ned penalty: for every write operation, you have 3 reads (read old data + parity) plus 3 writes (new data and parity)</li>
</ul>
Say I was offered a mid-range array with two Controllers, and I want to have about 20.000 of IOPS out of 15k SAS HDD. How many disks would I need?
First the assumptions:
<ul>
<li>About 30% of average cache-hit success on reads (which means 70% of reads will go Back-end)</li>
<li>Using RAID 5</li>
<li>Using 15k HDD, so about 200 IOPS per disk</li>
<li>60/40 % of Read/Write pattern</li>
</ul>
Out of these 20.000 total Front-End IOPS, Cache-hit percentage will be:
C = 20.000* %Read * %Cache-hit = 20.000 * 0,6 * 0,3 = 3.600
Theoretical Raw Sum of Back-end Disk IOPS = N * 200
Thus, to arrive at the total number of disks needed:
20.000 - 3.600 = (N*200)*0,6 + (N*200/4) *0,4
Thus N = 117.14 Disks.
So about 118 disks.
&nbsp;
Hope this helped.
