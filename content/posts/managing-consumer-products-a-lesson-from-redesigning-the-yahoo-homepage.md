---
title: "Managing Consumer Products: A Lesson from Redesigning the Yahoo Homepage"
date: 2018-12-13T07:25:02+0000
categories:
- autonomy
author: Bruno Fernandez-Ruiz
---

## Managing Consumer Products: A Lesson from Redesigning the Yahoo Homepage

Back in the early days, all computer users were software engineers. In fact, in the very early days, the people writing the software for computers were the ones using the computers. As an engineer you knew how the system worked, since you wrote it, and how to extend it whenever you needed to support a new feature. Explaining how the system worked to other engineers, or understanding what new features the system had to support, was also easy. A couple of decades later, the arrival of [UNIX]() made computers more mainstream, and much more powerful, and [MAN]() pages arrived as the way to communicate between the engineers writing the software and the engineers using the software. Eventually capturing new requirements also became more complex, and the architect role appeared. But, so far, everybody working with computers was a software engineer, and things were relatively easy.

At some point though, real users, not engineers writing the software themselves, started to use computers. MAN pages became insufficient, as many frustrated users were simply being redirected to [RTFM](). Architects had more and more trouble understanding the requirements, since it was no longer about *what *was required, but *why *it was being requested as a feature. At this point the product manager role appeared, as somebody who could interpret and communicate with real users, and prioritize the *what* and the *why*, leaving the *how* to engineers. The product manager was born.

With time, product management became an essential part of product development. In all areas of packaged software, enterprise services, internal software development, etc. the role of the product manager is exactly as initially anticipated: translate what users want into something that can be built (or not).

All areas but one. Product management has evolved the furthest away from the original role in one particular area: consumer applications. The generalized believe is that a combination of art, magic, wizardry, and luck is required to fulfil the role of a product manager for consumer applications. Nothing is furthest from the truth. Although the product manager working in consumer applications is a very different beast from the product manager working on "classic” products, the practice, processes and required tools to become a product manager for consumer applications, a successful one, are actually rather standard. Unfortunately, this practice is not widely followed in our industry, and most product managers follow a data-driven* *approach to making product *decisions at the start*, rather than using data to make product *decisions at the end*.

In early 2012, during a huge turmoil at Yahoo, I had the chance to start and lead the [full redesign the Yahoo homepage](). The process took a long time, and honestly only completed successfully because Marisa Mayer came onboard as CEO and cut through organizational inertia. I spent 12 months to fully roll out the organic side of the new homepage design, rolling out live in early 2013, and another 12 months rolling out its twin advertising system ([Yahoo Gemini]()). During this process I was lucky to learn how to build and grow a consumer product.


## From 1 to 500

Yahoo was going through a BIG mess at the time. An ousted CEO (Carol Bartz had been fired by the board), a failed PIPE, and a new CEO antagonistic of product development, Scott Thompson, and only looking into finance metrics to meet the street’s desire to release the cash behind Alibaba, Yahoo had become a dysfunctional organization, both at the board level and the management level. I met Scott a few times after he joined. Although he had a charming Boston accent, I lost all confidence in him in a meeting in January 2012, with PK, Filo, Ash Munish and Jay Rossiter, where we were walking Scott through the full technical platform at Yahoo, and the risks we were facing due to extremely high levels of attrition. Scott showed zero interest in technology, or the great engineers that were leaving the company, and he argued that the aerospace industry has been building systems since the 1960s, and that engineers were a commodity that one can always find a replacement for. Needless to say, I was totally devastated. The company I loved (there is such a thing as bleeding purple) was being lead into a total disaster by a complete moron who thought all engineers were fungible.

Right after this meeting, Blake brought me into “Alpha”, a secret project to reduce Yahoo’s workforce by thousands, driven by the CFO to meet the street results guidance. During alpha, it became more and more obvious that we could not keep Yahoo’s “jewels” (personalization and ad targeting systems) running given the CFO’s street guidance we were operating under. Even if we halved headcount overall for engineering, it would not create enough room to keep things running.

