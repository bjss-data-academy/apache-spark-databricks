# Partitions, Skew and Shuffles
Databricks works by dividing data up into chunks called _partitions_. That data is read into memory and operations work on the in-memory data. This concurrent processing results in high performance. 

## Partitions
Partitions are logical separations of the dataset into smaller chunks, often going into DataFrames.

Partitioning data allows it to be processed in parallel.

We can partition data based on a _partition key_, which is typically the values in a column. An example would be to partition sales data based on destination country.

## Liquid Clustering
Liquid Clustering automates partitioning of data. 

We choose a column to use to decide on the splits of data, then leave Liquid Clustering to automate the details of that split.

## Shuffle
A shuffle happens when data needs to move between different compute nodes. It involves network traffic, which is a slow operation.

## Skew
Skew refers to unevenly sized partitions:

![Skewed partitions](/images/skew.png)

When we partition on a column that has a larger number of some values than others, we will see skew. 

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
