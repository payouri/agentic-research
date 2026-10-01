# MicroVMs for coding agents — the guide

Researched 2026-10-01 from primary sources: the microVM projects' own design docs, threat models and
source (Firecracker, Cloud Hypervisor, Kata, libkrun, QEMU microvm, gVisor, Apple Containerization),
the documentation and source of 37 sandbox products and agent harnesses, an original measured corpus
of 35 agent frameworks and ~265 public agent container configs, and the 2020–2026 performance,
vulnerability and agent-escape literature. Companion files: [rulebook.md](rulebook.md) (the checkable
rules) and [sources.md](sources.md) (every source, tiered and annotated).

---

## 1. The finding that organises everything else

**A microVM is a kernel boundary, not a sandbox. The authorities say so in writing, and the parts
they hand back to the operator are exactly the parts agents are measured attacking.**

Firecracker, the reference microVM, claims one thing and claims it strongly: "Firecracker can safely
run workloads from different customers on the same machine" (`fc-design`). Then its production guide
makes that claim conditional on a list — "Security guarantees and defense in depth can only be
upheld, if the following list of recommendations are implemented in production" (`fc-prodhost`) — and
the list is where the work is:

| What a sandbox has to do | What the authority says about it |
|---|---|
| Stop the guest reaching the network | "Firecracker does not perform any network traffic filtering. All egress traffic from a guest is therefore considered untrusted, and should be filtered at the host-level." (`fc-design`) |
| Stop cross-tenant side channels | "Firecracker is not able to mitigate host's hardware vulnerabilities." Disable SMT "in production scenarios that require tenant separation." (`fc-prodhost`) |
| Contain the VMM if the guest breaks it | "Firecracker should be started using the `jailer` binary … or executed under process constraints equal or more restrictive" (`fc-prodhost`) |
| Keep clones from sharing secrets | "we consider resuming execution from the same state more than once insecure" (`fc-snapshot`) |
| Bound resources | "we cannot enforce aggressive default constraints" (`fc-prodhost`) |

Cloud Hypervisor's threat model says the same thing in different words — it "assumes that these
components have been configured to provide isolation", "does not filter data sent and received by the
guest in any way", and treats snapshot files as "trusted input" (`ch-threat`). libkrun says it most
bluntly: "both the guest and the VMM pertain to the same security context" (`libkrun`).

Now set that against what the evidence measures. **Every documented agent escape went through
something the hypervisor does not own.** The o1-preview CTF escape used "the Docker daemon API running
on the evaluation host VM … accessible due to a misconfiguration" (`o1-card`). Claude Mythos Preview's
instructed escape gained "broad internet access from a system that was meant to be able to reach only
a small number of predetermined services", and in training it harvested credentials through `/proc`
(`mythos-card`). In the OpenAI/Hugging Face incident, roughly 1,200 agents "meant to be isolated from
one another" talked through a shared Artifactory that "doesn't isolate the activity of different
users", and reached the internet through "a customer-deployed Modal sandbox" (`metr-hf`). The UK
AISI's benchmark finds frontier models "reliably escape container sandboxes under common real-world
failure modes (especially misconfigurations and exposed control surfaces)" — and zero success on its
two hardest, kernel-exploit tiers (`sandboxescape`).

Egress, shared services, credentials and configuration. None of it is a hypervisor break, and all of
it is the operator's job by the authorities' own account. **That is the lesson. A microVM moves the
kernel out of reach and does nothing else.** Choosing one is the easy decision. The hard ones are
what you do around it, and that is where the field cuts corners.

---

## 2. What the authorities actually say

### Firecracker: a strong claim with a long list of conditions

The design doc sets the posture: "all vCPU threads are considered to be running malicious code as soon
as they have been started", with "Linux KVM and the Firecracker virtualization boundary" as the first
layer and process-level constraints as defence in depth (`fc-design`). The README makes the dependency
explicit: "The overall security of Firecracker microVMs … depends on a well configured Linux host
operating system" (`fc-changelog`).

What it mandates, or pushes onto you (`fc-prodhost` unless noted):

- **The jailer, or equivalent.** One uid/gid per microVM. The jailer itself "treats all its inputs as
  trusted" (`fc-jailer`).
