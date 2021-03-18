---
layout: post
title: Build a Data Pipeline with AWS Athena and Airflow (part 2)
date: 2018-07-21 17:41:09.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories: []
tags: []
meta:
  _oembed_9f605de7c4f457330f22d59a0265cf55: "{{unknown}}"
  _oembed_20d6f29ebaa54c2c2c5a858e54be1995: "{{unknown}}"
  _thumbnail_id: '4054'
  timeline_notification: '1532194873'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '20249746580'
author:
  login: jrcferrao
  email: joao@mklabs.io
  display_name: João Ferrão
  first_name: ''
  last_name: ''
permalink: "/2018/07/21/build-a-data-pipeline-with-aws-athena-and-airflow-part-2/"
---
After learning the basics of Athena in <a href="http://datacenternotes.com/2018/06/07/build-a-data-pipeline-with-aws-athena-and-airflow-part-1/">Part 1</a> and understanding the <a href="http://datacenternotes.com/2018/05/14/airflow-create-and-manage-data-pipelines-easily/">fundamentals or Airflow</a>, you should  now be ready to integrate this knowledge into a continuous data pipeline.
The idea is for it to run on a daily schedule, checking if there's any new CSV file in a folder-like structure matching the day for which the task is running. For example, if the task is running for 2010-01-31, then then it will check if there is any file in s3://data/year=2010/month=01/day=31/*. If it finds a file there, it will add the "folder" as a partition to Athena so we can keep querying it.
<h2></h2>
<h2>Remind me again: why Athena?</h2>
At this point, if you are still wondering why Athena is so useful when you already have a pipeline in process to dump data somewhere (maybe a DB?) well, remember Athena is a "pay as you go" solution that will scale automatically for the desired queries you are running. The underlying costs are only associated with the S3 file hosting itself plus the execution of queries. Such queries, when combined with the Hive Metastore will provide a fast solution for querying heavy loads of data stored in several different types of files on an S3 bucket. On the other hand, provisioning a Database for dumping data will have fixed costs such as processing power, memory and storage amount which will surpass the first ones in case you are not using/needing the full blown features of having in place a proper database engine.
Before proceeding, there are three important assumptions:<!--more-->
<ol>
<li>My demo will assume you either have a "default" AWS profile with permissions to query Athena.</li>
<li>You already have some process in place - Airflow? :) - that is dumping daily data into your S3 buckets. After all, this is what Athena is used for.</li>
<li>You know basic Docker stuff or at least have it installed. This is where our Airflow code and pipelines will be living in.</li>
</ol>
<h2>Dataset</h2>
We'll be borrowing data from vizgr.com for important historical events between January and March 2010. The information is kept in an S3 bucket as below and inside each "folder" there will be a data.csv file which will have the relevant events of that particular day.
Since it isn't a lot of data, I included it, exactly with this structure in the github repo, on the very same branch as Athena.
[code gutter="false"]
└── year=2010
    ├── month=01
    │   ├── day=03
    │   ├── day=04
    │   ├── day=08
    │   ├── day=12
    │   ├── day=15
    │   └── day=25
    ├── month=02
    │   ├── day=03
    │   ├── day=12
    │   ├── day=18
    │   └── day=27
    └── month=03
        ├── day=16
        └── day=26
