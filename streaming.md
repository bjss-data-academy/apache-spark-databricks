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

## Time-Based Windows
We're often concerned with _time_ in our analyses. When did a thing happen? How many things happened in the last hour?

One challenge with streaming is "how do we treat time when we aggregate data"?

If we are aggregating some data value over time, it makes sense to chunk up time itself into discrete blocks.

We call these time blocks _windows_.

There are two variations on a time window in Spark: _Tumbling_ and _Sliding_ windows. Each has a different purpose.

### Tumbling window
Tumbling windows are intervals of time that _have no overlap_:

![Showing intervals of time not overlapping](/images/tumbling-window.png)

We see three events, happening at times 10:32, 11:59 and 13:02. Each one of these events fits into only one time window. The 10:32 event, for example, fits into the time window for events during 10:00 to 10:59.

This approach is useful for bucketing events at a fixed granularity. We can answer questions like "How many visitors did we have in the hour starting at 10:00?"

We can set up a windowing query easily in Python:
  
```python
vistor_stream_df.groupBy(col("visit_location"), window(col("time"), "1 hour")).count()
```

This would take a streaming dataframe which tracked visitors to different locations in real-time. The query would group by the visit location, then count how many vistors were present during one-hour time windows.

### Sliding window
Sliding windows have overlaps in the intervals they represent:

![Showing intervals of time overlapping](/images/sliding-window.png)

The same three events are happening at 10:32, 11:59 and 13:02 as before. But we're using sliding windows. Each window is one hour wide, but it _overlaps_ other windows so that we get a half-hour granularity. 

The 10:32 event will be recorded _twice_: Once in the 10:00-10:59 window, because the visit occureed in that period. But _also_ in the overalpping window of 10:30 - 11:29. The event also took place in that time period. 

The event is not being 'double counted' here. It is simply being included in every relevant time period we are monitoring.

We can ask the questions "How many visitors in the hour starting at 10:00?" and "How many visitors for the hour starting at 10:30?". Visits will be correctly included in the relevant time period.

## Event-Time Processing
A big question with time-related data is "what time do we use?"

This might sound like the answer is obvious: "The time the event happened". But we weren't there watching, so we don;t know. We must be told.

There are two main ways to decide when an event happened:

- _Embedded field_ such as a timestamp, recorded by a sensor, then written into the data record itself
- _Time of receipt_ the system clock time at which Spark first received the data

The two times are often different. An event may be recorded at 11:01. But we may only receive it two minutes later, due to some latency within the acquisition.

## Late data - Watermarks
TODO TODO

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
### Note on Partitions and stremaing

200 default - bad
change to something better

```python
spark.conf.set("spark.sql.shuffle.partitions", spark.sparkContext.defaultParallelism)
```

# Further Reading

- [Spark Streaming Guide](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)

# Next
[Back to Contents](/contents.md)
