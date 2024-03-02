# Introduction to Batch Processing

**Processing Data**
* **Batch**: Let's say you have a database with data for taxi rides taken on January 15th. Then we have a single job that takes the data from January 15th (00:00 - 23:59) and producing something else. Generally - processing huge chunks of data at one time in regular intervals.
* **Streaming (Week 6)**: Let's say we are in New York and want to hire a taxi. A yellow taxi comes to me and the driver puts something into a computer that sends event information about the ride, e.g., the ride has started, to a data stream. Something then processes the data from the stream and then puts it into another data stream. Generally - processing each event (small amounts of data) as it comes in real-time.

**Types of Batch Jobs**
* Weekly
* More typical: Daily
* More typical: Hourly
* Less typical: 3 x per hour, every 5 minutes

**Technologies for Batch Jobs**
* Python scripts (week 1): can be run anywhere like Kubernetes, AWS Batch, etc.
* SQL (week 4 with DBT)
* Spark (this week)
* Flink

**Typical Batch Processing Workflow**
* Uses some orchestration tool like Airflow, Prefect, or Mage
* Workflow example: Data Lake with CSV files - > Python Script -> SQL job (DBT) -> Spark -> Python

**Advantages of Batch Jobs**
* Easy to manage/orchestrate
* Easy to retry since the workflow typically has a parameter (like time)
* Easy to scale (just bigger machine, bigger cluster)

**Disadvantage of Batch Jobs**
* Delay between jobs
* Delay in processing entire workflow

The majority of data processing jobs (say, 80%) is batch processing.

# Spark Introduction
**What is Spark?**
Apache Spark is a "general purpose distributed processing engine". It's an open-source unified analytics engine for large-scale data processing. Spark pulls data from a data lake into its machines/executers, does something to it, and then saves it somewhere (data lake/warehouse). Processing happens in Spark so that's why it is an engine. It is distributed and so it has clusters where it can split up the work for the jobs. It is multi-language engine meaning it can be used with Java (native way) & Scala. There is a wrapper for Python called **PySpark** (and R and probably others). Typically companies will use Python or Scala or some combination of the two. Spark can be used for both batch jobs and streaming, but we won't cover how to use Spark for streaming.

**When would you use Spark?**
You have a data lake in S3/GCS in Parquet. Spark will pull the data out of the data lake, process it similarly as you would in SQL, and then output it into another data lake or data warehouse. Sometimes it is not easy to use SQL if you have many files in your data lake, there are ways around it though so that you can still use SQL (using Hive, Presto, Athena). But maybe your job is too difficult to do with SQL (lots of unit tests, lots of modules) then use Spark instead. This often can be machine learning tasks (like training and testing).

Typical pipeline all orchestrated with Airflow, Prefect, or Mage:
Raw data -> data lake -> some transformation in SQL (Presto, Hive) -> Spark -> Batch job in Python for training a model

Raw data -> data lake -> SQL -> Spark -> Spark for applying the model -> SQL

**Key recommendation:** Use SQL when you can, use Spark when what you want to do cannot be expressed with SQL.

# Installing Spark/PySpark

```bash
brew install openjdk
```

Then there are two commands we need to run:
(1) To use wrappers like pyspark:
```bash
sudo ln -sfn /opt/homebrew/opt/openjdk/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk.jdk
```

(2) assuming we are working in a virtual environment
```bash
export PATH="/opt/homebrew/opt/openjdk/bin:$PATH"
```

Then we install pyspark (which also installs Spark):

```bash
pip install pyspark
```

And we should be good to go!
# First Look at Spark/PySpark

We will start our Jupyter notebook with the following to connect to our Spark instance:

```python
import pyspark
from pyspark.sql import SparkSession

spark = SparkSession.builder \
		 .master("local[*]") \
		 .appName('test') \
		 .getOrCreate()
```

The spark object we created is how we will communicate with Spark. You can access the Spark UI at localhost:4040 or, if that's in use, port 4041. That will look like this:

![[Screenshot 2024-02-20 at 6.21.34 PM.png]]
Let's download the High Volume For-Hire Vehicle Trip Records using wget (high volume since spark is mean to be used for high volume data):

```python
!wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/fhvhv/fhvhv_tripdata_2021-01.csv.gz
```

