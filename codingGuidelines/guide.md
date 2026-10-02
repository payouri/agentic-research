# Agentic coding guidelines: a guideline is a check you haven't written yet

Researched 2026-10-02. Covers how to shape a codebase and its coding rules so that coding agents
produce good code, which implementation practices measurably help or hurt, and how guidelines get
enforced — prose versus mechanism. Every claim carries a source key resolving in
[sources.md](sources.md); rules distilled from this guide are in [rulebook.md](rulebook.md).

This dossier builds on its siblings and does not re-cover them: the instruction file itself
([agentsMd/](../agentsMd/)), git rules and CI gates ([gitGuardrails/](../gitGuardrails/)), the test
oracle and unattended pipelines ([softwareFactories/](../softwareFactories/)), and context length
([contextSmartZone/](../contextSmartZone/)).

---

## 1. The finding that organises everything else

Every vendor that ships a coding agent says, in its own documentation, that prose instructions are
advisory and that only a mechanism enforces. Anthropic: CLAUDE.md is "context, not enforced
configuration. To block an action regardless of what Claude decides, use a PreToolUse hook instead"
[cc-memory]. GitHub: Copilot "may not follow every instruction perfectly every time" [gh-review-tut].
Cursor: "Copying entire style guides: Use a linter instead" [cursor-rules]. OpenAI's own account of
building a product with no human-written code puts the policy in a sentence: "When documentation
falls short, we promote the rule into code" [oai-harness].

Real repositories do the opposite. Across 57 active, agent-using repositories measured for this
dossier, 1,295 coding and structure rules are stated to agents, and **62% are prose only**; 21% are
enforced by anything that fails [corpus]. Structure rules are the least enforced: **77% prose-only**.
Of the 830 rules a stock tool could check, **45% are still prose** [corpus]. Seventeen of the 57
repos (30%) state a rule their own configuration contradicts [corpus].

And the evidence says the prose is mostly not doing the work. When the rule lives in a file the
agent must choose to open, it opens it in **3.5% of runs**, and 97.6% of violations happen without the
policy ever being read [compliance-rules]. When quality guidance is in the prompt, it "reduces
initial verbosity and erosion by up to a third, without affecting degradation rates"
[slopcodebench]. The single best-measured enforcement mechanism in the literature — a linter that
rejects a syntactically broken edit — lifted SWE-agent from 15.0% to 18.0% [swe-agent].

Then the twist. That mechanism is being switched off by the people who built it. SWE-agent's current
default configuration ships `USE_LINTER = "false"` [src-sweagent]. OpenCode disabled LSP diagnostics
and formatters by default in April 2026 [pr-opencode-22997]. Cline v4's editor returns only the
diff [src-cline]. The hooks that teams write to replace them run the formatter and swallow the
result: of 7 corpus repos with a post-edit lint hook, **0 block**, and in a broader sample 11 of 38
lint-running hooks explicitly discard failures [corpus].

So the shape of the problem is: everyone agrees the rule should be a check; almost no one has written
the check; the harnesses that used to run it for you are retreating; and the hooks people add instead
fix silently and never tell the agent. The rest of this guide is the evidence for each step, and the
advice follows from it: **every rule that can be a check should be one, its failure message should be
written for the agent, and it should gate where the agent cannot route around it — CI first, hooks
second, prose last.**

---

## 2. What the authorities actually say

### Every vendor says prose is advisory and mechanism enforces

The statements are unusually uniform. Anthropic's best-practices page distinguishes hooks, which are
"deterministic and guarantee the action happens", from CLAUDE.md instructions, which "are advisory"
[cc-best]. Its memory docs: "Settings rules are enforced by the client regardless of what Claude
decides to do. CLAUDE.md instructions… are not a hard enforcement layer" [cc-memory]. Cursor: "AI
guidance should not be your only security control" [cursor-rules]. GitHub staff, on review
instructions: an instruction "may not work" to "deterministically change its behavior" [gh-review].

OpenAI's harness-engineering post is the strongest version of the position and the only one that
extends it to architecture [oai-harness]:

> "By enforcing invariants, not micromanaging implementations, we let agents ship fast without
> undermining the foundation."

> "we statically enforce structured logging, naming conventions for schemas and types, file size
> limits, and platform-specific reliability requirements with custom lints. Because the lints are
> custom, we write the error messages to inject remediation instructions into agent context."

