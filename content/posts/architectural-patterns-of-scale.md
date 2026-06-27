---
title: "Architectural Patterns of Scale"
date: 2022-04-03
categories:
- Architecture
author: Bruno Fernandez-Ruiz
---

One of most difficult aspects of systems development is achieving simplicity. Simplicity is the most important trait across all other non-functional requirements (the 'ilities) for a system. Simplicity is a necessary foundation for being able to understand, reason and work with complex system. Even though all initial designs for a new component or a new system look "elegant", the reality of systems development is that soon enough these systems become subject to changes, extensions, patches, bug fixes, in order to quickly accommodate the evolving business requirements. The prowess as engineers developing systems needs to be developed over time, with experience of building systems that eventually are too complex to maintain and lead to the dreaded half rewrite that never completes and just results in even more complexity.

Designing for simplicity was a necessity, the only viable solution, when faced with limited resource availability. Early UNIX version favourited small, independent, programs, which were easier to write, debug and use. It was also not possible to create very large programs, as the compilation cycles would be extremely long. Therefore a philosophy of design around small independent programs, which could communicate with each other over a standard input/output protocol, was born. This has evolved and remains today the underpinning philosophy of the Linux kernel and operating system alike. Simple utility programs and can be chained through pipes and filters in order to achieve much more complex tasks. When designing programs for Linux, the main question the architect needs to make is "can it do less and still be useful"?

Since network and distributed systems appeared there have been waves of decentralisation and re-centralisation in systems design. From the initial mainframe client model, to the client-services architecture, first ORBS and later on with J2EE, message bus architectures, service-oriented architectures, containerised architectures, and microservices, we keep seeing the pendulum swing between approaches seeking the simplicity of small units (the UNIX philosophy) and approaches reducing the control and coordination complexity of having hundreds of such small units in an architecture.

One of the triggers for such changes is the evolution of the compute platform. The availability of a fast network and cheap magnetic storage replaced the massive Oracle data scale-up centralised warehouse for the highly distributed Hadoop architecture on commoditised hardware. The appearance of fast SSDs and low footprint wafer nanometer technology has blended the difference between RAM and disk, allowing a move to more functional architectures, such as simple microservices on Kubernetes.

This evolution also characterised by a matching change in the programming languages that engineers use to create such systems. Programming languages designers have to make a trade-off between two rubrics: performance and safety (elegance and productivity are just projections of these two). Extremely performing programming languages are not safe, and extremely safe languages are not performing, to the point that some are simply not usable. The change in the underlying computing platform resources allows programming language designers to continuously push up the boundaries of safety without a sacrifice in performance, thanks to the platform advancement. Particularly, pure functional programming, in which there are no side effects is perfectly safe (but also nothing happens since there are no side effects). Recent programming languages try to limit the side effects, yet leaving some room to change state.

This same evolution towards more functional architectures is pushing system designers to think in terms of tuples (within messages) and we see some pure functional services with no side-effects (at least to the programmer it looks like the only side-effect is the output tuple) such as AWS Lambda and Cloudflare Workers. The largest systems in the world developed and managed by Google, Facebook, Twitter and Netflix are highly functional by design. Independent systems that consume and produce messages (tuples), and that clearly delineate the boundaries between stateful and stateless services. This strict split in workloads requires state to be preserved, and storage design defines at the end of the day the final scalability constraints of any large architecture.

As we go forward designing highly scalable systems that map the planet in real-time, we need to be aware of these constraints and leverage the push towards more functional system architectures, facilitated by the underlying computing platform evolution and programming language support. It's only by internalising and observing these traits that we can think and design simple systems in the context of complex architectures solving the extremely complex problem of mapping the planet.

One particular useful 50,000 feet view to do this task and guide us towards simple solutions that work well in isolation is to look at the architectural patterns of those tech giants. I'll try to give examples as we move forward for each pattern. So far I have identified the following workloads:

- Online services providing queries and lookups, without any synchronous writes. This is the example of Google's web search, the Instagram feed, or Twitter.
- Batch and streaming services, performing heavy asynchronous writes, sequential reads and scans and very few random data access requests. An example of a batch workload is the creation of the web map index, or calculating the targeting segments a user profile falls under; an example of a streaming workload is traffic protection (avoid fraud clicks in advertising), or advertiser budget management, or a social network message posting. One particular example blending both worlds is Nathan Marz’s "[lambda architecture](http://nathanmarz.com/blog/how-to-beat-the-cap-theorem.html)."
- Data analytics, used to ingest and analyse all facts from the production systems, usually in the form of a large data warehouse (nowadays decentralised) using a star schema and a rich domain model.
- Backoffice systems, used to manage and configure the dimensions of a system. For example, the Facebook Ads campaign manager. These systems exhibit much lower loads in terms of QPS, and strict ACID properties are usually expected and required to provide the service.

## Online Serving

The objective of online serving systems is to make it very easy, very quick and very cheap to serve extremely large amounts of data. There are two patterns of access:

- A query to find a list of matching resources,
- A lookup to retrieve one exact resource.

These two represent two different access patterns. A highly efficient index (or reverse index) should provide the ability to scale linearly with both the corpus and the QPS. This leads to designs of farms with columns and rows (or shards and replica). But at the end of the day, what makes these systems scale horizontally at linear cost is that write operations have a fixed O(1) cost, so that the time is spent on the query, as O(n).

Google's web search systems create the index offline. This index consists of thousands of segments files created by batch (or streaming) processes, that keep being pushed to the serving systems all the time. The segments are never updated, they are always replaced. At Yahoo, we used to compile the segment data structures into code, for an additional performance gain. What matters here is that the read operations are extremely cheap and the indexing is not done while serving. When the search query comes, the operation to find the matching range in the index is very cheap and fast. Finally, for serving, a document store provides fast look for the indices of the content returned via the search query in the index. This split of responsibilities (document index vs document store) is another trait of simplicity.

On less extreme architectures, we have Elasticsearch which leverages Lucene under the hood to provide its serving functions. Each shard contains Lucene segments. The optimal architecture for Elasticsearch is one where we have enough replica to do the indexing with no live traffic, and once indexing into the Lucene segment files ends, reroute live traffic to these replica. Indexing keeps happening on nodes without traffic. Read and write workloads do not overlap.

A similar architecture is found in Instagram. When you load your feed, you are only querying your user profile and your user feed storage. All posts from your connections have already been written to your feed, and new posts arrive the content nodes keep updated and rotated. Twitter follows a similar process, but they mix reads and writes. Twitter works off an in-memory only (Jedis) store for the content, but this is not the "source of record". When a cache goes down, Twitter replaces it from the persistent store. Because of this, Twitter writes twice, once to the persistent store, and once to the in-memory copy. Since it's RAM, and just a key-value store, the contention between reads and writes is minimal (threading bugs aside).

As a summary, when trying to achieve large scale we should focus on serving architectures that:

- Operate on specialised indices
- Never perform full table scans
- Separate document index from document store
- Separate write workloads from read workloads, particularly avoiding any write workloads at all to happen on serving nodes

## Lambda Architecture

The "lambda architecture" represents an elegant abstraction of how batch and streaming services can work together with each other (I presented at the Hadoop Summit 2013 keynote a concept called "[continuous computing](http://radar.oreilly.com/2013/06/moving-from-batch-to-continuous-computing-at-yahoo.html)", which took the lambda architecture and pushed it forward to combine the use Storm within Hadoop).

At the end of the day what non-serving production systems need to do is to allow the data coming in to be processed, analysed and projected in ways that the serving systems can use.

In the case of Google Web search, the crawlers are bringing in new page documents, and the serving systems are bringing in user events. A large and complex number of pipelines are responsible for processing this vast amount of data (130 billion events per day and 200 billion documents in the corpus in 2014). Some of the data takes longer to process, and requires waiting for all events to arrive (map-reduce); whereas some of the data comes immediately and can be processed one at the time (pipes-and-filters).

At a high level what we have is:

- One or more queues receiving incoming messages
- A batch architecture

  - A number of queue consumers aggregating messages to be ready for batch processing (since distributed map-reduce requires large files to perform well)
  - Batch processed running at regular interface processing these accumulated files and producing a projection of the data (e.g. the web search index files, or the user content stores)
  - The storage to support the batch process is designed to allow failures. The main workload is very large sequential read and write operations. Random access is usually non existent (and sometimes forbidden via firewalls and proxies, which is a problem for some problems like geocoding or map-matching). But at the end of the day, these are cheap disks, a lot of cheap disks, that together provide the necessary IOPS and durability.
- A streaming architecture

  - A number of queue consumers transforming incoming messages into tuples
  - An event-driven architecture dequeuing tuples from the sources (queues), and moving tuples from node to node till they reach the sinks (serving)
- A multi-phase ranking architecture

  - Batch and streaming projections are combined together at serving time using a multi-phase ranking architecture
  - Queries fetch data from the indices produced by the batch and streaming processes
  - A 2nd phase ranking performs the index merge
  - Finally, documents are retrieved from the document store

Twitter posts come to a queue which then feeds it into a Hadoop cluster accumulating the posts, where the trends are computed (and go from there into Cassandra). The same feeds are sent to a Storm cluster to compute updates. The writes are performed directly to the user stores and to the in-memory cache for all followers (except for celebrities that would result in millions of write operations). At read-time the Jeds results are mixed and re-ranked with the Cassandra results. Facebook and Instagram follow a very similar architecture and posts are copied onto all the follower feeds, before the serving happens.

In summary, these systems:

- Optimise for bulk, burst sequential writes
- Never perform upserts, only inserts
- Work with pipelines using queues and events, or map-reduce
- Produce as output the necessary document index and document stores required by the serving systems

## Data Analytics

One particular subtype of the batch and streaming systems are the data analytics systems. Whereas the lambda architecture does not make the "data lake" available except to programs, the data analytics systems expose the data as a first class citizen. This imposes additional requirements for access, either human-time or minutes, rather than hours, and for workloads, random reads rather than scans.

Data analytics otherwise follow a similar architecture to batch and streaming. The projections are computed regularly cheaply, and continuously updated on top. Dimensional and fact data is built into data marts that can then be used to explore, drill, etc. The storage systems that allow these type of operations are expensive, and usually powered by column storage models, such as Vertica.

## Backoffice Systems

Backoffice systems are necessary to edit the dimensions and configuration of the system. A CMS is a backoffice system (but not the serving of the pages). Google Ad Manager is a backoffice system. The user expects that writes are immediately recorded and visible on the next operation. Strong ACID is required. Relational databases are usually the best and most straightforward solution for this type of systems.

Backoffice systems also ingest data from the batch and streaming systems, as well as some from the data analytics. Editors modify and configure the system, and their edits are captured in a database providing ACID guarantees as they do that. External jobs are used to push the data out back to the batch and streaming systems, so that the configuration and dimensional data can be used in jobs and exposed for serving workloads.

## Conclusion

We have seen that there are four main types of data access, and that each of them leads to an architecture and a choice of storage systems. The boundaries between these four groups of systems are better tight, so that there is no leakage of abstractions between them that would lead to a degradation of performance, increase in complexity and reduced scalability.

We would like to embrace these architectural principles of design, based on decades of experience from large scale hi-tech companies, as we move forward building the world's largest crawler, index and search engine for road data. Such a task requires extremely high scalability, at very small unit cost of processing.

When I joined Yahoo, one of the things that surprised me most was the sentence "developers are cheap, systems are expensive," which is exactly the opposite of what I used to say and hear before I join. This obsession with simplicity, performance and cost is what defines these hi-tech companies and the only way to provide such systems at world scale. It's on us to now take on these learnings and apply daily to our craft, so that the architecture, designs and code we produce are simple, performant, and highly-scalable at very low unit cost.
