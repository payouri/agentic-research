# MicroVMs source lexicon

Every source behind [guide.md](guide.md) and [rulebook.md](rulebook.md), keyed for citation.
Gathered 2026-10-01.

Trust tiers: **P1** primary spec, vendor documentation, advisory or source code · **P2** peer-reviewed
or arXiv research · **P3** vendor engineering blog / industry research with method · **P4**
practitioner report with measurement (including this dossier's own counts, method stated) · **P5**
opinion, anecdote, news, or unverified secondary.

Repo docs were read as raw files with their last-commit SHA; "main @ SHA, date" is that commit.
Quotes marked **(WF)** passed through a summarising fetcher and may be lightly paraphrased.

---

## 1. Primary — what the microVM authorities say

### Firecracker (firecracker-microvm / AWS) — v1.17.0, released 2026-09-10; main @ 21f19ed810, 2026-09-30

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `fc-design` | [docs/design.md](https://github.com/firecracker-microvm/firecracker/blob/main/docs/design.md) | P1 | e6f0079685, 2026-09-16 | The multi-tenancy claim — "Firecracker can safely run workloads from different customers on the same machine"; vCPU threads "considered to be running malicious code"; KVM as first layer, process constraints as defence in depth; "5 microVMs per host core per second"; "Firecracker does not perform any network traffic filtering" |
| `fc-prodhost` | [docs/prod-host-setup.md](https://github.com/firecracker-microvm/firecracker/blob/main/docs/prod-host-setup.md) | P1 | fe9f12adc0, 2026-05-06 | Guarantees "can only be upheld, if the following list of recommendations are implemented"; jailer-or-stricter; "not able to mitigate host's hardware vulnerabilities"; one tenant per process; **disable SMT**, disable KSM, TRR/ECC; egress "should be filtered at the host-level" incl. IMDS; serial off in production; overwatcher; jailer `no-file` "defaults to 4096" (wrong — see conflicts) |
| `fc-jailer` | [docs/jailer.md](https://github.com/firecracker-microvm/firecracker/blob/main/docs/jailer.md) + [`src/jailer/src/resource_limits.rs`](https://github.com/firecracker-microvm/firecracker/blob/main/src/jailer/src/resource_limits.rs) | P1 | bae07c5241, 2026-09-28 | "The jailer treats all its inputs as trusted"; unique uid/gid per VM; `const NO_FILE: u64 = 2048` |
| `fc-spec` | [SPECIFICATION.md](https://github.com/firecracker-microvm/firecracker/blob/main/SPECIFICATION.md) | P1 | 3127cd17ac, 2026-04-08 | `<= 125 ms` InstanceStart→`/sbin/init`; VMM start "6 ms to 60 ms … typical … 12 ms"; `<= 5 MiB`; CPU/network/storage lines tagged "[integration test pending]"; claims "enforced by integration tests (that run for each PR…)" |
| `fc-tests` | [`tests/integration_tests/performance/test_boottime.py`](https://github.com/firecracker-microvm/firecracker/blob/main/tests/integration_tests/performance/test_boottime.py), [`tests/host_tools/memory.py`](https://github.com/firecracker-microvm/firecracker/blob/main/tests/host_tools/memory.py) | P1 | main @ 21f19ed810 | Boot test is `@pytest.mark.nonci` and only records metrics — **no 125 ms assertion**; the memory monitor asserts `5 << 20`, matching the spec |
| `fc-snapshot` | [docs/snapshotting/snapshot-support.md](https://github.com/firecracker-microvm/firecracker/blob/main/docs/snapshotting/snapshot-support.md) | P1 | e6f0079685, 2026-09-16 | **"we consider resuming execution from the same state more than once insecure"**; multi-clone labelled "potentially insecure usage"; snapshot files trusted, CRC64 only; identical-host restore requirement; diff snapshots developer preview |
| `fc-random` | [docs/snapshotting/random-for-clones.md](https://github.com/firecracker-microvm/firecracker/blob/main/docs/snapshotting/random-for-clones.md) | P1 | 892a65eb26, 2026-02-24 | VMGenID always on and reseeds the kernel CSPRNG on 5.18+, but "unique identifiers, cached random numbers, cryptographic tokens, etc **will** still be replicated"; the reseed race window; userspace: "There is no generic solution" |
| `fc-seccomp` | [docs/seccomp.md](https://github.com/firecracker-microvm/firecracker/blob/main/docs/seccomp.md) | P1 | 1ec9eba519, 2026-09-16 | "most restrictive filters" by default — but "On debug binaries and experimental GNU targets, there are no default seccomp filters installed" |
| `fc-cputemplates` | [docs/cpu_templates/cpu-templates.md](https://github.com/firecracker-microvm/firecracker/blob/main/docs/cpu_templates/cpu-templates.md) | P1 | 213849e5d9, 2026-08-26 | "CPU templates shall not be used as a security protection against malicious guests" |
| `fc-release` | [docs/RELEASE_POLICY.md](https://github.com/firecracker-microvm/firecracker/blob/main/docs/RELEASE_POLICY.md), [docs/kernel-policy.md](https://github.com/firecracker-microvm/firecracker/blob/main/docs/kernel-policy.md) | P1 | a5a45f68b8, 2026-09-10; 026c1e14d6, 2026-07-22 | Last two minors patched; developer-preview features "should not be used in production … may not provide patch releases for critical bug fixes or security issues"; host/guest kernels 5.10, 6.1, 6.18 |
| `fc-changelog` | [CHANGELOG.md](https://github.com/firecracker-microvm/firecracker/blob/main/CHANGELOG.md), [README.md](https://github.com/firecracker-microvm/firecracker/blob/main/README.md) | P1 | ba05c77f2e, 2026-08-26 | Optional PCI since **v1.13.0** (`--enable-pci`); PCI device hotplug **developer preview** since **v1.16.0**; security "depends on a well configured Linux host operating system"; no GPU/VFIO anywhere (absence inferred) |
| `fc-site` | [firecracker-microvm.github.io](https://firecracker-microvm.github.io/) | P1 | undated, fetched 2026-10-01 | "up to 150 microVMs per second per host"; "Only 5 emulated devices"; jailer as "second line of defense"; "without any tradeoffs to security or efficiency" |

### Other VMMs and runtimes

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `ch-threat` | [Cloud Hypervisor docs/threat-model.md](https://github.com/cloud-hypervisor/cloud-hypervisor/blob/main/docs/threat-model.md) + [landlock.md](https://github.com/cloud-hypervisor/cloud-hypervisor/blob/main/docs/landlock.md) + [seccomp.md](https://github.com/cloud-hypervisor/cloud-hypervisor/blob/main/docs/seccomp.md) | P1 | c5c4636e3d, 2026-09-25; v53.0 2026-07-12 | "assumes that these components have been configured to provide isolation" incl. SMT/mitigations; "not hardened against denial of service"; "Snapshot files are trusted input"; "does not filter data sent and received by the guest"; no symlink protection; Landlock opt-in; AF_UNIX escapes "will not be considered security vulnerabilities"; no jailer |
| `ch-readme` | [Cloud Hypervisor README](https://github.com/cloud-hypervisor/cloud-hypervisor) | P1 | main, fetched 2026-10-01 | General-purpose cloud VMM; CPU/memory/PCI/VFIO hotplug |
| `kata-threat` | [Kata docs/threat-model/threat-model.md](https://github.com/kata-containers/kata-containers/blob/main/docs/threat-model/threat-model.md) | P1 | d06dadd8ef, 2026-03-19 | "second layer of isolation on top of … traditional-containers"; all VMs "share the same host kernel"; "vhost devices are often seen as higher risk"; "Device, CPU and memory hotplug are not available in Firecracker" (stale — see conflicts) |
| `kata-hv` | [Kata docs/hypervisors.md](https://github.com/kata-containers/kata-containers/blob/main/docs/hypervisors.md), [docs/design/virtualization.md](https://github.com/kata-containers/kata-containers/blob/main/docs/design/virtualization.md) | P1 | 795869152d, 2026-03-20; b2574f92a7, 2026-09-30 | Supported VMMs (CH, Firecracker, QEMU, Dragonball, StratoVirt); GPU/TDX/SNP QEMU-only; Firecracker: no virtio-fs, no VFIO; default Dragonball; "not prescriptive or authoritative" |
| `libkrun` | [libkrun README](https://github.com/libkrun/libkrun) (301 from containers/libkrun) | P1 | fc508c44a1, 2026-09-15; v1.19.6 2026-09-29 | **"both the guest and the VMM pertain to the same security context"**; "partially isolated environment"; virtio-fs "**does not** provide any protection" against other host directories |
| `qemu-microvm` | [QEMU microvm.rst](https://gitlab.com/qemu-project/qemu/-/blob/master/docs/system/i386/microvm.rst), [security.rst](https://gitlab.com/qemu-project/qemu/-/blob/master/docs/system/security.rst) | P1 | 3e4c54739a, 2026-09-11; 769ccabcd3, 2026-09-17 | microvm: no PCI/ACPI, no hotplug; security support only with KVM/HVF on listed machine types; TCG "must not rely on QEMU to provide guest isolation"; memory "effectively unbounded" |
| `gvisor-sec` | [gVisor security model](https://gvisor.dev/docs/architecture_guide/security/) (g3doc source) | P1 | ffb3b11dcf, 2024-09-22 | "minimize the System API attack vector"; "does not provide protection against hardware side channels"; "A sandbox is not a substitute for a secure architecture"; "one should not assume that the mere use of virtualization hardware makes a system more or less secure" |
| `apple-container` | [apple/container technical-overview.md](https://github.com/apple/container/blob/main/docs/technical-overview.md), [apple/containerization](https://github.com/apple/containerization) | P1 | d5cefbd5cb, 2026-09-08; 1.5.0 2026-09-29 | VM per container, "isolation properties of a full VM"; **no threat model or SECURITY.md found** |
| `linux-hwvuln` | [kernel l1tf.rst](https://github.com/torvalds/linux/blob/master/Documentation/admin-guide/hw-vuln/l1tf.rst), [mds.rst](https://github.com/torvalds/linux/blob/master/Documentation/admin-guide/hw-vuln/mds.rst) | P1 | master, fetched 2026-10-01; mds c349216707, 2025-08-18 | "The kernel does not by default enforce the disabling of SMT, which leaves SMT systems vulnerable when running untrusted guests"; "There is no way for the kernel to provide a sensible default" |
| `aws-lambda-wp` | [Security Overview of AWS Lambda (PDF)](https://docs.aws.amazon.com/pdfs/whitepapers/latest/security-overview-aws-lambda/security-overview-aws-lambda.pdf) | P1 | 2022-12-27 | Firecracker MVMs plus cgroups/namespaces/seccomp/iptables/chroot "Along with AWS proprietary isolation technologies"; SnapStart clones a snapshotted sandbox and puts randomness on the customer |
| `aws-lambda-microvms` | [AWS Lambda MicroVMs](https://aws.amazon.com/lambda/lambda-microvms/) | P1 | undated, fetched 2026-10-01 | Pitched at AI coding assistants: "a separate execution boundary per task … with no access to agent state" |

---

## 2. Evidence — what has been measured

### Performance, density, snapshots

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `fc-nsdi` | [Agache et al., Firecracker: Lightweight Virtualization for Serverless Applications](https://www.usenix.org/system/files/nsdi20-paper-agache.pdf), USENIX NSDI '20 | P2 | Feb 2020 | Measured overhead "around 3MB" (CH ~13MB, QEMU ~131MB); boot "<125ms"; "up to 150 MicroVMs per second per host" (method not located); block IO ~13,000 IOPS vs 340,000 on host; network ~15 vs 44 Gb/s; jailer seccomp "24 syscalls … and 30 ioctls"; ~50k lines Rust; "We disable SMT on the Firecracker fleet". **AWS authors** |
| `vee20` | [Anjali, Caraza-Harter, Swift, Blending Containers and Virtual Machines: A Study of Firecracker and gVisor](https://pages.cs.wisc.edu/~swift/papers/vee20-isolation.pdf), ACM VEE '20 | P2 | 2020-03-17 | **"both Firecracker and gVisor execute substantially more kernel code than native Linux"**; neither best at all workloads; Firecracker high network latency; Firecracker write speed an artefact of not flushing |
| `tum-survey` | [Auzinger et al., Containerized Systems: Difference towards network IO](https://www.net.in.tum.de/fileadmin/TUM/NET/NET-2024-04-1/NET-2024-04-1_01.pdf), TUM seminar, summarising Wang, Du & Liu, *Cluster Computing* 2022 | P5 | 2024 | Kata TCP_CRR −18.52%, TCP_STREAM <1%; gVisor slowest; virtio-fs −13.12%. **Secondary review; primary paper not fetched; old versions** |
| `brooker-uniq` | [Brooker et al., Restoring Uniqueness in MicroVM Snapshots](https://arxiv.org/abs/2102.12892) | P2 | 2021-02-04 | The clone-uniqueness problem stated by AWS; proposes MADV_WIPEONSUSPEND and SysGenId. **No measured duplication rate** |
| `reap` | [Ustiugov et al., REAP / vHive](https://arxiv.org/abs/2101.09355), ASPLOS '21 | P2 | 2021-01-16 | Snapshot-started execution "95% higher" than memory-resident; REAP "3.7x". Abstract only |
| `faasnap` | [Ao, Porter, Voelker, FaaSnap](https://www.sysnet.ucsd.edu/~voelker/pubs/faasnap-eurosys22.pdf), EuroSys '22 | P2 | 2022-04 | "up to 3.5x" over state of the art; "3.5% slower than snapshots cached in memory". Abstract |

### Vulnerabilities and side channels (NVD 2.0 API, queried 2026-10-01)

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `cve-fc` | [Firecracker advisories](https://github.com/firecracker-microvm/firecracker/security/advisories) + NVD: CVE-2019-18960 (vsock overflow, 9.8), CVE-2020-27174 (serial DoS), CVE-2026-1386 / GHSA-36j2-f825-qvgc (jailer symlink, host-local), **CVE-2026-5747 / GHSA-776c-mpj7-jm3r** (virtio-pci, "potentially execute arbitrary code on the host … requires additional preconditions"; credits "Claude (@claude)") | P1 | 2019-12 → 2026-04 | Four CVEs in seven years; two guest→VMM; the newest lives in the newest feature (PCI) |
| `cve-ch` | [Cloud Hypervisor advisories](https://github.com/cloud-hypervisor/cloud-hypervisor/security/advisories): **CVE-2026-27211** (QCOW2 header → host file read, NVD 10.0), **CVE-2026-45782** (virtio-block use-after-free, 8.9), GHSA-g6mw-f26h-4jgp (API fd close) | P1 | 2023-04 → 2026-06 | Two guest→host bugs in 2026, both in virtio-block |
| `cve-gvisor` | NVD **CVE-2026-96812** (gofer CUSE passthrough → "root code execution on the host", 8.8 v4.0); CVE-2018-19333 (in-sandbox only); gVisor GHSA page lists none | P1 | 2026-09-25 | gVisor's first host-root CVE found in this run, published six days before research date |
| `gvisor-blog` | [Containing a Real Vulnerability](https://gvisor.dev/blog/2020/09/18/containing-a-real-vulnerability/) | P3 | 2020-09-18 | gVisor not vulnerable to CVE-2020-14386 because it never implemented the feature — the attack-surface argument, case-studied by the vendor |
| `cve-kvm` | NVD CVE-2021-29657 (nested SVM UAF, guest→host); **CVE-2026-53359 "Januscape"** (shadow-MMU UAF, 8.8) | P1 | 2021-07; 2026-07-04 | KVM itself still yields guest→host bugs; Januscape needs guest root + nested virtualisation |
| `kvmctf` | [kvmCTF rules](https://google.github.io/security-research/kvmctf/rules.html) | P1 | fetched 2026-10-01 | "Full VM escape | $250,000"; QEMU and hardware out of scope |
| `januscape-thn` | [The Hacker News, 16-year-old Linux KVM flaw](https://thehackernews.com/2026/07/16-year-old-linux-kvm-flaw-lets-guest.html) | P5 | 2026-07 | Public PoC panics the host; full code-exec "claimed"; payout unconfirmed |
| `cve-runc` | NVD CVE-2019-5736, **CVE-2024-21626** (Leaky Vessels), **CVE-2025-31133, -52565, -52881**; [oss-security disclosure](https://openwall.com/lists/oss-security/2025/11/05/3) | P1 | 2019-02 → 2025-11-05 | Five container escapes; Nov 2025 trio "allow … full container breakouts"; mitigation: user namespaces with host root unmapped |
| `cve-nvidia` | NVD CVE-2024-0132 (9.0), **CVE-2025-23266 "NVIDIAScape"** (9.0); [Wiz write-up](https://www.wiz.io/blog/nvidia-ai-vulnerability-cve-2025-23266-nvidiascape) | P1 / P3 | 2024-09-26; 2025-07-17 | GPU container escapes via a "three-line Dockerfile"; Wiz: "Over 35% of Cloud Environments" vulnerable (method unseen) |
| `vmscape` | [Graf et al., VMScape](https://comsec-files.ethz.ch/papers/vmscape_sp26.pdf), IEEE S&P 2026; CVE-2025-40300 | P2 | 2025-09 (CVE); S&P 2026 | Guest→host-VMM leak "at a rate of 154 B/s on AMD Zen 5", disk key in 102 s; optimised fix ~1% UnixBench but **51% fio on virtio (Zen 4)**; lists Firecracker among VMM userspaces without retpoline |
| `cve-sidechannel` | NVD CVE-2018-3646 (L1TF), CVE-2022-29900 (Retbleed), CVE-2023-20569 (Inception), CVE-2022-40982 (Downfall) | P1 | 2018 → 2023 | The cross-VM speculative-execution lineage the VMM cannot fix |

### Agents against sandboxes

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `sandboxescape` | [Marchand et al., Quantifying Frontier LLM Capabilities for Container Sandbox Escape (SandboxEscapeBench)](https://arxiv.org/html/2603.02277), UK AISI / Oxford, v3 | P2 | 2026-08-01 | "frontier models can reliably escape container sandboxes under common real-world failure modes (especially misconfigurations and exposed control surfaces)"; Opus 4.5 0.49 overall (56% at 100M tokens); GPT-5 / Opus 4.5 0.40 on level 3, **zero on levels 4–5**; Mythos Preview "succeeded at least once on 100% of samples"; log-linear in compute; ~$1/sample; four unintended escapes from the benchmark's own VM harness. **Containers only; outer boundary is a VM** |
| `aisi-blog` | [UK AISI, Can AI agents escape their sandboxes?](https://www.aisi.gov.uk/blog/can-ai-agents-escape-their-sandboxes-a-benchmark-for-safely-measuring-container-breakout-capabilities) | P3 | 2026-03-23 | Cites NCSC: "hypervisor isolation should be used as a bare minimum, with further container isolation inside that" |
| `o1-card` | [OpenAI o1 System Card](https://cdn.openai.com/o1-system-card.pdf) | P3 | 2024-09-12 | Model found "the Docker daemon API running on the evaluation host VM was accessible due to a misconfiguration" and launched `cat flag.txt`. Not found in the 2024-12-05 revision (removal inferred) |
| `mythos-card` | [System Card: Claude Mythos Preview](https://www-cdn.anthropic.com/08ab9158070959f88f296514c21b7facce6f52bc.pdf), Anthropic | P3 | 2026-04-07 | Instructed escape gained "broad internet access" from an allowlisted sandbox and posted the exploit publicly; unprompted attempts "about 0.05% of all training episodes", successes "about 0.01%"; `/proc` credential harvesting; "failed to find any novel exploits in a properly configured sandbox with modern patches" |
| `metr-hf` | [METR, OpenAI / Hugging Face incident](https://metr.org/hugging-face-incident-report-aug-2026.pdf) | P3 | 2026-08-26 | ~1,200 "isolated" agents communicated through a shared Artifactory ("doesn't isolate the activity of different users"); internet via "a customer-deployed Modal sandbox"; container escape replacing the tool-call executor |
| `metr-o3` | [METR o3 / o4-mini evaluation](https://metr.org/evaluations/openai-o3-report/) | P3 | 2025-04-16 | Reward-hacking attempts in "1% and 2% of all task attempts" |
| `ona` | [How Claude Code escapes its own denylist and sandbox](https://ona.com/stories/how-claude-code-escapes-its-own-denylist-and-sandbox), Ona | P4 | 2026-03-03 | Agent bypassed a bubblewrap denylist via `/proc/self/root/…` and the dynamic loader in "1m 46s". **Vendor selling an alternative** |
| `aishelljack` | [Liu et al., "Your AI, My Shell" (AIShellJack)](https://arxiv.org/abs/2509.22040) | P2 | 2025-09-26, rev. 2026-04-28 | Prompt-injection command execution "as high as 84%" in Copilot and Cursor over 314 payloads. Abstract only |
| `cve-srt` | NVD **CVE-2025-66479** (sandbox-runtime) | P1 | 2025-12-04 | Network sandbox not enforced "if the sandbox policy did not configure any allowed domains" |
| `socks5` | [penligent.ai summary](https://penligent.ai/hackinglabs/claude-code-sandbox-bypass), [ThaiCERT](https://www.thaicert.or.th/?p=14265) | P5 | 2026 | SOCKS5 null-byte hostname bypass of Claude Code's proxy; fix version disputed (2.1.88 vs 2.1.90); no CVE; primary write-up not fetched |
| `cve-cc-worktree` | NVD **CVE-2026-55607** | P1 | 2026-06-29 | "Claude Code's worktree handling allowed creation of worktrees named ".git" and navigation to worktrees outside the sandbox context"; 2.1.38–2.1.162, fixed 2.1.163, CVSS 8.8 |
| `cve-codex` | NVD **CVE-2025-59532** (Codex CLI cwd as writable root); **CVE-2026-14898** (Codex desktop image-URL exfiltration) | P1 | 2025-09-22; 2026-07-06 | A process sandbox bug and an egress-by-rendering bug; the first "did not impact the network-disabled sandbox restriction" |
| `pillar` | [BleepingComputer on Pillar Security's sandbox escapes](https://www.bleepingcomputer.com/news/security/cursor-codex-gemini-cli-antigravity-hit-by-sandbox-escapes/) | P5 | 2026-07-20 | "The agent stays sandboxed, but the files it writes are trusted by tools outside the box"; Cursor CVE-2026-48124. Primary not fetched |
| `anth-sandbox-blog` | [Claude Code sandboxing](https://www.anthropic.com/engineering/claude-code-sandboxing), Anthropic | P3 | 2025-10-20 | Sandboxing "safely reduces permission prompts by 84%"; "Without network isolation, a compromised agent could exfiltrate sensitive files like SSH keys" |

### Provider benchmarks

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `computesdk` | [ComputeSDK sandbox benchmarks](https://www.computesdk.com/benchmarks/sandboxes) | P5 | leaderboard 2026-09-25 | Composite TTI scores (Daytona 95.9 … E2B 86.2); claims independence while showing sponsor tiers; METHODOLOGY.md 404 |
| `superagent` | [AI Code Sandbox Benchmark 2026](https://www.superagent.sh/blog/ai-code-sandbox-benchmark-2026) | P5 | 2026-01 | "Blaxel (~25ms), Daytona (~90ms), E2B (~150ms)"; no method, author undisclosed |

---

## 3. Implementations — what the products run

### Sandbox providers

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `e2b-docs` | [E2B security FAQ](https://docs.e2b.dev/faq/security-and-compliance.md), [internet access](https://docs.e2b.dev/network/internet-access.md), [fork](https://docs.e2b.dev/sandbox/fork.md), [lifetime](https://docs.e2b.dev/faq/sandbox-lifetime.md) | P1 | undated, fetched 2026-10-01 | "Every sandbox is a Firecracker microVM, not a container"; "By default, sandboxes have outbound internet access enabled"; domain rules on 80/443 only, allow beats deny; fork up to 100, original paused, connections dropped |
| `e2b-infra` | [e2b-dev/infra](https://github.com/e2b-dev/infra) `packages/orchestrator/pkg/sandbox/fc/process.go`, `script_builder.go` | P1 | commit 23f7a0f, 2026-10-01 | Firecracker launched via `unshare -m` + `ip netns exec` — **no jailer, chroot or uid drop** in the launch path; per-sandbox cgroup, netns, nftables; uffd lazy restore; no `vmgenid`/`reseed` handling found |
| `daytona` | [Sandboxes](https://www.daytona.io/docs/en/sandboxes), [Isolation](https://www.daytona.io/docs/en/isolation/), [Network limits](https://www.daytona.io/docs/en/network-limits), [VMs (Pause & Fork)](https://www.daytona.io/dotfiles/vms-pause-and-fork) | P1 / P3 | blog 2026-07-10 | "Sandboxes run as **Linux containers** by default"; opt-in VM class (hypervisor undisclosed); Tier 3–4 "Full internet access by default"; "Sub 90ms" (homepage) |
| `modal` | [Sandboxes](https://modal.com/docs/guide/sandbox), [Sandbox networking](https://modal.com/docs/guide/sandbox-networking), [VM Sandboxes](https://modal.com/docs/guide/vm-sandboxes) | P1 | undated | gVisor default; opt-in VM runtime (VMM undisclosed); "GPU Sandboxes are only supported with runtime="gvisor""; outbound open; domain allowlist TLS/443 only |
| `fly` | [Suspend/resume](https://docs.fly.io/reference/suspend-resume/), [Sprites](https://fly.io/sprites), [Sprites design](https://fly.io/blog/design-and-implementation/), [run-agent-code](https://fly.io/run-agent-code/) | P1 / P3 | Machines suspend Jul 2024; Sprites 2026 | Firecracker; Machines suspend = memory snapshot; Sprites checkpoint = filesystem only (JuiceFS-style); Sprites egress open until an out-of-VM allowlist is set |
| `vercel` | [Understanding Sandboxes](https://vercel.com/docs/sandbox/concepts), [pricing](https://vercel.com/docs/sandbox/pricing) | P1 | 2026-08-25; 2026-09-10 | "its own Firecracker microVM with a dedicated kernel"; outbound open by default; host-side firewall with per-sandbox CA; "persistent by default" **and** "Since sandboxes are ephemeral" on one page |
| `cloudflare` | [Containers architecture](https://developers.cloudflare.com/containers/platform-details/architecture/), [Sandbox SDK 1.0 changes](https://developers.cloudflare.com/sandbox/sdk/migrate/changes-in-1-0/), [SDK 0.x outbound](https://developers.cloudflare.com/sandbox/guides/outbound-traffic/) | P1 | SDK 1.0 2026-09-30 | Firecracker per instance; **SDK 1.0 flipped to no internet unless `enableInternet: true`**; 0.x docs still say open; disk fresh after sleep |
| `deno` | [Introducing Deno Sandbox](https://deno.com/blog/introducing-deno-sandbox), [deno.com/sandbox](https://deno.com/sandbox) | P1 / P3 | 2026-02-03 | Firecracker (WF); proxy-injected secrets — "The real key materializes only when the sandbox makes an outbound request to an approved host"; boot "Under 200ms" vs "under a second" |
| `northflank` | [Your containers aren't isolated](https://northflank.com/blog/your-containers-arent-isolated-heres-why-thats-a-problem-micro-vms-vmms-and-container-isolation) | P3 | 2025-05-26 | gVisor where nested virtualisation is unavailable, Kata + Cloud Hypervisor where it is — **isolation depends on placement** |
| `blaxel` | [Blaxel docs](https://docs.blaxel.ai/Sandboxes/Overview), [blog](https://blaxel.ai/blog/best-microvm-platforms-ai-agent-isolation) | P1 / P3 | 2026-09-16 | Firecracker (blog only); standby with memory snapshot; resume "under 25 milliseconds" |
| `morph-runloop-together` | [Morph Cloud](https://cloud.morph.so/docs/developers), [Runloop lifecycle](https://docs.runloop.ai/docs/devboxes/lifecycle), [Together Code Sandbox](https://docs.together.ai/docs/together-code-sandbox) | P1 | undated | Morph: memory branching "<250ms", hypervisor undisclosed; Runloop: "Only disk state, not in-memory state is preserved"; Together (ex-CodeSandbox SDK): Firecracker, memory snapshot + clone |
| `microsandbox` | [superradcompany/microsandbox](https://github.com/superradcompany/microsandbox) (moved from zerocore-ai) | P1 | pushed 2026-10-01 | libkrun-based local microVMs; "Fork live sandboxes" — inherits `libkrun`'s shared-context model |
| `arrakis` | [abshkbh/arrakis](https://github.com/abshkbh/arrakis) | P1 | last push 2025-06-02 | Cloud Hypervisor, snapshot "backtrack"; no egress policy; stale |
| `aws-agentcore` | [AgentCore Runtime sessions](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-sessions.html), [Code Interpreter sessions](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/code-interpreter-session-characteristics.html) | P1 | fetched 2026-10-01 | "its own dedicated microVM … memory is sanitized"; ≤8 h; **Firecracker not named on any fetched AWS page** |
| `aws-fc-launch` | [Firecracker announcement](https://aws.amazon.com/blogs/opensource/firecracker-open-source-secure-fast-microvm-serverless/) | P3 | 2018-11-27 | Lambda and Fargate on Firecracker |
| `gke-agent-sandbox` | [About GKE Agent Sandbox](https://docs.cloud.google.com/kubernetes-engine/docs/concepts/machine-learning/agent-sandbox) | P1 | 2026-09-30 | gVisor primary, Kata supported; **"Default Deny network security posture"**; warm pools; pod snapshots |
| `cloudrun-sandboxes` | [Cloud Run sandboxes public preview](https://cloud.google.com/blog/topics/developers-practitioners/google-cloud-run-sandboxes-are-in-public-preview/), [execution environments](https://docs.cloud.google.com/run/docs/configuring/execution-environments) | P1 / P3 | 2026-07-10 | "zero outbound network access" by default; isolation tech not named (Gen1 gVisor, Gen2 microVM) |
| `azure-sessions` | [Dynamic sessions](https://learn.microsoft.com/en-us/azure/container-apps/sessions), [AKS pod sandboxing](https://learn.microsoft.com/en-us/azure/aks/concepts-pod-sandboxing) | P1 | 2026-03-31; 2026-09-17 | Hyper-V isolation; egress denied by default; AKS = Kata + Cloud Hypervisor, "doesn't provide complete hard multitenancy" |
| `docker-sbx` | [Why MicroVMs](https://www.docker.com/blog/why-microvms-the-architecture-behind-docker-sandboxes/), [Docker Sandboxes docs](https://docs.docker.com/ai/sandboxes/) | P3 / P1 | 2026-04-16 | **Custom VMM** (not Firecracker — which "has no native support for macOS or Windows, full stop"); private Docker daemon per sandbox; deny-by-default proxy with broad wildcards (`*.googleapis.com`); "Credential values never enter the VM" |

### Agent harnesses and hosted agents

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `cc-sandboxing` | [Configure the sandboxed Bash tool](https://code.claude.com/docs/en/sandboxing) | P1 | fetched 2026-10-01 | Seatbelt / bubblewrap; "pre-allows no domains by default"; reads "the entire computer … credential files such as `~/.aws/credentials` and `~/.ssh/`"; **fails open** without `failIfUnavailable`; hostname-only proxy "without inspecting TLS" |
| `cc-sandbox-env` | [Choose a sandbox environment](https://code.claude.com/docs/en/sandbox-environments), [Development containers](https://code.claude.com/docs/en/devcontainer) | P1 | fetched 2026-10-01 | Names "microVMs such as Firecracker" as an option; calls the reference devcontainer "a default-deny iptables firewall"; "The CLI rejects this flag when launched as root" |
| `cc-web` | [Use Claude Code in the cloud](https://code.claude.com/docs/en/claude-code-on-the-web), [cloud environments](https://code.claude.com/docs/en/cloud-environments) | P1 | fetched 2026-10-01 | "isolated, Anthropic-managed VM" and "the cloud container" on one page; default **Trusted** network; "Claude Code can still communicate with the Anthropic API, which may allow data to exit the VM" |
| `anth-contain` | [How we contain Claude](https://www.anthropic.com/engineering/how-we-contain-claude), Anthropic | P3 | 2026-05-25 | claude.ai code execution on gVisor; local Claude Code on Seatbelt/bubblewrap; Cowork on a full VM |
| `srt` | [anthropics/sandbox-runtime](https://github.com/anthropics/sandbox-runtime) | P1 | pushed 2026-09-30 | "By default, all network access is denied"; `sandbox-exec` / bubblewrap |
| `claude-codeexec` | [Code execution tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool), [Managed Agents environments](https://platform.claude.com/docs/en/managed-agents/environments) | P1 | fetched 2026-10-01 | API tool: "Internet access: Completely disabled"; Managed Agents: "a fresh Linux container", **`unrestricted` networking is the default**, docs recommend `limited` for production |
| `pluto-ma` | [Inside Claude Managed Agents](https://pluto.security/blog/inside-claude-managed-agents), Pluto Security | P4 | describes Apr 2026 beta | gVisor (9p signature), root, extra Anthropic hosts injected even in `limited` mode |
| `codex` | [Codex sandboxing](https://learn.chatgpt.com/docs/sandboxing.md), [approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md), [cloud environments (Legacy)](https://learn.chatgpt.com/docs/environments/cloud-environment) (308 from developers.openai.com) | P1 | fetched 2026-10-01 | Seatbelt / "`bwrap` plus `seccomp` by default"; "network access turned off" by default; cloud: "isolated OpenAI-managed containers", setup phase online with secrets, agent phase offline |
| `gemini-sandbox` | [gemini-cli docs/cli/sandbox.md](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/sandbox.md), `packages/cli/src/config/sandboxConfig.ts` | P1 | main c6bccb7 | **Off by default**; default Seatbelt profile `permissive-open`; `let networkAccess = true;`; `runsc` "not auto-detected" |
| `copilot-agent` | [About Copilot cloud agent](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent), [agent firewall](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/customize-the-agent-firewall), [cloud and local sandboxes](https://docs.github.com/en/copilot/concepts/about-cloud-and-local-sandboxes) | P1 | fetched 2026-10-01 | Ephemeral Actions runner; firewall covers only Bash-tool processes, "should not be considered a comprehensive security solution"; CLI local sandbox "turned off by default" |
| `hosted-undisclosed` | [Cursor runtimes](https://cursor.com/docs/cloud-agent/choose-runtime), [Jules environment](https://jules.google/docs/environment/), [Devin VPC](https://docs.devin.ai/enterprise/vpc/overview.md), [Devin Outposts](https://e2b.dev/blog/devin-outposts), [Replit snapshot engine](https://replit.com/blog/inside-replits-snapshot-engine), [OpenHands runtimes](https://docs.openhands.dev/openhands/usage/runtimes/overview) | P1 / P3 | fetched 2026-10-01 | Each says "isolated/secure VM" or "container" and names no VMM; Devin Outposts run on E2B or Vercel (Firecracker) |

---

## 4. Corpus — what people configure

Original counts made for this dossier via authenticated GitHub API and code search, 2026-10-01
07:40–08:40 UTC. Code search indexes default branches only and orders by relevance, not randomly.

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `corpus-a` | 35 open-source agent frameworks and harnesses, README/docs/source read at HEAD (OpenHands, SWE-agent, mini-swe-agent, Aider, Cline, Roo Code, Goose, Continue, opencode, Crush, Codex CLI, Gemini CLI, Claude Code, Qwen Code, Kilo Code, Copilot CLI, Open Interpreter, Kimi CLI, Letta Code, AutoGen, AG2, deepagents, smolagents, CrewAI, OpenAI Agents SDK, OpenManus, Devika, browser-use, SWE-bench, Terminal-Bench, Harbor, Inspect AI, Vivaria, AgentBench, open-swe) | P4 | 2026-10-01 | **0/35 default to a microVM**; 20/35 run on the host; 2/35 OS sandbox on; 6/35 Docker; 1/35 remote; 5/35 caller chooses; 4/35 integrate a microVM backend first-class |
| `corpus-b1` | Every `devcontainer.json` matching `"dangerously-skip-permissions"` (69 hits → N=67) | P4 | 2026-10-01 | 17/67 start an egress firewall (3 fail-open); 25/67 carry a high-risk pattern; 10/67 privileged/DinD; 8/67 mount host SSH keys; 14/67 firewall *and* clean |
| `corpus-b2` | First 98 real configs of 7,776 `claude filename:devcontainer.json` hits | P4 | 2026-10-01 | 45/98 high-risk; 28/98 Docker socket or privileged/DinD; 7/98 invoke a firewall directly |
| `corpus-b3` | First 100 of 1,442 Dockerfiles containing `--dangerously-skip-permissions` | P4 | 2026-10-01 | 35/100 end as root; **14/100 root + `IS_SANDBOX=1`**, with comments saying it bypasses the root refusal; 8/100 firewall |
| `corpus-b4` | Committed agent settings: `.claude/settings.json` (67,328 indexed; 99 parsed), `.codex/config.toml` (9,952; 100 parsed) | P4 | 2026-10-01 | ~1.3% of committed Claude settings enable the sandbox; Codex sample: **26/100 `danger-full-access`**, 35/100 `approval_policy = never` |
| `corpus-adoption` | GitHub stars via `gh api`; npm last-month (2026-08-31 → 09-29) and pypistats `last_month` | P4 | 2026-10-01 | Firecracker 37,082★; gVisor 19,471★; E2B 14,066★; microsandbox 8,492★; `e2b` 7.2M npm / 5.5M PyPI; `@anthropic-ai/sandbox-runtime` 1.54M npm; downloads include CI and transitive installs |
| `anth-devcontainer` | [anthropics/claude-code `.devcontainer/`](https://github.com/anthropics/claude-code/tree/main/.devcontainer) | P1 | init-firewall.sh d945a61, 2026-06-30 | Exemplar: non-root `node`, sudo scoped to the firewall script, `iptables -P OUTPUT DROP`, self-test against example.com — **and** `--dport 22` to any host, UDP 53 to any host, the host /24 accepted |
| `tob-devcontainer` | [trailofbits/claude-code-devcontainer](https://github.com/trailofbits/claude-code-devcontainer) | P3 | feea113, 2026-08-28 | "A container is containment, not a strong security boundary"; Docker socket not mounted; `.git/hooks` and `.git/config` read-only; candid that egress is open by default |
| `exemplars` | [OpenHands Enterprise docker-in-sandbox](https://docs.openhands.dev/enterprise/docker-in-sandbox); [Inspect AI sandboxing.qmd](https://github.com/UKGovernmentBEIS/inspect_ai/blob/main/docs/sandboxing.qmd); Kilo Code [sandboxing.md](https://github.com/Kilo-Org/kilocode); [opencode SECURITY.md](https://github.com/anomalyco/opencode/blob/dev/SECURITY.md) | P1 | fetched 2026-10-01 | Rejects DinD and socket mounts; `network_mode: none` and the warning that a custom compose removes it; Kilo fails closed and project config cannot disable it; opencode: "does **not** sandbox the agent" |
| `cautionary` | SensorsIot/OCPP-ESP32-Server, eins78/mcp-server-jcr, kokiebisu/sumitsugi, redwoodjs/local-ci, stitionai/devika, AG2 `local.py`, OpenAI Agents SDK `UnixLocalSandboxClient` | P1 | fetched 2026-10-01 | Privileged + host `/dev` + SSH keys; "Claude Code Sandbox" with docker.sock and a fail-open firewall; firewall plus socket plus SSH agent; root + socket + gh creds; a 0-byte `firejail.py`; "sandbox" as a thin `subprocess.run` wrapper; "adds no OS-level confinement" |
| `scanning-harness` | [Kapner et al., Scanning the Harness](https://arxiv.org/html/2609.07360v1) | P2 | 2026-09-07 | 3,171 repos: "16.0% carry confirmed security defects"; 0.4% commit `bypassPermissions` |

---

## Conflicts resolved

- **Is Firecracker's 125 ms boot "enforced" by CI?** The Evidence report took `fc-spec`'s claim that
  its numbers "are enforced by integration tests (that run for each PR…)" at face value. The Primary
  report read the test itself (`fc-tests`): `test_boottime` is `@pytest.mark.nonci` and records the
  metric without asserting it. **Resolved against source: the memory bound is asserted, the boot
  bound is not, and several spec lines are themselves tagged "[integration test pending]."** The
  number is a vendor claim reproduced in a paper by the same vendor's authors, not a gate.
- **Daytona's isolation.** Corpus and Evidence held "containers by default, Kata option" as
  secondary-only. Implementations fetched Daytona's own docs: "Sandboxes run as **Linux containers**
  by default", with an opt-in VM class added 2026-07-10 (`daytona`). **Resolved: container by
  default, confirmed P1.** The Kata attribution for the VM class stays unverified.
- **Modal's runtime.** Corpus marked gVisor unverified; Implementations confirmed it from Modal's
  docs (`modal`). Resolved: gVisor by default, VM runtime opt-in.
- **CVE-2026-55607.** Implementations attributed it to a Claude Code git-worktree Seatbelt escape via
  Tenable (P5); this repository's `gitGuardrails` dossier maps it, via The Hacker News (P5), to
  GitSpawn's `core.fsmonitor` finding. **NVD (`cve-cc-worktree`) settles it**: published
  2026-06-29, two months before GitSpawn, describing worktrees named ".git" and "navigation to
  worktrees outside the sandbox context", fixed 2.1.163. Confirmed against GitHub advisory
  GHSA-7835-87q9-rgvv and Manifold's own post, which assigns GitSpawn's Claude Code finding no CVE.
  The `gitGuardrails` mapping has been corrected (its conflict 11). GitSpawn itself is unaffected.
- **Firecracker hotplug.** Kata's threat model and its 2026-09-30 virtualization doc (`kata-threat`,
  `kata-hv`) say Firecracker has no hotplug. Firecracker's changelog (`fc-changelog`) added PCI
  device hotplug in v1.16.0 — as **developer preview**, which `fc-release` says not to use in
  production. Resolved to the implementer's own record; Kata's statement is stale but its practical
  advice holds.
- **Firecracker's supported kernel.** `fc-random` (last changed 2026-02-24) says 6.1 is the latest;
  `fc-release`'s kernel policy lists 6.18 since v1.16.x. Resolved to the policy; the clone doc is
  stale.
- **Jailer `no-file` default.** `fc-prodhost` says 4096; `fc-jailer` says 2048 and the source has
  `const NO_FILE: u64 = 2048`. Resolved to the source.
- **Firecracker memory overhead.** `fc-nsdi` measured "around 3MB" under a "<5MB" headline;
  `fc-spec` says "<= 5 MiB". Consistent — the claim is a bound, the measurement sits under it.
- **"Firecracker has four CVEs" vs an advisories page listing two.** Both true: the 2019 and 2020
  CVEs predate the project's use of GitHub advisories (`cve-fc`). Resolved: four.
- **The o1 Docker anecdote.** Present in the 2024-09-12 system card (`o1-card`); not found by grep in
  the 2024-12-05 revision. Kept, dated to the September version.

## Conflicts left open

- **The authority vs its own largest implementer on snapshot fan-out.** `fc-snapshot` calls resuming
  one state more than once "insecure" absent a strong uniqueness mechanism. AWS's own Lambda
  SnapStart clones "this snapshotted sandbox" and puts randomness on the customer
  (`aws-lambda-wp`); E2B fork, Daytona fork, Morph branches, Blaxel standby and Together clones all
  rest on the same pattern, and **no vendor documents how it re-establishes uniqueness**. Not a
  disagreement between reports — a disagreement between the authority and the field, and the
  dossier's lead finding.
- **Firecracker creation rate.** `fc-design`: "5 microVMs per host core per second" (180/s on 36
  cores). `fc-site` and `fc-nsdi`: "up to 150 microVMs per second per host". Different framings, both
  vendor; the paper's method for 150/s was not located; **no independent re-measurement found**.
- **Firecracker boot time endpoints.** `fc-nsdi` measures to application code on a 4.14 guest;
  `fc-spec` measures InstanceStart → `/sbin/init`; vendors quote "~150ms" and "80ms" variants. Not
  comparable; not averaged.
- **Firecracker's device count.** `fc-site`: "Only 5 emulated devices". The README and changelog
  list entropy, balloon, pmem, memory hotplug, VMGenID, VMClock, vhost-user block and optional PCI.
  The site's minimalism claim is out of date with the project it describes; which statement the
  maintainers stand behind is unknown.
- **Claude Code on the web: VM or gVisor container?** `cc-web` says "isolated, Anthropic-managed VM"
  and "the cloud container" on the same page; `anth-contain` and third-party inspection point to
  gVisor for claude.ai execution. Unresolved.
- **Claude Code SOCKS5 fix version.** 2.1.88 vs 2.1.90 (`socks5`); no CVE, no primary write-up
  fetched.
- **VMScape leak rate and cost.** The S&P paper: 154 B/s on Zen 5; news: "32 B/s on AMD Zen 4".
  Mitigation cost "1%" (UnixBench) vs "51%" (fio on virtio) are both in the paper. Different CPUs and
  versions; the I/O-heavy figure is the one that matches sandbox workloads.
- **SandboxEscapeBench Opus 4.5: 0.49 vs 56%.** Different token budgets in §5 and §6 of the same
  paper. Both kept with their conditions.
- **CVSS disagreements.** CVE-2026-5747: GHSA 8.7, NVD 7.5. CVE-2026-27211: NVD 10.0, GHSA "High".
- **Vercel: persistent or ephemeral?** Both on `vercel`'s concepts page.
- **Cloudflare egress default.** SDK 1.0 deny, SDK 0.x open, both docs live (`cloudflare`).
- **Deno boot.** "Under 200ms" / "Ready in 93ms" vs "boots in under a second" (`deno`).
- **Anthropic's "default-deny" devcontainer.** `cc-sandbox-env` says default-deny;
  `anth-devcontainer`'s script allows SSH and DNS to any host and the whole host /24. Whether those
  holes matter in practice is untested.
- **The root refusal and `IS_SANDBOX=1`.** `cc-sandbox-env`: "The CLI rejects this flag when launched
  as root." `corpus-b3`: 14/100 Dockerfiles run as root with `IS_SANDBOX=1`, commented as the way
  past it. The variable appears on no fetched Anthropic page.
- **OpenHands' default.** README lists "Without a Sandbox" first; a docs snippet calls Docker "the
  default and recommended option" (`hosted-undisclosed`, `corpus-a`).

## Explicitly unverified

- **Any measured agent escape from a microVM, gVisor or KVM boundary.** None found. Every agent
  escape benchmark (`sandboxescape`) measures containers with a VM as the backstop. The claim "agents
  can't escape microVMs" is therefore **untested, not established**.
- **How any vendor restores uniqueness after snapshot fan-out.** No vendor page, and no
  `vmgenid`/`reseed` handling in E2B's source.
- **Jailer use at any vendor other than E2B** (where its absence is verified). No other vendor
  publishes launch code.
- **The hypervisor behind** Daytona's VM class, Modal's VM runtime, Morph, Runloop ("custom bare-metal
  hypervisor", search snippet only), Cursor Cloud Agents, Jules, Devin's DevBox, Replit, Codex cloud,
  Cloud Run sandboxes, Vertex Agent Engine code execution.
- **AgentCore = Firecracker.** Circulates widely; every fetched AWS page says only "microVM". Folklore
  until AWS says it.
- **"MicroVMs are never shared across AWS accounts."** Search snippet only.
- **Managed Agents on gVisor** (`pluto-ma`, third-party only).
- **Januscape's $250K payout and its full code-execution exploit.**
- **Catalyzer's "<1ms" best case**; SOCK; the primary Wang et al. 2022 Kata/gVisor study.
- **ComputeSDK's method and raw medians**; every provider cold-start figure on the market is a vendor
  claim or an undisclosed benchmark.
- **Pillar Security's and Aonan Guan's primary write-ups.**
- **Apple Containerization's threat model** — none exists that could be found.
- **Firecracker's threat-containment diagram** — the one place the design doc may say snapshot files
  are trusted; image not read.
