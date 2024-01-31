# Architecture

1. Extract
	* Pull data from source (API - NYC Taxi dataset)
2. Transform
	* Data cleaning, transforming, and partitioning
3. Load
	* API to Mage, Mage to (Postgres, GCP, BigQuery)

# What is orchestration?
* A large part of data engineering is **extracting**, **transforming**, and **loading** data between sources.
* **Orchestration** is a process of dependency management, facilitated through **automation**.
* The data orchestrator manages scheduling, triggering, monitoring, and even resource allocation.
* Every workflow requires sequential steps
	* e.g. a french press with cold water will only brew disappointment
	* In a similar way, poorly sequenced transformations brew (haha) a storm far more bitter.
* Vocabulary
	* Steps -> Tasks
	* Workflows -> DAGS (directed acyclic graphs) or pipeline
* Image from *Fundamentals of Data Engineering* book: orchestration is deemed an *undercurrent* of the data engineering lifecycle in this framework which implies it is happening throughout the entire process

![[Screenshot 2024-01-28 at 9.01.36 AM.png]]
# What are properties of *good* orchestration solutions?
* A **good** orchestrator handles...
	* workflow management: (1) define, schedule, manage workflows efficiently; (2) ensure tasks are executed in the right order; (3) manages dependencies
	* automation
	* error handling: built-in solutions for handling errors using conditional logic, branching, and retrying failed tasks
	* recovery: e.g., if data is lost there needs to be a way to backfill / recover missing data
	* monitoring and alerting: if pipeline fails or retries are necessary the orchestrator solution should alert the owner of the pipeline
	* resource optimization: if the orchestrator is managing where jobs are executed, it should optimize the route for execution
	* observability: visibility into every part of the pipeline
	* debugging: should allow for debugging easily
	* compliance / auditing
* A **good** orchestrator prioritizes...
	* The *developer experience*
		* Flow state: not constantly switching between numerous tools and services
		* Feedback loops: getting immediate feedback while testing
		* Cognitive load: how complicated is it to use the tool?
* A good orchestration tool accomplishes all of the data engineering tasks with rapid and seamless data pipelines
# What is Mage?
An open-source pipeline tool for orchestrating, transforming, and integrating data.
* An **instance** can contain multiple **projects**.
* A **project** contains several **pipelines** (DAGs / data workflows).
* A **pipeline** is comprised of **blocks** which are the atomic units that make up a transformation in Mage. **Blocks** can be written in Python, SQL, or R and are typically used to export, transform, or load data.
* Block type examples:
	* Sensor blocks (trigger on some event)
	* Conditional blocks (branching / if-else logic)
	* Dynamic blocks (create dynamic children)
	* Webhook blocks, etc.
* Allows for the passing of objects between the pipeline (called unified pipelines)
* How does Mage accelerate pipeline development?
	* Hybrid environment
		* Use a GUI for interactive development (or not, you can just use an IDE)
		* Use blocks as testable, reusable pieces of code
	* Improved developer experience
		* Code and test in parallel.
		* Reduce your dependencies, switch between tools less, be efficient.
