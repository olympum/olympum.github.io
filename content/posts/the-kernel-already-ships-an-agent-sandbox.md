---
title: "The Kernel Already Ships an Agent Sandbox"
date: 2026-07-10
draft: true
categories:
- Linux
author: Bruno Fernandez-Ruiz
---

Some months ago I left a coding agent running unattended overnight against a slow refactor. In the morning transcript, somewhere around 2am, it had hit a failing integration test, decided the fix was a newer client library, and piped an install script from a stranger's README straight into bash. The script happened to be benign. Nothing about my setup made it so.

That transcript is the premise. A coding agent is a process that executes text nobody on my side wrote and nobody read, and prompt injection means every file it ingests is a potential instruction. So the agent is untrusted code by construction, whatever we think of the model behind it, and it deserves the treatment we give untrusted code. The reflex today is to reach for a sandboxing product. My conclusion, after a year of running agents on my own machines, is that on Linux we do not need one: the kernel already ships the primitives. User namespaces, mount namespaces, bubblewrap to assemble them, Landlock for filesystem and network rules, seccomp-bpf for the syscall surface. What is missing is assembly, and one architectural decision.

## Below the agent

Most agent harnesses ship a permission prompt: the model proposes a command, we approve it. That is a user-space control, and a user-space control is advisory to a process that can spawn children, load libraries, and issue raw syscalls. The child does not inherit the harness's hooks. The pattern-matcher that blocks `rm -rf` does not recognise the same command assembled from base64 fragments. And interception shims, whether `LD_PRELOAD` tricks or ptrace supervisors, are themselves attack surface with a long record of time-of-check bugs. The kernel has none of these problems, because the kernel never asks the process to cooperate; it enforces at a layer the process cannot reach.

Hence the two rules I hold. Enforcement lives below the agent, in the kernel. And the strength of the isolation is dialled to the risk of the task: a code-review agent reading my own repository does not need a virtual machine, while an agent chewing hostile input gets its own kernel. Everything between those two points is what follows.

## A view, not a boundary

Namespaces are where everyone starts, so let us be precise about what they are. A namespace gives a process a private view of one kernel resource: its own mount table, its own PID space, its own network stack. Since Linux 3.8 (2013) an unprivileged process can create a user namespace and, from inside it, the other namespace types; that is the mechanism that lets us build sandboxes without root, and it is the mechanism bubblewrap uses. But a namespace is an isolated view, not a security boundary. An empty network namespace happens to be a strong statement, because there is no interface and no route out, so there is nothing to firewall. A mount namespace, by contrast, is only as strong as what we choose to mount into it. The boundary comes from the assembly.

## bubblewrap assembles the view

bubblewrap (`bwrap`, the sandbox underneath Flatpak) is the assembler. Give the agent a read-only operating system, exactly one writable directory, and no network:

```bash
#!/usr/bin/env bash
# agent-jail WORKSPACE CMD [ARGS...]
# Read-only OS, one writable workspace, no network.
# Needs bubblewrap and unprivileged user namespaces.
exec bwrap \
  --unshare-all \
  --die-with-parent \
  --new-session \
  --clearenv \
  --setenv PATH /usr/bin:/bin \
  --setenv HOME /workspace \
  --ro-bind /usr /usr \
  --symlink usr/bin /bin \
  --symlink usr/lib /lib \
  --symlink usr/lib64 /lib64 \
  --ro-bind /etc /etc \
  --proc /proc \
  --dev /dev \
  --tmpfs /tmp \
  --bind "${1:?usage: agent-jail WORKSPACE CMD...}" /workspace \
  --chdir /workspace \
  -- "${@:2}"
```

Walking the flags: `--unshare-all` puts the process in fresh namespaces of every kind, and because we never pass `--share-net`, the network namespace stays empty: a downed loopback, no route to my LAN, no DNS. The visible root is my host's `/usr`, read-only; the only writable path is the repository, rebound as `/workspace`. `--clearenv` matters more than it looks, because a developer shell carries API keys and the agent should see none of them. `--die-with-parent` ties the sandbox's lifetime to mine. And `--new-session` detaches the sandbox from my controlling terminal so it cannot inject keystrokes into my shell through the `TIOCSTI` ioctl; that one is CVE-2017-5226, and it is exactly the kind of edge user-space sandboxes forget. Your distro may want one or two more read-only binds; add them one denial at a time.

What bwrap buys us is filesystem confinement by construction: paths outside the view do not exist. What it does not govern is what the process does with what it can see. For that we go deeper.

## Landlock travels with the process

Landlock is a kernel security module, unprivileged since Linux 5.13 (2021), that lets a process restrict itself before it executes anything interesting. The property that matters: the ruleset is enforced by the kernel, inherited across `fork` and `execve`, and impossible to remove. A child can tighten it further; nobody can loosen it. Where bwrap shapes what exists, Landlock forbids access even to what exists, and the rules travel with every process the agent spawns. glibc ships no wrappers yet, so we call the three syscalls directly:

