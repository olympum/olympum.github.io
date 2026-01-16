---
title: "The Harsh Realities of Self-Driving Cognition Development"
date: 2016-11-01T15:41:22+0000
categories:
- autonomy
author: Bruno Fernandez-Ruiz
---

![Image](https://cdn-images-1.medium.com/max/800/1*d4q_56xEp-R24lYxpr_jRA.jpeg)


## **The Harsh Realities of Self-Driving Cognition Development**

*I often get asked the questions “why are you building a dashcam network?”, “How do you intend to fulfill your vision of zero road fatalities?”, “What is so smart about dashcams anyway?” Enough with the questions, it’s time for answers.*

Eran Shir and myself founded Nexar in 2015 to embark on a journey to build a network of connected vehicles; we are building the first vehicle-to-vehicle network, the fundamental nodes of which being dashcams running on smartphones. But that is just the beginning. The camera’s artificial intelligence road-scene understanding is constantly developing and is increasingly able to warn and guide drivers away from dangers.

The almost-joke I usually make is that we are building a self-driving car without building the car. Which is mostly true. Through the dashcam app we are collecting driving data so that we can learn how humans drive and predict dangers. We have observed, recorded, and analyzed over 500,000 incidents, near-collisions and crashes over the course of 22,000,000 miles driven by our users in highway and urban environments. We’ve seen the good, the bad, the ugly of the road — people helping each other and hurting each other, we’ve seen unbelievable piles ups and mid-highway technical breakdowns. We’ve also seen a myriad of obstacles on the road, from potholes, to trees and even cows or monkeys.

But how much data is enough for a computer to learn how to drive? The [research by RAND Corporation]() says that “*autonomous vehicles would have to be driven hundreds of millions of miles and sometimes hundreds of billions of miles to demonstrate their reliability in terms of fatalities and injuries.*” Whilst one may disagree with the numbers, we all agree with the need to have autonomous drive algorithms train in real-life conditions.

Perhaps against what some industry experts forecasted a few years back, the technical community is now realizing that we are in fact closer than we had predicted to having autonomous vehicles on the road. We’ve just recently seen the 120 mile exit-to-exit delivery of Budweiser beer in an [Otto self-driving truck]() over the highway part of a pre-mapped journey. But the hard questions still need to be answered when handing over the keys completely to a door-to-door urban driving car. How would the autonomous vehicle licensing authorities know that the level of safety required has been met for all-weather, all-lighting, all-road conditions? And how would such level be defined?

The volume, variety and velocity — the three so called Vs of big data — of data collection we record at Nexar are unique: we know how humans react in all these situations, whether just cruising or avoiding an immediate danger, and we are using this to train algorithms to learn how to recognize dangers and obstacles and perform optimally at a critical moment.

The two elements that are required to reach the level of commercial vehicle safety are a visual driving cognition as the keystone and additional sensor data fused into that cognition to remove blind visual spots. I believe that no single party can address this challenge alone, and this is why I am really excited for Nexar to have joined Berkeley DeepDrive (BDD), an Industry leading Consortium led by Prof. Trevor Darrell, to develop deep automotive solutions using artificial intelligence and deep learning. Our partnership with BDD allows us to leverage our data library and participate in cutting-edge research in computer vision and automotive, alongside our industrial partners as Volkswagen, Ford, Toyota, Samsung, Bosch, Honda, Hyundai, Qualcomm and NVIDIA.

The research coming out of Berkeley in cross-modality transfer learning will allow us to use the phone’s Inertial Measurement Unit (IMU), GPS or supplemental radar sensors and relate these to the video stream, and vice versa, e.g. for learning object detection and fine grained segmentation without having to manually annotate millions of hours of video. Additional research in visual perception and tracking allows us to learn from millions of hours of driving to infer motion, and generally develop a true autonomous driving policy. We hope others will do the same and join this movement to contribute more visual data and annotations to build better autonomous driving algorithms, which will contribute to preventing the loss of human life whilst driving and reducing the personal and societal costs of road casualties.

The road to autonomous commercial vehicles is long and the next five years are critical to shape the industry, but what is really exciting to me is that these applications are not years ahead, waiting for new smart vehicles to come equipped with a full suite of sensors and systems. The beauty is that the work coming out of BDD is also applicable to every vehicle on the road riding with a smartphone: we can identify dangerous objects in each video frame, we can segment at the pixel-level looking for known object classes, we can detect and recognize vehicles, their positions, speeds and trajectories, we can reconstruct a 3D view of the world, we can guide a network of vehicles based on a real-world learnt policy.

More importantly, as we move forward together with BDD, we’ll help prevent accidents and save lives.
