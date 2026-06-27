---
title: "The Ecosystem Keystone"
date: 2022-10-24
categories:
- essays
author: Bruno Fernandez-Ruiz
---

An ecosystem grows a network faster than any company can grow it alone — for Nexar, at least 10x faster, through rich multi-lateral B2B relationships and partnerships. Ecosystem deals expose positive externalities: the sum of the parts is greater than the parts alone. The question is *what* and *why* the ecosystem can work, which then leads to *how* to make it work.

In high-tech, healthy ecosystems are characterized by a “keystone”. As partners develop 1:many relationship within the ecosystem, there is always an invariant, which is the keystone. In Intel’s ecosystem, the keystone is the CPU. There are then value-added products on top of it, from components and drivers, operating system(s), application software, computers, etc. Each player develops its own partnerships, e.g. HP with Microsoft for OS, HP with Samsung for SSD, etc. but it must always revolve around the keystone, the CPU.

What does it take to become the keystone of the ecosystem of driving? Several angles matter:

- Growth in consumer demand for IoT devices present in the home and in the vehicle, from wearables to car accessories, which require some form of AI running in the car (at the edge), and the corresponding network connectivity between devices and from/to the brain (cloud). Within this class of IoT devices, growth in consumer demand for cameras in vehicular environments (fisheye, inside camera, rear cameras, CCTV ,…) is growing very fast.
- Making claim evidence from the camera actionable by streamlining the carrier’s claim settling.
- Creating a technological competitive advantage that allows extremely low latency locally-aware content sharing between vehicles.

# IoT in the Car

Consumer demand for cameras in the vehicle is growing dramatically year-on-year. Even some early adopter automotive OEMs are starting to include driver-accessible dash cams (compared to safety and parking oriented cameras in the car, which are locked out).

- Car cameras are currently disconnected. To move data, the user needs to connect the camera via USB or an SD card to a computer, and in some case via a companion post-ride app. By connecting a Nexar compatible camera to the network, there is no longer need for USB or SD card, and we can remove these “guardrails”.
- Manufacturers for car cameras are stuck currently in what is essentially a *hardware-centric device*, not a *service-centric device*. When consumers buy cameras, they are buying disconnected systems with limited (*dumb*) application functionality. In contrast, consumers have become used to service-centric *smart* devices, ala Alexa, HomeKit, etc.
- In a service-centric world, manufacturers still want to differentiate themselves from each other, which includes the layer of “smartness”, by providing unique applications they provide, e.g. police vehicle detector, navigation, diagnostics, telematics, etc. But to do this, they all need a smart connected hub in the vehicle, since the cost of AI in the camera is much higher than riding the wave of AI technology investment going into mobile phones. So, although some AI may run on the camera, the price per FLOP will always be higher in a camera than in a phone.
- Hardware margins in cameras are razor thin. In a low-entry camera, a very, very, good manufacturer with a strong brand name might be able to sell (and really, this is a truly best-case scenario since most margins are single digit). But once they add recurring revenue from service fees, as small as these might be, this would translate into a jump in profitability for them, which is currently totally unheard of.
- In-vehicle connectivity (e.g. vehicle hotspot) has not caught up with consumers, even though most new vehicle units sold in 2017 were equipped with a GSM/LTE modem. Similarly, it’s unlikely that any other form of vehicle hotspot will catch the interest of consumers, e.g. cameras with built-in LTE are likely to remain a product targeting commercial fleets. It is very likely that the mobile phone will become and remain the smart connected hub in the vehicle for years to come.

In summary, manufacturers see and envy the trend towards smart services occurring other classes of consumer electronics, but they lack the expertise to transition from their current hardware-centricity to the new service-centricity. They need a keystone to enable this transition.

# Actionable Evidence

The main reason, at least currently, for installing a camera inside the car is capturing video evidence in case of a collision. But when the actual and unfortunate event of a collision occurs, the camera is unable to do anything with the video, and a manual, cumbersome, fragile, and complex process starts.