Let's now read this data as a dataframe using Spark:

```python
df = spark.read \
	.option("header", "true") \
	.csv('fhvhv_tripdata_2021-01.csv.gz')

df.show()
```

And we see in the output that the column headers were read correctly:
![[Screenshot 2024-02-20 at 6.32.52 PM.png]]

When we refresh the Spark UI, we see the two jobs (loading to dataframe and showing the dataframe):
![[Screenshot 2024-02-20 at 6.33.29 PM.png]]
Unlike pandas, Spark does not try to infer the types of the fields. We can see that by looking at the first five records and noting that the datetime fields are read as strings:
```python
df.head(5)
```
![[Screenshot 2024-02-20 at 6.35.09 PM.png]]

There is also a schema attribute for the dataframe class which is showing us that every field is interpreted as a string:
```python
df.schema
```
![[Screenshot 2024-02-20 at 6.37.13 PM.png]]

In week 1, we used pandas to infer the types to create a schema in our local database. However, pandas may not like having a 700+MB dataframe, so we will apply this method with just a small bit of this data. First let's unzip our dataset:

```python
!gunzip fhvhv_tripdata_2021-01.csv.gz
```

We can use the head command along with the parameter -n to only look at the first 101 rows of the dataset:

```python
!head -n 101 fhvhv_tripdata_2021-01.csv > head.csv
```

We can then use the wc bash command (standing for word count) with the flag -l for line count to check the number of rows head.csv has:

```python
!wc -l head.csv
```
![[Screenshot 2024-02-20 at 6.42.22 PM.png]]
Update head.csv to instead contain the first 1001 rows of the dataset. Let's load the data into a pandas dataframe and check out the schema:

```python
import pandas as pd
df_pandas = pd.read_csv('head.csv')

df_pandas.dtypes
```
![[Screenshot 2024-02-20 at 6.54.31 PM.png]]

Pandas does help us a bit, but it also can't infer the two datetime/timestamp fields.

We can now use Spark to take in this pandas dataframe and create a Spark dataframe from it.
```python
spark.createDataFrame(df_pandas)
```

If you take a look at the Spark dataframe schema, you'll see that it has inheritied the datatypes from pandas:

```python
spark.createDataFrame(df_pandas).schema
```
![[Screenshot 2024-02-20 at 6.57.54 PM.png]]

Since the two int64 columns were converted to longtype (which takes up 8 bytes) let's convert it to integer (taking up only 4 bytes) to be a little more efficient. If we copy the output there (written in Scala) and make the appropriate edits for it to the be interpreted with Python as well as the edits to the datatypes, we will have the following:

```python
StructType([
		   StructField('hvfhs_license_num', StringType(), True),
		   StructField('dispatching_base_num', StringType(), True),
		   StructField('pickup_datetime', TimestampType(), True),
		   StructField('dropoff_datetime', TimestampType(), True),
		   StructField('PULocationID', IntegerType(), True),
		   StructField('DOLocationID', IntegerType(), True),
		   StructField('SR_Flag', StringType(), True)
		   ])
```
We aren't completely sure what the SR_Flag field is for, it seems to be null everywhere and so that is interpreted as double. We changed it to StringType just in case and that is nullable so we won't run into any issues.
The list was turned into a Python list using square brackets. Now we need to import a package to be able to use these types, namely:

```python
from pyspark.sql import types
```

and thus we have to precede all our types with ```types.```:

```python
schema = types.StructType([
	types.StructField('hvfhs_license_num', types.StringType(), True),
	types.StructField('dispatching_base_num', types.StringType(), True),
	types.StructField('pickup_datetime', types.TimestampType(), True),
	types.StructField('dropoff_datetime', types.TimestampType(), True),
	types.StructField('PULocationID', types.IntegerType(), True),
	types.StructField('DOLocationID', types.IntegerType(), True),
	types.StructField('SR_Flag', types.StringType(), True)
])
```

Now that we have defined the schema, we can enforce it when we read the (large) CSV file:

```python
df = spark.read \
	.option("header", "true") \
	.schema(schema) \
	.csv('fhvhv_tripdata_2021-01.csv')
```

Let's see what we have:

```python
df.show()
```
![[Screenshot 2024-02-20 at 7.13.18 PM.png]]