```
<h2>Prepare Airflow</h2>
I'll use the Airflow image that I introduced in <a href="http://datacenternotes.com/2018/05/14/airflow-create-and-manage-data-pipelines-easily/">an earlier post</a> located in <a href="https://github.com/gosmarten/airflow/tree/master">this repo.</a> In order to keep simplicity for people reading the first post before this one, I went ahead and located the Athena code in a dedicated branch.
Start by cloning the repo, checking out the branch and launching the infrastructure. If it's the first time you run this Dockerfile, it will built it before launching.
Also notice we are using the Docker "-d" option to launch the containers - we don't need to clog our terminal for now.
[code language="bash" gutter="false" light="false"]
git clone git@github.com:gosmarten/airflow.git
cd airflow/docker/airflow/
git checkout origin/athena-bootstrap
docker-compose up -d
```
In case you didn't read the post on Airflow and Docker, you will find the docker-compose.yml file is heavily commented specifically so you understand what is happening with each command.
<h2>Initiating the Athena DB</h2>
Usually this part of the process might not be part of our pipeline, but instead be a part of whatever provisioning technology you are using, such as Ansible or Terraform. However, since these are out of scope, at this point you can create the Athena DB and the table with our desired schema. The data we are using has: date, description, lang, category1, granularity.
So, follow the steps as in <a href="http://datacenternotes.com/2018/06/07/build-a-data-pipeline-with-aws-athena-and-airflow-part-1/">part 1</a> to create the database (<strong>historydb</strong>) or run the following command:
```{sql} CREATE DATABASE historydb; ```
Now create the table for the events (<strong>events_table</strong>) for which we'll be using airflow to add partitions routinely.
```{sql}
CREATE EXTERNAL TABLE IF NOT EXISTS historydb.events_table (
  `date` string,
  `​description` string,
  `lang` string,
  `category1` string,
  `granularity` string)
PARTITIONED BY (
  `year` int,
  `month` int,
  `day` int)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
  'separatorChar' = ',',
  'quoteChar' = '\"',
  'escapeChar' = '\\' )
LOCATION 's3://datacenternotes-athena/data/';
```
<h2>Airflow DAG</h2>
Now that we have the Athena side ready to receive ADD PARTITION commands, let's review our <a href="https://github.com/gosmarten/airflow/blob/athena-bootstrap/src/dags/athena.py">DAG</a> which has a pretty standard structure. We could certainly make the code prettier and more modular, but it would hinder the objective of keeping our focus in what it's important: Airflow working alongside with Athena.
<ol>
<li>Definition of some important variables</li>
<li>Some functions which will be used as Python callables by our DAGs</li>
<li>The Default Arguments of the DAG</li>
<li>The tasks themselves</li>
<li>The relationship between tasks</li>
</ol>
<h3>Setting Variables</h3>
[code language='python']
# SPECIFIC DAG SETTINGS
START_DATE = datetime(2010, 1, 1)
END_DATE = datetime(2010, 3, 31)
BUCKET_NAME = 'datacenternotes-athena'
ATHENA_DB = 'historydb'
ATHENA_TABLE = 'events_table'
AWS_REGION = 'eu-central-1'
AWS_ACCESS_KEY_ID = ''  # Add your own credentials here
AWS_SECRET_ACCESS_KEY = ''  # Add your own credentials here
```
If you followed the suggestions for the DB and Table name until now, then you will only need to change the BUCKET_NAME variable as well as <strong>fill in your credentials for AWS.</strong>
<h3>Functions</h3>
[code language='python']
def check_if_file_for_date(ds: str, **kwargs) -&amp;amp;amp;amp;gt; None:
    date_year = ds[:4]
    date_month = ds[5:7]
    date_day = ds[8:]
    session = boto3.Session(region_name=kwargs['AWS_REGION'],
                            aws_access_key_id=kwargs['AWS_ACCESS_KEY_ID'],
                            aws_secret_access_key=kwargs['AWS_SECRET_ACCESS_KEY'])
    s3client = session.client('s3')
    try:
        print('Found files for {}'.format(ds))
        a = s3client.get_object(Bucket=BUCKET_NAME, Key='data/year={}/month={}/day={}/data.csv'.format(
            date_year, date_month, date_day))
        add_partition = True
    except Exception:
        add_partition = False
        print("There are NO world events for {}".format(ds))
    ti = kwargs['ti']
    ti.xcom_push(key='add_partition', value=add_partition)
    return add_partition
```
Our first function will receive the "ds" argument which Airflow will inject into any task when the <strong>provide_context </strong>parameter is True, which is our case. This will be a string containing the execution date of the particular DAG. If you're reading this so you can start writing your own DAGs, yo'll see yourself using this variable more often then not, since most data pipelines have some kind of dependency on the date it's executing.
In our case, it will use the date to decompose YEAR, MONTH and DAY and transform this into the s3:// bucket path where it should check for a file. If there is no file, the DAG will move on to the second task (as opposed to fail). This is happening by personal decision because I know from the beginning that there are certain days for which there are no relevant world news and from my perspective this shouldn't be a failed DAG.
Finally, we push a variable <strong>add_partition </strong>to XCom which will have the value True in case this day needs to have a partition associated with it or False if it doesn't. XCom is Airflow's mechanism for tasks to leave messages between themselves.
Now for the second function:
[code language='python']
def execute_query(ds: str, **kwargs) -&amp;gt; None:
    ti = kwargs['ti']
    add_partition = ti.xcom_pull(task_ids='check_for_new_files',
                                 key='add_partition')
    if add_partition == False:
        pass
    else:
        print("Adding Partition")
        session = boto3.Session(region_name=kwargs['AWS_REGION'],
                                aws_access_key_id=kwargs['AWS_ACCESS_KEY_ID'],
                                aws_secret_access_key=kwargs['AWS_SECRET_ACCESS_KEY'])
        client = session.client(service_name='athena')
        result_configuration = {"OutputLocation": "s3://{}/".format(BUCKET_NAME)}
        date_year = ds[:4]
        date_month = ds[5:7]
        date_day = ds[8:]
        query = """
                ALTER TABLE {database}.{table}
                ADD PARTITION ( year='{year}', month='{month}', day='{day}' )
                location 's3://{bucket}/data/year={year}/month={month}/day={day}/';
                """.format(database=ATHENA_DB, table=ATHENA_TABLE, year=date_year,
                           month=date_month, day=date_day, bucket=BUCKET_NAME)
        query_response = client.start_query_execution(
            QueryString=query,
            ResultConfiguration=result_configuration
        )
        return query_response
