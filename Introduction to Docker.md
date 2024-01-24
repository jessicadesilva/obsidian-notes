# **What is Docker and why do we need it?**
Docker is a set of "platform as a service" products that use operating system-level virtualization to deliver software in packages called **containers**. **Containers** are isolated from one another and bundle their own software, libraries, and configuration files, and can communicate with each other.

A **data pipeline** is a method in which raw data is ingested (input data) from various data sources and then ported to data store, like a data lake or data warehouse (output data), for analysis. We can use containers to run a data pipeline.

*Example:* Let's say your personal (i.e., host) computer is running Windows. We can create a **container** using Docker for the data pipeline that runs Ubuntu 20.04 with Python 3.9, Pandas, Postgres connection library, etc. The container has everything that the data pipeline needs to run.

We can have several Docker containers in one computer. A container can contain a single service (such as Postgres) without requiring installation on your personal (i.e., host) computer.

A **Docker image** is a set of instructions that are needed to create an instance of a container in any environment.

The main benefit of Docker is *reproducibility*. By eliminating the computing environment as a variable, others should be able to run the data pipeline with fewer issues. This allows us to run pipelines in the cloud (AWS Batch, Kubernetes jobs), Spark, Serverless (AWS Lambda, Google functions). Docker also makes it easier to run local experiments/CI/CD integration tests without installation.

Be sure to install Docker Desktop:
https://www.docker.com/products/docker-desktop/
and have Docker running.

The **Docker image** will be a file called Dockerfile (no extension) in your working directory. You should see a little whale icon to the left of the file name if you are using VSCode. You can test that Docker is functioning properly by running the follow line of code in your terminal (for MacOS):

```console
docker run hello-world
```

If you get a message asking "Is the docker daemon running?" be sure to spin up Docker Desktop before running the line of code. If you see the following text somewhere in the output, Docker ran properly:

![[attachments/Screenshot 2024-01-22 at 11.52.31 AM.png]]

The following line of code spins up an instance of a container with image name image-name. 

```console
docker run image-name
```

If you want to interact with the instance through the terminal once it has been spun up, you can use interactive mode with this command instead:

```console
docker run -it image-name
```

For example, running

```console
docker run -it ubuntu
```

gives this as the last line of output:
![[attachments/Screenshot 2024-01-22 at 11.56.16 AM.png]]
which means it is ready to run terminal commands in the root directory of the Ubuntu instance. You can exit the container using the keyword **exit** if running it in interactive mode. Otherwise, you'll need to (in a separate terminal) run the command **docker ps** to see all running containers, identify the container you want to kill and then use the command **docker kill "container id"**. Note that instances of an image are temporary and once exited (using the command **exit** if the "it" flag is used) spinning up another instance of the Docker image will have a completely fresh start. Also, what you do in the instance of the image (like delete all files) will not affect your host computer/machine in any way.

If you want to have a specific version of something in your container, such as Python 3.9, you can use the following syntax:

```console
docker run -it python:3.9
```

If you want to have certain Python libraries in the container, we must enter the python:3.9 container instance through bash. To do this, we can use the following syntax:
```console
docker run -it --entrypoint=bash python:3.9
```

Then we can pip install pandas (thereby installing pandas on the Docker container instance, not your local computer) then use the python command to start programming. But remember, container instances have no memory of itself once it is exited, so if we exit we will lose the installation of the pandas package that we did if we spin up the container again. This is why Dockerfiles are useful. The Dockerfile will be a series of commands that will run every time we spin up an instance of the container. Here is what our Dockerfile looks like that will install pandas in our python:3.9 container:

```dockerfile
FROM python:3.9.1

RUN pip install pandas

ENTRYPOINT [ "bash" ]
```

To run a Dockerfile, we first need to build the Docker image from the Dockerfile and create a name for the image.

```console
docker build -t test:pandas .
```

Here the image name will be test with the tag pandas and the image file will be created in the current directory as indicated by the ".".

Now if I go to Docker Desktop and click the section "Images" I should see the following:

![[attachments/Screenshot 2024-01-22 at 12.12.45 PM.png]]

