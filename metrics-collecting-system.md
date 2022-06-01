# System Design - Metrics Collecting System

CREATED 2022/01/14 REVISITED 2022/05/24,31

## Functional Requirements

For internal use only. System/operational metrics like CPU load, memory usage, disk usage, request count etc. And also business metrics.

The write load is heavy. As you can see, there can be many time-series data points written at any moment.

There are millions of operational metrics written per day, and many metrics are collected at high frequency, so the traffic is undoubtedly write-heavy.

## Non-Functional Requirements

Scalability - scalable to accommodate growing metrics and alert volumes

Low latency - low latency when querying in dashboard

Reliability - avoid missing critical alerts

Flexibility

## Main Components

Data collection, data transmission, data storage, alerting and visualization.

### Data Model

**single point data value** - (domain, service/dimension name, metric name, timestamp, value, labels ...)

**time-series data points(list)** - a list of data points in a certain time of period, an array of <timestamp, value> pairs.

## DB choice

### Relation DB

A general-purpose database, in theory, could support time-series data, but it would require expert-level tuning to make it work at our scale. Specifically, a relational database is not optimized for this solution. The complicated SQL is difficult to read.

### NoSQL

In theory, a few NoSQL databases on the market could handle time-series data effectively. Cassandra and BigTable can be both used for time series data. However, this would require deep knowledge to devise a scalable schema for effectively storing and querying time-series data.

### Better Choice

**Time-series** databases are very commonly used for storing metrics data, like **InfluxDB and Prometheus**.

According to DB-engines, the two most popular time-series databases are InfluxDB and Prometheus, which are designed to store large volumes of time-series data and quickly perform real-time analysis on that data. Both of them primarily rely on an in-memory cache and on-disk storage. And they both handle durability and performance quite well. According to the benchmark, an InfluxDB with 8 cores and 32GB RAM can handle over 250,000 writes per second.

Another feature of a strong time-series database is efficient aggregation and analysis of a large amount of time-series data by labels(or tags).

## Infra Graph

[**Metrics Source**] -> [**Metrics Collector**] -> [**Time Series DB**] <-> [**Query Services**] <- [**Alerting System**] & [**Visualization System**]

[**Alerting System**] -> Email, Text Message, HTTPS endpoints.

### Metrics Source

This can be application servers, SQL databases, message queues, etc.

### Metrics Collector

It gathers metrics data and writes data into the time-series database.

1 Pull Mode - It needs to know the complete list of service endpoints to pull data from.

Service discovery contains configuration rules about when and where to collect metrics from. Use a consistent hashing ring to ensure that one metrics source server is handled by only one collector.

Cons: It needs to poll for endpoint changes periodically. Some jobs might be short-lived and don't last long enough to be pulled.

2 Push Mode - Send metrics directly to the metrics collector. The collection agent, installed on every server, aggregates metrics locally and sends them into metric collectors. They can hold a small buffer of data locally on disk if sending meets an error or exception.

The metrics collector should be in an auto-scaling cluster with a load balancer in front of it.

Cons: It needs requiring authentication.

### Kafka (Optional)

Kafka is used as a highly **reliable and scalable** distributed messaging platform. It **decouples** the data collection and data processing services from each other. And it can easily prevent data loss.

### Consumers (Optional)

Consumers or streaming processing services such as Apache Storm, Flink and Spark, process and push data to the time-series database.

Optional - Use an intermediate queue: In case the Time Series DB is unavailable, we should introduce Kafka and Consumers in front of it. But this is not always required.

### Cache Layer (Optional)

Store the query results

### Time-series Database

This stores metrics data as time series. It usually provides a custom query interface for analyzing and summarizing a large amount of time-series data. It maintains indexes on labels to facilitate the fast lookup of time-series data by labels.

### Query Service

A cluster of query servers which takes the requests from clients(visualization and alerting systems).

## Others

### Down-sampling

Retention: 7 days, no sampling. 30 days, down-sample to 1 minute resolution. 1 year, to 1 hour resolution.

Purpose: reduce the data size.

### Visualization

Grafana is a visualization tool for time series data.

## Influx DB

InfluxDB is an open-source time series database (TSDB) developed by the company InfluxData. It is written in the Go programming language for storage and retrieval of time series data in fields such as operations monitoring, application metrics, Internet of Things sensor data, and real-time analytics.

## AresDB in Uber - https://eng.uber.com/aresdb/

TODO

## More General - Ad Click Event Aggregation

Not only operational and business metrics, but also log files containing click events like (ad_id, timestamp, user_id, ip, location, etc).

Main use cases: 1) Aggregate the number of clicks of ad_id in the last M minutes. 2) Return top K most clicked ad_ids in the last M minutes. 3) Data filtering.

Non-functional requirements: Correctness, Handle delayed and duplicated messages properly, Robustness and Latency Requirement.

Use Cassandra to store both the raw data (read low, write heavy) and the aggregated data (read heavy, write heavy). Old raw data could be moved to **cold storage** and aggregated data could be served as active data.

Raw data: (ad_id, click_timestamp, user_id, ip, country, ...)

Aggregated data: (ad_id, click_minute, count, ...) and (timestamp_minute, most_clicked_ads)

Time: Event time & Processing time. Trade-off to decide which is used for aggregation. Prefer **event time** for more accuracy. We can use **watermark**, which is regarded as an extension of an aggregation window (extended wait time), to include the events with short delays in order to help improve accuracy.

Aggregation windows: We choose Tumbling window (same length with non-overlapping) and sliding window (sliding across the data stream according to a specified interval).

Avoid data duplication: We can record the offset by using external file storage (HDFS or S3). To avoid data loss, we need to save the offset once we get an ack back from downstream service.

Reconciliation: compares raw data database and aggregation database. The results may not exactly same match because some events might arrive late (with latency).

Another option is to store data in Hive with an ElasticSearch layer built on it for faster queries.

## Reference

[1] Copied from Alex Xu's posts in LinkedIn website

<https://www.linkedin.com/posts/alex-xu-a8131b11_systemdesign-coding-interviewtips-activity-6887071466156892160-uMs3>

<https://www.linkedin.com/feed/update/urn:li:activity:6887439575577432064>

<https://www.linkedin.com/posts/alex-xu-a8131b11_systemdesign-coding-interviewtips-activity-6887794567282860032-hX9-/>

[2] System Design Interview Book Volume II. Alex Xu. Chapter 5. Metrics Monitoring and Alerting System.

[3] <https://en.wikipedia.org/wiki/InfluxDB>
