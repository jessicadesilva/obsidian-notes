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
	.withColumnsRenamed('lpep_pickup_datetime', 'pickup_datetime') \
	.withColumnsRenamed('lpep_dropoff_datetime', 'dropoff_datetime')
	
df_yellow = df_yellow \
	.withColumnsRenamed('tpep_pickup_datetime', 'pickup_datetime') \
	.withColumnsRenamed('tpep_dropoff_datetime', 'dropoff_datetime')
```

* Temporary tables
* Some simple queries from week 4

# Joins in Spark
* Merge sort join
* Broadcasting

# RDDs
* From DF to RDD
* map
* reduce
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
