# Agent sandboxing rulebook

Checkable rules for containing coding agents at the OS-process and harness layers. Each rule states
its test (how a third party checks compliance), its evidence tier, and its source keys (see
[sources.md](sources.md)).

Tiers: **P1** primary spec, vendor docs, advisory or shipped source · **P2** peer-reviewed or preprint
research · **P3** vendor engineering blog / industry research with method · **P4** practitioner
measurement (including this dossier's own counts) · **P5** opinion, anecdote, unverified.

Use it two ways: as a configuration checklist for any harness on a developer machine or CI runner,
and as a review gate. A setup that fails R1, R5, R12 or R20 does not pass. The hypervisor boundary and
cloud sandboxes are in [microVms/rulebook.md](../microVms/rulebook.md); git hook and config controls
are in [gitGuardrails/rulebook.md](../gitGuardrails/rulebook.md).

---

## A. The sandbox exists, and stays on

**R1. Run agent shell commands inside an OS sandbox.** Test: does each harness in use have its OS
sandbox enabled (Claude Code `sandbox.enabled: true`; Codex not `danger-full-access`; Gemini/Copilot
explicitly on), or run inside a VM/container that is? **P1** `cc-sandbox-docs`, `codex-docs`,
`gemini-docs`, `copilot-docs` · **P4** `corpus-claude` · see `dossier-microvms`

**R2. Fail closed when the sandbox can't start.** Test: with the sandbox backend removed, does the
agent refuse to run commands? (Claude Code `failIfUnavailable: true`; Copilot device-managed.) **P1**
`cc-binary`, `copilot-docs`, `kilo-src`, `codex-src`

**R3. Don't let the model request unsandboxed execution.** Test: is `allowUnsandboxedCommands: false`
(or the harness equivalent) set, or does every unsandboxed request require a human? **P1** `cc-binary`,
`cc-sandbox-docs`, `zed-src`, `codex-src` · **P4** `corpus-purposive`

**R4. Never pair bypass mode with no sandbox.** Test: does any config or alias enable
`bypassPermissions`, `--dangerously-skip-permissions`, `--yolo` or `danger-full-access` outside a
disposable VM or container? **P1** `cc-perm-docs`, `codex-docs` · **P4** `corpus-purposive`,
`corpus-aliases`, `corpus-codex`

## B. The planted-file boundary

**R5. Deny sandbox writes to every file something outside will execute or load.** Git hooks and
config, shell rc/profile, IDE settings and tasks, harness settings/hooks/skills/MCP config, toolchain
config (`.npmrc`, `.cargo`, `.envrc`, `.husky`, pre-commit), credential-helper config (`.aws`).
Test: from inside the sandbox, does a write to each fail? **P1** `srt`, `codex-src`, `cve-cc-25725`,
`cve-cursor`, `cve-cc-55607` · **P1** `atlas`

**R6. Protect those paths before they exist.** Test: with `.claude/settings.json` (or `.agents/`,
`.vscode/`) absent, does creating it from inside the sandbox fail? **P1** `cve-cc-25725`, `codex-src`

**R7. Don't let the agent choose its working directory or writable root.** Test: is the writable root
fixed by the operator, not derived from a model-supplied `cwd`? **P1** `cve-codex`, `cve-cursor`

**R8. Treat symlinks created in the sandbox as hostile to unsandboxed writers.** Test: does every
unsandboxed component that writes into the workspace refuse to follow symlinks or re-resolve paths?
**P1** `cve-cc-39861`, `cve-cursor`, `cve-others`

**R9. Keep binaries the host runs out of writable paths.** Interpreters in venvs, LSP servers, helper
binaries. Test: is no executable that a host process spawns writable from inside the sandbox? **P1**
`cve-cursor` (CVE-2026-50548, -73217) · **P5** `kilo-src` (inferred)

**R10. Don't give the sandbox a privileged helper.** No `docker.sock`, no privileged Docker Desktop,
no IDE WebSocket without origin checks. Test: are no host control sockets reachable from inside?
**P1** `cc-sandbox-docs`, `cve-cursor` · **P4** `c-dockersock` · see `dossier-microvms`

**R11. Check your harness's list against yours.** Lists differ (Copilot CLI leaves `.git` writable;
Gemini protects only `~/.gemini`). Test: has each harness's write-protected set been compared to R5?
**P1** `copilot-docs`, `gemini-src`, `codex-src`, `srt`

## C. Reads and network

**R12. Deny reads of credentials at the sandbox, not only in permission rules.** Test: from inside the
sandbox, does `cat ~/.ssh/id_* ~/.aws/credentials .env` fail? **P1** `cc-sandbox-docs`, `cc-perm-docs` ·
**P4** `c-denyleak`

**R13. Deny egress by default; allow named hosts only.** Test: is the domain list explicit, free of
`*`, and free of shared platforms where a narrower host would do? **P1** `cc-sandbox-docs`,
`codex-docs` · **P4** `c-wildcard`

**R14. Assume the proxy cannot see inside TLS.** Test: does the threat model treat any allowed domain
as a possible fronting or exfiltration channel? **P1** `cc-sandbox-docs`, `srt` · **P1** `cve-cc-other`

**R15. Never trust an empty allowlist to mean deny.** Test: with zero domains configured, is egress
blocked? **P1** `cve-srt-66479`

**R16. Count the non-HTTP channels.** DNS, ICMP, Unix sockets, MCP connections, web-search tools.
Test: is each either blocked or deliberately allowed? **P1** `cve-others`, `codex-docs`, `srt`

