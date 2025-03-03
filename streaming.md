# Streaming Data
intro todo

## Benefits

## Use Cases

## Micro-batching
its batches really

## Structured streaming
appends rows to unbounded table

![Stream of data appends rows to an unbounded table](/images/streaming-table.png)

## Inputs

- Kafka
- Event hubs
- Files

For debugging/test:
- sockets
- generator

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
