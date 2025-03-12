## Glossary of terms

- __Azure__ Microsoft's Cloud system
- __Databricks__ Product company adding features to Apache Spark
- __Apache Spark__ Open Source library providing Data Engineering tools
- __PySpark__ Python language bindings to Apache Spark
- __Petabyte__ 1,000,000 Gigabytes (10 to the power 12 bytes)
- __Driver__ Coordinates the execution of the job and schedules tasks
- __Worker__ A computing node in the cluster where executors run
- __Node__ A virtual computer running on a computer in a cluster
- __Cluster__ Group of computers available to run Spark analytics code and hold data
- __Serverless Compute__ Databricks automates compute cluster sizing and utilisation
- __Job__ Overall Spark application execution
- __Executor__ Runs tasks and stores data
- __Stage__ The division of work within a job
- __Task__ The smallest unit of work, executed on a data partition
- __Partition__ one chunk, possibly of many, of a data set
- __Shuffle__ moving data from one partition to another
- __Skew__ certain paritions have more data than others due to the distribution of values
- __Dataframe__ in-mempry object manipulating a partition of data for processing
- __Inner Join__ combining two dataframes on a matching value in a shared column
- __Left Join__ combining the entire contents of one dataframe with matching rows in another, based on a shared column
- __In-memory__ data is stored in fast system RAM where it is quicker to process (opposed to on disk)
- __Spill__ Once data outgrows the available memory it must spill over onto disk storage
- __Managed Table__ Relational table whose life cycle is managed by Databricks. DROP TABLE deletes all data and metadata.
- __External Table__ Relation table with data stored outside Databricks. DROP TABLE only deletes metadata, leaves data unaffected.
- __Liquid Clustering__ Databricks feature to automatically adjust data splits across partitions
- __Adaptive Query Execution (AQE)__ Databricks algorithms to optimise queries before execution
- __Photon Acceleration__ Databricks optimised native C++ query engine
- __Unit Test__ turns "should work" into "did work"
- __User Defined Function__ function we write registered for use within Spark SQL and Dataframes

# Next
[Back to Contents](/contents.md)