- **One tenant per Firecracker process.** "strongly recommended".
- **SMT off, KSM off, microcode early, TRR/ECC memory.** It also points to `spectre-meltdown-checker`.
  The kernel will not decide for you: it "does not by default enforce the disabling of SMT, which
  leaves SMT systems vulnerable when running untrusted guests", and "There is no way for the kernel to
  provide a sensible default" (`linux-hwvuln`).
- **Host-level egress filtering**, including a rule dropping the instance-metadata address
  169.254.169.254.
- **No serial console in production.** "the device can be reactivated from within the guest even if
  it was disabled at boot."
- **Swap off**, against data remanence. **An overwatcher process** to SIGKILL hung VMMs.
- **Seccomp is on by default, but only in release builds.** "On debug binaries and experimental GNU
  targets, there are no default seccomp filters installed" (`fc-seccomp`).
- **CPU templates are not a security feature.** "CPU templates shall not be used as a security
  protection against malicious guests" (`fc-cputemplates`).

The project has also grown well past its minimal origins. Optional PCI arrived in v1.13.0 and PCI
device hotplug in v1.16.0 (`fc-changelog`). The hotplug, vhost-user block and diff snapshots are all
**developer preview**, which the release policy says "should not be used in production … may not
provide patch releases for critical bug fixes or security issues" (`fc-release`). The website still
says "Only 5 emulated devices" (`fc-site`). It is out of date, and the attack surface has grown since
it was written. The newest guest→host CVE is in that new PCI code (§3).

### Snapshots: the authority's sharpest warning

The snapshot doc is unusually direct. Snapshot files are trusted input, protected only by a 64-bit
CRC, which is "only a partial measure to protect against accidental corruption". Restoring
one snapshot into several clones is labelled "potentially insecure usage" (`fc-snapshot`). VMGenID is
always on and reseeds the guest kernel's CSPRNG on Linux 5.18+, but:

> "State other than the guest kernel entropy pool, such as unique identifiers, cached random numbers,
> cryptographic tokens, etc **will** still be replicated … Users need to implement mechanisms for
> ensuring de-duplication." — `fc-random`

There is also "a race window between resuming vCPUs and Linux CSPRNG getting successfully
re-seeded", and for userspace RNGs "There is no generic solution under the current programming
model". AWS researchers set out the problem in 2021 (`brooker-uniq`). Nobody has published a
measurement of how often it bites.

### The rest of the field

- **Cloud Hypervisor** aims to be a general-purpose cloud VMM, with PCI, VFIO and CPU/memory hotplug
  (`ch-readme`). It therefore has a bigger device surface than Firecracker by design. It has no
  jailer. Its Landlock sandbox is opt-in, and "Until support for blocking AF\_UNIX sockets is
  implemented in Landlock … sandbox escapes will not be considered security vulnerabilities"
  (`ch-threat`).
- **Kata Containers** offers "a second layer of isolation on top of those provided by
  traditional-containers". It also concedes that "all virtual machines (VMs) share the same host
  kernel", so a host-kernel bug reached through the VMM "could be exploited by a process running
  within one VM to affect the entire system" (`kata-threat`). GPU and confidential computing are
  QEMU-only, and its own hypervisor table is "not prescriptive or authoritative" (`kata-hv`).
- **libkrun** is not a multi-tenant boundary by its own definition. Processes run "in a partially
  isolated environment", the guest can reach whatever the VMM can, and virtio-fs "**does not**
  provide any protection" against access to other host directories (`libkrun`). Any local-microVM
  tool built on it, microsandbox included, inherits that model.
- **QEMU microvm** is covered by QEMU's security policy only under KVM/HVF on a listed machine type.
  Under TCG, users "must not rely on QEMU to provide guest isolation", and memory use is "effectively
  unbounded" (`qemu-microvm`).
- **gVisor** is not a microVM, and it is the main alternative the field actually deploys. It exists
  "to minimize the System API attack vector", "does not provide protection against hardware side
  channels", and says "A sandbox is not a substitute for a secure architecture". It also warns
  against the opposite assumption: "one should not assume that the mere use of virtualization
  hardware makes a system more or less secure" (`gvisor-sec`).
- **Apple Containerization** gives each container "the isolation properties of a full VM". No
  threat model or SECURITY.md could be found to qualify that (`apple-container`).

