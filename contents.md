# Contents

## Overview of Spark
- [Overview](/spark-overview.md)
- Spark in databricks

## Working with Notebooks
- [Using Notebooks](/notebooks.md)
- Magic commands
- Exploring files
- Exploring tables
  
## Spark DataFrame
- [DataFrame](/spark-dataframe.md)
- Universal abstraction of dataset
- Optimised for Columns
- Compatible with Spark SQL
- Rename column
- Add column
- Supported data types
- Casting between data types
- reading and writing
- file formats

## Delta tables and Parquet files
- https://delta.io/blog/delta-lake-vs-parquet-comparison/
- default in databricks
- ACID
- Time Travel / versioning
- Audit hostory
- Builds on Parquet format
- Compatible with Spark API incl Spark SQL

## Transforming data using DataFrames
- [Aggregating Data](/aggregation.md)
- Complex data types
- Built-in functions
- User Defined Functions
- Performance ranking
- How Spark executes functions
- collect_set
- collect_list
- array_distinct
- pivot
- explode
- Wide and Narrow transformations

## Unit testing transformations
- Arrange: create in-memory dataframes
- Act: call Python transform function
- Assert: against expected result
- FIRST Tests
- Component tests
- Test-First: Requirements as Code
  
## Architecture
- [Architecture](/architecture.md)
- Visualising Spark core architecture
- Cluster
- Executor
- Driver
- Task
- Stage
- Job

## Spark Performance Topics
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