```c
/* llrun.c: exec a command with the filesystem cut down to one
 * writable tree and TCP denied outright.
 * As written this needs Linux 6.7+ (Landlock ABI 4); see notes.
 * Build: cc -Wall -O2 -o llrun llrun.c */
#define _GNU_SOURCE
#include <fcntl.h>
#include <linux/landlock.h>
#include <stdio.h>
#include <sys/prctl.h>
#include <sys/syscall.h>
#include <unistd.h>

static int ll_create(const struct landlock_ruleset_attr *a, size_t sz, __u32 f)
{
    return syscall(SYS_landlock_create_ruleset, a, sz, f);
}
static int ll_add(int fd, enum landlock_rule_type t, const void *a, __u32 f)
{
    return syscall(SYS_landlock_add_rule, fd, t, a, f);
}
static int ll_restrict(int fd, __u32 f)
{
    return syscall(SYS_landlock_restrict_self, fd, f);
}

#define FS_RO (LANDLOCK_ACCESS_FS_READ_FILE | LANDLOCK_ACCESS_FS_READ_DIR | \
               LANDLOCK_ACCESS_FS_EXECUTE)
#define FS_RW (FS_RO | LANDLOCK_ACCESS_FS_WRITE_FILE |                      \
               LANDLOCK_ACCESS_FS_TRUNCATE | LANDLOCK_ACCESS_FS_REFER |     \
               LANDLOCK_ACCESS_FS_MAKE_REG | LANDLOCK_ACCESS_FS_MAKE_DIR |  \
               LANDLOCK_ACCESS_FS_MAKE_SYM | LANDLOCK_ACCESS_FS_MAKE_FIFO | \
               LANDLOCK_ACCESS_FS_MAKE_SOCK |                               \
               LANDLOCK_ACCESS_FS_REMOVE_FILE | LANDLOCK_ACCESS_FS_REMOVE_DIR)

static int allow(int ruleset, const char *path, __u64 access)
{
    struct landlock_path_beneath_attr rule = { .allowed_access = access };
    int r;

    rule.parent_fd = open(path, O_PATH | O_CLOEXEC);
    if (rule.parent_fd < 0)
        return -1;
    r = ll_add(ruleset, LANDLOCK_RULE_PATH_BENEATH, &rule, 0);
    close(rule.parent_fd);
    return r;
}

int main(int argc, char **argv)
{
    if (argc < 3) {
        fprintf(stderr, "usage: %s WORKSPACE CMD [ARGS...]\n", argv[0]);
        return 2;
    }

    struct landlock_ruleset_attr attr = {
        /* handle every right ABI 4 knows: unhandled means ungoverned */
        .handled_access_fs = FS_RW | LANDLOCK_ACCESS_FS_MAKE_BLOCK |
                             LANDLOCK_ACCESS_FS_MAKE_CHAR,
        /* handled but never granted: TCP is default-deny */
        .handled_access_net = LANDLOCK_ACCESS_NET_BIND_TCP |
                              LANDLOCK_ACCESS_NET_CONNECT_TCP,
    };

    int ruleset = ll_create(&attr, sizeof(attr), 0);
    if (ruleset < 0) {
        perror("landlock_create_ruleset");
        return 1;
    }
    if (allow(ruleset, "/usr", FS_RO) || allow(ruleset, "/etc", FS_RO) ||
        allow(ruleset, argv[1], FS_RW)) {
        perror("landlock_add_rule");
        return 1;
    }
    if (prctl(PR_SET_NO_NEW_PRIVS, 1, 0, 0, 0) || ll_restrict(ruleset, 0)) {
        perror("landlock_restrict_self");
        return 1;
    }
    close(ruleset);

    execvp(argv[2], &argv[2]);
    perror("execvp");
    return 1;
}
```

Running it, the agent's world shrinks to read-only `/usr` and `/etc` plus one writable tree:

```
$ ./llrun ~/src/repo bash
$ cat ~/.ssh/id_ed25519
cat: /home/bruno/.ssh/id_ed25519: Permission denied
$ touch /usr/bin/backdoor
touch: cannot touch '/usr/bin/backdoor': Permission denied
$ strace -qq -e trace=connect curl -s http://example.com > /dev/null
connect(5, {sa_family=AF_INET, sin_port=htons(53), ...}, 16) = 0
connect(7, {sa_family=AF_INET, sin_port=htons(80), ...}, 16) = -1 EACCES (Permission denied)
```

Three things worth noticing. First, a ruleset declares which access rights it handles and then grants them back selectively; anything handled and not granted is denied, and anything not handled is ungoverned, so we handle everything the target ABI knows. Second, `PR_SET_NO_NEW_PRIVS` is mandatory before `landlock_restrict_self`, and it is what stops a confined process escalating through setuid binaries. Third, look closely at the curl trace: the DNS lookup on port 53 succeeded and the TCP connect failed. That is because Landlock governs TCP bind and connect and nothing over UDP, through ABI 6 at least. Hence the empty network namespace from bwrap is not redundant; the layers cover each other's edges.