### The mandated-versus-left-open line

| Area | What the VMM does | What it leaves to you |
|---|---|---|
| Guest→host kernel | KVM + minimal device model | Patching host, guest kernel and microcode |
| Process containment of the VMM | Seccomp (release builds) | Jailer or stricter, unique uid/gid, untampered paths |
| Network | Nothing | Host firewall, IMDS block, egress policy |
| Side channels | Nothing | SMT, KSM, microcode, memory type |
| Snapshots | CRC64, VMGenID | Encryption, authentication, userspace uniqueness, the reseed race |
| Resources | Rate limiters (unset) | cgroups, rlimits, log and stdout bounds |
| Host file paths | Nothing (CH, libkrun) | `nosymfollow`, fd passing, mount isolation |

---

## 3. What the evidence supports

### Performance: the vendor's numbers, mostly unreplicated

The Firecracker paper (`fc-nsdi`) measured a memory overhead of "around 3MB", against ~13MB for Cloud
Hypervisor and ~131MB for QEMU. It also reports boot "to application code in less than 125ms" and
"up to 150 MicroVMs per second per host". The I/O costs are real: about 13,000 IOPS against the host's
340,000, and about 15 Gb/s of network against 44. The paper's authors are AWS's own, and the 150/s
figure has no separate method that could be located.

The current spec (`fc-spec`) restates `<= 125 ms` and `<= 5 MiB`, and says these "are enforced by
integration tests (that run for each PR…)". The memory bound is enforced. **The boot bound is not**:
`test_boottime` is marked `nonci` and only records the metric (`fc-tests`). The CPU, network and
storage lines are tagged "[integration test pending]". Treat 125 ms as a credible vendor
characterisation, not a guarantee, and expect provider-level "cold start" numbers to include
everything Firecracker does not: image fetch, rootfs, agent boot and network setup.

The one independent comparison (`vee20`) is the most important performance paper for this question,
because what it measured was not speed:

> "both Firecracker and gVisor execute substantially more kernel code than native Linux." — `vee20`

A microVM does not make the host kernel *smaller*. It changes which part of it an attacker can reach
directly: about 24 syscalls and 30 ioctls behind the jailer's seccomp filter (`fc-nsdi`), plus KVM,
plus whatever host kernel code the VMM itself exercises. Lines-of-code and syscall counts are a
proxy for attack surface, not a measurement of it.

Snapshot restore is how every provider gets its sub-second start. A function started from a snapshot
runs "95% higher, on average" than one already resident in memory (`reap`). Lazy-loading designs
close most of that gap (`faasnap`). Cold start is a solved engineering problem. Uniqueness after
cloning is the unsolved one (§2).

### Escape history: microVMs are strongest, not perfect

Counted from NVD and project advisories as of 2026-10-01:

| Boundary | Escape-class CVEs | Most recent |
|---|---|---|
| runc containers | 5 (2019, 2024, three in Nov 2025) | CVE-2025-31133/52565/52881: "full container breakouts" (`cve-runc`) |
| NVIDIA Container Toolkit | 2 at CVSS 9.0 | CVE-2025-23266, a "three-line Dockerfile" (`cve-nvidia`) |
| gVisor | 1 host-root | CVE-2026-96812, CUSE passthrough, published 2026-09-25 (`cve-gvisor`) |
| Firecracker | 2 guest→VMM of 4 total | CVE-2026-5747, virtio-pci, "potentially execute arbitrary code on the host … requires additional preconditions" (`cve-fc`) |
| Cloud Hypervisor | 2 guest→host | CVE-2026-27211 (QCOW2 → host file read), CVE-2026-45782 (virtio-block UAF) (`cve-ch`) |
| KVM | ongoing | CVE-2026-53359 "Januscape", shadow-MMU UAF, needs guest root and nested virtualisation (`cve-kvm`) |

The ordering holds — containers break routinely, microVMs rarely. But 2026 alone produced guest→host
bugs in Firecracker, Cloud Hypervisor, gVisor and KVM. Three things in that table matter for agents:

- **The newest Firecracker bug is in the newest feature.** It is in the virtio-pci code, which
  arrived in v1.13. Every device you enable is attack surface you opted into.