```python
df.head(10)
```
![[Screenshot 2024-02-20 at 7.13.46 PM.png]]

Looks like datetime is properly parsed, great!

Now let's save this CSV to a parquet file. However, it's not great to just have one large file. To understand why this is, let's take a brief detour to look at the internals of Spark.

**Spark Internals**
The Spark cluster has a number of executors, that is, machines that load the data from a data lake and do some computation. The way they work is if we have a bunch of files (say from Google Cloud Storage) then files will be distributed to the executors one file per one executor. If there are leftover files, then once an executor is done with their first file they will pick up a file that hasn't been processed yet. If there is only one file, then only one executor can be used. This isn't optimal since the other executors will just be idle! So we want to break this large CSV into multiple files (called partitions). To do that, we can use the repartition method:

```python
# argument is number of partitions
df.repartition(24)
```

This command is a lazy command, though, as it only does something when the dataframe is being saved somewhere. For example, if we have 

```python
df = df.repartition(24)
```

Then df will behave just like it did before, except when it is being exported and saved somewhere else. For example, if we do this:

```python
df.write.parquet('fhvhv/2021/01/')
```

We can see the files saved in our directory:
![[Screenshot 2024-02-20 at 7.25.36 PM.png]]
Once you see the SUCCESS file (which is empty) you know the job is done.

In the Spark UI we could see the job there as well:
![[Screenshot 2024-02-20 at 7.26.49 PM.png]]

Using the du bash command we can see that the compressed files take up less space compared to the original CSV file:

```bash
du -h fhvhv
```

![[Screenshot 2024-02-20 at 7.28.59 PM.png]]

One reason for this is that the parquet files know the schema and so they are  more efficient with space for the specific datatypes.
# Spark DataFrames
We can easily read in the partitioned data in the form of parquet files in the following way:
```python
df = spark.read.parquet('fhvhv/2021/01/')
```
Note that the schema is read in as well:
```python
df
```
![[Screenshot 2024-02-20 at 7.34.26 PM.png]]
Here is a pretty way to view the schema:
```python
df.printSchema()
```

![[Screenshot 2024-02-20 at 7.35.43 PM.png]]

We can select just a few columns from the dataframe using select (like SQL):

```python
df.select('pickup_datetime', 'dropoff_datetime', 'PULocationID', 'DOLocationID')
```

We can do even more by filtering this dataframe:
```python
df.select('pickup_datetime', 'dropoff_datetime', 'PULocationID', 'DOLocationID') \
.filter(df.hvfhs_license_num == 'HV0003')
```

Now Spark hasn't done anything yet, because we haven't really asked it to do anything (read or write). But we can do that now:

```python
df.select('pickup_datetime', 'dropoff_datetime', 'PULocationID', 'DOLocationID') \
.filter(df.hvfhs_license_num == 'HV0003') \
.show()
```
![[Screenshot 2024-02-20 at 7.41.19 PM.png]]

And here is the job in the Spark UI:
![[Screenshot 2024-02-20 at 7.41.49 PM.png]]

This "laziness" characterizes the difference between actions and transformations.

## Actions vs. Transformations
**Transformations** are lazy, not executed right away.
* Selecting columns
* Filtering
* Applying functions to each column
* Repartitioning
* Joins
* Group by

**Actions** are eager and it triggers itself and all the dependent transformations to be executed immediately.
* Show/take/head
* Write

Now a lot of the transformations we discussed above can be done using SQL. But Spark is more flexible and has really great UDF (user defined functions). First, let's talk about the functions that Spark has already:

```python
from pyspqrk.sql import functions as F
```

Then when you do the following you can see the many options for functions already defined in Spark:

```python
F.
```

![[Screenshot 2024-02-20 at 7.49.16 PM.png]]

Let's use the to_date function which takes in a datetime and keeps only the date. The way these functions work, we can use the withColumn method on the dataframe which adds a new column to the dataframe (transformation). We give it the new column name and the transformation we want to do to get that new column. Note if you give it a name of a pre-existing column, it will override that column.

```python
df \
	.withColumn('pickup_date', F.to_date(df.pickup_datetime)) \
	.withColumn('dropoff_date', F.to_date(df.dropoff_datetime)) \
	.select('pickup_date', 'dropoff_date', 'PULocationID', 'DOLocationID') \
	.show()
```

