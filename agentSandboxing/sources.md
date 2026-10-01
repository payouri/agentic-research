# Agent sandboxing source lexicon

Every source behind [guide.md](guide.md) and [rulebook.md](rulebook.md), keyed for citation.
Gathered 2026-10-01.

Trust tiers: **P1** primary spec, vendor documentation, advisory, kernel/man-page text, or shipped
source · **P2** peer-reviewed or arXiv research · **P3** vendor engineering blog / industry research
with disclosed method · **P4** practitioner report with measurement (including this dossier's own
counts, method stated) · **P5** opinion, anecdote, news, or unverified secondary.

Source code was read from shallow clones at the commit stated; Claude Code 2.1.286 and Copilot CLI
1.0.90 are closed-source and were read with `strings` on the shipped binaries. No sandbox escape was
executed for this dossier. Advisories were read from `gh api` (global and repository endpoints) and the
NVD API; the central ones were re-fetched by this dossier and are marked **(re-verified)**. Quotes
marked **(WF)** passed through a summarising fetcher.

This dossier builds on [microVms/](../microVms/) (`dossier-microvms`) for the hypervisor boundary,
cloud sandbox runtimes, cloud egress defaults, snapshots, devcontainer firewalls and
SandboxEscapeBench, and cites [gitGuardrails/](../gitGuardrails/) (`dossier-git`),
[softwareFactories/](../softwareFactories/) (`dossier-factories`),
[agentMemory/](../agentMemory/) (`dossier-memory`) and
[contextSmartZone/](../contextSmartZone/) (`dossier-ctx`) where they overlap.

---

## 1. Primary — what the primitives, the vendors and the guidance bodies say

