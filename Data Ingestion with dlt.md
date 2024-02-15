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