Factory says the same from the tooling side: "Linters can encode your architecture, boundaries, and
ergonomics directly into the code generation loop" and "Precise messages and autofixes let agents
self-correct until green" [factory-linters]. Anthropic's tool-design guidance arrives at the same
place for error text generally: "prompt-engineer your error responses to clearly communicate specific
and actionable improvements, rather than opaque error codes or tracebacks" [ant-tools].

A widely-quoted line — "Never send an LLM to do a linter's job" — is often attributed to Anthropic.
It is from a HumanLayer blog post [humanlayer], and appears nowhere in Anthropic's docs. The position
is right; the attribution is folklore.

### The one practice everyone mandates: a check the agent can run

Anthropic's best-practices page opens with it — "Give Claude a check it can run: tests, a build, a
screenshot to compare" — and names the failure pattern "If you can't verify it, don't ship it"
[cc-best]. Codex lists "Not letting the agent see its work" as a common mistake [codex-best]. Cursor:
"Use typed languages, configure linters, and write tests" [cursor-blog]. Copilot: "If Copilot is
able to build, test and validate its changes in its own development environment, it is more likely to
produce good pull requests" [gh-best]. Google's shipped Gemini CLI system prompt: "Validation is the
only path to finality" [gemini-prompt].

Anthropic's long-running-harness work adds why the check must not be the agent's own judgement:
"Out of the box, Claude is a poor QA agent", and "Separating the agent doing the work from the agent
judging it proves to be a strong lever" [ant-harness]. OpenAI's ExecPlans say it in four words:
"Validation is not optional." [codex-execplans]

What they leave open is *which* check, and where it runs. Anthropic offers a prompt, a Stop hook, a
subagent reviewer, or a skill and does not rank them [cc-best]. No vendor says what to do when the
check is expensive — and §4 shows harnesses disagree about whether to run it at all.

### Structure: only OpenAI and Factory prescribe architecture

OpenAI prescribes a fixed forward-only layering per domain — "Types → Config → Repo → Service →
Runtime → UI" — with cross-cutting concerns entering "through a single explicit interface:
Providers. Anything else is disallowed and enforced mechanically", and calls it the kind of
architecture "you usually postpone until you have hundreds of engineers. With coding agents, it's an
early prerequisite" [oai-harness]. Factory's agent-readiness model makes "Style & Validation:
Linters, type checkers, formatters" its first pillar, and its lint guidance favours searchability:
"Prefer named exports instead of default exports", "Keep the file structure predictable"
[factory-linters; factory-readiness].

Anthropic and Google prescribe no module structure. Anthropic's structural advice is about
instruction placement ("In a monorepo that's one per package") and about types: "If you work with a
typed language, install a code intelligence plugin to give Claude… automatic error detection after
edits" [cc-best; cc-large]. **No vendor gives a numeric file- or function-size limit.** OpenAI lints
"file size limits" without saying what they are [oai-harness]; CodeScene, a vendor, gives the only
numeric quality target — "aim for a Code Health of at least 9.5" [codescene-blog].

### Style rules in prose: the vendors contradict themselves

Cursor says not to copy style guides into rules [cursor-rules]; the Rust project's contributor
policy says "If a more reliable tool, such as a linter or formatter, already exists… we strongly
suggest using that tool" [rustc-llm]. But Anthropic's memory docs use "Use 2-space indentation" as
their example of a good instruction [cc-memory], Gemini CLI's GEMINI.md example lists "Use 2 spaces
for indentation" [gemini-md], and Copilot's tutorial uses "Limit line length to 88 characters"
[gh-review-tut].

The corpus settles which advice survives contact with a real repository. The Model Context Protocol
TypeScript SDK's CLAUDE.md states "**Formatting**: 2-space indentation" — Anthropic's exact example —
while its `.prettierrc.json` sets `"tabWidth": 4` [corpus: mcp-ts-sdk]. gitbutler's instructions
claim "No trailing commas" under "Prettier Config", while its Prettier 3 default is `all` and CI
enforces it [corpus: gitbutler]. Every restatement of lint config in prose that the corpus checked
had drifted (gitbutler, crush, github-mcp-server) [corpus]. A style rule in prose is a second source
of truth, and the second source is the one that rots.

### Test-first and plan-first: recommended, conditional, softening

Anthropic's April 2025 post called TDD "an Anthropic-favorite workflow" [ant-best-2025]; that URL
now redirects to a docs page with no TDD section, leaving "write a failing test that reproduces the
issue, then fix it" [cc-best]. Cursor's January 2026 post repeats the 2025 steps nearly verbatim
[cursor-blog]. Google's Conductor mandates it ("CRITICAL… Do not proceed until you have failing
tests") [conductor]. Kiro argues example tests are "limited by their own biases" and adds
property-based testing [kiro-specs].

