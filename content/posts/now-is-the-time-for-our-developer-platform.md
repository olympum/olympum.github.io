---
title: "Now is the Time for Our Developer Platform"
date: 2022-03-28
categories:
- essays
author: Bruno Fernandez-Ruiz
---

Over the years we have evolved a dream, a vision for a safer and more intelligent future, into a company impacting the lives of hundreds of thousands of people who have or are driving with Nexar. We have also built systems to make this dream a reality. We started a team creating our platform, initially simply a team developing the server-side functionality required by the iOS and Android applications. Then we decided to shift gears and move into what we then called the “data platform” and make our platform the foundation for many AI-verticals we would go on and build based on the data that the Nexar network collects.

We initially launched a live map, and transformed how we consumed and processed frames on demand at scale. Then we took on making road work zones become actionable. We also added support for road signs. And we made huge progress over the past 6 months to sieve and derive insights from billions of frames coming in to our servers on the WiFi-uploaded timelapses. Only with this new ability to process these vast amounts of data, we were able to add change detection.

And in between we also tackled things like flexibility, scalability, availability, observability, and cost. There is much more to do here to make the platform world-class, but it is on the right path: a service at least 10x better and cheaper than what any competitor can provide today. Excellence on these non-functional quality attributes is what keeps a platform competitive for the long run.

However, we are at a unique turning point. The premise is that Nexar has a network which scans and maps the world in real-time. Thousands of cameras continuously roam the streets recording and detecting what is happening. We have built technology that is able to sieve through this data, find the signal in the noise, and collect and transform the most relevant data into information: we go from the raw image data seen by the cameras to putting detections on a map. We do not stop there, we further process these detections to go from the inference information to entity knowledge: we know that the detections are observations or sightings of an object in the real-world, for example a stop sign. This allows us to build applications that are actionable and transform that information into actionable knowledge. Our road work zones product does not stop at showing where orange cones are found, it groups them into segments, assigns these segments to a work zone, and puts the work zone into a time of day window. This allows the product to be actionable.

The ability to go from data to information, and from information to knowledge at the scale that we are operate under is only possible thanks to the flexibility and extensibility of the platform we have built. Road work zone detection or a virtual camera would not be possible without the platform we have built. There are many AI-verticals like these that could be built using our platform. We are indeed starting to tackle some of these, such as real-estate property, free parking spaces, etc. And there are many more, such as pedestrians, storefronts, double parking, bike lanes, sidewalk condition, road surface cracks, tree canopies, etc. We know these use cases, and many more we are not even aware of yet, can be cooked (implemented) using with our unique ingredients (our imagery) in our kitchen (our platform), by providing different recipes (models). Our ability to scale and implement all of these is limited, and risky. Anytime we make a bet to build a new AI-vertical, we are committing to building a product, something that does not end in a software project, but requires its own go-to-market strategy, marketing materials, market definition, sales workforce, customer support, etc. The effort and risk in building these AI-verticals is substantial. Each vertical is its own hypothesis, which may work or fail. And failure of the few verticals we can build ourselves would be mistaken with the failure of the network and platform we have built.

Up to now, Nexar has built a platform that allows us, and only us, to build AI-verticals. Now is the time to let other developers leverage that platform to create their own. The next step is to make it a true developer platform, while continuing to improve the AI-verticals we have already developed. These are our best case studies today for the platform we are bringing to the world, and it’s critically important that they reflect all the goodness possible in the platform.

Launching the Nexar developer platform rests on a few decisions and principles. Some are obvious, some are controversial.

*Developer Platform First, AI–Vertical Apps Second*

Up to now, we have gone to market offering AI-Vertical Apps, but we now need to shift the conversation to one where we bring a developer platform which enables our customers to gain understanding of the world through knowledge. The AI-Vertical apps we have developed, and will continue to develop and sell, are positioned as case studies. We always sell a developer platform, and our customers upgrade to include add-ons. These add-ons are our AI-Vertical apps. This is a market positioning strategy, as well as a pricing decision. And in the product, this means one thing: API first.