Although there is an extensive list of functions already, we can also define our own functions. Let's go ahead and define a function on the dispatching_base_num column that isn't so easy to recreate with SQL:

```python
def crazy_stuff(base_num):
	num = int(base_num[1:])
	if num % 7 == 0:
		return f's/{num:03x}'
	elif num % 3 == 0:
		return f'a/{num:03x}'
	else:
		return f'e/{num:03x}'
```

Let's test it:
```python
crazy_stuff('B02884')
```
![[Screenshot 2024-02-20 at 7.59.41 PM.png]]
The idea of "testing" is exactly why user-defined functions are so much easier to use in Spark than databases because it is just Python and so we can cover it with tests to make sure it works.

Let's give our function a name, we need to say what the return type is:
```python
crazy_stuff_udf = F.udf(crazy_stuff, returnType=types.StringType())
```

Let's add this to our sequence of transformations:
```python
df \
	.withColumn('pickup_date', F.to_date(df.pickup_datetime)) \
	.withColumn('dropoff_date', F.to_date(df.dropoff_datetime)) \
	.withColumn('base_id', crazy_stuff_udf(df.dispatching_base_num)) \
	.select('base_id', 'pickup_date', 'dropoff_date', 'PULocationID', 'DOLocationID') \
	.show()
```
![[Screenshot 2024-02-20 at 8.06.21 PM.png]]

# Preparing the Data
We'll create a bash script to download all of the yellow and green taxi data for 2020 - 2021. When you create a bash script, make sure to change the mode to executable before trying to run it using the following command:

```bash
chmod +x download_data.sh
```

Here is our URL:

https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2019-01.csv.gz

We need to format our data to have a leading 0, we can do this using the following syntax: %02d where 0 means leading 0, 2d means 2 digits. Here is our bash script to download the data:

```bash
# quit on the first nonzero code
set -e

# make configurable
TAXI_TYPE=$1 # yellow
YEAR=$2 # 2020

# https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2019-01.csv.gz
URL_PREFIX="https://github.com/DataTalksClub/nyc-tlc-data/releases/download"

for MONTH in {1..12}; do
	FMONTH=`printf "%02d" ${MONTH}`
	URL="${URL_PREFIX}/${TAXI_TYPE}/${TAXI_TYPE}_tripdata_${YEAR}-${FMONTH}.csv.gz"
	LOCAL_PREFIX="data/raw/${TAXI_TYPE}/${YEAR}/${FMONTH}"
	LOCAL_FILE="${TAXI_TYPE}_tripdata_${YEAR}_${FMONTH}.csv.gz"
	
	LOCAL_PATH="${LOCAL_PREFIX}/${LOCAL_FILE}"
	
	mkdir -p ${LOCAL_PREFIX}
	wget ${URL} -O ${LOCAL_PATH}

done
```

We can run the command below using yellow/green and 2020/2021.

```bash
./download_data.sh yellow 2020
```

Two directories will be made for August 2021 with empty csv.gz files for both green and yellow, go ahead and remove those.

Apply the same methods as in the previous video to define the schema for the table and save as partitioned parquet files.

# Spark SQL

Start the Spark session as we have done before and load the yellow and green taxi data from the parquet files into a Spark dataframe:

```python
df_green = spark.read.parquet('data/pq/green/*/*')
df_yellow = spark.read.parquet('data/pq/yellow/*/*')
```

When we inspect the schema using the printSchema() method we see that they are similar but not exactly the same:
![[Screenshot 2024-02-21 at 6.22.43 PM.png]]
So we will combine the two dataframes only using the fields that they both have. We can inspect the columns of a spark dataframe using the columns attribute and then find their intersection:

```python
set(df_green.columns) & set(df_yellow.columns)
```
![[Screenshot 2024-02-21 at 6.25.22 PM.png]]
We notice that pickup and dropoff times don't appear here, that is because the yellow and green taxi datasets use tpep and lpep respectively as prefixes. Let's rename those columns by removing their prefixes:

```python
df_green = df_green \
	.withColumnRenamed('lpep_pickup_datetime', 'pickup_datetime') \
	.withColumnRenamed('lpep_dropoff_datetime', 'dropoff_datetime')
	
df_yellow = df_yellow \
	.withColumnRenamed('tpep_pickup_datetime', 'pickup_datetime') \
	.withColumnRenamed('tpep_dropoff_datetime', 'dropoff_datetime')
```

