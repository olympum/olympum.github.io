---
title: "The Formidable Power of Code"
date: 2023-01-19
categories:
- essays
author: Bruno Fernandez-Ruiz
---

After writing last night the piece on engineering culture, Why Nothing Works: Kings and Priests, I also wrote about it in tangentially in Heroes and Generals, but except a few at Nexar, nobody has had a chance to interact with how I think of code, how I write code, and why I think code is *the* one thing that allows measuring whether an organisation is working, or not. My apparent failure in the first few months in Sunnyvale, how did I succeed? The answer is code.

I love programming languages (I also love actual languages, but that's a different "disease"). Every language teaches you new concepts, and gives you new tools to think about problems, sometimes even going back to previous languages you knew. Language designers have a strong influence in the programming constructs that engineers will use for years to come. Not just the syntax, but the actual design and architecture. And this is to me the key of why code is the only thing that matters at the end of the day. You can't have a restaurant without good food, and you can't have an engineering organisation without good code. You may have processes and people, but without exceptional food, you won't thrive.

So, what is it about code? Let me break down the most important traits to me.

## Code can bridge impedance mismatch

As organisations get larger and the number of people and systems grow, it's easier to "lose control" (Heroes or Generals?). Architecture discussions then appear and are done detached from code, as it would be otherwise too complex. Trade-offs are discussed in a wide context. Then the team moves into design and coding, and usually coding keeps changing the design implicitly. For example, a call from system to system may done synchronously or asynchronously. But such minor "threading" decision has profound architectural implications for upstream and downstream systems. What ends up happening in large organisations is that the architecture that gets reviewed at the ARB is not the same that is in the code. And when months later some architect or leader asks for a change, based on what was discussed, they realise that's no longer possible, as that's not *actually* how the system works. The code holds the only truth, and the only way to reduce the impedance mismatch between architecture and design, and between design and code, is to have "good" code (what's "good" will slowly reveal itself over the next paragraphs). Code allows navigating up and wide, but also down and deep. Nothing else in engineering allows this.

## Code defines organisational barriers

Code has surprising effects, related to Conway's law, but deeper than Conway’s, in regard to forming and destroying boundaries. The team maintaining a code base becomes tightly knit together. Other teams start to "fear" touching that code, and at some point "reject" touching it. It becomes religious: my code, your code. Code is powerful as a tool for developing cohesiveness. Do you want a new team to succeed? Let them write new code that the fully own. Do you want to merge two teams? Given them a shared code base.

## Code is there to be rewritten

I have never understood why so many engineers do not want to touch code, just in case it breaks, even if it’s their own. If that ever becomes the case, you are not proud. And if you are not proud, you should be. There really is nothing magical in code. The risk of changing things has nothing to do with whether it's new code or old code. It's just good or bad code. You still get a bug every 1000 lines. Writing code so that it can be rewritten should be one of the most important activities during development. Rewriting code is not bad, it's good! Imagine that you move into a house without internal walls and you start slowly creating new rooms and laying out walls that are fixed. And then things start to change. Perhaps you have a child, or you setup a home office. What would you prefer, having to break down and rebuild the walls, or having walls that you could push and move as you need them to shape the rooms differently? Except if you are a DIY fan, I think we all probably agree on which one we prefer. Minimising effort of rewrites is only possible by writing code designed to be rewritten. Which is related to …

## Code behaves better when following Bruno's mantra

You hear me talk about my decision-making principles, which I'll repeat here since it's key to understanding how I think about code design:

1. make decisions with the least number of assumptions (Occam’s razor),
2. make the decisions that are easiest to undo (corollary to Occam’s razor), and
3. don't make any decisions that you don't have to (also a corollary to Occam’s razor).

These rules not only apply to how one makes strategic decisions about a company, they apply to how you make architectural decisions, how you write code, and hell, even to how you buy a house.

## Code is an element of pride

I hope this was clear in the piece Why Nothing Works: Kings and Priests. But just in case, it's worth repeating. What makes a strong engineering culture is not that they have wonderfully dressed dailies, or that the JIRA board is groomed every week. That's all fine, but it's not what will make you proud. Sure thing the "external product" makes us proud, but that's as employees. As engineers what makes us proud are the systems we create and the code we write. A large reason why open source exists is not some socialist opinionated Richard Stallman philosophy of knowledge, but rather vanity. We like thinking our code is great and better than others. And there is nothing else than doing it in the open to be exposed to good, and bad. Inside the company things are no different. Your peers will read your code and your PRs. So you better be good, and you better be proud.

## Code is the best way of learning about systems

You can spend days reading Confluence pages and navigating designs, watching videos, reading Slack and browsing StackOverflow. But nothing, nothing, will get you closer to reality than code. I am part of a generation that learnt computer science, and coding, without the Internet. All we had back then were books *and* code. You would read both if you wanted to learn. If you see me code or read code, you may have noticed that I don't search online. I always make sure to have all my *MAN* pages (or the equivalent in whichever language) at my fingertips, usually in emacs just on command away, and in most cases also setup for navigating symbols in the source code for the programming language runtime, if there is such (e.g. Java, Go, Python).

## Code needs love to survive

This one is not understood by the higher ups of corporation organisations, but it's one of the most usual reason for technical debt, employee churn, and decline in productivity and velocity. As people leave the company, code becomes orphaned and abandoned. It's important for others to take it on, not as a burden, but as an honour. When Rick Reed left Yahoo to join Brian Acton and Jan Koum at WhatsApp, there were fights amongst the engineers for who was going to continue maintaining Rick's codebase (he had written a TCP stack from scratch in 1997 that became the foundation for Yahoo's BSD servers, yield a 1% optimisation in network and capital costs, and when you have a $1bn data centre bill per year, that's a lot of money). As a manager, assign the code to somebody else and give them the freedom to take it on as their own, and not as a keeper, but as a parent of the code. Let them rewrite what they want till it becomes theirs.

## What makes code *good code*?

And finally I get to what it is that I consider "good" code. And somebody much better than me wrote something many years ago about design principles, and it's not worth plagiarising (if it ever is a reason to do so). So go now and do yourself a favour and read probably the most important positional document written in the history of the Internet: [Axioms of Web Architecture](https://www.w3.org/DesignIssues/Principles.html).
