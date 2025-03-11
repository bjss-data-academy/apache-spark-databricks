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

## Skip index

## Adaptive Query Execution (AQE)

## Preditive Input/Output

## Lazy Evaluation
Queries are not computed in the same sequence that we code them in. 

Instead, certain trigger actions will start evalutaion of a chain of transformations. This works alongside Adaptive Query Execution to plan the most efficient execution plan for a query.

As a thought experiment, consider this query:

```sql
SELECT * FROM users u, messages m WHERE
u.id = m.user_fk AND
m.unread = "TRUE"
```

If the users table has one million users, and each user has sent ten messages, it will be faster to filter by `m.unread = "TRUE"` before doing the table join. That way, we are not joining one million times ten rows, only to filter out most of them.

This position would reverse if we had ten users and one million messages each.

The Adaptive Query Execution algorithm can factor in such facts and more, to provide measurable performance gains.

## Photon Acceleration
Photon Acceleration is a term given to Databricks' native C++ execution engine. It contains several optimisations. The code uses advanced C++ language features to make computing cycles work harder for us.

# Next
Next topic: Understanding Built-in functions and user-defined functions

[Functions](/functions.md)

[Back to Contents](/contents.md)
