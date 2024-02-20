# Introduction to Batch Processing
* Batch vs. Streaming
* Types of batch jobs
* Orchestrating batch jobs
* Advantages and disadvantages of batch jobs

Batch: processing a chunk of data at regular intervals. Stream: processing data on the fly.

Types of batch jobs: SQL, Python scripts, Spark, Flink

Advantages: easy to manage, retry, scale; easier to orchestrate
Disadvantages: delay

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

