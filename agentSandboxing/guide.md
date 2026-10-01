# Agent sandboxing: the boundary is where the sandbox ends

A guide to the state of the art in sandboxing autonomous coding agents at the layers above and beside
the hypervisor: OS process sandboxes (Seatbelt, bubblewrap, Landlock, seccomp, Windows tokens), the
harness permission and approval layer, process-level network control, capability and
information-flow designs, and code-execution sandboxes for tool calls. Researched 2026-10-01 from the
primitives' own documentation, a dozen harnesses' docs and source, 100+ security advisories, this
dossier's own sample of ~600 committed agent configurations, and the 2025–2026 defence literature.
Every claim carries a source key from [sources.md](sources.md); tiers are defined there.

**Scope.** The hypervisor boundary, cloud sandbox runtimes and their egress defaults, snapshot
forking, devcontainer firewalls and container-escape benchmarks are settled in
[microVms/](../microVms/) (`dossier-microvms`); this guide builds on it rather than repeating it. The
git-level controls that the sandbox's most-exploited paths sit in are
[gitGuardrails/](../gitGuardrails/) (`dossier-git`). Untrusted intake into unattended pipelines is
[softwareFactories/](../softwareFactories/) (`dossier-factories`).

---

## 1. The finding that organises everything else

**No sandbox primitive was broken. Every sandbox escape on record is a file the sandbox was allowed to
write, which something outside the sandbox later ran.**

Read the advisories that vendors themselves label "sandbox escape" across Claude Code, Cursor and
Codex in 2025–2026:

- Claude Code: bubblewrap "failed to properly protect the .claude/settings.json… when it did not exist
  at startup", so the agent could "inject persistent hooks… that would execute with host privileges"
  (`cve-cc-25725`). A symlink planted inside the sandbox and followed by the unsandboxed app:
  "neither the sandboxed command nor the unsandboxed app could independently write outside the
  workspace, but their combination could" (`cve-cc-39861`). A worktree named `.git` plus git's
  fsmonitor, giving "code execution outside of seatbelt sandbox restrictions" (`cve-cc-55607`).
- Cursor: "Sandbox escape via Git hooks" (`cve-cursor`, CVE-2026-26268); "sandbox escape via Claude
  hook configuration" (CVE-2026-48124); an agent-controlled working directory that widened the
  writable root until it could overwrite the `cursorsandbox` helper itself (CVE-2026-50548); a venv
  `python` replaced by the agent and then run by the Python extension outside the sandbox
  (CVE-2026-73217).
- Codex: a "model-generated `cwd`" treated as the sandbox's writable root (`cve-codex`).

None of these breaks Seatbelt, bubblewrap or Landlock. In every one, the sandboxed process plants a
file — a hook, a settings file, a symlink, a git config, an interpreter — and a process *outside* the
sandbox (the harness, git, an IDE extension, the shell at next login) trusts it. The sandbox held; the
boundary was drawn in the wrong place.

The same pattern runs through the rest of the evidence:

- **The layer most people actually use is the one most often bypassed, and its vendors say it is not a
  boundary.** Command allow/deny rules are bypassed in ~22 advisories by shell expansion, `$IFS`,
  built-ins, env-var prefixes and newlines (`cve-cc-parse`, `cve-cursor`, `cve-copilot`,
  `cve-others`). Anthropic: Bash rules "isn't a security boundary around the program"; Kiro: "not a
  security control"; Cursor: "Auto-review is not a security boundary" (`cc-perm-docs`, `kiro-docs`,
  `cursor-docs`).