Plan-first is universal and conditional: "If you could describe the diff in one sentence, skip the
plan" (Anthropic); "Not every task needs a detailed plan" (Cursor); plan "If the task is complex,
ambiguous" (Codex) [cc-best; cursor-blog; codex-best]. Conductor, Kiro specs and Jules plan approval
build it into the workflow instead [conductor; kiro-specs; jules].

### Review bots: none blocks by default

Claude Code Review: "The check run always completes with a neutral conclusion so it never blocks
merging through branch protection rules", and it "treats newly introduced [CLAUDE.md] violations as
nit-level findings" [cc-review]. Bugbot: findings "default to `neutral`" [cursor-bugbot]. Copilot code
review leaves gating to rulesets [gh-review]. Anthropic's own REVIEW.md example says to skip
"Anything CI already enforces: lint, formatting, type errors" [cc-review] — the review bot is
explicitly positioned *behind* the mechanical gate, not as one.

---

## 3. What the evidence supports

### Mechanism in the loop: the best-measured win is from 2024

SWE-agent's agent-computer-interface ablation remains the cleanest causal evidence that an
enforcement mechanism inside the loop helps: on SWE-bench Lite with GPT-4 Turbo, "edit w/ linting"
18.0%, plain edit 15.0%, no edit command 10.3% [swe-agent]. The mechanism matters because recovery
collapses after a failure: "any attempt at editing has a 90.5% chance of eventually being successful.
This probability drops off to 57.2% after a single failed edit" [swe-agent]. The same paper found a
100-line file viewer beat showing the full file (18.0% vs 12.7%) — an interface result, not a
file-length one.

That is the whole causal record for lint-in-the-loop on agents. **No 2025–2026 ablation of a lint or
type-check hook in a modern harness was found** [evidence-gap]. Everything since is either
generation-time (types) or iteration-loop (static analysis) work on narrower setups.

### Types: compile errors are type errors, and agents reach for `any`

On TypeScript generation, "on average 94% of compilation errors result from failing type checks";
type-constrained decoding "reduces compilation errors by more than half and increases functional
correctness relatively by 3.5% to 5.5%" [typed-decoding]. Given latitude, agents escape the type
system: in TypeScript PRs, "AI agents are 9x more prone to use the 'any' keyword" than humans
[ts-any]. Anthropic's Agent SDK guidance draws the practical conclusion: "it is usually better to generate
TypeScript and lint it than… pure JavaScript" [ant-sdk]. Language comparisons are confounded — TypeScript beat JavaScript in 6 of 8 SWE-PolyBench
configurations on different repositories [swe-polybench] — so the supportable claim is narrower than
"types help agents": *the type checker catches most of what goes wrong, and agents will disable it if
you let them.*

The corpus shows the gap exactly where the evidence points. "Never use `any`" is stated in 10 repos,
**fully enforced in 1**, and contradicted in 2 — n8n turns `@typescript-eslint/no-explicit-any` off
for a whole package while its AGENTS.md says "**NEVER use `any` type**" [corpus: n8n].

### Static-analysis feedback improves code; unguided iteration degrades it

With Bandit and Pylint findings fed back, GPT-4o's "security issues reduced from >40% to 13%,
readability violations from >80% to 11%, and reliability warnings from >50% to 11% within ten
iterations" [sa-feedback]. Iterative refinement *without* tool feedback produced "a 37.6% increase in
critical vulnerabilities after just five iterations" [sec-degradation]. Microsoft's CORE revised 59.2%
of Python files across 52 checks [core]. Both loop studies are abstract-level and single-shot code,
not agents in repositories — but they point the same way and the direction is the point: iteration
is only as good as the oracle it iterates against, which [gitGuardrails/](../gitGuardrails/) found for
tests too.

### Prose rules: unread, fading, occasionally effective

Three measurements, three different setups:

- **Unread.** Agents "proactively read rule files (excluding always-loaded AGENTS.md) in merely 3.5%
  of the runs"; refusal and handoff rules sit at 0% compliance for every agent tested. Verifier
  feedback lifted disclosure to 81–97% — but rules that ask the agent to *hold back* reached only
  0–33% even with feedback [compliance-rules]. Note the scope: this is policy files the agent must
  choose to open, not the always-loaded instruction file.
- **Fading.** In long-horizon iteration, agent code was "2.3x more verbose and 2.0x more eroded" than
  473 open-source repos, and prose quality guidance cut the starting level by up to a third without
  changing the rate of decay [slopcodebench]. CodeIF finds models "often ignore global formatting
  rules, such as line limits, and inconsistently follow naming conventions", best complete
  satisfaction 0.414 [codeif].
