---
layout: post
title: Getting through Deep Learning – Tensorflow intro (part 2)
date: 2017-05-09 07:21:40.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Deep learning
- machine learning
- python
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '4852198065'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2017/05/09/getting-through-deep-learning-tensorflow-intro-part-2/"
---
Yes, I kind of jumped the guns on <a href="https://datacenternotes.wordpress.com/2016/09/25/getting-through-deep-learning-part-1/" target="_blank" rel="noopener noreferrer">my initial post on Deep Learning straight into CNNs</a>. For me this learning path works the best, as I dive straight into the fun part, and eventually stumble upon the fact that maybe I'm not that good of a swimmer, and it might be good to practice a bit before going out in deep waters. This post attempts to be exactly that: going back to the basics.
This post is part of a tutorial series:
<ol>
<li><a href="https://datacenternotes.wordpress.com/2016/09/25/getting-through-deep-learning-part-1/" target="_blank" rel="noopener noreferrer">Getting through Deep Learning - CNNs (part 1)</a></li>
<li><a href="https://datacenternotes.wordpress.com/2017/05/09/getting-through-deep-learning-tensorflow-intro-part-2/" target="_blank" rel="noopener noreferrer">Getting through Deep Learning - TensorFlow intro (part 2)</a></li>
</ol>
TensorFlow is a great starting point for Deep Learning/ Machine Learning, as it provides a very concise yet extremely powerful API. It is an open-source project created by Google initially with numerical computation tasks in mind, and used for Machine Learning/Deep Learning.<!--more-->
TensorFlow provides APIs for both Python and C++, but it's backend is written in C/C++, allowing it to achieve much greater performance milestones. Moreover, it supports CPU, GPU, as well as distributed computing in a cluster.
The first thing to realize is that TensorFlow uses the concept of a <strong>session</strong>. A session is nothing more than a series of operations to manipulate tensors, organized in a structure of a data flow graph. This graph building activity pretty much works like Lego building, by matching nodes and edges. Nodes represent mathematical operations, and edges multi-dimensional arrays - aka: Tensors. As the name hints, a tensor is the central data structure in TensorFlow, and is described by its shape. For example, one would characterize a 2 row by 3 columns matrix as a tensor with shape of [2,3].
Important also to note is that the graph is lazy loaded, meaning that computation will only be triggered by an explicit run order for that session graph. OK, enough talking, let us get into coding, by exemplifying how a basic graph Session is built:
https://gist.github.com/diogoaurelio/5a0fd9501607ecf5b885b8cbecc95540
In the previous example, variables "a" and "b" are the nodes in the graph, and the summation is the operation connecting both of them.
A TensorFlow program is typically split into two parts: construction phase - where the graph is built - and a second one called execution phase, when actually resources (CPU and/or GPU, RAM and disk) are allocated until the session is closed.
Typically machine learning applications strive to iteratively update model weights. So, of course, one can also specify tensors of variable type, and even combine those constants.
https://gist.github.com/diogoaurelio/f25eb666176bed1b2eb6927ce2e460c1
By defining the dtype of a node, one can gain/loose precision, and at the same time impact on memory utilization and computation times.
Note lines 35 to 37 now:
r1 = op1.eval()
r2 = op2.eval()
result = f.eval()
TensorFlow automatically detects which operations depend on each other. In this case, TensorFlow will know that op1 depends on x and y evaluation, op2 on a, b and c evaluation,  and finally that f depends on both op1 and op2. Thus internally the lazy evaluation is also aligned with the computation graph. So far so good.
However, <span style="text-decoration:underline;">all nodes values are dropped between graph runs</span>, except for variable values, which are maintained by the session across graph runs. This has the import implication that op1 and op2 evaluation will not be reused upon f graph run - <span style="text-decoration:underline;">meaning the code will eveluate op1 and op2 twice</span>.
To overcome this limitation, one needs to instruct TensorFlow to run those operations in a single graph:
https://gist.github.com/diogoaurelio/aaeab834caea6759633cb1bde67e86f8
And yes, that is all for today. I want to blog more frequently, and instead of writing just once every couple of months (and in the meanwhile pilling up a lot of draft posts that never see the light of day), I decided to keep it simple. See you soon :)
Sources:
<ul>
<li><a href="https://www.tensorflow.org/get_started/get_started" target="_blank" rel="noopener noreferrer">Getting started with TensorFlow</a></li>
<li>A really recommendable book specially for Software Developers: "<a href="http://shop.oreilly.com/product/0636920052289.do" target="_blank" rel="noopener noreferrer">Hands-On Machine Learning with Scikit-Learn and TensorFlow</a>"</li>
</ul>
&nbsp;
&nbsp;
