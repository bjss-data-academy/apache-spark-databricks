# Spark Performance
> TODO reword intro to set context

Databricks works by dividing data up into chunks called _partitions_. That data is read into memory and operations work on the in-memory data, resulting in high performance. 

Each parition can be procees in parallel, on a group of _compute resources_ (_'computers'_) known as a _cluster_. Using _serverless compute_ is a recent option.

There are several techniques used in splitting that data up across multiple partitions.

The basic unit of processing is the _dataframe_.

## Dataframes 
A _DataFrame_ is a chunk of data read into memory, ready for processing.

In Python, it is represented as an [object](https://github.com/bjss-data-academy/python-essentials/blob/main/06-objects.md) of class [`DataFrame`](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.DataFrame.html). It has a rich set of methods available to filter, sort, aggregate and transform data.

## Partitions
Partitions are logical separations of the dataset into smaller chunks, often going into DataFrames.

Partitioning data allows it to be processed in parallel.

We can partition data based on a _partition key_, which is typically the values in a column. An example would be to partition sales data based on destination country.

## Liquid Clustering
Liquid Clustering automates partitioning of data. 

Again, we choose a column to use to decide on the splits of data. But Databricks automates that split.

## Shuffle
A shuffle happens when data needs to move between different compute nodes. It involves network traffic, which is a slow operation.

## Skew
Skew refers to unevenly sized partitions. When we partition on a column that has a larger number of some values than others, we will see skew. 

## Spill
When a compute resource has too little memory to fit all the data, data must _spill_ onto disk. This is a slow operation.


## Wide and Narrow transformations
p19

## Lazy evaluation
p19

## Built-in functions
## User Defined Functions
## Performance ranking
## How Spark executes functions
