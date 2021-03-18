---
layout: post
title: Getting started with the Spark (part 2) -  SparkSQL
date: 2015-11-01 20:40:05.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Big Data
- python
- Spark
tags: []
meta:
  _oembed_a204a9078951a64b28ba7034bd0a175e: "<div class=\"embed-databricks\"><blockquote
    class=\"wp-embedded-content\"><a href=\"https://databricks.com/blog/2015/02/17/introducing-dataframes-in-spark-for-large-scale-data-science.html\">Introducing
    DataFrames in Apache Spark for Large Scale Data Science</a></blockquote><script
    type='text/javascript'><!--//--><![CDATA[//><!--\t\t!function(a,b){\"use strict\";function
    c(){if(!e){e=!0;var a,c,d,f,g=-1!==navigator.appVersion.indexOf(\"MSIE 10\"),h=!!navigator.userAgent.match(/Trident.*rv:11./),i=b.querySelectorAll(\"iframe.wp-embedded-content\");for(c=0;c<i.length;c++)if(d=i[c],!d.getAttribute(\"data-secret\")){if(f=Math.random().toString(36).substr(2,10),d.src+=\"#?secret=\"+f,d.setAttribute(\"data-secret\",f),g||h)a=d.cloneNode(!0),a.removeAttribute(\"security\"),d.parentNode.replaceChild(a,d)}else;}}var
    d=!1,e=!1;if(b.querySelector)if(a.addEventListener)d=!0;if(a.wp=a.wp||{},!a.wp.receiveEmbedMessage)if(a.wp.receiveEmbedMessage=function(c){var
    d=c.data;if(d.secret||d.message||d.value)if(!/[^a-zA-Z0-9]/.test(d.secret)){var
    e,f,g,h,i,j=b.querySelectorAll('iframe[data-secret=\"'+d.secret+'\"]'),k=b.querySelectorAll('blockquote[data-secret=\"'+d.secret+'\"]');for(e=0;e<k.length;e++)k[e].style.display=\"none\";for(e=0;e<j.length;e++)if(f=j[e],c.source===f.contentWindow){if(f.removeAttribute(\"style\"),\"height\"===d.message){if(g=parseInt(d.value,10),g>1e3)g=1e3;else
    if(~~g<200)g=200;f.height=g}if(\"link\"===d.message)if(h=b.createElement(\"a\"),i=b.createElement(\"a\"),h.href=f.getAttribute(\"src\"),i.href=d.value,i.host===h.host)if(b.activeElement===f)a.top.location.href=d.value}else;}},d)a.addEventListener(\"message\",a.wp.receiveEmbedMessage,!1),b.addEventListener(\"DOMContentLoaded\",c,!1),a.addEventListener(\"load\",c,!1)}(window,document);//--><!]]></script><iframe
    sandbox=\"allow-scripts\" security=\"restricted\" src=\"https://databricks.com/blog/2015/02/17/introducing-dataframes-in-spark-for-large-scale-data-science.html/embed\"
    width=\"600\" height=\"338\" title=\"&#8220;Introducing DataFrames in Apache Spark
    for Large Scale Data Science&#8221; &#8212; Databricks\" frameborder=\"0\" marginwidth=\"0\"
    marginheight=\"0\" scrolling=\"no\" class=\"wp-embedded-content\"></iframe></div>"
  _oembed_aa3e6e9f4168c87a90eea9bbcfb0406f: "{{unknown}}"
  _oembed_3fc5fb3188b35bafa86ac045dd2b51b9: "{{unknown}}"
  _oembed_time_a204a9078951a64b28ba7034bd0a175e: '1474911726'
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '27239521997'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2015/11/01/getting-started-with-the-spark-part-2-sparksql/"
---
Update: this tutorial has been updated mainly up to Spark 1.6.2 (with a minor detail regarding Spark 2.0), which is not the most recent version of Spark at the moment of updating of this post. Nonetheless, for the operations exemplified you can pretty much rest assured that the API has not changed substantially. I will try to do my best to update and point out these differences when they occur.
SparkSQL has become one of the core modules of Spark ecosystem (with its API already available for all currently supported languages, namely <a href="http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.DataFrame">Scala</a>, <a href="http://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/sql/DataFrame.html">Java</a>, <a href="http://spark.apache.org/docs/latest/api/python/pyspark.sql.html#pyspark.sql.DataFrame">Python</a>, and <a href="http://spark.apache.org/docs/latest/api/R/index.html">R</a>), and is intended to allow you to structure data into named columns (instead of with raw data without a schema as with RDDs) which can be queried with SQL syntax taking advantage of a distributed query engine running in the background.
In practical terms, you can very much think of it as equivalent as an attempt to emulate the API's of popular dataframes in R or Python pandas, as well as SQL tables. However, if I do my job correctly, I succeed in proving you that they are not the same (and hopefully more).
In order for you to use SparkSQL you need to import SQLContext. Let's start how you would do it with Python:
```{python}
# import required libs
from pyspark import SparkConf, SparkContext
# here is the import that allows you to use dataframes
from pyspark.sql import SQLContext
# initialize context in Spark 1.6.2
conf = SparkConf().setMaster('local[2]').setAppName('SparkSQL-demo')
sc = SparkContext(conf=conf)
sqlContext = SQLContext(sc)
```
Note: in the new Spark 2.0, this would indeed vary as such:
```{python}
# import required libs
from pyspark.sql import SparkSession
# initialize context in Spark 2.0
&lt;span class=&quot;n&quot;&gt;spark&lt;/span&gt; &lt;span class=&quot;o&quot;&gt;=&lt;/span&gt; &lt;span class=&quot;n&quot;&gt;SparkSession&lt;/span&gt;\
    &lt;span class=&quot;o&quot;&gt;.&lt;/span&gt;&lt;span class=&quot;n&quot;&gt;builder&lt;/span&gt;\
    &lt;span class=&quot;o&quot;&gt;.&lt;/span&gt;&lt;span class=&quot;n&quot;&gt;appName&lt;/span&gt;&lt;span class=&quot;p&quot;&gt;(&lt;/span&gt;'SparkSQL-demo')&lt;span class=&quot;o&quot;&gt;.&lt;/span&gt;&lt;span class=&quot;n&quot;&gt;getOrCreate&lt;/span&gt;&lt;span class=&quot;p&quot;&gt;()&lt;/span&gt;
```
Good news, Scala does not vary much:
```{scala}
//import required libs
import org.apache.spark.{SparkConf, SparkContext}
//here is the import that allows you to use dataframes
import org.apache.spark.sql.SQLContext
# initialize context in Spark 1.6.2
val conf = new SparkConf().setMaster(&amp;quot;local[2]&amp;quot;).setAppName(&amp;quot;SparkSQL-demo&amp;quot;)
val sc = new SparkContext(conf=conf)
val sqlContext = new SQLContext(sc)
```
&nbsp;
A dataframe can be created from an existing RDD, as well as similarly to RDDs from a variety of different sources such astext files, HDFS, S3,  Hive tables, external databases, etc. Here's an example how to build a dataframe by reading from S3 (in Python):
```{python}
df = sqlContext.read.json('s3n://my-bucket/example/file.json')
```
&nbsp;
Note: we use "s3n" prefix which is valid for a serious of regions in AWS, but not valid for all of them. For example, to access S3 buckets in Frankfurt region you would rather use "s3a" as a prefix.
<strong>What sets Spark Dataframes apart</strong>
However, and even though the goal is to hide the maximum amount of disparities, there are nonetheless big conceptual differences between a Spark Dataframe and a Pandas dataframe. The first is that the <strong>immutability</strong> side has (thank God) not been removed. In other words, when you call an action on an Dataframe, you do <em><span style="text-decoration:underline;">need</span></em> to save it into a variable. I've seen a lot of data scientists stumble on this in their first Spark days, so let us illustrate this in a very simple example:
```{python}
# Create a Dataframe with pseudo data:
column_names = [&amp;quot;customer_name&amp;quot;, &amp;quot;date&amp;quot;, &amp;quot;category&amp;quot;,
&amp;quot;product_name&amp;quot;, &amp;quot;quantity&amp;quot;, &amp;quot;price&amp;quot;]
# Note that we create an RDD, and then cast it to
# Dataframe by providing a schema
column_names = [&quot;customer_name&quot;, &quot;date&quot;, &quot;category&quot;,
&quot;product_name&quot;, &quot;quantity&quot;, &quot;price&quot;]
df01 = sc.parallelize(
[(&quot;Geoffrey&quot;, &quot;2016-04-22&quot;, &quot;A&quot;, &quot;apples&quot;, 1, 50.00),
(&quot;Yann&quot;, &quot;2016-04-22&quot;, &quot;B&quot;, &quot;Lamp&quot;, 1, 38.00)]).toDF(column_names)
df02 = df01.select(&quot;customer_name&quot;, &quot;date&quot;, &quot;category&quot;, &quot;product_name&quot;, &quot;price&quot;, (df01['quantity']+1).alias(&quot;quantity&quot;))
df01.show()
df02.show()
+-------------+----------+--------+------------+--------+-----+
|customer_name|      date|category|product_name|quantity|price|
+-------------+----------+--------+------------+--------+-----+
|     Geoffrey|2016-04-22|       A|      apples|       1| 50.0|
|         Yann|2016-04-22|       B|        Lamp|       1| 38.0|
+-------------+----------+--------+------------+--------+-----+
# .. where df02 is, as expected df02.show()
+-------------+----------+--------+------------+-----+--------+
|customer_name|      date|category|product_name|price|quantity|
+-------------+----------+--------+------------+-----+--------+
|     Geoffrey|2016-04-22|       A|      apples| 50.0|       2|
|         Yann|2016-04-22|       B|        Lamp| 38.0|       2|
+-------------+----------+--------+------------+-----+--------+
```
The second most obvious difference (compared to pandas df) is the distributed nature of a Spark Dataframe, similarly as RDDs are. As a matter a fact,in previous versions of Spark Dataframes were even called SchemaRDD. So internally the same redundancy principles of RDDs apply, you should in fact still keep in mind the Driver - Executor programming model, as well as Actions and Transformations concept.
Also important to note is that Spark Dataframes use internally a query optmizer, one of the reasons why the performance difference between native Scala written Spark code vs Python or Java drops significantly as can be seen in the next illustration <a href="https://databricks.com/blog/2015/02/17/introducing-dataframes-in-spark-for-large-scale-data-science.html" target="_blank">provided by databricks</a>:
<img class="alignnone size-full wp-image-2825" src="{{ site.baseurl }}/assets/2015/11/perf_diffs_from_dataframes_vs_rdds.png" alt="perf_diffs_from_dataframes_vs_rdds" width="1024" height="457" />
&nbsp;
<strong>API</strong>
Let us view some useful operations one can perform with spark dfs. For the sake of brevity I will continue this tutorial mainly using the python API, as the syntax does not really vary that much.
Let us create a bigger fake (and complete utterly non-sense) dataset to make operations more interesting:
```{python}
customers = sc.parallelize([(&quot;Geoffrey&quot;, &quot;2016-04-22&quot;, &quot;A&quot;, &quot;apples&quot;, 1, 50.00),
(&quot;Geoffrey&quot;, &quot;2016-05-03&quot;, &quot;B&quot;, &quot;Lamp&quot;, 2, 38.00),
(&quot;Geoffrey&quot;, &quot;2016-05-03&quot;, &quot;D&quot;, &quot;Solar Pannel&quot;, 1, 29.00),
(&quot;Geoffrey&quot;, &quot;2016-05-03&quot;, &quot;A&quot;, &quot;apples&quot;, 3, 50.00),
(&quot;Geoffrey&quot;, &quot;2016-05-03&quot;, &quot;C&quot;, &quot;Rice&quot;, 5, 15.00),
(&quot;Geoffrey&quot;, &quot;2016-06-05&quot;, &quot;A&quot;, &quot;apples&quot;, 5, 50.00),
(&quot;Geoffrey&quot;, &quot;2016-06-05&quot;, &quot;A&quot;, &quot;bananas&quot;, 5, 55.00),
(&quot;Geoffrey&quot;, &quot;2016-06-15&quot;, &quot;Y&quot;, &quot;Motor skate&quot;, 7, 68.00),
(&quot;Geoffrey&quot;, &quot;2016-06-15&quot;, &quot;E&quot;, &quot;Book: The noose&quot;, 1, 125.00),
(&quot;Yann&quot;, &quot;2016-04-22&quot;, &quot;B&quot;, &quot;Lamp&quot;, 1, 38.00),
(&quot;Yann&quot;, &quot;2016-05-03&quot;, &quot;Y&quot;, &quot;Motor skate&quot;, 1, 68.00),
(&quot;Yann&quot;, &quot;2016-05-03&quot;, &quot;D&quot;, &quot;Recycle bin&quot;, 5, 27.00),
(&quot;Yann&quot;, &quot;2016-05-03&quot;, &quot;C&quot;, &quot;Rice&quot;, 15, 15.00),
(&quot;Yann&quot;, &quot;2016-04-02&quot;, &quot;A&quot;, &quot;bananas&quot;, 3, 55.00),
(&quot;Yann&quot;, &quot;2016-04-02&quot;, &quot;B&quot;, &quot;Lamp&quot;, 2, 38.00),
(&quot;Yann&quot;, &quot;2016-04-03&quot;, &quot;E&quot;, &quot;Book: Crime and Punishment&quot;, 5, 100.00),
(&quot;Yann&quot;, &quot;2016-04-13&quot;, &quot;E&quot;, &quot;Book: The noose&quot;, 5, 125.00),
(&quot;Yann&quot;, &quot;2016-04-27&quot;, &quot;D&quot;, &quot;Solar Pannel&quot;, 5, 29.00),
(&quot;Yann&quot;, &quot;2016-05-27&quot;, &quot;D&quot;, &quot;Recycle bin&quot;, 5, 27.00),
(&quot;Yann&quot;, &quot;2016-05-27&quot;, &quot;A&quot;, &quot;bananas&quot;, 3, 55.00),
(&quot;Yann&quot;, &quot;2016-05-01&quot;, &quot;Y&quot;, &quot;Motor skate&quot;, 1, 68.00),
(&quot;Yann&quot;, &quot;2016-06-07&quot;, &quot;Z&quot;, &quot;space ship&quot;, 1, 227.00),
(&quot;Yoshua&quot;, &quot;2016-02-07&quot;, &quot;Z&quot;, &quot;space ship&quot;, 2, 227.00),
(&quot;Yoshua&quot;, &quot;2016-02-14&quot;, &quot;A&quot;, &quot;bananas&quot;, 9, 55.00),
(&quot;Yoshua&quot;, &quot;2016-02-14&quot;, &quot;B&quot;, &quot;Lamp&quot;, 2, 38.00),
(&quot;Yoshua&quot;, &quot;2016-02-14&quot;, &quot;A&quot;, &quot;apples&quot;, 10, 55.00),
(&quot;Yoshua&quot;, &quot;2016-03-07&quot;, &quot;Z&quot;, &quot;space ship&quot;, 5, 227.00),
(&quot;Yoshua&quot;, &quot;2016-04-07&quot;, &quot;Y&quot;, &quot;Motor skate&quot;, 4, 68.00),
(&quot;Yoshua&quot;, &quot;2016-04-07&quot;, &quot;D&quot;, &quot;Recycle bin&quot;, 5, 27.00),
(&quot;Yoshua&quot;, &quot;2016-04-07&quot;, &quot;C&quot;, &quot;Rice&quot;, 5, 15.00),
(&quot;Yoshua&quot;, &quot;2016-04-07&quot;, &quot;A&quot;, &quot;bananas&quot;, 9, 55.00),
(&quot;Jurgen&quot;, &quot;2016-05-01&quot;, &quot;Z&quot;, &quot;space ship&quot;, 1, 227.00),
(&quot;Jurgen&quot;, &quot;2016-05-01&quot;, &quot;A&quot;, &quot;bananas&quot;, 5, 55.00),
(&quot;Jurgen&quot;, &quot;2016-05-08&quot;, &quot;A&quot;, &quot;bananas&quot;, 5, 55.00),
(&quot;Jurgen&quot;, &quot;2016-05-08&quot;, &quot;Y&quot;, &quot;Motor skate&quot;, 1, 68.00),
(&quot;Jurgen&quot;, &quot;2016-06-05&quot;, &quot;A&quot;, &quot;bananas&quot;, 5, 55.00),
(&quot;Jurgen&quot;, &quot;2016-06-05&quot;, &quot;C&quot;, &quot;Rice&quot;, 5, 15.00),
(&quot;Jurgen&quot;, &quot;2016-06-05&quot;, &quot;Y&quot;, &quot;Motor skate&quot;, 2, 68.00),
(&quot;Jurgen&quot;, &quot;2016-06-05&quot;, &quot;D&quot;, &quot;Recycle bin&quot;, 5, 27.00),
]).toDF([&quot;customer_name&quot;, &quot;date&quot;, &quot;category&quot;, &quot;product_name&quot;, &quot;quantity&quot;, &quot;price&quot;])
```
&nbsp;
The dataset is minimal for the example, so it really does not make a difference to cache it in memory. But my point is to illustrate that similarly to RDDs, you should also care about memory persistence, and can indeed use a similar API. Moreover, since next we are going to use several distinct operations of type "Action" (yielding back results to the driver), it is of good practice to signal that this dataframe should be kept in memory for further operations.
```{python}
print(customers.is_cached)
customers.cache()
print(customers.is_cached)
# False
# True
```
OK, so yes, there are some exceptions, and in this case as you can see the cache() method does not obey the laws of immutability. Moving on, let's take a peak on it:
```{python}
customers.show(10)
+-------------+----------+--------+---------------+--------+-----+
|customer_name|      date|category|   product_name|quantity|price|
+-------------+----------+--------+---------------+--------+-----+
|     Geoffrey|2016-04-22|       A|         apples|       1| 50.0|
|     Geoffrey|2016-05-03|       B|           Lamp|       2| 38.0|
|     Geoffrey|2016-05-03|       D|   Solar Pannel|       1| 29.0|
|     Geoffrey|2016-05-03|       A|         apples|       3| 50.0|
|     Geoffrey|2016-05-03|       C|           Rice|       5| 15.0|
|     Geoffrey|2016-06-05|       A|         apples|       5| 50.0|
|     Geoffrey|2016-06-05|       A|        bananas|       5| 55.0|
|     Geoffrey|2016-06-15|       Y|    Motor skate|       7| 68.0|
|     Geoffrey|2016-06-15|       E|Book: The noose|       1|125.0|
|         Yann|2016-04-22|       B|           Lamp|       1| 38.0|
+-------------+----------+--------+---------------+--------+-----+
only showing top 10 rows
```
&nbsp;
As previously referred, one advantage over RDDs is the ability to use standard SQL syntax to slice and dice over a Spark Dataframe. Let's check for example all distinct products that we have:
```{python}
spark_sql_table = 'customers'
sql_qry = &quot;select distinct(product_name) from {table}&quot;.format(table=spark_sql_table)
customers.registerTempTable(spark_sql_table)
distinct_product_names_df01 = sqlContext.sql(sql_qry)
distinct_product_names_df01.show(truncate=False)
+--------------------------+
|product_name              |
+--------------------------+
|space ship                |
|Book: The noose           |
|Rice                      |
|Motor skate               |
|Book: Crime and Punishment|
|Recycle bin               |
|Lamp                      |
|bananas                   |
|Solar Pannel              |
|apples                    |
+--------------------------+
```
&nbsp;
Similarly to how we work with RDDs, with Spark DataFrames you are not restricted to SQL, and can indeed call methods on dataframes, which yields a new dataframe (remember the immutability story, right?)
<div class="text_cell_render rendered_html">
Let's check for example the same previous example on how to get all distinct:
```{python}
distinct_product_names_df02 = customers.select('product_name').distinct()
distinct_product_names_df02.show(truncate=False)
distinct_product_names_df02.count()
+--------------------------+
|product_name |
+--------------------------+
|space ship |
|Book: The noose |
|Rice |
|Motor skate |
|Book: Crime and Punishment|
|Recycle bin |
|Lamp |
|bananas |
|Solar Pannel |
|apples |
+--------------------------+
10
```
</div>
&nbsp;
Note that you can as well work on a dataframe just like you iterate on an RDD, which causes the dataframe to be cast into a RDD in the background:
```{python}
distinct_product_names_rdd = customers.map(lambda row: row[3])
# calling distinct_product_names_rdd.show() will yield an exception
try:
 distinct_product_names_rdd.show()
except Exception as err:
 print('Error: {}'.format(err))
 print('BUT calling .take() this works, as it is an RDD now: {}' \
 .format(distinct_product_names_rdd.take(3)) )
distinct_product_names = distinct_product_names_rdd.distinct().collect()
print('Distinct product names are: {}'.format(distinct_product_names))
print('\n... And, as expected, they are {} in total.' \
 .format(len(distinct_product_names)))
Error: 'PipelinedRDD' object has no attribute 'show'
BUT calling .take() this works, as it is an RDD now: ['apples', 'Lamp', 'Solar Pannel']
Distinct product names are: ['Rice', 'bananas', 'space ship', 'Book: The noose', 'Solar Pannel', 'Recycle bin', 'apples', 'Lamp', 'Book: Crime and Punishment', 'Motor skate']
... And, as expected, they are 10 in total.
```
&nbsp;
Similar than in R or Python, you can have summary statistics. Let's get statistics for Category, quantity and price columns:
```{python}
customers.describe('category', 'quantity', 'price').show()
&amp;amp;amp;amp;amp;nbsp;
+-------+--------+------------------+------------------+
|summary|category|          quantity|             price|
+-------+--------+------------------+------------------+
|  count|      39|                39|                39|
|   mean|    null| 4.153846153846154| 68.94871794871794|
| stddev|    null|2.9694803863945936|59.759583559965165|
|    min|       A|                 1|              15.0|
|    max|       Z|                15|             227.0|
+-------+--------+------------------+------------------+
```
&nbsp;
Being a categorical variable, as you can see the summary regarding the 'category' column does not make much sense. We'll work on better on this later, but for now, let's do something simple and try to look into product_name and category.
Let's suppose we are interested in finding out how does purchasing behavior vary among categories, namely what is the average frequency of purchases per category.
```{python}
cat_freq_per_cust = customers.stat.crosstab('customer_name', 'category')
cat_freq_per_cust.show()
+----------------------+---+---+---+---+---+---+---+
|customer_name_category|  Y|  Z|  A|  B|  C|  D|  E|
+----------------------+---+---+---+---+---+---+---+
|              Geoffrey|  1|  0|  4|  1|  1|  1|  1|
|                Yoshua|  1|  2|  3|  1|  1|  1|  0|
|                  Yann|  2|  1|  2|  2|  1|  3|  2|
|                Jurgen|  2|  1|  3|  0|  1|  1|  0|
+----------------------+---+---+---+---+---+---+---+
```
&nbsp;
```{python}
cols = cat_freq_per_cust.columns
cat_freq_per_cust.describe(cols[1::]).show(truncate=False)
+-------+------------------+-----------------+-----------------+-----------------+---+------------------+------------------+
|summary|Y                 |Z                |A                |B                |C  |D                 |E                 |
+-------+------------------+-----------------+-----------------+-----------------+---+------------------+------------------+
|count  |4                 |4                |4                |4                |4  |4                 |4                 |
|mean   |1.5               |1.0              |3.0              |1.0              |1.0|1.5               |0.75              |
|stddev |0.5773502691896257|0.816496580927726|0.816496580927726|0.816496580927726|0.0|0.9999999999999999|0.9574271077563381|
|min    |1                 |0                |2                |0                |1  |1                 |0                 |
|max    |2                 |2                |4                |2                |1  |3                 |2                 |
+-------+------------------+-----------------+-----------------+-----------------+---+------------------+------------------+
```
&nbsp;
Note: "count" column does not have much meaning here; it simply counts number of rows blindly, and as expected since the rows without a given value are filled with "zero" instead of Null, it counts them too.
Let's check the same for products.
```{python}
# Let's view products vertically, for formatting reasons
customers.stat.crosstab('product_name', 'customer_name' )\
.show(truncate=False)
+--------------------------+--------+------+----+------+
|product_name_customer_name|Geoffrey|Jurgen|Yann|Yoshua|
+--------------------------+--------+------+----+------+
|Book: Crime and Punishment|0       |0     |1   |0     |
|Recycle bin               |0       |1     |2   |1     |
|Solar Pannel              |1       |0     |1   |0     |
|space ship                |0       |1     |1   |2     |
|Motor skate               |1       |2     |2   |1     |
|Rice                      |1       |1     |1   |1     |
|apples                    |3       |0     |0   |1     |
|Lamp                      |1       |0     |2   |1     |
|bananas                   |1       |3     |2   |2     |
|Book: The noose           |1       |0     |1   |0     |
+--------------------------+--------+------+----+------+
```
&nbsp;
```{python}
freq_products = customers.stat.crosstab('customer_name', 'product_name')
freq_prod_cols = freq_products.columns
freq_products.describe(freq_prod_cols[1::]).show()
+-------+-----------------+-----------------+------------------+----+------------------+-----------------+------------------+--------------------------+-----------------+------------------+
|summary|          bananas|       space ship|   Book: The noose|Rice|       Motor skate|             Lamp|            apples|Book: Crime and Punishment|      Recycle bin|      Solar Pannel|
+-------+-----------------+-----------------+------------------+----+------------------+-----------------+------------------+--------------------------+-----------------+------------------+
|  count|                4|                4|                 4|   4|                 4|                4|                 4|                         4|                4|                 4|
|   mean|              2.0|              1.0|               0.5| 1.0|               1.5|              1.0|               1.0|                      0.25|              1.0|               0.5|
| stddev|0.816496580927726|0.816496580927726|0.5773502691896257| 0.0|0.5773502691896257|0.816496580927726|1.4142135623730951|                       0.5|0.816496580927726|0.5773502691896257|
|    min|                1|                0|                 0|   1|                 1|                0|                 0|                         0|                0|                 0|
|    max|                3|                2|                 1|   1|                 2|                2|                 3|                         1|                2|                 1|
+-------+-----------------+-----------------+------------------+----+------------------+-----------------+------------------+--------------------------+-----------------+------------------+
```
Alternatively we could have answered suing a more SQL-like approach:
```{python}
cust_prod = customers.groupBy('customer_name', 'product_name') \
 .agg(func.count('product_name')) \
 .orderBy('product_name', 'customer_name')
cust_prod.show(cust_prod.count(), truncate=False)
+-------------+--------------------------+-------------------+
|customer_name|product_name              |count(product_name)|
+-------------+--------------------------+-------------------+
|Yann         |Book: Crime and Punishment|1                  |
|Geoffrey     |Book: The noose           |1                  |
|Yann         |Book: The noose           |1                  |
|Geoffrey     |Lamp                      |1                  |
|Yann         |Lamp                      |2                  |
|Yoshua       |Lamp                      |1                  |
|Geoffrey     |Motor skate               |1                  |
|Jurgen       |Motor skate               |2                  |
|Yann         |Motor skate               |2                  |
|Yoshua       |Motor skate               |1                  |
|Jurgen       |Recycle bin               |1                  |
|Yann         |Recycle bin               |2                  |
|Yoshua       |Recycle bin               |1                  |
|Geoffrey     |Rice                      |1                  |
|Jurgen       |Rice                      |1                  |
|Yann         |Rice                      |1                  |
|Yoshua       |Rice                      |1                  |
|Geoffrey     |Solar Pannel              |1                  |
|Yann         |Solar Pannel              |1                  |
|Geoffrey     |apples                    |3                  |
|Yoshua       |apples                    |1                  |
|Geoffrey     |bananas                   |1                  |
|Jurgen       |bananas                   |3                  |
|Yann         |bananas                   |2                  |
|Yoshua       |bananas                   |2                  |
|Jurgen       |space ship                |1                  |
|Yann         |space ship                |1                  |
|Yoshua       |space ship                |2                  |
+-------------+--------------------------+-------------------+
```
&nbsp;
```{python}
cust_prod.groupBy('product_name') \
 .agg(func.avg('count(product_name)')) \
 .show(truncate=False)
+--------------------------+------------------------+
|product_name              |avg(count(product_name))|
+--------------------------+------------------------+
|space ship                |1.3333333333333333      |
|Book: The noose           |1.0                     |
|Rice                      |1.0                     |
|Motor skate               |1.5                     |
|Book: Crime and Punishment|1.0                     |
|Recycle bin               |1.3333333333333333      |
|Lamp                      |1.3333333333333333      |
|bananas                   |2.0                     |
|Solar Pannel              |1.0                     |
|apples                    |2.0                     |
+--------------------------+------------------------+
```
Note that this does <i>not</i> take into account the quantity of each product bought, just how many times client was there to buy a given item. For our last question that may be OK, because we understand the different nature of products.
But let us assume it is not OK, and we do need to count by quantity. For that we can use pivot dataframes, a feature introduced in Spark 1.6
```{python}
total_products_bought_by_customer = customers.groupBy('customer_name')\
 .pivot('product_name').sum('quantity')
total_products_bought_by_customer.show()
+-------------+--------------------------+---------------+----+-----------+-----------+----+------------+------+-------+----------+
|customer_name|Book: Crime and Punishment|Book: The noose|Lamp|Motor skate|Recycle bin|Rice|Solar Pannel|apples|bananas|space ship|
+-------------+--------------------------+---------------+----+-----------+-----------+----+------------+------+-------+----------+
|         Yann|                         5|              5|   3|          2|         10|  15|           5|  null|      6|         1|
|       Yoshua|                      null|           null|   2|          4|          5|   5|        null|    10|     18|         7|
|     Geoffrey|                      null|              1|   2|          7|       null|   5|           1|     9|      5|      null|
|       Jurgen|                      null|           null|null|          3|          5|   5|        null|  null|     15|         1|
+-------------+--------------------------+---------------+----+-----------+-----------+----+------------+------+-------+----------+
```
&nbsp;
Hope this has been a good introduction, and I suggest that you further check out <a href="https://databricks.com/blog/2015/02/17/introducing-dataframes-in-spark-for-large-scale-data-science.html" target="_blank">this</a> great post from databricks covering similar operations.
For the last part of this tutorial series I plan to cover window functions, something that until the more recent Spark 2.0 was possible thanks to HiveContext.
