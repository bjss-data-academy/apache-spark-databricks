# Spark Execution Architecture

To get the best performance from Spark, we need to take advantage of how it operates on large data sets.

Spark is designed to work with large volumes of data in a cloud-native environment. It will manage concurrent processing of data in a cluster of servers, and collection of results into a single output.

Let's review all the various pieces that make up a Spark cluster:

![Spark execution architecture](/images/executors.png)

- **Driver** Coordinates the execution of the job and schedules tasks
- **Job** Overall Spark application execution
- **Worker:** A computing node in the cluster where executors run
- **Executor:** Runs tasks and stores data
- **Stage:** The division of work within a job
- **Task:** The smallest unit of work, executed on a data partition


## Driver and worker nodes
As Databricks processes data in parallel across a cluster of computers, something needs to coordinate the work.

A _Node_ is a _compute resource_ - something that can independently run code. It may be a process on a single computer, or it may be hosted on a different computer in the cluster. 

The _Driver_ node is a single node that coordinates all activity a charge. It is responsible for sending chunks of data out to compute nodes and collecting the results.

A _Worker_ node performs part of the data processing for our Spark application. There are typically many worker nodes, each running concurrently.

## Executors
The code that does the work runs inside an executor. Executors hold the data in memory (or local disk) for the code to work on. 

## Job, Stage and Task
An executor runs one or more stages. A _stage_ is a set of tasks that can run concurrently. 

A _task_ is the smallest unit of work in a Spark application. This is where the proverbial rubber hits the road and numbers get crunched. A task is executed on a partition of the data, and there is one task for each partition.


Xxxxxxxxxxxx

be executed in parallel within a Spark job. Stages are created based on the **shuffle boundaries**. If a shuffle (e.g., a `groupByKey`, `join`, etc.) is required between operations, it will cause a stage boundary.
- **Role:** Stages break down a job into smaller units of work. Each stage can be split into multiple tasks that run in parallel across the cluster.
- **Example:** If you have an operation like a `groupBy` or `join`, Spark will divide the job into multiple stages depending on whether a shuffle is required between operations.

### 5. **Task**
- **Definition:** A **task** is the smallest unit of work in a Spark job. A task is executed on a partition of the data, and there is one task for each partition.
- **Role:** Tasks are distributed across the cluster's workers and executors. The number of tasks corresponds to the number of data partitions for the stage being executed.
- **Example:** If you're processing a dataset with 100 partitions, Spark will create 100 tasks to process that data in parallel, assuming sufficient worker resources.

### 6. **Job**
- **Definition:** A **job** represents the entire computation that Spark runs. A job consists of one or more stages, which in turn are made up of tasks. A job is triggered by an action (e.g., `collect()`, `save()`, `count()`).
- **Role:** The job is the highest-level unit of work. It is the complete execution flow from the start to the end of the Spark application.
- **Example:** If you're running a `df.filter().groupBy().agg()` operation in a Databricks notebook, Spark will convert this operation into one or more jobs, depending on the number of transformations and actions involved.

---

The boundary between two stages is drawn when transformations cause data shuffling across partitions. 

Transformations in Spark are categorized into two types: narrow and wide. 

Narrow transformations, like map(), filter(), and union(), can be done within a single partition. 

wide transformations like groupByKey(), reduceByKey(), or join(), data from all partitions may need to be combined, thus necessitating shuffling and marking the start of a new stage.


- **Driver:** The process that coordinates the execution of the job and schedules tasks.-
- **Job:** The overall Spark application execution.
- **Worker:** The node in the cluster where executors run.
- **Executor:** The process that runs tasks and stores data.
- **Stage:** The division of work within a job, separated by shuffle operations.
- **Task:** The smallest unit of work, executed on a data partition.



