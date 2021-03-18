---
layout: post
title: Check out video Understanding Using Temporal Cycle-Consistency Learning
date: 2019-08-13 19:15:30.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Deep learning
tags: []
meta:
  timeline_notification: '1565723735'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '34042886645'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2019/08/13/check-out-video-understanding-using-temporal-cycle-consistency-learning/"
---
It has been a long time since I last blogged. João and I have been busy getting things ready to launch a new project to streamline multi-cloud management that we have been working in the last couple of months. We will talk soon in more detail about it, so stay put.
Meanwhile just wanted to share this very interesting blog post from Google AI blog - <a href="https://ai.googleblog.com/2019/08/video-understanding-using-temporal.html" target="_blank" rel="noopener">video understanding using temporal cycle-consistency learning</a> - where they propose a <a href="https://hackernoon.com/self-supervised-learning-gets-us-closer-to-autonomous-learning-be77e6c86b5a" target="_blank" rel="noopener">self-supervised learning</a> method to classify different actions, postures, etc. in videos.
{:.lead}
This approach intends to overcome the issue of expensive time consuming manual video per-frames labeling process. It does so by using "[...] <em>correspondences between examples of similar sequential processes to learn representations particularly well-suited for fine-grained temporal understanding of videos</em>". The process involves training a network to learn a frame encoder, such as a ResNet; then choosing a reference frame from a given video, and then comparing it to the embeddings of another video using Nearest Neighbors (NN). The last step (which provides the cycle part), involves applying NN from video 2 to video 1, to assure it references the same frame.
<img class=" size-full wp-image-4268 aligncenter" src="{{ site.baseurl }}/assets/2019/08/self_supervised_learning.gif" alt="self_supervised_learning" width="561" height="287" />
I strongly encourage you to <a href="https://ai.googleblog.com/2019/08/video-understanding-using-temporal.html" target="_blank" rel="noopener">read the blog post</a>, as it does a great job at explaining the approach.
Last but not least, the code base is available <a href="https://github.com/google-research/google-research/tree/master/tcc" target="_blank" rel="noopener">here</a>.