**R17. Know whether the proxy is enforced or cooperative.** Test: does a program ignoring `HTTP_PROXY`
fail to connect? **P1** `srt`, `copilot-docs`

## D. Rules, hooks and approval

**R18. Don't use command-string rules as containment.** Test: is every security-relevant deny also
enforced by the sandbox (filesystem or network), not only by a `Bash(...)` pattern? **P1**
`cc-perm-docs`, `kiro-docs`, `cursor-docs` · **P1** `cve-cc-parse`

**R19. Don't pair a Read deny with interpreters or recursive search allows.** Test: for every
`Read(.env)`-style deny, are `grep -r`, `rg`, `python`, `node`, `bash` not auto-allowed? **P1**
`cc-perm-docs` · **P4** `c-denyleak`, `corpus-claude`

**R20. Repo config must not change trust or mode.** Test: can a committed project settings file enable
bypass mode, trust folders, or register MCP servers without a prompt? **P1** `cve-cc-33068`,
`cve-gemini`, `cve-codex`, `cve-others`

**R21. Test that policy hooks block.** Test: does each security hook read stdin JSON and return a deny
or exit 2, and has it been seen to block? **P1** `cc-perm-docs` · **P4** `c-noophook`, `c-kubelog`

**R22. Check rule ordering semantics.** Test: in last-match-wins systems, is the catch-all first?
**P1** `opencode-src` · **P4** `c-opencode-order`

**R23. Don't let the agent grade its own risk.** Test: is no approval decision derived from a
risk label the agent itself supplied? **P1** `openhands-src`

**R24. Treat human approval as a click.** Test: does the design remain safe if every prompt is
approved? **P3** `cc-autodefault`, `cc-automode` · **P2** `turan`

**R25. Treat classifier approval as a filter with a miss rate.** Test: is a classifier-approved action
still bounded by R1–R17? **P3** `cc-automode`, `oai-autoreview` · **P2** `nasr-adaptive`,
`monitor-bypass` · **P1** `mythos-risk`

**R26. Put state-changing instructions in deny rules, not chat.** Chat boundaries can be lost on
compaction. Test: is every "never do X" that matters encoded in config? **P1** `cc-perm-docs` · see
`dossier-ctx`

## E. Hygiene

**R27. Never commit `settings.local.json` or equivalent local approvals.** Test: is it in `.gitignore`,
and does a secret scan over history find none? **P4** `corpus-claude-local` · **P1** `cc-perm-docs`

**R28. Don't alias agents to their bypass flags.** Test: do shell rc files define no alias adding
`--dangerously-skip-permissions`, `--yolo`, `-y` or `--allow-all-tools`? **P4** `corpus-aliases`

**R29. Disable bypass mode by policy where you can.** Test: is `disableBypassPermissionsMode` (or the
harness equivalent, e.g. Gemini `secureModeEnabled`) set in managed settings? **P1** `cc-perm-docs`,
`gemini-docs` · **P4** `c-exemplar`

**R30. Patch harnesses on their release cadence.** 100+ containment advisories in 15 months. Test: is
each harness within its last minor release? **P1** `cve-cc-parse`, `cve-cursor`, `cve-codex`,
`cve-others`

## F. Code-execution tools

**R31. Give tool-call sandboxes no capabilities by default.** Test: is network (`globalOutbound`,
`--allow-net`), process spawning (`--allow-run`), FFI and host bindings off unless enumerated? **P1**
`cf-workers`, `cf-codemode`, `deno-perms`

**R32. Don't treat Pyodide-in-Deno as a boundary.** Test: is untrusted Python run in an OS sandbox or
VM, not only in Pyodide? **P1** `pyodide-sbx`, `pyodide-cves`, `deno-cves`

## G. Choosing

**R33. Prefer harnesses that fail closed.** Test: for each harness, has its behaviour on sandbox
unavailability, parse failure and missing classifier verdict been checked? **P1** `codex-src`,
`zed-src`, `aider-src`, `kilo-src`, `cc-perm-docs`

**R34. Don't read "sandbox" as one thing.** A process sandbox, a container, a classifier and an
approval prompt are different boundaries. Test: does each document describing the setup name the
mechanism? **P1** `seccomp`, `bwrap`, `cursor-docs`, `kiro-docs` · see `dossier-microvms`

**R35. Track capability and IFC designs for adoption.** Test: is there a decision on record for when
CaMeL/FIDES-class enforcement would be adopted, with its utility cost? **P2** `camel`, `fides`,
`progent`, `permdenied`

---

## Review checklist

An agent setup does not pass review until every one of these holds:

| # | Gate | Rules |
|---|---|---|
| 1 | OS sandbox on, fails closed, no model-requested escape | R1, R2, R3 |
| 2 | No bypass mode without a disposable VM/container around it | R4, R28, R29 |
| 3 | Every file executed or loaded from outside is write-denied, even if absent | R5, R6, R11 |
| 4 | Writable root fixed; symlinks, host-run binaries and helpers handled | R7, R8, R9, R10 |
| 5 | Credentials read-denied at the sandbox | R12, R19 |
| 6 | Egress default-deny with a specific allowlist; non-HTTP channels counted | R13, R15, R16 |
| 7 | Repo config cannot change trust, mode or MCP | R20 |
| 8 | Design survives every approval being granted | R24, R25 |
| 9 | No local approvals or credentials committed | R27 |

**The four rules that gate everything else:** R1, because a harness without an OS sandbox has only
string rules, which its vendors say are not a boundary; R5, because every sandbox escape on record is
a planted file run from outside; R12, because default read access is the whole machine; and R20,
because a repository that can set its own trust level owns the agent before the first prompt.
