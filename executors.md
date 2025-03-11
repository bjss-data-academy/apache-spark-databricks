# Spark Execution Architecture

To get the best performance from Spark, we need to take advantage of how it operates on large data sets.

Spark is designed to work with large volumes of data in a cloud-native environment. It will manage concurrent processing of data in a cluster of servers, and collection of results into a single output.

Let's review all the various pieces that make up a Spark cluster:

![Spark execution architecture](/images/executors.png)


## Driver
As Databricks processes data in parallel across a cluster of computers, something needs to coordinate the work.

The _Driver_ node is a single node that coordinates all activity a charge. 

The driver is responsible for sending chunks of data out to compute nodes and collecting the results.

## Worker
A _Worker_ node performs part of the data processing for our Spark application. There are typically many worker nodes, each running concurrently.

## Node
A _Node_ is a _compute resource_ - something that can independently run code. 

It may be a process on a single computer, or it may be hosted on a different computer in the cluster. 

## Executors
The code that does the work runs inside an executor. Executors hold the data in memory (or local disk) for the code to work on. 

## Job
A _Job_ represents the entire computation that Spark runs. 

A job is triggered by an action (e.g., `collect()`, `save()`, `count()`).

- **Role:** The job is the highest-level unit of work. It is the complete execution flow from the start to the end of the Spark application.
- **Example:** If you're running a `df.filter().groupBy().agg()` operation in a Databricks notebook, Spark will convert this operation into one or more jobs, depending on the number of transformations and actions involved.

## Stage  
A _stage_ is a set of tasks that can run concurrently. 

## Task
_Tasks_ are the smallest unit of work in a Spark application. Tasks are where numbers get crunched. 

A task is executed on a partition of the data. There is one task for each partition.

## Partition
A _Partition_ is a chunk of a dataset. 

Many partitions are required to represent the extent of a large dataset.

# Next
How data is organised across partitions affects processing times. Let's learn how:

[Partitions, skew and shuffle](/partitions.md)

[Back to Contents](/contents.md)
