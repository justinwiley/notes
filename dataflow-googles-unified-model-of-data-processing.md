# Dataflow: Googles Unified Model of Data Processing

https://www.youtube.com/watch?v=PGTSZvBK8-Y
https://medium.com/teads-engineering/give-meaning-to-100-billion-analytics-events-a-day-d6ba09aa8f44

## Unified Model

Everything is a stream, Dataflow is notable for ["treating static data sets as streams which happen to have a beginning and an end"](https://data-artisans.com/blog/why-apache-beam)

## DataFlow

### Open Source - Beam and Flink

Google open-sourced Dataflow as Apache Beam.

https://beam.apache.org/

#### Runners

- Apex - "stream processing platform and framework for low-latency, high-throughput and fault-tolerant analytics applications on Apache Hadoop"
- Flink - "Apache Flink® is an open-source stream processing framework for distributed, high-performing, always-available, and accurate data streaming applications"
- Gearpump
- Spark
- Cloud Dataflow (duh)

## History

### Hadoop

## The Lambda Architecture Model

The Lambda Architecture was an earlier unification effort, that wasn't really unified.  It has three distinct layers: *speed* (i.e. stream), *batch* and *serving* (query).  Each layer had a distinct code-base which added complexity.

[The Lambda Model](https://en.wikipedia.org/wiki/Lambda_architecture)

## APIs

### Higher-Level APIs

#### Scio

"Scio is a Scala API for Apache Beam and Google Cloud Dataflow inspired by Apache Spark and Scalding"

https://github.com/spotify/scio