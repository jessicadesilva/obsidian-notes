In this workshop, we will be discussing stream processing in SQL with RisingWave. RisingWave aims to build a cost-effective and scalable database solution designed to streamline data processing and query serving.

# Why stream processing?
## Batch vs. Stream (Data Ingestion)

Batch ingestion is when data is coming in hourly/daily/weekly batches and so the data is more static and the load is lower as well. Stream ingestion is when you have dynamic, high volume, high velocity data.

Once the data has been ingested by the upstream source, then for batch it will be ingested into a database. Then the application will run batch queries and the entire batch processing pipeline will run each time we run the query. Examples of this application are PostgreSQL and ClickHouse.

On the other hand, for a stream engine like RisingWave we run things in an incremental fashion. When events come in, they get streamed into our pipeline and each node will calculate the delta (change) in the stream and propagate it throughout to the application. The latency is much lower.

For example, you can use batch or stream to analyze social media data for sentiment analysis. If you want the analysis to be real-time you should use a streaming engine. If you want to monitor the performance of a fleet of IoT devices, you want real-time monitoring so streaming is necessary for low latency.

# What is RisingWave?
RisingWave is a distributed system with 4 key components. For the first component, the user connects to the **frontend** through PGSQL which is called the "Serving Layer". The frontend will take care of parsing, validation, optimization, and query planning. The query plan is then passed to the compute node which lies in the **processing layer**. The processing layer executes the optimized query plan, data ingestion, and data delivery. The **persistent layer** has object storage and since the object storage is lsm3 it needs to have regular compaction with the compactors. The final piece is the **MetaServer** which keeps track of the metadata, scheduling, and monitoring stream jobs.

Here are some characteristics of a stream processing engine and what that looks like in RisingWave:

| **Criteria** | **RisingWave** |
| ---- | ---- |
| Processing Model | Both Stream processing and Batch processing |
| Latency | Low (seconds - milliseconds) |
| Throughput | High |
| Fault tolerance | Exactly-one semantics with checkpoints and snapshots - if a node crashes then if/when it recovers it won't reprocess data so you won't get duplicate data |
| API Support | SQL only |
| Storage system | Internal storage system based on Hummock tiered storage |
| State management | Advanced (supports stateful queries with incremental view maintenance and materialized views) |
# Data Ingestion & Delivery

Some examples of upstream data sources RisingWave can ingest from are mongoDB, Kafka, PostgreSQL, MySQL, etc. Then data can be delivered to a variety of sinks, such as elasticsearch, kafka, ClickHouse, iceberg, etc. For the workshop we will use Kafka and ClickHouse.

Now let's talk about how stateless computations are done with RisingWave. Consider the query below

```SQL
CREATE TABLE t (v1 int)
CREATE MATERIALIZED VIEW m1 AS SELECT v1 + 1 FROM t WHERE v1 > 0; 
```

If you use the keyword EXPLAIN in front of the materialized view then you will get the following query plan (read from bottom to top):

```
StreamMaterialize {
	columns: [v1, t._row_id(hidden)],
	stream_key: [t._row_id],
	pk_columns: [t._row_id],
	pk_conflict: NoCheck
}
|-StreamProject { exprs: [(t.v1 + 1:Int32) as $exprt, t._row_id] }
	|-StreamFilter { predicate: (t.v1 > 0:Int32) }
		|-StreamTableScan { table: t, columns: [v1, _row_id] }
(4 rows)
```

The intermediate steps here, filter and project (+ 1), are stateless since nothing needs to be stored. Now let's say for example we have two data points one with an n value of 0 and another with an n value of 1 that we want to insert into the table t according to this set of operations. The delta here is + since the change is to try to add these values. Since 0 isn't greater than 0, it will be filtered out. So 1 will be added only to the record with n value 1, then it will be materialized (which is a stateful computation). Now instead if we wanted to delete these values from the table t, everything would be the same except the delta here is now - since we want to delete and that would be interpreted in the materialize step.

Okay let's move on to stateful computations, such as the following:

```SQL
CREATE TABLE t (v1 int);
CREATE MATERIALIZED VIEW m2 AS SELECT count(*) FROM t;
```

The count(\*) here is an aggregation function. Let's say the initial value is 0, then since the delta here is + we will update the count to 2 to represent the 2 additional records (0 and 1). Then when we propagate the changes to the materialized view we tell it to remove the value of 0 and add the value of 2 to represent the new total count.