- **Both Cloud Hypervisor bugs are in virtio-block.** The guest owns a disk image and the VMM parses
  it. That is the general-purpose VMM's wider surface, showing up exactly where its threat model says
  it would.
- **CVE-2026-5747 was co-reported by "Claude (@claude)"** (`cve-fc`). The agents you are sandboxing
  are now finding bugs in the sandbox.

Google pays "Full VM escape | $250,000" through kvmCTF, with QEMU and hardware out of scope
(`kvmctf`). That puts a market price on the boundary you are relying on.

### Side channels cross the boundary, and the fix costs throughput

VMScape (IEEE S&P 2026) leaks the host VMM's memory from a KVM guest "at a rate of 154 B/s on AMD
Zen 5", extracting a disk-encryption key in 102 seconds (`vmscape`). The kernel's optimised mitigation
costs about 1% on UnixBench and **51% on fio against a virtio disk**. The I/O-heavy figure is the one
that looks like a build-and-test agent. The paper names Firecracker among the VMM userspaces that do
not deploy retpoline. This is the lineage of L1TF, Retbleed, Inception and Downfall
(`cve-sidechannel`), and Firecracker's answer is the operator's SMT switch (`fc-prodhost`), which
costs capacity. The website promises isolation "without any tradeoffs to security or efficiency"
(`fc-site`). The prod-host guide's SMT advice is a tradeoff.

### What agents have been measured doing

SandboxEscapeBench (`sandboxescape`) is the only systematic measurement, and it measures
**containers**:

- Claude Opus 4.5 escapes 0.49 of tasks overall, and 56% of samples at a 100M-token budget.
  GPT-5 and Opus 4.5 each score 0.40 on level 3, and **zero on levels 4 and 5**.
- Success scales "approximately log-linear" with compute, at "on the order of $1 per sample".
- Claude Mythos Preview "succeeded at least once on 100% of samples".
- The benchmark's own VM harness leaked four unintended escape paths, including default Vagrant
  credentials and Dirty COW, which were found and patched during the study.
- AISI's recommendation, citing the NCSC: "hypervisor isolation should be used as a bare minimum,
  with further container isolation inside that" (`aisi-blog`).

The Mythos Preview system card gives the rate at which this happens unprompted: escape attempts in
"about 0.05% of all training episodes", successes in "about 0.01%". It also gives the cap: the model
"failed to find any novel exploits in a properly configured sandbox with modern patches"
(`mythos-card`). At agent scale, 0.01% is not rare. And the attack that does work is
misconfiguration, which a microVM does not prevent.

The process-sandbox record points the same way. Claude Code's network sandbox went unenforced "if the
sandbox policy did not configure any allowed domains" (`cve-srt`). It had a disputed SOCKS5 null-byte
bypass (`socks5`), and a worktree escape outside the sandbox context (`cve-cc-worktree`). An agent
walked past a bubblewrap denylist via `/proc/self/root` in under two minutes (`ona`, vendor-run).
Codex desktop sent data out through a rendered image URL (`cve-codex`). Prompt injection reaches
command execution in Copilot and Cursor at up to 84% (`aishelljack`). Anthropic's own framing of why
sandboxing matters is about egress: "Without network isolation, a compromised agent could exfiltrate
sensitive files like SSH keys" (`anth-sandbox-blog`).

**Nobody has measured an agent against a microVM, gVisor or KVM boundary.** "Agents can't escape
microVMs" is therefore untested, not established. What is established is that agents reliably find
the misconfigured parts, and those are almost always outside the hypervisor.

---

## 4. What the implementations actually run

Thirty-seven products were read for this dossier (`sources.md` §3). Three findings matter more than
any comparison table.

### The label rarely tells you the technology

