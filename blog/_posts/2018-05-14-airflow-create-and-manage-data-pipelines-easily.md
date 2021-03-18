---
layout: post
title: 'Airflow: create and manage Data Pipelines easily'
date: 2018-05-14 13:47:25.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Big Data
- Continuous Delivery
- docker
- python
tags:
- Airflow
- Data Pipelines
- Pipelines
meta:
  _wp_old_date: '2018-05-11'
  _rest_api_published: '1'
  timeline_notification: '1526305649'
  _rest_api_client_id: "-1"
  _publicize_job_id: '17836127513'
  _oembed_5842a40c285aac2c092231a6ee482d6e: "{{unknown}}"
author:
  login: jrcferrao
  email: joao@mklabs.io
  display_name: João Ferrão
  first_name: ''
  last_name: ''
permalink: "/2018/05/14/airflow-create-and-manage-data-pipelines-easily/"
---
This bootstrap guide was originally published at <a href="http://gosmarten.com/airflow.html">GoSmarten</a> but as the use cases continue to increase, it's a good idea to share it here as well.
<div class="title-bordered border__dashed">
<h2>What is Airflow</h2>
</div>
The need to perform operations or tasks, either simple and isolated or complex and sequential, is present in all things data nowadays. If you or your team work with lots of data on a daily basis there is a good chance you're struggled with the need to implement some sort of pipeline to structure these routines. To make this process more efficient, Airbnb developed an internal project conveniently called Airflow which was later fostered under the Apache Incubator program. In 2015 Airbnb open-sourced the code to the community and, albeit its trustworthy origin played a role in its popularity, there are many other reasons why it became widely adopted (in the engineering community). It allows for tasks to be set up purely in Python (or as Bash commands/scripts).
<div class="title-bordered border__dashed">
<h2>What you'll find in this tutorial</h2>
</div>
Not only we will walk you through setting up Airflow locally, but you'll do so using Docker, which will optimize the conditions to learn locally while minimizing transition efforts into production. Docker is a subject for itself and we could dedicate a few dozen posts to, however, it is also simple enough and has all the magic needed to show off some of its advantages combining it with Airflow. <!--more-->Even if you are not familiar, the only requirement you have for this tutorial is to make sure you have Docker Engine installed on your Linux/macOS/Windows environment <a href="https://docs.docker.com/install/">here</a>. If you are on Windows or macOS, an additional tool called <strong>docker-compose</strong>. If you are instead on a Linux machine, you can get the installation instructions <a href="https://docs.docker.com/compose/install/">here</a>.
<div class="title-bordered border__dashed">
<h2>Using Docker</h2>
</div>
Docker will basically replicate whichever environment with specific settings and package requirements (an image): an AWS EC2 instance, an Ubuntu based machine running a load balancer or if you simply want to run an isolated environment running TensorFlow or PySpark. When you are sure that a Docker image is set up the way you want it, you simply need to instantiate it with a terminal command. You'll be able to launch and test all possible infrastructures, apps in your local machine while simulating near real production environments. It also means you can rest assured that all components are working before spending resources in the cloud or your own data center or cloud infrastructure.
<div class="title-bordered border__dashed">
<h2>Step by step</h2>
</div>
Make a git clone of our repo <a href="https://github.com/gosmarten/airflow.git">here</a> and check its structure. You will notice that directories seem deeper than needed for the project. We are merely separating the infrastructure logic from the code logic - in this case where your tasks will be written. In the future, if you need to add your own <i>containerised</i> (and customised) micro services, you would place it side by side in the Docker directory.
```{bash}
&gt;&gt; git clone https://github.com/gosmarten/airflow.git
```
Assuming you already installed Docker engine, we’ll now begin building our custom image of Airflow. Although it might be somehow limiting, think of a Dockerfile as the series of commands you would do if you wanted to start with a clean slate linux installation (whichever you’d like) and the series of commands you would run in the terminal to run the desired service. In our case this image will have Airflow setup and all packages needed for it on top of a slim version of Linux image that already has Python installed. Also, luckily for us, there is a popular customized Dockerfile by Puckel which is ready for all possible Airflow environments which we'll use.
<div class="title-bordered border__dashed">
<h2>Building our custom image for Airflow</h2>
</div>
The Dockerfile is located in <code class="code-block">docker/airflow/Dockerfile has all definitions for our Airflow image setup. Pay attention to the command bellow where Airflow is installed with accompanying packages (inside the brackets), with the version which we define at the beginning of the document.
[code language="bash" firstline=51 highlight=55 ]
&amp;&amp; pip install pytz \
&amp;&amp; pip install pyOpenSSL \
&amp;&amp; pip install ndg-httpsclient \
&amp;&amp; pip install pyasn1 \
&amp;&amp; pip install apache-airflow[crypto,celery,postgres,hive,jdbc]==$AIRFLOW_VERSION \
&amp;&amp; pip install celery[redis]==4.0.2 \
&amp;&amp; apt-get purge --auto-remove -yqq $buildDeps \
&amp;&amp; apt-get clean \
```
Now, for Airflow configuration itself check out the config.cfg file under docker/airflow. This file has all the configuration you would need Airflow to have when running: information like URI with PostgreSQL credentials and host (by default it will run on SQLite), Airflow home directory (otherwise it will just store everything under /airflow at the root of the server/pc it is running from) and where DAGs should be stored. Another important choice you can make in settings is what kind of Executor you want to run. By default, Airflow runs in Sequential mode, which means it can only execute one task at a time. While this is ideal for development you should choose another option when deploying into staging or production. This leads us to 2 other options you can select from: Local or Celery. Local will be able to process several tasks on the same machine (without extra containers running other services). Celery will allow you to set up a Scheduler/Worker with separation of processing and scheduling between multiple nodes (for example, several different containers) in isolated environments.
We won't go into the details of this last option, as Celery would be a whole different topic, but rest assured that with the help of an image we are using from Puckel, you won't need to change anything in the image itself. In case you want to evolve your setup to a Celery execution in the future - you’d only need to adjust the next settings.
If you are an intermediate Docker user, you might prefer passing information like login credentials and host specifications upon instantiating the container and that is perfectly fine. Airflow accepts Environmental variables with configuration for most options inside the config.cfg file and these variables take precedence over the file. If you want this, just make sure the format is as follows: <code class="code-block">$AIRFLOW__{SECTION}__{KEY} (notice the double underscore). So if you wanted to define/override Airflow home setting, you would use <code class="code-block">$AIRFLOW__CORE__AIRFLOW_HOME.
We are happy with the Dockerfile now, so it's time to build it. This means Docker will run all these commands and store the desired "state" of those commands in an image in our computer cache. To do so, run the following command in terminal inside the same directory as the Dockerfile.
```{python}
&gt;&gt; docker build -t airflow_tutorial:latest .
```
<div class="title-bordered border__dashed">
<h2>Orchestrating containers</h2>
</div>
Next step is to take a look at docker-compose.yml file in the same directory as the Dockerfile. The docker-compose basically orchestrates which services (or containers) are launched together and their respective settings. We have extensively commented this file in order for you to have a better sense of what is happening line by line and tweak if necessary.
In summary, we are launching 2 different containers, Postgres and webserver, both in the same virtual network so they can communicate by their names (without the need for IP addresses between them). Airflow uses this database to store metadata on the DAGs, tasks, users and their statuses. Airflow is also ready to store and encrypt credentials for services that you need for your tasks: S3 buckets, other Postgres instances, MySQL, etc.
<div class="title-bordered border__dashed">
<h2>Enter Airflow</h2>
</div>
Finally we get to the functionality of Airflow itself. In nutshell, a DAGs (or directed acyclic graph) is a set of tasks. For each workflow we define, we can define as many tasks as we want as well as priority, importance and all sorts of settings. Then, the DAGs are pushed.
For each task inside a DAG, Airflow relies mainly on Operators. Operators execute tasks. There are some preloaded ones, like BashOperator or PythonOperator which executes Bash and Python code respectively.
<div class="title-bordered border__dashed">
<h2>Designing the DAG</h2>
</div>
In our tutorial, we'll use default Airflow tutorial.py file which will help us understand the basic concepts of DAGs. It defines a DAG comprised of 2 tasks that run, only if a third one (actually the first one) is successfully executed. All tasks use BashOperator. One of them includes some Jinja2 templating - pretty popular for who uses frameworks like Django or Flask. If you would like to know more about Jinja2 check <a href="http://jinja.pocoo.org/docs/2.10/">their website</a>.
The DAG itself is an instantiation of a class, but for it to work it needs:
<ul>
<li>The DAG object, and usually some predefined default arguments. It's a good practice to defined some default arguments so we don't repeat too much code and then pass this dictionary as a key, value pair in the DAG itself.</li>
<li>We need tasks which are basically defined by the type of Operator we need to use. In this case we'll use Bash to print out the date and show off some Jinja2 templating features of Airflow. Notice this templated command is then injected into the task itself as a parameter for the Bash command. Alternatively, if it was a PythonOperator that was calling a function, we could call arguments to the function through the op_kwargs or op_args argument of the Operator.</li>
<li>Finally, we need to setup the execution order for tasks. This is what the last 2 lines of code are doing: we are saying that basically <code class="code-block">t2 and <code class="code-block">t3 can run in parallel but only if <code class="code-block">t1 runs successfully. This could make sense in an ETL scenario where you would extract data from some source in t1 and then insert it in a database in <code class="code-block">t2 while backing it in a form of CSV in <code class="code-block">t3 at the same time. They would only be dependent on having <code class="code-block">t1 finished.</li>
</ul>
<div class="title-bordered border__dashed">
<h2>Launching the services</h2>
</div>
Having all this done, we can now launch our terminal, navigate to the docker-compose folder and run the following command:
```
&gt;&gt; docker-compose up
```
This will spin up both services as stated in the docker-compose.yml file. Now you can access <a href="http://localhost:8080/">http://localhost:8080</a> in your browser and you'll see your Airflow application listing the DAG we just described before. You should see something very similar to this:
<figure class="entry-thumbnail"><img class=" aligncenter" src="{{ site.baseurl }}/assets/2018/05/airflow_main.png" alt="" /></figure>
If you noticed the docker-compose comments, we are using docker bind mount of the dags folder which is located in repo/src/dags. Whatever you change in a DAG or create a new one it will be immediately available to Airflow.
To see your DAG in action you have 2 options.
<ul>
<li>By default, the DAGs are turned off, so on the home page you can turn it on by clicking the button on the left, right before DAG’s title.</li>
<li>Alternatively you can press the “play” button, located on the right.</li>
</ul>
After making the DAG run and seeing the nice GUI showing status of the task, you can check the logs by clicking the task itself and then clicking again the task (on the graph below). The DAG we used is only executing Bash commands, printing dates and templated code in Jinja2 into the terminal. It's also possible to see DAG in action in the terminal where you launched the containers.
<figure class="entry-thumbnail"><img class=" aligncenter" src="{{ site.baseurl }}/assets/2018/05/airflow_graph.png" alt="" /></figure>
<div class="title-bordered border__dashed">
<h2>Wrapping up</h2>
</div>
Soon we'll write about how to move this image to a cloud service and develop a more advanced DAG. If you are familiar with Python, you now have the building blocks to develop whatever functions and design tasks that use all sorts of APIs already available in the Python ecosystem.
The big advantage of using Puckel's image in the next tutorial is that we will be able to use that very same image for different containers that will assume different roles (worker vs. scheduler), without having to modify the image itself - we'll only tweak the docker-compose.yml file to our liking.