In order to preserve the ordering of the columns, we won't use the set notation, instead a list comprehension.

```python
common_cols = [col for col in df_green.columns if col in df_yellow.columns]
```

Now as we select only these common columns, we want to include an extra column that says whether it was green or yellow taxi data:

```python
from pyspark.sql import functions as F

df_green_sel = df_green \
	.select(common_cols) \
	.withColumn('service_type', F.lit('green'))

df_yellow_sel = df_yellow \
	.select(common_cols) \
	.withColumn('service_type', F.lit('yellow'))
```

Now let's union these two dataframes:

```python
df_trips_data = df_green_sel.unionAll(df_yellow_sel)
```

And see that the data was transformed correctly:

```python
df_trips_data.groupBy('service_type').count().show()
```
![[Screenshot 2024-02-21 at 6.42.03 PM.png]]

Great, it works!

Now let's start creating SQL statements with Spark. To do that, we need to create temporary views of our Spark dataframe for it to access:

```python
df_trips_data.createOrReplaceTempView('trips_data')
```

Now we can refer to the table "trips_data" in our SQL queries:

```python
spark.sql("""
SELECT
	service_type,
	count(1)
FROM
	trips_data
GROUP BY
	service_type
""").show()
```
![[Screenshot 2024-02-21 at 6.50.08 PM.png]]
Now let's execute some of the queries from Week 4.

```python
df_result = spark.sql("""
SELECT
	PULocationID AS revenue_zone,
	date_trunc('month', pickup_datetime) AS revenue_month,
	service_type,
	SUM(fare_amount) AS revenue_monthly_fare,
	SUM(extra) AS revenue_monthly_extra,
	SUM(mta_tax) AS revenue_monthly_mta_tax,
	SUM(tip_amount) AS revenue_monthly_tip_amount,
	SUM(tolls_amount) AS revenue_monthly_tolls_amount,
	SUM(improvement_surcharge) AS revenue_monthly_improvement_surcharge,
	SUM(total_amount) AS revenue_monthly_total_amount,
	SUM(congestion_surcharge) AS revenue_monthly_congestion_surcharge,
	AVG(passenger_count) AS avg_monthly_passenger_count,
	AVG(trip_distance) AS avg_monthly_trip_distance

FROM
	trips_data
GROUP BY
	1, 2, 3
""")

df_result.show()
```
![[Screenshot 2024-02-21 at 7.10.23 PM.png]]

Let's go ahead and write our results:

```python
df_result.write.parquet('data/report/revenue/')
```

Here is the DAG visualization of the job:

![[Screenshot 2024-02-21 at 7.12.33 PM.png]]
If we want to combine parquet files so there aren't as many of them, we can do the opposite of partitioning using the coalesce method:

```python
df_result.coalesce(1).write.parquet('data/report/revenue/', mode='overwrite')
```

# Spark Cluster

* Driver sends job instructions to Master
* Master communicates with executors and tells them what to do
* Executors download data from Cloud Storage (S3 or GCS) and execute the Spark jobs

*Note:* The notation ```master("local[*]")``` means we are creating a local cluster

We have a package with Spark code written in Python, Scala, etc. We then submit the package to the Master (think of it as the entry-point to the Spark cluster) using the ```spark-submit``` command. We can specify some information like what kind of resources we need for this job. Then the Master coordinates between executors by sending them instructions on what to do. Master will know if an executor goes away and sends assigns the job at that executor to some other executor. The executor first needs to pull data from the Spark DataFrame (living in Cloud Storage like S3 or GCS, historically Hadoop/HDFS) which is partitioned (just parquet files). When we submit a job to Master then executors will pull individual parquet files in to process. When Hadoop is used, the files are actually stored in the executors with some redundancy in case a node/executor goes away. In this way, data is local and they only need to download the code. This made a lot of sense since the files can be quite large but the code is relatively small. But these days, since we have S3 and GCS those are in the same Data Center as the cluster and it is fast to download/export data.

# Groupby in Spark
Let's load in our green taxi data in a new Spark Session:

