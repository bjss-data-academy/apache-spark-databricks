## Glossary of terms

- __Driver__ Coordinates the execution of the job and schedules tasks
- __Job__ Overall Spark application execution
- __Worker__ A computing node in the cluster where executors run
- __Executor__ Runs tasks and stores data
- __Stage__ The division of work within a job
- __Task__ The smallest unit of work, executed on a data partition
- __Partition__ one chunk, possibly of many, of a data set
- __Shuffle__ moving data from one partition to another
- __Skew__ certain paritions have more data than others due to the distribution of values
- __Spill__ Once data outgorws the available memory it must spill over onto disk storage
- __Dataframe__ in-mempry object manipulating a partition of data for processing
- __Inner Join__ combining two dataframes on a matching value in a shared column
- __Left Join__ combining the entire contents of one dataframe with matching rows in another, based on a shared column
- __In Memory__ data is stored in fast system RAM where it is quicker to process (opposed to on disk)
