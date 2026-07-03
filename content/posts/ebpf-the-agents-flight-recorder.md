---
title: "eBPF as the Agent's Flight Recorder"
date: 2026-07-10
draft: true
categories:
- Linux
author: Bruno Fernandez-Ruiz
---

A log written by the process it describes is testimony, not evidence.
Aviation internalised this decades ago: the flight recorder is not the
pilot's diary. It is a separate instrument, in a separate failure
domain, and it records what the aircraft actually did regardless of
what anyone on board believed at the time. When the thing flying is a
language model driving a process tree on a Linux box, the same
principle applies, and almost nobody applies it.

An agent session is a process tree. The harness spawns shells, the
shells spawn compilers, linters, package managers, and each of those
executes commands, opens files, and makes network connections. I want
every one of those actions attributed to the session that caused it,
and I want the attribution to survive an agent that is confused,
prompt-injected, or outright hostile. So we cannot ask the agent. Its
transcript of tool calls is the pilot's account: useful as narrative,
worthless as evidence, because anyone who can influence the agent's
context can influence what it reports. The recorder must be
un-forgeable by the thing it records, which means it lives below the
agent, not beside it. On Linux, below means the kernel, and the way
into the kernel without writing a kernel module is eBPF.

I have spent most of my career one layer below the application, from
edge proxies at Yahoo pushing tens of billions of requests a day to
vehicle telematics at Nexar, and the lesson repeats: the counters you
can trust come from underneath the code you are measuring.

## Why the kernel and not a wrapper

The tempting alternatives are all user-space: a shell shim around the
agent's commands, an `LD_PRELOAD` interposer, a patched CLI. Every one
of them is advisory. A static binary ignores `LD_PRELOAD`. A shim sees
only what passes through it, and the agent can write a script to disk
and exec it directly, or spawn a child that inherits nothing of the
wrapper. The kernel has no such blind spot, because every process in
the tree must cross the syscall boundary to do anything that matters.
Nothing real happens without `execve`, `openat`, or `connect`. So we
instrument the boundary itself, and the agent's honesty stops being a
dependency.

## Ten seconds of bpftrace

Before writing any C, prove the signal exists. Every command the
session runs, with arguments:

```bash
sudo bpftrace -e '
tracepoint:syscalls:sys_enter_execve {
  printf("%d %s exec: ", pid, comm);
  join(args->argv);
}'
```

Every outbound TCP connection, with destination:

```bash
sudo bpftrace -e '
kprobe:tcp_connect {
  $sk = (struct sock *)arg0;
  printf("%d %s -> %s:%d\n", pid, comm,
         ntop($sk->__sk_common.skc_daddr),
         bswap($sk->__sk_common.skc_dport));
}'
```

Run these while an agent works and the effect is immediate: the
`git status` you expected, and then the `curl` you did not. That is
the argument, in two one-liners. The kernel already sees everything;
we are only adding memory.

## A session is a cgroup

The one-liners record the whole machine. Attribution needs a boundary
around one session, and cgroup v2 is the right primitive because
membership is inherited by every child the agent spawns and cannot be
shed by an unprivileged process. Before launching the agent, give the
session a cgroup:

```bash
sudo mkdir /sys/fs/cgroup/agent-session
echo $$ | sudo tee /sys/fs/cgroup/agent-session/cgroup.procs
exec my-agent --task "refactor the billing module"
```

The shell moves itself into the cgroup and then execs the agent, so
the entire tree under it carries the label. (`systemd-run --scope`
does the same thing with more ceremony.) bpftrace can filter on it
directly:

```bash
sudo bpftrace -e '
tracepoint:syscalls:sys_enter_execve
/cgroup == cgroupid("/sys/fs/cgroup/agent-session")/ {
  join(args->argv);
}'
```

One caveat: the `cgroup` builtin compares the task's exact cgroup id,
so processes in child cgroups will not match. For a flat session scope
that is fine; the proper program below does hierarchy-aware matching.

## The recorder, properly: libbpf and CO-RE

A recorder you deploy needs to compile once and run on any reasonably
modern kernel, which is what CO-RE (compile once, run everywhere)
gives us via BTF. Three files. A shared header:

```c
/* flightrec.h */
#ifndef FLIGHTREC_H
#define FLIGHTREC_H

#define MAX_ARGS 8
#define ARG_LEN  64

struct event {
	unsigned int pid;
	unsigned int ppid;
	char comm[16];
	unsigned char nargs;
	char args[MAX_ARGS][ARG_LEN];
};

#endif
```

The BPF side. Note the `session` map: a cgroup array holding one fd,
which `bpf_current_task_under_cgroup` checks hierarchically, so every
descendant cgroup of the session matches too:

