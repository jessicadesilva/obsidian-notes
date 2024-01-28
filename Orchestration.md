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