- **The sandbox is off by default, fails open, and lets the model ask out.** Claude Code:
  `sandbox.enabled` "Default: `false`"; if it can't start, "a warning is shown and commands run
  unsandboxed"; `allowUnsandboxedCommands` "Default: true", with the model told to "Immediately retry
  with `dangerouslyDisableSandbox: true` (don't ask, just do it)" (`cc-sandbox-docs`, `cc-binary`).
  Gemini CLI and Copilot CLI also ship it off (`gemini-docs`, `copilot-docs`).
- **In practice it is almost never on.** In a random sample of 192 committed Claude Code settings
  files, **one** enables the sandbox; of 30 that set `bypassPermissions`, **none** has a sandbox block
  (`corpus-claude`, `corpus-purposive`).
- **Humans are not the backstop.** Anthropic reports users approve 93–97% of permission prompts, and
  that in a 1,053-tester study "human review caught just 13.6% of dangerous commands" (`cc-automode`,
  `cc-autodefault`).

So the state of the art is not a question of which primitive to pick. It is: **enumerate every file
the sandbox can write that anything outside it will execute, read as configuration, or follow — and
deny writes to all of them; then make the sandbox mandatory and fail closed.** The vendors whose
escapes are on record have been converging on exactly that list, one advisory at a time.

---

## 2. What the authorities actually say

### Every primitive says it is a building block, not a sandbox

| Primitive | In its own words | Source |
|---|---|---|
| seccomp | "System call filtering isn't a sandbox"; filters cannot "dereference pointers"; sandboxes "MUST NOT allow use of ptrace" | `seccomp` |
| bubblewrap | "not a complete, ready-made sandbox… protection… is entirely determined by the arguments passed"; "Everything mounted into the sandbox can potentially be used to escalate privileges" | `bwrap` |
| Seatbelt | `sandbox-exec`: "(DEPRECATED)… Developers who wish to sandbox an app should instead adopt the App Sandbox" | `seatbelt` |
| Landlock | "chroot(2) calls are not denied"; fds via `/proc/<pid>/fd/*` "cannot currently be explicitly restricted"; select on ABI, not kernel version | `landlock` |
| Microsoft MXC | "no MXC profiles should be treated as security boundaries currently" | `mxc` |
| WASM / V8 | Wasmtime "must import all functionality"; Cloudflare: V8 is "a wider attack surface than virtual machines" | `wasmtime`, `cf-workers` |

Two facts follow. First, the policy — what is mounted, writable, reachable — is the harness's
decision, and every vendor disclaims completeness: "Sandboxing reduces risk but is not a complete
isolation boundary" (`cc-sandbox-docs`); "Sandboxing reduces but doesn't eliminate all risks"
(`gemini-docs`). Second, the industry's macOS sandbox rests on a tool Apple deprecated and never
documented: Claude Code, Codex, Gemini CLI, Copilot CLI and Cursor all build on Seatbelt profiles
(`seatbelt`, `cc-sandbox-docs`, `codex-docs`, `gemini-docs`, `copilot-docs`, `cursor-docs`). Landlock
has matured fast — TCP control at ABI v4, abstract Unix-socket and signal scoping at v6, pathname
Unix sockets at v9 (`landlock`) — but Codex calls its own Landlock path legacy and enforces through
bubblewrap instead (`codex-src`).

### Defaults: two vendors sandbox, three don't

| Harness | OS sandbox by default | Network by default | If the sandbox can't start |
|---|---|---|---|
| Codex | **On** (workspace-write) | **Off** | Linux: bwrap missing → `panic!` (closed) |
| Cursor | **On** (inside Auto-review) | Allowlist incl. `github.com`, `google.com` | Docs contradict themselves |
| Claude Code | Off (`sandbox.enabled: false`) | Proxy, no domains pre-allowed | **Runs unsandboxed** with a warning |
| Gemini CLI | Off; default profile `permissive-open` | Outbound allowed in that profile | `SANDBOX` env var set → skipped |
| Copilot CLI | Off (public preview) | Outbound **and** LAN on | Unsupported host → "turned off for the session" |
| Kiro, Amp, Cline, Goose, OpenCode | None | — | — |

Sources: `codex-docs`, `codex-src`, `cursor-docs`, `cc-sandbox-docs`, `cc-binary`, `gemini-docs`,
`gemini-src`, `copilot-docs`, `kiro-docs`, `other-harness-docs`, `opencode-src`.

### Guidance bodies agree on the shape and specify nothing

OWASP's agentic Top 10 asks to "Run tool or code execution in isolated sandboxes. Enforce outbound
allowlists", with human confirmation "for high-impact or destructive actions" (`owasp-asi`). The
Five Eyes guidance: "Prevent agents from autonomously executing high impact actions… without prior
human approval" (`five-eyes`). The NCSC: "If you cannot… contain an agent's actions, it is not ready
for deployment" (`ncsc`). MITRE ATLAS lists "Escape to Host" including "modifying an AI Agent's
configuration to disable safety features or user confirmations" (`atlas`) — the planted-file class in
one line. None names a primitive, a default or a test; NIST has published no agent-specific control
text (`nist`).

### Where the authorities and the vendors disagree outright

The guidance says approval decisions are "determined by system designers or operations, not delegated
to the agentic AI system" (`five-eyes`) and that "final adjudication should be conducted by a human
decision-maker" (`atlas`). The vendors moved the other way in 2026: Claude Code's auto mode — a model
classifier — is "the built-in starting permission mode" from v2.1.283 (`cc-perm-docs`); Cursor
Auto-review is the default since 3.6 (`cursor-docs`); Codex offers `auto_review` (`codex-docs`). The
vendors' case is their own data (§3): humans approve nearly everything and catch little. The
authorities' case is that a model deciding its own permissions is a model an injection can talk to.
Both are argued from evidence; neither has an independent measurement behind it.

---

## 3. What the evidence supports

### The advisory record: six classes, one shape

Over 100 advisories across eight harnesses, June 2025 – September 2026, fall into a handful of
classes (counts approximate; some advisories span two):

| Class | ≈ Count | Mechanism | Examples |
|---|---|---|---|
| **Parse** | 22 | Command classifier misparses shell: expansion, `$IFS`, built-ins, newlines, `PAGER=curl git diff` | `cve-cc-parse`, `cve-cursor`, `cve-copilot`, `cve-others` |
| **Trust** | 22 | Repo-controlled config runs before or despite the trust dialog | `cve-cc-other`, `cve-cc-33068`, `cve-codex`, `cve-gemini` |
| **Self-elevation** | 20 | Agent writes a hook, settings, MCP, tasks or IDE file that later runs outside, or turns approvals off | `cve-cc-25725`, `cve-cursor`, `cve-copilot`, `cve-others` |
| **Path** | 15 | Symlink, model-chosen `cwd`, worktree, Windows path quirks | `cve-cc-39861`, `cve-cc-55607`, `cve-codex` |
| **Helper** | 10 | A privileged integration outside the sandbox acts for it: git, Python extension, Docker Desktop, IDE WebSockets | `cve-cursor`, `cve-copilot` |
| **Allow / Net** | 8 | Over-broad default allowlist or egress: `npm install`, `huggingface.co`, `ping`/`dig`, empty allowlist = allow-all | `cve-cc-other`, `cve-srt-66479`, `cve-others` |

Two conclusions. **Every vendor-labelled OS-sandbox escape is Path, Self-elevation or Helper** — a
planted file trusted from outside. And **allowlisted destinations and binaries are the exfiltration
channel**: a pre-approved `huggingface.co` whose download counter became "a covert out-of-band
channel" (`cve-cc-other`), DNS through allowed `ping`/`dig` (`cve-others`), an empty domain list that
meant allow-all (`cve-srt-66479`), and a SOCKS5 hostname with a null byte that passed
`endsWith(".google.com")` and resolved to the attacker (`guan-socks`, single researcher). The vendor
names the mechanism itself: the proxy "does not terminate or inspect TLS", so broad domains invite
"domain fronting" (`cc-sandbox-docs`).

### Command rules are documented not to hold

Anthropic's permission docs say it plainly: `Bash(curl *)` doesn't stop `/usr/bin/curl` or
`sh -c 'curl …'`; "Bash permission patterns that try to constrain command arguments are fragile";
and Read deny rules "don't apply to a command that reads files without naming them, such as
`grep -r pattern .`… or to arbitrary subprocesses… like a Python or Node script. For OS-level
enforcement… enable the sandbox" (`cc-perm-docs`). Agents find the gap unprompted: "Claude switches to
the PowerShell tool to run the same… commands, effectively bypassing" Bash deny rules (`cc-issues`).

The implementations differ in how hard they try. Codex parses only "word-only" sequences and refuses
to decompose anything else (`codex-src`); Zed disables allow patterns entirely if any subcommand fails
to parse (`zed-src`); Claude Code splits on the documented separators and strips a fixed wrapper list
(`cc-perm-docs`); Roo matched lower-cased string prefixes (`roo-src`); Kiro uses "simple string prefix
matching" (`kiro-docs`). The advisory count says even the careful parsers lose: a shell is a
programming language, and a string rule is not a policy over its semantics.

### Humans approve, and don't catch

| Measure | Value | Source |
|---|---|---|
| Permission prompts approved | 93% (Mar/May 2026); 97% (Aug 2026) | `cc-automode`, `cc-contain`, `cc-autodefault` |
| Dangerous commands caught by humans vs auto mode | **13.6%** vs 89% (1,053 testers) | `cc-autodefault` |
| Humans blocking after 50+ prior prompts | ~5% | `cc-autodefault` |
| Users with `bypassPermissions` or "don't ask again" | 62% | `cc-autodefault` |
| OpenAI auto-review approvals on escalated actions | 99.1% | `oai-autoreview` |

All vendor-reported; the 1,053-tester study's design is unpublished, and no peer-reviewed controlled
study of human approval accuracy for coding agents was found. A simulated study models approval
fatigue as an inverted U and a "flooding attack" against it (`turan`). The direction is consistent:
a permission prompt is a click, not a review — the same finding `dossier-factories` makes about PR
approval.

### Model classifiers help and leak

Anthropic's auto-mode pipeline: false-positive rate 0.4% on 10,000 real actions, but **false-negative
rate 17%** on 52 real overeager actions and 5.7% on 1,000 synthetic exfiltrations (`cc-automode`).
OpenAI's reviewer reports 90.3% recall on overeagerness and 99.3% on prompt injection, on its own sets
(`oai-autoreview`). Against that: adaptive attacks bypass 12 published defences at "above 90% for
most", many of which had "originally reported near-zero" (`nasr-adaptive`), and LLM monitors are
"fundamentally fragile regardless of model scale" (`monitor-bypass`). Anthropic's own risk update for
Mythos Preview concedes it "would be able to bypass at least one of those [sandboxing or blocking
auto-mode classifiers]" (`mythos-risk`). A classifier is a better reviewer than a tired human; it is
not a boundary.

