---
title: "IoT Needs Edge Compute, and More Than You May Think"
date: 2021-01-28T10:18:03+0000
categories:
- autonomy
author: Bruno Fernandez-Ruiz
---

## IoT Needs Edge Compute, and More Than You May Think


## The First Internet

The computer industry of the 1980s was based on a simple premise. It saw continuous hardware improvements that led to exponential growth in computation capabilities. This made it worthwhile to create high-value software.

A decade later, in the 1990s, the Internet and HTTP became mainstream, computers connected to one another, and we were no longer limited to the information in our local drives. Rather, we could access and download information across the world. Throughout, the user paradigm was not much different than the one of the client-server architecture of the 1990s which had replaced mainframes. In the First Internet, we used our PCs to access and download information. The heavy lifting was still done, for the most part, on our PCs.


## The Second Internet

But compute capabilities continued to increase to a point that the common software running on a PC could not realistically harvest all this power. Virtualisation technology appeared as a consequence to allow multiple operating systems to run concurrently on the same hardware, and with it came the proliferation of server farms and eventually the cloud business model. In Oct/2010, Ray Ozzie wrote a highly influential public memo, on leaving his job as Microsoft’s CTO. It was called the “Dawn of a New Day”; in it he called out the inflection point of the client-server architecture in favour of the new paradigm of “continuous services and connected devices.” The client-server paradigm saw its end in favour of the Second Internet, a model where lightweight clients accessed information that was heavily digested and processed in the cloud, on Google’s and Amazon’s computers. We just use our phones and tablets to access this information. The heavy crunching, the sorting, the storing and petabytes of videos is done in the cloud. Today, the device we hold in our hands is simply a remote control, a client, to the brains sitting in the cloud, the server. Directories, web search, messaging apps and social networks provide a thick network of information, and our phones are the remote controls to access all this information.

And by the same Moore Law that saw transistor density double every 12 months over the past decades, these phones that we use as remote controls have actually become tremendously powerful. If we had a time machine to go back to the 1990s with our tablets and phones, they would be considered super-computers back then. The processing capabilities in these devices are huge, not just compared to those of two decades ago, but also relative to what we find today in the servers in the cloud. If we look at Apple’s most powerful computer, the Mac Pro 2019, it achieves a Geekbench score of 1130 in single-core, whereas an iPhone 11 Pro Max scores 1326. The iPhone is equipped with an Apple A13 Bionic, rated at 6w, compared to the Mac Pro, which comes with an Intel Xeon W-3275M, rated at 205w. At 188 points/watt, the iPhone is almost 30 times more power efficient than the Mac Pro, sitting at 6.5 points / watt. Even when we look at multi-core performance, the iPhone is 5.9 times more power efficient than the Mac Pro. The device you carry in your pocket is actually more powerful than the one in the cloud!

With all this computing capabilities at our disposal, how’s the paradigm changing this time? Shouldn’t we see a shift towards a model where most of the computation happens on our mobile phones?

Well, no, and the reason for that is that networks have not improved over the years as fast as CPUs. Even though you carry a small super computer in your pocket, the Internet isn’t capable of supporting everyone to continuously download and upload orders of magnitude higher than Netflix video streaming, to then crunch it locally on their phone. Additionally, it is questionable whether it makes sense to distribute the data this way, as there isn’t a clear advantage to the end user.


## The Third Internet

This is where the Third Internet, the Internet of Things (IoT), comes in. Millions of sensor devices are entering our lives, on our wrist, in our homes, at the office, in our car, in the cities, … These IoT devices are either fully connected with some network technology, such as Z-Wave, Zigbee, LoRa … and all the way to LTE for IoT devices. Across the board, all these devices are collecting immense amounts of data. But both because of the need to limit network use in order to preserve battery power, as well as the asymmetric design of the consumer-side of the Internet, which is optimised for download rather than upload workloads, these IoT devices are unable to funnel all the data they collect up to the cloud.


![Image](https://cdn-images-1.medium.com/max/800/1*ui7WzuaYsQKtlP43cOqBKw@2x.png)

In order to address this network limitation, IoT devices crunch the data locally, using those super power-efficient chipsets. At the same time, in absolute terms, the Mac Pro is still 6 times more powerful than an iPhone 11 Pro, and as a result some of the complex tasks can’t be performed locally on the iPhone. We still require uploading data to offload some number crunching to nodes with a much higher compute density. Some workload can be done locally, but some requires offloading it to a bigger computer. Rather than uploading the data to a far away data centre where the cloud provider operates these mega servers, which can be prohibitive cost-wise, is there a better way, where we could process the data locally, perhaps in a nearby facility? Meet Multi-access Edge Compute (MEC).

Just like Content Distribution Networks make Netflix and YouTube possible, by pushing the download and caching the content to a server nearby the consumers, MEC provides cloud-like compute services nearby where the IoT data is being generated, so that it can be crunched there, locally, and only then the derived works move upstream, either to the cloud or broadcast to other clients near the MEC. These are “Content Aggregation Networks,” gathering partially processed data from the IoT devices, generating information that can then be cached and distributed, either to other IoT devices, or to the cloud. At the end of the day, most of the information about your home is only relevant near your home, or in a smart city, within that city.

By carefully trading off computational and network limitations at the edge, and blending in the available resources into a continuous funnel of compute-network, the Third Internet enables the viability of valuable smart services that use all these smart field sensors. This Internet is no longer one of downloads and streaming, but of applications and services collecting and analysing the real world. Continuous services blend in computational resources on device (“consumer edge”) and the MEC (“provider edge”). Just like previous improvements in computational capabilities led to PCs, the Internet, cloud services, the current improvements in power-performance of these chipsets finally make it possible to build software for the real world, at the edge.
