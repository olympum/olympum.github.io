---
title: "Credentials Are the New Syscalls"
date: 2026-07-10
draft: true
categories:
- Architecture
author: Bruno Fernandez-Ruiz
---

The dangerous thing an agent does is rarely the command it runs on its own machine. It is the credential it holds, and the remote authority that credential unlocks. We argue endlessly about sandboxing the filesystem while the agent sits on an AWS key that can empty a production bucket from anywhere on earth. The local blast radius is a container. The remote blast radius is your business.

Operating systems met this exact class of problem fifty years ago. The first computers I programmed had no boundary at all: any program could scribble on any address, and one bad pointer took down the machine. Then came protected mode, rings, and the syscall. A userland process that wants to write to disk does not get the disk controller. It requests an effect through a narrow, mediated interface; the kernel performs the privileged part and hands back a result. The process never holds the mechanism. That one boundary is what made it safe to run code you did not entirely trust next to code that mattered, and everything from time-sharing to the cloud is downstream of it.

For an autonomous agent, the privilege boundary is not the syscall. It is the credential. What an agent can do to your world is defined by the keys in its environment, not the binaries on its disk. An agent with no credentials is a very fast intern with opinions. An agent with your deploy key is a production incident waiting for the right sentence.

The industry default hands the key to the workload. A secret manager authenticates the process and gives it the secret, and the process uses it directly. Which is correct for a service you wrote, reviewed, and deployed. That service executes the code you gave it and nothing else; giving it the key is giving the key to yourself.

An agent is a different kind of worker. It reads tickets, web pages, pull requests, and README files, and anyone who can place a sentence in front of it can influence what it does next. That is the job description, not a defect: we hired it precisely because it acts on what it reads. So once a key is in the agent's context, anyone who can influence the input can walk that key out through any channel the agent can reach: a commit message, a DNS lookup, an innocent-looking POST. You cannot fix this with instructions, because the model that follows your system prompt is the same model that follows the attacker's ticket. The exposure is structural. Handing an agent a long-lived secret is running userland in ring 0 and asking it nicely to behave.

The fix is the same inversion the kernel made: never materialise the secret into the worker. A broker on the host side, outside the agent's reach, holds the credential and mediates every privileged action. Two mechanisms cover nearly everything. Inject at egress: a host proxy terminates the agent's outbound call at the boundary and attaches the real credential on the way out, so the secret never enters the agent's environment. Or mint on demand: for tools that insist on holding a credential, the broker issues a short-lived, narrowly scoped token for that single call, so only an expiring derivative ever exists inside the boundary. Either way the agent asks for an action and receives a result. Capability without disclosure.

The difference in practice fits in a sketch.

```sh
# Today: the mechanism lives inside the agent
AWS_SECRET_ACCESS_KEY=wJalrXUt... agent run "publish the release"

# Inverted: the agent requests the effect from a local broker
curl --unix-socket /run/broker.sock \
  -d '{"action":"s3.put","bucket":"releases","file":"v1.4.2.tgz"}'
# => {"ok":true,"etag":"9b2cf5..."}
```

The obvious objection is that this merely moves the problem: now the broker holds the keys, and the broker can be attacked. True, and it is the same objection people raised against kernels. The answer is also the same. The broker is small, deterministic, and auditable; the agent is enormous, probabilistic, and talks to strangers all day. Concentrating privilege in the small verifiable thing while starving the large unverifiable thing is the whole trick. You still have to do the boring work: deny the verbs whose only purpose is to read a credential back out, cap the output, time-box every grant, fail closed when the broker is absent. But those are engineering problems with known shapes. "The model might obey a stranger" is not.

What you buy with the inversion is the one property that matters: compromising the agent yields no credentials, because none were ever in its memory. Prompt-inject it, jailbreak it, dump its entire context to a paste site. There is nothing there to take. The agent had authority the way a userland process has a disk: by asking.

We spent decades teaching processes to request effects instead of holding mechanisms, and that discipline is what made it safe to run untrusted code at scale. An agent is untrusted code that reads its own instructions off the open internet. So treat the credential the way the kernel treats the hardware: the agent may request the effect. It never holds the mechanism.