So now that we have built the image, we can run the image using the syntax from before:

```console
docker run -it test:pandas
```

The container instance will open in bash as we indicated with ENTRYPOINT on the Dockerfile. We can use the command python to run python code, and if you run import pandas you should see that it will import with no errors (possibly deprecation warnings as of 1/22/2024).

Now if we want to run a local file in our Docker container instance, we can create the file in our directory and, in the Dockerfile, copy the local file to the Docker container instance via:

```dockerfile
COPY local_file_name copied_file_name
```

To change the working directory in the Docker container instance, you can use the following line in the Dockerfile:

```dockerfile
WORKDIR /directory_name
```

Our Dockerfile now looks as follows:

```dockerfile
FROM python:3.9.1

RUN pip install pandas

WORKDIR /app

COPY pipeline.py pipeline.py

ENTRYPOINT [ "bash" ]
```

Remember that every time you change the Dockerfile, you need to build the image before spinning up an instance of the container. We see that when we spin up an instance of the container with the revised Dockerfile, the pipeline.py file is copied into the app directory of our Docker container instance.

We can run the file that we have copied via the command in bash:
```console
python pipeline.py
```

Once the code in pipeline.py completes, the container instance is exited automatically.

We can also run this file automatically when the container instance is spun up by changing the entrypoint to ["python", "pipeline.py"] in our Dockerfile:

```dockerfile
FROM python:3.9.1

RUN pip install pandas

WORKDIR /app

COPY pipeline.py pipeline.py

ENTRYPOINT [ "python", "pipeline.py"]
```

If the python file you are trying to run requires input arguments, such as this python file:

```python
import sys
import pandas as pd

print(sys.argv)

day = sys.argv[1]

# some fancy stuff with pandas

print(f"Job finished successfully for day = {day}")
```

you can feed in the arguments with the docker run command:

```console
docker run -it test:pandas 2024-01-22
```

and we will see the following output:

![[attachments/Screenshot 2024-01-22 at 12.31.53 PM.png]]

Remember that if you make changes to the .py file that you are copying into your container instance, you will need to rebuild the image before spinning up the instance.

## Homework question
Run the command to get information on Docker
```console
docker --help
```

Now run the command to get help on the "docker build" command:

```console
docker build --help
```

Do the same for "docker run".

```console
docker run --help"
```

Which tag has the following text? - *Automatically remove the container when it exits*

* ```--delete```
* ```--rc```
* ```--rmc```
* ```--rm```
**The correct answer is:** ```--rm```

Run docker with the python:3.9 image in an interactive mode and the entrypoint of bash. Now check the python modules that are installed (use ```pip list```).

What is the version of the package *wheel*?
* 0.42.0
* 1.0.0
* 23.0.1
* 58.1.0
**The correct answer is:** 0.42.0

# **How to run Postgres locally with Docker**

We will need a Docker image of Postgres, here is the postgres:13 Dockerfile that Docker already has pre-built, that is, we do not need to create our own Dockerfile with this information as this image is one of the defaults stored by Docker:

```dockerfile
services:
	postgres:
		image: postgres:13
		environment:
			POSTGRES_USER: test_user
			POSTGRES_PASSWORD: test_password
			POSTGRES_DB: test_db
		volumes:
			- postgres-db-volume:/var/lib/postgresql/data
		port:
			5432:5432
		healthcheck:
			test: ["CMD", "pg_isready", "-U", "test_user"]
			interval: 5s
			retries: 5
		restart: always
```

* The docker container will be running the service "postgres" and the name of the image is going to be "postgres" with the tag of "13".
* We have some environment variables to set: POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB which are going to be know/used by Postgres. Because this Zoomcamp isn't using airflow and it can be confusing to know which "airflow" is actually being used where, I have changed the defaults of the three environment variables from that of the video so that they are unique and independent of airflow.
* There are some files that Postgres will need to save (like log files), and we want to have a mapping between the files on the container instance from postgres to our local/host machine directory. This also makes sure that when we exit and then spin up a new instance of this container, postgres will still have the old files since they were stored on our local machine and that directory is mapped to the container instance. This mapping from the host computer to a directory in the container is called mounting. This is where the "volumes" command comes into play, here we are mapping the postgres-db-volume folder in our local directory to /var/lib/postgresql/data in the container instance.
* Postgres uses a port number to make connections between the client who has SQL queries it wants to run on the database, and the server which hosts the database. We want to map the port number for Postgres on the container to one on our host machine so that we can send SQL queries to it. By default this is 5432. Remember the first port is the local one and the second is the Docker container port.
* The healthcheck command will run the command pg_isready -U test_user to ensure postgres is working correctly with 5 seconds between each of the 5 retries (if necessary).

