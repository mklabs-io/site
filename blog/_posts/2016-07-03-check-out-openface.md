---
layout: post
title: Check out OpenFace
date: 2016-07-03 18:24:05.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Deep learning
- Image/Video Processing
tags: []
meta:
  _oembed_760cedddb6db57949e3ab7ad5cf59166: "{{unknown}}"
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '24437513420'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2016/07/03/check-out-openface/"
---
If you're interested in using machine learning (ML) on image and video datasets, then you might be interested in heaving a look on a relatively new project called <a href="https://cmusatyalab.github.io/openface/" target="_blank">OpenFace</a> (first released in October 2015), with&nbsp; Brandon Amos, Ludwiczuk Bartosz and Mahadev Satyanarayanan as authors.
<strong>TL;DR: For the impatient</strong>
<ul>
<li>Pitch me:&nbsp;<a href="https://github.com/cmusatyalab/openface">Open source project</a>&nbsp;(aka free for you to use) developed&nbsp;inside Carnegie Mellon University &nbsp;for face recognition with deep neural networks, with a Python API</li>
<li>What do I get from it: improved accuracy <em><span style="text-decoration:underline;"><strong>and</strong></span></em> reduced&nbsp;training time</li>
<li>Need to see to believe (and so one should)? You can start playing with it with <a href="https://hub.docker.com/r/bamos/openface/">Docker</a>, and check the provided&nbsp;<a href="https://github.com/cmusatyalab/openface/tree/master/demos/web">demos</a></li>
</ul>
<strong>What about it</strong>
Even though face recognition research has already started since the 1970s, it is still far from stagnant. The usual strategy&nbsp;for solving the problem has been divided into three main steps; given an image with a set of faces, first run face detection algorithm to isolate the faces from the rest, then preprocess this cropped part to reduce the high dimensionality, and finally classification. However, what makes this whole process so challenging is that many factors can create noise around this process, such as images can be taken from different angles, different lighting conditions, the face itself also suffers changes throughout time (for example due to&nbsp;age or style), etc.
Now one important fact to point out is that state of the art top performing algorithms are using convolutional neural networks. OpenFace is inspired by&nbsp;<a href="https://www.cs.toronto.edu/~ranzato/publications/taigman_cvpr14.pdf" target="_blank">Facebook’s DeepFace</a> and (mainly)&nbsp;<a href="https://arxiv.org/abs/1503.03832" target="_blank">Google’s FaceNet</a> systems.&nbsp;The performance smack down that the&nbsp;authors present using the "Labeled Faces in the wild" dataset (<a href="http://vis-www.cs.umass.edu/lfw/">LFW</a>) for eveluation, and achieved some <a href="http://cmusatyalab.github.io/openface/models-and-accuracies/" target="_blank">interesting results</a>.
Another interesting point is that, as the authors state, the implementation is tuned for using the model in mobile devices, so the &nbsp;"[...]&nbsp;<em>key design consideration is a system that gives high&nbsp;</em><em>accuracy with low training and prediction times</em>".
Note:&nbsp;In case you are&nbsp;wondering what's the difference to <a href="http://openbiometrics.org/" target="_blank">OpenBiometrics</a>&nbsp;(OpenBR). As stated by the authors of <a href="https://news.ycombinator.com/item?id=10389289" target="_blank">OpenFace in HackerNews</a>, the main difference lies on the approach taken - deep convolutional networks - and could potentially be integrated into OpenBR's pipeline.
<strong>Internal Guts</strong>
As you might imagine (as any image/video processing package), dependencies are complex and time consuming, so prepare yourself for some dependencies troubleshooting in case your machine is still new to this world.
The project's API is written in Python 2 - entry point <a href="https://github.com/cmusatyalab/openface/tree/master/openface" target="_blank">here</a> - given its dependencies on <a href="http://opencv.org/" target="_blank">OpenCV</a> and <a href="http://dlib.net/" target="_blank">DLib</a>. OpenCV provides the computer vision base, DLib enhances OpenCV face detection ability, numpy for matrix algebra operations and scikit-learn for classification operations.
For training the convolutional network&nbsp;openface uses&nbsp;<a href="http://torch.ch/">Torch</a>, Lua and <a href="http://luajit.org/" target="_blank">Luajit</a>&nbsp;which is written in Lua programming language. In this case, Torch allows the neural networks to be executed either in CPU or CUDA enabled GPUs.
The following illustration was extracted from the recent&nbsp;technical report&nbsp;"<a href="http://reports-archive.adm.cs.cmu.edu/anon/anon/2016/CMU-CS-16-118.pdf" target="_blank">OpenFace: A general-purpose face recognition</a>
<a href="http://reports-archive.adm.cs.cmu.edu/anon/anon/2016/CMU-CS-16-118.pdf" target="_blank">library with mobile applications</a>", by&nbsp;the authors, and provides interesting insight:
<img class="alignnone size-full wp-image-1288" src="{{ site.baseurl }}/assets/2016/07/openface_diagram.jpg" alt="openface_diagram" width="846" height="564" />
So important to note is that you do have the option to use already pretrained models (which use the <a href="http://www.cbsr.ia.ac.cn/english/CASIA-WebFace-Database.html" target="_blank">CASIA-WebFace</a> and <a href="http://vintage.winklerbros.net/facescrub.html" target="_blank">FaceScrub</a> databases) to help with face detection, which you can find in the <a href="https://github.com/cmusatyalab/openface/tree/master/models" target="_blank">models</a> directory. The provided bash script downloads them.
<strong>Where&nbsp;to get started</strong>
To setup either locally or with Docker you can check the <a href="https://github.com/cmusatyalab/openface/blob/master/docs/setup.md" target="_blank">provided documentation</a>.
Finally, you might also be interested in having a look at other projects using deep neural networks&nbsp;for face recognition:&nbsp; <a href="http://www.robots.ox.ac.uk/~vgg/software/vgg_face/" target="_blank">Visual Geometry Group (VGG) Face Descriptor</a>, and <a href="http://arxiv.org/abs/1511.02683" target="_blank">Lightened&nbsp;Convolutional Neural Networks </a>(CNNs)
&nbsp;
