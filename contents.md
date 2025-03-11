# Contents
- [Overview](/README.md)

## Working with Notebooks
- [Using Notebooks](/notebooks.md)
- Magic commands
- Selecting a Programming language
- Writing formatted text using markdown
- Working with the file system
- Executing shell commands
- Installing Python libraries
- Entering Spark SQL
  
## Spark DataFrame
- [Overview of DataFrames](/dataframe.md)
- Benefits of dataframes
- Reading data into a dataframe
- Displaying the schema
- Displaying content
- Using SQL-like methods
- Working with columns
- Writing data from a dataframe

## Transforming data using DataFrames
- [Aggregating Data](/transforming-data.md)
- How dataframes support medallion architecture
- Select
- Filter/Where
- Limit
- OrderBy/Sort
- Distinct
- Join
- Union
- Complex data types
- Struct
- Navigating struct columns
- Array
- Explode array to rows
- Pivot rows to columns
- Map
- Explode map to rows
- Combine rows with collect_list
- Combine rows with collect_set
- Remove duplicates with array_distinct
- Flatten
- Wide and Narrow transformations
- Lazy Evaluation
- Labs
  
## Working with Spark SQL
- [Overview of Spark SQL](/spark-sql.md)
- Convert Dataframe to temporary view
- Exploring data
- Spark Extensions to SQL
- Creating tables
- Spark DDL extensions
- Handling existing tables
- Converting data files into tables
- Auto-incrementing identity columns
- Using CTAS - Create Table As Select
- Creating External tables
- Working with Unity Catalog and Schemas
- Using SQL in Python

## Data Management in Databricks
- [Data Management in Databricks](/architecture.md)
- Unity Catalog
- Schema
- Tables
- Views
- Volumes
- Functions
- AI Models
- Managed and External tables
- Delta tables
- Transaction Log
- Data Governance in Databricks
- Three level governance

## Spark Execution Architecture
- [Visualising Spark execution architecture](/executors.md)
- Driver
- Worker
- Node
- Executors
- Job
- Stage
- Task
- Partition

## Partitions, skew and shuffles
- [Partitions](/partitions.md)
- Liquid Clustering
- Shuffle
- Skew
- Spill
- Performance Features
- Liquid Clustering
- Predictive Input/Output
- Lazy Evaluation
- Adaptive Query Execution
- Deletion Vectors
- Vacuum
- Optimize
- Photon Acceleration

## Functions
- [Built-in functions](/functions.md)
- User Defined Functions
- Performance ranking
- How Spark executes functions
- Streaming

## Working with Streaming Data
- [Working with streaming data](/streaming.md)
- Use cases for streaming
- Spark support for streaming
- Streaming input sources
- Trigger types - when does Spark process this data?
- Streaming output sinks
- Aggregations over streaming data
- Time-based Windows
- Event time processing
- When data turns up late
- Fault tolerance features
- Shuffle partition optimisation
  
## Unit testing transformations
- [What are unit tests?](/spark-unit-testing.md)
- Arrange: create in-memory dataframes
- Act: call Python transform function
- Assert: against expected result
- FIRST Tests
- Component tests
- Test-First: Requirements as Code

## Certification Resources
- [Studying for Certificates](/certification.md)
- Databricks study guide
- Official past paper source




