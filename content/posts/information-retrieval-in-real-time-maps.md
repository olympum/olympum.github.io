---
title: "Information Retrieval in Real-Time Maps"
date: 2022-04-04
categories:
- Architecture
author: Bruno Fernandez-Ruiz
---

As I wrote in a position paper for our real-time mapping work, the act of building a map of the world, albeit a real-time one, is an information retrieval problem. The target domain, the roads of the world, is one of the largest information retrieval problems in the world, orders of magnitude larger than Google’s internet search index. But by formulating this as an information retrieval system, we can then think in IR terms.

# Background

So far we have focused on creating a platform responsible for anything that it’s “non client.” Large part of that stems from organisational history. However, we are are starting to see fractures in this model that require addressing:

- The work of the client(s) needs to be tightly coordinated with the server. This has become more obvious as we try to optimise the yield of data collection (for reference, I refer to yield here as the conditional probability of detecting and having image evidence for any change on the road we are interested in, given that a visit by a Nexar car). We prepare “deficit” tables on the server that inform the client about what to collect, and the client is supposed to upload data if and when it passes those locations.
- We have more heterogeneous clients and architectures. We need to be able to consume this data, but in a smart way, so that cost does not quickly balloon.
- The skills required to run inference and analytics at the edge, whether on-device or the MEC, collect the data and move the right data are very different from the skills required to index and serve this data. In web search terms, the crawlers, the index creation and the serving are three different teams with very clearly defined boundaries and interfaces. This allows each team to focus on their individual KPIs,  which makes it easier as a whole to drive lower costs.
- We are moving from a model of cloud technology running on the cloud provider’s own data centres to one where the cloud technology can run on any data centre, e.g. AWS Outpost, Local Zones, CloudFlare Workers, etc. Not only this helps reduce latency, which we don’t quite need yet but we will need at some point, but most importantly it can help significantly reduced costs, as we don’t need to backhaul everything to the core of the Internet, and our storage requirements are not very strong, we just need some evidence for a while, not forever, and evidence about San Francisco is only useful in San Francisco. The temporal and highly localised nature of our data makes the architecture highly distributed but extremely easy and cheap to scale. This is of course unlike our user’s content, e.g. collisions, routes, etc. which still needs to go back to the core of the Internet, to the region.

Even though I have been talking about cost for a while now, we should put a huge emphasis on cost going forward. And this is not because it costs us more or less every month, and we don’t like big AWS bills. But because cost is our main competitive advantage. Only by having a fraction of the cost of our mapping competitors, we can do what we do, and offer an order of magnitude better coverage and unparalleled freshness. Decisions on small costs quickly sum up at a very large scale. Even though for example the cost of storage for doing work zones in Las Vegas with option A vs option B might only be $100 vs $200, so we would ignore the $100 here, it’s twice the cost. And when we move from Las Vegas to the US, or worldwide, we see that we need to process 1.2 billion times more data (billion!) then suddenly those $100 become the deal breaker that make our business model feasible, or not. Once we have such a cost, we can offer much higher coverage than anybody else.

# Information Retrieval

Everything we see on the road is a document. A document has features (time, location, speed, acceleration, heading, detections, etc.). Our first objective is to optimise the recall, we want as many documents as possible, and miss as little as possible, in order to build the most comprehensive possible corpus (even before starting to analyse the documents to put them in an index). The role of the acquisition systems (i.e. crawlers) is to build the right technology, at the cheapest possible cost, to get us the data we need.

Once we have amassed a huge corpus we need to make sense out of it, this is the indexing phase. This index is the full twin of the world, and it’s an extremely large and complex graph of the world. This is not an artefact we can access easily, and traversal is costly and expensive. But it’s this massive index what allows us to provide intelligence about the world. The index itself will most likely be sharded geographically, but as a whole it’s huge. The role of the indexing systems is to build this huge index with everything we know about the world.

This massive index needs to be projected into actual serving indices we can use to serve the needs of our customers, whether our cameras, other vehicles, or B2B partners accessing our data via APIs. The role of the serving systems is to build the APIs, data stores and services required for our customers to access the data.

