# Dataflow: Googles Unified Model of Data Processing

### Key Resources:

https://www.youtube.com/watch?v=PGTSZvBK8-Y
https://medium.com/teads-engineering/give-meaning-to-100-billion-analytics-events-a-day-d6ba09aa8f44
https://research.google.com/pubs/pub43864.html
https://youtu.be/PGTSZvBK8-Y?t=3m11s

## Unified Model

Everything is a stream, Dataflow is notable for ["treating static data sets as streams which happen to have a beginning and an end"](https://data-artisans.com/blog/why-apache-beam)

## DataFlow

### Open Source - Beam and Flink

Google open-sourced Dataflow as Apache Beam, the programming model and SDK.

https://beam.apache.org/

It retained operational knowledge of how to run and scale Beam, and productized it as DataFlow.

https://cloud.google.com/dataflow/

## History - Infinite, Unordered, Unbounded

Google processes a large amount of data (whaaaaat?).  In many cases, this data is unordered and unbounded.

For example, you're collecting events or activities from an online game.  They arrive as hundreds of thousands or millions of players take actions in the game (this is apparently what they mean by *infinite*).  Initially you can group them into buckets, but to generalize, it's a stream of data points.

How they are collected and received is *unordered*:

1. Created and received at 7:59am
2. Created at 8am, but not received until 11am.
3. Created and received at 8.01 am

The data is virtually *unbounded*, in that any value, or almost any value can show up.

Storing and processing this infinite, unordered, unbounded data is the challenge that led Google, and the industry as a whole, to develop tools that eventually led to Dataflow.

### GFS and MapReduce

Web retrieval and indexing the first obvious domain.  Reliably storing huge amounts of data was the first challenge. [Google File System](https://research.google.com/archive/gfs.html) was developed to be schema-less, durable, resilient, automatically scaled storage fabric to be shared between machines.

[MapReduce](http://research.google.com/archive/mapreduce.html) was developed concomitantly with GFS to process data in a parallel, distributed, and fault-tolerant way.  One of the key insights of MapReduce, and how MapReduce and GFS work together, is moving computation to data, instead of trying to pipe data back to some centralized processing system.

[Apache Lucene, Apache Nutch and Apache Hadoop](https://medium.com/@markobonaci/the-history-of-hadoop-68984a11704) re-implemented these as open-source technology.

### Element-wise computations

The MapReduce approach is 

TODO: Spark

### The Lambda Architecture Model

Circa 2011 [Stream processing was coming into vogue](https://youtu.be/PGTSZvBK8-Y?t=3m11s) for large scale data-processing.  Apache Storm made design decisions that sacrificed accurate for reduced latency. Nathan Mars, the creator of storm, envisioned a way to ["beat the CAP theorem"](http://nathanmarz.com/blog/how-to-beat-the-cap-theorem.html).  His proposal was the Lambda Architecture.

The Lambda Architecture was an earlier unification effort, that wasn't really unified.  It has three distinct layers: *speed* (i.e. stream), *batch* and *serving* (query).  Each layer had a distinct code-base which added complexity.

[The Lambda Model](https://en.wikipedia.org/wiki/Lambda_architecture)

The speed layer has reduced accuracy and fast results, and the batch / slow layer offers perfect accuracy and long run-times.

*The results from both of these layers are merged*, and the fast approximate answers are eventually replaced with fully correct versions.

### Drawbacks to the Lambda Model

- Two separate systems is expensive in terms of fixed costs and complexity
- Merging the results is expensive

### Google's Take: Dataflow

Dataflow is a "unified approach" allows you to choose speed or batch layers, or both, but you write both using a single framework.

- Abstracts data processing from the specific runner
    + Runner could be Spark or Flink or Dataflow


#### Runners

- Apex - "stream processing platform and framework for low-latency, high-throughput and fault-tolerant analytics applications on Apache Hadoop"
- Flink - "Apache Flink® is an open-source stream processing framework for distributed, high-performing, always-available, and accurate data streaming applications"
- Gearpump
- Spark
- Cloud Dataflow (duh)

## APIs

### Higher-Level APIs

#### Scio

"Scio is a Scala API for Apache Beam and Google Cloud Dataflow inspired by Apache Spark and Scalding"

https://github.com/spotify/scio