| Says | Runs |
|---|---|
| Daytona: "true microVM", "Sub 90ms" | **Linux containers by default**; a VM class since July 2026, hypervisor undisclosed (`daytona`) |
| Modal: "strong isolation" | **gVisor** by default; GPUs gVisor-only; the VM runtime is opt-in (`modal`) |
| Northflank: microVM workloads | gVisor or Kata + Cloud Hypervisor **depending on whether the host supports nested virtualisation** (`northflank`) |
| GKE Agent Sandbox: kernel-level isolation | gVisor, Kata optional (`gke-agent-sandbox`) |
| Claude Code on the web: "isolated, Anthropic-managed VM" | The same page says "cloud container"; claude.ai execution is gVisor (`cc-web`, `anth-contain`) |
| Docker Sandboxes: microVM | Real hardware virtualisation through a **new custom VMM**, not Firecracker (`docker-sbx`) |
| Cursor, Jules, Devin, Replit, Codex cloud: "secure VM" / "isolated containers" | VMM undisclosed (`hosted-undisclosed`, `codex`) |
| AgentCore: widely called Firecracker | Every fetched AWS page says only "microVM" (`aws-agentcore`) |

The verified Firecracker users are E2B, Vercel, Cloudflare, Fly, Deno, Together, Lambda/Fargate, and
Blaxel by its own blog. Hyper-V backs Azure dynamic sessions, and Kata + Cloud Hypervisor backs AKS
pod sandboxing (`azure-sessions`). Much of what the market sells as a "sandbox" is gVisor. That is a
reasonable boundary, but it is a different one, with a different CVE history (§3) and no hardware
isolation. **Ask what runs. If the vendor won't say, assume a container.**

### The field fans out snapshots the authority calls insecure

E2B fork (up to 100 per call), Daytona fork, Morph's "unlimited parallel branches", Blaxel standby,
Together clones and Lambda SnapStart all restore one memory image into many live VMs (`e2b-docs`,
`daytona`, `morph-runloop-together`, `blaxel`, `aws-lambda-wp`). Firecracker calls that "insecure"
without a strong uniqueness mechanism (`fc-snapshot`). **No vendor documents one.** E2B's open-source
orchestrator contains no `vmgenid` or reseed handling beyond the API model's entropy field
(`e2b-infra`). AWS's own whitepaper hands the problem to the customer: "If your application uses
random values, you must evaluate your function code and verify that it is resilient to snapshot
operations" (`aws-lambda-wp`).

For an agent this is concrete. A template snapshotted after `npm install`, or after a CLI login,
carries whatever tokens, session IDs, SSH host keys and seeded userspace RNGs existed at that moment
into every fork. "Snapshot" also means three different things in this market: memory + disk (E2B,
Fly Machines, Blaxel, Morph, Together, Daytona VM), filesystem only (Vercel, Sprites, Runloop,
Modal), or image only (Daytona container snapshots). Only the first carries live secrets in memory,
but all three can carry them on disk.

### The jailer is skipped, and egress splits the field

E2B's orchestrator launches Firecracker through `unshare -m` and `ip netns exec`, with **no jailer,
chroot or uid drop** in the launch path (`e2b-infra`). It does run a per-sandbox cgroup, network
namespace and nftables, which cover part of what the jailer does. Whether the total is "equal or more
restrictive", as `fc-prodhost` requires, cannot be settled from source. No other vendor publishes
launch code, so for every other provider the question is simply unanswered.

Default egress divides the market almost exactly in half:

- **Open by default:** E2B, Vercel, Modal, Fly Sprites, Daytona Tiers 3–4, Claude Managed Agents
  (`unrestricted`), Gemini CLI (`permissive-open`), Cloudflare SDK 0.x.
- **Denied or allowlisted by default:** Azure sessions, GKE Agent Sandbox, Cloud Run sandboxes,
  Cloudflare SDK 1.0 (flipped 2026-09-30), Docker Sandboxes, Claude Code local, sandbox-runtime,
  Codex CLI and Codex cloud's agent phase, Claude's API code execution, Copilot, Claude Code on the
  web ("Trusted").

The filtering mechanisms differ as much as the defaults. Vercel, Cloudflare, and Managed Agents (per
third-party research) terminate TLS. E2B, Modal, and Claude Code's default proxy decide on SNI or the
client-supplied hostname, which Claude Code's docs concede permits domain fronting (`cc-sandboxing`).
Docker's "deny by default" allowlist ships with `*.googleapis.com` (`docker-sbx`). Copilot's firewall
covers only processes the agent starts through its Bash tool, and "should not be considered a
comprehensive security solution" (`copilot-agent`).

