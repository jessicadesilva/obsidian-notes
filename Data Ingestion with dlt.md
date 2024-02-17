In this hands-on workshop, we'll learn how to build data ingestion pipelines. In particular, we will cover the following steps:
* Extracting data from APIs, or files.
* Normalizing and loading data.
* Incremental loading (but not incremental extraction)

By the end of this workshop, you'll be able to write data pipelines like a senior data engineer: Quickly, concisely, scalable, and self-maintaining.

**dlt (data load tool)** is a library that automates the tedious part of data ingestion: loading, schema management, data type detection, scalability, self healing, scalable extraction, etc.

Due to its simplicity of use, dlt enables beginners to
* Build pipelines 5 - 10x faster than without it
* Build self-healing, self maintaining pipelines with all the best practices of data engineers. Automating schema changes removes the bulk of maintenance efforts.
* Govern your pipelines with schema evolution alerts and data contracts
* and generally develop pipelines like a senior, commercial data engineer.
## What is data loading, or data ingestion?

Data ingestion is the process of extracting data from a producer, transporting it to a convenient environment, and preparing it for usage by normalizing it, sometimes cleaning, and adding metadata.

In many data science teams, data magically appears - because the engineer loads it.
* Sometimes the format in which it appears is structured, and with explicit schema
	* In that case, they can go straight to using it; Examples: parquet, avro, or table in a database,
* Sometimes the format is weakly typed and without explicit schema, such as csv, json
	* in which case some extra normalization or cleaning might be needed before usage
	
*What is a schema?* The schema specifies the expected format and structure of data within a document or data store, defining the allowed keys, their dat types, and any constraints or relationships.

As a data engineer, you will be the one making datasets magically appear. Here's what you need to learn to build pipelines:
* Extracting data
* Normalizing, cleaning, adding metadata such as schema and types
* and incremental loading, which is vital for fast, cost effective data refreshes.

### What else does a data engineer do?
* It might seem simplistic, but in fact a data engineer's main goal is to ensure data flows from source systems to analytical destinations
* So besides building pipelines, running pipelines and fixing pipelines, a data engineer may also focus on optimizing data storage, ensuring data quality and integrity, cost management, implementing effective data governance practices, and continuously refining data architecture to meet the evolving needs of the organization.
* Ultimately, a data engineer's role extends beyond the mechanical aspects of pipeline development, encompassing the strategic management and enhancement of the entire data lifecycle.
* This workshop focuses on building robust, scalable, self-maintaining pipelines with built-in governance - in other words, best practices applied.

## Extracting data
### Considerations when extracting data
In this section, we will learn about extracting data from source systems, and what to care about when doing so.

Most data is stored behind an API
* Sometimes that's a RESTful API for some business application, returning records of data
* Sometimes the API returns a secure file path to something like a json or parquet file in a bucket that enables you to grab the data in bulk
* Sometimes the API is something else (mongo, sql, other databases or applications) and will generally return records as JSON - the most common interchange format.

As an engineer, you will need to build pipelines that "just work" -- that pull data out of these sources and put it in a destination.

So here's what you need to consider on extraction to prevent the pipelines from breaking, and to keep them running smoothly.

* Hardware limits: During this course we will cover how to navigate the challenges of managing memory
* Network limits: Sometimes networks can fail. We can't fix what could go wrong but we can retry network jobs until they succeed. For example, dlt library offers a requests "replacement" that has built in retries. We won't focus on that during the course but you can read the docs on your own.
* Source API limits: Each source might have some limits such as how many requests you can do per second. We would call these "rate limits". Read each source's docs carefully to understand how to navigate these obstacles. You can find some examples of how to wait for rate limits in our verified sources repositories.

### Extracting data without hitting hardware limits

What kind of limits could you hit on your machine? In the case of data extraction, the only limits are memory and storage. This refers to the RAM or virtual memory, and the disk, or physical storage.

**Managing memory**
* Many data pipelines run on serverless functions or on orchestrators that delegate the workloads to clusters of small workers.
* These systems have a small memory or share it between multiple workers - so filling the memory is BAD. It might lead to not only your pipeline crashing but crashing the entire container or machine that might be shared with other worker processes, taking them down too.
* The same can be said about disk - in most cases your disk is sufficient but in some cases it's not. For those cases, mounting an external drive mapped to a storage bucket is the way to go. Airflow for example supports a "data" folder that is used just like a local folder but can be mapped to a bucket for unlimited capacity.

