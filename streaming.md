# Streaming Data
Up to now, data has been a big old pile of values that we process as a single unit. This is batch processing. It is useful for, well - batches of data. Things like sales reports that come in at the end of a working day.

A lot of real world data happens in real time. It is constantly being added to. 

This data is said to be _streaming_. A stream of data trickles through our system constantly.

Spark provides easy to use tools to work with streaming data. Best of all, they are based on the familiar dataframe based tools we already know.

## Use Cases
Processing data as it hapens can be important:

- Real-time scoring systems for sports
- Sentiment analysis on a messaging platform
- Intrusion Detection based on network traffic
- IoT sensors. For example, weather station data

## Spark structured streaming
Behind the scenes, Spark handles streaming data by treating it as _micro-batches_. 

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

### Trigger types
- Default
- Fixed interval
- Available now
- Continuous

Process each micro-batch as soon as the previous
one has been processed

Fixed interval Micro-batch processing kicked off at the
user-specified interval

Available now
Process all of the available data in multiple
micro-batches and then automatically stop the query

Continuous
Processing
Long-running tasks that continuously read, process,
and write data as soon events are available
*Experimental See Structured Streaming Programming Guide
  
### Outputs

- Kafka
- Event hub
- Files
- Foreach

For debug/test

- Console
- Memory

### Output modes

APPEND
Add new records
only
UPDATE
Update changed
records in place
COMPLETE
Rewrite full output

## Fault tolerance
End-to-end fault tolerance
Guaranteed in Structured Streaming by
Checkpointing and write-ahead logs
Idempotent sinks
Replayable data sources

## Aggregations
intro aggs over stream data why, use cases

Aggregations
Windows
Watermarking

examples
Sentiment analysis on messaging platforms as messages come in
IoT results: analysing real-time weather data 
Unusual Activity Detection: Bank transactions, HTTP traffic with DDoS attacks

> Many applications: wherever real-time analytics would be an advantage over delayed batch processing

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