```
This one picks up the XCom variable we defined earlier <strong>add_partition</strong> and, if set to True, will run an <strong>ALTER TABLE ADD PARTITION</strong> to our Athena DB which, once again, relies on the <strong>ds</strong> argument that is injected by Airflow.
<h3>Default Arguments</h3>
[code language='python']
default_args = {
    'owner': 'airflow',
    'depends_on_past': True,
    'start_date': datetime(2010, 1, 1),
    'end_date': datetime(2010, 3, 31),
    'retries': 1,
    'retry_delay': timedelta(hours=1),
}
dag = DAG(dag_id='add-events-partition-to-athena',
          default_args=default_args,
          schedule_interval='@daily',
          default_view='graph',
          max_active_runs=1)
```
It's worth pointing out that I'm are limiting the execution of the DAG between January and March 2010 because all our intermittent events are contain within those dates. If you are using this as forever-running Pipeline, just don't use an end date and will run until you shut it off.
Also worth mentioning is the fact that we are telling it to have a <strong>max_active_runs</strong> of 1 for this DAG which will, as expected, limit the ammount of concurrent tasks. In this case it shouldn't hurt, but for such a simple task I don't see the need to have it eating memory and CPU furiously for 16 tasks at a time.
<h3>Tasks</h3>
[code language='python']
task_1 = PythonOperator(task_id='check_for_new_files',
                        default_args=default_args,
                        python_callable=check_if_file_for_date,
                        dag=dag,
                        op_kwargs={'AWS_REGION': AWS_REGION,
                                   'AWS_ACCESS_KEY_ID': AWS_ACCESS_KEY_ID,
                                   'AWS_SECRET_ACCESS_KEY': AWS_SECRET_ACCESS_KEY},
                        provide_context=True)
task_2 = PythonOperator(task_id='update_partition',
                        default_args=default_args,
                        python_callable=execute_query,
                        dag=dag,
                        op_kwargs={'AWS_REGION': AWS_REGION,
                                   'AWS_ACCESS_KEY_ID': AWS_ACCESS_KEY_ID,
                                   'AWS_SECRET_ACCESS_KEY': AWS_SECRET_ACCESS_KEY},
                        provide_context=True)
```
Having understood the functions, the task definitions are now very straightforward. If you were still having doubts where those <strong>kwargs</strong> came from, then you should pay attention to the <strong>op_kwargs</strong> passed as optional in each of the tasks. While the <strong>ds</strong> argument is passed explicitly by Airflow, the kwargs will be passed under that same dictionary and will be accessible as expected.
As precedence goes, we are simply telling airflow that task_1 and task_2 should be ran sequentially.
<h3>Push Play</h3>
Now the only thing missing to make it work is actually turning on our DAG but going to <strong>http://localhost:8080</strong> (if you didn't change anything in the <strong>docker-compose.yml</strong> file and pushing the magic button:
<img class="alignnone size-full wp-image-4048" src="{{ site.baseurl }}/assets/2018/07/screen-shot-2018-07-21-at-18-34-00.png" alt="Screen Shot 2018-07-21 at 18.34.00.png" width="1602" height="253" />
Now just wait until it processes some days and, since we know the first event of our dataset happens on January, 4th 2010, we can query for it just as soon as it processed tasks for that day.
Go back to AWS Athena in the AWS console and run the query that will show you have succeeded in creating your Athena Pipeline with Airflow with standard SQL.
```{sql}
SELECT * FROM historydb.events_table
  WHERE month=01 AND day=04;
```
<h3>Conclusion</h3>
Thank you for reading through and I really hope you learned the fundamentals of how we can integrate both technologies in your day to day. As homework, I would suggest you add another function and a task_0 (meant to run before task_1) that would read data of the execution day of the DAG from some REST API and deposits this data in a CSV in a S3 bucket which has the same structure as we have been using so far. Crypto Compare has a free API that provides hourly data, so you could even go 1 layer down (to hours) with the S3 organization: https://min-api.cryptocompare.com/ Good luck!
Found an error or have any questions? Leave a comment down below!