**So how do we avoid filling the memory?**
* We often do not know the volume of data upfront
* And we cannot scale dynamically or infinitely on hardware during runtime
* So the answer is: Control the max memory you can use

**Control the max memory used by streaming the data**
Streaming here refers to processing the data event by event or chunk by chunk instead of doing bulk operations.

Let's look at some classic examples of streaming where data is transferred chunk by chunk or event by event:
* Between an audio broadcaster and an in-browser audio player
* Between a server and a local video player
* Between a smart home device or IoT device and your phone
* Between Google Maps and your navigation app
* Between Instagram Live and your followers
What do data engineers do? We usually stream the data between buffers, such as
* from API to local file
* from webhooks to event queues
* from event queue (Kafka, SQS) to Bucket

**Streaming in python via generators**
Let's focus on how we build most data pipelines:
* To process data in a stream in python, we use generators, which are functions that can return multiple times - by allowing multiple returns, the data can be released as it's produced, as stream, instead of returning it all at once as a batch.
Take the following theoretical example:
* We search twitter for "cat pictures". We do not know how many pictures will be returned - maybe 10, maybe 10,000,000. Will they fit in memory? Who knows.
* So to grab this data without running out of memory, we would use a python generator.
* What's a generator? In simple words, it's a function that can return multiple times. Here's an example of a regular function, and how that function looks if written as a generator.

**Generator examples:**
Let's look at a regular returning function, and how we can rewrite it as a generator.

**Regular function** collects data in memory. Here you can see how data is collected row by row in a list called data before it is returned. This will break if we have more data than memory.

```python
def search_twitter(query):
	data = []
	for row in paginated_get(query):
		data.append(row)
	return data

# Collect all the cat picture data
for row in search_twitter("cat pictures");
	# Once collected,
	# print row by row
	print(row)
```

