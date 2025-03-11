# Partitions, Skew and Shuffles
Databricks works by dividing data up into chunks called _partitions_. That data is read into memory and operations work on the in-memory data. This concurrent processing results in high performance. 

## Partitions
Partitions are logical separations of the dataset into smaller chunks, often going into DataFrames.

Partitioning data allows it to be processed in parallel.

We can partition data based on a _partition key_, which is typically the values in a column:

![Partitioning staff by job function](/images/partition.png)

In the previous image, we have a list of all staff along with their job function. We chunk that up into partitions by their job function.

## Shuffle
A shuffle happens when data needs to move between different compute nodes. It involves network traffic, which is a slow operation.

## Skew
Skew refers to unevenly sized partitions:

![Skewed partitions](/images/skew.png)

When we partition on a column that has a larger number of some values than others, we will see skew. In the example, we are partitioning on `job_function` and we see a skewed partition for _Academy Team'.

Skew can be a problem that slows performance. Instead of processing being evenly split across compute resources, the partition with the most data has to do the most work.

The end result cannot be known until the _last_ result comes in. Processing a skewed data set will take longer than processing a balanced one.

## Spill
When a compute resource has too little memory to fit all the data, data must _spill_ onto disk:

![Data spilling onto disk](/images/spill.png)

Spill is a problem becomes disk storage is much slower to access than system memory. 

This slow access time results in slow data processing. The code cannot go any faster than the data source can keep up.

# Performance
Databricks adds features to boost performance. Many of these work right out of the box, without requiring code.

## Liquid Clustering
Liquid Clustering automates partitioning of data. 

We choose a column to use to decide on the splits of data, then leave Liquid Clustering to automate the details of that split.

## Preditive Input/Output
Predictive I/O uses machine learning to figure out the best way to access data. It's part of the Photon Engine set of optimisations and can accelerate reading data and writing data updates.

## Lazy Evaluation
Queries are not computed in the same sequence that we code them in. 

Instead, certain trigger actions will start evalutaion of a chain of transformations. This works alongside Adaptive Query Execution to plan the most efficient execution plan for a query.

## Adaptive Query Execution (AQE)
Adaptive Query Optimisation happens at runtime, when Databricks has information about the specific dataset partitions sizes, and number of shuffles. AQE can work out an efficient execution plan for queries using this information.

AQE is enabled by default and provides four main features:

- Dynamically changes sort merge join into broadcast hash join
- Dynamically coalesces partitions (combines partitions where this would improve throughput)
- Dynamically handles skew in sort merge join and shuffle hash join by splitting (and replicating if needed) skewed tasks into roughly evenly sized tasks
- Dynamically detects and propagates empty relations.

AQE looks for operations that would be slow, and makes adjustments to improve performance.

## Deletion Vectors
Deletion vectors are a storage optimisation feature, improving the performance of deleting rows. 

Normally, deleting a row would involve rewriting the entire file. A _Deletion Vector_ instead marks the row as "not to be used anymore". This is faster than rewriting the table.

Queries can pick up on this and not include the row. 

The file can be updated and rewritten at a convemient time using the `VACUUM` or `OPTIMIZE` commands.

### VACUUM
Removes data marked for deletion.

### OPTIMIZE
Rewrites data files to improve data layout for Delta tables. 

For tables with liquid clustering enabled, OPTIMIZE rewrites data files to group data by liquid clustering keys. For tables with partitions defined, file compaction and data layout are performed within partitions.

## Photon Acceleration
Photon Acceleration is a term given to Databricks' native C++ execution engine. It contains several optimisations. The code uses advanced C++ language features to make computing cycles work harder for us.

# Further Reading
- [Liquid Clustering](https://docs.databricks.com/aws/en/delta/clustering)
- [Adaptive Query Execution](https://docs.databricks.com/aws/en/optimizations/aqe)
  
# Next
Next topic: Understanding Built-in functions and user-defined functions

[Functions](/functions.md)

[Back to Contents](/contents.md)