* Engineering best-practices built-in
	* In-line testing and debugging in a notebook-style environment
	* Fully-featured observability with transformations in one place: dbt models, streaming, etc.
	* DRY (Don't Repeat Yourself) principles by minimizing DAGs that have duplicate functionality and strange imports
* Reduce time in undifferentiated work (setup, configuration, etc.) and focuses instead on something that produces a tangible outcome.
* **Projects**
	* A project forms the basis for all the work you can do in Mage -- think of it like a GtHub repository.
	* It contains the code for all of your pipelines, blocks, and other assets.
	* A Mage instance has one or more projects.
* **Pipelines**
	* A pipeline is a workflow that executes some data operation -- maybe extracting, transforming, and loading data from an API (called *DAGs on other platforms*).
	* In Mage, pipelines can contain **Blocks** (written in SQL, Python, or R) and **charts**.
	* Each pipeline is represented by a YAML file in the "pipelines" folder of your project.
		* With this, you can dynamically create pipelines or template them.
* **Blocks**
	* A block is a file that can be executed independently or within a pipeline.
	* Together, blocks form Directed Acyclic Graphs (DAGs), which we call *pipelines*.
	* A block won't start running in a pipeline until all of its upstream dependences are met.
	* Blocks are reusable, atomic pieces of code that perform certain actions.
	* Changing one block will change it everywhere it's used, but **don't worry**, it's easy to detach blocks to separate instances if necessary.
	* Blocks can be used to perform a variety of actions, from simple data transformations to complex machine learning models.
	* Structure of a block:
		* Imports
		* Decorator
		* Definition of a function that **returns a dataframe** (this is required)
		* Test / assertion that is ran on the output dataframe of the block
	* The only thing that is getting returned when running the block is the returned dataframe from the function; which will be passed on downstream.
# Configuring Mage
1. Clone the GitHub repo at this URL:
```bash
https://github.com/mage-ai/mage-zoomcamp
```
2. In your terminal, navigate to the cloned repo directory.
3. Change the name of the file dev.env to just .env so that it is not committed to your repo.
```bash
cp dev.env .env
```
4. Build the Docker image using the included Docker Compose file.
```bash
docker compose build
```
Note that if you some of the environment variables defined in the .env file were also set previously via the shell, Docker Compose uses environment variable values according to the following precedent list:
	1. Set using ```docker compose run -e``` in the command line interface (CLI).
	2. Substituted from your shell.
	3. Set using the environment attribute in the Compose file.
	4. Use of the ```--env-file``` argument in the CLI.
	5. Use of the ```env_file``` attribute in the Compose file.
	6. Set using an ```.env``` file placed at the base of your project directory.
	7. Set in a container image in the ENV directive.
So, as you can see, the .env file is pretty low on the precedent list and so if changing values in the .env file does not correspond to changes when you use docker compose build/up then it is likely because those variables are defined using a method earlier in this precedent list.

To get an updated version of the Docker image, you can use the command:
```bash
docker pull mageai/mageai:latest
```
and then rebuild the image.
5. Initialize a container using the Docker image:
```bash
docker compose up
```
6. In your browser, navigate to localhost:6789 to get to your Mage instance. If you inspect the port matching in the Docker compose file under the magic service you will see we are mapping the port for this service (6789) to our host computer's port (6789).

## Docker compose inspection

We see that there are two services in our Docker compose file:
1. Magic (our Mage instance)
	* This service is built off the Docker file included in the repo.
	* The Docker file also installs Python packages listed in the requirements.txt file.
	* We will be able to access our Mage instance using port 6789.
1. Postgres
	* The .env file contains credentials for logging into Postgres as well as which port we want it to use on our host machine.

As mentioned previously, for both services we specify that our environment variables (which often include credentials) are stored in the .env file with 
```YAML
env_file:
	- .env
```

# Navigating Mage

In the upper left corner we can see the name of our project ```magic-zoomcamp``` and by default we are taken to the list of pipelines within this project.

![[Screenshot 2024-01-30 at 7.05.30 PM.png]]

When we click on ```example_pipeline``` we are taken to the Triggers section of our pipeline. While in Triggers (as well as the Edit Pipeline section) we can see the basic structure of the pipeline:
* Our pipeline has three blocks: load_titanic, fill_in_missing_values, export_titanic_clean
* The dependencies of the blocks are that load_titanic must run successfully before fill_in_missing_values and fill_in_missing_values must run successfully before export_titanic_clean. Using airflow syntax this would be:
  load_titanic << fill_in_missing_values << export_titanic_clean
  * The dependencies also imply that the dataframe output of load_titanic gets fed in as an input to fill_in_missing_values, and the dataframe output of fill_in_missing_values gets fed in as an input to export_titanic_clean.
  * The load_titanic block is a "data loader" block written as a python (PY) file 
  * The fill_in_missing_values block is a "transformer" block also written as a python (PY) file
  * The export_titanic_clean block is a "data exporter" block also written as a python (PY) file
![[Screenshot 2024-01-30 at 7.07.47 PM.png]]

When you navigate to the Edit Pipeline section in the left navigation, on the left panel you will see a file tree of all the files in the project:
![[Screenshot 2024-01-30 at 7.13.40 PM.png]]
Notice that all of these files also exist in VSCode (because we mapped folders to each other in Docker compose via volumes) and so if you want to make changes to, for instance, code for certain blocks you can do that either in the Mage GUI or just in VSCode.

If you switch over to the Current Blocks tab, you will see a list of the code files for the blocks that are within the specific pipeline we are inspecting.
![[Screenshot 2024-01-30 at 7.18.37 PM.png]]
In the middle panel, you will see the code for the blocks in the specific pipeline we are looking at. For example, the file at the top is the code for the "data loader" block in our pipeline:
![[Screenshot 2024-01-30 at 7.15.44 PM.png]]
It is very important to remember that code for blocks are reused throughout the entire project, so changes in the code for this block will affect any other pipelines that use this block. There are ways around that, but it is important to remember that this is how it works by default.

The play button in the upper right corner of each code block will only run that single block. If that block depends on another which has not yet been ran, you will get an error when you try to run the block. Instead, you can select the three dots within the circle to show more options and select the ```Execute with all upstream blocks``` option to run any blocks this one depends on (and others which those depend on, and so on) in order and then finally run this block.

The code templates offered typically have the function definition (preceded by a decorator such as ```@data_loader```) as well as a test function. The test function is not required and can be deleted.

While in the Edit Pipeline section, you can change the dependences by deleting/adding connections between blocks on the right panel. To do this, right click a connection and select the option to either add or delete.

If there is a line which is connected from Block_A to Block_B where Block_A is above Block_B, that implies that the dataframe returned from Block_A is fed in as an input to Block_B.
![[Screenshot 2024-01-30 at 7.29.39 PM.png]]
The image above shows that the load_titanic code must run before fill_in_missing_values and as the dataframe output from load_titanic is fed in as the input to fill_in_missing_values.

If we remove that connection, we can add a new one that reverses the relationship described by choosing fill_in_missing_values as the first endpoint of the connection and then choosing load_titanic as the second endpoint. Mage will automatically vertically order/group the blocks once you select the endpoints of the connection.
![[Screenshot 2024-01-30 at 7.36.05 PM.png]]
By reversing the relationship, we can see in the diagram above that the output of fill_in_missing_values is going to be fed into two blocks now: load_titanic and export_titanic_clean.

# Configuring Postgres

If you inspect the io_config.yaml file in our project directory, you will see that we can configure **profiles** and there already exists a **default** profile:
![[Screenshot 2024-01-30 at 7.49.08 PM.png]]

We can create a developer profile (dev) to separate developer and production credentials within this file. The developer profile will use the credentials specified in the .env file that Docker is aware of. This file uses Jinja templating which you will see whenever there is use of the double curly braces to fill in variable values:

```YAML
dev:
	# PostgresSQL
	POSTGRES_CONNECT_TIMEOUT: 10
	POSTGRES_DBNAME: "{{ env_var('POSTGRES_DBNAME') }}"
	POSTGRES_SCHEMA: "{{ env_var('POSTGRES_SCHEMA') }}" # Optional
	POSTGRES_USER: "{{ env_var('POSTGRES_USER') }}"
	POSTGRES_PASSWORD: "{{ env_var('POSTGRES_PASSWORD') }}"
	POSTGRES_HOST: "{{ env_var('POSTGRES_HOST') }}"
	POSTGRES_PORT: "{{ env_var('POSTGRES_PORT') }}"
```

# Question
Perhaps there is a misuse of environment variables here. My understanding is POSTGRES_PORT is the port on the host machine, but I believe Mage is looking for the port on the container?

Let's create a (batch) pipeline from scratch by going to File --> New standard pipeline.

![[Screenshot 2024-01-30 at 7.54.10 PM.png]]

Let's start by changing the name of our pipeline to ```test_config``` within the pipeline settings via Edit --> Pipeline settings:
![[Screenshot 2024-01-30 at 7.55.49 PM.png]]
![[Screenshot 2024-01-30 at 7.57.22 PM.png]]
Click Save pipeline settings and head over to Edit pipeline.

To test that we have a connection to Postgres, let's create a **Data loader** block that is written in SQL.
![[Screenshot 2024-01-30 at 7.59.22 PM.png]]

Let's set the Connection to PostgreSQL.
![[Screenshot 2024-01-30 at 8.00.21 PM.png]]

Set our Profile to dev.
![[Screenshot 2024-01-30 at 8.00.50 PM.png]]

Then finally we will remove the Mage templating by selecting the Use raw SQL option.
![[Screenshot 2024-01-30 at 8.01.40 PM.png]]

Because we are just checking the connection, let's use the command 
```SQL
SELECT 1;
```
and ensure that the connection is made and 1 is returned.

# Writing an ETL pipeline

We can delete the previous test_config pipeline and start fresh with a new (batch) standard pipeline called api_to_postgres.

Let's create a Data Loader block called load_api_data written in Python with an API template:
![[Screenshot 2024-01-30 at 8.13.29 PM.png]]

We will define the url variable to be the following:
```python
url = 'https//github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_traip_data_2021-01.csv'
```
We will also tell pandas the datatypes to expect to (1) save on memory used to process the dataset, (2) if the datatypes are different from what is expected then an error will occur and the pipeline owner will be notified.

```python
taxi_dtypes = {
			   'VendorID' : pd.Int64Dtype(),
			   'passenger_count' : pd.Int64Dtype(),
			   'trip_distance' : float,
			   'RatecodeID' : pd.Int64Dtype(),
			   'store_and_fwd_flag' : str,
			   'PULocationID' : pd.Int64Dtype(),
			   'DOLocationID' : pd.Int64Dtype(),
			   'payment_type' : pd.Int64Dtype(),
			   'fare_amount' : float,
			   'extra' : float,
			   'mta_tax' : float,
			   'tip_amount' : float,
			   'tolls_amount' : float,
			   'improvement_surcharge' : float,
			   'total_amount' : float,
			   'congestion_surcharge' : float
			}
```

We will also have pandas parse the two datetime columns: tpep_pickup_datetime and tpep_dropoff_datetime. The code for our entire block is as follows.

```python
import io
import pandas as pd
import requests

if 'data_loader' not in globals():
	from mage_ai.data_preparation.decorators import data_loader

if 'test' not in globals():
	from mage_ai.data_preparation.decorators import test

@data_loader
def load_data_from_api(*args, **kwargs):
	url = 'https//github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz'

	taxi_dtypes = {

		'VendorID' : pd.Int64Dtype(),
		'passenger_count' : pd.Int64Dtype(),
		'trip_distance' : float,
		'RatecodeID' : pd.Int64Dtype(),
		'store_and_fwd_flag' : str,
		'PULocationID' : pd.Int64Dtype(),
		'DOLocationID' : pd.Int64Dtype(),
		'payment_type' : pd.Int64Dtype(),
		'fare_amount' : float,
		'extra' : float,
		'mta_tax' : float,
		'tip_amount' : float,
		'tolls_amount' : float,
		'improvement_surcharge' : float,
		'total_amount' : float,
		'congestion_surcharge' : float
	}

  
	parse_dates = ['tpep_pickup_datetime', 'tpep_dropoff_datetime']

	return pd.read_csv(url,
		sep=',',
		compression='gzip',
		dtype=taxi_dtypes,
		parse_dates=parse_dates)

@test
def test_output(output, *args) -> None:
	assert output is not None, 'The output is undefined'
```

Now that we have loaded the data, let's go ahead and apply a transformer. This transformer will drop all rows that have 0 passengers. First, let's create a transformer block called transform_taxi_data in Python without a template.
![[Screenshot 2024-01-30 at 8.41.48 PM.png]]

We will skip the details related to the code in this block as we will simply use basic pandas operations. Also note the test function at the bottom, the output will not be given to the next code block unless the test is passed.

```python
if 'transformer' not in globals():
	from mage_ai.data_preparation.decorators import transformer

if 'test' not in globals():
	from mage_ai.data_preparation.decorators import test

  
  

@transformer
def transform(data, *args, **kwargs):
	print(f"Preprocessing: rows with zero passengers: { (data['passenger_count'] == 0).sum() }")
	return data[data["passenger_count"] > 0]

@test
def test_output(output, *args) -> None:
	assert (output["passenger_count"] > 0).sum() == 0, 'There are rides with zero passengers.'
```

Finally, we will export our data to Postgres. Let's create a Data Exporter block called taxi_data_to_postgres in Python with the PostgreSQL template.
![[Screenshot 2024-01-30 at 8.54.06 PM.png]]

In the template, we need to specify the following:
* ```schema_name```
* ```table_name```
* ```config_profile```
So the only things that changed from the template was the following:
```python
schema_name = 'ny_taxi' # Specify the name of the schema to export data to
table_name = 'yellow_taxi_data' # Specify the name of the table to export data to
config_path = path.join(get_repo_path(), 'io_config.yaml')
config_profile = 'dev'
```

To check that the exporter worked, we can use a SQL Data Loader block called load_taxi_data. Don't forget to specify the Connection as PostgreSQL and the Profile to Dev and Raw SQL:

```SQL
SELECT * FROM ny_taxi.yellow_taxi_data LIMIT 10;
```