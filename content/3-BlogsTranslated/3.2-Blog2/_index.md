---
title: "How Karrot Built its Feature Platform on AWS, Part 2: Feature ingestion"
date: "2025-08-14T10:00:00+07:00"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

> By: Hyeonho Kim, Jinhyeong Seo, Minjae Kwon, Hyuk Lee, Jinhyun Park, Jungwoo Song, and Nak-Kwon Choi | on: AUGUST 14, 2025 | In: [Advanced (300)](https://aws.amazon.com/blogs/architecture/category/learning-levels/advanced-300/), [Amazon Managed Streaming for Apache Kafka (Amazon MSK)](https://aws.amazon.com/msk/), [AWS Batch](https://aws.amazon.com/batch/), [AWS Fargate](https://aws.amazon.com/fargate/), [Customer Solutions](https://aws.amazon.com/blogs/architecture/category/post-types/customer-solutions/) | [Permalink](https://aws.amazon.com/blogs/architecture/how-karrot-built-a-feature-platform-on-aws-part-2-feature-ingestion/) | [Comments](https://aws.amazon.com/blogs/architecture/how-karrot-built-a-feature-platform-on-aws-part-2-feature-ingestion/#comments) |
>
> *This article is co-authored by Hyeonho Kim, Jinhyeong Seo, and Minjae Kwon from Karrot.*

In [Part 1]({{< ref "/3-blogstranslated/3.1-blog1" >}}) of this series, we discussed how [Karrot](https://about.daangn.com/en/) developed a new feature platform, consisting of three main components: feature serving, a stream ingestion pipeline, and a batch ingestion pipeline. We discussed the requirements, solution architecture, and feature serving using a multi-level cache. In this post, we will share the stream and batch ingestion pipelines and how they ingest data into an online store from various event sources.

---

## Solution overview

The solution overview diagram was introduced in [Part 1]({{< ref "/3-blogstranslated/3.1-blog1" >}}).
<img src="/images/bl2-1.png" alt="" width="50%">

## Stream ingestion

Stream ingestion is the process of collecting data from various event sources in real time, transforming it into features, and storing them. It consists of two main components:

* **Message broker** – [Amazon Managed Streaming for Apache Kafka](https://aws.amazon.com/msk/) (Amazon MSK) is used to store events published from various platform services and change data capture (CDC) events from [Amazon Aurora](https://aws.amazon.com/rds/aurora/).
* **Consumer** – These are pods on [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/) (Amazon EKS) that are responsible for processing events according to the feature group specifications defined in the feature platform and loading them into the database and remote cache.

Consumers process not only source events but also [re-published](https://docs.confluent.io/platform/current/streams/faq.html#streams-faq-republish-output-topic) events. When loading features, they are performed considering various strategies, such as [write-through](https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/write-through.html) and write-around, and are loaded in detail by considering cardinality, data size, and access patterns.

Most features are created based on two types of events: events that occur due to real-time user actions, and asynchronous events that occur due to state changes in user and article data. These events and features have an M:N relationship, meaning that one event can be the source of multiple features, and one feature can be created based on multiple events.

> *Architecture diagram of the stream ingestion pipeline.*
> <img src="/images/bl2-2.png" alt="" width="50%">

To efficiently handle M:N relationships, a structure is needed to receive events and distribute them to multiple feature processing logics. Two core components were designed for this purpose:

* **Dispatcher** – Receives events from multiple consumer groups and passes them to the relevant feature processing logic
* **Aggregator** – Processes the events received from the Dispatcher into actual features

This stream processing pipeline enables real-time feature creation and storage.

### Message broker optimization: Fast at-least-once delivery

The feature platform processes up to 25,000 events per second, including user behavior log events, at a high speed. However, when worker traffic spikes, event processing failures or infrastructure failures sometimes cause event loss. To solve this problem, the existing [automatic commit mode](https://cwiki.apache.org/confluence/display/KAFKA/KIP-41+-+KafkaConsumer+Max+Poll+Records) was changed to manual commit in Amazon MSK. This allows events to be committed only when they have been reliably processed, and failed events are sent to a separate retry topic and post-processed via a dedicated worker.

However, processing a large volume of events synchronously with manual commit made the processing speed about 10 times slower and increased the latency. Although consumer group resources were available, simply increasing the number of partitions in Amazon MSK was not a solution due to the partition permissions specific to each group. The platform designed parallel processing inside single partitions and implemented a custom consumer that supports retry functionality. The core of the implementation is to read the number of messages by fetch size from the partition at a time and process them by creating parallel worker threads for each message. When the processing is complete, the offsets of the successful messages are sorted and a manual commit is performed for the largest offset, and the failed messages are republished to the retry topic. This allows for parallel processing even within a single partition, and the concurrency can be controlled automatically. As a result, the event processing speed is faster than the existing automatic commit method, and it is processed stably without lag even when the number of events increases.

---

## Stream processing

The stream ingestion pipeline only performs simple extract, transform, and load (ETL) and validation logic. There were many requests for complex stream processing in the feature platform, and a separate service was created to meet them. The feature platform did not resolve these requests for the following reasons:

* The purpose of stream ingestion in the feature platform is to collect and store features in real time, while the main purpose of stream processing is to process data.
* Not all features require complex processing. We decided that it was not appropriate to complicate the entire stream collection process for some features.
* The resulting data of stream processing can be used outside the feature platform, and there are requirements that need to consider this. Therefore, creating a separate service was more in line with Karrot’s situation.
* In addition, some source data does not exist in AWS, which could lead to significant additional costs if everything is processed in the feature platform.

Although it is a separate service from the feature platform, the following is a brief introduction to how the feature platform uses data through stream processing:

* **Various content embedding cases** – We perform stream processing using models, and use various contents (articles, images, etc.) as input values to pre-trained models to create embeddings. These embeddings are stored in the feature platform and used as features during the recommendation process to improve recommendation quality.
* **Rich feature generation cases** – Some processed data is further processed using large language models (LLMs) to be used as features. An example is predicting which category a specific used product belongs to and using this predicted value as a feature.

---

## Batch ingestion

Batch ingestion is responsible for processing and storing large amounts of data into features in batches. This is divided into a cron job that runs periodically and a backfill job that loads a large amount of data at once.

For this purpose, [AWS Batch](https://aws.amazon.com/batch/) based on [AWS Fargate](https://aws.amazon.com/fargate/) is used. AWS Batch jobs running on Fargate are provided independently of other environments, allowing for safe large-scale processing. For example, even if more than 1,000 servers or 10,000 vCPUs are used for backfilling a large amount of data, they are operated separately from other services and can be operated efficiently with a usage-based billing method.

When adding new features, batch loading past data or periodically loading large amounts of data is one of the core functions of the feature platform. The main requirements considered in the design are:

* It must be able to process a large amount of data.
* It must be able to start at the user's desired time and complete the work within an appropriate time.
* It must have low operating costs. It is best if it is a managed service, and it is better if there is little additional work or specific expertise to operate. In addition, it should reuse existing service code as much as possible.
* Complex operations for features or configuration of Directed Acyclic Graphs (DAGs) are not necessarily required.

There were several options to choose from, such as [Apache Airflow](https://airflow.apache.org/), but AWS Batch was chosen to avoid over-engineering when considering the operating costs according to the current requirements.

> *Architecture diagram of the batch ingestion pipeline.*
> <img src="/images/bl2-3.png" alt="" width="50%">

The main components are:

* **Scheduler** – It extracts the targets to perform batch jobs according to the specifications such as `FeatureGroupSpec` and `IngestionSpec` written by the user on the feature platform, and registers the corresponding job specifications to an AWS Batch job (submit job).
* **AWS Batch** – The jobs submitted by the Scheduler are executed using the pre-configured job queue and computing environment. In the case of AWS Batch, you can configure a Fargate environment separate from other production services, so even if you provision large-scale resources and perform tasks, you can perform tasks stably without affecting other production services.

### Future improvements for Batch ingestion

The current configuration works well and reliably, but there are several areas for improvement:

* **No DAG support** – The feature platform initially performed relatively simple tasks, such as parsing batch data sources, converting them to the feature schema, and storing them. However, as the platform became more advanced, more complex operations became necessary, and thus, support for DAG configurations that can process features by sequentially performing various dependent jobs became necessary.
* **Manual configuration for parallel processing** – Currently, when processing large-scale data in parallel, the worker must manually estimate the number of jobs to be processed in parallel and provide it in the specification, and the Scheduler performs parallel job submission based on this. This method is entirely based on experience, and for the system to become more advanced, the system must be able to automatically abstract and optimize the appropriate level of parallel processing.
* **Limited AWS Batch monitoring usability** – AWS Batch monitoring has several limitations, such as jobs not transitioning from Runnable to Running state, lack of a proper notification system for such cases, and inability to directly check failed jobs via URL parameters when receiving alerts. These aspects should be improved from the perspective of operational convenience.

---

## Results

As of February 2025, Karrot has resolved the main problems mentioned in the early stages of feature platform development:

* **Decoupling recommendation logic from flea market server** – The recommendation system now uses the feature platform on more than 10 different spaces and recommendation services.
* **Securing scalability of features used in recommendation logic** – With over 1,000 high-quality and rich features obtained from various services such as flea market, advertisements, local jobs, and real estate, we are contributing to the advancement of recommendation logic and helping all Karrot engineers easily explore and add features.
* **Maintaining the reliability of feature data sources** – Through the feature platform, we are providing reliable data by using a consistent schema and ingestion pipeline.

Karrot engineers are constantly improving the user experience by enhancing recommendations through high-quality features thanks to the feature platform. This has contributed to a 30% increase in click-through rates and a 70% increase in conversion rates compared to before by recommending articles that users might be interested in.

This was possible thanks to the AWS services used in the feature platform, which provided solid support. **[Amazon DynamoDB](https://aws.amazon.com/dynamodb/)** has amazing scalability in all aspects of read, write, and storage, so it can handle dynamically changing workloads without incurring separate operating costs. **[Amazon ElastiCache](https://aws.amazon.com/elasticache/)** has shown high reliable service stability, so we can use it with confidence. In addition, scaling up, scaling down, scaling in, and scaling out are very easy and stable, so the operational burden can be reduced. It also integrates seamlessly with the Redis OSS ecosystem, so we can use open source ecosystems like [Redis Exporter](https://github.com/oliver006/redis_exporter). Amazon MSK also supports reliable operation and seamless integration with the Apache Kafka ecosystem, making it easy to develop and operate the feature platform.

Furthermore, working with AWS enables cost-effective operation based on their diverse support and expertise. Recently, we encountered an over-provisioning issue with our ElastiCache cluster. Right-sizing the ElastiCache cluster with the help of various experts (including Solutions Architects) helped optimize ElastiCache costs by nearly 40%. These technical human resources from AWS have been invaluable in operating the feature platform using AWS products.

---

## Conclusion

In this series, we discussed how Karrot built a feature platform on AWS. We believe that by combining AWS services and our experience, you can develop and operate a feature store without difficulty by tailoring it to your company's requirements. Feel free to experiment with this implementation and let us know your thoughts in the comments section.