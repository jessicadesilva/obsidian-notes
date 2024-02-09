# OLTP vs. OLAP

OLTP: Online transaction processing
* Used in back-end services
* Group SQL queries together and fall/roll-back if one fails
* Updates are fast, but small
OLAP: Online analytical processing
* Used for discovering hidden insights in a lot of data
* Analytical purposes for data analysts and scientists
* Data is periodically refreshed and data size is larger in comparison to OLTP

|  | **OLTP** | **OLAP** |
| ---- | ---- | ---- |
| **Purpose** | Control and run essential business operations in real time | Plan, solve problems, support decisions, discover hidden insights |
| **Data updates** | Short, fast updates initiated by user | Data periodically refreshed with scheduled, long-running batch jobs |
| **Database design** | Normalized databases for efficiency | Denormalized databases for analysis |
| **Space requirements** | Generally small if historical data is archived | Generally large due to aggregating large datasets |
| **Backup and recovery** | Regular backups required to ensure business continuity and meet legal and governance requirements | Lost data can be reloaded from OLTP database as needed in lieu of regular backups |
| **Productivity** | Increases productivity of end users | Increases productivity of business managers, data analysts, and executives |
| **Data view** | Lists day-to-day business transactions | Multi-dimensional view of enterprise data |
| **User examples** | Customer-facing personnel, clerks, online shoppers | Knowledge workers such as data analysts, business analysts, and executives |

# What is a data warehouse

A data warehouse is an **OLAP** solution used for reporting and data analysis. It generally consists of raw data, meta data, and summary data. Data warehouses have many sources, such as operating systems, flat files systems, OLTP databases, which report to a staging area that then writes to a data warehouse. Data warehouses can be transformed into a data mart (such as one for purchasing, another for sales) and the data marts are accessed by end-users (analysts, data scientists, etc.). However, it may make sense in some situations for end-users to pull data directly from the data warehouse.

![[Screenshot 2024-02-07 at 7.35.04 PM.png]]

# BigQuery

* Serverless data warehouse
	* There are no servers to manage or database software to install
* Software as well as infrastructure with these two things in mind:
	* scalability
	* high availability
* Built-in features like
	* machine learning
	* geospatial analysis
	* business intelligence
* BigQuery maximizes flexibility by separating the compute engine that analyzes your data from your storage

## BigQuery interface
On the left side, you can see your project which contains a folder for the schema/data and within that folder, tables.
![[Screenshot 2024-02-07 at 7.42.51 PM.png]]

By default, BigQuery caches queries and so it may be good to turn caching off for consistent results. You can do that in a SQL query by going to MORE then Query Settings and uncheck Use cached results.

BigQuery provides open-source public datasets. If you search for a public dataset be sure to click SEARCH ALL PROJECTS for them to appear.

![[Screenshot 2024-02-07 at 7.46.05 PM.png]]

When you view a table, the SCHEMA tab shows us the columns, data types, and description if provided.

![[Screenshot 2024-02-07 at 7.47.29 PM.png]]

If you query the table via the QUERY tab, it will prepopulate the SQL query with the reference to the corresponding table.
![[Screenshot 2024-02-07 at 7.48.20 PM.png]]

In the query:

![[Screenshot 2024-02-07 at 7.49.30 PM.png]]

Let's run the following query on this table:

```sql
SELECT station_id, name
FROM `bigquery-public-data.new_york_citibike.citibike_stations`
LIMIT 100
```

Using the RUN button.
![[Screenshot 2024-02-07 at 7.50.54 PM.png]]

We see the query results at the bottom with options to export the data or explore it with Sheets, Looker Studio, etc.

![[Screenshot 2024-02-07 at 7.52.55 PM.png]]

## BigQuery Costs

For this section, we will start by (manually) uploading the following files from https://github.com/DataTalksClub/nyc-tlc-data/releases/tag/yellow/download to our GCS mage-zoomcamp-jessica-desilva bucket:
* yellow_tripdata_2019-01.csv
* yellow_tripdata_2019-02.csv
* yellow_tripdata_2019-03.csv
* yellow_tripdata_2020-01.csv
* yellow_tripdata_2020-02.csv
* yellow_tripdata_2020-03.csv

Two pricing models
* On demand pricing (based on the amount of data you scan)
	* 1 TB of data processed is $5
* Flat rate pricing
	* Based on number of pre-requested slots
	* 100 slots -> $2,000/month = 400 TB data processed on demand pricing
