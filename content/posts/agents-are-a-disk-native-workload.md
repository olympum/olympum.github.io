---
title: "Agents Are a Disk-Native Workload"
date: 2026-07-10
draft: true
categories:
- Architecture
author: Bruno Fernandez-Ruiz
---

In the autumn of 2011 I wrote a short post called [The Praxis of Event Loops](/http/the-praxis-of-event-loops/). It was an argument against a theoretical position, fashionable at the time, which held that at full utilisation threads and events have the same theoretical throughput. Which is true. It is also beside the point, because the interesting workloads never run at full utilisation. Yahoo!'s edge was serving over twenty billion requests a day on a few dozen threads per host, because every one of those hundreds of thousands of connections was mostly waiting: on a slow client, on a remote origin, on a scatter-gather fan-out across a thousand index servers. When a connection is mostly waiting, you stop giving it a thread. You give it a file descriptor, a callback, and a place in the loop, and the cost of the next connection falls to almost nothing. That was the praxis.

This post is the direct sequel, fifteen years late, because it took fifteen years for the next workload with exactly that shape to arrive. The workload is the agent.

## Two idle clocks

An agent session has two clocks inside it, and both are idle clocks. Take a coding agent doing real work. A turn is one user prompt carried end to end: the agent loops through model calls and tool calls until it comes back with an answer. A session is the string of turns that makes up one piece of work, and between the turns sits the human gap, the minutes in which a person reads the answer, thinks, and decides what to ask next. That outer clock is idle by construction; no engineering can fill it, because a human is inside the loop.

But the inner clock is idle too. Within a turn, the agent spends most of its wall time blocked on a remote model or on tool I/O. The compute it does between those waits (parsing output, editing a file, running a build) is a small fraction of the elapsed time. A turn runs for seconds to minutes; the gap after it runs for minutes to hours; and through nearly all of both, the agent barely touches the CPU it holds.

This is the profile of an I/O-bound connection, scaled up three orders of magnitude in duration and wearing a virtual machine.

## The naive design, and the trap next to it

The naive design gives each session a microVM and holds it for the life of the session. On the whiteboard this is defensible, and the 2011 corollary comes back almost word for word: at full utilisation, a held VM and a leased VM have the same theoretical cost. But an agent fleet never runs at full utilisation. Hold the VM and you pay for its gigabytes of RAM through every idle gap, which is to say through most of the wall-clock life of the session. The fleet fills up with memory that nobody is computing against.

The obvious remedy is the one everybody reaches for first: snapshot the idle VM's RAM out to disk, reclaim the slot, and restore the snapshot when the user comes back. We reached for it too, building agent infrastructure at fleet scale. It cost more than it saved. The lesson arrived quickly, and in an unexpected column of the dashboard: the snapshot writes, not the agents, were saturating the disk. Every gap in every session turned into gigabytes of sequential writes, and that traffic became by far the dominant source of host I/O, dwarfing what the agents themselves generated doing actual work. Worse, the admission controller watched I/O pressure as one of its signals, so hosts with plenty of free CPU and free memory started refusing new work. The remedy for idle RAM had manufactured a disk bottleneck. We were paying a heavy I/O bill to preserve state that, it turned out, did not need preserving.

## The observation the design falls out of

A coding agent's real memory is its workspace on disk, not anything in RAM. The repository, the edited files, the build artefacts, the dependency caches: all of it already sits on the volume, written there in the ordinary course of the turn. What RAM holds at the end of a turn is a process image (a shell, a language runtime, some page cache) that can be reconstructed from the disk in less time than it takes a person to read the first line of the answer. Snapshotting that RAM is copying out a cache and calling it state. The state was on disk all along.

So the correct move is to lease close to the turn and let the RAM go between turns. Within a turn the VM stays live across the model and tool waits, which are only seconds apart; tearing it down inside a turn would buy little and cost coherence. When the turn ends and the long human gap opens, we discard the RAM entirely, which costs no I/O at all, leave the disk volume exactly where it is, and return the slot to the pool. When the next prompt arrives, we cold-boot a fresh microVM and reattach the volume. The boot is cheap because the kernel and the root filesystem are shared across every microVM on the host and stay hot in cache; the only cold thing is the workspace, and the agent was going to read that anyway.

