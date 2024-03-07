In this workshop, we will be discussing stream processing in SQL with RisingWave. RisingWave aims to build a cost-effective and scalable database solution designed to streamline data processing and query serving.

# Why stream processing?
## Batch vs. Stream (Data Ingestion)

Batch ingestion is when data is coming in hourly/daily/weekly batches and so the data is more static and the load is lower as well. Stream ingestion is when you have dynamic, high volume, high velocity data.

Once the data has been ingested by the upstream source, then for batch it will be ingested into a database