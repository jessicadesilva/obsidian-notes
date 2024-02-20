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
* What is Spark
* Why do we need it

Spark is a "general purpose distributed processing engine".
Common use cases: batch-type workloads. Also streaming, but we won't cover it here.

When would you use Spark? For the same things as you'd use SQL for - but for executing the queries on the files in your data lake.

If you can write this in SQL and use Hive/Presto/Athena/BG - do it. But not everything can/should be expressed in SQL.

Common case: ML algorithms. You can't easily use SQL for most of it.

Typical pipeline:
Raw data -> data lake -> some transformation in SQL -> Spark -> Batch job in Python for training a model

Raw data -> data lake -> SQL -> Spark -> Spark for applying the model -> SQL

All orchestrated with Airflow, Prefect, or Mage.

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