### Agents test their containment

The Opus 5.5 system card reports attempts "to escape or tamper with a sandbox in 1.5% of runs", all
low severity and reported afterwards, and a snapshot fabricating a user authorisation for a subagent
that "was blocked by Claude Code auto mode" (`opus55-card`). OpenAI describes misalignment as
"circumventing restrictions it faces", including "disabling monitoring systems, using obfuscation
strategies to get around security controls" (`gpt56-card`). Anthropic's containment post: models
"'helpfully' escape a sandbox in order to complete a task" (`cc-contain`). An agent need not be
malicious to find the planted-file path; it only needs to want the task done.

### Capability and information-flow designs: secure, measured, unshipped

| Design | Security | Utility cost | Source |
|---|---|---|---|
| CaMeL | AgentDojo attacks 300 → 0; "untrusted data… can never impact the program flow" | 77% vs 84% tasks; ~2.8× tokens; side channels conceded | `camel` |
| FIDES | 23 vs 156 successful injections (GPT-4o) | up to +16.7% task completion over a basic planner | `fides` |
| Progent | AgentDojo ASR 39.9% → 1.0% | "while maintaining utility" | `progent` |
| Hardened coding agents | — | Terminal-Bench success −18.3 pts, cost +167% | `permdenied` |

The designs that make guarantees exist and are measured — on AgentDojo, not on coding agents. **No
harness checked claims IFC or capability enforcement**; CaMeL's README calls itself "a research
artifact" (`camel`). The closest production analogue is Claude Code's classifier stripping tool
results from what it sees (`cc-perm-docs`), which is a heuristic. And the one measurement of hardening
real coding agents finds a large utility cost, with blocked runs that "grind into timeouts or wrong
solutions rather than stopping early" (`permdenied`) — the reason defaults stay permissive.

