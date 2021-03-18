---
layout: post
title: Getting started with AWS IoT and a dockerized device
date: 2018-09-09 18:50:40.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- AWS
- Big Data
- docker
- IoT
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '21965704601'
  timeline_notification: '1536519048'
  _thumbnail_id: '4116'
  _wp_old_slug: getting-started-with-an-aws-iot-with-a-dockerized-device
  _oembed_ac5bc77d7a31798256a6072e3c4bd2e6: "{{unknown}}"
  _oembed_02d50030d0094d9a1faae013fee8f07c: "{{unknown}}"
author:
  login: jrcferrao
  email: joao@mklabs.io
  display_name: João Ferrão
  first_name: ''
  last_name: ''
permalink: "/2018/09/09/getting-started-with-aws-iot-and-a-dockerized-device/"
---
IoT isn't a new term and has, actually, been one of the buzz words of the XXI century, right alongside Crypto Currency. The two of them are seen holding hand in hand in the good and the bad, from interesting crypto projects focused on IoT usage, or actual IoT devices being hacked to mine crypto currencies. But this article is actually about IoT: it seems the last few years have given us the perfect momentum between three main ingredients for its popularity: the exponential increase of network capable devices, the easily available processing power and, finally, the hungry-for-data dynamic we from most parties nowadays.
For someone working as a Data Engineer (or related) there isn't there much of a more end-to-end project than one which goes from setting up an actual no-so-intelligence device, design the methods to connect these devices to a network and, still, go about designing all streaming/batch processing infrastructure needed to deal with all the data. It really is a huge challenge.
In the following lines, I'll focus on how AWS IoT has come to help in the first and second challenge and show an example of <strong>how to emulate an actual IoT device</strong> with the help of docker and walk through some of the nice and easy features AWS IoT offers such as IoT Core, management, topics and rules as well as integration with other AWS services such as Kinesis, Firehose or Lambdas.
<!--more--><a href="https://aws.amazon.com/blogs/aws/aws-iot-now-generally-available/">AWS IoT stack has been generally available since the end of 2015</a> with a very comprehensive list of services available. Looking at the AWS IoT catalog, one might get somehow overwhelmed at all the bits and pieces:
<img class="alignnone size-full wp-image-4110" src="{{ site.baseurl }}/assets/2018/09/screen-shot-2018-09-06-at-11-15-27.png" alt="AWS IoT Service Offering" width="1906" height="1880" />
<h2>The Juice</h2>
First question I asked myself when I started thinking about this service: but what exactly are the device requirements to be able to connect it to IoT? Or do all devices can connect? And this question isn't totally answered upfront.
Basically, your device you want to connect to IoT must meet one of the following requirements:
<ol>
<li>Have <strong><a href="https://www.freertos.org/">FreeRTOS</a></strong> with whom AWS has partnered with;</li>
<li>Be compatible with any of the following languages: Python, Java, JavaScript;</li>
<li>Be one of these devices: iOS, Android, Arduino</li>
<li>Be able to run AWS Greengrass.</li>
</ol>
I believe they are all pretty clear, in terms of requirements, except for nr 4. Greengrass is software that runs on your device and, for this reason, is conceived for more-than-network-capable devices.
Out of of all the options, it's the one which provides the developer the most seamless experience for the AWS services, even allowing them to trigger lambdas
<h2>IoT Core</h2>
This is the actual gem of the service. Call it the console or the service that really exposes the cloud potential of having the devices (correctly) connected to it. When you connect a device to AWS IoT Core you get some very interesting features:
<h3><strong>Topics</strong></h3>
Using one of tools mentioned in the previous sections, your device(s) can be programmed to send data in a Queue-like fashion, meaning, you can have different types of information being set to different set of topics.
<h3><strong>Device Shadows</strong></h3>
If configured correctly, one of those topics, which is automatically created per device connected, is reserved for <strong>shadow updates</strong>. What is a shadow you may ask: well, it's the topic that will concentrate the status of the device that yourself can define what is. Additionally, this is also where you would push your "desired" status back into the device because, well, IoT might and should be more interesting when communication happens both ways.
For example, if you have an Arduino with a RBG (multi colour) LED, you might want to know the correct status of the device on the shadow and check it is Red. If you want to change it to blue, the action would be made on the Shadow of the device.
<strong>But that's not all</strong>, if the device is offline and you push a "new" desired state, the shadow will make sure to push this command to the device as soon as it connects - another headache taken from the development team.
<h2>IoT Analytics</h2>
Also awesome part of the service offering. The features for this are endless but the one that I most liked was the fact that you can define rules for what to do on what comes our of the AWS Topic(s) <strong>and</strong> you can do so using SQL-like language. Moreover, there are no limits to how many rules you can run on each topic and they are all processed at the same time.
It then becomes really easy to choose sending information to a Kinesis Stream of Firehose so data ends up on an S3 bucket or even define a lambda to be triggered. The possibilities of having seamless integration with remainder of the AWS world are very easy to conceive.
<h2>Other services</h2>
<h3>IoT Button</h3>
AWS IoT Button is a physical button which is instantly compatible and easily configurable on AWS IoT core features. With this AWS owned piece of HW, you as developer, can start programming your favourite lambda functions triggered by the actual push of this button. It's interesting if you have a really interesting use case or if you just like the rush of launching endless EC2 instances with a push of a button.
Companies started using this so customers would setup the button at their homes to order specific products out of Amazon marketplace, such as washing detergent, branding each button to their product.
Although this might seem interesting, the device only has "relevant" information to push when the button is either pressed or released, instead of, say, a car which has the potential to have hundreds of sensors push data at any moment.<img class="alignnone size-full wp-image-4112" src="{{ site.baseurl }}/assets/2018/09/aws_iot_button-62b50cbaeae340c18aa411d87c86029e188caab0.png" alt="aws_iot_button-62b50cbaeae340c18aa411d87c86029e188caab0" width="1000" height="1000" />
<h3>IoT 1-click</h3>
It's essentially the same service but unbound to any hardware. If you have an Arduino lying around with ON/OFF switch or similar, you can configure it with the SDK to simply trigger the very type of actions described in the previous section.
<h1>Now that you, too, have been introduced</h1>
Now that you know what you're up against, let's get started with emulating a IoT device. The code can be found in this <a href="https://bitbucket.org/joaoferrao/datacenternotes-docker-iot-elf-simulator/.">repo.</a> It's a modification (and simplification for bootstrap) or <a href="https://github.com/aws-samples/aws-iot-elf">AWS's original release here</a>. They call this service <strong>ELF,</strong> for <strong>Extremely Low Frequency</strong> device.
The code available on the repository will instantiate a docker container which will create all the needed resources on AWS IoT through API calls of boto3 and will start sending data.
<h3>Requirements</h3>
<ul>
<li>Your <strong>AWS Profile</strong>, preferably with permissions to create new profiles (to limit our IoT device permissions)</li>
<li>A second AWS Profile to which you have access to AWS KEY and AWS SECRET so you can attach it the IoT policy</li>
<li><strong>Docker </strong>&amp;<strong> docker-compose</strong> - in the remote case you don't have it, <a href="https://docs.docker.com/docker-for-mac/install/">installation is very straightforward</a></li>
<li><strong>.env</strong> in repo's root dir (literally, just <strong>.env</strong> with credentials). Check the <strong>.exampleenv</strong> to see what information is needed inside the file.</li>
</ul>
<h2></h2>
<h3>AWS ELF Profile</h3>
As expressed earlier, as a best practice, it's advisable to create a dedicated AWS profile for your ELF, so you can limit the permissions to this particular (or related) experiments.
Go the AWS IAM console and, after creating a new user with Access Keys, attach the following policy:
```{bash}
{
Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ELF",
            "Effect": "Allow",
            "Action": [
                "iot:AttachPrincipalPolicy",
                "iot:AttachThingPrincipal",
                "iot:CreateKeysAndCertificate",
                "iot:CreatePolicy",
                "iot:CreateThing",
                "iot:CreateTopicRule",
                "iot:DeleteCertificate",
                "iot:DeletePolicy",
                "iot:DeleteThing",
                "iot:DeleteTopicRule",
                "iot:DescribeEndpoint",
                "iot:DetachPrincipalPolicy",
                "iot:DetachThingPrincipal",
                "iot:ReplaceTopicRule",
                "iot:UpdateCertificate",
                "iot:UpdateThing"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}```
