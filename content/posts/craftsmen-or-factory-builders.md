---
title: "Craftsmen or Factory Builders?"
date: 2026-11-17
draft: true
categories:
- essays
author: Bruno Fernandez-Ruiz
---

A few weeks ago someone asked me what makes the move to agents so hard for some of our teams. I had a month of data and pages of analysis, yet the answer that came out fit in one line: it is the step from "Claude writes code I review" to "I write the workflows that Claude runs."

I have been thinking about that line ever since, because it reaches every one of us, in every function, and not only in engineering.

## The curve was always there

A month after we reorganised engineering into small crews, I looked at who had written what. In that month, two of the engineers on the crew roster, 3% of them, wrote half of all the pull requests, and the first 15% wrote four fifths. The median builder went from 0.44 pull requests a day over the previous twelve months to 0.50, which is to say that the typical engineer works much as they did a year ago, while the output of the team has multiplied.

I checked it against the same month a year earlier, for the same engineers. The total had grown almost sevenfold, while the median person only went from 10 pull requests a month to 17. The two people at the top of this year's curve had multiplied their own output more than sixtyfold.

Talent has always been steeply uneven, and every one of us has managed around it for years because we could not see it. What agents have done is remove the ceiling of one person's attention, so that a curve which was always there is now steep enough to count.

Now, pull requests are volume and not value, and agents inflate them. I would not level anybody on a chart like this. But a gap of a hundred times is not a measurement problem, and the interesting question is what the people at the top are doing that the others are not. The answer is not that they use agents more, or better.

## Craftsmen and Factory Builders

I spent years at Yahoo in what I once described as a school of craftsmanship. Engineers showed off their prowess in libraries, code and releases, juniors looked up to seniors to learn the trade, and the only way to make progress in that culture was to be admired through code. I loved it, and I still bleed purple.

An agent in the hands of a craftsman is a better tool. The piece gets written faster, reviewed and admired, and the gain ends where one person's attention ends. I call this person the Craftsman, and most engineers today are Craftsmen, some of them exceptional.

The people at the top of the curve have stopped making pieces. They build the machine that makes the pieces, and then other people and other agents run it all night. I call them Factory Builders, and the test for one is simple: do other people ship through what you built? A thousand changes that only their author ever reads is a very fast Craftsman, while a harness that another team adopts is a factory.

In 2022 I wrote that we did not want to scale with [Generals, but with Heroes](/posts/heroes-or-generals/). I still believe it, and I now know what the Hero does all day.

The Hero builds the factory.

## Where the curve is steep

A colleague asked whether the curve shows up everywhere, so we ran it on every pull request that our three GitHub organisations received in that month, with bots left out. It does. Across the company, the most active fifth of authors wrote 82% of the pull requests and the bottom half wrote 4%. In each of our four engineering domains the top fifth wrote between 70% and 94%.

Inside the repositories, though, the curve splits in two. Where the work runs through agents, one or two people wrote two thirds or more of everything that landed. In the older repositories, the top fifth wrote about half and the bottom half wrote up to a fifth, which is how work was spread before agents. Measured as a Gini coefficient, the agentic repositories sit around 0.7 to 0.8 and the legacy ones around 0.4.

The history of a single repository tells the same story. Our newest product has been built by people and agents working under one shared doctrine for thirteen months, and in that time it has taken almost as many commits as our platform monorepo gathered in eleven years. Commits are volume too, but eleven years of history in a little over one is what a factory looks like from the outside.

A word on the shape, for the statisticians. The tail is heavy wherever we look, and a power law fits it with an exponent between 1.5 and 2.3. A log-normal fits just as well, and with so few people in each tail the likelihood-ratio test cannot tell the two apart. So I will say that the curve is steep and getting steeper, and I will not yet claim that it is a power law.

## It is not just engineering

Our finance automation team set out to run the monthly close, for the first time, on revenue that comes out of a model in the warehouse rather than a spreadsheet. In their status note there is a small exchange I keep coming back to. Finance asked how the model treats a refund on a prepaid plan, and the engineer answered that there was no rule yet, and that whenever finance defined one they would turn it into the model.

That is the new job. Instead of producing the number, the finance expert defines the policy that produces every number, and the time that comes back goes to what we never had the capacity to do: reconciliation we can trust, and planning that looks forward instead of backward.

Sales has built a prospecting tool, operations a production trace, partner operations a portal that adds cameras unattended, and marketing a brand panel. Every one of these started with a domain expert who got tired of doing the work by hand.

## Making it visible

So the bar is the same for every one of us, in every function and at every level, starting with leadership: build something others ship through, or ship through what somebody else built.

To make it visible, we grade functions rather than people. A leaderboard counts the work that other people do in the apps each function built, what runs on a schedule, and what is wired to the systems the company runs on, and it names each function's next step. For engineering, each person sees their own profile and where they sit in their crew, and nobody else's. We read all of it as a direction rather than a measurement, because the numbers count volume and adoption far better than they count value.

This is a leap, and I know it. We are asking finance experts, marketers and firmware engineers to work the way a handful of people have shown is possible, on the strength of a few months and a few stories. But the people at the top of the curve did not wait for a structure that allowed them to work this way.

Go build your factory.
