# MicroVMs rulebook

Checkable rules for isolating coding agents. Each rule states its test (how a third party checks
compliance), its evidence tier, and its source keys (see [sources.md](sources.md)).

Tiers: **P1** primary spec, vendor docs, advisory or source · **P2** peer-reviewed or preprint
research · **P3** vendor engineering blog / industry research with method · **P4** practitioner
measurement (including this dossier's own counts) · **P5** opinion, anecdote, news, unverified.

Use it two ways: as a design checklist when building or buying agent sandboxes, and as a review gate.
A setup that fails R1, R9, R15 or R22 is not a sandbox, whatever it runs on.

---

## A. Choosing the boundary

**R1. Run unattended or untrusted-input agents behind hardware virtualisation.** Containers break
routinely, and agents reliably escape misconfigured ones. Test: is there a KVM/Hyper-V/HVF boundary
between the agent's code and any host holding other tenants' data or your credentials? **P2**
`sandboxescape` · **P3** `aisi-blog` · **P1** `cve-runc`, `cve-nvidia`

**R2. Don't call a process sandbox, gVisor or a container a microVM.** They are different
boundaries with different failure histories. Test: does every document describing the setup name the
actual technology? **P1** `gvisor-sec`, `cc-sandboxing`, `modal`, `daytona`

**R3. Find out what the provider actually runs.** "Secure VM" with no VMM named is not an answer, and
labels diverge from technology across the market. Test: can you name the VMM or runtime, citing a
vendor page rather than a third party? **P1** `daytona`, `modal`, `cc-web`, `aws-agentcore` · **P3**
`northflank`

**R4. Assume a container when the vendor won't say.** Test: is any undisclosed runtime treated, in
your threat model, as sharing a kernel with other tenants? **P1** `hosted-undisclosed`, `codex`

**R5. Pin the isolation class.** Some providers choose gVisor or a VM by placement or tier. Test: is
the isolation class set explicitly in your config, not left to a default or to placement? **P3**
`northflank` · **P1** `daytona`, `modal`

**R6. Don't treat libkrun-based local microVMs as a multi-tenant boundary.** The guest shares the
VMM's security context. Test: is the libkrun VMM itself run inside an OS-level isolation context, and
are virtio-fs mounts limited to what the guest may read? **P1** `libkrun`, `microsandbox`

**R7. Layer a container inside the VM, not instead of it.** Test: if the agent escapes its container,
does it land in a disposable VM rather than on a shared host? **P3** `aisi-blog` · **P1**
`kata-threat`

**R8. Prefer the smallest device model that does the job.** New device code is where new escapes
appear. Test: are PCI, hotplug, vhost-user and passthrough off unless a named workload needs them?
**P1** `cve-fc`, `cve-ch`, `fc-release`

---

## B. Network egress

**R9. Deny egress by default.** It is the documented vector in every real agent incident, and the
VMM does no filtering. Test: from inside a fresh sandbox, does `curl https://example.com` fail?
**P1** `fc-design`, `ch-threat` · **P3** `mythos-card`, `metr-hf`

**R10. Enforce the allowlist outside the guest.** Test: can root inside the guest change or remove
the egress policy? It must not be able to. **P1** `fc-design`, `vercel`, `docker-sbx`, `fly`

**R11. Block the cloud metadata endpoint.** Test: does a request to 169.254.169.254 from the guest
time out? **P1** `fc-prodhost`

**R12. Know whether your filter reads hostnames or content.** SNI/hostname-only filtering permits
domain fronting. Test: can you state whether the proxy terminates TLS, and is that accepted in the
threat model? **P1** `cc-sandboxing`, `e2b-docs`, `modal`

**R13. Audit the default allowlist.** "Deny by default" can still ship broad wildcards, SSH to any
host, or DNS to any host. Test: does the allowlist contain any wildcard over a shared-tenant domain,
or any rule allowing port 22 or 53 to any destination? **P1** `docker-sbx`, `anth-devcontainer`

**R14. Count every side channel the harness leaves open.** Model API, MCP and setup-phase traffic
often bypass the firewall. Test: is each bypass listed, with the data it can carry? **P1** `cc-web`,
`copilot-agent`, `codex`

---

## C. Credentials and control surfaces

**R15. Never hand the sandbox its host's control surface.** No Docker socket, privileged mode, host
`/dev`, or Docker-in-Docker on the host daemon. Test: does the config contain `docker.sock`,
`--privileged`, `privileged: true`, or the docker-in-docker feature? **P1** `o1-card`,
`exemplars` · **P2** `sandboxescape` · **P4** `corpus-b1`, `corpus-b2`

**R16. Keep real credentials out of the VM.** Inject them at an egress proxy. Test: does `env`, a
filesystem search, or a `/proc/*/environ` read inside the guest reveal any long-lived secret? **P1**
`deno`, `docker-sbx`, `cc-web` · **P3** `mythos-card`

**R17. Don't mount host identity.** Test: are host `~/.ssh`, `~/.aws`, `~/.config/gh`,
`~/.kube`, `SSH_AUTH_SOCK` and `$HOME` absent from mounts and forwarded sockets? **P4**
`corpus-b1`, `corpus-b2`

**R18. Scope any credential that must enter the guest per task, and treat it as disclosed.** Test:
does each in-guest token have its own expiry and scope, and would revoking it touch nothing else?
**P3** `mythos-card` · **P1** `codex`

**R19. Isolate shared services per sandbox.** A package proxy or artifact store that doesn't
separate users becomes a channel between agents. Test: can sandbox A read anything sandbox B wrote
through any shared service? **P3** `metr-hf`

**R20. Don't run the agent as root to get past a safety check.** Test: does the image set a non-root
`USER`, and is `IS_SANDBOX` unset unless an outer VM boundary is documented? **P1**
`cc-sandbox-env` · **P4** `corpus-b3`

**R21. Make git hooks and config read-only inside the sandbox.** Files the agent writes may be run
by tools outside it. Test: are `.git/hooks` and `.git/config` read-only from inside? **P3**
`tob-devcontainer` · **P5** `pillar` · **P1** `cve-cc-worktree`

---

## D. Failing closed and verifying

**R22. Fail closed.** Test: if the sandbox or firewall cannot start, does the agent refuse to run?
Check `failIfUnavailable`, and look for any `|| echo` or `;` after the firewall step. **P1**
`cc-sandboxing`, `exemplars` · **P4** `corpus-b1`

**R23. Verify the firewall from inside, every start.** Test: does startup fail if a known-blocked
host is reachable, as Anthropic's reference script does? **P1** `anth-devcontainer`

**R24. Make the firewall survive restarts.** Test: after a container or VM restart, does R9's check
still fail closed? **P4** `corpus-b2`

**R25. Don't let project config weaken a global sandbox.** Test: can a repository-committed setting
disable or loosen a sandbox the user enabled globally? **P1** `exemplars`

**R26. Check that a "sandbox" exists.** Test: does the code path labelled sandbox apply any OS
isolation at all, or does it call `subprocess.run` or `exec` on the host? **P1** `cautionary`

---

## E. Snapshots and forks

**R27. Don't fork a snapshot that has seen a secret.** Firecracker calls multi-resume insecure, and
userspace state is replicated. Test: was the template taken before any login, token fetch, key
generation or seeded RNG? **P1** `fc-snapshot`, `fc-random` · **P2** `brooker-uniq`

**R28. Regenerate identity on restore.** Test: do two forks of one snapshot produce different SSH
host keys, machine IDs, session tokens and userspace RNG output? **P1** `fc-random`, `aws-lambda-wp`

**R29. Use a guest kernel that consumes VMGenID.** Test: is the guest on Linux 5.18 or later, so the
kernel CSPRNG reseeds on restore? **P1** `fc-random`

**R30. Treat snapshot files as secrets and as trusted input.** Test: are snapshots encrypted and
authenticated at rest, and only restored from a store the tenant cannot write? **P1** `fc-snapshot`,
`ch-threat`

**R31. Ask your provider how it handles fork uniqueness.** No vendor documents it. Test: do you have
a written answer from the provider? **P1** `e2b-docs`, `daytona`, `morph-runloop-together` · **P1**
`e2b-infra`

**R32. Know which kind of snapshot you have.** Memory, filesystem or image snapshots carry different
state. Test: can you state which one your provider's "snapshot" or "fork" is? **P1** `e2b-docs`,
`fly`, `vercel`, `morph-runloop-together`

---

## F. Running a VMM yourself

**R33. Use the jailer, or document why your constraints are equal or stricter.** Test: is Firecracker
launched via `jailer` with a unique uid/gid, or is there a written comparison? **P1** `fc-prodhost`,
`fc-jailer` · **P1** `e2b-infra` (counter-example)

**R34. One tenant per VMM process.** Test: does any Firecracker process ever serve workloads from
two tenants? **P1** `fc-prodhost`

**R35. Disable SMT and KSM on hosts that separate tenants.** Test: do
`/sys/devices/system/cpu/smt/control` and `/sys/kernel/mm/ksm/run` show off, and does
`spectre-meltdown-checker` pass? **P1** `fc-prodhost`, `linux-hwvuln` · **P2** `vmscape`

**R36. Patch host kernel, guest kernel and microcode on the distribution's cadence.** Test: is each
within its vendor's current advisory? **P1** `fc-prodhost`, `cve-kvm`

**R37. Release builds only.** Debug and GNU builds ship without seccomp. Test: is the deployed binary
a `*-linux-musl` release build, launched without `--no-seccomp`? **P1** `fc-seccomp`

**R38. No developer-preview features in production.** Test: are PCI hotplug, vhost-user block and
diff snapshots off? **P1** `fc-release`, `fc-changelog`

**R39. Bound everything the VMM won't.** Test: are memory, CPU, file descriptors, stdout/log size and
process lifetime capped by cgroups, rlimits and an overwatcher? Is the serial console off? Is swap
off? **P1** `fc-prodhost`, `ch-threat`, `qemu-microvm`

**R40. Don't rely on CPU templates for security.** Test: is no part of the threat model resting on a
CPU template? **P1** `fc-cputemplates`

---

## G. Reading the claims

**R41. Don't quote 125 ms as a guarantee.** The memory bound is CI-asserted; the boot bound is
recorded, not asserted. Test: is any SLA or plan built on a vendor boot figure without your own
measurement? **P1** `fc-spec`, `fc-tests` · **P2** `fc-nsdi`

**R42. Measure cold start yourself, end to end.** Provider benchmarks are vendor-run or undisclosed.
Test: is the number you rely on from your own measurement of API call → first command? **P5**
`computesdk`, `superagent`

**R43. Don't claim agents can't escape microVMs.** It is untested, not established. Test: does any
document cite a measurement of an agent against a microVM boundary? There is none to cite. **P2**
`sandboxescape`

**R44. Don't equate syscall or line counts with attack surface.** A microVM still exercises more host
kernel code than a container. Test: is the isolation argument based on reachable-surface reasoning,
not LOC? **P2** `vee20`, `fc-nsdi`

**R45. Budget for side-channel mitigation cost on I/O-heavy agents.** Test: has the VMScape
mitigation's cost been measured on your actual build/test workload? It ranges from ~1% to 51%.
**P2** `vmscape`

---

## Review checklist

An agent sandbox does not pass review until every one of these holds:

| # | Gate | Rules |
|---|---|---|
| 1 | Hardware-virtualisation boundary for unattended or untrusted work, with the technology named | R1, R2, R3 |
| 2 | Egress denied by default, enforced outside the guest, metadata blocked | R9, R10, R11 |
| 3 | Allowlist audited for wildcards and port 22/53-to-anywhere | R13 |
| 4 | No Docker socket, privileged mode or host identity mounts | R15, R17 |
| 5 | No long-lived secret readable inside the guest | R16 |
| 6 | Sandbox and firewall fail closed, verified from inside on every start | R22, R23 |
| 7 | No snapshot fan-out from a template that has seen a secret | R27, R28 |
| 8 | If self-hosting a VMM: jailer or equivalent, one tenant, SMT off, release build | R33, R34, R35, R37 |
| 9 | No claim relies on a vendor boot number or on "agents can't escape microVMs" | R41, R43 |

**The four rules that gate everything else:** R1, because a container is not the boundary agents
fail to cross; R9, because egress is the incident vector on record; R15, because a mounted control
surface is an escape that needs no exploit; and R22, because a sandbox that fails open is a sandbox
you only believe in.