```c
/* flightrec.bpf.c */
#include "vmlinux.h"
#include <bpf/bpf_helpers.h>
#include <bpf/bpf_core_read.h>
#include "flightrec.h"

char LICENSE[] SEC("license") = "GPL";

struct {
	__uint(type, BPF_MAP_TYPE_CGROUP_ARRAY);
	__uint(max_entries, 1);
	__type(key, __u32);
	__type(value, __u32);
} session SEC(".maps");

struct {
	__uint(type, BPF_MAP_TYPE_RINGBUF);
	__uint(max_entries, 256 * 1024);
} events SEC(".maps");

struct {
	__uint(type, BPF_MAP_TYPE_ARRAY);
	__uint(max_entries, 1);
	__type(key, __u32);
	__type(value, __u64);
} drops SEC(".maps");

SEC("tracepoint/syscalls/sys_enter_execve")
int trace_execve(struct trace_event_raw_sys_enter *ctx)
{
	if (bpf_current_task_under_cgroup(&session, 0) != 1)
		return 0;

	struct event *e = bpf_ringbuf_reserve(&events, sizeof(*e), 0);
	if (!e) {
		__u32 k = 0;
		__u64 *d = bpf_map_lookup_elem(&drops, &k);
		if (d)
			__sync_fetch_and_add(d, 1);
		return 0;
	}

	struct task_struct *task = (struct task_struct *)bpf_get_current_task();
	e->pid  = bpf_get_current_pid_tgid() >> 32;
	e->ppid = BPF_CORE_READ(task, real_parent, tgid);
	bpf_get_current_comm(&e->comm, sizeof(e->comm));

	const char **argv = (const char **)ctx->args[1];
	e->nargs = 0;
	for (int i = 0; i < MAX_ARGS; i++) {
		const char *p = NULL;
		bpf_probe_read_user(&p, sizeof(p), &argv[i]);
		if (!p)
			break;
		bpf_probe_read_user_str(e->args[i], ARG_LEN, p);
		e->nargs++;
	}

	bpf_ringbuf_submit(e, 0);
	return 0;
}
```

The licence string is load-bearing; several of these helpers are
GPL-only. And the userspace loader, which pins the session cgroup fd
into the map and drains the ring buffer:

```c
/* flightrec.c */
#include <stdio.h>
#include <signal.h>
#include <fcntl.h>
#include <bpf/libbpf.h>
#include <bpf/bpf.h>
#include "flightrec.h"
#include "flightrec.skel.h"

static volatile sig_atomic_t stop;

static void on_sigint(int sig) { stop = 1; }

static int on_event(void *ctx, void *data, size_t len)
{
	const struct event *e = data;
	printf("%-7u %-7u %-16s", e->pid, e->ppid, e->comm);
	for (int i = 0; i < e->nargs; i++)
		printf(" %s", e->args[i]);
	putchar('\n');
	return 0;
}

int main(int argc, char **argv)
{
	if (argc != 2) {
		fprintf(stderr, "usage: %s <cgroup path>\n", argv[0]);
		return 1;
	}

	struct flightrec_bpf *skel = flightrec_bpf__open_and_load();
	if (!skel)
		return 1;

	int cgfd = open(argv[1], O_RDONLY | O_DIRECTORY);
	if (cgfd < 0) {
		perror("open cgroup");
		return 1;
	}

	__u32 key = 0;
	if (bpf_map_update_elem(bpf_map__fd(skel->maps.session),
				&key, &cgfd, BPF_ANY)) {
		perror("update session map");
		return 1;
	}

	if (flightrec_bpf__attach(skel))
		return 1;

	struct ring_buffer *rb = ring_buffer__new(
		bpf_map__fd(skel->maps.events), on_event, NULL, NULL);

	signal(SIGINT, on_sigint);
	while (!stop)
		ring_buffer__poll(rb, 100 /* ms */);

	ring_buffer__free(rb);
	flightrec_bpf__destroy(skel);
	return 0;
}
```

Build and run:

```bash
bpftool btf dump file /sys/kernel/btf/vmlinux format c > vmlinux.h
clang -g -O2 -target bpf -D__TARGET_ARCH_x86 -c flightrec.bpf.c -o flightrec.bpf.o
bpftool gen skeleton flightrec.bpf.o > flightrec.skel.h
cc -g -O2 flightrec.c -lbpf -lelf -lz -o flightrec
sudo ./flightrec /sys/fs/cgroup/agent-session
```

Every `execve` inside the session, and nothing outside it, attributed
by the kernel with pid, parent, and arguments. Add the same pattern on
`sys_enter_connect` and `sys_enter_openat` and you have the full
recording. The agent was never asked, and could not have lied if it
had been.

## From recording to refusal: BPF LSM

A recorder tells you what happened. The harder question is whether the
kernel can refuse on our behalf, in the same breath, below the agent.
It can, since Linux 5.7: BPF LSM lets a BPF program attach to the
kernel's security hooks and return a verdict, and a verdict is
enforcement, not observation. Prerequisites first, because this is
where deployments stumble:

```bash
grep CONFIG_BPF_LSM /boot/config-$(uname -r)   # must be =y
cat /sys/kernel/security/lsm                   # must contain "bpf"
```

