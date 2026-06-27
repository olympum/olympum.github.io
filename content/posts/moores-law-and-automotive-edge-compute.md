---
title: "Moore's Law and Automotive Edge Compute"
date: 2023-02-02
categories:
- cloud
author: Bruno Fernandez-Ruiz
---

With the advent of 5G, and the industry estimates for 1000x more data to flow from the vehicle to the cloud, what can telecom operators, and the industry at large, do?

The reason cloud computing work is because we found a better way of sharing infrastructure, increasing average utilisation, which effectively reduced total cost of ownership for those servers.

This was possible because the cost of moving the data was low, and when the latency to move data was low. There wasn’t just enough data, so putting more and more fibre to move data where the compute was seemed like a better solution, as long as both things remained true: cost and latency. Computing got faster and cheaper, and the model seemed perfect. Moore’s Law was allowing this.

Moore's Law can be described as the ratio of total computational operations to total computational costs. This ratio has been increasing due to advancements in concurrency, with billions of mobile cores and millions of aggregation cores contributing to the growth in operations. The cost growth is capped by economic growth which is much slower than computation growing Moores metric.

The rapid growth in concurrency for consumption was driven by the widespread use of video and social networking on smart phones. The need for an aggregation fabric for consumption was removed through the use of content replication, resulting in the decentralisation of consumption aggregation cores, for example AWS S3, to the CDN edge.

Data production volumes, especially when we focus on the predicted growth in the automotive industry, are rapidly growing due to advancements in AI, with the telco’s mobile core processing their immediate environment, and aggregation cores on the cloud consolidating this production and adding generative insights. Unfortunately the capacity of networks does not grow as fast as Moore’s Law, as it is capacity is linear proportional to its deployment costs.

While a dollar we add to increasing computational power yields much more than its cost, the network does not, and keeps lagging behind, getting further and further away. When cars will product so much data that this model will break, and it won’t be economical anymore to lay out more fibre so that we can move the data from the vehicle to the cloud. The only model that will scale with this explosion of data is one where it’s no longer required to rely on the network to back-haul to the cloud all this data. Instead, we need to bring the computing data fabric capacity closer to the data generation.

This shift in big data is not new. Hadoop and BigTable were a solution to the problem of centralised relational databases no longer scaling, at least economically, to cope with the vast datasets that computing advances had allowed the Internet to appear. The volume of data was such that partioning and distributing the data amongst many storage servers, but centralising the computation of the queries, did no longer work, as moving the data from the storage nodes to the compute nodes increased latency and increased costs beyond value, even despite the introduction of fast lane technologies such as SAN. Hadoop reversed the architecture, and brought the computation next to the data. This Hadoop-like distributed compute model is what telcos need to do now. Without it, the industry won’t be able to move and process all those Petabytes, and we need that to unlock so many services and products, including real-time mapping which Nexar is doing, but also to improve autonomy, improve safety and reduce emissions.

If telcos need to invest 1000x more in network deployment to address the 1000x predicted data growth, the connected automotive future won’t happen. To allow the decentralisation of aggregation cores to the edges, we will require new computing data fabric, with fit-for-purpose network programming models, source-routed sharding, and replication of Function-as-a-Service logic. As for this to happen, we’ll need real-estate investments in data centre facilities, and great cross-connectivity, so that this computing fabric can run elastically and dynamically.

Second, we will need to see a shift and significant drop on network transfer costs, which is unlikely to come from operators themselves. This is were the innovation in local metro-level networks will start to pay off, particularly LTE Band48 (CBRS). The aggregation of wireless technologies, in order to provide a spectrum and protocol agnostic service, could finally unbundle and disintermediate the operators. An aggregation service running on this metro-level data computing fabric has very little upstream link requirements (Nexar could run on a metro like Las Vegas on a small 1 Gbps uplink to AWS). Today CDN and edge fabric providers such as CloudFlare are still charging for the bandwidth, due to a “bandwidth neutrality alliance” that protects the interests of the operators, but also makes it no more expensive to route back to AWS than to remain close to the radio network and mobile core. Unfortunately, the jury is still out on this one though.
