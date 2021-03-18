---
layout: post
title: 'AWS: Steps to get a free VM up & running'
date: 2013-07-12 16:53:18.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Cloud
- Computing
tags:
- AWS
meta:
  _edit_last: '51216153'
  _publicize_pending: '1'
  geo_public: '0'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2013/07/12/aws-steps-to-get-a-free-vm-up-running/"
---
This post is intended to be a very simplistic post on how to get you started with Amazon Cloud (AWS) with a VM. It will allow you to start experimenting AWS, without any costs (if you take the right steps). Naturally Amazon itself provides more detailed steps <a href="http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html?r=1874" target="_blank">here</a>.
What you will need:
<ul>
<li><strong>email account </strong></li>
<li><strong>cellphone</strong> - for security reasons, to make sure you're not hosting CPU power for any DDoS and other type of stuff</li>
<li><strong>credit card</strong> - though you actually need to input credit card details, you can indeed run a free VM in a limit amount of time. Amazon will simply not charge you anything for it, as long as you stick to the greenzone.</li>
</ul>
Overview of the process:
<ol>
<li>Sign up for AWS account.</li>
<li>Launch a "t1.micro instance"</li>
<li>Beware of the 750 hours free-use limit.</li>
</ol>
Simple, right? Here are more detailed steps:
<ol>
<li>Go to aws.amazon.com and click on "Sign up".</li>
<li>Next enter your email address and select "I am a new user."</li>
<li>Enter Login credentials</li>
<li>Enter your contact information</li>
<li>Enter your payment information</li>
<li>Next you will have an Identity verification through cellphone, which consists of being contacted by an automated system that prompts you to enter the PIN number they provided you.</li>
<li>After that, it may take while until your account is actually activated.</li>
<li>Next go to aws.amazon.com/products and click on the "My account Console", and "AWS Management Console". After you successfully login, you'll land at this page:<a style="font-style:inherit;line-height:1.625;" href="http://datacenternotes.files.wordpress.com/2013/07/aws-portfolio_ii.png"><img class="alignnone size-full wp-image-306" alt="AWS portfolio_II" src="{{ site.baseurl }}/assets/2013/07/aws-portfolio_ii.png" width="584" height="393" /></a></li>
<li>Click on "EC2  Virtual Servers in the Cloud".</li>
<li>You will land on the EC2 Dashboard. Right in the middle click on "Launch Instance".</li>
<li>On the "Create a New Instance" menu select "Quick Launch Wizard"</li>
<li>Then Select "Create new", enter the name of your new Instance (i.e. VM) and select for instance an Ubuntu, to make sure you stay on the free-way. Nowadays you have two versions of Ubuntu Server available: 12.04.2 LTS and 13.04. If you are following specific tutorials with the previous version of ubuntu, then you might prefer to choose that one. <strong>Before </strong>hitting continue, make sure you click on the "Download" button. You will download a "pem" file (<a href="http://en.wikipedia.org/wiki/X.509" target="_blank">Privacy Enhance Mail</a>), which is the certificate you will need to sucessfully establish a console session to that VM you're about to launch. <strong>After</strong> downloading that file, hit "Continue".</li>
<li>On the next screen you will be able to confirm that the type of instance you are launching is a "t1.micro". It is very important you do not change this, if you want to stick with the free experimenting. Hit launch.</li>
<li>Your VM might take a few minutes until the orchestration on Amazon backend is completed. When the initialization is completed, right-click on your instance and select connect.</li>
<li>Select "Connect with a standalone SSH Client". You will have instructions provided by Amazon there. Make sure you copy the command line provided by Amazon, and launch a terminal session. In Windows I suggest you use <a href="http://www.putty.org/" target="_blank">Putty</a>. If you're using a Mac, just go to "Applications &gt; Utilities &gt; Terminal".</li>
<li>On Mac: Now this part might be a little more tricky if you're not used to CLI. You need to change the permissions of that "pem" file you downloaded, so change to the directory where you stored the pem file (Usually "Downloads" directory), and change permissions. Enter the following lines, where the text contained in each quotes is a different line: "cd ~/Downloads" , "chmod 400 <em>name-of-the-pem-file-you-downloaded.pem</em>".</li>
<li>Still on Mac: Now is the time to paste the line you copied from Amazon. Something like "ssh -i <em>name-of-the-pem-file-you-downloaded.pem ubuntu@name-of-your-instance.compute-1.amazonaws.com"</em></li>
<li>On windows: <a href="http://blog.linuxacademy.com/linux/connect-to-amazon-ec2-using-putty-private-key-on-windows/" target="_blank">follow this tutorial to connect with Putty</a>.</li>
<li>There! You should have a Welcome page in your terminal console from Ubuntu.</li>
</ol>
Up and running. It is very important for you to note that you <span style="text-decoration:underline;">keep your total usage of t1.micro instances under 750 hours per month</span> to avoid charges from Amazon. In order to do that, you have to terminate (yes delete everything!) your instance. You might not be using CPU power, but you will still be using Storage, which is also included in Amazon's business...
Now its time for experimenting things. Why not start with a <a href="http://www.howtoforge.com/ubuntu_lamp_for_newbies" target="_blank">LAMP</a>?
