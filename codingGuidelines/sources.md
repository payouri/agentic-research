# Agentic coding guidelines source lexicon

Every source behind [guide.md](guide.md) and [rulebook.md](rulebook.md), keyed for citation.
Gathered 2026-10-02.

Trust tiers: **P1** primary spec, vendor documentation, or shipped source code · **P2** peer-reviewed
or arXiv research · **P3** vendor engineering blog / industry research with disclosed method · **P4**
practitioner report with measurement (including this dossier's own counts, method stated) · **P5**
opinion, anecdote, news, or unverified secondary.

Source code was read from shallow clones at the commit stated. **[ABS]** marks a number taken from an
abstract only. **(WF)** marks a quote that passed through a summarising fetcher and may be lightly
paraphrased; the headline quotes in guide §1 and §3 (compliance-rules, slopcodebench, ts-any,
sa-feedback, sec-degradation, cursor-did, code-health, swe-agent) and the Claude Code and OpenAI
harness quotes were re-fetched raw and checked verbatim during reconciliation. openai.com returned
HTTP 403 throughout; the harness-engineering post was read via the r.jina.ai mirror and a Wayback
snapshot (2026-09-24).

Sibling dossiers cited by key: `dossier-agentsmd` ([agentsMd/](../agentsMd/)), `dossier-git`
([gitGuardrails/](../gitGuardrails/)), `dossier-factories` ([softwareFactories/](../softwareFactories/)).

---

## 1. Primary — what vendors and tool authorities say

### Anthropic

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `cc-memory` | [Claude Code memory (CLAUDE.md)](https://code.claude.com/docs/en/memory) | P1 | fetched 2026-10-02 | CLAUDE.md is "context, not enforced configuration. To block an action regardless of what Claude decides, use a PreToolUse hook instead"; "Settings rules are enforced by the client regardless of what Claude decides to do. CLAUDE.md instructions… are not a hard enforcement layer"; "Use 2-space indentation" as the example good instruction; "Path-scoped rules trigger when Claude reads files matching the pattern, not on every tool use" |
| `cc-best` | [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) (old anthropic.com/engineering URL redirects here) | P1 | fetched 2026-10-02 | "Give Claude a check it can run: tests, a build, a screenshot to compare"; "If you can't verify it, don't ship it"; CLAUDE.md "advisory", hooks "deterministic and guarantee the action happens"; "If you could describe the diff in one sentence, skip the plan"; "address the root cause, don't suppress the error"; "Chasing every finding leads to over-engineering"; code-intelligence plugin for "automatic error detection after edits"; no TDD section |
| `ant-best-2025` | [Claude Code: Best practices for agentic coding (archived)](https://web.archive.org/web/20250602202220/https://www.anthropic.com/engineering/claude-code-best-practices) | P3 | 2025-04-18 | TDD as "an Anthropic-favorite workflow" — since removed |
| `cc-large` | [Large codebases](https://code.claude.com/docs/en/large-codebases) | P1 | fetched 2026-10-02 | "Per-subdirectory CLAUDE.md… In a monorepo that's one per package" |
| `cc-hooks` | [Hooks reference](https://code.claude.com/docs/en/hooks), [Hooks guide](https://code.claude.com/docs/en/hooks-guide) | P1 | fetched 2026-10-02 | "treats exit code 1 as a non-blocking error and proceeds with the action"; "Any other exit code doesn't block on its own for most hook events"; PostToolUse exit 2 "Shows stderr to Claude; the tool already ran"; `decision:"block"` adds the reason, "Claude still sees the original output"; Stop hook continuation cap 8 (`CLAUDE_CODE_STOP_HOOK_BLOCK_CAP`) |
| `cc-plugins` | [Plugins reference](https://code.claude.com/docs/en/plugins-reference), code-intelligence plugins; npm 2.1.287 | P1 | 2026-10-01 | LSP diagnostics only via plugin with a separately installed binary; "In cloud sessions, Claude Code doesn't start plugin language servers, so Claude gets no diagnostics" |
| `cc-review` | [Code Review](https://code.claude.com/docs/en/code-review) (research preview) | P1 | fetched 2026-10-02 | "The check run always completes with a neutral conclusion so it never blocks merging through branch protection rules"; CLAUDE.md violations as "nit-level" findings; REVIEW.md rules "land more reliably"; example skips "Anything CI already enforces: lint, formatting, type errors" |
| `ant-tools` | [Writing effective tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents) | P3 | 2025-09-11 | "prompt-engineer your error responses to clearly communicate specific and actionable improvements, rather than opaque error codes or tracebacks" |
| `ant-sdk` | [Building agents with the Claude Agent SDK](https://claude.com/blog/building-agents-with-the-claude-agent-sdk) (redirected) | P3 | 2025-09-29 | "Code linting is an excellent form of rules-based feedback"; better "to generate TypeScript and lint it than… pure JavaScript" |
| `ant-harness` | [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents); [Harness design for long-running apps](https://www.anthropic.com/engineering/harness-design-long-running-apps) | P3 | 2025-11-26; 2026-03-24 | "only one feature at a time"; "It is unacceptable to remove or edit tests"; "Out of the box, Claude is a poor QA agent"; "Separating the agent doing the work from the agent judging it proves to be a strong lever" |

### OpenAI

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `oai-harness` | Ryan Lopopolo, [Harness engineering: leveraging Codex in an agent-first world](https://openai.com/index/harness-engineering/) (403; read via r.jina.ai and Wayback 20260924171239) | P3 | 2026-02-11 | "By enforcing invariants, not micromanaging implementations"; layers "enforced mechanically via custom linters… and structural tests"; "we statically enforce structured logging, naming conventions for schemas and types, file size limits… with custom lints"; "we write the error messages to inject remediation instructions into agent context"; "When documentation falls short, we promote the rule into code"; "Types → Config → Repo → Service → Runtime → UI"; "an early prerequisite"; "minimal blocking merge gates"; short AGENTS.md (~100 lines) as "a map" |
| `codex-best` | [Codex best practices](https://learn.chatgpt.com/guides/best-practices) (308 from developers.openai.com/codex/learn/best-practices) | P1 | fetched 2026-10-02 | "Not letting the agent see its work" as a mistake; "Goal / Context / Constraints / Done when"; plan "If the task is complex, ambiguous"; "When Codex makes the same mistake twice, ask it for a retrospective and update AGENTS.md" |
| `codex-hooks` | [Codex hooks](https://learn.chatgpt.com/docs/hooks) (308 from developers.openai.com/codex/hooks) | P1 | fetched 2026-10-02 | PostToolUse exit 2 replaces the tool result with feedback; Stop `decision:"block"` "automatically creates a new continuation prompt"; "a PreToolUse callback error, timeout, or malformed response can fail the hook without blocking the tool" |
| `codex-review` | [Codex in GitHub](https://learn.chatgpt.com/docs/third-party/github); Wayback 2026-01-02 | P1 | fetched 2026-10-02 | "Codex flags only P0 and P1 issues"; rules under `## Code Review Rules` (formerly `## Review guidelines`, closest file only) |
| `codex-execplans` | Aaron Friel, [Using PLANS.md for multi-hour problem solving](https://developers.openai.com/cookbook/articles/codex_exec_plans) | P3 | 2025-10-07 | "Every ExecPlan must be fully self-contained"; "Validation is not optional." |

### Google, GitHub, Cursor

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `gemini-prompt` | [gemini-cli `packages/core/src/prompts/snippets.ts`](https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/prompts/snippets.ts) | P1 (prompt, not guaranteed behaviour) | last commit 2026-09-29 | "Validation is the only path to finality"; reproduce bugs with a test first; "NEVER use hacks like disabling or suppressing warnings, bypassing the type system (e.g.: casts in TypeScript)" |
| `gemini-md` | Gemini CLI docs, GEMINI.md | P1 | 2026-06-18 | Example "Use 2 spaces for indentation" |
| `gemini-hooks` | Gemini CLI `docs/hooks/reference.md` @ fb972b2 | P1 | v0.62.0 | AfterTool exit 2 "Hides the tool result… The turn continues"; "`Other`: Warning" (contradicted by code, see `src-gemini-hooks`); AfterAgent exit 2 retries |
| `conductor` | [Conductor workflow.md](https://github.com/gemini-cli-extensions/conductor/blob/main/skills/conductor-setup/assets/workflow.md) | P1 | 2026-07-14 | "The Plan is the Source of Truth"; "CRITICAL… Do not proceed until you have failing tests"; ">80% code coverage" |
| `jules` | [Jules docs](https://jules.google/docs/) | P1 **(WF)** | undated | "Always include commands to install packages, run linters, or execute tests"; plan review before changes |
| `gh-best` | [Get the best results from Copilot cloud agent](https://docs.github.com/en/copilot/tutorials/coding-agent/get-the-best-results) (old best-practices URL 404) | P1 | fetched 2026-10-02 | "If Copilot is able to build, test and validate its changes… more likely to produce good pull requests"; "Complete acceptance criteria"; "Instructions must be no longer than 2 pages" |
| `gh-env` | [Customize the agent environment](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-environment) | P1 | fetched 2026-10-02 | "The job MUST be called `copilot-setup-steps` or it will not be picked up" |
| `gh-validation` | [Changelog: configure Copilot coding agent's validation tools](https://github.blog/changelog/2026-03-18-configure-copilot-coding-agents-validation-tools) | P1 | 2026-03-18 | Runs "your project's tests and linter" plus CodeQL, secret scanning; "enabled by default" |
| `gh-hooks` | [About hooks](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-hooks) | P1 | fetched 2026-10-02 | preToolUse "can approve or deny"; postToolUse no blocking capability |
| `gh-review` | [About Copilot code review](https://docs.github.com/en/copilot/concepts/agents/code-review); github/docs commits `cf11c1c3e6` (2026-02-26), `f0946763` (2026-06-15); community #178108 | P1 | 2026-06-15 | Reads instructions from the head branch; 4,000-character note restored then removed; staff: instruction "may not work" to "deterministically change its behavior"; gating via rulesets |
| `gh-review-tut` | [Using custom instructions for code review](https://docs.github.com/en/copilot/tutorials/use-custom-instructions) | P1 | fetched 2026-10-02 | "may not follow every instruction perfectly every time"; example "Limit line length to 88 characters" |
| `cursor-rules` | [Cursor Rules](https://cursor.com/docs/rules) | P1 | fetched 2026-10-02 | "Copying entire style guides: Use a linter instead"; "AI guidance should not be your only security control" |
| `cursor-blog` | [Best practices for coding with agents](https://cursor.com/blog/agent-best-practices) | P3 | 2026-01-09 | "Use typed languages, configure linters, and write tests"; "Not every task needs a detailed plan"; TDD steps; stop hook "keeps an agent working until all tests pass" |
| `cursor-hooks` | [Cursor Hooks](https://cursor.com/docs/agent/hooks) | P1 | fetched 2026-10-02 | Exit 2 deny; `afterFileEdit` no output fields; `postToolUse` `additional_context`; Stop "default limit is 5 auto follow-ups per script, configurable via the `loop_limit`" |
| `cursor-bugbot` | [Bugbot](https://cursor.com/docs/bugbot) | P1 | fetched 2026-10-02 | Findings "default to `neutral`"; "project rules (*.mdc…) do not apply to Bugbot runs"; rule truncated at 30,000 chars, 100,000 combined; yaml from base ("a PR cannot change how Bugbot reviews itself") |
| `cursor-forum-lint` | [Cursor forum, staff answer (Colin)](https://forum.cursor.com/t/168705) | P1 (staff) | 2026-08-28 | Agent "is instructed to check files it just edited and fix errors it introduced (capped at roughly 3 fix iterations per file)"; "This only works in the IDE… the CLI and Cloud Agents don't run an extension host at all" |
| `cursor-forum-rules` | Cursor forum topics 150931, 154389, 163738, 167098, 168224 | P1 (staff) | 2026-02-11 vs 2026-08-13 | Staff: glob rules trigger on "reads or edits" (Feb) vs "If the agent edits or creates the file without opening it first, the rule never gets pulled in" (Aug) |

### Other agent vendors and tool authorities

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `factory-linters` | Alvin Sng, [Using Linters to Direct Agents](https://factory.com/news/using-linters-to-direct-agents) (308 from factory.ai); [Factory-AI/eslint-plugin](https://github.com/Factory-AI/eslint-plugin) e713ca4 | P3 | 2025-09-05 | "Linters can encode your architecture, boundaries, and ergonomics directly into the code generation loop"; "Precise messages and autofixes let agents self-correct until green"; "Prefer named exports instead of default exports"; "Keep the file structure predictable"; "'lint green' as the merge gate"; plugin README "Our recommendation is NOT to simply import this package" |
| `factory-readiness` | [Introducing Agent Readiness](https://factory.com/news/agent-readiness) | P3 | 2026-01-20 | First pillar "Style & Validation: Linters, type checkers, formatters"; "The agent is not broken. The environment is." |
| `kiro-specs` | [Kiro specs](https://kiro.dev/docs/specs/), [correctness](https://kiro.dev/docs/specs/correctness/) | P1 | 2026-08-27 | requirements/design/tasks; example tests "limited by their own biases" → property-based testing |
| `kiro-exit` | [Kiro hooks](https://kiro.dev/docs/hooks/) (2026-09-30), [exit codes](https://kiro.dev/docs/reference/exit-codes) (2026-09-17) | P1 | 2026-09-30 | "Enforce standards - run linters, formatters, or type checks automatically after agent file changes"; exit 2 "(PreToolUse only) Block tool execution" |
| `windsurf-docs` | [Cascade](https://docs.devin.ai/desktop/cascade) (307 from docs.windsurf.com) | P1 | fetched 2026-10-02 | Lint auto-fix "This is turned on by default" |
| `windsurf-hooks` | [Cascade hooks](https://docs.devin.ai/desktop/cascade/hooks) | P1 | fetched 2026-10-02 | "Only **pre-hooks** … can block actions using exit code 2" |
| `rustc-llm` | [rustc-dev-guide LLM guidance](https://rustc-dev-guide.rust-lang.org/llm-guidance/writing.html) | P1 (project policy) | 2026-09-02 | "If a more reliable tool, such as a linter or formatter, already exists… we strongly suggest using that tool"; prefer ast-grep for mass rewrites |
| `codescene-blog` | Adam Tornhill, [Agentic AI Coding: Best Practice Patterns](https://codescene.com/blog/agentic-ai-coding-best-practice-patterns-for-speed-with-quality) | P3 (vendor) | 2026-02-20 | "aim for a Code Health of at least 9.5, ideally a perfect 10.0"; "Low Code Health increases the likelihood that agents fail" |
| `humanlayer` | Kyle, [Writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md) | P5 | 2025-11-25 | Origin of "Never send an LLM to do a linter's job" — not Anthropic |
| `eslint-mcp` | [ESLint MCP server](https://eslint.org/docs/latest/use/mcp); `@eslint/mcp` 0.3.13 (eslint/rewrite 9421376) | P1 | fetched 2026-10-02 | Single `lint-files` tool; "you must ask the user for confirmation before attempting to fix" |
| `arch-tools` | READMEs: [ArchUnit](https://github.com/TNG/ArchUnit), [dependency-cruiser](https://github.com/sverweij/dependency-cruiser), [import-linter](https://github.com/seddonym/import-linter), [Nx enforce-module-boundaries](https://nx.dev/docs/features/enforce-module-boundaries), eslint-plugin-boundaries, [ast-grep prompting](https://ast-grep.github.io/advanced/prompting.html) | P1 | fetched 2026-10-02 | What each claims to enforce; ArchUnit `.because()`, eslint-plugin-boundaries message templates, ast-grep `note`; none makes agent-specific claims; ast-grep: an AGENTS.md prompt "may not utilize it as instructed" |
| `qlty` | qltysh/qlty v0.649.0 `slop-one` | P1 | 2026-10-02 | Pre-push block on maintainability drop with "a refactoring prompt a coding agent can act on" |

---

## 2. Implementations — what the harnesses and tools actually do

| Key | Source | Tier | Date / SHA | What it settles |
|---|---|---|---|---|
| `src-aider` | Aider-AI/aider `aider/args.py:543-558`, `aider/linter.py`, `base_coder.py` | P1 | 5dc9490 (v0.86.0, 2025-08-09); [docs](https://aider.chat/docs/usage/lint-test.html) | `--auto-lint` `default=True`, `--auto-test` False; built-in lint is tree-sitter syntax only; TypeScript skipped (#1132); "Attempt to fix lint errors?"; `max_reflections = 3` |
| `pr-opencode-22997` | anomalyco/opencode (sst/opencode redirects) PR #22997 by thdxr | P1 | merged 2026-04-17 | "`lsp` unset: all built-in LSPs disabled"; "`formatter` unset: all formatters disabled" |
| `opencode-docs` | [opencode.ai/docs/lsp](https://opencode.ai/docs/lsp), /docs/formatters; source `packages/opencode/src/lsp/diagnostic.ts` @ 1ddb087 | P1 | 2026-10-02 | "LSP is disabled by default."; when on: errors only (`severity === 1`), `MAX_PER_FILE = 20`, 5 other files, "LSP errors detected in this file, please fix:" |
| `src-kilo` | Kilo-Org/kilocode `packages/core/src/v1/config/config.ts`, settings docs | P1 | a10fa8e | Inherits OpenCode; "**LSP integration**" under *Experimental* |
| `src-crush` | charmbracelet/crush `internal/agent/tools/diagnostics.go`, `internal/config/config.go:472` | P1 | 73d1a50 (v0.97.1) | `auto_lsp` default true; errors + warnings, 10 per block; "Only `PreToolUse` is currently supported"; exit 49 halts |
| `src-cline` | cline/cline `README.md:136`, `sdk/.../executors/editor.ts`, `responses.ts:267-300` | P1 | c269dbb (4.1.22) | Editor returns `` `Edited ${filePath}\n${diff}` ``; problem templates uncalled (grep, INFERRED no other path); README still claims lint monitoring |
| `src-roo` | RooCodeInc/Roo-Code `DiffViewProvider.ts:250-268` | P1 | b867ec9 (archived 2026-05-15) | `diagnosticsEnabled ?? true`; new errors only, "warnings can be distracting" |
| `src-codex-prompt` | openai/codex `codex-rs/core/gpt_5_2_prompt.md:142-149` | P1 | 14a477e (rust-v0.160.0) | "hold off on running tests or lint commands until the user is ready" in untrusted/on-request modes; formatting fixes "up to 3 times" |
| `src-codex-hooks` | openai/codex `codex-rs/hooks/src/events/post_tool_use.rs`, `features/src/lib.rs` | P1 | 14a477e | `Some(2) ... should_block = true` only if `can_apply_control_effects()`; empty stderr → "exited with code 2 but did not write feedback to stderr"; hooks `default_enabled: true` |
| `src-gemini-hooks` | google-gemini/gemini-cli `packages/core/src/hooks/hookRunner.ts:537-560` | P1 | fb972b2 (v0.62.0) | "All other non-zero exit codes (including 2) are blocking" on the plain-text path — contradicts its docs |
| `src-sweagent` | SWE-agent/SWE-agent `config/default.yaml`, `tools/edit_anthropic/bin/str_replace_editor`, `tools/windowed_edit_linting/bin/edit` | P1 | 3ea751c | Default `USE_LINTER = REGISTRY.get("USE_LINTER", "false")`; legacy guard: "Your changes have NOT been applied." |
| `src-openhands` | OpenHands/software-agent-sdk `openhands-tools/.../file_editor/editor.py:192` | P1 | cc97bf2 | Docstring "enable_linting: Whether to run linting on the changes"; no such parameter |
| `src-vscode` | microsoft/vscode `computeAutomaticInstructions.ts`; microsoft/vscode-docs c642b58 | P1 | 6ac40e9 | Docs: `applyTo` on files the agent "creates or modifies"; code matches attached files, prepends `**/` |
| `src-continue` | continuedev/continue | P1 | 5522c6f | Directory match by `includes(`${dirName}/`)`; CLI drops glob and regex rules |
| `cc-issue-19009` | [anthropics/claude-code#19009](https://github.com/anthropics/claude-code/issues/19009) | P1 (issue) | 2026-01-18, closed not planned | UI "PostToolUse:Edit hook blocking error" while "the edit **succeeds**" |
| `cc-issue-96361` | [anthropics/claude-code#96361](https://github.com/anthropics/claude-code/issues/96361) | P1 (issue) | 2026-09-23, open | "Creating a new file with `Write` loads neither" |
| `cc-issue-88565` | [anthropics/claude-code#88565](https://github.com/anthropics/claude-code/issues/88565) | P1 (issue) | 2026-08 | Bash/sed edits in auto mode never trigger path rules; related #95526 (any-depth match), #87217 (user-scope `paths:` disputed across versions) |
| `src-semgrep` | semgrep/semgrep `post_tool.py` @ 31729a1; semgrep/guardian eee2e50; [Guardian docs](https://docs.semgrep.dev/semgrep-guardian/overview) | P1 | 2.5.4 | Blocks in Claude Code and Codex; "Cursor's afterFileEdit hook does not support any outputs. There is no way to communicate the scan result to Cursor"; "The agent decides whether to regenerate" |
| `src-oxlint` | oxc-project/oxc PR #22068 | P1 | cda8eb4, 2026-05-17 | `OutputFormat::Agent` auto-selected by `is_agent()` (CLAUDECODE, GEMINI_CLI, CODEX_SANDBOX, OPENCODE…); "one line per diagnostic, no source excerpts, no summary" |
| `src-depcruise` | sverweij/dependency-cruiser `src/report/error.mjs` | P1 | a210586 (v18.5.0) | Rule `comment` printed only by `err-long`, not default `err` |
| `src-nx` | nrwl/nx `get-agent-rules.ts`, enforce-module-boundaries; issue #37181 | P1 | 6db155f | No message field in `depConstraints`; generated agent rules omit boundaries; `nx mcp` has no boundary tool |
| `import-linter-364` | seddonym/import-linter #364, PR #366 | P1 | 2026-08-10 | `--no-logo` because "the logo ascii art gets added to the context" when used "as a guardrail by AI agents" |
| `review-bots` | Bugbot docs; github/docs; CodeRabbit docs; [Greptile docs](https://greptile.com/docs); Graphite; qodo-ai/pr-agent 8e5a9295; anthropics/claude-code-action 97c53473 (`restore-config.ts`) | P1 | fetched 2026-10-02 | Config branch per bot: head (Copilot, CodeRabbit) vs base (Bugbot yaml, claude-code-action CLAUDE.md, PR-Agent, Greptile auto-approve); PR-Agent `repo_context_max_lines` 500 |
| `coderabbit-schema` | CodeRabbit `schema.v2.json`, docs.coderabbit.ai | P1 | fetched 2026-10-02 | ~59 tools (ast-grep, ESLint with repo config, Ruff, Semgrep…); `path_instructions` 20,000 chars; `mode: error` blocks only with `request_changes_workflow` (default false) |
| `src-cc-review-plugin` | anthropics/claude-code `plugins/code-review` @ 52c76441; issues #94789, #91167 | P1 | 2026-10-02 | Reads CLAUDE.md only, never AGENTS.md; "do not run the linter to verify"; README scoring absent from the command file; silent no-comment success reported |

---

## 3. Corpus — what real repositories state and enforce

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `corpus` | This dossier's measurement: 57 active public repositories with agent instruction files, cloned at pinned SHAs (2026-09-20 to 2026-10-02); 1,295 coding/structure rules extracted verbatim, each classified (STRUCTURE / STYLE / PROCESS; MECHANIZABLE / JUDGEMENT) and statused (ENFORCED / PARTIAL / PROSE-ONLY / CONTRADICTED) with file:line evidence by 8 coders on one codebook; ~15 contradictions and exemplars re-checked by hand, all held; aggregates re-run during reconciliation. Git/PR/permission rules excluded (see `dossier-git`) | P4 | 2026-10-02 | 62% prose-only, 21% enforced, 15% partial, 2% contradicted; structure 77% prose; 45% of mechanizable rules prose; 0 judgement rules enforced; 266/273 enforced rules backed by CI; 7/57 repos with post-edit lint hooks, 0 blocking; 17/57 repos contradict their own config; "no `any`" stated in 10, enforced in 1; 48/57 have repo-specific lint, 43 with remediation text, 2 addressed to agents. Selection bias toward mature devtool/AI repos; no inter-coder agreement measured |
| `corpus-search` | GitHub code-search API, 18 paced queries; top-100 sample of `PostToolUse path:.claude filename:settings.json` classified by hand | P4 | 2026-10-02 ~12:50 CEST | ~67,328 `.claude/settings.json`-path files, ~16,352 mention `PostToolUse`; of 71 PostToolUse configs in the sample, 38 run a formatter/linter/typechecker/test, 11 of those swallow failures; 2,388 `.cursor/hooks.json`, 664 with `afterFileEdit`. Counts are files (forks, templates inflate); sample is best-match, not random |

Corpus exemplars and cautionary cases cited by sub-key (`corpus: <repo>`), each at its pinned SHA:

| Sub-key | Repository @ SHA | What it shows |
|---|---|---|
| posthog | [PostHog/posthog @ 463a6cd](https://github.com/PostHog/posthog/blob/463a6cd6f5eedbe207b56dd7191035637c73570c/AGENTS.md#L159) | `[lint: <id>]` / `[review]` tags (11 / 37); L276 "do not add `PreToolUse`, `PostToolUse`, or `Notification` hooks as they add latency and are fragile" |
| n8n | [n8n-io/n8n @ 0352a19](https://github.com/n8n-io/n8n/blob/0352a196542703a8af80a116c7f94973b6eb2cf2/AGENTS.md#L197) | "**NEVER use `any` type**" vs `nodes.ts:45` `'@typescript-eslint/no-explicit-any': 'off'`; `misplaced-n8n-typeorm-import` message; "every lint script runs with `--quiet`, so a `warn` enforces nothing" |
| bun | [oven-sh/bun @ bc7a813](https://github.com/oven-sh/bun/blob/bc7a813b10b6ef8accc00c931b9a501331ac8c5c/.claude/hooks/pre-bash-guard.js#L165) | CRITICAL prose → PreToolUse deny "use `bun bd test <file>`"; ~86 `disallowed-methods` with reasons; PostToolUse ends `process.exit(0)` |
| temporal | [temporalio/temporal @ 79c4676](https://github.com/temporalio/temporal/blob/79c467689b592727762da92c7e7a0d7bb2384595/.github/.golangci.yml#L56) | Lint message with fix and sanctioned `//nolint:forbidigo // <justification>`; 6 contradictions incl. `require.Eventually` advised in prose but banned in lint |
| mcp-ts-sdk | [modelcontextprotocol/typescript-sdk @ e16d277](https://github.com/modelcontextprotocol/typescript-sdk/blob/e16d27729ea2fae144a6d957fd0372d9eb0e6c58/CLAUDE.md#L41) | "2-space indentation" vs `.prettierrc.json` `"tabWidth": 4`; "Lowercase with hyphens" vs `unicorn/filename-case` camelCase |
| gitbutler | [gitbutlerapp/gitbutler @ ec23b46](https://github.com/gitbutlerapp/gitbutler/blob/ec23b46ac6f11d7cf5d9f4ccc4526b3462a3af7e/.github/copilot-instructions.md#L144) | "No trailing commas" under Prettier Config vs Prettier ^3.8.1 default `all` ([Prettier docs](https://raw.githubusercontent.com/prettier/prettier/main/docs/options.md): "Default value changed from `es5` to `all` in v3.0.0") |
| cloudflare | [cloudflare/workers-sdk @ f23dcb3](https://github.com/cloudflare/workers-sdk/blob/f23dcb32ca0df051eaaf0222086a612a615c80a3/AGENTS.md#L83) | "The checked-in configuration is authoritative"; highest enforced count (12/27) |
| strands | [strands-agents/sdk-python @ 563f57d](https://github.com/strands-agents/sdk-python/blob/563f57d7a387931760a2c3106014cd7c0db4c9bc/strands-py/AGENTS.md#L66) | "This guide does not re-list those — it covers the conventions a linter *cannot* check" |
| transformers | [huggingface/transformers @ 35924ec](https://github.com/huggingface/transformers/blob/35924ec379eec682bbdca219886e16eff4df8b09/utils/check_noisy_comments.py#L14) | Blocking check on "verbose, low-signal AI output" comments; copilot-instructions point to non-existent `make fixup` |
| calcom | [calcom/cal.com @ 54343aa](https://github.com/calcom/cal.com/blob/54343aa685ae8f33159d2f485ec4a57bad5c574a/agents/rules/testing-coverage-requirements.md#L12) | "This is enforced automatically in our CI pipeline" — no coverage threshold exists; "the linter will block it" — no feature import restriction |
| tldraw | [tldraw/tldraw @ 5b268fc](https://github.com/tldraw/tldraw/blob/5b268fc9711250805b36648d9062ede7b1a1565d/.claude/settings.json#L46) | Stop hook `pnpm exec prettier --write \|\| true` in an oxfmt repo; oxlint rule addressing agents ("must have a colocated `AGENTS.md`") |
| vscode | [microsoft/vscode @ 6ac40e9](https://github.com/microsoft/vscode/blob/6ac40e93521e5d9222fb09b6e1227dd2c85e915c/.github/hooks/build.json.disabled#L1) | Only agent check shipped disabled; "minimize their use" of builds |
| langchain | [langchain-ai/langchain @ ff46bb4](https://github.com/langchain-ai/langchain/blob/ff46bb478bfbef98dd35ea705fa4e92a33881142/AGENTS.md#L201) | ">20 lines" rule vs ruff ignores `PLR09`, `C90` |
| sentry-javascript | getsentry/sentry-javascript @ 9e77e9d | `max-lines` 300 and `complexity` 33 enforced, unmentioned to agents |
| others | twenty a28e8c8, crush 73d1a50, zed 7362739, github-mcp-server f89f884, adk-python f33343f, mastra 8a2a729, claude-code-action 97c5347 | Contradictions and over-claims listed in guide §5 |

---

## 4. Evidence — what has been measured

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `swe-agent` | Yang et al., [SWE-agent](https://arxiv.org/abs/2405.15793) v3 (NeurIPS 2024) | P2 | 2024-11-11 | Lint-gated edit 18.0% / plain 15.0% / no edit 10.3% (SWE-bench Lite, GPT-4 Turbo); 100-line viewer 18.0% vs full file 12.7%; "90.5% chance… drops off to 57.2% after a single failed edit" |
| `typed-decoding` | Mündler et al., [Type-Constrained Code Generation](https://arxiv.org/abs/2504.09246) v2 | P2 | 2025-05-08 | "94% of compilation errors result from failing type checks"; compile errors halved; correctness +3.5–5.5% relative; 2–34B open models |
| `ts-any` | Lee, Hassan, Hindle, [Mining Type Constructs in AI-Generated Code](https://arxiv.org/abs/2602.17955) | P2 | 2026-02-20 | "AI agents are 9x more prone to use the 'any' keyword"; agentic TS PRs accepted 45.8% vs 25.3% |
| `swe-polybench` | Rashid et al., [SWE-PolyBench](https://arxiv.org/abs/2504.08703) v3 | P3 | 2025-04-23 | TS beat JS in 6/8 configurations (confounded by repository) |
| `sa-feedback` | Blyth et al., [Static Analysis as a Feedback Loop](https://arxiv.org/abs/2508.14419) (SCAM 2025) [ABS] | P2 | 2025-08-20 | "security issues reduced from >40% to 13%, readability violations from >80% to 11%, and reliability warnings from >50% to 11% within ten iterations" |
| `sec-degradation` | Shukla et al., [Security Degradation in Iterative AI Code Generation](https://arxiv.org/abs/2506.11022) v2 [ABS] | P2 | 2025-09-26 | "37.6% increase in critical vulnerabilities after just five iterations" without tool feedback |
| `core` | Wadhwa et al. (Microsoft), [CORE](https://arxiv.org/abs/2309.12938) [ABS] | P2 | 2023-09-22 | 59.2% of Python files revised across 52 checks |
| `compliance-rules` | Yang, He, Zhou, [A First Look at Coding Agents' Compliance with AI Contribution Rules](https://arxiv.org/abs/2607.26819) | P2 | 2026-07-29 | "proactively read rule files (excluding always-loaded AGENTS.md) in merely 3.5% of the runs"; "Of the 248 Native violations, 242 (97.6%) happened without the policy ever being opened"; refusal/handoff 0%; verifier feedback lifts disclosure 81–97%, hold-back rules 0–33% |
| `slopcodebench` | Orlanski et al., [SlopCodeBench](https://arxiv.org/abs/2603.24755) v2 [ABS] | P2 | 2026-05-07 | "2.3x more verbose and 2.0x more eroded"; guidance "reduces initial verbosity and erosion by up to a third, without affecting degradation rates" |
| `codeif` | Yan et al., [CodeIF](https://arxiv.org/abs/2502.19166) v3 (ACL 2025 Industry) **(WF)** | P2 | 2025-08-04 | Best CSR 0.414; "models often ignore global formatting rules, such as line limits, and inconsistently follow naming conventions" |
| `minimal-edits` | Zhu, Lim, Kan, [When Models Edit Too Much](https://arxiv.org/abs/2609.04061) (EMNLP 2026) [ABS] | P2 | 2026-09-03 | "preserve" instruction: excess edit distance 0.195→0.131, cognitive complexity −26.6%, Pass@1 +2.3 |
| `swe-live` | Zhang et al., [SWE-bench Goes Live!](https://arxiv.org/abs/2505.23419) v2 **(WF)** | P3 | 2025-06-01 | Under 100 files "often… above twenty per-cent", over 500 files "rarely exceed five per-cent"; "Patches that touch seven or more files are never solved"; 48% for single-file <5-line patches |
| `swe-pro` | Deng et al., [SWE-Bench Pro](https://arxiv.org/abs/2509.16941) v2 **(WF)** | P3 | 2025-11-14 | Sharp declines with file count; without requirements/interface specs GPT-5 25.9→8.4%, Opus 4.1 22.7→8.2% |
| `featbench` | Chen, Li, Li, [FeatBench](https://arxiv.org/abs/2509.22237) v2 **(WF)** | P3 | 2026-02-18 | "Regressive implementation… 73.6% of analyzed failure cases"; "'scope creep'"; 60–70% small repos vs 10–30% large |
| `difficulty` | Al-Haque, Johnson, [What Makes Issue Resolution Tasks Difficult](https://arxiv.org/abs/2608.18280) [ABS] | P2 | 2026-08-18 | AUC 0.863; "Patch fragmentation" and "Repository scale" |
| `code-health` | Borg, Hagatulah, Tornhill, Söderberg, [Code for Machines, Not Just Humans](https://arxiv.org/abs/2601.02200) (FORGE 2026; CodeScene co-authors) | P2 (vendor co-authored) | 2026-01-05 | 5,000 competitive-programming Python files; break-rate risk reduction "over 30%" (Qwen); "Claude Sonnet showed no significant difference" |
| `codescene-pr` | CodeScene press release, [PRNewswire](https://tools.prnewswire.com/en-us/live/20813/release/20260128EN71904) | P5 | 2026-01-28 | Reframes as "increase defect risk by at least 30 percent" — not what the paper measured |
| `modularity` | Kang, Seo, Kim, [Revisiting the Impact of Pursuing Modularity](https://arxiv.org/abs/2407.11406) (EMNLP Findings 2024) [ABS] | P2 | 2024 | "modularity is not a core factor" (function-level) |
| `doc-effect` | Macke, Doyle, [Testing the Effect of Code Documentation](https://arxiv.org/abs/2404.03114) (NAACL Findings 2024) [ABS] | P2 | 2024 | Incorrect docs "can greatly hinder"; missing docs do not significantly affect |
| `cursor-did` | He, Miller, Agarwal, Kästner, Vasilescu, [Speed at the Cost of Quality](https://arxiv.org/abs/2511.04427) v3 (MSR 2026) | P2 | 2026-01-26 | 806 vs 1,380 repos; "static analysis warnings increase significantly by 30.3%, and code complexity increases by 41.6%"; "A 100% increase in code complexity and static analysis warnings causes a 64.5% and 50.3% decrease in development velocity" |
| `less-reuse` | Huang et al., [More Code, Less Reuse](https://arxiv.org/abs/2601.21276) (MSR 2026) | P2 | 2026-01-29 | AMR 0.2867 vs 0.1532 (1.87×); redundancy subset is one repository (crewAI, 617 PRs) |
| `echoes` | Borg et al., [Echoes of AI](https://arxiv.org/abs/2507.00788) v3 (ICSME 2025) [ABS] | P2 | 2026-02-26 | 151 participants; "did not detect systematic maintainability advantages or disadvantages" |
| `ticoder` | Fakhoury et al., [TiCoder](https://arxiv.org/abs/2404.10100) (TSE 2024) [ABS] | P2 | 2024-10-02 | "average absolute improvement of 45.97% in the pass@1" |
| `tdflow` | Han et al., [TDFlow](https://arxiv.org/abs/2510.23761) v2 (EACL 2026) [ABS] | P2 | 2026-01-22 | 94.3% SWE-bench Verified with human tests; 7 test-hacking instances in 800 runs |
| `agent-tests` | Chen et al., [Rethinking the Value of Agent-Generated Tests](https://arxiv.org/abs/2602.07900) **(WF)** | P2 | 2026-02-08 | "changes in the volume of agent-written tests do not significantly change final outcomes"; ≤2.6 pp |
| `review-actions` | Sun et al., [Does AI Code Review Lead to Code Changes?](https://arxiv.org/abs/2508.18771) v2 | P2 | 2026-04-25 | 16 review Actions; 0.9–19.2% addressed |
| `human-ai-review` | Zhong et al., [Human-AI Synergy in Agentic Code Review](https://arxiv.org/abs/2603.15911) | P2 | 2026-03-16 | AI 16.6% vs human 56.5% adopted; adopted AI suggestions complexity +0.106 vs +0.003 |
| `coderabbit-study` | Lin et al., [Is Agentic Code Review Helpful?](https://arxiv.org/abs/2607.03316) v2 [ABS] | P2 | 2026-07-23 | 36.4% accepted |
| `rovodev` | Tantithamthavorn et al. (Atlassian), [RovoDev Code Reviewer](https://arxiv.org/abs/2601.01129) v2 (ICSE-SEIP 2026) | P2 | 2026-01-20 | 38.70% resolution; PR cycle −30.8%; human comments −35.6% |
| `autocommenter` | Vijayvergiya et al. (Google), [AutoCommenter](https://arxiv.org/abs/2405.13565) (AIware 2024) | P2 | 2024-05-22 | ~40% resolved; "For 33/50 (66%) of these best practices, violation detection is beyond the scope of traditional static analysis" |
| `beko` | Cihan et al., [Automated Code Review In Practice](https://arxiv.org/abs/2412.18531) (ICSE-SEIP 2025) | P2 | 2024-12-28 | 73.8% "Resolved"; closure time 5h52m → 8h20m |
| `coderabbit-report` | CodeRabbit, [State of AI vs human code generation](https://www.coderabbit.ai/blog/state-of-ai-vs-human-code-generation-report) | P4 (vendor) | 2025-12-17 | "~1.7× more issues"; 470 PRs, heuristic authorship, vendor-labelled |
| `veracode` | Veracode, [GenAI code security report](https://www.veracode.com/blog/genai-code-security-report/) | P4 (vendor) | 2025-07-30 | "45% of code samples failed security tests" on 80 curated tasks |
| `uplevel` | Uplevel Data Labs, [GenAI productivity](https://static.simonwillison.net/static/2025/uplevel-genai-productivity.pdf) (mirror) | P5 | 2024 | Origin of "+41% bug rate" — Copilot autocomplete era, bug rate undefined |
| `evidence-gap` | Evidence-axis search, 2026-10-02 | — | 2026-10-02 | Records absences: no 2025–2026 lint/typecheck-hook ablation in a modern harness; no measured rate of agent-added suppressions; no isolated file-length study; no study of architecture-check effect on agents; no study of remediation text vs bare errors |
| `dossier-git` | [gitGuardrails/](../gitGuardrails/) — arXiv:2605.10039 factorial study, ImpossibleBench, generated-test overfitting | P2 (via sibling) | 2026-09-10 | Instruction-file structure null; non-compliance rises with time on task; weak oracles are gamed |
| `dossier-agentsmd` | [agentsMd/](../agentsMd/) — PRIME (arXiv:2606.22470) | P2 (via sibling) | 2026-09-10 | Models "seldom recognize contradictions or request clarification" |

---

## Conflicts resolved

1. **"Never send an LLM to do a linter's job" is attributed to Anthropic.** The primary axis grepped
   Anthropic's docs and found it absent; it originates in HumanLayer's blog (2025-11-25). Recorded as
   `humanlayer` (P5) and the attribution called folklore.
2. **"Agents rarely read rule files" (3.5%).** The evidence report stated it as prose rules generally
   going unread. The full text scopes it: "rule files (excluding always-loaded AGENTS.md)". The guide
   states the scoped claim.
3. **SWE-agent "Invalid edits are discarded" vs current behaviour.** The paper (2024) and the
   implementation axis's read of `config/default.yaml` @ 3ea751c are both correct: the default changed.
   The guide cites the paper for the measurement and the source for the current default.
4. **OpenAI harness-engineering post.** The sibling `softwareFactories` dossier recorded it as P5,
   snippets-only via InfoQ, because openai.com returned 403. This run read the full text through a
   mirror and a Wayback snapshot, and the author and date (Ryan Lopopolo, 2026-02-11) were confirmed
   from the archived HTML; quotes were re-fetched during reconciliation. Upgraded here to P3 (vendor
   engineering blog).
5. **OpenCode "LSP feedback out of the box."** Commentary predating 2026-04-17 says so; PR #22997 and
   current docs say off by default. Resolved to the source.
6. **Copilot postToolUse.** The primary axis flagged that the docs describe it only for logging; the
   implementation axis confirmed it has no blocking capability. Agreement.
7. **Copilot code review 4,000-character instruction limit.** The primary axis could not find it in
   current docs; the implementation axis traced its removal in github/docs `f0946763` (2026-06-15)
   after an earlier restore. Resolved as "documented limit removed"; runtime behaviour stays
   unverified.
8. **Claude Code PostToolUse "blocking error."** The UI label (issue #19009) vs the docs: the docs are
   right — the edit lands.
9. **Gemini CLI exit codes.** Docs say other non-zero codes warn; `hookRunner.ts` @ fb972b2 blocks on
   any non-zero ≥2 with plain-text output. Resolved in favour of the code for behaviour; the docs are
   wrong.
10. **Style rules in prose.** Cursor and the Rust project say use a linter; Anthropic, Gemini and
    Copilot use style rules as their examples. Resolved on the corpus evidence (P4): every
    restatement checked had drifted, including Anthropic's own example in the MCP TypeScript SDK.
11. **Code health and defect risk.** CodeScene's press release ("increase defect risk by at least 30
    percent") vs its paper (refactoring break rate, competitive-programming code, no effect for
    Sonnet 4.5). Resolved to the paper.
12. **Anthropic on TDD.** "Anthropic-favorite workflow" (2025-04-18) vs current docs with no TDD
    section. Both dated and recorded; the guide treats the change as a softening.
13. **Corpus line reference.** The corpus report cited PostHog's tag definition at L158; it is at
    L159. Corrected.

## Conflicts left open

1. **Does agent assistance degrade maintainability?** Cursor DiD (+30.3% warnings, +41.6% complexity,
   persistent) and SlopCodeBench (erosion) vs the Echoes RCT (no maintainability effect; assistant,
   one Java feature). Different tools, setups and outcome measures.
2. **Agent security.** Veracode (45% insecure, curated) and the vibe-coded app audit (91% of apps)
   vs AIDev (agent PRs fewer security smells, OR 0.63) and SWE-bench Verified patches ("no new bugs or
   security vulnerabilities"). Greenfield generation vs scoped edits.
3. **Do structure and code health matter to the strongest models?** Significant for mid-size models,
   not for Sonnet 4.5 at n=1,000; modularity "not a core factor" at function level.
4. **Merge gates.** OpenAI: "minimal blocking merge gates". Factory: "'lint green' as the merge gate".
   CodeScene: strict coverage gates.
5. **Fix autonomously or ask?** OpenCode's "please fix", Cursor's ~3 iterations, Factory's
   "self-correct until green" vs ESLint MCP ("must ask the user"), Aider ("Attempt to fix lint
   errors?"), Codex's prompt ("hold off").
6. **Errors only or warnings too?** OpenCode and Roo (errors only) vs Crush and Claude Code (both).
7. **Lint message style.** zed's lint-creator skill: "Flag only; never suggest how to fix" vs OpenAI
   and 43 corpus repos with remediation text. No measurement either way (`evidence-gap`).
8. **Post-edit hooks at all.** PostHog forbids them as latency and fragility; six corpus repos commit
   them.
9. **Cursor glob rules on edit.** Two staff answers six months apart disagree; the later says read
   only.
10. **Review config from head or base.** A factual divergence between bots, not resolvable by source;
    recorded per bot.
11. **Review bots and cycle time.** Atlassian: −30.8% cycle time, −35.6% human comments. Beko: slower
    closure, no change in human comments.
12. **Why harnesses turned feedback loops off.** No source documents the reasoning for OpenCode's,
    SWE-agent's or Cline's change; latency and noise are inference.

## Explicitly unverified

- Whether Cline v4 has any diagnostics path outside the grepped code (read, not run).
- Whether the Copilot cloud agent finds tests and linters without `copilot-setup-steps.yml`.
- Junie: whether it runs IDE inspections after edits; its hook semantics beyond `blockOnError`.
- Devin: exact docs wording on running lint before commit (search snippet only).
- Kiro: what PostToolUse / PostFileSave / Stop shell-hook output does for the agent.
- Copilot code review's actual instruction cap after 2026-06-15; whether its ESLint/PMD integration
  (2025-11-20 changelog preview) still ships.
- Cursor's "Iterate on Lints" setting name and current default (forum and staff only).
- Gemini Code Assist `.gemini/styleguide.md` enforcement (summarised fetch only).
- 2605.29442: "38.33% Developer Constraint Violation" (search snippet only).
- 2607.18057: "2.6× deletion-to-addition ratio" for tests (snippet only).
- 2509.14745: human PR merge rate 91.0% vs Claude Code 83.8% (snippet for the human figure).
- 2504.09246 venue (PLDI 2025 likely, not confirmed).
- GitClear page date and headline (page "4x more code cloning" vs press "eightfold").
- Constraint Decay (2605.06445): 27.28 pp (v2) vs ~30 points (indexed version).
- Inter-coder agreement in the corpus classification (not measured).
- **Folklore, no primary source found:** "agents work better on files under 500 lines"; "agents
  routinely add `eslint-disable` / `@ts-ignore`" (no measured rate); "lint hooks make agents better"
  (no modern ablation); "AI code has 41% more bugs" (Uplevel, autocomplete era, undefined metric).