- **Occasionally effective.** An explicit "preserve" instruction cut excess edit distance from 0.195
  to 0.131 and cognitive complexity by 26.6% while raising Pass@1 2.3 points [minimal-edits].

The sibling dossier's factorial study found no effect of instruction-file structure at all, with
non-compliance driven by time on task [dossier-git: 2605.10039]. Read together: prose moves the
starting point; it does not hold the line. The things it measurably helps with — scope ("preserve",
"minimal") — are the judgement rules no tool can check.

### Structure: scale and scatter dominate; code health matters less for stronger models

The strongest structural predictor of agent success is how big the repository is and how scattered
the change. SWE-bench-Live: projects under 100 files and 20k LOC "often yield success rates above
twenty per-cent, whereas projects exceeding five-hundred files rarely exceed five per-cent", and
"Patches that touch seven or more files are never solved" [swe-live]. FeatBench: 60–70% on small
repos, 10–30% on large [featbench]. A difficulty predictor reaches AUC 0.863 on static features led
by "Patch fragmentation" and "Repository scale" [difficulty]. All observational, all confounded with
task difficulty — but they agree that what costs agents is non-local change.

On code quality itself, CodeScene's peer-reviewed study is more careful than its marketing. On 5,000
competitive-programming Python files, healthier code cut refactoring break-rate risk by "over 30%"
for the best mid-size open model — but "Claude Sonnet showed no significant difference" at n=1,000
[code-health]. The press release reframed this as AI assistants that "increase defect risk by at
least 30 percent" [codescene-pr]; the paper measured refactoring break rate, not production defects.
Function-level work finds "modularity is not a core factor" for generation [modularity]; incorrect
documentation "can greatly hinder code understanding" while missing documentation does not
[doc-effect]. **No study isolates file length.** "Agents work best on files under 500 lines" has no
source [evidence-gap].

### Agent code quality: over-building, duplication, complexity

The failure that dominates is doing too much. FeatBench: "Regressive implementation is the
predominant failure reason, accounting for 73.6% of analyzed failure cases", driven by "'scope creep'
by proactively refactoring code or extending features beyond the explicit user intent" [featbench].
Agent PRs carry 1.87× the redundancy of human PRs on one measured repository [less-reuse]. In a
difference-in-differences study of 806 Cursor-adopting repositories against 1,380 controls, velocity
rose transiently while "static analysis warnings increase significantly by 30.3%, and code complexity
increases by 41.6%" — persistently — and "A 100% increase in code complexity and static analysis
warnings causes a 64.5% and 50.3% decrease in development velocity" [cursor-did].

The counterweight is a two-phase controlled experiment that "did not detect systematic
maintainability advantages or disadvantages" for downstream maintainers [echoes] — with an
assistant, not an agent, on one Java feature. The conflict is recorded open. Vendor figures that
circulate ("1.7× more issues", "45% insecure", "+41% bugs") are each a single vendor's method on a
curated or heuristic sample [coderabbit-report; veracode; uplevel] and should not be quoted as
measurements of agents.

### Tests: the tests you give help; the tests it writes barely matter

Supplied tests help strongly: TiCoder's interactive test-first loop gives "average absolute
improvement of 45.97% in the pass@1" [ticoder]; TDFlow reaches 94.3% on SWE-bench Verified given
human tests [tdflow]. Agent-written tests do not: across six models, "changes in the volume of
agent-written tests do not significantly change final outcomes", at most 2.6 points [agent-tests].
Test-first is worth its cost when a human or a spec writes the test; prompting the agent to "write
tests first" for itself is not measured to buy anything.

### Review bots: acceptance is a definition

Acceptance of AI review comments ranges from 0.9% to 73.8% depending on what "acted on" means — 16
GitHub Actions bots at 0.9–19.2% addressed [review-actions], AI reviewers adopted 16.6% vs 56.5% for
humans [human-ai-review], CodeRabbit 36.4% [coderabbit-study], Atlassian 38.7% [rovodev], Google
~40% [autocommenter], Beko 73.8% "Resolved" [beko]. Google's finding is the most useful one for this
dossier: "For 33/50 (66%) of these best practices, violation detection is beyond the scope of
traditional static analysis" [autocommenter] — which is where a reviewer, human or model, earns its
place. And adopted AI-review suggestions raised cyclomatic complexity 0.106 against 0.003 for human
suggestions [human-ai-review]: the reviewer is another source of over-building.