When calling for row in search_twitter("cat pictures): all the data must first be downloaded before the first record is returned.

Let's see how we could rewrite this as a generator.

**Generator for streaming the data.** The memory usage here is minimal. 

As you can see, in the modified function, we field each row as we get the data, without collecting it into memory. We can then run this generator and handle the data item by item.

```python
def search_twitter(query):
	for row in paginated_get(query):
		yield row

# Get one row at a time
for row in search_twitter("cat pictures"):
	# print the row
	print(row)
	# do something with the row such as cleaning it and
	# writing it to a bucket
	# continue requesting and printing data
```

When calling for row in search_twitter("cat pictures"): the functino only runs until the first data item is yielded, before printing - so we do not need to wait long for the first value. It will then continue until there is no more data to get.

If we wanted to get all the values at once from a generator instead of one by one, we would need to first "run" the generator and collect the data. For example, if we wanted to get all the data in memory we could do data-list(search_twitter("cat pictures")) which would run the generator and collect all the data in a list before continuing.

### 3 Extraction examples
**Example 1: Grabbing data from an api**
* This is the bread and butter of how data engineers are pulling data, so follow along in the colab or in your local setup.
For these purposes, we created an api that can serve the data you are already familiar with, the NYC taxi dataset.

The api documentation is as follows:
* There are a limited number of records behind the api
* The data can be requested page by page, each page containing 1000 records
* If we request a page with no data, we will get a successful response with no data. This means that when we get an empty page, we know there is no more data and we can stop requesting pages - this is a common way to paginate but not the only one - each api may be different.
* details:
	* method: get
	* url: https://us-central1-dlthub-analytics.cloudfunctions.net/data_engineering_zoomcamp_api
	* parameters: page integer. Represents the page number you are requesting. Defaults to 1.
So how do we design our requestor?
* We need to request page by page until we get no more data. At this point, we do not know how much data is behind the api
* It could be 1000 records or it could be 10GB of records. So let's grab the data with a generator to avoid having to fit an undetermined amount of data into ram.
In this approach in grabbing data from apis, we have pros and cons:
* Pros: **Easy memory management** thanks to api returning events/pages
* Cons: **Low throughput**, due to the data transfer being constrained by the api.

```python
import requests

BASE_API_URL = "https://us-central1-dlthub-analytics.cloudfunctions.net/data_engineering_zoomcamp_api"

  

# I call this a paginated getter
# as it's a function that gets data
# and also paginates until there is no more data
# by yielding pages, we "microbatch", which speeds up downstream processing

def paginated_getter():
	page_number = 1
	
	while True:
		# Set the query parameters
		params = {'page': page_number}

		# Make the GET request to the API
		response = requests.get(BASE_API_URL, params=params)
		response.raise_for_status() # Raise an HTTPError for bad response
		page_json = response.json()
		print(f'got page number {page_number} with {len(page_json)} records')

		# if the page has no records, stop iterating
		if page_json:
			yield page_json
			page_number += 1
		else:
			# no more data, break the loop
			break

if __name__ == '__main__':
	# use the generator to iterate over pages
	for page_data in paginated_getter():
		# process each page as needed
		print(page_data)
```

**Example 2: Grabbing the same data from file - simple download**
* This part is demonstrative, so you do not need to follow along, just pay attention.
Why am I showing you this? So when you do this in the future, you will remember there is a best practice you can apply for scalability.

Some apis respond with files instead of pages of data. The reason for this is simple: throughput and cost. A restful api that returns data has to read the data from storage and process and return it to you by some logic - if this data is large, this costs time, money, and creates a bottleneck.

A better way is to offer the data as files that someone can download from storage directly, without going through the restful api layer. This is common for apis that offer large volumes of data, such as ad impressions data.

In this example, we grab exactly the same data as we did in the API example above, but now we get it from the underlying file instead of going through the API.
* Pros: **High throughput**
* Cons: **Memory** is used to hold all the data

This is how the code could look. As you can see in this case our data and parsed_Data variables hold the entire file data in memory before returning it. Not great.

```python
import requests
import json

url = "https://storage.googleapis.com/dtc_zoomcamp_api/yellow_tripdata_2009-06.jsonl"

def download_and_read_jsonl(url):
	response = requests.get(url)
	response.raise_for_status() # Raise an HTTPError for bad responses
	data = response.text.splitlines()
	parsed_data = [json.loads(line) for line in data]
	return parsed_data

downloaded_data = download_and_read_jsonl(url)

if downloaded_data:
	# process of print the downloaded data as needed
	print(downloaded_data[:5]) #print the first 5 entries as an example
```

**Example 3: Same file, streaming download**
* This is the bread and butter of data engineer pulling data, so follow along in the colab.
Okay, downloading files is simple, but what if we want to do a stream download.

That's possible too - in effect giving us the best of both worlds. In this case, we prepared a jsonl file which is already split into lines making our code simple. But json (not jsonl) files could also be downloaded in this fashion, for example using the ijson library.

What are the pros and cons of this method of grabbing data?

Pros: **High throughput, easy memory management** because we are downloading a file

Cons: **Difficult to do for columnar file formats**, as entire blocks need to be downloaded before they can be deserialized to rows. Sometimes, the code is complex too.

Here's what the code looks like - in a jsonl file each line is a json document, or a "row" of data, so we yield them as they get downloaded. This allows us to download one row and process it before getting the next row.

```python
import json

url = "https://storage.googleapis.com/dtc_zoomcamp_api/yellow_tripdata_2009-06.jsonl"

def stream_download_json(url):
	response = requests.get(url, stream=True)
	response.raise_for_status() # Raise an HTTPError for bad responses
	for line in response.iter_lines():
		if line:
			yield json.loads(line)

# time the download
import time
start = time.time()

# Use the generator to iterate over rows with minimal memory usage
row_counter = 0
for row in stream_download_json(url):
	print(row)
	row_counter += 1
	if row_counter >= 5:
		break

# time the download
end = time.time()
print(end - start)
```

## Load the generators

We have 3 ways to download the same data. Let's use the fast and reliable way to load data and inspect it in DuckDB. In this example, we are using dlt library to do the loading, which will process data from the generators incrementally, following the same memory management paradigm.

We will discuss more details about dlt or "data load tool" later.

Here we will use DuckDB. DuckDB is an in-memory analytical database (like SQLite). It is not a persistent database like PostgreSQL (persistent meaning if systems are down, processes or objects can continue on). When we load data to DuckDB, we create some files underneath with data and DuckDB, in this Python process, is able to read the data. So we can use it in a notebook, which is why dlt likes to use it. You can also think of it as development mode, we can iterate on it quickly since it is right here and then push it to production in BigQuery once it's ready. dlt is database agnostic, so if it works on DuckDB it will work on any other database. The usual flow is to use DuckDB for testing locally with dlt, then to production we say instead of writing to DuckDB write it to BigQuery (the only thing that changes is the destination).

```python
import dlt # make sure to pip install dlt(duckdb)
import duckdb # make sure to pip install

# define the connection to laod to
# we now use duckdb, but you can switch to BigQuery later
generators_pipeline = dlt.pipeline(destination='duckdb',
								  dataset_name='generators')

# we can load any generator to a table at the pipeline destination as follows:
info = generators_pipeline.run(paginated_getter(),
						  table_name='http_download',
						  write_disposition='replace')

# the outcome metadata is returned by the load and we can inspect it by printing it
print(info)

# we can load the next generator to the same or to a different table
info = generators_pipeline.run(stream_download_json(url),
							  table_name='stream_download',
							  write_disposition='replace')

print(info)
```

```python
conn = duckdb.connect(f"{generators_pipeline.pipeline_name}.duckdb")

# let's see the tables
conn.sql(f"SET search_path = '{generators_pipeline.dataset_name}'")
print('Loaded tables:')
display(conn.sql("show tables"))

# and the data

print("\n\n\n http_download table below:")

rides = conn.sql("SELECT * FROM stream_download").df()
display(rides)

print("\n\n\n stream_download table below:")

passengers = conn.sql("SELECT * FROM stream_download").df()
display(passengers)

# As you can see, the same data was loaded in both cases
```

In the colab notebook, you can also find a code snippet to load the data - but we will load some data later in the course and you can explore the colab on your own after the course.

What is worth keeping in mind at this point is that our loader library that we will use later, dlt or data load tool, will respect the streaming concept of the generator and will process it in an efficient way keeping memory usage low and using parallelism where possible.

Let's move over to the colab notebook and run examples 2 and 3, compare them, and finally load examples 1 and 3 to DuckDB.

## Normalizing data

You often heat that data people spend most of their time "cleaning" data. What does this mean?

Let's look granularly into what people consider data cleaning.

Usually we have 2 parts:
* Normalizing data without changing its meaning,
* and filtering data (like taking out outliers) for a use case, which changes its meaning.

**Part of what we can often call data cleaning is just metadata work:**
* Add types (string to number, string to timestamp, etc.)
* Rename columns: Ensure column names follow a supported standard downstream 0 such as no strange characters in the names.
* Flatten nested dictionaries: Bring nested dictionary values into the top dictionary row
* Unnest lists or arrays into child tables: Arrays or lists cannot be flattened into their parent record, so if we want flat data we need to break them out into separate tables.
* We will look at a practical example next, as these concepts can be difficult to visualize from text.

**Why prepare data? Why not just use json as is?**
* json is really a transfer format, so it doesn't describe the data much.
* We do not easily know what is inside a json document due to lack of schema
* Types are not enforced between rows of json - we could have one record where age is 25 and another where age is twenty five, and another where it's 25.00. Or in some systems, you might have a dictionary for a single record, but a list of dicts for multiple records. This could easily lead to applications downstream breaking.
* We cannot just use json data easily, for example we would need to convert strings to time if we want to do a daily aggregation.
* Reading json loads more data into memory, as the whole document is scanned - while in parquet or databases we can scan a single column of a document. This causes costs and slowness.
* json is not fast to aggregate - columnar formats are
* json is not fast to search
* Basically, json is designed as a "lowest common denominator format" for "interchange" / data transfer and is unsuitable for direct analytical usage.

**Practice example**
* This is the bread and butter of data engineers pulling data, so follow along in the colab notebook.

In the case of the NY taxi rides data, the dataset is quite clean - so let's instead use a small example of more complex data. Let's assume we know some information about passengers and stops.

For this example we modified the dataset as follows.
* We added nested dictionaries
```json
"coordinates": {
			"start": {
				"lon": -73.787442
				"lat": 40.641525
				}
			},
```
* We added nested lists
```json
"passengers": [
			{"name": "John", "rating": 4.9},
			{"name": "Jack", "rating": 3.9}
			],
```
* We added a record hash that gives us a unique id for the record, for easy identification
```json
"record_hash": "b00361a396177a9cb410ff61f20015ad";
```
We want to load this data to a database. How do we want to clean the data?
* We want to flatten dictionaries into the base row
* We want to flatten lists into a separate table
* We want to convert time strings into time type

```python
data = [
    {
        "vendor_name": "VTS",
		"record_hash": "b00361a396177a9cb410ff61f20015ad",
        "time": {
            "pickup": "2009-06-14 23:23:00",
            "dropoff": "2009-06-14 23:48:00"
        },
        "Trip_Distance": 17.52,
        "coordinates": {
            "start": {
                "lon": -73.787442,
                "lat": 40.641525
            },
            "end": {
                "lon": -73.980072,
                "lat": 40.742963
            }
        },
        "Rate_Code": None,
        "store_and_forward": None,
        "Payment": {
            "type": "Credit",
            "amt": 20.5,
            "surcharge": 0,
            "mta_tax": None,
            "tip": 9,
            "tolls": 4.15,
			"status": "booked"
        },
        "Passenger_Count": 2,
        "passengers": [
            {"name": "John", "rating": 4.9},
            {"name": "Jack", "rating": 3.9}
        ],
        "Stops": [
            {"lon": -73.6, "lat": 40.6},
            {"lon": -73.5, "lat": 40.5}
        ]
    },
]
```

Now let's normalize this data.

## Introducing dlt

dlt is a python library created for the purpose of assisting data engineers to build faster, and more robust pipelines with minimal effort.

You can think of dlt as a loading tool that implements the best practices of data pipelines enabling you to just "use" those best practices in your own pipelines, in a declarative way.

This enables you to stop reinventing the flat tire (people have the same problems over and over), and leverage dlt to build pipelines much faster than if you did everything from scratch.

dlt automates much of the tedious work a data engineer would do, and does it in a way that is robust. dlt can handle things like:
* Schema: Inferring and evolving schema, alerting changes, using schemas as data contracts (e.g., only accept events with a specific format).
* Typing data, flattening structures, renaming columns to fit database standards. In our example we will pass the "data" you can see above and see it normalized.
* Processing a stream of events/rows without filling memory. This includes extraction from generators.
* Loading to a variety of dbs or file formats.

Let's use it to laod our nested json to duckdb:

Here's how you would do that on you rlocal machine. I will walk you through before showing you in colab as well.

First, install dlt.

```bash
# Make sure you are using Python 3.8-3.11 and have pip installed
# spin up a venv
python -m venv .venv
source .venv/bin/activate
# pip install
pip install dlt[duckdb]
```

Next, grab your data from above and run this snippet
* here we define a pipeline, which is a connection to a destination
* and we run the pipeline, printing the outcome

```python
# define the connection to laod to
# we now use DuckDB, but you can switch to BigQuery later
pipeline = dlt.pipeline(pipeline_name="taxi_data",
						   destination='duckdb',
						   dataset_name='taxi_rides')

# run the pipeline with default settings, and capture the outcome
info = pipeline.run(data,
				   table_name="rides",
				   write_disposition="merge",
				   primary_key="record_hash")

# show the outcome
print(info)
```

If you are running dlt locally you can use the built in streamlit app by running the cli command with the pipeline name we chose above.

**Inspecting the nested structure, joining the child tables**

Let's look at what happened during the load:
* By looking at the loaded tables, we can see our json document got flattened and sub-documents got split into separate tables.
* We can re-join those child tables to the parent table by using the generated keys on ```parent_table.dtl_id = child_table._dlt_parent_id```.
* Data types: if you will pay attention to datatypes, you will note that the timestamps, which in json are of string type, are now of timestamp type in the db.

```python
# show the ouctomes

conn = duckdb.connect(f"{pipeline.pipeline_name}.duckdb")

# let's see the tables
conn.sql(f"SET search_path = '{pipeline.dataset_name}'")
print('Loaded tables: ')
display(conn.sql("show tables"))

print("\n\n\n Rides table below: Not the times are properly typed")
rides = conn.sql("SELECT * FROM rides").df()
display(rides)

print("\n\n\n Passengers table")
passengers = conn.sql("SELECT * FROM rides__passengers").df()
display(passengers)
print("\n\n\n Stops table")
stops = conn.sql("SELECT * FROM rides__stops").df()
display(stops)

# to reflect the relationships between parent and child rows, let's join them
# of course this will have 4 rows due to the two 1:n joins

print("\n\n\n joined table")

joined = conn.sql("""
SELECT *
FROM rides as r
LEFT JOIN rides__passengers as rp
	ON r._dlt_id = rp._dlt_parent_id
LEFT JOIN rides__stops as rs
	ON r._dlt_id = rs._dlt_parent_id
"""	
).df()
display(joined)
```