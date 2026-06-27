---
title: "AI, Moore's Law, and Video Tapes"
date: 2023-02-07
categories:
- cloud
author: Bruno Fernandez-Ruiz
---

*With contributions from Sharon Barkai.*

The implementation of Moores law, which states that the **capacity** of computing power increases at a reduced **cost**, has been realized in the past two decades through the networking and connected concurrency. In the current centralized internet (cloud) the system uses millions of aggregation cores “Hadoop-ed” by microsecond hyper-scale fabrics to anchor billions of mobile cores, in order to perform tasks such as global social networking and content searches. The massive increase in compute power is partly paid for by the efficiencies economy of scale, and partly by draining viability of whole legacy industries.

This evolution faced challenges in the past ten years with mobile content consumption mostly due to concurrency bottlenecks. The two inherit volume bottlenecks in the system are in the cloud fabrics and the network links from mobile to these fabrics. **Before volume** the speed-up fabric compared to mobile latency meant **there was no concurrency problem**. These challenges were addressed by utilizing content distribution caches and off-path resolution methods like DNS and HTTP redirect. These eliminated the data-path bottlenecks by replicating media at the edges on-demand, reducing the volume strain on the cloud and mobile-cloud links. The solution gave rise to new industries such as content streaming, TikTok, and Instagram, so as bottlenecks were eliminated and growth in concurrency continued the expansion was paid for by media content driven targeted ads and subscriptions.

The rise of **AI technology** and the generation of large amounts of data from mobile devices' e.g. preprocessed sensory inputs, have posed new challenges to the current system. The solution which is used for content consumption is not effective for addressing these challenges. Off-path resolution methods rely on mobile client based mappings. As the produced content constantly changes each consolidation context must persists. Therefore off-path resolutions generate a massive coherency problem, where simple rebalancing of persistent context in order to address demand patterns invalidates the mappings cached in billions of clients. This leaves on-path resolution as the only viable option.

On-path underlay resolutions like Anycast may introduce complexity to the basic interoperable connectivity and are not suited for the scaling dynamics of such consumer scale magnitude. On-path overlay resolution methods, like LISP or SFC, which allow for the resolution of contexts within the network are more promising. They allow the network to reduce the volume of data at the first addressable context post mobile-access by pre-fetching all necessary meta- data and functions (FaaS) to these contexts. This is done without involving clients and without interrupting basic interoperable connectivity.

The basic scale and capacity of on-path overlay approaches are proven in the IT-fication of mobile network functions (NFV) and the new 5G core, yet need to be further evolved for prime time content producing application networks.

Such evolution is demonstrated by designing the **context and addressing** schemas in a manner that shards and reduces the data **per domain**. In other words, in-order to keep growing compute concurrency we again must eliminate blocking, but unlike generic consumption we need to figure addressable shards by the (producing) application domain. For example geospatial addressing for dynamic street-mapping, in the automotive edge, is a basis for concurent non- blocking consolidation - of the AI data constantly produced by moving vehicles.

The costs of extra edge compute leverage established cloud clusters and mobile smartphone economies of scale, **but** costs must also be offset by the economy of power-and-cooling enabled by the reduced concentration. In **addition** each AI production application needs to identify which legacy economy is drains to pay for the increase in compute concurrency. Typically this will involve eliminating manual labor. For example cartography surveys and annotation by v2v vision dynamic mapping networks at the automotive edge.

In conclusion, the current centralized internet system was successful in addressing the challenges posed by mobile content consumption. Consequently content streaming and media economies grew along with compute concurrency at the expense of legacy industries. However the **consumption** technological solutions as well as the business models may not be relevant for the new challenges posed by AI and mobile content **production**. To address these new innovative network computing and business models may be necessary.