### What nobody has measured

- A modern (2025–2026) ablation of lint or type-check hooks inside an agent harness.
- Whether remediation text in lint messages ("use X instead") changes agent behaviour versus a bare
  error.
- The rate at which agents add `eslint-disable`, `@ts-ignore`, `# type: ignore` or `noqa` — the `any`
  rate is the nearest measurement [ts-any].
- File length as an isolated variable.
- Whether architecture-boundary checks (dependency-cruiser, Nx, import-linter) change agent outcomes.

---

## 4. What the implementations actually do

### The retreat from the in-loop feedback loop

Open-source harnesses have been turning feedback loops off by default.

| Harness | Post-edit lint/typecheck/format | Evidence |
|---|---|---|
| OpenCode | **Off** since 2026-04-17: "`lsp` unset: all built-in LSPs disabled"; "Formatters are disabled by default" | [pr-opencode-22997; opencode-docs] |
| Kilo Code | Inherits OpenCode; LSP listed under *Experimental* | [src-kilo] |
| SWE-agent | Default `USE_LINTER = "false"`; the edit-rejecting guard survives only in legacy configs | [src-sweagent] |
| Cline v4 | Editor returns `` `Edited ${filePath}\n${diff}` ``; old problem templates have no callers | [src-cline] |
| OpenHands SDK | Docstring advertises `enable_linting`; the parameter does not exist | [src-openhands] |
| Aider | `--auto-lint` default True — but built-in lint is a tree-sitter syntax check, TypeScript skipped | [src-aider] |
| Crush | `auto_lsp` default true; errors and warnings, 10 per block | [src-crush] |
| Roo Code | Diagnostics on, *new* errors only ("warnings can be distracting"); repo archived 2026-05-15 | [src-roo] |
| Windsurf / Devin Desktop | Lint auto-fix "turned on by default" | [windsurf-docs] |
| Copilot cloud agent | Runs "your project's tests and linter", "enabled by default", if it can find them via `copilot-setup-steps.yml` | [gh-validation; gh-env] |
| Cursor | ~3 lint-fix iterations per file — **IDE only**: "the CLI and Cloud Agents don't run an extension host at all" | [cursor-forum-lint] |
| Claude Code | Nothing built in; diagnostics only via an LSP plugin, and "In cloud sessions, Claude Code doesn't start plugin language servers" | [cc-plugins] |
| Codex CLI | Nothing built in; the system prompt tells the model to "hold off on running tests or lint commands" in interactive approval modes | [src-codex-prompt] |

The split is IDE-hosted and closed products keeping the loop on; open-source CLIs switching it off.
Two consequences follow. A team on one vendor gets different feedback in the IDE than in the CLI or
cloud agent. And the SWE-agent result (§3) — the main causal evidence the loop helps — is now
describing a configuration its own authors do not ship. Why they turned it off is not documented in
any source found; latency and noise are the plausible reasons, and are inference.

### No post-edit hook can undo an edit, and "block" means different things

| Harness | PostToolUse / after-edit "block" does | Only true gate |
|---|---|---|
| Claude Code | Exit 2 "Shows stderr to Claude; the tool already ran"; `decision:"block"` adds the reason next to the result | Stop hook, capped at 8 consecutive continuations |
| Codex | Exit 2 replaces the tool result with stderr (sync hooks only; empty stderr is an error) | Stop `decision:"block"` → continuation prompt |
| Gemini CLI | Exit 2 "Hides the tool result… The turn continues" | AfterAgent exit 2 → retry |
| Cursor | `afterFileEdit` has no output fields at all; `postToolUse` may add context | Stop `followup_message`, loop_limit 5 |
| Copilot | postToolUse non-blocking; only preToolUse "can approve or deny" | — |
| Windsurf, Kiro, Crush | Only pre-hooks block | — |

[cc-hooks; codex-hooks; src-codex-hooks; gemini-hooks; cursor-hooks; gh-hooks; windsurf-hooks;
kiro-exit; src-crush]

Exit codes are a trap. In Claude Code "Any other exit code doesn't block on its own for most hook
events" — a lint script that exits 1, the Unix convention for failure, is silently non-blocking
[cc-hooks]. Gemini CLI's docs say other codes are a "Warning"; its code says "All other non-zero exit
codes (including 2) are blocking" [gemini-hooks; src-gemini-hooks]. Claude Code's UI once displayed
"PostToolUse:Edit hook blocking error" while "the edit **succeeds**" [cc-issue-19009].

