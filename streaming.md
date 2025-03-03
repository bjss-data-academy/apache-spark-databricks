# Streaming Data
Up to now, data has been a big old pile of static values that we process as a single unit. This is batch processing. It is useful for, well - batches of data. Things like sales reports that come in at the end of a working day.

A lot of real world data isn't like that. It happens in real time. It is constantly changing. 

This data is said to be _streaming_. A stream of data trickles through our system constantly.

Spark provides easy to use tools to work with streaming data. Best of all, they are based on the familiar dataframe based tools we already know.

## Use Cases for streaming
Processing data as it happens can be important:

- Real-time scoring systems for sports
- Sentiment analysis on a messaging platform
- Intrusion Detection based on network traffic
- IoT sensors. For example, weather station data

> Streaming is useful wherever real-time analytics would be an advantage over delayed batch processing

## Spark support for streaming
Spark handles streaming data by treating it as _micro-batches_. 

Data is received and batched up. At frequent intervals, this small batch of data is appended to rows in a dataframe:

![Stream of data appends rows to an unbounded table](/images/streaming-table.png)

This table is unbounded, allowing for many rows.

## Inputs
Data can be received from a number of sources by Spark:

- __[Apache Kafka](https://kafka.apache.org/)__ The popular event-handling software 
- __Event hubs__ can be wired up
- __Files__ can be scanned for changes and ingested

Spark provides some input sources aimed at unit testing and manual debugging. These are:
- __Sockets__ Regular TCP sockets can serve as input sources
- __Generator__ a function can generate data

### Trigger types - when does Spark process this data?
We've got the latest small batch of data into Spark - now what?

We must choose a _trigger condition_ for when our Spark transform code runs. 

Choose from:
- _Default_: Run the code as soon as the last code run completes
- _Fixed interval_: Run the code at fixed time intervals. Example: every 5 minutes
- _Available Now_: Run the code against all the micro-batches available now, then stop
- _Continuous Processing_: Long running cycle of fetch data, process, output and repeat
  
### Outputs
Once our code has transformed the data, we need to send the results somewhere.

Options are:
- _Kafka_: Post a Kafka message to be processed there
- _Event hub_: Raise an event on some event hub
- _Files_: Write a file containing the results
- [_Foreach_](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html): Apply a function to each row, possibly writing in a unique way

For debug and test purposes, Spark can also output to _Console_ and _Memory_.

### Output modes
There are three different approaches to writing output:

![Output modes listed](/images/streaming-output-modes.png)

The modes differ in how the existing data is treated. `Append` adds to it. `Update` overwrites it. `Complete` rewrites it.

## Fault tolerance features
Things can go wrong with real-time data capture. Spark builds in some fault-tolerance features.

### Checkpointing and write-ahead logs
Before Spark makes a change to some data, it writes a log entry saying what change it is about to make. This is a _write-ahead log entry_.

If there is a problem after this, the log entry will persist. Spark can then recover from the problem, and have a record of the data that needs to be included.

### Idempotent sinks
_Idempotence_ means that an operation can be applied multiple times without changing the end result.

Many things are not idempotent. Say I want to pay only £10 into my bank, but I accidentally hit the `pay #10` buttone twice. £20 will go into my bank - not what I wanted to do at all. We need to make this button idempotent, so that we can press it as many times as we like, and still only have the bank balance go up by £10.

Spark adds idempotence to write operations. This avoids accidental duplication of results data.

### Replayable data sources
What happens if real-time data fails to be captured?

Normally, that data is simply lost forever. We were looking the other way at the critical moment.

Spark adds some features to allow a data source to be _replayed_. We can ask for the data to be repeated to us.

## Aggregations
We have seen how dataframes support [aggregate operations](/transforming-data.md) like sum and average.

These are very useful to run over streaming data sources. An example would be finding the total number of website visits in the last hour from web server logs.

Spark makes aggregating streaming data easy to do. To us, it looks like any other dataframe operation. Behind the scenes, Spark tracks which data values were included in the last run, and accounts for that.



Aggregations
Windows
Watermarking

## Windows
Time-Based Windows
Tumbling Windows Sliding Windows
No window overlap
Any given event gets
aggregated into only one
window group
e.g. 1:00–2:00 am, 2:00–3:00
am, 3:00-4:00 am, ...
Windows overlap
Any given event gets
aggregated into multiple
window groups
e.g. 1:00-2:00 am, 1:30–2:30 am,
2:00–3:00 am, ...

sliding windows example
- make graphic
  
## code
(streamingDF
.groupBy(col("device"),
window(col("time"), "1 hour"))
.count())

### Note on Partitions and stremaing

200 default - bad
change to something better

```python
spark.conf.set("spark.sql.shuffle.partitions", spark.sparkContext.defaultParallelism)
```

## Event-Time Processing
EVENT-TIME DATA
Process based on event-time
(time fields embedded in data)
rather than receipt time

### WATERMARKS
Handle late data and limit how
long to remember old data

```python
(streamingDF
.withWatermark("time", "2 hours")
.groupBy(col("device"),
window(col("time"), "1 hour"))
.count()
)
```

# Further Reading

- [Spark Streaming Guide](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)

# Next
[Back to Contents](/contents.md)
