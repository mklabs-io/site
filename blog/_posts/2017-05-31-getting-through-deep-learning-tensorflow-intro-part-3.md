---
layout: post
title: Getting through Deep Learning – Tensorflow intro (part 3)
date: 2017-05-31 20:34:18.000000000 +02:00
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
  _publicize_job_id: '5643020764'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2017/05/31/getting-through-deep-learning-tensorflow-intro-part-3/"
---
This post is part of a tutorial series:
<ol>
<li><a href="https://datacenternotes.wordpress.com/2016/09/25/getting-through-deep-learning-part-1/" target="_blank" rel="noopener noreferrer">Getting through Deep Learning - CNNs (part 1)</a></li>
<li><a href="https://datacenternotes.wordpress.com/2017/05/09/getting-through-deep-learning-tensorflow-intro-part-2/" target="_blank" rel="noopener noreferrer">Getting through Deep Learning - TensorFlow intro (part 2)</a></li>
<li><a href="https://datacenternotes.wordpress.com/2017/05/31/getting-through-deep-learning-tensorflow-intro-part-3/" target="_blank" rel="noopener noreferrer">Getting through Deep Learning - TensorFlow intro (part 3)</a></li>
</ol>
Alright, lets move on to more interesting stuff: linear regression. Since the main focus in TensorFlow, and given the abundancy of online resources on the subject, I'll just assume you are familiar with Linear Regressions.
As previously mentioned, a linear regression has the following formula:
<img class="  wp-image-3926 aligncenter" src="{{ site.baseurl }}/assets/2017/05/linear_regression.png" alt="linear_regression" width="401" height="131" />
Where Y is the dependent variable, X is the independent variable, and <b>b0</b> and <b>b1</b> being the parameters we want to adjust.
Let us generate random data, and feed that random data into a linear function. Then, as opposed to using the closed-form solution, we use an iterative algorithm to progressively become closer to a minimal cost, in this case using gradient descent to fit a linear regression.<!--more-->
https://gist.github.com/diogoaurelio/6c98b6da8f3e2b0a6fb632978f404f3a
We start by initializing the weights - b0 and b1 - with random variables, which naturally results in a poor fit. However, as one can see through the print statements as the model trains, b0 approaches the target value of 3, and b1 of 5, with the last printed step: [3.0229037, 4.9730182]
The next figure illustrates the progressive fitting of lines of the model, as weights change:
<img class="alignnone size-full wp-image-3939" src="{{ site.baseurl }}/assets/2017/05/lr_fitting1.png" alt="lr_fitting" width="590" height="368" />
Sources:
<ul>
<li>Big Data University course "<a href="https://cognitiveclass.ai/courses/deep-learning-tensorflow/" target="_blank" rel="noopener noreferrer">Deep Learning with TensorFlow</a>"</li>
<li><a href="http://www.edupristine.com/blog/demystifying-linear-regression-analysis-for-frm-level-1-exam" target="_blank" rel="noopener noreferrer">Demystifying Linear Regression Analysis</a></li>
</ul>