Third-party enforcement inherits the weakest contract. Semgrep Guardian returns a block with findings
in Claude Code and Codex; its source for Cursor says "Cursor's afterFileEdit hook does not support any
outputs. There is no way to communicate the scan result to Cursor" [src-semgrep]. The same scanner
gates in one harness and is mute in another.

### Scoped rules attach on read, not on write

Claude Code: "Path-scoped rules trigger when Claude reads files matching the pattern" [cc-memory];
an open issue documents that "Creating a new file with `Write` loads neither" [cc-issue-96361], and
Bash/sed edits never trigger them [cc-issue-88565]. Cursor staff, 2026-08-13: "If the agent edits or
creates the file without opening it first, the rule never gets pulled in" — contradicting a staff
answer six months earlier [cursor-forum-rules]. VS Code Copilot's docs say `applyTo` matches files
the agent "creates or modifies"; its code matches attached files [src-vscode]. Continue's CLI drops
glob rules entirely [src-continue].

The practical consequence: a rule scoped to `src/api/**` does not exist for the agent writing the
first file in `src/api/`. Scoped prose is weakest exactly when a new module is being shaped.

### Lint messages and architecture tools

The "write the lint message for the agent" pattern is real but unevenly supported. oxlint is the only
linter found that auto-switches to an agent output format when it detects `CLAUDECODE`,
`GEMINI_CLI`, `CODEX_SANDBOX` or `OPENCODE` — "one line per diagnostic, no source excerpts, no
summary", keeping `help:` [src-oxlint]. import-linter added `--no-logo` because "the logo ascii art
gets added to the context" [import-linter-364]. Qlty's `slop-one` pre-push hook blocks on a
maintainability drop and prints "a refactoring prompt a coding agent can act on" [qlty].

Off-the-shelf boundary tools often hide the rationale: dependency-cruiser prints a rule's `comment`
only with `err-long`, not the default reporter [src-depcruise]; Nx `depConstraints` has no message
field and Nx's generated agent rules say nothing about boundaries [src-nx]. ArchUnit's `.because()`,
eslint-plugin-boundaries' message templates and ast-grep's `note` do carry it [arch-tools]. ESLint's
MCP server tells the model to ask before fixing [eslint-mcp] — the opposite of Factory's "self-correct
until green".

### Review bots: rules from the head or the base

Copilot code review and CodeRabbit read their configuration from the **PR head**, so a PR can rewrite
its own review rules; Bugbot's yaml ("a PR cannot change how Bugbot reviews itself"),
claude-code-action's restored CLAUDE.md, PR-Agent and Greptile's auto-approve read the **base**
[review-bots]. CodeRabbit is the most mechanical — it runs ~59 real tools with the repo's own configs
— and still blocks only if `request_changes_workflow` is on, default false [coderabbit-schema]. The
Claude `code-review` plugin explicitly does not lint: "do not run the linter to verify"
[src-cc-review-plugin].

### Where the docs and the code disagree

| Tool | Docs | Code / issue |
|---|---|---|
| Gemini CLI | Other exit codes: "Warning" | "All other non-zero exit codes (including 2) are blocking" |
| Aider | "built in linters for most popular languages" | tree-sitter syntax check; TypeScript returns early |
| Cline | "monitors linter and compiler errors… fixing issues" | v4 editor returns the diff only |
| SWE-agent | Paper: "Invalid edits are discarded" | Default config: linter off, edit applied |
| OpenHands | `enable_linting` parameter | No such parameter |
| VS Code Copilot | `applyTo` on files the agent "creates or modifies" | matches attached files |
| dependency-cruiser | `comment` explains the rule | default reporter omits it |
| Semgrep Guardian | "Hooks fire on every file write" | Cursor: "not supported" |

[implementation sources as keyed in §4 above; full list in sources.md §2]

---

## 5. What real repositories do

The corpus is 57 active repositories (TS 22, Python 14, Rust 13, Go 8, Zig 1) pinned at SHAs dated
2026-09-20 to 2026-10-02, each rule extracted with a verbatim quote and each status with file:line
evidence [corpus]. Selection skews toward mature devtool and AI-vendor repos with strong CI; the
general population is likely worse.

| Category | n | Enforced | Partial | Prose-only | Contradicted |
|---|---|---|---|---|---|
| Style | 756 | 15% | 17% | 65% | 3% |
| Process | 295 | 45% | 13% | 42% | 0% |
| Structure | 244 | 10% | 12% | **77%** | 1% |
| *Mechanizable only* | 830 | 33% | 19% | **45%** | 3% |
| *Judgement only* | 465 | 0% | 8% | 92% | 0% |