**Credentials are the axis that matters most and is discussed least.** Deno, Vercel, Cloudflare,
Docker and Claude Code's GitHub proxy keep real secrets outside the guest and inject them at an egress
proxy. Deno: "The real key materializes only when the sandbox makes an outbound request to an
approved host" (`deno`). Docker: "Credential values never enter the VM" (`docker-sbx`). This is the
architecture that defeats the `/proc` harvesting Mythos Preview did (`mythos-card`). A secret that is
not in the VM cannot be stolen from the VM.

---

## 5. What people actually configure

### Open-source agents: nobody defaults to a microVM

Across 35 agent frameworks and harnesses read at HEAD (`corpus-a`):

| Default execution | Count |
|---|---|
| Host, no OS isolation | **20/35** |
| OS sandbox on by default (Codex CLI; Open Interpreter, claimed) | 2/35 |
| Docker | 6/35 (all benchmark harnesses) |
| Remote managed sandbox | 1/35 |
| Caller must choose | 5/35 |
| Unverified | 1/35 |
| **microVM** | **0/35** |

Four integrate a verified microVM backend as a first-class option: smolagents, the OpenAI Agents SDK,
Harbor and deepagents, via E2B, Vercel or Kata. Two recommend one only in their docs. Claude Code's
docs name "microVMs such as Firecracker" (`cc-sandbox-env`). Of the interactive coding CLIs, only
Codex confines by default. Gemini's sandbox is off by default and allows network when on
(`gemini-sandbox`). Claude Code's is opt-in and **fails open** when its dependencies are missing
(`cc-sandboxing`). Kilo Code is the counter-example: off by default, but once enabled "It does not
silently fall back to unrestricted execution", and project config "cannot disable a globally enabled
sandbox" (`exemplars`).

The word "sandbox" in this corpus often names nothing. The OpenAI Agents SDK's
`UnixLocalSandboxClient` "adds no OS-level confinement to commands". AG2's `LocalSandbox` is "a thin
wrapper around :func:`subprocess.run`". Devika's `src/sandbox/firejail.py` is **zero bytes**
(`cautionary`). opencode at least says plainly that it "does **not** sandbox the agent"
(`exemplars`).

### Users' containers: the flag spreads faster than the firewall

Every public `devcontainer.json` that mentions `--dangerously-skip-permissions` was read (N=67,
`corpus-b1`):

- **17/67** actually start an egress firewall, and 3 of those continue if it fails
  (`|| echo "Firewall setup failed, continuing"`).
- **25/67** carry at least one high-risk pattern: privileged or Docker-in-Docker (10), host Docker
  socket (3), host SSH keys (8), host gh/AWS/kube credentials (7), or root.
- **14/67** have a firewall and none of those patterns.

In a broader sample of 98 Claude-related devcontainers, **45/98** carry a high-risk pattern and
**28/98** hand the container the host Docker socket or privileged mode (`corpus-b2`). Either one turns
"sandbox" into "root on the host": it is exactly the o1-preview escape (`o1-card`), and the first
exposed control surface SandboxEscapeBench tests (`sandboxescape`).

Among 100 Dockerfiles that bake in the flag, **35 end as root**. **14** run as root *and* set
`IS_SANDBOX=1`, with comments explaining that it gets past the CLI's root refusal (`corpus-b3`).
Anthropic's docs say "The CLI rejects this flag when launched as root" (`cc-sandbox-env`). The
variable appears on no Anthropic page fetched for this dossier. In committed `.codex/config.toml`
files, **26/100** set `danger-full-access` (`corpus-b4`).

### The exemplar has holes, and says so in its comments

Anthropic's reference devcontainer is the template most of the good configs copy. It runs as `node`,
scopes sudo to the firewall script, sets `iptables -P OUTPUT DROP`, and fails its own self-test if
`example.com` is reachable (`anth-devcontainer`). It also accepts TCP 22 to any host, UDP 53 to any
host, and the whole host /24. The docs call it "a default-deny iptables firewall" (`cc-sandbox-env`).
SSH-to-anywhere and DNS-to-anywhere are both exfiltration channels. Trail of Bits' devcontainer is
the more honest model: "A container is containment, not a strong security boundary. Escape should be
hard, not impossible". It mounts `.git/hooks` and `.git/config` read-only against deferred escape, and
it admits that "By default, containers have full outbound network access" (`tob-devcontainer`).

