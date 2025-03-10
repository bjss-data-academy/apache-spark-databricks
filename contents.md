# Contents

## Overview of Spark
- [Overview](/spark-overview.md)
- Spark in databricks

## Working with Notebooks
- [Using Notebooks](/notebooks.md)
- Magic commands
- Selecting a Programming language
- Writing formatted text using markdown
- Working with the file system
- Executing shell commands
- Installing Python libraries
- Working with Spark SQL
  
## Spark DataFrame
- [Overview of DataFrames](/spark-dataframe.md)
- Benefits of dataframes
- Reading data into a dataframe
- Displaying the data schema
- Displaying dataframe content
- Using SQL-like methods
- Working with columns
- Writing data from a dataframe

## Transforming data using DataFrames
- [Aggregating Data](/aggregation.md)
- Medallion architecture
- Select
- Filter/Where
- Limit
- OrderBy/Sort
- Distinct
- Join
- Union
- Complex data types
- Struct
- Dot notation
- Array
- Explode array to rows
- Pivot rows to columns
- Map
- Explode map to rows
- Collect_list
- Collect_set
- Array_distinct
- Flatten
  
## Spark SQL
- [Overview of Spark SQL](/spark-sql.md)
- Convert Dataframe to temporary view
- Basic SQL queries
- Defining a schema
- Managed versus Unmanaged tables
- Creating tables with SQL DDL
- Creating views
- Metadata: What's inside this database?
- Reading tables into dataframes
- Reading CSV into table

## Unit testing transformations
- [What are unit tests?](/spark-unit-testing.md)
- Arrange: create in-memory dataframes
- Act: call Python transform function
- Assert: against expected result
- FIRST Tests
- Component tests
- Test-First: Requirements as Code
  
## Delta tables and Parquet files
- https://delta.io/blog/delta-lake-vs-parquet-comparison/
- default in databricks
- ACID
- Time Travel / versioning
- Audit hostory
- Builds on Parquet format
- Compatible with Spark API incl Spark SQL
  
## Architecture
- [Spark Architecture](/architecture.md)
- Visualising Spark core architecture
- Cluster
- Executor
- Driver
- Task
- Stage
- Job

## Spark Performance Topics
-[Getting good performance](/spark-performance.md)

- Wide and Narrow transformations
- 
- Partitioning
- Skew
- Shuffle
- Liquid Clustering
- Deletion Vectors
- Vacuum
- Optimize
- Predictive I/O
- Adaptive Query Execution
- Photon Engine

## Certification Resources
- databricks guide
- official past paper source
