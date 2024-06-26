Data Build Tool (dbt) sits at the transformation stage of ELT.

![[Screenshot 2024-05-20 at 6.06.02 PM.png]]

**What is the problem dbt wants to solve?**
* Lack of testing and documentation
* Easier to re-write stored procedures code than find or fix existing code
* Analysts don't know what to trust. Hard to understand transformation code.
* Data chaos

**dbt viewpoint**
Code Reigns
* dbt is SQL-first
* data democratization
Work like engineers
* Testing
* Version Control
* DRY (Don't Repeat Yourself) code
* Documentation
* Other software engineering best practices
Data Lineage/Dependency Management

**What is dbt?**
* dbt is a tool that enables anyone comfortable with SQL to work in transformation pipelines using the best practices in software engineering
* Compiler and runner: dbt compiles SQL code and send it to your data warehouse to run it
* dbt does not store data and it has no compute power
* What dbt can do is what the Data Warehouse can do
* dbt code can be stored in your git provider for versioning
* **dbt-core**
	* open-source code
	* all core functionalities (developing/testing/documentation)
	* python package (interact through CLI)
	* requires more knowledge
* **dbt cloud**
	* cloud-managed platform
	* runs dbt-core
	* more user-friendly
	* has its own IDE
	* handles complex features like CI/CD, FBAC, environments, notifications

There are many dbt adapters, for example: AlloyDB, BigQuery, Databricks, Dremio, Postgres, Redshift, Snowflake, Spark, Starburst/Trino, Microsoft Fabric, Azure Synapse, Teradata, ...
Some of these are dbt-owned and others are community-provided.

**dbt project structure**
Resources
* Sources (references to objects in your data platform)
* Models (sql code that will create your objects)
* Tests (sql code that will test your data)
* Snapshots (SCD type 2)
* Seeds (csv files)
* ...
dbt projects are basically **.sql** and **.yml** files in hierarchical folder structure.

**Models**
* Chunks of code that are materialized as an object in your Data Warehouse
* Select statement
* Can be written in SQL or Python (limited support)
* Uses Jinja (Python templating library that extends SQL possibilities)
	* {{ ref() }} and {{ source() }}: Jinja functions used for lineage and dependency management
	* allows us to create macros
* Materializations:
	* View
	* Table
	* Incremental
	* Ephemeral
![[Screenshot 2024-05-20 at 6.27.07 PM.png]]

* Governance
	* Access
	* Version
	* Contracts

**Best practices on models**
* **staging**
	* 1:1 with sources
	* separated by source
	* light transformation
	* stg\_\[source\]\_\_\[entity\]s.sql
* **intermediate**
	* modular
	* break down complexity
	* group by area of concern
	*  w  
* **marts**