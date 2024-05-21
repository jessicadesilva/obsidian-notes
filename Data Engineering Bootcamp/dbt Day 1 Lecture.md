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
* dbt