<h3>Create .env file</h3>
Now create a file .env in the root directory of the repo with the following content:
```{bash}
AWS_ACCESS_KEY_ID=AWS_ACCESS_KEY_HERE_NO_QUOTES
AWS_SECRET_ACCESS_KEY=AWS_SECRET_KEYHERE_NO_QUOTES
AWS_DEFAULT_REGION=eu-west-1
```
<h2>Source code fundamentals</h2>
Most of the magic happening in the docker/elf/elf.py file which was also created by the AWS team. It's written in Python 2.7, which is not desirable, but it's OK. The script uses argparse to accept params and individual json messages.
I took the liberty of tweaking the following two functions to parse the entire json files and spit instead line by line:
https://gist.github.com/joaoferrao/360917cb735c50889fec03009da82d2d
The first function is OK to change the name. However, the run() method belongs to class ElfPoster which in turn inherits from ElfThread. This makes it so that we actually need to override that very same method. Both changes follow below in which the <strong>msg</strong> is basically transformed into a List and then the run() method will read line by line until the time is up. By default, the ELF will stream data during 120 seconds.
https://gist.github.com/joaoferrao/b2cec776aff8fad7fbc8fba37c3fd07d
Finally, you can take a look around in the argparse arguments to check what are some of the options you can also use to tweak the script. Important to this article, is the fact that the messages are going to be published to a topic named <strong>elf/</strong>.
<h2>Launch the ELF container and make it stream!</h2>
Before proceeding, make sure you have your AWS_PROFILE activated in the terminal window you are about to use. You can check this by running <strong>echo $AWS_PROFILE</strong>. If it comes back empty or with a different named profile than expected, you can run the following and you are good to go. Also, if by any change you don't know what the name of your profile is (or maybe you set it as default) you can also check the file <strong>$(HOME)/.aws/credentials</strong> and then export the correct profile.
```{bash}
$ export AWS_PROFILE=name_of_your_profile
```
Good, now that all setup is in process, it's time we start pushing events to the AWS IoT and check what we can do with the streamed data. In the following example, I choose to specifically use the sample file "radar-counts". If you wish to use your own dataset, just copy it to the <strong>data/</strong> directory and change the SAMPLE_FILE when calling the following bootstrap command.
```{bash}
$ make bootstrap SAMPLE_FILE=radar-counts
```
<h2>AWS IoT Management tour</h2>
Now that our device is sending signals to AWS IoT, let's head to the <strong>IoT Core console</strong> and checkout what can we do with our streaming info to our account's IoT endpoint.
<h3>Publish and Subscribe (Test)</h3>
Under <strong>Manage Things</strong> you should see Thing_0 which was automatically created by your container, if all credentials and policies were created correctly. The neat feature where you can play with is under <strong>Test</strong> where you can Subscribe (or publish) to any IoT topic to check out the information flowing through there.
In our specific scenario, you should be able to subscribe to <strong>elf</strong>, which is the topic our docker container is publishing to.
<h3>Rules (Act)</h3>
The other important menu with which you can play around is called <strong>Act</strong>. This is where you can define rules for the intended processing of the streamed data.
As commented earlier, the rules are relatively easy to start using as they can be defined using the familiar SQL-like<em> </em>syntax, as you can see below.
<img class="alignnone size-full wp-image-4114" src="{{ site.baseurl }}/assets/2018/09/screen-shot-2018-09-09-at-19-37-56.png" alt="Screen Shot 2018-09-09 at 19.37.56.png" width="1910" height="928" />
The other very cool feature of Rules is that this part of the service is seamlessly integrated with other AWS services that you are likely to have already worked with, so you'll feel right at home from here on end.
<img class="alignnone size-full wp-image-4113" src="{{ site.baseurl }}/assets/2018/09/screen-shot-2018-09-09-at-19-36-23.png" alt="Screen Shot 2018-09-09 at 19.36.23.png" width="1954" height="1672" />
<h2>Clean up</h2>
Last, but very important step, is to clean up the infra: especially the services that were created in AWS during our tests. After all, only the IoT device itself was simulated locally, the rest of resources were all actually launched in AWS.
Confirm you are in the root directory of this repo and run the following command. It will both destroy both resources launched by your docker daemon as well as all the ones created remotely on AWS.
```{bash}
$ make clean
```
<h2>Conclusion</h2>
From here, I believe you should have the basis to drive your curiosity through this service and start to feel familiar with it. If you would like to know more about AWS IoT and get into a specific example using shadows, please let me know and I will do my best.
