# Agentic coding guidelines rulebook

Checkable rules for shaping a codebase and its coding guidelines so coding agents produce good code,
and for enforcing those guidelines. Each rule states its test (how a third party checks compliance),
its evidence tier, and its source keys (see [sources.md](sources.md)).

Tiers: **P1** primary spec, vendor docs or shipped source · **P2** peer-reviewed or preprint research
· **P3** vendor engineering blog / industry research with method · **P4** practitioner measurement
(including this dossier's own corpus counts) · **P5** opinion, anecdote, unverified.

Use it two ways: as a design checklist when writing or revising a repository's agent guidelines, and
as a review gate for an existing setup. A repository that fails R1, R4, R12 or R20 does not pass,
whatever else it gets right. The instruction file's own shape is in
[agentsMd/rulebook.md](../agentsMd/rulebook.md); git rules and CI gate evasion are in
[gitGuardrails/rulebook.md](../gitGuardrails/rulebook.md); the test oracle is in
[softwareFactories/rulebook.md](../softwareFactories/rulebook.md).

---

## A. Prose or mechanism — deciding where each rule lives

**R1. Classify every stated rule as a check or a judgement.** A check is anything a stock or custom
tool can decide; a judgement is everything else. Test: can each coding rule in the agent instruction
files be mapped to either a tool rule id or an explicit "review-only" marker? **P1** `cc-memory`,
`cursor-rules` · **P3** `oai-harness` · **P4** `corpus`

**R2. Move every checkable rule into a tool.** 45% of mechanizable rules in the corpus are still
prose. Test: for each rule classified as a check, does a lint, type, format, architecture or CI rule
exist that fails on a violation? **P3** `oai-harness`, `factory-linters` · **P4** `corpus`

**R3. Tag each prose rule with what enforces it.** Test: does each rule carry a marker naming its
enforcing rule id, or stating that only review enforces it (PostHog's `[lint: <id>]` / `[review]`)?
**P4** `corpus` (posthog)

**R4. Never restate lint, format or type config in prose.** Every restatement the corpus checked had
drifted, including Anthropic's own example rule. Test: does any instruction file restate a value that
lives in a formatter, linter or compiler config (indentation, quotes, trailing commas, line length,
filename case)? Any instance fails. **P1** `cursor-rules`, `rustc-llm` · **P4** `corpus`
(mcp-ts-sdk, gitbutler)

**R5. Point to the config as the authority.** Test: does the instruction file name the command that
runs every check (`pnpm check`, `just lint`) and say the checked-in configuration wins? **P4**
`corpus` (cloudflare, strands)

**R6. Make prose and config agree.** 30% of corpus repos state a rule their own config contradicts.
Test: for each rule stating a banned construct, is the corresponding tool rule enabled at error
everywhere the rule claims to apply, with no package-level `off` overrides? **P4** `corpus` (n8n,
twenty, langchain, crush, zed) · **P2** `dossier-agentsmd` (PRIME)

**R7. Don't claim enforcement that doesn't exist.** Test: for each sentence in the instruction files
asserting a linter, CI job or threshold "will block" or "enforces", does that mechanism exist and fail
on a violation? **P4** `corpus` (calcom, github-mcp-server, adk-python)

**R8. Keep prose for judgement rules, and make the scope rules specific.** Scope creep is the dominant
failure; a "preserve" instruction is the one prose rule measured to help. Test: does the instruction
file contain an explicit minimal-change rule ("change only what the task requires; don't refactor
adjacent code")? **P2** `minimal-edits` · **P3** `featbench`

---

## B. Writing checks the agent can act on

**R9. Every custom lint message says what to do instead.** Test: does each repo-specific lint rule's
message name the replacement API or pattern? **P3** `oai-harness`, `ant-tools`, `factory-linters` ·
**P4** `corpus` (temporal, n8n, bun)

**R10. Name the sanctioned exception and how to take it.** Test: where a rule has legitimate
exceptions, does its message state the suppression form and require a justification
(`//nolint:rule // <why>`)? **P4** `corpus` (temporal)

**R11. Pre-empt the workaround in the message.** Test: for rules agents can trivially route around
(relabelled import, inline disable, re-export), does the message or prose say that the workaround
also fails? **P4** `corpus` (n8n)

**R12. Errors, not warnings.** "a `warn` enforces nothing" when lint runs `--quiet`. Test: is every
rule the instruction files depend on configured at error severity, and does CI fail on it? **P4**
`corpus` (n8n)

**R13. Turn on the reporter that prints the rationale.** Test: for architecture tools, is the output
mode the agent sees one that includes the rule's reason (dependency-cruiser `err-long`, ArchUnit
`.because()`, ast-grep `note`)? Nx `depConstraints` has no reason field — document the reason next to
the tag. **P1** `src-depcruise`, `src-nx`, `arch-tools`

**R14. Prefer agent-friendly output formats where the tool has one.** Test: are linters invoked so
agents get compact, one-line diagnostics with the help text kept (oxlint auto-detects; others via
`--format`)? **P1** `src-oxlint`, `import-linter-364`

---

## C. Structure

**R15. Enforce module boundaries mechanically.** Test: does an import-boundary or layering check
(dependency-cruiser, eslint-plugin-boundaries, Nx, import-linter, ArchUnit, depguard, custom test) run
in CI and fail on a violation? **P3** `oai-harness`, `factory-linters` · **P1** `arch-tools`

**R16. Make the layering explicit and one-directional.** Test: is there a written dependency direction
(e.g. Types → Config → Repo → Service → Runtime → UI) with cross-cutting concerns through one named
interface, and does a check enforce it? **P3** `oai-harness`

**R17. Structure for local change.** Agent success falls with files touched and repository scale.
Test: does a typical feature in the repository touch one module and its tests, rather than requiring
coordinated edits across packages? (Judgement; check against recent PRs.) **P3** `swe-live`,
`difficulty`, `swe-pro` · **P3** `featbench`

**R18. If you limit file or function size, do it in the linter.** No vendor publishes a number and no
study isolates file length. Test: is any size limit stated in prose backed by `max-lines` /
`max-lines-per-function` / `complexity` (or the language's equivalent) at error? **P3** `oai-harness`
· **P4** `corpus` (sentry-javascript, langchain) · **P5** `evidence-gap`

**R19. Optimise for grep and glob.** Test: are exports named (not default), and do files sit at
predictable paths that a glob can find? **P3** `factory-linters`

---

## D. Types

**R20. Close the type escape hatch at error.** Agents use `any` 9× as often as humans; "no `any`" is
enforced in 1 of 10 repos that state it. Test: is `no-explicit-any` (or the language equivalent) at
error with no package overrides, and is strict mode on? **P2** `ts-any`, `typed-decoding` · **P4**
`corpus`

**R21. Require a justification for every suppression.** Test: do `@ts-ignore`, `@ts-expect-error`,
`# type: ignore`, `noqa` and `eslint-disable` require a reason comment, enforced by a lint rule
(e.g. `ban-ts-comment` with description, `eslint-comments/require-description`)? **P1**
`gemini-prompt` · **P5** `evidence-gap` (no measured rate)

**R22. Give the agent type diagnostics after edits, in every surface it runs.** Test: in each
surface the team uses (IDE, CLI, cloud agent), does a typecheck run after edits or at Stop — not only
in the IDE? **P1** `cc-plugins`, `cursor-forum-lint`, `pr-opencode-22997`

---

## E. In-loop enforcement: hooks and harness settings

**R23. A post-edit hook must report failure to the model.** 0 of 7 corpus post-edit hooks block; 11
of 38 sampled lint hooks swallow failures. Test: does the hook exit 2 (Claude Code, Codex, Gemini) or
return a block decision on failure, with no `|| true`, `2>/dev/null` or forced `exit 0`? **P1**
`cc-hooks`, `codex-hooks` · **P4** `corpus`, `corpus-search`

**R24. Never rely on exit 1.** Test: does any enforcement hook rely on exit 1 to block? In Claude Code
it does not. Fail if so. **P1** `cc-hooks`

**R25. Put the expensive gate on Stop, not on every edit.** Only Stop-type hooks can refuse "done";
post-edit hooks cannot undo an edit. Test: is the full typecheck/test suite wired to a Stop (or
AfterAgent) hook rather than to every PostToolUse? **P1** `cc-hooks`, `codex-hooks`, `gemini-hooks`,
`cursor-hooks`

**R26. Know which hooks cannot speak in your harness.** Test: is any enforcement wired to a hook that
has no feedback channel (Cursor `afterFileEdit`, Copilot postToolUse, post-hooks in Windsurf, Kiro,
Crush)? Fail if it is the only enforcement. **P1** `cursor-hooks`, `gh-hooks`, `windsurf-hooks`,
`kiro-exit`, `src-semgrep`

**R27. Turn feedback loops on explicitly; don't trust defaults.** Test: are LSP diagnostics and
formatters explicitly enabled in harness config where the default is off (OpenCode `lsp`, `formatter`;
Kilo; SWE-agent `USE_LINTER`)? **P1** `pr-opencode-22997`, `src-sweagent`, `src-kilo`

**R28. Scope hooks to changed files and keep them correct.** Test: does the hook operate on changed
files only, using the repo's actual formatter and config (not `--no-config` across the whole tree, not
a formatter absent from the lockfile)? **P4** `corpus` (tldraw, claude-code-action)

**R29. Don't depend on scoped rules for new files.** Path-scoped rules attach on read. Test: is any
rule that must apply when *creating* files in a path delivered by something other than a path-scoped
rule (root file, lint rule, generator, CI)? **P1** `cc-memory`, `cc-issue-96361`,
`cursor-forum-rules`

**R30. CI is the gate; hooks are the fast path.** Test: is every rule enforced by a hook also enforced
in CI, so a different harness, a disabled hook or a cloud surface cannot bypass it? **P4** `corpus` ·
**P1** `cc-plugins`

---

## F. Tests and verification

**R31. Give the agent a check it can run.** Test: does the instruction file name the exact command(s)
to verify a change, and do they run in the agent's environment (including cloud agents —
`copilot-setup-steps.yml` job name exact)? **P1** `cc-best`, `codex-best`, `gh-best`, `gh-env`

**R32. Supply tests; don't use agent-written tests as the oracle.** Test: for tasks with test-first
workflows, is the failing test written or approved by a human or derived from a spec before the agent
implements? **P2** `ticoder`, `tdflow`, `agent-tests`

**R33. Iterate only against a tool oracle.** Unguided refinement degrades security; static-analysis
feedback improves it. Test: does every self-repair loop feed back a tool's output, with an iteration
cap? **P2** `sa-feedback`, `sec-degradation`

---

## G. Review bots

**R34. A review bot is not a gate.** First-party bots default to neutral. Test: is every rule that
must hold enforced by a required CI check, independent of review-bot findings? **P1** `cc-review`,
`cursor-bugbot`, `gh-review`

**R35. Point review bots at what static analysis can't see.** Test: does the review-bot instruction
file (REVIEW.md, BUGBOT.md, `## Code Review Rules`) exclude what CI already checks and focus on
judgement rules? **P1** `cc-review` · **P2** `autocommenter`

**R36. Read review config from the base branch where the bot allows it.** Test: for each bot in use,
is it known whether config comes from the PR head (Copilot, CodeRabbit) or base, and for head-reading
bots is a change to their config files gated by CODEOWNERS? **P1** `review-bots`,
`coderabbit-schema`

**R37. Put review rules in the file the bot actually reads.** Test: are Bugbot rules in `BUGBOT.md`
(not `.cursor/rules`), Codex rules under `## Code Review Rules`, Claude review rules in CLAUDE.md or
REVIEW.md (not only AGENTS.md)? **P1** `cursor-bugbot`, `codex-review`, `src-cc-review-plugin`

---

## H. Maintaining guidelines

**R38. Promote a rule to code when it is violated twice.** Test: for rules that review comments
repeatedly cite, does a lint or CI check now exist? **P3** `oai-harness` · **P1** `codex-best`

**R39. Lint the instruction files themselves.** Test: does CI check that commands named in the
instruction files exist (Makefile targets, scripts) and that instruction files stay within a size
budget? **P4** `corpus` (transformers `make fixup`, mastra TOKEN_LIMIT)

**R40. Re-verify harness behaviour on upgrade.** Docs and code disagreed in at least eight places.
Test: is there a recorded check (smoke test or checklist) of hook exit semantics, LSP/formatter
defaults and scoped-rule loading for each harness version the team uses? **P1** `src-gemini-hooks`,
`src-aider`, `src-cline`, `src-openhands`, `src-vscode`

---

## Review checklist

A repository's agent guidelines do not pass review until every one of these holds:

| # | Gate | Rules |
|---|---|---|
| 1 | Every coding rule is classified as a check or a judgement, and tagged with its enforcer | R1, R3 |
| 2 | Every checkable rule is a tool rule at error, run in CI | R2, R12, R30 |
| 3 | No prose restates or contradicts config, or claims enforcement that doesn't exist | R4, R6, R7 |
| 4 | Lint messages name the fix, the exception, and the closed workaround | R9, R10, R11 |
| 5 | Module boundaries and type escape hatches are enforced mechanically | R15, R20, R21 |
| 6 | Hooks report failures (exit 2), never swallow them, and are backed by CI | R23, R24, R30 |
| 7 | The agent has a runnable check in every surface it uses | R22, R31 |
| 8 | Review bots are behind, not instead of, the mechanical gate | R34, R35 |

**The four rules that gate everything else:** R1, because a rule nobody has classified is a rule
nobody will enforce; R4, because a restated config value is a second authority that drifts; R12,
because a warning-level rule is prose with extra steps; and R20, because the type checker catches most
of what goes wrong and the agent will reach for the escape hatch nine times as often as a person.