Yahoo had been running till then with an organizational structure inherited from the days of the Overture acquisition. There was SATG, originally led by Qi Lu, and the rest. These were run almost like totally separate companies, with different systems and engineering teams. A few of us, lead by Raymie Stata, believed there was a potential opportunity to leverage the complex and state-of-the-art ad targeting systems that Yahoo was running to also power the publisher side of the business. When we were faced with Alpha, we realized that unless somebody would step in as a leader to carry unification through, things would never happen. With the CFO pressure, this became the only answer: merge the content and advertising part of the business into a single system, serving both publisher and display advertising needs.

But this was not enough. Even merging systems and deduping teams, cutting headcount by more than half, would not allow meeting the CFO guidance. Then Mark Morrissey called me a *pig*. Which was the beginning of an adventure that would last till 2015, when Eran and I started Nexar.

During an alpha planning meeting, we were trying to make numbers meet by exploring moving development to China. This was a risky possibility, but not totally unlikely. The team in Beijing had been working on keeping alive Yahoo’s sponsored search advertising systems during the transition of search to Microsoft’s Bing. They knew a lot of the pieces required to transition the knowledge from Sunnyvale. But we needed somebody with the knowledge to make it happen. And here comes eggs and bacon for breakfast, where the chicken is involved, but only the pig is fully committed. Mark nominated me as a pig to go and lead the transition from Beijing. Blake Irving supported the move. Ash, Jay, Filo supported it. Even Scott supported. And most critically important, my wife supported the decision.

So in the period of one month, we packed, took the kids from school, and moved to Beijing. By the end of May 2012, over 500 people were reporting to me worldwide, across engineering, product management, product marketing and part of sales. I had gone from a team of one, to a team of 500. I was scared to death. Not only that, but the systems I had become responsible for were the underpinnings of everything running at Yahoo that was generating views and money. The publisher stack. YUI and the Yahoo developer network. Our mobile teams. The advertising targeting systems. The creative advertising platform (under Eran at the time). And the content personalization systems.


## My Journey to Product Management

Having to deal with 18 different office locations was painful. But more painful was having to deal with declining businesses worldwide. One of the systems I was responsible for was [Yahoo CORE](), powering the Yahoo homepage. The Yahoo homepage was declining fast, both in North America in Europe. I could either invest further in Yahoo CORE, or cut the cord and jump into unifying the content and advertising systems to be able to provide a totally new homepage experience not only showing Yahoo’s content, but the whole web.