Now let's look at a join:

```SQL
CREATE TABLE t (v1 int, v2 int);
CREATE TABLE t2 (v1 int, v2 int);
CREATE MATERIALIZED VIEW m3 AS
	SELECT t.v1 AS v1, t.v2 AS x2, t2.v2 AS y2
	FROM t JOIN t1 ON t.v1 = t2.v1; 
```

Let's say we have these records coming in:

| Op | V1 | V2 |
| ---- | ---- | ---- |
| + | 0 | 200 |
| + | 1 | 300 |
Then once the table is scanned, there will be a HashJoin (which is stateful with two states, one for the LHS and one for the RHS). The HashJoin will update each state with the information coming in, so if only the LHS has come in then it will see there's nothing to join on the right and nothing will be sent to materialize. But it still keeps record of this in the HashJoin. 

Now let's say we have some records coming in from the right side:

| Op | V1 | V2 |
| ---- | ---- | ---- |
| + | 1 | 500 |
| + | 2 | 600 |
Then when it goes to the HashJoin there is a match on the join predicate. So then the output is propagated to the materialize step.

Now we will move on to the hands-on project.

# Hands-on Project
## Prerequisites
1. Docker and Docker Compose
2. Python 3.7 or later
3. pip and virtualenv for Python
4. psql
5. Clone the repo

Navigate to the project folder and run the following commands:
```bash
% check that you have psql
psql --version
source commands.sh

% docker-compose up
start-cluster

% install python dependencies
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

There are two datasets we will be using (already included in the repo):
* yellow_tripdata_2022-01.parquet
* taxi_zone.csv

We are using RisingWave's DockerCompose file with just slight modifications to have ClickHouse as the downstream system.

So what we will do is replace the timestamp field in the yellow tripdata file with timestamps closer to the current time done with the seed_kafka.py file. The first thing we will do is ingest data into RisingWave using Kafka. The seed_kafka.py file has the logic to process data and populate RisingWave from Kafka. Now we will start the Kafka stream by running this Python file:

```bash
% runs seed_kafka.py
stream-kafka
```
# Homework
## Question 1
```SQL
CREATE MATERIALIZED VIEW taxi_zone_stats AS
WITH trip_time AS (
SELECT
	tpep_dropoff_datetime - tpep_pickup_datetime AS trip_time,
	pulocationid,
	dolocationid
FROM trip_data
)
SELECT
	AVG(trip_time) AS avg_trip_time,
	MIN(trip_time) AS min_trip_time,
	MAX(trip_time) AS max_trip_time,
	tz_pickup.Zone AS pickup_zone,
	tz_dropoff.Zone AS dropoff_zone
FROM trip_time
JOIN taxi_zone AS tz_pickup ON tz_pickup.location_id=trip_time.pulocationid
JOIN taxi_zone AS tz_dropoff ON tz_dropoff.location_id=trip_time.dolocationid
GROUP BY pickup_zone, dropoff_zone;
```

```SQL
SELECT
pickup_zone,
dropoff_zone,
avg_trip_time AS max_avg
FROM taxi_zone_stats
WHERE avg_trip_time=(SELECT MAX(avg_trip_time) FROM taxi_zone_stats);
```

![[Screenshot 2024-03-06 at 8.34.23 PM.png]]
## Question 2
```SQL
WITH max_avg_trip AS (
SELECT
	pickup_zone, dropoff_zone, avg_trip_time
FROM taxi_zone_stats
WHERE avg_trip_time=(SELECT MAX(avg_trip_time) FROM taxi_zone_stats))
SELECT
	COUNT(*),
	max_avg_trip.pickup_zone,
	max_avg_trip.dropoff_zone
FROM max_avg_trip
LEFT JOIN taxi_zone_stats ON taxi_zone_stats.pickup_zone=max_avg_trip.pickup_zone AND taxi_zone_stats.dropoff_zone=max_avg_trip.dropoff_zone
GROUP BY 2, 3;
```
![[Screenshot 2024-03-06 at 8.49.03 PM.png]]

## Question 3
```SQL
WITH latest_pickup_time AS (
SELECT * FROM latest_pickup_time
)
SELECT
	taxi_zone.Zone AS pickup_zone,
	COUNT(*)
FROM trip_data
JOIN taxi_zone ON taxi_zone.location_id=trip_data.pulocationid
WHERE trip_data.
```