---
title: Stall by Incremental Releases
date: 2007-08-26T09:51:57+0100
categories:
- Architecture
author: Bruno Fernandez-Ruiz
aliases:
- /architecture/stall-by-incremental-releases/
---

Have you have ever been part of the inception of a software system
that later became big and complex? Have you later felt the frustration
of not being able to make further changes to the core architecture?
Did you end up being taken hostage by the software? I did.

<p>We hope as developers to be able to adapt software as requirements and bugs arise, and to be able to organize our software releases accordingly. By doing incremental releases we hope to work with a stable code-base where we can release often. We then receive feedback more often and we are therefore agile and responsive to business needs.</p>
<p>Until one day we become legacy. A competitor starting from scratch has been able to innovate and quickly implement these new cool features the market really wants. We can't catch up with them, because they don't have the legacy we have: the other thirty thousand features we need for all revenue-generating customers we have.</p>
<p>We set ourselves to large regeneration plans, greenfield development. "This time around, we'll do it right", we say. Two to three years later, after again a cycle of incremental releases, we become again legacy.</p>
<p>And this is how the software industry is run. Competition between those entering the legacy stage and those on the startup curve is what keeps us employed. And yet this continuous hostage situation by incremental releases is what annoys me most about software.</p>
<p>There are two realistic approaches to solving the problem. You could plan upfront for a software life of 2 to 3 years and not 5 to 10 as some would hope. Or you could take the red pill.</p>
<p>I would rather see more people taking the red pill. It ain't easy, I admit it, but it is possible. Architectural refactoring is extremely painful - you are basically breaking the system as hard as it can be broken before pulling it up together again -. I have done it, but don't take my word for it, and look at Linux going from 2.4 to 2.6, at Windows going from 98 to 2000, at Mac OS going to X. They took the pain, and it paid off.</p>
<p>Let's stop building software and let's start thinking of software reconstruction. After a few incremental releases, do an architectural release. I know you need it. You know you need it. It's your choice, the red pill.</p>
