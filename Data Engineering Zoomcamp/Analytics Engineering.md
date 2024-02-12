# What is analytics engineering?

## Context: Data Domain Developments
* Massively parallel processing (MPP) databases
	* Cloud data warehouses like BigQuery, Snowflake, or Redshift lower cost of storage and computing
* Data pipelines as a service
	* Fivetran, Stitch simplify the ETL process
* SQL-first
	* Looker
* Version control systems
* Self service analytics
	* Mode (BI tools)
* Data governance
	* Affected how data teams work with data and how stakeholders consumed the data

These developments highlighted caps in the data team.
## Roles in a data team
* **Data engineer**: prepares and maintain the infrastructure the data team needs. Typically have good software engineering skills but not the background to know how the data will be used by business users.
* **Analytics engineer**: introduces the good software practices to the efforts of data analysts and data scientists.
* **Data analyst**: uses data to answer questions and solve problems. Typically haven't been trained in engineering and don't focus their efforts on it.

## Tooling for analytics engineering
* Data loading (like Fivetran)
* Data storing (Cloud data warehouses like Snowflake, BigQuery, Redshift)
* Data modeling (Tools like dbt and Dataform)
* Data presentation (BI tools like Looker, Mode, or Tableau)

## ETL vs. ELT
* In ETL, we extract the sources, then transform it, and load it to a data warehouse.
	* Slightly more stable and compliant data analysis
	* Higher storage and compute costs
* In ELT, we transform the data after it has already been loaded to the data warehouse.
	* Faster and more flexible data analysis
	* Lower cost and lower maintenance

## Kimball's dimensional modeling
**Objective**
* Deliver data understandable to the business users
* Deliver fast query performance
**Approach**
* Prioritize user understandability and query performance over non-redundant data (3NF)
**Other Approaches**
* Bill Inmon
* Data vault

## Elements of dimensional modeling (star schema)
**Fact tables**
* Measurements, metrics or facts
* Corresponds to a business *process*
* "Verbs"
**Dimension tables**
* Corresponds to a business *entity*
* Provides context to a business process
* "Nouns"

## Architecture of dimensional modeling
**Stage Area**
* Contains the raw data
* Not meant to be exposed to everyone
**Processing area**
* From raw data to data models
* Focuses on efficiency
* Ensuring standards
**Presentation area**
* Final presentation of the data
* Exposure to business stakeholder

## What is dbt?
* dbt is a transformation workflow that allows anyone that knows SQL to deploy analytics code
* allows the user to follow software engineering best practices like modularity, portability, CI/CD, and documentation
	* Developing SQL code in something like a sandbox where each developer has their own separate environment
	* Testing and documentation
	* Deployment using version control and CI/CD
* sits on top of data warehouse to take raw data and transform it (using data modeling)

### How does dbt work?
**Each dbt model is:**
* a \*.sql file
* Select statement
	* No DDL, that is data definition language (like CREATE TABLE, ALTER TABLE, DROP TABLE)
	* No DML, that is data manipulation language (like INSERT, UPDATE, DELETE)
* A file that dbt will compile and run in our data warehouse.

### How to use dbt?
Choose between dbt Core and dbt Cloud.
**dbt Core**
* Open-source project that allows the data transformation
* Builds and runs a dbt project (.sql and .yml files)
* Includes SQL compilation logic, macros and database adapters
* Includes a CLI interface to run dbt commands locally
* Open source and free to use
**dbt Cloud**
SaaS application to develop and manage dbt projects.
* Web-based IDE and cloud CLI to develop, run and test a dbt project
* Managed environments
* Jobs orchestration
* Logging and alterting
* Integrated documentation
* Admin and metadata API
* Semantic layer

We will use dbt in two ways: Version A is dbt Cloud for BigQuery and Version B is dbt Core for Postgres.

**BigQuery**
* Development using cloud IDE
* No local installation of dbt core
**Postgres**
* Development using a local IDE of your choice
* Local installation of dbt core connecting to the Postgres database
* Running dbt models through the CLI