To create an instance of this Docker image, we need to specify all of the parameters if they are not the same as default in the following way:

```console
docker run -it \
	-e POSTGRES_USER="root" \
	-e POSTGRES_PASSWORD="root_password" \
	-e POSTGRES_DB="ny_taxi" \
	-v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
	-p 5432:5432 \
	postgres:13
```
Notice that -e is the abbreviation for environment (variable), -v abbreviates volumes, and -p is the flag for port. The name of the image is postgres with the tag 13 (version number here).

We should see this as the last line of the output when we run the above in the terminal:
![[attachments/Screenshot 2024-01-22 at 1.47.43 PM.png]]

Notice that you now have a folder in the current directing of your host machine that is called "ny_taxi_postgres_data" with all of the files that are used to store information related to the postgres database "ny_taxi".

In order to access the postgres database, we need some sort of GUI or command line interface to see/interact with it. To do that on the command line, you can use the python package pgcli which stands for postgres command line interface. Note that this is all going to happen on your local/host machine because we have mapped the postgres port from the container to a port on our local machine which allows us to communicate with it.

In a new terminal window, pip install pgcli. Then run the following:
```console
pgcli -h localhost -p 5432 -U root -d ny_taxi
```

* The flag -h is the host, since it is mapped to a port on our local computer this is localhost.
* The flag -p is the port (on our local/host computer).
* -U is the username that we used in the docker run command.
* -d is the database name that we are trying to access from the docker run command.

Assuming you used root as the username, you should see the following:
![[attachments/Screenshot 2024-01-22 at 2.00.58 PM.png]]
You should type the password given in the docker-run command and then enter. When you type characters for the password, the cursor will not move. If successful, you should see the following:

![[attachments/Screenshot 2024-01-22 at 2.01.45 PM.png]]
Since we don't have any data in our ny_taxi database, there is nothing to query. We can make sure the connection is working by typing the following command:

```console
\dt
```

And we will get the following output to show that we have no tables in our database:
![[attachments/Screenshot 2024-01-22 at 2.03.38 PM.png]]
# **Ingesting NY taxi data into Postgres with Python**

To download the data csv file into our local working directory, we can run the following two commands in a new terminal (navigated to the working directory):

```console
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz
```
The above downloads the zipped file, then we unzip:
```console
gunzip yellow_trip_data_2021-01.csv.gz
```

We can take a peak at our file using ```less```

```console
less yellow_trip_data_2021-01.csv
```

This will output the following and many other rows:

![[attachments/Screenshot 2024-01-22 at 2.13.10 PM.png]]

