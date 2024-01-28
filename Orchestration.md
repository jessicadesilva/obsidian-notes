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
	
	