```python
import pyspark
from pyspark.sql import SparkSession

spark = SparkSession.builder \
	.master("local[*]") \
	.appName('test') \
	.getOrCreate()
```

```python
df_green = spark.read.parquet('data/pq/green/*/*')
df_green.createOrReplaceTempView('green')
```

Now we will create a query that gives us the total amounts and number of records for each hour and each zone using groupby:

```python
df_green_revenue = spark.sql("""
	SELECT
		date_trunc('hour', lpep_pickup_datetime) AS hour,
		PULocationID AS zone,
		SUM(total_amount) AS amount,
		COUNT(1) AS number_records

	FROM
		green
	WHERE
		lpep_pickup_datetime >= '2020-01-01 00:00:00'
	
	GROUP BY
		1, 2
	
	ORDER BY
		1, 2
""")
```

Now we can save the DataFrame into partitioned parquet files:

```python
df_green_revenue.write.parquet('data/report/revenue/green')
```

And on localhost:4040 we can see what's happening in the cluster when we run this job.
![[Screenshot 2024-02-27 at 8.00.22 PM.png]]
The first stage prepares the data for Group By, the second stage does the Group By, and the third stage is Order By.

Now let's see how Spark executes a query like this.

Each executor pulls in a part from the partition. In Stage 1, each executor will do the filtering and groupby for the parquet file it takes in. Now for each partition we have temporary files for sub-results. That completes the first stage.
In Stage 2, we need to combine these sub-result files together. This is called **reshuffling** because it shuffles the records we have in each part of the partition and move them between each other. As a result, all the records with the same key should end up in the same part of the partition. It uses the External Merge Sort algorithm to do this reshuffling. Now we have some number of parts of a partition but within a part any distinct key that appears, all of the records pertaining to that key are within that same of the partition. Then we apply Group By again to obtain one record for each key across all parts of the partition (but one part will have multiple distinct keys).

When Spark executes the Order By command, it will do reshuffling to make sure the results are ordered. Let's remove the Order by in the query for green taxi data and repeat the process for yellow taxi.

Let's repartition them into 20 parts since the data is relatively small:

```python
df_green_revenue \
	.repartition(20) \
	.write.parquet('data/report/revenue/yellow', mode='overwrite')
```
![[Screenshot 2024-02-27 at 8.16.16 PM.png]]

Now we see a third stage that will do the repartitioning. We can see the amount of data that is shuffled under **Shuffle Read** and we want to make sure this stays small.

![[Screenshot 2024-02-27 at 8.17.12 PM.png]]

We can look at the size of the files in our command line using this command:
```bash
ls -lhR report/revenue
```

R is for recursive. And we will see the yellow data is 15MB and green data is 6.2MB.
# Joins in Spark
We will talk about two types of joins: (1) when the tables are approximately the same size and (2) when one table is much smaller than the other. We will also talk about the External Merge Sort algorithm since that's how joins work in Spark.

Looking aback at our df_green_revenue and df_yellow_revenue Spark Dataframes, they both have an hour column and a zone column. When we join on those two columns, we will end up with size columns: 2 coming from the joined hour and zone columns, 2 coming from revenue for each of green and yellow, and 2 coming from number of trips for each of green and yellow taxis. Let's first rename the revenue and number of trips columns for green and yellow taxi data so that it indicates which of the two it corresponds to:

```python
df_green_revenue_temp = df_green_revenue \
	.withColumnRenamed('amount', 'green_amount') \
	.withColumnRenamed('number_records', 'green_number_records')

df_yellow_revenue_temp = df_yellow_revenue \
	.withColumnRenamed('amount', 'yellow_amount') \
	.withColumnRenamed('number_records', 'yellow_number_records')
```

Now we can (outer) join the two tables along the matching columns.

```python
df_join = df_green_revenue_temp.join(df_yellow_revenue_temp, on=['hour','zone'], how='outer')
```

Here is our visual of what Spark is doing to complete this job:

![[Screenshot 2024-02-28 at 7.26.53 PM.png]]
In the first two stages we are reading in the green and yellow taxi data respectively and doing the Group By.

For the join, it will first take each record and convert it into a key/record pair where the key is, in this case, a combined key of the Hour and Zone. Then it will use reshuffling to put all the records with a given key in the same part of the partition. Then we take two records with matching primary keys and convert them into one record with the combined information. If there is only one record for a given key, since we use an outer join there will be a record corresponding to that key with null values in certain fields. This is called the MergeSortJoin algorithm.