Version notes, since Landlock is versioned by ABI rather than by feature flags: filesystem rules need 5.13 (ABI 1), truncate needs 6.2 (ABI 3), the TCP rules need 6.7 (ABI 4), and scoping of signals and abstract Unix sockets arrived in 6.12 (ABI 6). The code above wants 6.7. Production code probes `landlock_create_ruleset(NULL, 0, LANDLOCK_CREATE_RULESET_VERSION)` and degrades to whatever the running kernel supports.

## seccomp shrinks the syscall table

The remaining surface is the syscall table itself. x86_64 exposes more than 450 syscalls; an agent's toolchain needs perhaps a third of them, and the rest are pure downside, because kernel privilege-escalation bugs live disproportionately in the corners no ordinary workload visits. seccomp-bpf, in the kernel since 3.5 (2012), attaches a BPF program that classifies every syscall by number and register arguments before it runs. It cannot dereference pointers, which is why path rules are Landlock's job and syscall rules are seccomp's.

```python
#!/usr/bin/env python3
# confine.py CMD [ARGS...]
# apt install python3-seccomp (libseccomp bindings)
import errno, os, sys
import seccomp

CLONE_NEWUSER = 0x10000000

f = seccomp.SyscallFilter(defaction=seccomp.ALLOW)

# no introspection of, or injection into, other processes
for sc in ("ptrace", "process_vm_readv", "process_vm_writev"):
    f.add_rule(seccomp.ERRNO(errno.EPERM), sc)

# kernel surface the agent has no business touching
for sc in ("bpf", "keyctl", "add_key", "request_key",
           "mount", "umount2", "pivot_root", "kexec_load"):
    f.add_rule(seccomp.ERRNO(errno.EPERM), sc)

# no new user namespaces from inside the sandbox
f.add_rule(seccomp.ERRNO(errno.EPERM), "unshare",
           seccomp.Arg(0, seccomp.MASKED_EQ, CLONE_NEWUSER, CLONE_NEWUSER))
f.add_rule(seccomp.ERRNO(errno.EPERM), "clone",
           seccomp.Arg(0, seccomp.MASKED_EQ, CLONE_NEWUSER, CLONE_NEWUSER))
# clone3 carries its flags in a struct in user memory, where BPF
# cannot look; fail it with ENOSYS so libc falls back to clone.
f.add_rule(seccomp.ERRNO(errno.ENOSYS), "clone3")

f.load()
os.execvp(sys.argv[1], sys.argv[1:])
```

Tracing from outside the filter shows the denial:

```
$ strace -f -qq -e trace=unshare ./confine.py unshare -U true
unshare(CLONE_NEWUSER)                  = -1 EPERM (Operation not permitted)
unshare: unshare failed: Operation not permitted
+++ exited with 1 +++
```

The `clone3` rule deserves its comment: its flags live in memory rather than a register, so an honest filter fails it with `ENOSYS` and lets glibc fall back to plain `clone`, whose flags we can inspect. Docker and systemd do the same. And I will concede the larger point before anyone raises it: a denylist like this is a teaching example. A production filter is an allowlist, deny by default, generated from a trace of what the agent actually needs; Docker's default profile is the prior art, and bwrap will load a compiled filter for you through `--seccomp` so the two layers compose.

The `unshare` and `clone` rules close a loop from earlier. User namespaces are how we built this sandbox without root; they are also, once inside, a road to kernel code paths that were root-only for twenty years, and they carry a steady CVE record for it. Ubuntu has been restricting unprivileged user namespaces through AppArmor since 23.10 for exactly this reason. So the sandbox spends one user namespace on the way in, then denies the agent the ability to make more. A ratchet, not a ladder.

## Dial the isolation to the risk

Assembled, the stack is bwrap outermost, then a small launcher inside that installs the Landlock ruleset and the seccomp filter and execs the agent. For the common case, a capable agent working my own repository on inputs I mostly trust, that stack is proportionate: it starts in milliseconds, costs no memory worth mentioning, and turns a 2am surprise into a diff I can review over coffee.

But I will name the edge rather than wave at it. Everything above shares my kernel. Namespaces, Landlock, and seccomp are all kernel code, and a single local privilege-escalation bug ends the argument; seccomp shrinks that attack surface without removing it. So when the input is hostile by construction, an agent triaging pull requests from strangers, browsing arbitrary pages, or executing attacker-supplied test cases, the shared kernel is the wrong trade and the agent gets its own: a microVM under Firecracker or QEMU, booting in roughly 125 milliseconds with a few megabytes of overhead. The opposite mistake is just as real, though. Treat the VM as the only respectable answer, impose that tax on every code-review task, and people will find the friction intolerable and run agents with no sandbox at all. Proportionate beats maximal, because proportionate is what survives daily use.

There is no product here. Three kernel primitives, a page of C, a page of Python, and a bash script, all of it maintained in the kernel tree or one package install away. Agents are going to run unattended either way. The only open question is what sits below them when they do, and the kernel is already there.
