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

# Installing Spark (Linux)

Connecting to an instance on GCP and installing it there

# First Look at Spark/PySpark
* Reading CSV files
* Partitions
* Saving data to Parquet for local experiments
* Spark master UI

# Spark DataFrames
* Actions vs transformations
* Partitions
* Functions and UDFs

# Spark SQL
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