What gets written down and what gets enforced barely overlap. The most stated rules — how to write
tests (99 rules), "use X not Y" (85), preferred idioms (76), comments (56 rules in 32 repos, 49 prose),
file placement (54), module design (54) — are overwhelmingly prose. The reliably enforced ones are the
boring ones: regenerate artifacts (30 of 38 enforced), run lint (24/25), run tests (20/26),
formatting (15/23) — almost all by **CI**, which backs 266 of the 273 enforced rules. Agent hooks fully
enforce 7 rules in 3 repos. Function- and file-size rules appear 3 times in 57 repos — and
sentry-javascript enforces `max-lines` 300 and `complexity` 33 without its AGENTS.md mentioning
either [corpus].

The exemplars share a move: they tie each prose rule to the check behind it.

- **PostHog tags every rule.** "`[lint: <id>]` means a linter, semgrep rule, or invariant test blocks
  it, so CI is the check and this text is only the reasoning. `[review]` means nothing catches it
  automatically — a reader is the only control." By its own count 11 rules are `[lint`, 37 `[review]`
  [corpus: posthog].
- **n8n closes the escape hatch in the message.** Prose: "enforced by the
  `misplaced-n8n-typeorm-import` lint rule; a new import (or an inline `eslint-disable` of the rule)
  fails CI." Lint message: "In business logic, add a use-case repository method instead — do not
  relabel the import". And on why warnings are useless: "every lint script runs with `--quiet`, so a
  `warn` enforces nothing" [corpus: n8n].
- **bun turns a CRITICAL rule into a deny with the fix.** "**CRITICAL**: Never use `bun test`
  directly" becomes a PreToolUse guard returning "error: In development, use `bun bd test <file>` to
  test your changes" [corpus: bun].
- **temporal writes lint messages as instructions, including the sanctioned exception.** "use
  ns.ActiveClusterName(routingKey) instead. For genuine namespace-level checks (no workflow context),
  suppress with //nolint:forbidigo // <justification>." [corpus: temporal]
- **strands and cloudflare refuse to restate the linter.** "ruff, isort, mypy, and pydocstyle already
  enforce… This guide does not re-list those — it covers the conventions a linter *cannot* check";
  "The checked-in configuration is authoritative" [corpus: strands; cloudflare].
- **transformers blocks AI-style comments mechanically** — the only blocking check on the
  ubiquitous "few comments" rule in the corpus: "Flag comment blocks that are long enough to read as
  verbose, low-signal AI output" [corpus: transformers].

The cautionary cases share the opposite move: prose that claims a mechanism.

- cal.com: "the linter will block it" (no such import restriction exists); "This is enforced
  automatically in our CI pipeline" (no coverage threshold exists) [corpus: calcom].
- tldraw's Stop hook runs `xargs -r pnpm exec prettier --write || true` in a repo that formats with
  oxfmt — a hook that, as far as the committed files show, fails silently every time [corpus: tldraw].
- vscode ships its only agent check as `build.json.disabled` and tells agents to "minimize" builds
  [corpus: vscode].
- langchain asks agents to "break up complex functions (>20 lines)" while ignoring `PLR09` and `C90`
  in ruff [corpus: langchain].

Repos disagree on the hooks themselves. PostHog: "do not add `PreToolUse`, `PostToolUse`, or
`Notification` hooks as they add latency and are fragile" [corpus: posthog]. vercel/ai, supabase,
bun, uv, claude-agent-sdk-python and claude-code-action all commit PostToolUse format hooks
[corpus]. On public GitHub, ~16,350 committed `.claude/settings.json`-path files mention
`PostToolUse`; in a 73-file sample, 38 of 71 PostToolUse configs run a formatter, linter, typechecker
or test, and 11 of those 38 swallow failures (`|| true`, `2>/dev/null`, `exit 0`) [corpus-search].

---

## 6. Failure modes, ranked by how often the evidence shows them

1. **The rule is prose and nothing checks it.** 62% of stated rules; 77% of structure rules; 45% of
   rules a stock tool could check [corpus]. Prose shifts the start and doesn't hold the line
   [slopcodebench; codeif].
2. **The agent does more than asked.** Scope creep is 73.6% of feature-task failures [featbench];
   agent PRs are more redundant [less-reuse] and complexity rises persistently after adoption
   [cursor-did]. "Minimal change" is stated 25 times in the corpus and enforced 0 times [corpus].
