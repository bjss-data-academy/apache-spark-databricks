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

The 10:32 event will be recorded _twice_: Once in the 10:00-10:59 window, because the visit occureed in that period. But _also_ in the overlapping window of 10:30 - 11:29. The event also took place in that time period. 

The event is not being 'double counted' here. It is simply being included in every relevant time period we are monitoring.

We can ask the questions "How many visitors in the hour starting at 10:00?" and "How many visitors for the hour starting at 10:30?". Visits will be correctly included in the relevant time period.

## Event-Time Processing
A big question with time-related data is "when did the event happen?"

This might sound like the answer is obvious: "The time on the clock when the event happened". But we weren't there watching, so we don't know. We must be told.

There are two main approaches to recording the time of an event:

- _Embedded field_ A field in the data record itself, set from a system clock reading when the data was read.
- _Time of receipt_ The system clock time when Spark first received the data.

The two times are different. An event may be detected by the sensor at 11:01. It takes time to transmit to us. The time of receipt will always be _later_. Possibly a lot later. 

Analytics usually wants the time the event happened, not the time it was received. 

## When data turns up late
Here's a problem. 

We are using tumbling windows. We are couting visitors in one-hour slots. We have a real-time visitor sensor. But it does not always send events right away.

This means the "new visitor at 10:32" message sometimes gets to us late. Sometimes very late, say at 14:47:

![Event at 10:32 am not received until 14:47](/images/late-data.png)

Oh dear. 

This means our results cannot be correct. When we worked out how many visits we had between 10:00 and 10:59, we actually had one more than we knew about at that time.

This is an inescapable problem with real-time analytics. 

Can we do anything about this?

### Watermarked data
We can! 

In principle, we simply defer the analytics calculation until all the late data has arrived. We accept new data and place it in the correct window, even when that data is received after the window has closed.

But there's a little detail there: _all the late data has arrived_. 

How can we know that all data is going to have arrived? 

Simply put, _we cannot_. Instead, we decide on a cutoff point, beyond which we discard late data.

We put our engineering finger in the air, and guesstimate that any data arriving more than ...ummm... one hour late we will ignore. Choose whatever timeframe makes sense for your application.

This is called a _watermark_ in Spark terms.

> _Watermark_ : maximum time to wait for late data

A watermark gives us two benefits:

- We capture _most_ of the late data points. This makes calculations more accurate
- We are not waiting _forever_ for data

Not waiting forever is practical. It saves us retaining data in memory indefinitely; we can discard it after the watermark. It provides a time at which we can run our calculation.

Adding a watermark is easy. It is configuration. Here is code that sets a 2 hour watermark on a streaming dataframe:

```python
visitor_stream_df.withWatermark("time", "2 hours")\
  .groupBy(col("device"), window(col("time"), "1 hour"))
  .count()
```

This uses a tumbling window of one hour time slots, with a 2 hour watermark to wait for late data. 

## Fault tolerance features
Things can go wrong with real-time data capture. We have seen how data can be late. 

But data may never arrive. 

Or it may arrive at Spark just as our Spark server crashes. Or the power goes off. 

Or an error in our Python code causes us to exit before we process that data. (_surely not! surely we unit tested it? - Ed_)  

A Data Engineer's lot is not always a happy one ;)

Thankfully, Databricks Spark helps us by providing fault-tolerance features.

### Write-ahead logs
Before Spark makes a change to some data, it writes a log entry saying what change it is about to make. This is a _write-ahead log entry_.

If there is a problem after this, the log entry will persist. Spark can then recover from the problem, and have a record of the data that needs to be included.

## Checkpointing
We can configure a storage location (usually cloud) to store _checkpoints_.

Checkpoints are files containing progress information, context and intermediate results from processing.

Spark can resume processing from a checkpoint - as if nothing had happened.

### Idempotent sinks
_Idempotence_ means that an operation can be applied multiple times without changing the end result.

Many things are not idempotent. Say I want to buy one item from an online store. I accidentally hit `Buy Now` twice. I get charged twice for the one item. We need to fix this and make this button idempotent. Then, we can press it as many times as we like, and still only pay one time.

Spark adds idempotence to write operations. This avoids accidental duplication of results data.

### Replayable data sources
What happens if real-time data fails to be captured?

Normally, that data is simply lost forever. We were looking the other way at the critical moment.

Spark adds some features to allow a data source to be _replayed_. We can ask for the data to be repeated to us.

## Partition number and streaming

200 default - bad
change to something better

```python
spark.conf.set("spark.sql.shuffle.partitions", spark.sparkContext.defaultParallelism)
```

# Further Reading

- [Spark Streaming Guide](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html)

# Next
[Back to Contents](/contents.md)
