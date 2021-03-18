---
layout: post
title: Getting through Deep Learning - CNNs (part 1)
date: 2016-09-25 17:51:00.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Deep learning
- Image/Video Processing
- kaggle
- machine learning
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '27199256998'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2016/09/25/getting-through-deep-learning-part-1/"
---
The number of available open source libraries making Deep learning easier to use is spreading fast as hype continuous to build. However, without understanding the background principles, it just feels like poking around a black box.
In this post (or several, most likely) will try to give an introduction to Convolution Neural Networks (CNNs). Note that, for the sake of brevity, I assume that you already know the basics about Neural Networks. If not, I would suggest you go through the <a href="https://www.analyticsvidhya.com/blog/2016/08/evolution-core-concepts-deep-learning-neural-networks/" target="_blank" rel="noopener noreferrer">following introduction</a>.
This post is part of a tutorial series:
<ol>
<li><a href="https://datacenternotes.wordpress.com/2016/09/25/getting-through-deep-learning-part-1/" target="_blank" rel="noopener noreferrer">Getting through Deep Learning - CNNs (part 1)</a></li>
<li><a href="https://datacenternotes.wordpress.com/2017/05/09/getting-through-deep-learning-tensorflow-intro-part-2/" target="_blank" rel="noopener noreferrer">Getting through Deep Learning - TensorFlow intro (part 2)</a></li>
<li><a href="https://datacenternotes.wordpress.com/2017/05/31/getting-through-deep-learning-tensorflow-intro-part-3/" target="_blank" rel="noopener noreferrer">Getting through Deep Learning - TensorFlow intro (part 3)</a></li>
</ol>
Disclaimer: this post uses images and formulas from distinct sources. I would suggest to have a look over the complete list of sources at the end of the post, as usual.
<strong>Inspiration</strong>
In 1958 and 1959 David H. Hubel and Torsten Wiesel performed a series of experiments, whereby they concluded that many neurons in the visual cortex focus on a limited region in the vision field.
This insight provided the notion of a <em>local receptive field</em> - a narrow sub-region of what is available in the whole visual field which serves as input - thus giving rise for a different architecture than the previously fully connected neural network architecture.
<strong>Basics - Convolution Layer</strong>
The first thing to realize is that Convolution networks are simply the application of "mini-neural networks" to segments of input space. In the case of images, that results in that neurons in the first convolutional layer are not connected to every single pixel in their Receiptive Field (RF).  The following image (<a href="http://ufldl.stanford.edu/tutorial/supervised/ConvolutionalNeuralNetwork/" target="_blank" rel="noopener noreferrer">source</a>) shows an illustration of how a a a convolution layer is built using an image from the famous MNIST dataset - whereby the goal consists in identifyying the digits from handwritten numbers pictures.
<img class="  wp-image-3824 aligncenter" src="{{ site.baseurl }}/assets/2016/09/cnn_layer.png" alt="Cnn_layer" width="495" height="377" />
&nbsp;
<!--more-->OK, let's break this down. In the MNIST dataset each, each image is 28 by 28 pixels - respectively <strong>height</strong> and <strong>width.</strong> For a fully connected neural network, the input space for the first layer of 28 x 28 = 728px if we were only to include height and width.
However, in a so-called convolution, you would instead apply a <strong>mini-neural network</strong> to just a single portion/segment of the image - let's say a 3x3 px (width and height) rectangle. This 3x3 receptive field is also often refered to as a <strong>filter</strong> or <strong>kernel</strong> in Deep Learning (DL) lingo.
Next you would slide that kernel over the image, let us say 1px to the left in each step until we reach the far right end, and then 1px down until we reach the lower bound. The following animated illustration - credits go to Vincent Dumoulin and Francesco Visin with awesome CNN architectual overview available <a href="http://arxiv.org/pdf/1603.07285v1.pdf" target="_blank" rel="noopener noreferrer">here </a> - shows the building of a convolution layer using this 3x3 Receptive Field/Filter/Kernel step-by-step building what is called a <strong>Feature Map -</strong> a layer full of neurons using the same filter.
<img class=" size-full wp-image-2205 aligncenter" src="{{ site.baseurl }}/assets/2016/09/base_conv_no_padding_no_strides.gif" alt="base_conv_no_padding_no_strides" width="244" height="259" />
&nbsp;
Thus a Feature Map can also be thought of a multitude of these mini-Neural Networks - whereby each filter - the dark blue rectangle - has its own weights, bias term, and activation function, and produces one output (darker green square).
The following illustration shows a detailed view of the progressive application of these mini neural network across filters of the initial input space -again credits go to Vincent Dumoulin and Francesco Visin - and producing a Feature map.
<img class="alignnone size-full wp-image-2369" src="{{ site.baseurl }}/assets/2016/09/deep_learning_basics_trimmed.png" alt="deep_learning_basics_trimmed.png" width="889" height="501" />
Note that the illustrations used an iterative sliding of a kernel of 3x3 in a step of 1 px. However this is not a requirement, as one can use for example a step size of 2, reducing the dimention of output space. By now you should be realizing that indeed this step size - called the <strong>stride</strong> in DL lingo - is yet another hyper parameter, just as the filter size.
OK, we are almost completed with the basic concepts related to a CNNs: we take a local receptive field  - which we call <strong>filter/kernel - </strong>slide it through an image in given step size - which we call <strong>stride</strong> - and produce a set of mini neural networks - which we call a <strong>feature map</strong>.
The missing detail that builds uppon the previous knowledge is the fact that we can simultaneously use different filters to the same input space, thus producing several feature maps as a result. The only restriction is that feature maps in a given layer have the same dimention. The following illustration from the <a href="http://shop.oreilly.com/product/0636920052289.do" target="_blank" rel="noopener noreferrer">excellent book "Hands-On Machine Learning with Scikit-Learn and TensorFlow"</a>  gives more insight to the intuition of stacking multiple feature maps.
<img class="  wp-image-3826 aligncenter" src="{{ site.baseurl }}/assets/2016/09/feature_maps_cnn.jpg" alt="feature_maps_cnn" width="557" height="487" />
Until so far we have represented a convolution layer in a thin 2-D output space (1 single feature map). However, if one produces several feature maps in one layer, the result is a 3-D volume.  And this is when the notion of convolution comes into place: the output of this convolution will have a new height, width and depth.
<img class=" size-full wp-image-2528 aligncenter" src="{{ site.baseurl }}/assets/2016/09/conv_layer_depth.png" alt="conv_layer_depth" width="229" height="154" />
&nbsp;
It is thus frequent to see convolutional networks illustrated as "tall and skinny" boxes (aka with high values of height and low of depth), and progressively getting "shorter and fatter" (smaller height and bigger depth).
&nbsp;
<img class="  wp-image-2518 aligncenter" src="{{ site.baseurl }}/assets/2016/09/conv_layers.png" alt="Conv_layers.png" width="499" height="273" />
<strong>Basics - Pooling Layer</strong>
Before we go into more detail about different architectures a Convolution Layer may have (in the next post of this series), it is important to cover some ground aspects. To start, one is that you are not restricted to use <strong>Convolution Layer </strong>when creating a new hidden layer. In fact there are two more main types of layers: <strong>Pooling Layer</strong> (which we will cover in just a bit) and <strong>Fully-Connected layer</strong> (exactly as a regular Neural Network).
Also note that similarly to regular Neural Networks, where you can have as many hidden layers as you want, the same goes the CNN. That is, you can build convolutions that serve as input space for a next convolution layer or a pooling layer for example,  and so on.
Finally, and again similarly to Neural Networks, the last fully-connected layer will always contain as many neurons as the number of classes to be predicted.
<strong>Typical Activation functions</strong>
Again to be perfectly clear, what we do in each step of the sliding filter in a convolution is to a apply the dot product of a given set of weights (which we are trying to tune with training), plus a given bias. This is effectively a linear function represented in the following form:
<img class="  wp-image-2414 aligncenter" src="{{ site.baseurl }}/assets/2016/09/linear_regression_trimmed.png" alt="linear_regression_trimmed" width="263" height="57" />
where the weights is a vector represented by W, Xi would be the pixel values in our example inside a given filter, and b the bias. So what usually happens (except from pooling)  is that we pass the output of this function to a neuron, which will then apply given activation function. The typical activation functions that are usually implemented are:
<span style="text-decoration:underline;">Softmax</span> - is a generalized form of the logistic function/sigmoid function, which turns the outputs into probabilities (thus comprising in interval between 0 and 1).
<img class=" size-full wp-image-2433 aligncenter" src="{{ site.baseurl }}/assets/2016/09/softmax_trimmed.png" alt="softmax_trimmed" width="163" height="67" />
<span style="text-decoration:underline;">ReLU</span> - Rectified Linear Unit functions have a smoothing effect on the output, making results always bigger than zero.
<img class="  wp-image-2460 aligncenter" src="{{ site.baseurl }}/assets/2016/09/relu.png" alt="relu" width="238" height="77" />
<span style="text-decoration:underline;">Tanh</span> - hyperbolic tangent function, which enables activation functions to range from -1 to +1.
<img class="  wp-image-2464 aligncenter" src="{{ site.baseurl }}/assets/2016/09/tanh.png" alt="tanh" width="182" height="65" />
<strong>Typical Cost Functions</strong>
As you know, cost functions are what makes the whole training of models possible. Here are three of the main, where cross entropy is probably the most frequently used.
<span style="text-decoration:underline;">Mean Squared Error</span> -  used to train linear regression models
<img class="  wp-image-2444 aligncenter" src="{{ site.baseurl }}/assets/2016/09/mean_squared_error.png" alt="mean_squared_error" width="212" height="68" />
<span style="text-decoration:underline;">Hinge Loss</span> - used to train Support Vector Machines (SVM) models
<img class=" size-full wp-image-2441 aligncenter" src="{{ site.baseurl }}/assets/2016/09/hinge_loss.png" alt="hinge_loss" width="348" height="74" />
&nbsp;
<span style="text-decoration:underline;">Cross entropy</span> - used to train logistic regression models
<img class=" size-full wp-image-2452 aligncenter" src="{{ site.baseurl }}/assets/2016/09/cross_entropy_trimmed.png" alt="cross_entropy_trimmed" width="243" height="93" />
<strong>Regularization</strong>
Beyond activation functions applied to convolutions, there are also some other useful tricks applied to build a Deep Neural Network (DNN), which address the well known problem of over-fitting.
<span style="text-decoration:underline;">Pooling Layer</span> - bottom line, pooling layers are used to reduce dimension. They sample from input space also using a filter/kernel with a given output dimension, and simply applying a reduce function. Usually a <strong>max</strong>()  - usually called <strong>max pooling</strong> - or <strong>mean</strong>()  - <strong>average pooling</strong> - functions.
For example, if one of kernels was a 2x2 matrix with the values [ [1,2], [3,4]], then max pooling would yield 4 as an output, where average pooling would yield 2.5 .
<span style="text-decoration:underline;">Dropouts</span> -dropouts goal is exactly the same as regularization (it is after all a regularization technique); that is, it is intended to reduce over-fitting on outer sample. Initially it was used to simply turn off passing a portion of output of neurons at every iteration during training. That is, instead of passing all weights dot product computed against the input layer, we randomly (with a given specifiable probability) consciously decide to not add a given set of weights to the output layer.
In case you are wondering if this is similar trick to bagging that Random Forests does to Decision Trees, the answer would be not quite. The operation of averaging through lots of Decision Trees (wich have high propensity to over-fit data) using  sampling  with replacement is computationally doable (at today' standards). However the same does not hold true to train distinct DNNs. So the dropouts technique is a practical method to average internally the outputs among layers of a network.
<strong>Final Notes</strong>
In part 2 I plan to get into more detail about convolution architectures, as well as provide a coding example in order to bring all these concepts home.
However, I didn't want to terminate this post without any code snippet. So even though not going to go through it, just as a demonstration that with the concepts covered before you can already have some intuition around it, here is a code snippet where a training session with Google's deep learning library TensorFlow is defined with two convolution layers:
https://gist.github.com/diogoaurelio/ad998857f213ba17e99ad3380f5526ab
As usual, here are the sources used for this post:
<ul>
<li><a href="https://github.com/vdumoulin/conv_arithmetic" target="_blank" rel="noopener noreferrer">Really neat visual explanation of Convolution architectures by Vincent Dumoulin and Francesco Visin</a></li>
<li><a href="http://cs231n.github.io/convolutional-networks/" target="_blank" rel="noopener noreferrer">Stanford Deep Learning for Computer Vision (Convolutions explained)</a></li>
<li><a href="https://www.analyticsvidhya.com/blog/2016/04/deep-learning-computer-vision-introduction-convolution-neural-networks/" target="_blank" rel="noopener noreferrer">Analytics Vidhya blog intro to deep learning</a></li>
<li><a href="https://en.wikipedia.org/wiki/Convolutional_neural_network" target="_blank" rel="noopener noreferrer">Wikipedia Convolutional Neural Network</a></li>
<li><a href="http://ufldl.stanford.edu/tutorial/supervised/ConvolutionalNeuralNetwork/" target="_blank" rel="noopener noreferrer">Convolutional Neural Networks</a></li>
</ul>
