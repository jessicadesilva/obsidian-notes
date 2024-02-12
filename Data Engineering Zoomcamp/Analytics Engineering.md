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