On top of this, we find additional cross-organisational requirements:

- The need to analyse data quality end-to-end
- The need to create new “applications”, which impact how the acquisition, indexing and serving operate.

# Architectural Overview

Web search and all information retrieval architectures follow the [architectural patterns for scale](/posts/architectural-patterns-of-scale/). To create an IR system, there are three separate tasks:

- Gather the corpus
- Index the corpus
- Serve queries against the corpus

Each of these tasks results in a separate set of subsystems with clearly defined and tight boundaries. The idea is that the design and scalability of one subsystem should not impact the others. We can rewrite the serving without impacting the index. It’s also prudent to think in terms of organisational design (following Conway’s Law) for each of these subsystems.

Moving from the 50,000 ft view of the architecture to 12,000 ft, we would have the following systems:

Let’s break down the responsibilities:

- Road crawlers are responsible for coverage and freshness at low cost. Road crawlers optimisation objective is to maximise the conditional probability of having a true positive or a true negative for any driven at least once by a Nexar camera (aka “yield”), where an event (true or false) is a frame or a detection, subject to a number of constraints, out of which cost is the most important one (taking into account device CAPEX costs, network OPEX, server CAPEX/OPEX, etc.). All frame analysis (including inference) is done in the crawlers, whether device, MEC/edge or cloud region (see below).
- Indexing systems are responsible for taking the per hexseg analysis (not frames, just metadata), ie the true positive and true negative events from the crawlers, and proceeding to localise, aggregate and analyse change over time, and from that creating the “Digital Twin”, usually in data stores that are highly horizontally scalable (“data lake”) with no real-time access guarantees. All this information in the data lake is then projected into a strict index that can be used for serving.
- Serving systems are responsible for taking the index created by the indexing systems and providing the necessary external query API and document fetch API. A query is “find detections such that …” or “find frames such that … “ and a document is “get me this detection …” or “get me this frame ...” It’s important that realise that the index is NOT the document store

# The Road Crawlers

As we build the map, we start to understand in which areas we have enough data (coverage), and recent data (freshness). We can infer where we can have deficits (analogous to Google inferring the web crawlers need to visit CNN every 2 minutes, whereas Bruno’s blog only needs to be visited once a year). These deficits are computed per hexseg (unlike Google that tracks unique URLs each being a document, our documents are hexsegs). We want to know everything in a hexseg document, but for that we need to instruct our crawlers (cars) to pay attention when they drive and collect images. This is a pro-active model: I am instructing the cars to collect frames in the future.

There is also a re-active model, we discover the deficit in hexseg documents after the fact, and we need to instruct the cars to give us frames that they have seen in the past. We can now remotely access the client and ask them to upload exactly what we need. Instead of greedly uploading everything to S3 and then seeing what it is we need, we can do this remotely with our clients. We need to be aware of latency, and that it will take them time to maybe be able to connect and upload the data. But that’s part of the optimisation problem to be played.

Then there is the greedy mode. Where if we can, because we can, for example when connected to home WiFi, we can upload a load of evidence.

And finally there is the intelligence at the device itself, running neural networks to find out things outside of the deficit, discovering new documents (same as crawlers discover new links).

Putting all these options together, the objective of the road crawler is to analyse every hexgseg of the road, and doing this requires figuring out *what* to transfer, *when* to transfer it, and *how* to transfer it.

As for what, the decision of what needs to transfer goes as follows:

- As input signals coming from the device in real-time over the V2V channel:

  - There are inference on-device detecting objects on the road.
  - There are GNSS traces and IMU signals.
  - Frames are being cherry picked and cached locally on-device, based on heuristics for local collection candidates (e.g. detection above threshold *or* detection way below the threshold)
- The control layer on the server which tells the client what to transfer:

  - V2V data is being sessionised per device and per hexseg. Sessionisation includes snap-to-road or map-matching.
  - At the device level, if the server learns that a device has seen something that it was not aware of, then it mark a particular frame as a candidate to be fetched from the device to the edge-crawler.
  - At the hexseg level, if the server has prior knowledge of a road feature, which should be present on a hexseg, but a vehicle went by and nothing was collected, then the server should mark a particular frame as an upload candidate; similarly if the server has a deficit of evidence frames, it may mark frames for upload
