---
layout: post
title: Getting Started with Spark (part 3) -  UDFs & Window functions
date: 2016-10-03 19:30:37.000000000 +02:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Big Data
- Spark
tags: []
meta:
  _rest_api_published: '1'
  _rest_api_client_id: "-1"
  _publicize_job_id: '27470464887'
author:
  login: diogoaurelio
  email: diogo@mklabs.io
  display_name: Diogo Aurélio
  first_name: Diogo
  last_name: Aurélio
permalink: "/2016/10/03/getting-started-with-spark-part-3-udfs-window-functions/"
---
This post attempts to continue the previous introductory series "<a href="https://datacenternotes.wordpress.com/2015/11/01/getting-started-with-spark-in-pythonscala-part-1/" target="_blank" rel="noopener">Getting started with Spark in Python</a>" with the topics UDFs and Window Functions.
<ul>
<li><a href="https://datacenternotes.com/2015/11/01/getting-started-with-spark-in-pythonscala-part-1/" target="_blank" rel="noopener">Part 1 Getting Started - covers basics on distributed Spark architecture, along with Data structures (including the old good RDD collections (!), whose use has been kind of deprecated by Dataframes)</a></li>
<li><a href="https://datacenternotes.com/2015/11/01/getting-started-with-the-spark-part-2-sparksql/" target="_blank" rel="noopener">Part 2 intro to Dataframes</a></li>
<li><a href="https://datacenternotes.com/2016/10/03/getting-started-with-spark-part-3-udfs-window-functions/" target="_blank" rel="noopener">Part 3 intro to UDFs and Window Functions </a></li>
<li><a href="https://wordpress.com/post/datacenternotes.com/4209" target="_blank" rel="noopener">Part 4 unit testing in PySpark environments</a></li>
</ul>
Note: For this post I'm using Spark 1.6.1. There are some minor differences in comparison to the <a href="https://spark.apache.org/releases/spark-release-2-0-0.html" target="_blank" rel="noopener">new coming Spark 2.0</a>, such as using a SparkSession object to initialize the Spark Context, instead of HiveContext as I do here. Nonetheless, the important parts are common in both.
{:.lead}
<strong>HiveContext</strong>
Now up until Spark 1.6.2, the only way for you to enrich your SQL queries would be to use a HiveContext instead of Spark SQLContext.
With a HiveContext you got the same features of a SparkContext, but with some of additional advantageous, such as ability to use window functions, access to Hive UDFs, besides the ability to read data from Hive tables. For more detail, please refer <a href="http://stackoverflow.com/questions/33666545/what-is-the-difference-between-apache-spark-sqlcontext-vs-hivecontext" target="_blank" rel="noopener">here for a concise well explained answer to the differences between SQLContext and HiveContext</a>.
OK, let us start by importing all required dependencies for this tutorial:
```{python}
# python dependencies
import sys
from datetime import datetime as dt
# pyspark dependencies
from pyspark import SparkConf, SparkContext
from pyspark.sql import SQLContext, HiveContext
from pyspark.sql.dataframe import DataFrame
from pyspark.sql.window import Window
import pyspark.sql.functions as func
from pyspark.sql.types import *
from pyspark.sql.functions import lit
from pyspark.sql.functions import udf
```
... and initialize the HiveContext:
```{python}
conf = SparkConf().setMaster("local[4]").setAppName("window-demo") \
      .set("spark.driver.memory", "4g")
sc = SparkContext(conf=conf)
sqlContext = HiveContext(sc)
```
As a reminder, our (extremely)  dummy dataset is comprised of the following (non-sense) data:
```{python}
# note: to simplify, not providing the schema, Spark Df api will infer it
customers = sc.parallelize([("Geoffrey", "2016-04-22", "A", "apples", 1, 50.00),
("Geoffrey", "2016-05-03", "B", "Lamp", 2, 38.00),
("Geoffrey", "2016-05-03", "D", "Solar Pannel", 1, 29.00),
("Geoffrey", "2016-05-03", "A", "apples", 3, 50.00),
("Geoffrey", "2016-05-03", "C", "Rice", 5, 15.00),
("Geoffrey", "2016-06-05", "A", "apples", 5, 50.00),
("Geoffrey", "2016-06-05", "A", "bananas", 5, 55.00),
("Geoffrey", "2016-06-15", "Y", "Motor skate", 7, 68.00),
("Geoffrey", "2016-06-15", "E", "Book: The noose", 1, 125.00),
("Yann", "2016-04-22", "B", "Lamp", 1, 38.00),
("Yann", "2016-05-03", "Y", "Motor skate", 1, 68.00),
("Yann", "2016-05-03", "D", "Recycle bin", 5, 27.00),
("Yann", "2016-05-03", "C", "Rice", 15, 15.00),
("Yann", "2016-04-02", "A", "bananas", 3, 55.00),
("Yann", "2016-04-02", "B", "Lamp", 2, 38.00),
("Yann", "2016-04-03", "E", "Book: Crime and Punishment", 5, 100.00),
("Yann", "2016-04-13", "E", "Book: The noose", 5, 125.00),
("Yann", "2016-04-27", "D", "Solar Pannel", 5, 29.00),
("Yann", "2016-05-27", "D", "Recycle bin", 5, 27.00),
("Yann", "2016-05-27",&amp;nbsp; "A", "bananas", 3, 55.00),
("Yann", "2016-05-01", "Y", "Motor skate", 1, 68.00),
("Yann", "2016-06-07", "Z", "space ship", 1, 227.00),
("Yoshua", "2016-02-07", "Z", "space ship", 2, 227.00),
("Yoshua", "2016-02-14", "A", "bananas", 9, 55.00),
("Yoshua", "2016-02-14", "B", "Lamp", 2, 38.00),
("Yoshua", "2016-02-14", "A", "apples", 10, 55.00),
("Yoshua", "2016-03-07", "Z", "space ship", 5, 227.00),
("Yoshua", "2016-04-07", "Y", "Motor skate", 4, 68.00),
("Yoshua", "2016-04-07", "D", "Recycle bin", 5, 27.00),
("Yoshua", "2016-04-07", "C", "Rice", 5, 15.00),
("Yoshua", "2016-04-07", "A", "bananas", 9, 55.00),
("Jurgen", "2016-05-01", "Z", "space ship", 1, 227.00),
("Jurgen", "2016-05-01", "A", "bananas", 5, 55.00),
("Jurgen", "2016-05-08", "A", "bananas", 5, 55.00),
("Jurgen", "2016-05-08", "Y", "Motor skate", 1, 68.00),
("Jurgen", "2016-06-05", "A", "bananas", 5, 55.00),
("Jurgen", "2016-06-05", "C", "Rice", 5, 15.00),
("Jurgen", "2016-06-05", "Y", "Motor skate", 2, 68.00),
("Jurgen", "2016-06-05", "D", "Recycle bin", 5, 27.00),
]).toDF(["customer_name", "date", "category", "product_name", "quantity", "price"])
```
What if we wanted to answer a question such as: <b>What is the cumulative sum of spending of each customer throughout time?</b>
A way of thinking of a cumulative sum is as a recursive call where for every new period you sum the current value plus the all the previous accumulated. So one way to solve this is by using <b>Window Functions</b>, a functionality added back in Spark 1.4. However, let us start by adding a column with amount spent, using Spark <b>User Defined Functions (UDFs)</b> for that. These functions basically apply a given function to every row on one or more columns.
```{python}
# create the general function
def amount_spent(quantity, price):
   """
   Calculates the product between two variables
   :param quantity: (float/int)
   :param price: (float/int)
   :return:
           (float/int)
   """
   return quantity * price
# create the general UDF
amount_spent_udf = udf(amount_spent, DoubleType())
# Note: DoubleType in Java/Scala is equal to Python float; thus you can alternatively specify FloatType()
# Apply our UDF to the dataframe
customers02 = customers.withColumn('amount_spent', amount_spent_udf(customers['quantity'], customers['price'])).cache()
customers02.show(3, truncate=False)
+-------------+----------+--------+------------+--------+-----+------------+
|customer_name|date      |category|product_name|quantity|price|amount_spent|
+-------------+----------+--------+------------+--------+-----+------------+
|Geoffrey     |2016-04-22|A       |apples      |1       |50.0 |50.0        |
|Geoffrey     |2016-05-03|B       |Lamp        |2       |38.0 |76.0        |
|Geoffrey     |2016-05-03|D       |Solar Pannel|1       |29.0 |29.0        |
+-------------+----------+--------+------------+--------+-----+------------+
only showing top 3 rows
```
To compute a cumulating sum over time, we need to build a window object and specify how it should be partitioned (aka how to determine which intervals should be used for the aggregation computation, meaning which column to use), and optionally the interval to build a window.
```{python}
window_01 = Window.partitionBy("customer_name").orderBy("date", "category").rowsBetween(-sys.maxsize, 0)
# note: func was the name given to functions, a Spark API for a suite of convenience functions
win_customers01 = customers02.withColumn("cumulative_sum", func.sum(customers02['amount_spent']).over(window_01))
win_customers01.show(10, truncate=False)
+-------------+----------+--------+--------------------------+--------+-----+------------+--------------+
|customer_name|date      |category|product_name              |quantity|price|amount_spent|cumulative_sum|
+-------------+----------+--------+--------------------------+--------+-----+------------+--------------+
|Yann         |2016-04-02|A       |bananas                   |3       |55.0 |165.0       |165.0         |
|Yann         |2016-04-02|B       |Lamp                      |2       |38.0 |76.0        |241.0         |
|Yann         |2016-04-03|E       |Book: Crime and Punishment|5       |100.0|500.0       |741.0         |
|Yann         |2016-04-13|E       |Book: The noose           |5       |125.0|625.0       |1366.0        |
|Yann         |2016-04-22|B       |Lamp                      |1       |38.0 |38.0        |1404.0        |
|Yann         |2016-04-27|D       |Solar Pannel              |5       |29.0 |145.0       |1549.0        |
|Yann         |2016-05-01|Y       |Motor skate               |1       |68.0 |68.0        |1617.0        |
|Yann         |2016-05-03|C       |Rice                      |15      |15.0 |225.0       |1842.0        |
|Yann         |2016-05-03|D       |Recycle bin               |5       |27.0 |135.0       |1977.0        |
|Yann         |2016-05-03|Y       |Motor skate               |1       |68.0 |68.0        |2045.0        |
+-------------+----------+--------+--------------------------+--------+-----+------------+--------------+
only showing top 10 rows
```
Cumalative sum calculation is partitioned by each customer in an interval from the beginning (-sys.maxsize effectively mean start at the very first row in that partition) until the current row (which when the aggregation function is sliding has an index of Zero), and finally ordered by date and category ascending (default).
So just to be sure we're perfectly clear:
<b>cumlative_sum row zero = (amount_spent row zero)</b>;
<b>cumlative_sum row one = (cumlative_sum row zero + amount_spent row zero); </b>
Also, and as a side note, alternatively than defining an UDF, we could specify directly the sum function over the product of two columns (as the Functions.sum is also a UDF).
```{python}
# Note: instead of defining an UDF, you could alternatively specify directly
window_01 = Window.partitionBy("customer_name").orderBy("date").rowsBetween(-sys.maxsize, 0)
win_customers01_B = customers.withColumn("cumulative_sum", func.sum(customers['price']*customers['quantity']).over(window_01))
win_customers01_B.show(3, truncate=False)
+-------------+----------+--------+--------------------------+--------+-----+--------------+
|customer_name|date      |category|product_name              |quantity|price|cumulative_sum|
+-------------+----------+--------+--------------------------+--------+-----+--------------+
|Yann         |2016-04-02|A       |bananas                   |3       |55.0 |165.0         |
|Yann         |2016-04-02|B       |Lamp                      |2       |38.0 |241.0         |
|Yann         |2016-04-03|E       |Book: Crime and Punishment|5       |100.0|741.0         |
+-------------+----------+--------+--------------------------+--------+-----+--------------+
only showing top 3 rows
```
<div class="cell text_cell unselected rendered">
<div class="inner_cell">
<div class="text_cell_render rendered_html">
If the rowsBetween() method still smells a bit funky, no worries, it will become clearer in the next example.
</div>
</div>
</div>
<div class="cell text_cell unselected rendered">
<div class="prompt input_prompt"></div>
<div class="inner_cell">
<div class="text_cell_render rendered_html">
What about if we want to understand how much customers spend <i>on average</i> overall/in total (not grouped by product), throughout time? In other words, how does the average spending vary across a given time periodicy - aka: moving Average?
</div>
</div>
</div>
```{python}
conf = SparkConf().setMaster("local[4]").setAppName("window-demo") \
      .set("spark.driver.memory", "4g")
sc = SparkContext(conf=conf)
sqlContext = HiveContext(sc)
```
```{python}
window_02 = Window.partitionBy("customer_name").orderBy("customer_name", "date").rowsBetween(-3, 0)
win_customers02 = win_customers01.withColumn("movingAvg", func.avg(customers02['amount_spent']).over(window_02) )
win_customers02.show()
+-------------+----------+--------+--------------------+--------+-----+------------+--------------+-----------------+
|customer_name|      date|category|        product_name|quantity|price|amount_spent|cumulative_sum|        movingAvg|
+-------------+----------+--------+--------------------+--------+-----+------------+--------------+-----------------+
|         Yann|2016-04-02|       A|             bananas|       3| 55.0|       165.0|         165.0|            165.0|
|         Yann|2016-04-02|       B|                Lamp|       2| 38.0|        76.0|         241.0|            120.5|
|         Yann|2016-04-03|       E|Book: Crime and P...|       5|100.0|       500.0|         741.0|            247.0|
|         Yann|2016-04-13|       E|     Book: The noose|       5|125.0|       625.0|        1366.0|            341.5|
|         Yann|2016-04-22|       B|                Lamp|       1| 38.0|        38.0|        1404.0|           309.75|
|         Yann|2016-04-27|       D|        Solar Pannel|       5| 29.0|       145.0|        1549.0|            327.0|
|         Yann|2016-05-01|       Y|         Motor skate|       1| 68.0|        68.0|        1617.0|            219.0|
|         Yann|2016-05-03|       C|                Rice|      15| 15.0|       225.0|        1842.0|            119.0|
|         Yann|2016-05-03|       D|         Recycle bin|       5| 27.0|       135.0|        1977.0|           143.25|
|         Yann|2016-05-03|       Y|         Motor skate|       1| 68.0|        68.0|        2045.0|            124.0|
|         Yann|2016-05-27|       A|             bananas|       3| 55.0|       165.0|        2210.0|           148.25|
|         Yann|2016-05-27|       D|         Recycle bin|       5| 27.0|       135.0|        2345.0|           125.75|
|         Yann|2016-06-07|       Z|          space ship|       1|227.0|       227.0|        2572.0|           148.75|
|       Yoshua|2016-02-07|       Z|          space ship|       2|227.0|       454.0|         454.0|            454.0|
|       Yoshua|2016-02-14|       A|             bananas|       9| 55.0|       495.0|         949.0|            474.5|
|       Yoshua|2016-02-14|       A|              apples|      10| 55.0|       550.0|        1499.0|499.6666666666667|
|       Yoshua|2016-02-14|       B|                Lamp|       2| 38.0|        76.0|        1575.0|           393.75|
|       Yoshua|2016-03-07|       Z|          space ship|       5|227.0|      1135.0|        2710.0|            564.0|
|       Yoshua|2016-04-07|       A|             bananas|       9| 55.0|       495.0|        3205.0|            564.0|
|       Yoshua|2016-04-07|       C|                Rice|       5| 15.0|        75.0|        3280.0|           445.25|
+-------------+----------+--------+--------------------+--------+-----+------------+--------------+-----------------+
only showing top 20 rows
```
Before explaining how this is working, let us revisit the rowsBetween() method. Note that here we specify to compute between the interval of a maximum of 3 rows behind the current one. Alternatively we could say for example two values behind and two ahead interval:
```{python}
window_03 = Window.partitionBy("customer_name").orderBy("customer_name", "date").rowsBetween(-2, 2)
win_customers03 = win_customers01.withColumn("movingAvg", func.avg(customers02['amount_spent']).over(window_03) )
win_customers03.show(5)
+-------------+----------+--------+--------------------+--------+-----+------------+--------------+---------+
|customer_name|      date|category|        product_name|quantity|price|amount_spent|cumulative_sum|movingAvg|
+-------------+----------+--------+--------------------+--------+-----+------------+--------------+---------+
|         Yann|2016-04-02|       A|             bananas|       3| 55.0|       165.0|         165.0|    247.0|
|         Yann|2016-04-02|       B|                Lamp|       2| 38.0|        76.0|         241.0|    341.5|
|         Yann|2016-04-03|       E|Book: Crime and P...|       5|100.0|       500.0|         741.0|    280.8|
|         Yann|2016-04-13|       E|     Book: The noose|       5|125.0|       625.0|        1366.0|    276.8|
|         Yann|2016-04-22|       B|                Lamp|       1| 38.0|        38.0|        1404.0|    275.2|
+-------------+----------+--------+--------------------+--------+-----+------------+--------------+---------+
only showing top 5 rows
```
The first row (247.0) is simply the current value plus the next two, devided by the total:
(165.0 + 76.0 + 500.0)/3 = 247.0
Simple, right?
Going back to how the computation is partioned, the way we structured this is to compute a moving average per customer but iterating over each event.
<b>However let's say we want to know how the customer spending varies on average across daily/weekly/monthly basis? For that, let's extract those from our date column.</b>
```{python}
win_customers01.printSchema()
root
 |-- customer_name: string (nullable = true)
 |-- date: string (nullable = true)
 |-- category: string (nullable = true)
 |-- product_name: string (nullable = true)
 |-- quantity: long (nullable = true)
 |-- price: double (nullable = true)
 |-- amount_spent: double (nullable = true)
 |-- cumulative_sum: double (nullable = true)
```
Spark automatically infered the type of our date column as being String (as we did not specify the schema when we created the Dataframe). Let's use a UDF to cast it to datetime (using an anonymous function - lambda)
```{python}
from datetime import datetime as dt
# create the general UDF
string_to_datetime = udf(lambda x: dt.strptime(x, '%Y-%m-%d'), DateType())
# Create a new column called datetime, and drop the date column
win_customers01_B = win_customers01.withColumn('datetime', string_to_datetime( win_customers01['date'])).drop('date')
# Add month and Week columns
win_customers01_C = win_customers01_B.withColumn('year', func.year( win_customers01_B['datetime'] )) \
.withColumn('month', func.month( win_customers01_B['datetime'] )) \
.withColumn('week', func.weekofyear( win_customers01_B['datetime']))
win_customers01_C.show(10, truncate=False)
+-------------+--------+--------------------------+--------+-----+------------+--------------+----------+----+-----+----+
|customer_name|category|product_name              |quantity|price|amount_spent|cumulative_sum|datetime  |year|month|week|
+-------------+--------+--------------------------+--------+-----+------------+--------------+----------+----+-----+----+
|Yann         |A       |bananas                   |3       |55.0 |165.0       |165.0         |2016-04-02|2016|4    |13  |
|Yann         |B       |Lamp                      |2       |38.0 |76.0        |241.0         |2016-04-02|2016|4    |13  |
|Yann         |E       |Book: Crime and Punishment|5       |100.0|500.0       |741.0         |2016-04-03|2016|4    |13  |
|Yann         |E       |Book: The noose           |5       |125.0|625.0       |1366.0        |2016-04-13|2016|4    |15  |
|Yann         |B       |Lamp                      |1       |38.0 |38.0        |1404.0        |2016-04-22|2016|4    |16  |
|Yann         |D       |Solar Pannel              |5       |29.0 |145.0       |1549.0        |2016-04-27|2016|4    |17  |
|Yann         |Y       |Motor skate               |1       |68.0 |68.0        |1617.0        |2016-05-01|2016|5    |17  |
|Yann         |C       |Rice                      |15      |15.0 |225.0       |1842.0        |2016-05-03|2016|5    |18  |
|Yann         |D       |Recycle bin               |5       |27.0 |135.0       |1977.0        |2016-05-03|2016|5    |18  |
|Yann         |Y       |Motor skate               |1       |68.0 |68.0        |2045.0        |2016-05-03|2016|5    |18  |
+-------------+--------+--------------------------+--------+-----+------------+--------------+----------+----+-----+----+
only showing top 10 rows
```
Let us group customers by spending:
```{python}
#
customer_grp_by_day = win_customers01_C.groupBy('customer_name', 'datetime', 'year') \
.agg({'amount_spent': 'sum'}) \
.withColumnRenamed('sum(amount_spent)', 'amount_spent') \
.orderBy('customer_name', 'datetime')
customer_grp_by_day.show(20)
+-------------+----------+----+------------+
|customer_name|  datetime|year|amount_spent|
+-------------+----------+----+------------+
|     Geoffrey|2016-04-22|2016|        50.0|
|     Geoffrey|2016-05-03|2016|       330.0|
|     Geoffrey|2016-06-05|2016|       525.0|
|     Geoffrey|2016-06-15|2016|       601.0|
|       Jurgen|2016-05-01|2016|       502.0|
|       Jurgen|2016-05-08|2016|       343.0|
|       Jurgen|2016-06-05|2016|       621.0|
|         Yann|2016-04-02|2016|       241.0|
|         Yann|2016-04-03|2016|       500.0|
|         Yann|2016-04-13|2016|       625.0|
|         Yann|2016-04-22|2016|        38.0|
|         Yann|2016-04-27|2016|       145.0|
|         Yann|2016-05-01|2016|        68.0|
|         Yann|2016-05-03|2016|       428.0|
|         Yann|2016-05-27|2016|       300.0|
|         Yann|2016-06-07|2016|       227.0|
|       Yoshua|2016-02-07|2016|       454.0|
|       Yoshua|2016-02-14|2016|      1121.0|
|       Yoshua|2016-03-07|2016|      1135.0|
|       Yoshua|2016-04-07|2016|       977.0|
+-------------+----------+----+------------+
```
Next, let's check per customer visit (assuming each customer does not visit the store more than once), how much the customer's pending progresses, with a 7 iterations back interval average:
```{python}
window_04 = Window.partitionBy("customer_name").orderBy("customer_name", "datetime").rowsBetween(-7, 0)
win_customers04 = customer_grp_by_day.withColumn("movingAvg", func.avg(customer_grp_by_day['amount_spent']).over(window_04))
win_customers04.show(30)
+-------------+----------+----+------------+------------------+
|customer_name|  datetime|year|amount_spent|         movingAvg|
+-------------+----------+----+------------+------------------+
|         Yann|2016-04-02|2016|       241.0|             241.0|
|         Yann|2016-04-03|2016|       500.0|             370.5|
|         Yann|2016-04-13|2016|       625.0| 455.3333333333333|
|         Yann|2016-04-22|2016|        38.0|             351.0|
|         Yann|2016-04-27|2016|       145.0|             309.8|
|         Yann|2016-05-01|2016|        68.0|             269.5|
|         Yann|2016-05-03|2016|       428.0|292.14285714285717|
|         Yann|2016-05-27|2016|       300.0|           293.125|
|         Yann|2016-06-07|2016|       227.0|           291.375|
|       Yoshua|2016-02-07|2016|       454.0|             454.0|
|       Yoshua|2016-02-14|2016|      1121.0|             787.5|
|       Yoshua|2016-03-07|2016|      1135.0| 903.3333333333334|
|       Yoshua|2016-04-07|2016|       977.0|            921.75|
|     Geoffrey|2016-04-22|2016|        50.0|              50.0|
|     Geoffrey|2016-05-03|2016|       330.0|             190.0|
|     Geoffrey|2016-06-05|2016|       525.0| 301.6666666666667|
|     Geoffrey|2016-06-15|2016|       601.0|             376.5|
|       Jurgen|2016-05-01|2016|       502.0|             502.0|
|       Jurgen|2016-05-08|2016|       343.0|             422.5|
|       Jurgen|2016-06-05|2016|       621.0| 488.6666666666667|
+-------------+----------+----+------------+------------------+
```
Let us group customers by weekly spending:
```{python}
customer_grp_by_week = win_customers01_C.groupBy('customer_name', 'year', 'week') \
.agg({'amount_spent': 'sum'}) \
.withColumnRenamed('sum(amount_spent)', 'amount_spent') \
.orderBy('customer_name', 'week')
customer_grp_by_week.show(20)
+-------------+----+----+------------+
|customer_name|year|week|amount_spent|
+-------------+----+----+------------+
|     Geoffrey|2016|  16|        50.0|
|     Geoffrey|2016|  18|       330.0|
|     Geoffrey|2016|  22|       525.0|
|     Geoffrey|2016|  24|       601.0|
|       Jurgen|2016|  17|       502.0|
|       Jurgen|2016|  18|       343.0|
|       Jurgen|2016|  22|       621.0|
|         Yann|2016|  13|       741.0|
|         Yann|2016|  15|       625.0|
|         Yann|2016|  16|        38.0|
|         Yann|2016|  17|       213.0|
|         Yann|2016|  18|       428.0|
|         Yann|2016|  21|       300.0|
|         Yann|2016|  23|       227.0|
|       Yoshua|2016|   5|       454.0|
|       Yoshua|2016|   6|      1121.0|
|       Yoshua|2016|  10|      1135.0|
|       Yoshua|2016|  14|       977.0|
+-------------+----+----+------------+
```
And computing the weekly moving average:
```{python}
window_05 = Window.partitionBy('customer_name').orderBy('customer_name', 'week', 'year').rowsBetween(-4, 0)
win_customers05 = customer_grp_by_week.withColumn("movingAvg", func.avg(customer_grp_by_week['amount_spent']).over(window_05))
win_customers05.show(30)
+-------------+----+----+------------+-----------------+
|customer_name|year|week|amount_spent|        movingAvg|
+-------------+----+----+------------+-----------------+
|         Yann|2016|  13|       741.0|            741.0|
|         Yann|2016|  15|       625.0|            683.0|
|         Yann|2016|  16|        38.0|            468.0|
|         Yann|2016|  17|       213.0|           404.25|
|         Yann|2016|  18|       428.0|            409.0|
|         Yann|2016|  21|       300.0|            320.8|
|         Yann|2016|  23|       227.0|            241.2|
|       Yoshua|2016|   5|       454.0|            454.0|
|       Yoshua|2016|   6|      1121.0|            787.5|
|       Yoshua|2016|  10|      1135.0|903.3333333333334|
|       Yoshua|2016|  14|       977.0|           921.75|
|     Geoffrey|2016|  16|        50.0|             50.0|
|     Geoffrey|2016|  18|       330.0|            190.0|
|     Geoffrey|2016|  22|       525.0|301.6666666666667|
|     Geoffrey|2016|  24|       601.0|            376.5|
|       Jurgen|2016|  17|       502.0|            502.0|
|       Jurgen|2016|  18|       343.0|            422.5|
|       Jurgen|2016|  22|       621.0|488.6666666666667|
+-------------+----+----+------------+-----------------+
```
Finally, let us move to monthly groupping and calculations.
```{python}
customer_grp_by_month = win_customers01_C.groupBy('customer_name', 'year', 'month')\
.agg({'amount_spent': 'sum'}) \
.withColumnRenamed('sum(amount_spent)', 'amount_spent') \
.orderBy('customer_name', 'month')
customer_grp_by_month.show(20)
+-------------+----+-----+------------+
|customer_name|year|month|amount_spent|
+-------------+----+-----+------------+
|     Geoffrey|2016|    4|        50.0|
|     Geoffrey|2016|    5|       330.0|
|     Geoffrey|2016|    6|      1126.0|
|       Jurgen|2016|    5|       845.0|
|       Jurgen|2016|    6|       621.0|
|         Yann|2016|    4|      1549.0|
|         Yann|2016|    5|       796.0|
|         Yann|2016|    6|       227.0|
|       Yoshua|2016|    2|      1575.0|
|       Yoshua|2016|    3|      1135.0|
|       Yoshua|2016|    4|       977.0|
+-------------+----+-----+------------+
```
```{python}
# This shows how much the customer's pending progresses across months, with a 3 iterations back interval avg
window_06 = Window.partitionBy('customer_name').orderBy('customer_name', 'month', 'year').rowsBetween(-3, 0)
win_customers06 = customer_grp_by_month.withColumn("movingAvg", func.avg(customer_grp_by_month['amount_spent']).over(window_06))
win_customers06.show(30)
+-------------+----+-----+------------+-----------------+
|customer_name|year|month|amount_spent|        movingAvg|
+-------------+----+-----+------------+-----------------+
|         Yann|2016|    4|      1549.0|           1549.0|
|         Yann|2016|    5|       796.0|           1172.5|
|         Yann|2016|    6|       227.0|857.3333333333334|
|       Yoshua|2016|    2|      1575.0|           1575.0|
|       Yoshua|2016|    3|      1135.0|           1355.0|
|       Yoshua|2016|    4|       977.0|           1229.0|
|     Geoffrey|2016|    4|        50.0|             50.0|
|     Geoffrey|2016|    5|       330.0|            190.0|
|     Geoffrey|2016|    6|      1126.0|            502.0|
|       Jurgen|2016|    5|       845.0|            845.0|
|       Jurgen|2016|    6|       621.0|            733.0|
+-------------+----+-----+------------+-----------------+
```
As usual, I suggest further checking these sources:
<div>- <a href="https://databricks.com/blog/2015/07/15/introducing-window-functions-in-spark-sql.html" target="_blank" rel="noopener">Databricks introduction to window functions </a></div>
<div>
<div>- <a href="https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-sql-windows.html" rev="en_rl_minimal">https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-sql-windows.html</a></div>
</div>
&nbsp;