A smart camera can radically transform the process by closing the loop and making the video evidence actionable, which requires (i) the automatic detection of the collision, (ii) the collection of all the necessary vision and sensor rich evidence, (iii) the analysis and automatic interpretation of the events leading to the collision and associated damages and losses, (iv) the transfer of all the collision evidence to the insurance carrier as a claim submission, and (v) the settlement of the claim back to the user. If we can close the loop, the camera shifts from being a camera lense with an attached SD card, to a smart IoT device creating a completely new and highly efficient process to solve claims management.

Insurance carriers have strong incentives to promote this new automated flow (alike how the introduction of personal computers if office environments changed how clerical filing work happened, using cabinets and folders). Most carriers however lack the ability to execute the steps required to make this transformation from today’s manual claim management process to a new smart and automated process (without manual guardrails). To make this transformation true, one must:

- Work with manufacturers to define the data evidence formats, data quality, connectivity, and data protocols for moving collision evidence from the camera to the cloud.
- Develop and grow the availability of AI services that are able to capture, transform and ingest video camera data.
- Scale the supply and demand. A manufacturer:carrier one-on-one relationship is possible, but it is hardly scalable. One carrier can at most work with one camera manufacturer with the help of a systems integrator to develop a smart service closing the collision claim loop.

In summary, to make actionable cameras work, both manufacturers and insurance carriers require a keystone. There are also other players in the middle, selling policies, doing claim processing, crash investigation, loss adjustment, claim settlement, … all which is done in a fragmented way today and can become part of the ecosystem.

# Edge Computing

For last two decades, the Intelligent Transportation Systems (ITS) community has been advocating the benefits of Vehicle-to-Vehicle (V2V) communications for safety, energy savings and reduction in emissions. The V2V solution promoted by the US Department of Transformation (DOT) and the regulatory national highways transport and safety agency (NHTSA) is the use of Dedicated Short-Range Communications (DSRC), a WIFI 802.11p based protocol extension for automotive safety applications, which was devised now 20 years ago in the ITS forums under Ford’s leadership. But DSRC is a regulation is flawed with [critical issues](https://www.mercatus.org/publications/department-transportation-v2v-technology-mandate), from an exorbitant cost of deployment ($4 billion per year in the US), to lack of any consumer benefits in the first 7-10 years of deployment, and going through all sort of technical issues including data privacy, security and trust.

Unfortunately, the FCC allocated 75 MHz in the 5.9 GHz spectrum for V2V applications back in 2005 and assigned it to the American OEMs, who want to keep it and don’t care whatsoever if it’s used for 802.11p or cellular communications in these allocated frequencies. But as spectrum has becomes very limited in 4G, and with the pressure of new services on 5G, the FCC wants to release the 5.9 GHz spectrum back, but faces the opposition of the OEMs. The OEMs are lobbying NHTSA to pass the V2V regulation to secure the spectrum, which would lead to a flawed solution becoming the norm. Cellular carriers are obviously opposed to the regulation, and would like the FCC to release the spectrum.

Aside reliving traffic congestion, cellular carriers lack a good reason for how the liberation of the spectrum would lead to both consumer demand, and also the appearance of vehicular safety applications. Cellular carriers have invested in infrastructure to bring compute close to the subscriber, with edge pods near the cellular towers, and avoiding having to route traffic outside the cell onto the Internet. However, they lack the *killer app* to leverage this architecture, since most services run perfectly when hosted in a data centre, and fronted with a content delivery network for proxying and caching. The *good* reasons for leveraging edge compute are: production and consumption of content with local-affinity (things that are only relevant to this cell), and services sensitive to extremely low latencies. An application which requires extremely low latency locally-aware content sharing between vehicles is the perfect use case cellular carriers need to justify and monetise the investment in edge compute. And viceversa, edge compute is the perfect technical fit to the V2V network we are building. And a partnership here would also put the whole 5.9 GHz DSRC debate to rest for good.