3. **The prose contradicts the config.** 30% of repos [corpus]. The agent sees two authorities and
   models "seldom recognize contradictions" [dossier-agentsmd: PRIME].
4. **The hook fixes silently and never gates.** 0 of 7 corpus post-edit hooks block; 11 of 38 sampled
   swallow failures [corpus; corpus-search]. Exit 1 does not block in Claude Code [cc-hooks].
5. **The type escape hatch.** `any` at 9× the human rate [ts-any]; "no `any`" enforced in 1 of 10
   repos that state it [corpus].
6. **The harness doesn't run the check you think it does.** Feedback loops off by default in
   OpenCode, Kilo, SWE-agent and Cline v4; IDE-only in Cursor; absent in Claude Code cloud sessions
   [§4].
7. **The scoped rule never loads.** Path rules attach on read; a new file never triggers them
   [cc-issue-96361; cursor-forum-rules].
8. **The review bot is treated as a gate.** All three first-party bots default to neutral; comment
   uptake is 0.9–73.8% by definition [cc-review; cursor-bugbot; review-actions; beko].
9. **The architecture tool hides its reason.** dependency-cruiser's default reporter and Nx omit the
   rationale the agent needs to fix the violation correctly [src-depcruise; src-nx].

---

## 7. What to do

**Decide per rule whether it is a check or a judgement, and write it as one or the other.** A rule a
tool can check belongs in the tool, at error severity, run in CI; the instruction file keeps at most
the *why* and a pointer to the rule id. A rule no tool can check — scope, reuse, "minimal diff",
module design — is what prose and human review are for. PostHog's `[lint: id]` / `[review]` tagging
is the cleanest form of this, and makes the ratio visible.

**Never restate lint config in prose.** Every restatement checked had drifted. Point to the config:
"The checked-in configuration is authoritative" [corpus: cloudflare].

**Write lint messages for the agent.** Say what to do instead, name the sanctioned exception and how
to take it, and pre-empt the workaround ("do not relabel the import"; "an inline `eslint-disable`…
fails CI") [oai-harness; corpus: n8n; corpus: temporal]. Turn on the reporter that prints the reason
(dependency-cruiser `err-long`); prefer tools with a rationale field (ArchUnit `.because()`,
ast-grep `note`). Make warnings errors or delete them — "a `warn` enforces nothing" [corpus: n8n].

**Gate in CI first.** CI is where 266 of 273 enforced rules actually live [corpus], it runs whatever
harness the agent uses, and the agent cannot edit its way past it (subject to the gate-evasion cases
in [gitGuardrails/](../gitGuardrails/)). Treat hooks as a fast feedback layer on top, never the only
layer.

**If you write a post-edit hook, make it talk.** Use exit 2 (Claude Code, Codex, Gemini) so failures
reach the model; never `|| true`. Scope it to changed files and keep it fast. Put the expensive
check — full typecheck, tests — on Stop, which is the only hook that can refuse "done". Know the
harness: Cursor's `afterFileEdit` cannot speak, Copilot's postToolUse cannot block, Claude Code
ignores exit 1 [§4].

**Close the type escape hatch mechanically.** `no-explicit-any` (or the language's equivalent) at
error, `@ts-ignore`/`# type: ignore`/`noqa` requiring a justification, `strict` on — because agents
reach for `any` nine times as often as people do [ts-any].

**Structure for local change.** The evidence on structure is about scatter, not style: success falls
with files touched and repository size [swe-live; difficulty]. Boundaries the agent can see and a
tool enforces (OpenAI's fixed layering, import rules with reasons) keep changes local; file- and
function-size limits are reasonable as *lint rules* (sentry-javascript's `max-lines` 300) and
unsourced as magic numbers.

**Give the agent tests; don't ask it to write its own as the oracle.** Supplied tests move outcomes
by tens of points; agent-written tests by at most 2.6 [ticoder; tdflow; agent-tests]. Test-first is
worth doing when a person or a spec writes the test.

**Put scope rules in prose, and make them specific.** The one prose instruction measured to help is
"preserve" [minimal-edits]; scope creep is the dominant failure [featbench]. "Change only what the
task requires; do not refactor adjacent code" is worth its line.

**Use review bots for what static analysis can't see,** behind the mechanical gate, not instead of
it — 66% of Google's best practices were beyond static analysis [autocommenter]. Read their
configuration from the base branch where the bot allows it.

**Verify the harness does what its docs say.** Pin and test the behaviours you depend on — LSP on or
off, hook exit semantics, scoped-rule loading — because in this research the docs and the code
disagreed in at least eight places.