MergeSortJoin is used when the two tables are approximately the same size. When they are very different in sizes, it will use a different approach. Let's create a (small) DataFrame corresponding to the zone data:

```python
df_zones = spark.read.parquet('zones/*')
```

We can now join this DataFrame with our previous one:

```python
df_result = df_join.join(df_zones, df_join.zone == df_zones.LocationID)
```

Finally we will write the result with duplicate columns removed:

```python
df_result.drop('LocationID', 'zone').write.parquet('tmp/revenue-zones')
```

Here is what it looks like in Spark (very similar to before):

![[Screenshot 2024-02-28 at 7.40.36 PM.png]]

But we see that actually some actions were taken prior to this, in particular:
![[Screenshot 2024-02-28 at 7.50.51 PM.png]]
Which is doing a broadcast exchange:
![[Screenshot 2024-02-28 at 7.51.05 PM.png]]
When we have one dataframe that is small compared to the other that will be joined, the executors will be sent a part in the partition of the big dataframe and a copy of the entire small dataframe (that's the broadcasting). Then the join happens within the executor and no shuffling is needed, which is much faster.
# Operations on Spark RDDs
RDD stands for resilient distributed dataset. It is the basis of what Spark uses for distributed computations. The datasets we have been using are built upon an RDD, so it gives us an added layer of abstraction so we don't actually need to use RDD. In this section we will talk about simple operations (map and reduce) and what RDDs are. In the next section we will talk about mapPartition.

An RDD is a distributed dataset (collection of objects) whereas a DataFrame is this but also it has a schema. If you have a DataFrame, you can access the RDD it is built off of with the following command:

```python
df_green.rdd
```
This gives us the following result:
![[Screenshot 2024-03-01 at 3.52.09 PM.png]]
The take method on a DataFrame gives us a list of the first five "row objects" which are what make up the corresponding RDD:
```python
df_green.take(5)
```
![[Screenshot 2024-03-01 at 3.54.05 PM.png]]

Our goal here is to recreate this SQL query using RDD operations:
```SQL
	SELECT
		date_trunc('hour', lpep_pickup_datetime) AS hour,
		PULocationID AS zone,
		SUM(total_amount) AS amount,
		COUNT(1) AS number_records

	FROM
		green
	WHERE
		lpep_pickup_datetime >= '2020-01-01 00:00:00'
	
	GROUP BY
		1, 2
```

Let's focus on the RDD of just some columns from this dataframe:
```python
rdd = df_green \
	.select('lpep_pickup_datetime','PULocationID','total_amount') \
	.rdd
```
We can check that the RDD is correct by inspecting 5 rows with the take method.

We can implement a where clause on the RDD using the filter method with a corresponding function. In this example, we will have a where clause that depends on a datetime field, so let's import corresponding python packages:

 ```python
 from datetime import datetime

# for example
start = datetime(year=2020, month=1, day=1)
```
We can create a function to check that the lpep_pickup_datetime is at least the date indicated above to remove outliers:

```python
def filter_outliers(row):
	return row.lpep_pickup_datetime >= start
```
We want this function to return a boolean value in order to use the filter method as a where clause:
```python
rdd \
	.filter(filter_outliers) \
	.take(1)
```
And this is our result:
![[Screenshot 2024-03-01 at 4.05.51 PM.png]]

Filter returns either true or false and it is used to discard records.

In order to do the GroupBy operation, we will first need to create a pair for each row of the form (key, value). In our example of GroupBy before, we grouped by the pickup hour and zone, then selected columns giving the sum of the total amounts and the number of records per key. To create this key-value pair, we will use **map** which applies a function to every element (row) of the RDD and it gives us something else as a result. Let's create our function to return the key-value pair for a given row:
```python
def prepare_for_grouping(row):
	# key is hour and zone
	hour = row.lpep_pickup_datetime.replace(minute=0, second=0, microsecond=0)
	zone = row.PULocationID
	key = (hour, zone)

	# value is sum of amounts
	amount = row.total_amount
	# to count number of records
	count = 1
	value = (amount, count)

	return (key, value)
```
Let's see what we get when we apply this map:

```python
rdd \
	.filter(filter_outliers) \
	.map(prepare_for_grouping) \
	.take(10)
```
![[Screenshot 2024-03-01 at 4.19.05 PM.png]]

So it seems to be working. Now we want to add these things together using **reduce**, or **reduceByKey** function. What it does is it takes elements of a group with the same key and reduces it to only one record according to some input function. Let's say the rows with a specific key have corresponding values value_0, value_1, value_2, etc. This function will first be applied to value_0 and value_1 and it will give a result which is the same number of fields as value_0&1. This output along with value_2 is then fed into the function again, and this repeats until all values are fed into the function. Here is the function that can give us the sum of the amounts and total number of records from this procedure:
```python
	def calculate_revenue(left_value, right_value):
		# in our example the value is amount, 1
		left_amount, left_count = left_value
		right_amount, right_count = right_value

		output_amount = left_amount + right_amount
		output_count = left_count + right_count

		return (output_amount, output_count)
```
Then we can feed this function into the reduceByKey method to complete our GroupBy operation:

```python
rdd \
	.filter(filter_outliers) \
	.map(prepare_for_grouping) \
	.reduceByKey(calculate_revenue) \
	.take(10)
```
![[Screenshot 2024-03-01 at 4.34.50 PM.png]]
No we want to unnest these pairs and turn it into a dataframe. First, let's define the column names using namedtuple data structure:

```python
from collections import namedtuple

RevenueRow = namedtuple('RevenueRow', ['hour', 'zone', 'revenue', 'count'])
```
Then create a function to take the two tuples and return a single named tuple.

```python
def unwrap(row):
	return RevenueRow(
	hour=row[0][0],
	zone=row[0][1],
	revenue=row[1][0],
	count=row[1][1]
	)
```
We can apply this function using the map method and then turn it back into a dataframe using the toDF method:

```python
rdd \
	.filter(filter_outliers) \
	.map(prepare_for_grouping) \
	.reduceByKey(calculate_revenue) \
	.map(unwrap) \
	.toDF() \
	.show()
```
We see that the columns were interpreted correctly:

![[Screenshot 2024-03-01 at 4.45.45 PM.png]]

Let's save our result:

```python
df_result = rdd \
	.filter(filter_outliers) \
	.map(prepare_for_grouping) \
	.reduceByKey(calculate_revenue) \
	.map(unwrap) \
	.toDF() 
```
This takes a long time because it is trying to figure out the schema. Now we can adjust the schema as we did before:
```python
df_result.schema
```
```python
from pyspark.sql import types

schema = types.StructType([
types.StructField('hour', types.TimestampType(), True),
types.StructField('zone', types.IntegerType(), True),
types.StructField('revenue', types.DoubleType(), True),
types.StructField('count', types.IntegerType(), True)
])
```
Now when we save result using the predefined schema in the toDF method it won't take any time unless we want to show the result.

```python
df_result = rdd \
	.filter(filter_outliers) \
	.map(prepare_for_grouping) \
	.reduceByKey(calculate_revenue) \
	.map(unwrap) \
	.toDF(schema)
```

Let's save the result and see what's happening in Spark:
```python
df_result.write.parquet('tmp/green-revenue')
```
![[Screenshot 2024-03-01 at 4.59.21 PM.png]]

We see it is in two stages just like the SQL example. The two stages here are because the reduceByKey is doing reshuffling to make sure rows with the same key end up in the same part of the partition.

Now let's talk about the mapPartitions operation on RDDs. It is similar to map, where now it takes in an RDD (instead of a row) and returns an RDD. By RDD in this case we mean a partition. So this operation is applied to parts of a partition independently. One example of this would be if we had our machine learning model as a function fed into mapPartition and it will chunk the large dataset into pieces and apply the model to each piece separately (in parallel). Then it will combine results and upload it to a data lake.


* mapPartition
* From RDD to DF

# Spark Internals
* Driver, master, and executors
* Partitioning + coalesce
* Shuffling
* Group by or not group by?
* Broadcasting

# Spark and Docker
* TBD
# Running Spark in the Cloud (GCP)
* https://cloud.google.com/solutions/spark

# Connecting Spark to a DWH
* Spark with BigQuery (Athena/presto/hive/etc similar)
* Reading from GCP and saving to BG