Most distro kernels compile it in; fewer activate it. Fedora ships
`bpf` in the active LSM list, Ubuntu does not, and if it is missing
you add `lsm=landlock,lockdown,yama,integrity,apparmor,bpf` to the
kernel command line and reboot. With that in place, the guard lives in
the same object file as the recorder and consults the same `session`
map, so one map defines both what is recorded and what is governed:

```c
#include <bpf/bpf_tracing.h>

#define AF_UNIX 1
#define AF_INET 2
#define EPERM   1

struct {
	__uint(type, BPF_MAP_TYPE_HASH);
	__uint(max_entries, 4096);
	__type(key, __u32);   /* IPv4 destination, network byte order */
	__type(value, __u8);
} egress_allow SEC(".maps");

SEC("lsm/socket_connect")
int BPF_PROG(guard_connect, struct socket *sock,
	     struct sockaddr *address, int addrlen)
{
	struct sockaddr_in sin;

	if (bpf_current_task_under_cgroup(&session, 0) != 1)
		return 0;                /* outside the session: not ours */

	if (addrlen < sizeof(sin))
		return 0;                /* short sockaddr: let the stack reject it */

	if (bpf_probe_read_kernel(&sin, sizeof(sin), address))
		return 0;

	if (sin.sin_family == AF_UNIX)
		return 0;                /* local IPC: a separate policy question */

	if (sin.sin_family != AF_INET)
		return -EPERM;           /* no v6 policy yet, so fail closed */

	if (bpf_map_lookup_elem(&egress_allow, &sin.sin_addr.s_addr))
		return 0;

	return -EPERM;
}
```

The `#define`s are there because `vmlinux.h` owns the type universe in
a BPF object and does not coexist with the libc headers that would
normally supply them. Userspace populates `egress_allow` with
`bpf_map_update_elem` and the destinations the session is entitled to,
and every other `connect` in the session dies with `EPERM` before a
single packet leaves. Not an alert. A refusal, in the kernel, from a
hook the agent cannot see, cannot unload without `CAP_BPF` plus
`CAP_MAC_ADMIN`, and cannot negotiate with. (For the connect case
alone there is an older road: `cgroup/connect4` programs have attached
directly to a cgroup since 4.17 and can deny per-cgroup without BPF
LSM. The LSM route wins once you want the same treatment on
`file_open`, `bprm_check_security`, and the rest of the hook surface.)

## The edges, named

Now the parts that do not hold. A recorder that hides its own blind
spots is just a prettier transcript, and the transcript is the thing we
set out not to trust.

Overhead is real and mostly irrelevant here. A tracepoint plus a ring
buffer submit costs on the order of a microsecond; an `execve` costs
hundreds of microseconds and a network connect costs milliseconds, so
for an agent workload the recorder tax is noise. It stops being noise
if you hook genuinely hot paths (`file_open` on a build tree, for
one), which is exactly when you measure before you moralise. The ring
buffer is the honest edge: under a fork storm, `reserve` fails and
events drop. That is why the `drops` counter exists. A recorder that
loses data silently is worse than none; a recorder that counts its own
gaps is still an instrument.

The verifier constrains the recorder's fidelity. Loops must be
provably bounded (bounded loops arrived in 5.3), so `MAX_ARGS` is 8
and `ARG_LEN` is 64, and a longer command line is truncated. Fine,
provided the truncation is visible in the record rather than papered
over.

`sys_enter_execve` has a subtler flaw: it reads `argv` from memory the
calling process still owns, and a hostile multi-threaded process can
rewrite that memory after the tracepoint fires and before the kernel
copies it, showing the recorder one command while running another. The
demo is honest against a confused agent, and forgeable by a determined
one. For the record of truth, capture at `sched_process_exec` or at
the `bprm` LSM hooks, which fire after the kernel has committed the
new program image and cannot be raced.

Kernel gating is the deployment reality: BTF and CO-RE want
`CONFIG_DEBUG_INFO_BTF=y`, the ring buffer wants 5.8, BPF LSM wants
5.7 plus the boot parameter, and in practice anything 5.15 or later
ships all of it. Older fleets fall back to perf buffers and
`cgroup/connect4`, and lose the wider hook surface.

Finally, the boundary itself. An unprivileged process cannot leave its
cgroup, because that takes write access to `cgroup.procs`, so the
label sticks exactly as long as the agent stays unprivileged. If the
agent gets root, this whole layer is over, and the answer is a virtual
machine boundary, which is a different essay. And an IPv4 allowlist is
only as good as its relationship to DNS; names are resolved at the
proxy layer and pinned to address sets at the kernel, or the allowlist
is theatre.

None of this requires trusting the agent, and that is the point. The
agent's transcript remains what the pilot's account has always been:
context for the investigation, checked against the instrument. The
kernel sees every `execve` and every `connect` whether or not the
agent chooses to mention them, and with BPF LSM it can refuse the ones
we never authorised. Put the recorder below the pilot. It cannot be
talked out of what it saw.