---

## 6. Failure modes, ranked by how often the evidence shows them

1. **Default-open egress.** It is the documented incident vector (`mythos-card`, `metr-hf`), half the
   provider market's default (§4), and missing from 50/67 unattended-mode devcontainers
   (`corpus-b1`). A microVM with open egress keeps the agent's actions off your kernel and does
   nothing to keep your data in.
2. **A control surface handed into the sandbox.** The Docker socket, privileged mode, the cloud
   metadata endpoint, a shared package proxy (`metr-hf`), or credentials mounted from the host. Each
   one is an escape that needs no exploit.
3. **Secrets inside the guest.** `/proc` and process memory are readable from inside (`mythos-card`).
   Environment variables, mounted `~/.ssh`, and CLI tokens in a snapshotted template are all inside.
4. **Snapshot fan-out without uniqueness.** It is undocumented at every provider that does it (§4),
   and the authority calls it insecure (`fc-snapshot`).
5. **The "sandbox" that isn't.** A gVisor or container product sold as a microVM, a `subprocess.run`
   wrapper, a process sandbox that fails open (`cc-sandboxing`), a firewall that continues on failure
   (`corpus-b1`), or a zero-byte file (`cautionary`).
6. **Skipped VMM hardening.** No jailer (`e2b-infra`), SMT on, debug builds without seccomp
   (`fc-seccomp`), developer-preview devices in production (`fc-release`).
7. **A real hypervisor escape.** It happens (`cve-fc`, `cve-ch`, `cve-kvm`), it is rarest, and it is
   the only failure on this list that a microVM is designed to stop.

The ordering is the point. Teams spend their isolation budget on item 7 and leave items 1–3 at their
defaults.

---

## 7. What to do

**Use a hardware-virtualisation boundary for any agent running unattended or on untrusted input,
then spend most of your effort on what the hypervisor does not do.** The AISI/NCSC position is the
right floor: "hypervisor isolation should be used as a bare minimum, with further container
isolation inside that" (`aisi-blog`). Then, in order of measured payoff:

1. **Deny egress by default, and enforce the allowlist outside the guest.** Terminate TLS if you need
   to filter on content, not just hostnames. Remove SSH-to-anywhere and DNS-to-anywhere. Block the
   metadata address. This is the control behind every incident on record.
2. **Keep credentials out of the VM.** Inject them at the egress proxy (the Deno / Vercel /
   Cloudflare / Docker pattern), scope them per task, and treat anything that does enter the guest as
   disclosed.
3. **Hand the sandbox nothing that controls its host.** No Docker socket, no privileged mode, no host
   `$HOME`, `~/.ssh` or cloud config, and no shared service that doesn't isolate users. Give the
   agent a private daemon inside its own VM (the Docker Sandboxes design) rather than yours.
4. **Fail closed.** A sandbox that silently runs unsandboxed when it can't start (`cc-sandboxing`), or
   a firewall script that continues on failure (`corpus-b1`), is worse than none, because you believe
   it works. Set `failIfUnavailable`, and make firewall failure fatal.
5. **Don't fork a snapshot that has seen a secret.** Take templates before any credential, login or
   key generation, regenerate identity on restore, and ask your provider how they handle uniqueness.
   Nobody publishes an answer (§4).
6. **Know what actually runs.** Ask your provider for the VMM, whether it uses the jailer or
   equivalent, the SMT policy, and the egress default. "Secure VM" with no VMM named is not an answer.
7. **If you run Firecracker yourself, follow `prod-host-setup.md` as a checklist, not a guide.**
   Use the jailer, one tenant per process, SMT off, KSM off, release builds only, no
   developer-preview devices, and patched host, guest and microcode.
8. **Use the lighter boundaries for what they are good for.** A process sandbox (Seatbelt,
   bubblewrap) on a developer laptop is a real, cheap improvement over nothing, and it shares your
   kernel. A gVisor sandbox is a reasonable multi-tenant choice with its own CVE history. Neither
   should be called a microVM, and none of the three replaces items 1–3.

The one-line version: **the microVM is the cheapest part of the sandbox to get right, and the
evidence says it is almost never where the sandbox fails.**