The honest cost is the wake. A cold boot takes a few hundred milliseconds where a warm restore takes a few tens. For a latency-critical request path that difference is fatal. For an agent turn that begins after a human has spent three minutes reading a diff, it is noise. We concede the slower wake and take back gigabytes of idle RAM per session, and the trade is not close.

The three strategies, side by side:

| Strategy | RAM through the gap | Wake latency | Host I/O per gap |
|---|---|---|---|
| Hold the VM | full footprint, every gap | none | none |
| Snapshot RAM to disk | zero, after paying to write it out | tens of ms to page back | gigabytes written, then read back |
| Evict to disk, cold-boot | zero | a few hundred ms | none: the state was already there |

Or, as pseudo-configuration:

```yaml
lease:
  boundary: turn          # not session
  within_turn: stay_live  # model/tool waits are seconds apart
  on_turn_end:
    ram: discard          # zero I/O; it was only a cache
    volume: keep
  on_next_prompt:
    boot: cold            # shared kernel + rootfs, hot in cache
    volume: reattach
```

The middle row of the table is the trap. It looks like the sensible compromise, and it is the worst of the three, because it converts an idle-memory problem into an active-I/O problem on the one resource the fleet actually contends for.

## The event loop, one level up

Reclaiming RAM across the gaps is only half of the density. The other half is the 2011 argument transplanted whole. Because the agent is I/O-bound even while its turn is running, blocked on the model or a tool rather than on the CPU, a host can multiplex many active turns at once, exactly the way an asynchronous server carries tens of thousands of connections that are each mostly waiting. The mapping is one to one. The microVM is the connection. The turn is the request. The disk volume is the durable state, playing the part the origin server played behind the edge. The scheduler that admits and places turns is the loop.

And the two oversubscriptions compound. Memory is reclaimed across the human gaps between turns, while the CPU is shared across the turns that are active but blocked, and because both hold at the same time a host carries far more agents than counting whole VMs against whole cores would ever suggest. Adding N agents to a host costs something closer to K than to N. In 2011 the trick was that ten thousand connections cost almost nothing until they actually did something. The trick has not changed. Only the unit has.

## The Lambda inversion

The instructive comparison is AWS Lambda, which runs on the same Firecracker substrate and lands, correctly, on the opposite default. Lambda keeps recently used environments warm in memory and freezes them between requests, so a follow-up invocation thaws in milliseconds, while the rarely used tail cold-starts from nothing. That is the right call for Lambda's workload, because a function inverts every property of an agent session. Its state is small and lives in RAM; an agent's state runs to gigabytes and lives on disk. Its invocations last milliseconds and are latency-critical; an agent's turns last seconds and follow minutes of human thought. Its gaps are short; an agent's gaps are the dominant term of the whole session. Hold those three inversions in mind and both defaults become obvious. Lambda pays a little warm memory to win a large latency prize. An agent fabric pays a small latency cost to win a large memory prize.

Lambda has even shown that snapshots can be engineered well, snapshotting an initialised environment and paging it back lazily to soften its cold starts. But that is an answer to a cold-start question. A disk-native workload never has to ask it.

The substrate decided none of this. Firecracker is the same hypervisor under both fabrics, and Kata will package a rootfs for either without an opinion. The workload decided.

## Praxis, again

I closed the 2011 post by saying that event loops have a field of practical applicability, and that a seasoned practitioner should know when, and when not, to use them. The same discipline applies one level up. If your workload's state lives in RAM and its gaps are milliseconds, keep it warm; Lambda is the proof. If your workload's state lives on disk and its gaps are human, let the RAM go and boot from the disk; the agent is the proof. At full utilisation the strategies have the same theoretical cost, and no fleet you will ever operate runs at full utilisation.

Fifteen years ago the connection was mostly waiting, so we stopped giving it a thread. Today the agent is mostly waiting, so we stop giving it a VM between turns. The praxis holds.