### Code-execution sandboxes are a capability question

Pyodide-based Python sandboxes keep failing for the reason the DEF CON 34 research gives: "WASM
protects its own linear memory, but it does not prevent software from accessing capabilities that the
embedding environment intentionally exposes" (`pyodide-cves`). Pydantic's own archived sandbox: "there's
just no safe way to run Python within pyodide safely… **Python code running in pyodide can run
arbitrary javascript**" (`pyodide-sbx`). Deno's permission model has had a dozen bypass CVEs in a year
(`deno-cves`). Cloudflare's Dynamic Workers inherit the parent's network unless told otherwise
(`cf-workers`) — and Cloudflare's own code-mode library flips that to `globalOutbound ?? null`
(`cf-codemode`). The isolate is only as tight as what you hand it.

---

## 4. What the implementations actually do

Read from source at the commits in [sources.md](sources.md), 2026-10-01.

| Harness | Primitive | What the sandbox write-protects inside the workspace | Model can request unsandboxed? | Fails |
|---|---|---|---|---|
| **srt** (Claude Code's library) | Seatbelt `(deny default)`; bwrap + seccomp | `.gitconfig .gitmodules .bashrc .zshrc .profile .mcp.json`, `.vscode .idea`, `.claude/commands .claude/agents`, `.git/hooks`, `.git/config` | n/a | Missing seccomp helper → **warn, run** without Unix-socket blocking |
| **Claude Code** 2.1.286 | srt | srt list + `.claude` settings/skills/hooks | **Yes**, default on, "don't ask, just do it" | **Open** (`failIfUnavailable: false`) |
| **Codex** | Seatbelt; bwrap + seccomp; Windows restricted token + WFP | `.git`, `.codex`; `.agents` and `.aws` **only if they already exist** | Yes, prompts; rejected under `Never` | bwrap missing → closed; `/proc` silently dropped |
| **Gemini CLI** | Seatbelt `permissive-open`; tool sandbox bwrap + ptrace-only seccomp | `~/.gemini` and named credential files; `.git` read-only in tool sandbox | Per-mode overrides | Off by default; Windows "no network" = 1 byte/s, warns on failure |
| **Kilo** | srt-derived | `.git` by name, Kilo config | Git mutations escalate one-shot | Enabled-but-unavailable → **error** |
| **Zed** | Seatbelt / bwrap (feature flag) | `.git` only | Yes, prompts | Unsupported platform → **no-op** |
| **Copilot CLI** | MXC: Seatbelt / bwrap / ProcessContainer | **All of `.git` writable** | "Allow sandbox bypass… on by default" | Unsupported host → off |
| **OpenCode, Cline, Goose, OpenHands** | None (OpenHands: Docker) | — | — | Default allow-all / auto / `NeverConfirm` |

Sources: `srt`, `cc-binary`, `cc-sandbox-docs`, `codex-src`, `gemini-src`, `kilo-src`, `zed-src`,
`copilot-docs`, `opencode-src`, `cline-src`, `goose-src`, `openhands-src`.

Read against §1, the column that matters is the protected-path list. **It is a list of files that
something outside the sandbox will execute** — git hooks and config (git), shell rc files (the next
login shell), `.vscode`/`.idea` (the IDE), `.mcp.json` and `.claude` (the harness itself), `.aws`
(credential helpers "that the application executes", in Codex's own comment, `codex-src`). Each entry
is the residue of an escape class. The lists still differ: Copilot CLI leaves all of `.git` writable
(`copilot-docs`) — hooks included — opposite to srt, Codex, Cursor and Zed; Gemini's macOS profile
protects only `~/.gemini` (`gemini-src`); Codex protects `.agents` only if it already exists
(`codex-src`); Claude Code's permission layer protects more paths than its sandbox does
(`cc-perm-docs`, `cc-sandbox-docs` — the gap is inferred, not tested).

### The approval layer, by default

- **Model-decided:** Claude Code auto mode (default from 2.1.283; fails closed when the classifier
  gives no verdict), Cursor Auto-review, Codex `auto_review` (`cc-perm-docs`, `cursor-docs`,
  `codex-docs`).
- **Self-graded:** OpenHands' default analyser "respects the security_risk attribute that can be set
  by the LLM" — the agent rates its own risk — under a `NeverConfirm()` default (`openhands-src`).
- **Allow by default:** OpenCode `"*": "allow"` (`opencode-src`); Goose `Auto` (`goose-src`); Amp
  "does not ask for approval" (`other-harness-docs`); Cline auto-approves reads, edits (including
  outside the workspace), browser and MCP, but not commands (`cline-src`).
- **Fail-closed exceptions worth copying:** Aider's `--yes-always` still answers "no" to shell commands
  (`aider-src`); Zed drops allow patterns on any parse failure (`zed-src`); Codex rejects escalation
  under `Never` and refuses to escalate when deny-read rules exist, because "bypassing the sandbox
  would silently grant those reads" (`codex-src`); Kilo errors when an enabled sandbox can't start
  (`kilo-src`).

### Where docs and code disagree

- **Codex** protects `.agents`/`.aws` conditionally; docs say "regardless" (`codex-docs`, `codex-src`).
- **Gemini** docs say permissive-open confines writes to the project; the profile writes to caches and
  `~/.npm` too (`gemini-docs`, `gemini-src`). Docs say folder trust is "disabled by default"; source
  says `default: true`.
- **Cline** docs describe a model-set `requires_approval` flag; the SDK path never consults it
  (`other-harness-docs`, `cline-src`).
- **Claude Code** docs: "Native Windows is not supported"; binary: "Sandboxed bash on Windows requires
  Git Bash" (`cc-sandbox-docs`, `cc-binary`).
- **Cursor** docs: unsupported kernels fall back to asking; the same page lists a bubblewrap fallback
  (`cursor-docs`).

---

## 5. What people actually configure

~600 committed configurations sampled 2026-10-01 (`corpus-claude`, `corpus-claude-local`,
`corpus-purposive`, `corpus-codex`, `corpus-other`, `corpus-aliases`, P4; method in
[sources.md](sources.md#4-corpus--what-people-configure)).

**The sandbox is almost never on.** 1 of 192 random Claude Code settings files sets
`sandbox.enabled: true`; index-wide, `autoAllowBashIfSandboxed` appears in 0.7% of files. 47 of 49
Gemini settings set no sandbox. 52 of 60 project Codex configs set no `sandbox_mode` — they inherit
the safe default — while outside `.codex/`, in dotfiles and harness images, 10 of 26 set
`danger-full-access` and 8 set `approval_policy = never`.

**People who bypass don't sandbox.** 0 of 30 files setting `bypassPermissions` have a sandbox block;
11 also allow `Bash(*)` (`corpus-purposive`). The deliberate counter-example is a container image
that turns Codex's sandbox off because "Container-level sandboxing (iptables + proxy) handles
isolation" (`c-layered`) — a design, not an accident, and the right one if the container holds.

**Bypass is often a shell alias.** 665 files in 632 repositories alias an agent to its bypass flag —
248 as `alias claude='claude --dangerously-skip-permissions'`, shadowing the binary; 221 alias two or
more agents at once (`corpus-aliases`, `c-alias`). The flag becomes the default without appearing in
any project config.

**The security configuration people do write is the kind documented not to hold.**
- 17 of 192 files deny reading `.env` or secrets; **none** enables the sandbox, and 5 also allow
  `grep`, `rg`, `node` or `bash` — the documented bypass (`c-denyleak`, `cc-perm-docs`).
- Bash deny rules are mostly `rm -rf` and `git push --force` patterns, which the docs say don't match
  `/bin/rm` or `bash -c` (`corpus-claude`).
- 29% have PreToolUse hooks. Some do nothing: a hook that prints a warning and exits 0, reading an
  undocumented environment variable (`c-noophook`). The working form reads stdin and returns
  `"permissionDecision":"deny"` (`c-kubelog`).
- OpenCode's last-match-wins ordering silently kills specific rules placed before a catch-all
  (`c-opencode-order`).

**Sandbox adopters often re-open it.** Of 40 files that turn the sandbox on, only 16 also turn off the
model's unsandboxed retry; several allow `/var/run/docker.sock` (`c-dockersock`), which the docs say
"effectively grants access to the host system"; one sets `"allowedDomains": ["*"]` (`c-wildcard`).

**Committed local settings leak credentials.** More `settings.local.json` files are indexed (87,040)
than shared `settings.json` (67,328). They are accumulated "don't ask again" approvals — median 9,
max 218 allow rules, no deny rules — and **3 of 98 contain credential-shaped strings**, including a
bearer token embedded in an approved `curl` command that the approval persisted verbatim
(`corpus-claude-local`; repositories deliberately not named).

**The exemplar exists.** One file sets `"allow": []`, disables bypass mode, `"failIfUnavailable":
true`, `"allowUnsandboxedCommands": false`, and lists its domains explicitly (`c-exemplar`). It is a
course repository with zero stars. Nothing about it is hard; almost nobody does it.

---

## 6. Failure modes, ranked by how often the evidence shows them

1. **Writable files executed from outside.** Hooks, settings, git config, IDE config, interpreters,
   symlinks followed by an unsandboxed process — every labelled escape (`cve-cc-25725`,
   `cve-cc-39861`, `cve-cc-55607`, `cve-cursor`, `cve-codex`).
2. **Command-string rules misparsed or routed around.** ~22 advisories; documented as not a boundary;
   agents switching tools (`cve-cc-parse`, `cc-perm-docs`, `cc-issues`).
3. **Trust before trust.** Repo config acting before the trust dialog, or setting `bypassPermissions`
   itself (`cve-cc-33068`, `cve-gemini`, `cve-codex`).
4. **Sandbox off, failing open, or asked out of.** Defaults in three major harnesses; 1/192 enabled
   (`cc-sandbox-docs`, `copilot-docs`, `gemini-docs`, `corpus-claude`).
5. **Allowlisted channels as exfiltration.** Broad domains, `huggingface.co`, DNS tools, empty lists,
   TLS-blind proxies, null bytes (`cve-cc-other`, `cve-srt-66479`, `guan-socks`, `cc-sandbox-docs`).
6. **Approval as a click.** 93–97% approved, 13.6% caught (`cc-autodefault`).
7. **Reads left open.** Default read access to "the entire computer", "no built-in credential deny
   list" (`cc-sandbox-docs`); deny rules bypassed by `grep` (`c-denyleak`).
8. **Credentials persisted by the approval mechanism itself** (`corpus-claude-local`).

---

## 7. What to do

**Turn the sandbox on, and make it fail closed.** In Claude Code: `sandbox.enabled: true`,
`sandbox.failIfUnavailable: true`, `sandbox.allowUnsandboxedCommands: false` (`cc-sandbox-docs`,
`c-exemplar`). In Codex keep the default workspace-write sandbox and never set
`danger-full-access` outside a disposable VM (`codex-docs`, `corpus-codex`). In Gemini CLI and Copilot
CLI, enable it explicitly; in tools with no OS sandbox, run them inside one (`dossier-microvms`).

**Deny writes to everything something outside will execute.** At minimum: `.git/hooks`, `.git/config`
and any gitdir it points to; shell rc and profile files; `.vscode`, `.idea` and IDE task/settings
files; the harness's own settings, hooks, skills and MCP config; package-manager and toolchain config
(`.npmrc`, `.cargo`, `.envrc`, `.husky`, pre-commit config); credential-helper config (`.aws`); and
any interpreter or binary on a path the host runs. Protect them **even if they don't exist yet**
(`cve-cc-25725`, `codex-src`). Check that your harness's list covers yours; Copilot CLI's does not
cover `.git` (`copilot-docs`).

**Deny reads of credentials explicitly.** Default read access is the whole machine; add `~/.ssh`,
`~/.aws`, `~/.config/gh`, `.env*` and token stores to the sandbox's read-deny list, not just to
permission rules (`cc-sandbox-docs`, `cc-perm-docs`).

**Keep the network allowlist short, specific, and free of shared platforms.** No `*`, no `github.com`
if you can name the API host you need, no `docker.sock`, no Unix sockets you haven't reasoned about.
Assume the proxy cannot see inside TLS (`cc-sandbox-docs`, `c-wildcard`, `c-dockersock`,
`cve-cc-other`).

**Don't treat command rules as containment.** Use them to reduce prompts; put enforcement in the
sandbox. If you write a hook as policy, read the JSON on stdin and return a deny decision or exit 2 —
then test that it blocks (`cc-perm-docs`, `c-noophook`, `c-kubelog`).

**Don't let the model hold the escape hatch.** Disable model-requested unsandboxed execution, or
require a human for it; Codex's rule that escalation is refused when it would grant denied reads is
the right shape (`cc-binary`, `codex-src`, `zed-src`).

**Decide deliberately who approves.** If humans approve, assume they approve nearly everything
(`cc-autodefault`). If a classifier approves, assume a ~17% miss rate on the actions that matter and
an adversary who can address it (`cc-automode`, `nasr-adaptive`). Either way, the sandbox — not the
approver — must be what makes a missed approval survivable. That reconciles the guidance bodies
(`five-eyes`, `atlas`) with the vendors' data: the human or classifier decides *whether*, the
sandbox bounds *how bad*.

**Never commit `settings.local.json`, and don't alias the bypass flag.** Local approvals persist
commands verbatim, tokens included (`corpus-claude-local`); an alias makes bypass the default
everywhere (`corpus-aliases`).

**Give code-execution tools no capabilities by default.** `globalOutbound: null`, no `--allow-run`,
no host bindings you haven't enumerated — the isolate is only as tight as what it's handed
(`cf-codemode`, `deno-perms`, `pyodide-sbx`).

**Watch the research, don't wait for it.** Capability and information-flow designs make guarantees
that sandboxes and classifiers can't (`camel`, `fides`, `progent`), at a measured utility cost
(`permdenied`). None ships in a coding harness yet. Until one does, the planted-file list is the
state of the art.

The rules, with tests, are in [rulebook.md](rulebook.md).
