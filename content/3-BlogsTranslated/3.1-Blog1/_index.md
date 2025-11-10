---
title: "How Karrot Built its Feature Platform on AWS, Part 1: Motivation and Feature Serving"
date: "2025-08-14T10:00:00+07:00"
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

> Authors: Hyeonho Kim, Jinhyeong Seo, Minjae Kwon, Hyuk Lee, Jinhyun Park, Jungwoo Song, and Nak-Kwon Choi | on: AUGUST 14, 2025 | In: [Advanced (300)](https://aws.amazon.com/blogs/architecture/category/learning-levels/advanced-300/), [Amazon DynamoDB](https://aws.amazon.com/dynamodb/), [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/), [Amazon ElastiCache](https://aws.amazon.com/elasticache/), [Customer Solutions](https://aws.amazon.com/blogs/architecture/category/post-types/customer-solutions/)
>
> *This article is co-authored by Hyeonho Kim, Jinhyeong Seo, and Minjae Kwon from Karrot.*
<img src="/images/bl1.png" alt="" width="70%">

**[Karrot](https://about.daangn.com/en/)** is South Korea's leading local community, a service focused on all possible connections within a neighborhood. Going beyond simple flea markets, Karrot strengthens connections between neighbors, local shops, and public organizations, creating a warm and vibrant neighborhood as its core value.

Karrot uses a recommendation system to provide users with connections tailored to their interests and neighborhood, delivering a personalized experience. Personalized content is continuously updated by analyzing user activity patterns without needing to set up a special interest category. The core of the feed is to provide new and interesting content, and Karrot constantly strives to improve user satisfaction for this purpose. Karrot actively uses its recommendation system to provide personalized content and suggestions.

In this system, the **feature platform** plays a key role alongside the **ML recommendation model**. The feature platform acts as a data repository that stores and serves data needed for the ML recommendation model, such as user behavior history and article information.

This two-part series begins by presenting Karrot's motivation, requirements, and solution architecture, focusing on feature serving. [Part 2]({{< ref "/3-blogstranslated/3.2-blog2" >}}) will cover the real-time feature collection process and batch ingestion into the online store, along with technical approaches for stable operation.

---

## Background of the Feature Platform at Karrot

Karrot recognized the need for a feature platform in early 2021, about 2 years after implementing a recommendation system in their application. At that time, Karrot was achieving significant growth (over 30% click-through rates) and higher user satisfaction through the active use of its recommendation system. As the impact of the recommendation system continued to grow, the ML team faced the challenge of enhancing the system.

In ML-based systems, a variety of high-quality input data (like clicks, conversion actions, etc.) is considered a key factor. This input data is often called **features**. At Karrot, data including user behavior logs, action logs, and state values are collectively called *user features*, while logs related to posts/classifieds are called *article features*.

To improve the accuracy of personalized recommendations, various types of features are needed. A system that can efficiently manage these features and quickly provide them to ML recommendation models is essential. Here, *serving* means the process of providing real-time data needed when the recommendation system suggests personalized content to a user. However, the existing feature management approach in the recommendation system had several limitations, with the following main issues:

* **Dependency on the flea market server** – Since the original recommendation system existed as an internal library on the flea market server, the web application's source code had to be changed whenever the recommendation logic was modified or a feature was added. This reduced deployment flexibility and made resource optimization difficult.
* **Limited scalability of recommendation logic and features** – The original recommendation system directly depended on the flea market database and only considered flea market articles. This made it impossible to expand to new article types like local communities, local jobs, and advertisements, which were managed by other data sources. Additionally, feature-related code was hardcoded, making it difficult to explore, add, or modify features.
* **Lack of reliability of feature data sources** – Although features were retrieved from various repositories like [Amazon Simple Storage Service](https://aws.amazon.com/s3/) (Amazon S3), [Amazon ElastiCache](https://aws.amazon.com/elasticache/), and [Amazon Aurora](https://aws.amazon.com/rds/aurora/), the reliability of data quality was low due to the lack of a consistent schema and collection pipeline. This was a major limitation in ensuring the latest features and consistency.

> *Diagram illustrating the backend structure of the original recommendation system.*
<img src="/images/bl1-1.png" alt="" width="55%">

To address these issues, we needed a new central system that could efficiently support feature management, real-time ingestion, and serving, and so we began the feature platform project.

---

## Feature Platform Requirements

The following functional requirements were organized by separating the feature platform into an independent service:

* Record and quickly serve the N most recent actions performed by a user. Allow parameterization of both the N value and the lookup time range.
* Support user-specific features, such as notification keywords, in addition to action features.
* Handle features from various article types, beyond just flea market articles.
* Handle arbitrary data types for all features, including primitive types, lists, sets, and maps.
* Provide real-time updates for both action features and user characteristic features.
* Provide flexibility in the list of features, quantity, and lookup time range for each request.

To implement these functional requirements, a new platform was necessary. This platform needed three core capabilities: real-time ingestion of various feature types, storage with a consistent schema, and a fast response to diverse query requests. Although these requirements initially seemed vague, designing a general structure allowed for the efficient configuration of data ingestion pipelines, storage methods, and serving schemas, leading to clearer development goals.

In addition to functional requirements, the technical requirements included:

* **Serving traffic:** 1,500 or more requests per second (RPS)
* **Ingestion traffic:** 400 or more writes per second (WPS)
* **Top N values:** 30–50
* **Single feature size:** Up to 8 KB
* **Total number of features:** Over 3 billion or more

At the time, the variety and number of features being used were limited, and the recommendation models were simple, leading to modest technical requirements. However, considering the rapid growth rate, a significant increase in system requirements was anticipated. Based on this prediction, higher goals were set beyond the initial requirements. As of February 2025, serving and ingestion traffic has increased by about 90 times compared to the initial requirements, and the total number of features has increased by hundreds of times. The ability to handle this rapid growth was made possible by the highly scalable architecture of the feature platform, which we discuss in the following sections.

---

## Solution Overview

> *Diagram illustrating the architecture of the feature platform.*
> 
> <img src="/images/bl1-2.png" alt="" width="55%">

The feature platform consists of three main components: feature serving, a stream ingestion pipeline, and a batch ingestion pipeline.

Part 1 of this series will cover **feature serving**. Feature serving is the core function of receiving requests from clients and providing the necessary features. Karrot designed this system with four main components:

* **Server** – A server that receives and processes feature serving requests, and is a pod running on [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/) (Amazon EKS).
* **Remote cache** – A remote cache layer shared by the servers, using ElastiCache.
* **Database** – A persistence layer that stores features, using [Amazon DynamoDB](https://aws.amazon.com/dynamodb/).
* **On-demand feature server** – A server that serves features that cannot be stored in the remote cache and database due to compliance issues, or that require real-time computation each time.

From a data store perspective, feature serving should serve high-cardinality features with low latency at a large scale. Karrot introduced a multi-level cache and classified serving strategies according to the characteristics of the features:

* **Local cache (tier 1 cache)** – An in-memory store located inside the server, suitable for cases where the data size is small and frequently accessed or requires a fast response time.
* **Remote cache (tier 2 cache)** – Suitable for cases where the data size is medium and frequently accessed.
* **Database (tier 3 cache)** – Suitable for cases where the data size is large and not frequently accessed or is less sensitive to response time.

---

## Schema Design

The feature platform stores multiple features together using the concept of **feature groups**, similar to column families. All feature groups are defined via a feature group schema, called **feature group specifications**, and each feature group specification defines the name of the feature group, the required features, etc.

Based on this concept, the key design is defined as follows:

Partition key: <feature_group_name>#<feature_group_id>
Sort key: <feature_group_timestamp> or a string representing null

To illustrate how this works in practice, let's explore an example of a feature group representing the *recently clicked flea market articles* by user 1234. Consider the following scenario:

* **Feature group name:** recent_user_clicked_fleaMarketArticles
* **User ID:** 1234
* **Click timestamp:** 987654321
* **Features in the feature group:**
    * Clicked article ID: a
    * User session ID: 1111

In this example, the keys and feature group are created as follows:

Partition key: recent_user_clicked_fleaMarketArticles#1234
Sort Key: 987654321
Value: {"0": "a", "1": "1111"}

The features defined in the feature group specification maintain a fixed order, using this order as an enum when saving the feature group.

---

## Feature Serving Read/Write Flow

The feature platform uses a multi-level cache and database for feature serving, as shown in the following diagram.

> *Diagram illustrating the read/write flow of feature serving.*
> <img src="/images/bl1-3.png" alt="" width="55%">

To illustrate this process, let's consider how the system retrieves feature groups 1, 2, and 3 from flea market articles. The **read flow** (solid line in the previous diagram) demonstrates data access optimization using a multi-level cache strategy:

1.  When a query request arrives, first check the **local cache**.
2.  Data not in the local cache is searched for in **ElastiCache**.
3.  Data not in ElastiCache is searched for in **DynamoDB**.
4.  The feature groups found at each stage are collected and returned as the final response.

The **write flow** (dotted line in the previous diagram) includes the following steps:

1.  Feature groups that resulted in cache misses are stored at each cache level.
2.  Data not found in the local cache but found in the remote cache or database is stored in the upper-level cache.
3.  Data found in ElastiCache is stored in the local cache.
4.  Data found in DynamoDB is stored in both ElastiCache and the local cache.
5.  Cache write operations are performed asynchronously in the background.

This approach presents a strategy for maintaining data consistency and improving future access times in a multi-level cache structure. In an ideal situation, serving would work well without any issues with just the previous flow. However, reality is not so. Problems encountered include cache misses, consistency, and penetration issues:

* **Cache miss problem** – Frequent cache misses slow down response times and place a burden on the next-level cache or database. Karrot uses the [Probabilistic Early Expirations](https://cseweb.ucsd.edu//~varghese/papers/caching-in-cache-ton.pdf) (PEE) technique to proactively refresh data likely to be retrieved again in the future, thereby maintaining low latency and minimizing cache stampedes.
* **Cache consistency problem** – If the Time-To-Live (TTL) of the cache is set incorrectly, it can affect recommendation quality or reduce system efficiency. Karrot sets separate soft and hard TTLs, and sometimes uses a [write-through caching strategy](https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/write-through.html) to synchronize the cache and database to alleviate consistency issues. Additionally, [jitter](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/) is added to distribute TTL expiration times to mitigate [cache stampede](https://en.wikipedia.org/wiki/Cache_stampede) for feature groups written at similar times.
* **Cache penetration problem** – Continuous queries for non-existent feature groups can lead to DynamoDB queries, increasing costs and response times. The platform addresses this problem through [negative caching](https://aws.amazon.com/blogs/database/caching-empty-results-with-amazon-elasticache-for-redis/), storing information about non-existent feature groups to reduce unnecessary database queries. Additionally, the system monitors the rate of missing feature groups in DynamoDB, negative cache hit rates, and potential consistency issues.

---

## Future Improvements for Feature Serving

Karrot is considering the following future improvements for their feature serving solution:

* **Large data caching** – Recently, the demand for storing large data features is increasing. This is because as Karrot grows, the number of features also increases. Additionally, as the demand for embeddings has grown with the rapid development of large language models (LLMs), the size of the data needing to be stored has increased. Accordingly, we are considering more efficient serving by using an embedded database.
* **Efficient cache usage:** Even if an effective TTL value is set initially, efficiency tends to decrease as user usage patterns change and models are modified. Also, as more feature groups are defined, monitoring becomes more difficult. It needs to be simple to find the optimal TTL value for the cache based on data. We are considering a method to use memory efficiently while maintaining high recommendation quality through cache hit rate and feature group loss prevention. Should we cache a feature group that is retrieved only once? What about a feature group retrieved twice? The current feature platform tries to cache even if a cache miss occurs only once. We believe that all feature groups that miss the cache are worth caching. This naturally increases caching inefficiency. An advanced policy is needed to identify and cache feature groups that are worth caching based on various data. This will increase cache usage efficiency.
* **Multi-level cache optimization:** Currently, the feature platform has a multi-level cache structure, and complexity will increase if an embedded database is added in the future. Therefore, it is necessary to find and set optimal settings by considering different cache levels. In the future, we will try to maximize efficiency by considering different levels of cache settings.

---

## Conclusion

In this article, we reviewed how Karrot built their feature platform, focusing on feature serving capabilities. As of February 2025, the platform reliably handles over 100,000 RPS with a P99 latency of less than 30 milliseconds, providing stable recommendation services through a scalable architecture that efficiently manages traffic increases.

[Part 2]({{< ref "/3-blogstranslated/3.2-blog2" >}}) will explore how features are created using consistent feature schemas and ingestion pipelines through the feature platform.