![Image](https://cdn-images-1.medium.com/max/800/1*oU6Zr4dTPTRPEPwVhuh9_A.jpeg)

At this point the Yahoo homepage was mainly a firehouse into the rest of the Yahoo network. Pretty much every Yahoo web property depended on the traffic coming from the Yahoo homepage. When you have over 3 billion page views per day in the homepage, even a tiny percentage of traffic is important. Properties like Yahoo News for example fully depended on the homepage. Only a few properties, like Yahoo Finance, had enough organic traffic. I had to make a decision: keep declining, or rock the boat and lose revenue but potentially survive.


## Initial Product Decisions

The Yahoo homepage was running back then an algorithmically optimized editorial module. What this meant is that Yahoo CORE was exploring and exploiting the best performing stories for each audience group, from the pool of available stories on Yahoo properties and a few selected content partners. The available pool was small compared to what people were reading on the web. The consequence was that the profile we were building was heavily biased to what was available on Yahoo, not what users really wanted. Therefore, we had two fundamental issues to solve: creating full interest profiles, and scaling our content pool by 1000 times.

I could either continue working to improve this system, or cut the cord, and merge the advertising and content systems. Despite repeated pressure from the SVPs/Head of NA, Europe and Asia, I decided to stop supporting Yahoo CORE and invest all resources into developing the next generation content system leveraging the advertising stack and engineers I now had in Beijing. The SVPs, unhappy with my decision and seeing the continued revenue decline, escalated repeatedly to the CEOs at the time, first to Scott Thompston, and later to Ross Levinsohn who had become interim CEO after Scott got fired (ok, ok, he “resigned”). The regional sales folks were screaming they were losing traffic because of Yahoo CORE. I was decided to ignore them, as I knew that by Yahoo CORE being a hybrid editorial-algorithmic system, the human impact was always more important, especially with the high attrition Yahoo was seeing in the editorial teams. I found data to prove my case. And in any case, I did not quite care what Scott thought, since I was the pig (and that made it *much easier*).

Putting Yahoo CORE on minimum maintenance was a huge risk, and I knew I had to move fast to get the replacement up and running. I diverted engineers from the ad targeting systems. We started building the profiles. Then indexing all the fresh content we could get our hands on, which was being shared on the web in any way. We went from a content pool of about 1,000 editorial ranked articles on CORE, to 76,000 articles on Yahoo, to over 800,000 daily articles on the web. In 3 months, by the end of August, we had a system up and running, leveraging all the user’s interest, not just the ones from CORE. The engineering teams in Sunnyvale and Beijing pulled their weight together, worked extremely hard, and proved Scott wrong: engineers are not fungible, they are your first and only capital asset in a high-tech company.


## Knowing Upfront What is Good or Bad is Impossible

We had a tiny bucket on the frontpage which we were using for testing the carrousel. Our click-through-rates were similar to Yahoo CORE. Our repeat visits were better, but not all that good. Unfortunately our clicks were to articles off the network, which meant we were “leaking” revenue, and this was a no-go. Even with all the “power” I had accumulated as a *pig*, this was something I had no influence over. I was stuck.

At this point, I could have given up. My hopes for much better engagement were not showing up. The revenue was severely hit. Had I made a bad product decision? I was hoping for much better CTRs and repeat visits, which would proof the value in sacrificing revenue. But this was not happening.

Deep down, I was still convinced though that my initial qualitative decision was the correct one. We were lacking content 1000x fold. And our profiles were also 1/1000th of what they should be. That was true by any analytical standard. But in the real world, despite my intellectual constructs, results were not showing.

I did not know what to do. Eventually, months later, things worked, and with ten-fold returns. Should I have decided at this point to cancel the project, Yahoo may have never had a new homepage, never created Yahoo Gemini, and never seen the $1.2 billion ARR new business associated with it.

But I did not give up. Marissa Mayer, only 30 days in as Yahoo's new CEO, stepped in, and she told me to double down. We were bringing the search-like approach I had been working on to the home page. That was the right thing to do, even with results being bad at the time.

This taught me an important lesson: even if a product looks like it's failing, it might actually have product-market fit. You just can’t tell yet. Unfortunately, you can't test it upfront or immediately after launching. This difficulty in measuring market fit is what many product managers fail to understand. No data-driven approach will give you answers at this stage, simply because the data and the user behavior you are looking for is not there (yet).

Instead, during the first launch of a product, our criteria should be meeting the original rationale and reasons we had to create the product, not how the product behaves right now in the real world.


## It Takes Time for a Product to Grow

Over the next few months, Mike Kerns led the design of the new frontpage in partnership with my team. We made changes to the user experience, but just a few. Most of the effort was slowly adding more and more users into the new system. By the end of November, we were seeing user sessions becoming longer and more sticky. CTRs were also starting rocket. With little changes, things were starting to work.

Two things were happening. Marissa was patient, and she knew very well why. First, user behavior is slow to change. Initially, our user research was extremely critical of the change we were doing. People wanted their old homepage back, and were upset that we had touched it.

As a product manager you should most definitely conduct and read the user research, but in a consumer application you must not make decisions based on user research, or *thou shall be doomed*.

Over time, we saw our users getting used to the new user experience, the new concept with a bigger content pool including non-Yahoo content, … all adapted their taste and interests. They started to like it, and they told us so!

The second thing that happens is that algorithmic trends are slow to happen. It took time for the system to grow and learn which were the best content sources on the web. Which meant that our user profiles also took time to improve. Over three months, we were seeing CTRs increase, with little other reason than time passing by.

I learnt that real results heavily trail development. This meant we had to keep moving without seeing results, guided by instinct. It also meant that we had to measure absolutely everything we could imagine, since we could not back in history and look at the data that we had not been measuring. We built for-purpose infrastructure to do just this: fully understand the user engagement journey.


## The Thin Line Separating Unfit from Fit

But even then, even with much better CTRs, less bounce rates, and a huge increase in repeated visits, we were losing revenue.

Marissa asked for tens of test buckets, each containing a tiny iteration of the frontpage. With labels, without labels. Bigger pictures, smaller pictures. Round corners, squared corners. Purple, yellow. With source attribution, without it. Larger content pool, confined pool.

We kept iterating on these many, many, small changes over the months. Discovering thing after thing that made a 1% difference at the time.

Each week we would review with Marissa the results for all buckets, kill some, keep some, add a few new ones. We were iterating the product really fast, but with very small changes.

We were getting better, week after week. Marissa had set some goals for engagement and revenue which we finally met on February 2013. And then 100% of the Yahoo homepage traffic switched to the new design.


![Image](https://cdn-images-1.medium.com/max/800/1*9ZnTewlAYd8V5D2fyshyVA.png)

A few months later, on May 2013, the twin system for showing content ads on the new homepage also came live. Yahoo Gemini was born (and FWIW on the naming, I had twins, Eran had twins, Marissa had twins).

Yahoo had gone from an editorially curated homepage assisted by algorithms, to an algorithmic homepage and search-like organic content stream with embedded (native) search-like sponsored ads. We had switched the Yahoo publisher and Yahoo display advertising models. Just like we had planned 18 months earlier … when Mark called me a pig.


![Image](https://cdn-images-1.medium.com/max/800/1*-J49xbnDmlxljMzFN5Qo7A.png)


## Knowing When to Cut Your Losses

It’s all good when things go well, and get continuous improvements through small iterations. We should have never given up when we first rolled out the first test with users and the results were bad. But if after doing a few iterations things don’t improve, when should we stop? We need to figure out if things are not improving because the original hypothesis was wrong, or things are not improving because the small incremental changes are not the right ones.

How can we tell? The answer is probably [lies, damned lies, and statistics](). Trying to use statistics alone to prove that the hypothesis was right or wrong will leave us with big doubts. As much as a whole, the product might be failing, there is normally always a user cohort for which the product works, works well, or even works much better than before. So even though statistics tell you otherwise, success of the hypothesis can be seen upfront by identifying any consumer segments for which the change drives an immediate and radical response. This might be a very low number of users, but if the behavioral change is big, you have a leading indicator. If you can’t find any trace of change amongst your customers, it is unlikely there will be a product market fit for your audience despite any further iterative development.


## Applying the Lessons to Nexar

The principles learnt whilst building the redesigned Yahoo homepage, which in fact came from a combination of Yahoo’s and Marissa’s experience building and growing web search, apply equally to Nexar.

- Upfront product decisions cannot be solved quantitatively. Decisions need to be made on the fundamental underlying qualitative issues our users experience while interacting with our app: the things that create friction and prevent the habit creation. And once you have a decision, stick to it. Unless the fundamental qualitative reasons that originally justified the decision change, don't change course half-way through, even if data tells you so (“*data lies”*).
- Whatever our delivery output is, it takes time and focus for consumers to create and develop a habit. Which is exactly why it’s a habit. There are no quick wins, but hard work. Continuing to move, define, measure and observe long trends in user engagement and behavior are critical. We need to trust and own our data, as product decisions heavily trail behind data acquisition.
- At the end of the day there is no silver bullet, but rather success comes from tons of very small changes executed in a very fast iterative manner. Owning a scalable bucket testing infrastructure, which allows the end-to-end testing in parallel of 10s of these small changes, is critical to get those very much needed 1% wins.

## Nexar’s Associate Product Manager (APM)

There is a method to learn how to become a great consumer product manager, but it takes time, practice and it requires exposure to the multiple facets of designing and rolling-out a product to millions of consumers. And like any trade, an apprenticeship is the optimal way to master the craftsmanship of product development. In order to provide promising candidates wanting to move into a product or design role the exposure to these facets within in a short period of time, we recently launched [Nexar’s Associate Product Manager (APM) and Associate Product Designer (APD) Program](). A 12 month program, with 4 month rotations into each of the facets of product development, will allow the selected apprentices to be in a very dynamic environment where they will get the chance to learn what it means to be a product manager of consumer applications.