*API First*

So far, we have built AI-Vertical apps that drove the design of the platform and the APIs supporting these apps. Although the APIs can be extended and reused to a degree, we now need to shift to think API first, and AI-vertical second. The API must support, but not end with the AI-Vertical. Creating true open developer APIs requires its own discipline and expertise, one where we don’t design APIs that are either too constrained and won’t be extensible beyond the initial AI-Vertical app, nor APIs that are too wide and generic and hard to reason about, comprehend, and take a long time to develop. We must continue to optimise for velocity. Practically speaking this means we need to adopt a number of non-functional quality attributes as our guiding mantras:

- Extensibility
- Flexibility
- Composability
- Self-describing
- Scalability

*Complexity and Possibility*

Platforms solve two issues, and we are no different:

- Make simple today what was complex yesterday.
- Make possible today what was impossible yesterday.

Complexity is the main way to get platform adoption. The initial wedge usually is an existing process, with existing tools, developers, budgets, stakeholders … but also issues, problems and complexity. The way to get into the organisation is to become a better alternative, where better for platforms is 99% of the times a synonym for simpler. Here we make processes where today humans must drive, take notes, record images, transcribe into a system, something of the past. We provide a platform that allows developing applications that automate through AI complex processes.

Only once we are in and have a seat at the table of our customers, we will have a license to sell the impossible as a platform.

*Build a Platform, Sell a Product*

Although I said that the developer platform comes first, and the AI-Vertical apps come second, our sales effort must be focused on selling the add-ons to the platform, which are the AI-verticals. But the shift here is that our customers are buying the platform, with added packages on top.

*Developer Portal*

Community management is always critical in reaching developers. The developer audience we are targeting however is not the millions of developers out there who can code in Python or Javascript and create an app using our data. Those may come and use our platform, but they aren’t our initial customer. Our customers are larger organisations already providing analytics based solutions from real-world data, possibly already doing CV/ML with vision data, and that have developed tools and services used internally within their organisations, and most likely as software services to 3rd parties. They become are partners to build AI-Verticals on our platform, that they then take to market to their customers. Our customers are large organisations, and their customers are counted in the thousands. This means that we need to make easier for inbound demand to get to Nexar. We must provide a top-down developer portal that reflects on the value of the Nexar developer platform. This portal not only caters for developers but for decision makers and stakeholders at these organisations. The developer portal becomes our B2B door to Nexar, where we describe the family of use cases and markets the platform can address: auto insurance claim insights, smart cities, mapping, logistics, … Our APIs are mapped to how they enable these solutions. Our AI-Vertical apps (products) are shown as add-on solutions built on top of our platform. The developer portal is self-contained, self-service, and serves as a huge lead generator we can use with our developer partner organisations.

*Standards and Open*

In the eyes of these larger developer partners, we are a small company bringing to the table two assets: a network of vehicles collecting data, and a platform that makes possible to extract information and derive knowledge from that data. But we are a small company, and to compensate our developer partners usually go through a process that requires clearing a number of internal barriers:

- Is there already budget in place for doing this work, but using an alternative method?
- How severe is the lock-in that occurs with Nexar immediately, as a trade-off to the value the platform brings?
- How quickly can we leverage this?

The answers to all these questions are complex and require our sales organisation to work hard. But we also need to put in place a system to create positive network externalities that make it easier to say “yes” to starting to use Nexar. That involves pushing for standards (IETF) and open. Open does not mean developing the platform as open source, but providing an open platform, having a generous free tier to use, and offering a good premium trial program.

*In Summary*

Teenage comes to every child, with its pains and gains. Nexar is about to go through this: graduating from a company selling a dash cam app to a network and platform that extracts knowledge from the world. It won’t be easy. But these principles are how a formidable asset gets built. Platform adoption increases supply and demand, and network growth generates even more platform value. Densities get high enough to bring the interval between revisits to a road segment from days, to hours, to minutes, to seconds — the air traffic control for the road, the virtual sensor for cars and cities, that keeps us all safe.