The file provided at this link gives us an idea of what the columns mean in the table:
 [https://www1.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf](https://www1.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf)

Now, in a Jupyter Notebook, we can import the data into a dataframe and then ingest it into Postgres. First, let's import the (first 100 rows of) data using pandas:

```python
import pandas as pd

pd.__version__

df = pd.read_csv('yellow_tripdata_2021-01.csv', nrows=100)

df
```

However, some of the columns are have datetime objects as values and pandas cannot automatically detect that. To tell it which columns are datetime objects, let's use the parse_dates argument within read_csv:

```python
import pandas as pd

pd.__version__

df = pd.read_csv('yellow_tripdata_2021-01.csv',
				 parse_dates=['tpep_pickup_datetime',
							 'tpep_dropoff_datetime'],
				 nrows=100)

df
```

In order to ingest the data into Postgres, we should define its schema. We can get the schema with a module from pandas that outputs how the table schema would be defined in SQL:

```python
print(pd.io.sql.get_schema(df, name='yellow_taxi_data'))
```

We should see the following output:
![[attachments/Screenshot 2024-01-22 at 2.31.41 PM.png]]

To run SQL queries against our postgres database, we will want to create a query engine with a connection to our postgres database using SQLalchemy:

```python
from sqlalchemy import create_engine

# type of database is postgresql
# username is root
# user password is root_password
# host name is localhost
# port for host is 5432
# database name is ny_taxi
engine = create_engine('postgresql://root:root_password@localhost:5432/ny_taxi')
engine.connect()
```

We can get the table schema from io.sql.to_schema in the correct postgreSQL syntax for our database by specifying the engine we just created that is connected to our database. To do this, we will use the same command from above but now specify the connection to be the SQL query engine we created:

```python
print(pd.io.sql.get_schema(df, name='yellow_taxi_data', con=engine))
```

We see the output differs only slightly from what we had above:

![[attachments/Screenshot 2024-01-22 at 2.52.26 PM.png]]

Now we know that if we were to ingest the dataframe df into postgres, the above is the schema the table would have. Note that the line above did not actually create a table in our database.

Now that we know df will have the correct schema, let's go ahead and upload the data itself to our postgres database. We can ignore the original read_csv command and df we already created at the moment. Instead of just those first 100 rows, let's upload the entire dataset using the iterator capabilities of read_csv by turning on the iterator option and specifying the chunksize:

```python
df_iter = pd.read_csv('yellow_tripdata_2021-01.csv',
				parse_dates=['tpep_pickup_datetime',
							'tpep_dropoff_datetime'],
				iterator=True,
				chunksize=100000)

# to get first chunk call next(df_iter)
df = next(df_iter)
```

When you print df_iter, it will not print a dataframe but instead it tells you that it is an iterator (you can think of it like a linked list).

To ingest the data from a dataframe into a table in our postgres database, we can use the .to_sql method on the dataframe. The to_sql method requires the table name, connection (to the database), and it wants you to specify what should happen if the table already exists (fail, replace, append, etc.). For this initial set of data, we will replace the table. Once we take the next chunk of data, we will append.

```python
df.to_sql(name='yellow_taxi_data',
		  con=engine,
		  if_exists='replace')
```

If you run this and go back to the terminal where you are running pgcli, you should be able to run the \dt command and see the table in the database.
![[attachments/Screenshot 2024-01-22 at 3.07.39 PM.png]]

In order to ingest the rest of the table into postgres, we will loop through the iterator. For funzies, we will time it and print out a little message but that is not necessary. Further, the checking of count == 0 is to ensure we don't re-insert the first chunk:

```python
from time import time

count = 0
for df in df_iter:
	if count == 0:
		count += 1
		continue
	t_start = time()
	df.to_sql('yellow_taxi_data',
				con=engine,
				# note update to option
				if_exists='append')
	t_end = time()
	print(f'Inserted another chunk..., took {(t_end - t_start):.3f} seconds')
	count += 1
```

Now we have successfully ingested the entire table into postgres! Awesome.

## Homework
Run Postgres and load data as shown in the videos. We'll use the green taxi trips from September 2019:
```console
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-09.csv.gz
```
You will also need the dataset with zones:
```console
wget https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv
```

Download this data and put it into Postgres (with jupyter notebooks or with a pipeline).

Step 1: Run the two commands above in the terminal inside our working directory.
Step 2: Run the following in the terminal to unzip the file:
```console
gunzip green_tripdata_2019-09.csv.gz
```
Step 3: Run the scripts below in Jupyter notebook:
```python
# Ingesting green taxi data
df_iter_green = pd.read_csv('green_tripdata_2019-09.csv',
	parse_dates=['lpep_pickup_datetime',
	'lpep_dropoff_datetime'],
	iterator=True,
	chunksize=100000)

count = 0
for df in df_iter_green:
	t_start = time()
	if count == 0:
		df.to_sql('green_taxi_data',
			con=engine,
			if_exists='replace')
	else:
		df.to_sql('green_taxi_data',
			con=engine,
			if_exists='append')
	t_end = time()
	print(f'Inserted another chunk..., took {t_end - t_start:.3f} seconds')
	count += 1
```

```python
# Ingesting taxi zone lookup data
df = pd.read_csv('taxi+_zone_lookup.csv')

df.to_sql('taxi_zone_lookup',
	con=engine,
	if_exists='replace')
```
# **Connecting pgAdmin and postgres**

pgAdmin is a web-based GUI tool used to interact with the Postgres database sessions. Our goal from here on out is to be able to run SQL queries in pgAdmin on our postgres database using Docker. We can spin up an instance of a pgAdmin container using the image name dpage/pgadmin4. Since there are several environment variables we need to specify, let's write them here:

```console
docker run -it \
	-e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
	-e PGADMIN_DEFAULT_PASSWORD="pgadmin_password" \
	-p 8080:80 \
	dpage/pgadmin4
```

The PGADMIN_DEFAULT_EMAIL and PGADMIN_DEFAULT_PASSWORD is used to login to the pgAdmin interface. When you run this in the terminal (make sure Docker Desktop is running) then you can go to pgAdmin through your browser by typing in localhost:8080 as you would a url to a website. There you will see a login page where you use the credentials you specified above.
![[attachments/Screenshot 2024-01-22 at 3.41.59 PM.png]]


To create our connection to our existing postgres database, we need to host the two containers in the same network so that they can talk to each other. We can do this by specifying the network when we spin up the Docker containers. Let's start with the postgres container:

```console
docker run -it \
	-e POSTGRES_USER="root" \
	-e POSTGRES_PASSWORD="root_password" \
	-e POSTGRES_DB="ny_taxi" \
	-v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
	-p 5432:5432 \
	--network=pg-network \
	--name pg-database \
	postgres:13
```

So the network that our Docker container of postgres is living in is called pg-network. When we are working within that network and want to locate our postgres database, we can refer to it by its name: pg-database.

Now we will spin up our instance of pgAdmin within the same network:

```console
docker run -it \
	-e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
	-e PGADMIN_DEFAULT_PASSWORD="pgadmin_password" \
	-p 8080:80 \
	--network=pg-network \
	--name pg-admin \
	dpage/pgadmin4
```

We won't really need to refer to pgAdmin by its name pg-admin, but we include it here anyway. Now to go pgAdmin on your localhost:8080 port within a browser. Login using the credentials above and then click Add New Server.

![[attachments/Screenshot 2024-01-22 at 3.55.17 PM.png]]

Create a name for the server, here we will use Docker localhost.
![[attachments/Screenshot 2024-01-22 at 3.56.40 PM.png]]

In the Connection tab, the Host name/address is the name of our postgres database (pg-database) that we gave it when we created the container instance. The port is the port on the container that has the database (5432). For Maintenance database, we will use the name postgres. Then the Username and Password is that which we specified when creating the postgres container instance.
![[attachments/Screenshot 2024-01-22 at 4.01.49 PM.png]]

To take a quick look at our data, click Docker localhost on the left, then Databases (2), then Schemas (1), Tables (1), right click yellow_taxi_data, View/Edit Data, First 100 Rows and we should see the following:

![[attachments/Screenshot 2024-01-22 at 4.05.34 PM.png]]
## Homework
**Question 3. Count records**
How many taxi trips were totally made on September 18th 2019?
Tip: started and finished on 2019-09-18.

Remember that lpep_pickup_datetime and lpep_dropoff_datetime columns are in the format timestamp (date and hour+min+sec) and not in date.
* 5767
* 15612
* 15859
* 89009

I ran the following SQL query:

```SQL
SELECT COUNT(*)
FROM green_taxi_data
WHERE DATE(lpep_pickup_datetime)='2019-09-18' AND DATE(lpep_dropoff_datetime)='2019-09-18'
```

It returned the value 15612.

**Question 4. Largest trip for each day**
Which was the pick up day with the largest trip distance? Use the pick up time for your calculations.
* 2019-09-18
* 2019-09-16
* 2019-09-26
* 2019-09-21
  
I ran the following SQL query:

```SQL
SELECT lpep_pickup_datetime
FROM green_taxi_data
WHERE trip_distance=(SELECT MAX(trip_distance) FROM green_taxi_data)
```

It returned the value: 2019-09-26 19:32:52

**Question 5. Three biggest pickup boroughs**
Consider lpep_pickup_datetime in '2019-09-18' and ignoring Borough as Unknown.
Which were the 3 pickup boroughs that had a sum of total_amount superior to 50000?
* "Brooklyn","Manhattan","Queens"
* "Bronx", "Brooklyn", "Manhattan"
* "Bronx", "Manhattan", "Queens"
* "Brooklyn", "Queens", "Staten Island"

I ran the following SQL query:

```SQL
WITH gtd AS (
	SELECT *
	FROM green_taxi_data
	WHERE DATE(lpep_pickup_datetime)='2019-09-18'
	)
SELECT tzl."Borough"
FROM gtd
JOIN taxi_zone_lookup AS tzl ON gtd."PULocationID" = tzl."LocationID"
GROUP BY tzl."Borough"
HAVING SUM(gtd."total_amount") > 50000 AND tzl."Borough"!='Unknown'
```

It returned the values: Brooklyn, Manhattan, Queens.

**Question 6. Largest tip**
For the passengers picked up in September 2019 in the zone named Astoria, which was the dropoff zone that had the largest tip? We want the name of the zone, not the id.

Note: it's not a typo, it's tip not trip.

* Central Park
* Jamaica
* JFK Airport
* Long Island City/Queens Plaza

I ran the following SQL query:

```SQL
WITH astoria AS (
SELECT gtd."tip_amount" AS "tip_amount", gtd."DOLocationID" AS "DOLocationID"
FROM green_taxi_data AS gtd
JOIN taxi_zone_lookup AS tzl ON gtd."PULocationID"=tzl."LocationID"
WHERE tzl."Zone"='Astoria')

SELECT tzl."Zone"
FROM astoria
JOIN taxi_zone_lookup AS tzl ON astoria."DOLocationID"=tzl."LocationID"
WHERE astoria."tip_amount"=(SELECT MAX(tip_amount) FROM astoria)
```

It returned the value JFK Airport.

# **Putting the ingestion script into Docker**

We are going to run the ingestion script using Docker. First, we convert the Juptyer notebook to a Python script and make appropriate edits so that it is (1) connects to our Postgres database, and (2) is generalizable for any data source URL related to our New York Taxi project. To obtain (1), we will take in as arguments:
* **user** : username for our Postgres database
* **password** : password for Postgres database
* **host** : name of host for Postgres database (e.g., localhost or pg-database)
* **port** : port where the Postgres database is hosted
* **db** : the name of the database in Postgres we are accessing
* **table_name** : the script will create a new table (or replace existing) with this table name and containing the given data
* **url** : url to data source that will be stored in the Postgres table with the name table_name

Once the script parses the arguments, it will download the data from the provided url and unzip the downloaded file if necessary. Then, it will create a SQL query engine that is connected to the Postgres database using the credentials given in the set of input arguments. The (CSV) data is then converted into an iterator of dataframes using chunks of 100,000. When we loop through the iterator, we first change the datatype for any column whose name contains the term "datetime" to datetime type. Then, for the first dataframe, we will replace any table with the input argument table_name in the db database with a new table containing the first chunk of data. The remaining dataframes will then be appended to that table in chunks of 100,000 (rows). The script is below.

```python
import pandas as pd
from sqlalchemy import create_engine
from time import time
import argparse
import os

def main(params):
	user = params.user
	password = params.password
	host = params.host
	port = params.port
	db = params.db
	table_name = params.table_name
	url = params.url
	
	csv_name = "output.csv"
	
	if url[-2:] == "gz":
		file_name = url.split("/")[-1]
		os.system(f"wget {url} -O {file_name}")
		os.system(f"gunzip -c {file_name} > {csv_name}")
	else:
		os.system(f"wget {url} -O {csv_name}")

	engine = create_engine(f"postgresql://{user}:{password}@{host}:{port}/{db}")
	engine.connect()

	df_iter = pd.read_csv(
		csv_name,
		iterator=True,
		chunksize=100000,
		)
	count = 0
	for df in df_iter:
		for col in df.columns:
			if "datetime" in col:
				df[col] = pd.to_datetime(df[col])
		t_start = time()
		
		if count == 0:
			df.to_sql(table_name, con=engine, if_exists="replace")
		else:
			df.to_sql(table_name, con=engine, if_exists="append")
		
		t_end = time()
		print(f"Inserted another chunk..., took {t_end - t_start:.3f} seconds")
		count += 1

if __name__ == "__main__":
	parser = argparse.ArgumentParser(description="Ingest CSV data to Postgres")
	parser.add_argument("--user", help="user name for postgres")
	parser.add_argument("--password", help="password for postgres")
	parser.add_argument("--host", help="host for postgres")
	parser.add_argument("--port", help="port for postgres")
	parser.add_argument("--db", help="database name for postgres")
	parser.add_argument("--table_name", help="name of table results will be written to")
	parser.add_argument("--url", help="url of the csv file")
	
	args = parser.parse_args()

	main(args)
```

Now some of the packages we imported come standard with any Python distribution (i.e., os, time, argparse) but we need to pip install pandas and sqlalchemy in our container image. Similarly, the "wget" command we use does not come standard and so it will need to be installed. Furthermore, we will update the Dockerfile of our container image to copy and run ingest_data.py instead of pipeline.py. Here is the updated Dockerfile:

```dockerfile
FROM python:3.9

RUN apt-get install wget
RUN pip install pandas sqlalchemy psycopg2

WORKDIR /app
COPY ingest_data.py ingest_data.py

ENTRYPOINT [ "python", "ingest_data.py" ]
```

Now when we create our container image in Docker, will will run:

```console
docker build -t taxi_ingest:v001 .
```

The image is now called taxi_ingest with tag v001. Now we can spin up an instance of the container, but be sure to specify the network as we have done before:

```console
docker run -it \
	--network=pg-network \
	taxi_ingest:v001 \
		--user=root \
		--password=root_password \
		--host=pg-database \
		--port=5432 \
		--db=ny_taxi \
		--table_name=yellow_taxi_data \
		--url=https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz
```
# **Running pgAdmin and postgres with Docker Compose**

(Docker) Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a single YAML configuration file to configure your application's services. Then with a single command you create and start all the services from your configuration file.

To start, create a file called docker-compose.yaml in our working directory. There should be a pink whale icon to the left of the file name if you are using VS Code. First we will specify our two services: pgdatabase and pgadmin. For each service, we specify the image (postgres:13 and dpage/pgadmin4, respectively). Further, we specify all the environment variables that we had when we ran each of those services in separate containers. For the volumes, compose does not require a full (directory) path so we can use "." to represent the current directory. We do need to specify read/write privileges with :rw at the end of the container's file path.

```yaml
services:
	pgdatabase:
		image: postgres:13
		environment:
			- POSTGRES_USER=root
			- POSTGRES_PASSWORD=root_password
			- POSTGRES_DB=ny_taxi
		volumes:
			- "./ny_taxi_postgres_data:/var/lib/postgresql/data:rw"
		ports:
			- "5432:5432"
	pgadmin:
		image: dpage/pgadmin4
		environment:
			- PGADMIN_DEFAULT_EMAIL=admin@admin.com
			- PGADMIN_DEFAULT_PASSWORD=pgadmin_password
		ports:
			- "8080:80"
```

The awesome part is when we use Compose, these services will automatically run within the same network.

When using Compose, we do not need a two-step build/run process. Instead, to spin up instances of the containers in the Docker Compose file, just run the following command in the directory where the docker-compose.yaml file lives:

```console
docker compose up
```

This will search for the file with the exact name docker-compose.yaml and spin up the containers specified in the configuration file. You can use the d flag (d for detach) in order to return to the terminal once the containers are finished starting up.

```console
docker compose up -d
```

![[attachments/Screenshot 2024-01-23 at 12.26.40 PM.png]]
In order to stop the containers, we can run the command:

```console
docker compose down
```
# Port mapping and networks in Docker