### OS primitives

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `seatbelt` | `sandbox-exec(1)` (Xcode man page, via [mirror](https://keith.github.io/xcode-man-pages/sandbox-exec.1.html)) | P1 | man page 2017-03-09 | "execute within a sandbox (DEPRECATED)… Developers who wish to sandbox an app should instead adopt the App Sandbox". No Apple-published SBPL reference found |
| `landlock` | [Landlock kernel docs](https://docs.kernel.org/userspace-api/landlock.html) (7.3.0-rc5 rendering) + [landlock(7)](https://man7.org/linux/man-pages/man7/landlock.7.html) (man-pages 6.19) | P1 | Aug 2026; 2026-07-18 | ABI v1 5.13 (fs) … v4 6.7 (TCP bind/connect) … v6 6.12 (abstract-unix and signal scope) … v8 7.0 (TSYNC), v9 7.1 (pathname Unix sockets); "chroot(2) calls are not denied"; fds via `/proc/<pid>/fd/*` "cannot currently be explicitly restricted"; "use the Landlock ABI version rather than the kernel version" |
| `seccomp` | [Seccomp BPF](https://docs.kernel.org/userspace-api/seccomp_filter.html) + [seccomp_unotify(2)](https://man7.org/linux/man-pages/man2/seccomp_unotify.2.html) | P1 | fetched 2026-10-01 | "System call filtering isn't a sandbox"; filters cannot "dereference pointers"; "seccomp-based sandboxes MUST NOT allow use of ptrace"; unotify "explicitly not intended as a method implementing security policy" |
| `bwrap` | [containers/bubblewrap](https://github.com/containers/bubblewrap) README + releases | P1 | v0.13.0 2026-09-22; setuid removed v0.12.0 2026-08-26 after CVE-2026-41163 | "not a complete, ready-made sandbox… protection… is entirely determined by the arguments passed"; `--new-session` needed against TIOCSTI (CVE-2017-5226); "Everything mounted into the sandbox can potentially be used to escalate privileges" |
| `ubuntu-userns` | Canonical [blog](https://ubuntu.com/blog/ubuntu-23-10-restricted-unprivileged-user-namespaces) (2023-10-09) and [24.04 release notes](https://documentation.ubuntu.com/release-notes/24.04/) | P1 | 2023–2024 | Unprivileged user namespaces restricted by AppArmor; two descriptions of the mechanism (see conflicts) |
| `appcontainer` | Microsoft, [AppContainer isolation](https://learn.microsoft.com/en-us/windows/win32/secauthz/appcontainer-isolation) | P1 | ms.date 2025-07-08 | Isolation of credentials, devices, files, network, processes; "Read-only access is less restricted" |
| `mxc` | [microsoft/mxc](https://github.com/microsoft/mxc) README | P1 | pushed 2026-10-01 | Backend under Copilot CLI's Windows sandbox: "no MXC profiles should be treated as security boundaries currently" |
| `wasmtime` | [Wasmtime security](https://docs.wasmtime.dev/security.html) | P1 | fetched 2026-10-01 | "inherently sandboxed by design (must import all functionality)" |
| `cf-workers` | Cloudflare [Workers security model](https://developers.cloudflare.com/workers/reference/security-model/), [Dynamic Workers API](https://developers.cloudflare.com/dynamic-workers/api-reference/index.md) | P1 | 2026-09-10 | V8 isolates "a wider attack surface than virtual machines", plus a namespace+seccomp layer; Dynamic Workers: "If `globalOutbound` is not specified, the default is to inherit the parent's network access" |
| `deno-perms` | Deno, [Security and permissions](https://docs.deno.com/runtime/fundamentals/security/) | P1 | 2026-06-17 | `--allow-run` / `--allow-ffi` equivalent to allow-all |

### Harness documentation

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `cc-sandbox-docs` | Anthropic, [Configure the sandboxed Bash tool](https://code.claude.com/docs/en/sandboxing) | P1 | fetched 2026-10-01 (CC 2.1.286) | `sandbox.enabled` "Default: `false`"; Seatbelt on macOS, bubblewrap+socat on Linux/WSL2, "Native Windows is not supported"; if unavailable it "runs commands without sandboxing" unless `failIfUnavailable`; `allowUnsandboxedCommands` default true; `excludedCommands` "a convenience, not a security boundary"; reads cover "the entire computer", "still allows reading credential files such as `~/.aws/credentials`", "There is no built-in credential deny list"; proxy "does not terminate or inspect TLS", "domain fronting"; broad domains like `github.com` "can create paths for data exfiltration"; `docker.sock` "effectively grants access to the host system"; `enableWeakerNestedSandbox` "considerably weakens security"; "Sandboxing reduces risk but is not a complete isolation boundary"; sandbox write-deny list (`.claude` settings/skills/agents/commands/hooks, `.mcp.json`, shell rc, `.gitconfig`, `.vscode`, `.idea`, `.git/hooks`, `.git/config`) |
| `cc-perm-docs` | Anthropic, [Configure permissions](https://code.claude.com/docs/en/permissions), [Permission modes](https://code.claude.com/docs/en/permission-modes), [Hooks](https://code.claude.com/docs/en/hooks.md) | P1 | fetched 2026-10-01 | "With Claude Code v2.1.283 or later, auto mode is the built-in starting permission mode"; auto mode "does not guarantee safety", "Tool results are stripped"; chat boundaries "can be lost if context compaction removes the message"; 3 consecutive / 20 total blocks fallback; `bypassPermissions` "offers no protection against prompt injection", refused as root, not settable by project settings; separators `&& \|\| ; \| \|& &` split; fixed wrapper strip list; `Bash(curl *)` doesn't stop `/usr/bin/curl` or `sh -c`; Bash rules "isn't a security boundary around the program"; Read deny rules "don't apply to a command that reads files without naming them, such as `grep -r pattern .`… or to arbitrary subprocesses"; "For OS-level enforcement… enable the sandbox"; protected paths never auto-approved; "Hook decisions don't bypass permission rules"; exit 2 blocks |
| `srt` | [anthropic-experimental/sandbox-runtime](https://github.com/anthropic-experimental/sandbox-runtime) README + `src/sandbox/sandbox-utils.ts`, `linux-sandbox-utils.ts`, `macos-sandbox-utils.ts` | P1 | 5d196e09, v0.0.78, 2026-10-01 | "Beta Research Preview"; macOS profile `(deny default …)`; Linux `--unshare-net --unshare-pid --unshare-user --cap-drop ALL`; `DANGEROUS_FILES = ['.gitconfig','.gitmodules','.bashrc',…,'.mcp.json']` (re-verified); `// Git hooks are always blocked for security`; missing seccomp helper: "apply-seccomp binary not available - unix socket blocking disabled" at `warn`, command still runs (re-verified, line 3139); filter "does not prevent operations on Unix socket file descriptors inherited… or passed via `SCM_RIGHTS`"; proxy via env vars "may be ignored by programs that don't respect these variables"; Windows alpha: "DNS resolution via the system resolver is not fenced"; `allowAppleEvents` "removes code-execution isolation" |
| `cc-binary` | `@anthropic-ai/claude-code` 2.1.286 native binary ([npm](https://registry.npmjs.org/@anthropic-ai/claude-code)) | P1 | 2026-09-30 | `failIfUnavailable`: "When false (default), a warning is shown and commands run unsandboxed"; `allowUnsandboxedCommands` "Default: true"; model prompt: "Immediately retry with `dangerouslyDisableSandbox: true` (don't ask, just do it)" and "A sandbox denial on a credential, a file or a host that the task does not involve is the boundary above at work: tell the user rather than retrying"; built-in audit strings naming both settings as weakenings; "Sandboxed bash on Windows requires Git Bash" |
| `cc-automode` | J. Hughes, [How we built Claude Code auto mode](https://www.anthropic.com/engineering/claude-code-auto-mode) | P3 | 2026-03-25 | "Claude Code users approve 93% of permission prompts"; full pipeline FPR 0.4% (n=10,000 real), **FNR 17%** (n=52 real overeager), 5.7% (n=1,000 synthetic exfiltration); internal incidents: deleted remote branches, "uploading an engineer's GitHub auth token to an internal compute cluster" |
| `cc-contain` | Anthropic, [How we contain Claude across products](https://www.anthropic.com/engineering/how-we-contain-claude) | P3 | 2026-05-25 | "users approved roughly 93% of permission prompts"; sandboxing gave "84% reduction in permission prompts"; models "'helpfully' escape a sandbox in order to complete a task"; red-team phishing injection exfiltrated credentials in 24 of 25 retries |
| `cc-autodefault` | Anthropic, [Auto mode is now the default](https://claude.com/blog/auto-mode-default-in-claude-code) | P3 | 2026-08-07 | "users approve 97% of permission prompts"; 1,053 testers; "human review caught just 13.6% of dangerous commands, while auto mode caught 89%"; humans blocked ~5% after 50+ prompts; harm in 6.3% of manual vs 2.4% of auto sessions; 62% used `bypassPermissions` or "don't ask again". Study design unpublished |
| `codex-docs` | OpenAI, [Sandbox](https://learn.chatgpt.com/docs/sandboxing), [Agent approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security), [Windows sandbox](https://learn.chatgpt.com/docs/windows/windows-sandbox) (308s from developers.openai.com) | P1 | fetched 2026-10-01 | Workspace-write + on-request default; "By default, the agent runs with network access turned off"; `.git`, `.agents`, `.codex` read-only "regardless"; "Linux uses bwrap plus seccomp by default"; Windows `elevated` / `unelevated`; proxy does not filter "web search, app or connector tool calls, MCP server connections, browser or Computer Use"; `danger-full-access` "No sandbox; no approvals (not recommended)"; `auto_review` "denies critical-risk actions" |
| `codex-src` | [openai/codex](https://github.com/openai/codex) `protocol/src/permissions.rs`, `linux-sandbox/`, `core/src/exec_policy.rs`, `codex-rs/linux-sandbox/README.md` | P1 | d6c3b448, 2026-10-01 | `.agents` and `.aws` protected only `if … is_dir()`; `.codex` protected even when missing at the workspace root; "// AWS profiles can select credential helpers that the application executes." (re-verified, lines 2370–2386); bwrap missing → `panic!` (fail closed); `/proc` retry "silent", `mount_proc = false`; seccomp denies network syscalls, `ptrace`, `io_uring_*`, allows `AF_UNIX`; legacy Landlock `BestEffort`; dangerous + `Never` → `Forbidden`; escalation under `Never` rejected; escalation downgraded when deny-read rules exist ("bypassing the sandbox would silently grant those reads"); word-only parser; no `is_safe_command` remains; code mode V8 isolate `default_enabled: false` |
| `gemini-docs` | Google, `docs/cli/sandbox.md`, `trusted-folders.md`, `reference/policy-engine.md`, `configuration.md` ([gemini-cli](https://github.com/google-gemini/gemini-cli)) | P1 | main, fetched 2026-10-01 | "Sandboxing is disabled by default"; default `permissive-open` allows "broad file reads and network access"; "Sandboxing reduces but doesn't eliminate all risks"; YOLO "can only be enabled via command line"; MCP `trust` "bypass all tool call confirmations"; Workspace policy tier "currently non-functional" |
| `gemini-src` | gemini-cli `sandbox.ts`, `sandbox-macos-permissive-open.sb`, `LinuxSandboxManager.ts`, `GeminiSandbox.cs`, `policy-engine.ts`, `sandboxConfig.ts`, `settingsSchema.ts` | P1 | c6bccb7e, 2026-09-30 | `SEATBELT_PROFILE ??= 'permissive-open'`: `(allow file-read*)`, `(allow network-outbound)`, writes to target/tmp/cache/`~/.npm`/`~/.cache`; only `~/.gemini` and named credential files write-denied; Linux tool sandbox `--unshare-all`, seccomp matches only `SYS_ptrace` (re-verified); Windows "no network" is `MaxBandwidth = 1`, warning on failure; YOLO + parse failure → ALLOW; `SANDBOX` env var set → skipped; folder trust `default: true` |
| `copilot-docs` | GitHub, [About cloud and local sandboxes](https://docs.github.com/en/copilot/concepts/about-cloud-and-local-sandboxes), [Configuring local sandbox settings](https://docs.github.com/en/copilot/how-tos/cloud-and-local-sandboxes/configuring-local-sandbox-settings), [About Copilot CLI](https://docs.github.com/en/copilot/concepts/agents/about-copilot-cli) | P1 | fetched 2026-10-01 (CLI 1.0.90) | Local sandbox "turned off by default", public preview; before it, commands "use your credentials without restriction"; outbound and local network "Turned on by default"; "Allow sandbox bypass… Turned on by default"; unsupported host → sandbox "turned off for the session"; file tools "best-effort"; read/write "to everything in and below the repository's `.git` directory"; trusted-directory scoping "heuristic" |
| `cursor-docs` | Cursor, [Run modes](https://cursor.com/docs/agent/security/run-modes.md) | P1 | changelog through 3.6, 2026-05-29 | Seatbelt; Linux "Landlock and seccomp" on 6.2+; Auto-review default since 3.6; "Auto-review is not a security boundary"; classifier Gemini 3.5 Flash Lite / Claude 4.5 Haiku; default allowlist includes `github.com`, `*.githubusercontent.com`, `*.googleapis.com`, `google.com` |
| `kiro-docs` | Kiro, [Privacy and security](https://kiro.dev/docs/privacy-and-security/), [Autopilot](https://kiro.dev/docs/ide/chat/autopilot/) | P1 | updated 2026-08-04 | No OS sandbox; Supervised mode "is a code review workflow, not a security control… does not function as a sandbox"; trusted commands by "simple string prefix matching" |
| `other-harness-docs` | [Amp manual](https://ampcode.com/manual) / [permissions](https://ampcode.com/permissions); [Cline auto-approve](https://docs.cline.bot/features/auto-approve); goose `goose-permissions.md`; [Zed tool permissions](https://zed.dev/docs/ai/tool-permissions); [OpenHands runtimes](https://docs.openhands.dev/openhands/usage/runtimes/overview); [Warp permissions](https://docs.warp.dev/agents/cli/permissions-and-profiles.md); [Continue tool permissions](https://docs.continue.dev/cli/tool-permissions) | P1 | fetched 2026-10-01 | Amp "does not ask for approval before running tools"; Goose "`Autonomous Mode` is applied by default"; Cline YOLO "disables all safety checks" and "model marks each command with a `requires_approval` flag"; Zed default `"confirm"`; OpenHands Process sandbox "unsafe… No container isolation"; Warp auto-approve runs commands "that match your own command denylist" |

### Harness source (beyond the vendors above)

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `kilo-src` | [Kilo-Org/kilocode](https://github.com/Kilo-Org/kilocode) `kilocode/sandbox/policy.ts`, `kilocode/agent/index.ts` | P1 | 40127d26, 7.8.1, 2026-10-01 | Sandbox `enabled ?? false`; enabled-but-unavailable → error (fail closed); writable `Global.Path.bin`; default agent allows `tar *`, `cp *`, `mv *`, `echo *`, `rg *`; operator blocklist only in `readOnlyBash`: "This is defense-in-depth, not a sandbox" |
| `zed-src` | [zed-industries/zed](https://github.com/zed-industries/zed) `crates/agent/src/sandboxing.rs` | P1 | 71456c40, 2026-10-01 | Feature-flagged; "on platforms without one the per-command wrap is a no-op"; model parameters `unsandboxed`, `allow_fs_write_all`, `allow_hosts` each trigger a prompt; Seatbelt protects only `.git`; "If any command failed to parse, disable allow patterns for safety" |
| `opencode-src` | [sst/opencode](https://github.com/sst/opencode) `agent/agent.ts` + [docs](https://opencode.ai/docs/permissions/) | P1 | 0112a92c, 1.18.34, 2026-10-01 | Default `"*": "allow"`; no OS sandbox; "Most permissions default to `"allow"`"; "last matching rule winning" |
| `cline-src` | [cline/cline](https://github.com/cline/cline) `apps/vscode/src/shared/AutoApprovalSettings.ts`, SDK approval path | P1 | 8eee168b, 4.1.22, 2026-09-30 | `DEFAULT_AUTO_APPROVAL_SETTINGS`: `readFiles`, `readFilesExternally`, `editFiles`, `editFilesExternally`, `useBrowser`, `useMcp` all `true`; `executeSafeCommands: false` (re-verified); decision by tool name; "The SDK defaults unlisted tools to auto-approved"; `requires_approval` not consulted in the SDK path |
| `roo-src` | [RooCodeInc/Roo-Code](https://github.com/RooCodeInc/Roo-Code) `commands.ts` | P1 | b867ec91, 3.53.0, archived 2026-05-15 | Lower-cased `startsWith` prefix match + regex blocklist |
| `openhands-src` | [OpenHands/software-agent-sdk](https://github.com/OpenHands/software-agent-sdk) `conversation/state.py`, security analyzer | P1 | fad63774, 1.50.1, 2026-09-30 | Default `NeverConfirm()`; `LLMSecurityAnalyzer` "respects the security_risk attribute that can be set by the LLM" |
| `goose-src` | [block/goose](https://github.com/block/goose) `goose_mode.rs`, v1.25 blog | P1 | bab8ff64, 1.53.0 | `#[default] … Auto`; "The macOS seatbelt sandbox… was experimental and has been removed" |
| `aider-src` | [Aider-AI/aider](https://github.com/Aider-AI/aider) `io.py` | P1 | 5dc9490b | `--yes-always` answers "n" when `explicit_yes_required` (shell commands) — fail closed |
| `pyodide-sbx` | [pydantic/mcp-run-python](https://github.com/pydantic/mcp-run-python) (archived 2026-01-30); [langchain-sandbox](https://github.com/langchain-ai/langchain-sandbox) (archived 2026-01-14) | P1 | 2026-01 | `allow_networking: bool = True`; "there's just no safe way to run Python within pyodide safely… **Python code running in pyodide can run arbitrary javascript**"; "We do not recommend using `langchain-sandbox` for any production use cases" |
| `cf-codemode` | `@cloudflare/codemode` 0.5.2 ([npm](https://registry.npmjs.org/@cloudflare/codemode)) | P1 | fetched 2026-10-01 | `this.#globalOutbound = options.globalOutbound ?? null;` — closes the platform's open default |

### Guidance and standards

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `owasp-asi` | OWASP, [Top 10 for Agentic Applications 2026](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) | P1 (advisory) | 2025-12-09 | "Least-Agency"; ASI02: "Run tool or code execution in isolated sandboxes. Enforce outbound allowlists"; human confirmation "for high-impact or destructive actions"; ASI05: "Never run as root" |
| `owasp-llm06` | OWASP, [LLM06:2025 Excessive Agency](https://genai.owasp.org/llmrisk/llm062025-excessive-agency/) | P1 (advisory) | 2025 | Avoid "open-ended extensions" such as shell; complete mediation "rather than relying on an LLM to decide" |
| `five-eyes` | CISA, NSA, ASD ACSC, CCCS, NCSC-NZ, NCSC-UK, [Careful Adoption of Agentic AI Services](https://cyber.gc.ca/en/guidance/careful-adoption-agentic-ai) | P1 (advisory) | 2026-05-01 | "Prevent agents from autonomously executing high impact actions… without prior human approval"; approval decisions "determined by system designers or operations, not delegated to the agentic AI system"; isolation "where possible" |
| `ncsc` | UK NCSC, [Thinking carefully before adopting agentic AI](https://www.ncsc.gov.uk/blogs/thinking-carefully-before-adopting-agentic-ai) | P1 (advisory) | 2026-05-15 | "If you cannot… contain an agent's actions, it is not ready for deployment" |
| `nist` | NIST [AI 600-1](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf) (Jul 2024; no "sandbox" text), [CAISI hijacking blog](https://www.nist.gov/news-events/news/2025/01/technical-blog-strengthening-ai-agent-hijacking-evaluations) (2025-01-17), [AI Agent Standards Initiative](https://nist.gov/news-events/news/2026/02/announcing-ai-agent-standards-initiative-interoperable-and-secure) (2026-02-17), [COSAiS](https://csrc.nist.gov/projects/cosais) (agent overlays unreleased) | P1 | 2024–2026 | No agent-specific control text published |
| `atlas` | MITRE [ATLAS 2026.09](https://github.com/mitre-atlas/atlas-data) | P1 | 2026.09 | AML.M0031 isolate execution; AML.M0028 "final adjudication should be conducted by a human decision-maker"; "Escape to Host" includes "modifying an AI Agent's configuration to disable safety features or user confirmations" |

---

## 2. Evidence — what has been measured or disclosed

### Harness advisories (central entries re-verified 2026-10-01)

| Key | Advisory | Tier | Published | What it settles |
|---|---|---|---|---|
| `cve-cc-25725` | [GHSA-ff64-7w26-62rf](https://github.com/advisories/GHSA-ff64-7w26-62rf) / CVE-2026-25725, Claude Code < 2.1.2 | P1 | 2026-02-06 (re-verified) | "Sandbox Escape via Persistent Configuration Injection in settings.json": bubblewrap "failed to properly protect the .claude/settings.json… when it did not exist at startup… inject persistent hooks… that would execute with host privileges" |
| `cve-cc-39861` | [GHSA-vp62-r36r-9xqp](https://github.com/advisories/GHSA-vp62-r36r-9xqp) / CVE-2026-39861, < 2.1.64 | P1 | 2026-04-21 (re-verified) | Symlink following: "neither the sandboxed command nor the unsandboxed app could independently write outside the workspace, but their combination could" |
| `cve-cc-55607` | [GHSA-7835-87q9-rgvv](https://github.com/advisories/GHSA-7835-87q9-rgvv) / CVE-2026-55607, ≥ 2.1.38 < 2.1.163 | P1 | global 2026-07-24 (re-verified); repo 2026-06-25 | Worktree named `.git` + symlinks + fsmonitor → "code execution outside of seatbelt sandbox restrictions". Also in `dossier-git` and `dossier-microvms` |
| `cve-cc-33068` | [GHSA-mmgp-wc2j-qcv7](https://github.com/advisories/GHSA-mmgp-wc2j-qcv7) / CVE-2026-33068, < 2.1.53 | P1 | 2026-03-19 (re-verified) | Repo `.claude/settings.json` sets `defaultMode` to `bypassPermissions`, "causing the trust dialog to be silently skipped" |
| `cve-srt-66479` | [GHSA-9gqj-5w7c-vx47](https://github.com/advisories/GHSA-9gqj-5w7c-vx47) / CVE-2025-66479, sandbox-runtime < 0.0.16 | P1 | 2025-12-04 (re-verified), severity "low" | Network sandbox not enforced "if the sandbox policy did not configure any allowed domains" |
| `cve-cc-parse` | Claude Code PARSE-class advisories: CVE-2025-54795 (`echo`), -58764 (`rg`), -64755 (`sed`), -66032 (`$IFS`, short flags), CVE-2026-24053 (zsh `>\|`), -24887 (`find`), -25722 (`cd`), -25723 (piped `sed`) — via [anthropics/claude-code advisories](https://github.com/anthropics/claude-code/security/advisories) | P1 | 2025-08 to 2026-02 | Repeated command-validation bypasses |
| `cve-cc-other` | CVE-2025-55284 (over-broad safe-command list read+send "without user confirmation"), CVE-2026-54316 (`huggingface.co` pre-approved; download counter as "a covert out-of-band channel"), CVE-2025-59829 / CVE-2026-25724 (deny via symlink), CVE-2025-59536 / -65099 / CVE-2026-21852 / -40068 (pre-trust execution) | P1 | 2025-08 to 2026-06 | ALLOW, NET, PATH and TRUST classes |
| `cve-cursor` | [cursor/cursor advisories](https://github.com/cursor/cursor/security/advisories): GHSA-8pcm-8jpx-hv8r / CVE-2026-26268 ("Sandbox escape via Git hooks", 2026-02-13), GHSA-pc9j-3qc2-95wv / CVE-2026-48124 ("sandbox escape via Claude hook configuration", 2026-05-21), GHSA-3p48-7v9f-v5cw / CVE-2026-50548 (critical, "agent-controlled working directory", 2026-06-05) — all re-verified; plus CVE-2026-50549 (canonicalisation fallback), -73217 (venv `python` run by the Python extension), -73218 (Docker Desktop privileged container), CVE-2025-54135 (creating `.cursor/mcp.json` unapproved), -54131 (backtick / `$()`), CVE-2026-22708 (built-ins) | P1 | 2025-04 to 2026-07 | Cursor's sandbox escapes are all planted-file / helper / path classes. Several advisories list a vulnerable range that contradicts their patched version |
| `cve-codex` | [GHSA-w5fx-fh39-j5rw](https://github.com/advisories/GHSA-w5fx-fh39-j5rw) / CVE-2025-59532, Codex 0.2.0–0.38.0 (re-verified, 2025-09-19); [GHSA-xrxf-jgv3-qmrm](https://github.com/advisories/GHSA-xrxf-jgv3-qmrm) / CVE-2025-61260 | P1 | 2025-09-19; 2026-04-14 | Model-generated `cwd` treated as writable root; project `.env` → `CODEX_HOME` → MCP auto-exec |
| `cve-gemini` | [GHSA-wpqr-6v78-jr5g](https://github.com/advisories/GHSA-wpqr-6v78-jr5g) / CVE-2026-12537 | P1 | GHSA 2026-04-24; NVD 2026-06-24 | Headless auto-trust → "pre-sandbox host-level code execution" |
| `cve-copilot` | CVE-2025-53773 ([NVD](https://nvd.nist.gov/vuln/detail/CVE-2025-53773)) + [Embrace The Red](https://embracethered.com/blog/posts/2025/github-copilot-remote-code-execution-via-prompt-injection/) (P4); Copilot CLI [GHSA-g8r9-g2v8-jv6f](https://github.com/advisories/GHSA-g8r9-g2v8-jv6f) / CVE-2026-29783 (parameter-transformation operators bypass read-only classification); [GHSA-9ccr-r5hg-74gf](https://github.com/advisories/GHSA-9ccr-r5hg-74gf) / CVE-2026-45033 (`core.fsmonitor` "and 15+ similar keys") | P1 | 2025-08 to 2026-05 | `"chat.tools.autoApprove": true` written to `.vscode/settings.json`; parser and git-config classes |
| `cve-others` | Zed CVE-2025-55012 (agent writes project config → RCE), CVE-2026-44463 ("`PAGER=curl git diff` matches the `^git\b` pattern"), CVE-2026-27967 (symlink); Roo CVE-2025-53536 (VS Code settings), CVE-2025-58374 (`npm install` default auto-approve); Amazon Q / Kiro [AWS-2025-019](https://aws.amazon.com/security/security-bulletins/AWS-2025-019) (`ping`/`dig` DNS exfiltration; settings changes without HITL); Kiro CVE-2026-10591 (`.vscode/tasks.json`); Windsurf CVE-2026-30615 (MCP auto-registration) | P1 | 2025-07 to 2026-06 | Same classes across every harness |
| `guan-socks` | Aonan Guan, [Second time, same sandbox](https://oddguan.com/blog/second-time-same-sandbox-anthropic-claude-code-network-allowlist-bypass-data-exfiltration/) | P4 | 2026-05-20 | SOCKS5 hostname `attacker-host.com\x00.google.com` passes `endsWith(".google.com")`, resolves the prefix; affected CC 2.0.24–2.1.89, fixed 2.1.90; no advisory issued. Single researcher |
| `deno-cves` | [denoland/deno advisories](https://github.com/denoland/deno/security/advisories): CVE-2025-48888, -48934, -48935, -61785/6; CVE-2026-49401, -49406, -49411, -49859/60, -49983, -27190 and its bypass -32260 | P1 | 2025-06 to 2026-06 | Permission-model bypasses in the runtime under Pyodide sandboxes |
| `pyodide-cves` | n8n [CVE-2025-68668](https://nvd.nist.gov/vuln/detail/CVE-2025-68668); Grist [CVE-2026-24002](https://nvd.nist.gov/vuln/detail/CVE-2026-24002) ("pyodide on node does not have a useful sandbox barrier"); DEF CON 34 via [eSecurityPlanet](https://www.esecurityplanet.com/threats/def-con-34-one-pyodide-flaw-exposed-seven-products/) (P5) | P1/P5 | 2025-12 to 2026-08 | "WASM protects its own linear memory, but it does not prevent software from accessing capabilities that the embedding environment intentionally exposes" |

### Approval and monitoring

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `oai-autoreview` | Trębacz et al., [Auto-review](https://alignment.openai.com/auto-review) | P3 | 2026-04-30 | "Auto-approval Rate On Escalated Actions: 99.1%"; overeagerness recall 90.3%; prompt-injection recall 99.3% (own sets) |
| `turan` | E. Turan, [Oversight Has a Capacity](https://arxiv.org/abs/2606.08919) | P2 | 2026-06-08 | Fatigue "inverted-U and the flooding attack are modeling results" — simulated; κ=0.52 |
| `dhanorkar` | Dhanorkar, Passi, Vorvoreanu, [arXiv 2606.05391](https://arxiv.org/abs/2606.05391) | P2 | 2026-06-03 | 17-developer qualitative oversight study; no approval rates |

### Defences

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `camel` | Debenedetti et al., [Defeating Prompt Injections by Design](https://arxiv.org/abs/2503.18813) + [repo](https://github.com/google-research/camel-prompt-injection) | P2 | v2 2025-06-24 | "untrusted data retrieved by the LLM can never impact the program flow"; AgentDojo "77% of tasks with provable security (compared to 84% with an undefended system)"; attacks 300 → 0 (Gemini 2.5 Pro); ~2.8× tokens; "vulnerable to side-channel attacks"; README: "a research artifact… might not be fully secure" |
| `fides` | Costa et al. (Microsoft), [arXiv 2505.23643](https://arxiv.org/abs/2505.23643) | P2 | v2 2025-09-03 | IFC labels, "deterministically enforces security policies"; GPT-4o 23 vs 156 successful injections **(WF)** |
| `progent` | Shi et al., [Progent](https://arxiv.org/abs/2504.11703) | P2 | v3 2026-05-14 | AgentDojo ASR "39.9% to 1.0%"; ASB "70.3% to 3.9%" **(WF)** |
| `dual-llm` | S. Willison, [The Dual LLM pattern](https://simonwillison.net/2023/Apr/25/dual-llm-pattern/) | P5 | 2023-04-25 | Quarantined LLM "does not have access to tools" |
| `isolategpt` | Wu et al., [IsolateGPT](https://arxiv.org/abs/2403.04960) (NDSS 2025) | P2 | v2 2025-01-30 | Overhead "under 30% for three-quarters of tested queries" [ABS] |
| `nasr-adaptive` | Nasr, Carlini, … Tramèr, [The Attacker Moves Second](https://arxiv.org/abs/2510.09023) | P2 | 2025-10-10 | 12 defences bypassed, ASR "above 90% for most", many "originally reported near-zero" [ABS] |
| `firewalls` | Bhagwatkar et al., [arXiv 2510.05244](https://arxiv.org/abs/2510.05244) | P2 | rev. 2026-03-23 | Tool firewalls "achieve perfect security" on standard benchmarks, which have "flawed success metrics, implementation bugs, and… weak attacks" [ABS] |
| `monitor-bypass` | Isbarov & Kantarcioglu, [Agent-as-a-Proxy](https://arxiv.org/abs/2602.05066) | P2 | 2026-02-25 | LLM monitors "fundamentally fragile regardless of model scale" [ABS] |
| `permdenied` | Davidovich et al., [Permission Denied](https://arxiv.org/abs/2608.02670) | P2 | 2026-08-02 | 12 agents on Terminal-Bench 2.1: max hardening success −18.3 pts, cost +167.3%; blocked runs "grind into timeouts or wrong solutions rather than stopping early" [ABS] |

### Agents against their containment

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `opus55-card` | Anthropic, [Claude Opus 5.5 System Card](https://www-cdn.anthropic.com/fc1b44717c85dc068bc6ba5024219938094694bd/Claude%20Opus%205.5%20System%20Card.pdf) | P1 | 2026-09-22 | "attempted to escape or tamper with a sandbox in 1.5% of runs" (low severity, reported afterwards); fabricated user authorisation for a subagent "blocked by Claude Code auto mode"; prompt-injection eval: with auto mode "no attack succeeded" |
| `mythos-risk` | Anthropic, [Alignment Risk Update: Claude Mythos Preview](https://www-cdn.anthropic.com/79c2d46d997783b9d2fb3241de43218158e5f25c.pdf) | P1 | 2026-04-07 | "it's likely Mythos Preview would be able to bypass at least one of those [sandboxing or blocking auto-mode classifiers]"; sandboxes "may sometimes be misconfigured in a way that could allow escapes" |
| `gpt56-card` | OpenAI, [GPT-5.6 System Card](https://deploymentsafety.openai.com/gpt-5-6/gpt-5-6.pdf) | P1 | 2026-07-09 | Misalignment as "circumventing restrictions it faces", including "disabling monitoring systems, using obfuscation strategies to get around security controls" |
| `cc-issues` | [anthropics/claude-code#60935](https://github.com/anthropics/claude-code/issues/60935) (2026-05-20: "Claude switches to the PowerShell tool to run the same… commands, effectively bypassing" Bash deny rules); [#43142](https://github.com/anthropics/claude-code/issues/43142) (2026-04-03, subagent bypassed `Bash(git *)` deny) | P5 | 2026 | First-hand reports, labelled `has repro` / `area:security` |

---

## 3. Implementations — cross-reference

The implementation sources are the P1 code entries in §1 (`srt`, `cc-binary`, `codex-src`, `gemini-src`,
`kilo-src`, `zed-src`, `opencode-src`, `cline-src`, `roo-src`, `openhands-src`, `goose-src`,
`aider-src`, `pyodide-sbx`, `cf-codemode`). The comparison table is in
[guide.md §4](guide.md#4-what-the-implementations-actually-do).

---

## 4. Corpus — what people configure

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `corpus-claude` | This dossier's sample: GitHub REST `search/code` (legacy index), `path:.claude filename:settings.json` in 6 `size:` buckets, seeded random draw (seed 20261001), one file per repo, N=200 (192 parsed); median 25★, median last commit 2026-05-19; plus index-wide keyword counts over 67,328 files | P4 | 2026-10-01 11:22–11:50 UTC | `permissions` block 57%; `sandbox.enabled: true` **1/192**; `bypassPermissions` 3/192 (index ratio 1.8%); `autoAllowBashIfSandboxed` 0.7% of index; PreToolUse hooks 29%; interpreter allows 34/192; `git push` allows 18; `Bash`/`Bash(*)` 6; 17 secret-deny files, **0 with sandbox**, 5 also allowing `grep`/`rg`/`node`/`bash` |
| `corpus-claude-local` | Same method, `settings.local.json`, N=100 (98 parsed); 87,040 indexed (more than shared `settings.json`) | P4 | same | Median 9 allow rules, max 218; deny ≈ 0; 0 sandbox; **3/98 with credential-shaped strings**, including a bearer token embedded inside a "don't ask again" allow rule. Repositories not named here |
| `corpus-purposive` | Purposive: 40 files matching `autoAllowBashIfSandboxed`; 30 matching `bypassPermissions` | P4 | same | Sandbox adopters: `allowUnsandboxedCommands: false` 16/40, unset 21; `allowedDomains` 12 (1 is `["*"]`); `docker.sock` allowed in several. Bypass adopters: **0/30 have a sandbox block**; 11 also allow `Bash(*)` |
| `corpus-codex` | `path:.codex filename:config.toml` N=60; `filename:config.toml sandbox_mode NOT path:.codex` N=26 | P4 | same | `.codex/`: 52/60 set no `sandbox_mode`, 40 mainly MCP; elsewhere (dotfiles, harness images): `danger-full-access` 10/26, `approval_policy = never` 8/26 |
| `corpus-other` | Gemini N=49 (47 no sandbox); OpenCode N=45 (10 no `permission` block; last-match-wins traps); Cursor CLI N=20 (`Shell(**)` in 2); Copilot CLI N=20; Goose N=15 (`auto` 6); Kilo/Roo/Cline: no committed configs found | P4 | same | Sandboxing absent across tools; configs mostly MCP setup |
| `corpus-aliases` | Six alias queries, three pages each, regex-verified on fragments | P4 | same | **665 files in 632 repos** alias an agent to its bypass flag (lower bound); shadowing the binary: claude 248, codex 187, gemini 35, copilot 17; 221 alias two or more agents |

Exemplars and cautionary cases (no secrets):

| Key | File | Last commit | Why it is cited |
|---|---|---|---|
| `c-exemplar` | [theyur/course_m03_09 `.claude/settings.json`](https://github.com/theyur/course_m03_09/blob/c747cde369303d443d92c301d279d38d56430632/.claude/settings.json) | 2026-09-06 | `"allow": []`, `"disableBypassPermissionsMode": "disable"`, `"failIfUnavailable": true`, `"allowUnsandboxedCommands": false`, explicit `allowedDomains` |
| `c-couchdb` | [matteobortolazzo/couchdb-net](https://github.com/matteobortolazzo/couchdb-net/blob/afbefdaa335f920d0bbd88f09f857b79aaa1a416/.claude/settings.json) | 2026-06-17 | Escape hatch off and domains listed — but `github.com` allowed and `docker`/`podman` excluded |
| `c-kubelog` | [kube-logging/logging-operator](https://github.com/kube-logging/logging-operator/blob/5ef2a7254148e0c2d3f245197999e3c684368ced/.claude/settings.json) | 2026-06-15 | Secret deny rules plus a stdin-reading hook emitting `"permissionDecision":"deny"`; no sandbox |
| `c-wildcard` | [devrsi0n/doodleshooter](https://github.com/devrsi0n/doodleshooter/blob/2bfa3525b5d5a012a1d317dc9c90c2cf63fb0d70/.claude/settings.json) | 2026-09-17 | Sandbox on with `"allowedDomains": ["*"]`, `"allowAllUnixSockets": true` |
| `c-dockersock` | [WhatIfWeDigDeeper/algorithms-in-ts](https://github.com/WhatIfWeDigDeeper/algorithms-in-ts/blob/e3d3bf6b209c9d984427727f6a6682706d4bbd21/.claude/settings.json) | 2026-02-18 | `"allowUnixSockets": ["/var/run/docker.sock"]` |
| `c-bypass` | [luanmorenommaciel/agentspec](https://github.com/luanmorenommaciel/agentspec/blob/74fab99fee5074ef9b6ea670f519d635b4ac5861/.claude/settings.json) | 2026-03-26 | `"defaultMode": "bypassPermissions"`, `"Bash(*)"`, `"deny": []` |
| `c-denyleak` | [sKuhLight/Axis](https://github.com/sKuhLight/Axis/blob/6b87bd2472fd88854421fda0dd1d2d7a02d2dd19/.claude/settings.json), [CliMA/EnsembleKalmanProcesses.jl](https://github.com/CliMA/EnsembleKalmanProcesses.jl/blob/d10e5214171ead2a3f4fdeedc6df8ae9be5a4831/.claude/settings.json) | 2026-07/08 | `.env` Read deny alongside `Bash(rg:*)` / `Bash(grep *)` allows |
| `c-noophook` | [hoangsonww/WealthWise-Finance-Tracker](https://github.com/hoangsonww/WealthWise-Finance-Tracker/blob/8abd4ea065e461233584a63e65b70252604f67f9/.claude/settings.json) | 2026-03-03 | PreToolUse hook that warns then `exit 0`, reading `$CLAUDE_TOOL_INPUT_COMMAND` (not a documented variable) |
| `c-centaur` | [paradigmxyz/centaur `harness/codex/config.toml`](https://github.com/paradigmxyz/centaur/blob/fee192b989315a065ae03a9781a580f213bcba6f/harness/codex/config.toml) | 2026-09-01 | `approval_policy = "never"`, `danger-full-access`, `[projects."/"] trust_level = "trusted"` |
| `c-layered` | [mattolson/agent-sandbox](https://github.com/mattolson/agent-sandbox/blob/c5b65e7cbd8f5b3bbf4e3ea40900c0014eedfa04/images/agents/codex/config.toml) | 2026-02-24 | Deliberately disables Codex's sandbox: "Container-level sandboxing (iptables + proxy) handles isolation." |
| `c-opencode-order` | [Nuitka/Nuitka-Watch `opencode.json`](https://github.com/Nuitka/Nuitka-Watch/blob/4fa58d29a7c855490abf47d6b68083da1df4e077/opencode.json) | 2026-09-01 | 24 specific rules then `"*": "ask"` last — last-match-wins makes them dead |
| `c-alias` | [mikker/dotfiles `aliases.zsh`](https://github.com/mikker/dotfiles/blob/2aac13108b224027b176f2afed4ef4bc2e3a38d4/zsh/aliases.zsh) | — | `codex --yolo`, `claude --dangerously-skip-permissions`, `gemini -y` |

---

## Conflicts resolved

- **Cline's command defaults.** Implementations reported commands auto-approved by default. Re-reading
  `AutoApprovalSettings.ts` (`cline-src`): reads, external reads, edits, external edits, browser and MCP
  default `true`; `executeSafeCommands: false`; the approval path keys commands on
  `executeSafeCommands`. **Resolved: edits, browser and MCP auto-approved by default; commands not.**
- **Codex protected paths.** `codex-docs` says `.git`, `.agents`, `.codex` are read-only "regardless";
  `codex-src` adds `.aws` and protects `.agents` and `.aws` only if they already exist. **Resolved
  against source** (re-read lines 2365–2390). Whether a fresh `.agents/` can be created unprompted is
  inferred, not tested.
- **Gemini's permissive-open profile.** Docs say writes are confined to the project directory; the
  profile also allows tmp, cache, `~/.npm`, `~/.cache` and include-dirs, and leaves the project's
  `.gemini/` writable. **Resolved against `gemini-src`.**
- **Gemini folder trust default.** `trusted-folders.md`: "disabled by default"; `settingsSchema.ts`:
  `default: true`. **Resolved against source: on.**
- **Gemini seccomp.** Reported as ptrace-only; **re-verified** in `LinuxSandboxManager.ts`.
- **srt's missing-seccomp path.** **Re-verified**: warns and runs with Unix-socket blocking disabled.
- **CVE-2026-55607's date.** Repo GHSA 2026-06-25, global GHSA 2026-07-24 (re-verified), NVD
  2026-06-29 (per `dossier-microvms`). **Same advisory, three publication events.**
- **CVE-2025-53773's mechanism.** NVD says "command injection"; Embrace The Red describes writing
  `chat.tools.autoApprove`. **Both: the second is the mechanism of the first.** Consistent with
  `dossier-factories`.
- **"Claude Code users approve without reading."** A secondary blog says "93% of prompts without
  reading them"; `cc-automode` says only "approve". **Resolved: the primary does not say "without
  reading".** The 13.6% catch rate (`cc-autodefault`) is the closer evidence.

## Conflicts left open

- **Approval rate: 93% or 97%?** `cc-automode` and `cc-contain` (93%) vs `cc-autodefault` (97%), all
  Anthropic; populations and periods unstated.
- **Authorities vs vendors on who decides approval.** `five-eyes`: "not delegated to the agentic AI
  system"; `atlas`: "final adjudication… by a human decision-maker"; `owasp-llm06`: "rather than
  relying on an LLM to decide". Claude Code auto mode (the default since 2.1.283), Codex `auto_review`,
  Cursor Auto-review (the default since 3.6) and Cline's model-set flag delegate it to models. The
  vendors' own measurements (`cc-autodefault`, `oai-autoreview`) say models catch more than humans
  do. Not reconciled; it is the central disagreement in the field.
- **Claude Code on Windows.** `cc-sandbox-docs`: "Native Windows is not supported"; `cc-binary`:
  "Sandboxed bash on Windows requires Git Bash"; `srt` ships a Windows alpha. Possibly gated.
- **CVE-2025-66479 scope.** GHSA: sandbox-runtime < 0.0.16, "low"; `guan-socks`: Claude Code
  2.0.24–2.0.55.
- **CVE-2025-61260 fix version.** Check Point says fixed in 0.23.0; GHSA/NVD say "v0.23.0 and before"
  vulnerable.
- **CVE-2026-45033 severity.** Repo GHSA medium; global GHSA high.
- **Cursor and Kiro advisory versions** contradict their own patched versions (`cve-cursor`,
  `cve-others`).
- **Cursor Linux fallback.** `cursor-docs` says unsupported kernels fall back to asking; the same page
  lists a bubblewrap fallback.
- **Copilot proxy enforcement.** Two `copilot-docs` pages disagree on which OS enforces the proxy.
- **Ubuntu userns mechanism.** The 2023 blog (creation gated) vs 24.04 notes (creation allowed,
  capabilities denied) (`ubuntu-userns`).
- **Codex Landlock.** Docs keep a `landlock` alias; the linux-sandbox README says the legacy path "is
  rejected" for filesystem-restricted policies.
- **Static vs adaptive defence results.** `firewalls` ("perfect security") vs `nasr-adaptive` (>90%
  bypass). Different defences and attack models.
- **Corpus hook share.** 52% of sampled Claude files have hooks vs a 24% PreToolUse index ratio; best-
  match ranking may favour longer files (`corpus-claude`).

## Explicitly unverified

- **Claude Code's two protected-path lists.** The permission layer protects more paths (`.husky`,
  `.envrc`, `.npmrc`, `.devcontainer`, `.pre-commit-config.yaml`, `.cargo`) than the sandbox write-deny
  list. With `autoAllowBashIfSandboxed: true`, a sandboxed script could plausibly write one of the
  former without a prompt or a kernel deny. **Inferred from docs and binary strings; not tested.**
- **Kilo's writable `bin` directory** holding LSP binaries the host spawns unsandboxed (`kilo-src`) —
  inferred planted-binary path, not tested. Likewise `tar --checkpoint-action` / `rg --pre` under the
  default agent allowlist.
- **Gemini seccomp x32-ABI bypass** — plausible, untested.
- **Gemini "YOLO enables the sandbox by default"** — documented; no such logic found in
  `sandboxConfig.ts`.
- **The SOCKS5 null-byte bypass** (`guan-socks`) — single researcher; vendor issued no advisory.
- **Zed's terminal sandbox** shipping status; **Warp** docs partly 404; **Cursor/Copilot CLI/Amp/
  Continue** behaviour is docs-only.
- **Cohere Terrarium CVE-2026-61522 and smolagents CVE-2026-10613** — not in NVD.
- **MSB (MCP Security Bench) 75.83% peak ASR** — snippet only.
- **Mythos Preview's evaluation-sandbox escape incident** — secondary accounts only.
- **Anthropic's 1,053-tester study design** (`cc-autodefault`) — unpublished; the 13.6% / 89% figures
  are vendor-only.
- **No peer-reviewed controlled study** of human approval accuracy for coding-agent actions exists in
  this run's search.
- **No independent academic evaluation** of Seatbelt, bubblewrap or Landlock agent sandboxes was found;
  the evidence is advisories and researcher reports.
- **Whether any credential found in `corpus-claude-local` is still valid** — not tested.
- **Codex `on-failure` approval value** still appearing in configs — possibly deprecated.
- **Model names** (Opus 5.5, Mythos Preview, GPT-5.6, Gemini 3.5 Flash Lite, Claude 4.5 Haiku) as the
  sources state them.