Generally, it doesn't make sense to use flat rate pricing unless you are using more than 400 TB of data per month. Also with regard to flat rate pricing, you have to worry about queries competing with each other since only 100 slots are available, that is, only 100 queries can run at one time (and the others would need to wait).

In BigQuery, we can create an external data from data stored in GCS by running the following SQL query:

```SQL
CREATE OR REPLACE EXTERNAL TABLE `taxi-rides-ny.nytaxi.external_yellow_tripdata`
OPTIONS (
	format = 'CSV',
	uris = ['gs://mage-zoomcamp-jessica-desilva/nyc_taxi/yellow_tripdata_2020-*.csv', 'gs://mage-zoomcamp-jessica-desilva/nyc_taxi/yellow_tripdata_2029-*.csv']
);
```

Notice that when we do this, external_yellow_tripdata will show up as a table in our ny_taxi schema and it has inferred the data types of the columns automatically:

![[Screenshot 2024-02-08 at 3.58.39 PM.png]]

However, BigQuery doesn't have full information about this table because the table is external to BigQuery (that is it only lives in GCS).

### Partitioning

Here we will compare query performance and costs with tables and partitioned tables. First, let's create a table in BigQuery for the external_yellow_tripdata table:

```SQL
CREATE OR REPLACE TABLE `iron-cycle-412122.ny_taxi.yellow_tripdata_non_partitioned` AS
SELECT * FROM `iron-cycle-412122.ny_taxi.external_yellow_tripdata`
```

We see this ran successfully as yellow_tripdata_non_partitioned shows up in our ny_taxi schema on the left. Now let's take this raw data and convert it into a partitioned table. As we have seen before, partitioning takes as an argument a column and then splits up the raw data into smaller tables according to the value of that column.

![[Screenshot 2024-02-08 at 4.06.58 PM.png]]

To create a partitioned table, use the ```PARTITION BY``` command:

```SQL
CREATE OR REPLACE TABLE `iron-cycle-412122.ny_taxi.yellow_tripdata_partitioned`
PARTITION BY
	DATE(tpep_pickup_datetime) AS
SELECT * FROM `iron-cycle-412122.ny_taxi.yellow_tripdata_non_partitioned`
```

Now let's inspect our partitioned table:

![[Screenshot 2024-02-08 at 4.21.13 PM.png]]

Let's see the impact of the partitioned table when we query with a filter on the date:

```SQL
SELECT DISTINCT(VendorID)
FROM `iron-cycle-412122.ny_taxi.yellow_tripdata_non_partitioned`
WHERE DATE(tpep_pickup_datetime) BETWEEN '2019-01-01' AND '2019-01-31'
```

When we run this query and go to the execution details, we see the following:

![[Screenshot 2024-02-08 at 4.26.49 PM.png]]

Compared to when we do this same query on the partitioned table:

```SQL
SELECT DISTINCT(VendorID)
FROM `iron-cycle-412122.ny_taxi.yellow_tripdata_partitioned`
WHERE DATE(tpep_pickup_datetime) BETWEEN '2019-01-01' AND '2019-01-31'
```

We see significant improvement with 1/10th the number of records read, 1/3 the number of bytes shuffled, and much less slot time consumed.

![[Screenshot 2024-02-08 at 4.27.43 PM.png]]

We can take a look at the parts of the partition by looking into the INFORMATION_SCHEMA table of the dataset/schema (which for us is ny_taxi).

```SQL
SELECT table_name, partition_id, total_rows
FROM ny_taxi.INFORMATION_SCHEMA.PARTITIONS
WHERE table_name = 'yellow_tripdata_partitioned'
ORDER BY total_rows DESC;
```

We see that each of the parts of our partition are approximately the same size (i.e., same number of rows).

![[Screenshot 2024-02-08 at 4.38.13 PM.png]]

### Clustering

If you typically filter your data according to two columns, you can **cluster** a partitioned table. What this will do is it will take a partitioned table and then each of the parts of the partition will be clustered (sort of like partitions of the partition) according to the secondary column.

```SQL
CREATE OR REPLACE TABLE `iron-cycle-412122.ny_taxi.yellow_tripdata_partitioned_clustered`
PARTITION BY DATE(tpep_pickup_datetime)
CLUSTER BY VendorID AS
SELECT * FROM `iron-cycle-412122.ny_taxi.external_yellow_tripdata`
```