- The network layer for the server to instruct the client what to replicate/transfer (feedback loop)
- The client devices can then transfer the candidates the edge-servers marked, within the *when* and *how* budget and time constraints.
- As actual frames get uploaded, an inference layer on the edge-servers:

  - Further localises the objects based on visual analysis of the frame
  - 3D analysis if required
  - Extract embedding vector(s)
- The results of this analysis from the road crawler are:

  - A stream of highly localised detections, with all the metadata of the visual analysis
  - A number of cherry picked evidence frames

So to summarise what’s in the road crawler:

1. Pro-active dynamic FOD. Computing the areas of deficit, per application, and determining the most efficient and cheap way of gathering the data. We need to start being smart here and take into account mean time between visits and how people drive. If I am the only one who sees Pearl St daily, but I also drive the I-93 like another 10000 people, we want the transfer budget to be optimised for Pearl St, since that’s the rare data. This work needs to be done on iOS and Android.
2. Re-active dynamic FOD. Computing the areas of historical deficit, on a retro-active basis going back and asking cameras to send us data as soon as they can. Here we need to use the same deficit, and budgets per user, as well as user patterns to optimise the remote fetch and access, but also take into account the reachability of the user device (for example what’s the expected time till the camera becomes reachable again, can we wait that long or try with another user; should we ask several users in parallel even though this is costing us more cellular; what’s the expected upload rate)
3. Aggressive bulk of upload of image frames and short videos, so that we can discover “new things”.  There is a lot to be explored here.
4. Neural networks to identify interesting things as we drive. This requires fine tuning and training models, building the  server-side MLOps infrastructure to not only cross-compile to the target architecture but also to support OTA of models, A/B testing, canary testing, etc.
5. The infrastructure to optimise what gets sent. If I drive alongside a construction zone, which of the evidence in the past 100m is best to send along to the server.
6. All the server-side components, aside MLops, including those to keep the inventory of gaps and deficit, coverage and freshness, in order to instruct and guide the crawlers. As well as the queuing and storage required to expose all these frames onwards to the indexing, aggregation and serving systems.
7. The server-side inference systems, in case the device did not get a chance to run the NN, so that the input into the indexing systems is simply all possible documents found, and no further inference is required.
8. The MEC-side inference, deficit keeping and lambda (“data reduction”) systems, to allow sharding the crawlers into hundreds of small data centres.

# Indexing Systems

The indexing systems receive the data from the road crawlers. Their objective is to create a digital twin representation of the world using the stream of detections being generated by the road crawlers. The main complexities of these are:

- Scaling the indexing to global scale. There are 1.2 billion h3.res12 hexsegs with roads in the US. As we do res13/14/15 and globally, we get to an index with over 1 trillion documents.
- Scaling the analysis itself to global scale, by being able to aggregate and perform change detection on all those billions of hexsegs (documents are hexsegs rather than “pages”)
- Creating a non-ephemeral representation of the layers in the map, by having an online aggregation so that we can associate the detections from the stream with objects and product a reliable change detection output
- Further analyse and index the embedding space to provide discovery and browsing capabilities over this immense index
- Creating the actual index files for serving. See [Architectural Patterns of Scale](/posts/architectural-patterns-of-scale/)
- While doing all of this at a fraction of the cost of our competitors

# Serving Systems

Finally, the serving systems expose all the data back to API consumers (note that devices themselves are connected over the V2V/nexagons network).

The complexity in the serving systems:

- Providing a service with very low latency requirements that our OEM partners can use in their cloud-to-cloud integrations (as vehicles themselves will be hooked up to Nexagons)
- Scaling the index serving of hexsegs to tens of billions of documents.
- Scaling the index serving of detections to tens of millions of documents.
- Scaling the CDN for image serving to petabytes of data
- Providing both REST API and bulk-data APIs
- Doing this at a fraction of the cost of our competitors.
