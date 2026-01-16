---
title: "Is DSRC dead? Vehicle to vehicle mobility network interoperability with LISP"
date: 2019-05-06T11:43:52+0000
categories:
- autonomy
author: Bruno Fernandez-Ruiz
---

![Image](https://cdn-images-1.medium.com/max/800/1*wprtNTB2iS4Dx-eyV1cPIg.jpeg)


## Is DSRC dead? Vehicle to vehicle mobility network interoperability with LISP

Toyota’s recent announcement to [halt the deployment of 802.11p]() (DSRC or ITS-G5) vehicle-to-vehicle technology in new cars may come as a surprise to some. But this is a rational and sensible decision by one of the world’s largest OEMs, a natural consequence of the evolution of communication networks, vehicle connectivity, and the introduction of more vision based automated driving features in cars.

The ad-hoc nature of DSRC’s wireless network was attractive to Nexar when we started. From a research angle, this seemed like a formidable challenge (and still is a trending topic for academic research). However, we quickly came to realize that such approach was not feasible in the real world. For a wireless ad-hoc network like DSRC to function reliably, it requires almost ubiquitous penetration, as the protocol is stateless and there is no shared state between the cars, so all information needs to be continuously broadcast. Creating a mission critical automated driving system based on a peer-to-peer, or all-to-all, technology that exhibits random reliability signifies a tremendous barrier of adoption to any automotive manufacturer. We heard from manufacturers that they would happily adopt DSRC … once and if such a network existed. But due to the highly fragmented market share amongst OEMs, no manufacturer can create such a network by itself. So for two decades and running, this issue has been a true game of chicken and egg between advocacy groups, NHTSA and the OEMs — and has hindered any real adoption of DSRC. All historical attempts to legislate DSRC have been faced with technological challenges, budgetary constraints, spectrum wars … all rooted in the flawed conception of the protocol.

In contrast, rather than having an all-to-all communications network, we adopted an approach that enjoys having shared state between the vehicles, and which by design creates network effects from the moment the 2nd car joins the network.

We [launched]() our over-the-top vehicle-to-vehicle (V2V) mobility network in 2016, and three years in, Nexar’s is the world’s only operational V2V mobility network with a meaningful presence in all top 100 US cities. Nexar users across US cities help each other be safer on the road, sharing in real-time information about traffic lights, potholes, road blockages, dangerous intersections, emergency vehicles, and many more.

Nexar’s network was built from the get go to be independent of the choice of physical network technology, whether 4G, 5G, or 802.11p / DSRC. Just like the logical protocol that powers the web, HTTP, has proven to be future proof and resilient to change over the past 30 years, Nexar’s logical network, is abstracted and resilient to the choice of physical communications network. Logical protocols, like HTTP or Nexar’s network, are abstracted of physical layer decisions and automatically improve with the advancement of physical networks.

When we [responded to NHTSA’s notification of public rule making](), we emphasized our support for V2V technology, but highlighted the importance of ensuring technology agnostic regulation. We still strongly believe that regulating DSRC as the technological solution would be costly, risky, and unnecessary. Rather, we sustain, as we proposed in our response to the NPRM, that an approach based on interoperability operating at layer 3 of the OSI network stack, one where the choice of physical infrastructure and frequency spectrum is not imposed, is the only viable solution.

To achieve interoperability the industry requires an open standard working at the logical layer. For the past few months, we have taken steps to open our V2V protocol, and we are working with academia and members of the IETF to draft it into an Internet standard that not only Nexar, but that any telecommunications provider, automotive OEM, fleet operator, etc. can adopt and ensure interoperability, regardless of the physical layer of choice.

The proposed protocol design leverages the work done in the identity-location separation [IETF RFC6830 (LISP) draft](). Identity-location separation completely aligns how we index the vehicles and road observations and how we communicate about them. Unlike in all-to-all networks (e.g. DSRC/C-V2X), an indirection is provided by IPv6 addressable geo-spatial tiles, using [Uber’s H3]() consistent index of the world, that describe the shared state of the road observations. Mobility clients (cars roaming the roads) publish messages to geo-spatial tiles, and subscribe to geo-spatial tiles to get real-time updates of the shared state of the road.

This indirection is a network simplification enabling interoperability with implicit safety and privacy. We can then make trade-offs, either host the geo-spatial tiles at the metropolitan level, to provide a round-trip latency of tens of milliseconds, or host it next to the PDN Gateway (PGW) to leverage the native cellular multicast and compute capabilities, and bring latency down to a few milliseconds.

We are enthusiastic about the prospects and possibilities this standard creates. This level of efficiency, privacy and interoperability is necessary for managing 250 million cars roaming the roads at any point of time across the globe, and being able to finally start making a dent into congestion management and road safety. If your organization works in smart cities, road mapping, DSRC, cellular technology, or any other autonomous solution, I highly encourage you to check the [draft IETF submission]() and get involved.
