# AGENTS.md rulebook

Checkable rules for writing and reviewing an `AGENTS.md`. Each rule states the test, the evidence
strength, and the source key (see [sources.md](sources.md)).

Evidence: **[S]** peer-reviewed or controlled · **[V]** vendor documentation · **[F]** field pattern
across many real files · **[W]** practitioner report with some measurement · **[O]** opinion.

Use it two ways: as a checklist when writing, and as a review gate — a line that fails R1 or R2 comes
out, no discussion.

---

## A. Admission — does this line belong at all?

**R1. The deletion test.** For every line: would removing it cause the agent to make a mistake? If
not, delete it. **[V]** `anthropic-best-practices`

**R2. The derivability test.** Delete anything the agent can learn by reading the repo — directory
layouts, dependency lists, architecture overviews, file-by-file descriptions. Repository overviews are
*measurably* unhelpful for task success. **[S]** `eth-agentsmd` · **[V]** `anthropic-best-practices`

**R3. The defaults test.** Delete anything a competent agent does anyway: "write clean code", "follow
existing style", "use TypeScript strict mode", "prevent XSS". If the agent already complies without
the line, the line is noise. **[V]** `anthropic-best-practices`

**R4. Fact, not procedure.** A multi-step procedure (deploy, release, review workflow) goes in a
skill, not here. Test: does it read as *"X is true"* or as *"first do A, then B"*? **[V]**
`anthropic-memory`, `anthropic-skills`

**R5. Global, not local.** A rule that only applies to some files goes in a path-scoped rule next to
those files (or a nested `AGENTS.md`), not the root file. **[V]** `anthropic-memory`

**R6. Advisory, not enforcement.** A rule that must hold *every* time is a hook, a permission
deny-rule, a lint rule or a CI check — not prose. "An instruction like 'never edit `.env`'… is a
request, not a guarantee." Keep the prose version only if you also want the agent to *understand*
the constraint. **[V]** `anthropic-features`

**R7. No harness fighting.** Do not restate the agent's own system prompt: response length, tone,
"be proactive", "mimic existing style", "you are an experienced developer". `temporalio/temporal` is
the cautionary example — a pasted CLI system prompt that says nothing about Temporal. **[F]**
`corpus`

**R8. No duplication within the file.** Say each thing once. `TanStack/router` repeats the same nx
invocations in three sections; repetition inflates cost without adding a rule. **[F]** `corpus`

**R9. Point, don't copy.** Reference the authoritative source rather than copying versions, script
lists or counts into the file — copies go stale silently. **[F]** `cloudflare-workers-sdk`

## B. Phrasing — is this line actionable?

**R10. Verifiable or nothing.** Every rule must be checkable by someone who did not write it.
"Use 2-space indentation", not "format code properly". "API handlers live in `src/api/handlers/`",
not "keep files organized". **[V]** `anthropic-memory`

**R11. No vibe checks.** Reject rules where the agent is the sole judge of compliance: "Would a
senior engineer say this is overcomplicated?", "avoid over-engineering", "keep changes focused".
Replace with the checkable form — `astro`'s "Every changed line should trace directly to the user's
request" is the good version of the same idea. **[F]** `corpus` · **[V]** `github-2500`

**R12. Numbers, not adjectives.** `openai/codex`: "the total number of changed lines should not
exceed 800 lines. For complex logic changes the size should be under 500 lines." The agent can
measure that against its own diff. **[F]** `corpus`

**R13. Attach the reason.** A rule with its rationale generalises to cases you did not foresee.
`astral-sh/uv`: "PREFER exhaustive `match` expressions without wildcard (`_`) arms over `matches!`,
so new enum variants require explicit handling." Anthropic: "Claude is smart enough to generalize
from the explanation." **[V]** `anthropic-prompting` · **[F]** `corpus`

**R14. Prefer the positive form where one exists.** "Answer in under 80 words" beats "don't be
verbose", because it closes the target. **[V]** `anthropic-prompting`. *But* do not purge negatives:
Anthropic's own docs list `"never do X" rules` as appropriate content, and the "negatives backfire"
claim is anecdotal, not measured. Keep a short, sharp `Never` list. **[O]** `pink-elephant`

**R15. Prohibitions carry an enforcement mechanism or a consequence.** The strongest forms in the
corpus: `prisma`'s "The `no-bare-cast` plugin + CI ratchet enforce no per-PR cast increases"; `zod`'s
"will be rejected regardless of how well it is tested"; `pytorch`'s "banned and will be closed".
A bare "never do X" with no teeth is a wish. **[F]** `corpus`

**R16. Close the loophole.** Anticipate the reading the agent will use to justify itself.
`grafana`: "'Open a PR' in a task description is intent, not permission to push without review."
`astral-sh/uv`: "NEVER assume clippy warnings or test failures are pre-existing, it is very rare that
`main` has warnings." **[F]** `corpus`

**R17. State exceptions inline.** A rule without its exception list gets over-applied. `airflow`:
"Write **Dag** (title case) in all prose. Keep the all-caps or lowercase spelling only when
reproducing a literal code token — never rewrite these, even inside fenced code blocks." **[F]**
`corpus`

**R18. Name the failure mode.** Instructions land harder when they say what goes wrong. `sentry`:
"Do not run pytest by itself; it'll take forever!" `ghostty`: "Prefer to run targeted tests with
`-Dtest-filter` because the full test suite is slow to run." **[F]** `corpus`

**R19. Headings are instructions.** "Rebuilding Before Running Tests" (next.js) tells the agent
something; "Important Development Notes" (bun), "Key Notes", "Notes" do not. Junk-drawer headings
defeat scanning. **[F]** `corpus`

**R20. Emphasise exactly one line, if any.** "If Claude keeps skipping one instruction, add emphasis
such as 'IMPORTANT' to that line alone. If you emphasize many lines, none of them stands out."
**[V]** `anthropic-best-practices`

**R21. The colleague test.** "Show your prompt to a colleague with minimal context on the task and
ask them to follow it. If they'd be confused, Claude will be too." **[V]** `anthropic-prompting`

**R22. Right altitude.** Not hardcoded brittle branching, not vague platitudes: "specific enough to
guide behavior effectively, yet flexible enough to provide the model with strong heuristics."
**[V]** `anthropic-context-eng`

## C. Structure and budget

**R23. Target under 200 lines.** "Longer files consume more context and reduce adherence." **[V]**
`anthropic-memory`. Corpus median is ~155 lines / ~1200 words, and excellent files exist at 32–40
lines. **[F]** `corpus`

**R24. Stay under 32 KiB across all files on any one path.** Codex concatenates root→cwd and "stops
adding files once the combined size reaches… `project_doc_max_bytes` (32 KiB by default)" — silent
truncation. This is the binding portability cap. **[V]** `codex-agents-md`

**R25. Highest-value content at the top.** Retrieval is U-shaped in position and models show "a bias
towards earlier instructions". The long middle is the worst real estate; put boundaries and commands
first, reference tables later. **[S]** `lost-in-middle`, `ifscale`

**R26. Budget the instruction count, not just the lines.** At 500 simultaneous instructions the best
models reach 68% accuracy. Density degrades adherence independently of file length. **[S]** `ifscale`

**R27. Headers and bullets, not prose.** "Claude scans structure the same way readers do: organized
sections are easier to follow than dense paragraphs." **[V]** `anthropic-memory`

**R28. No contradictions — anywhere in the loaded set.** Check the root file, every nested file on
the path, and path-scoped rules together. Models "seldom recognize contradictions or request
clarification"; they silently pick one, so contradictions surface as irreproducible behaviour, not
errors. **[S]** `prime` · **[V]** `anthropic-memory`

**R29. Nested files are additive, never overriding.** Override semantics are portable to almost no
tool: most concatenate, two honour "nearest wins", two load exactly one file, one guarantees no
order. If a subtree genuinely inverts a root rule, write the inversion out in prose. **[V]**
`codex-agents-md`, `charlie-docs`, `vscode-custom-instructions`

**R30. Include a Boundaries block: Always / Ask first / Never.** This gives a permission model rather
than a wish list, and makes escalation explicit. **[F]** `corpus`

**R31. Include a verification gate, narrowest-first, last in the file.** Name the narrow command for
development and the blocking command tied to "done". ~24 of 37 real files have a named gate section.
**[F]** `corpus`

**R32. Warn about false greens.** If a build can pass without typechecking, say so. `librechat`:
"A green build is not a typecheck: … `tsdown`, which emits without checking types. Run
`npx tsc --noEmit` in the workspace you changed." **[F]** `corpus`

**R33. Write a Gotchas section, and treat it as the file's core.** "Here is the thing you will think
is a bug" is the highest-value genre and the only one that is impossible to derive.
`huggingface/transformers`: "**`attr = AttributeError()` deletes an inherited attribute** — it is an
instruction to the converter, not a bug or placeholder." **[F]** `corpus`

**R34. List generated files and their regeneration command.** ~12 of 37 do this; hand-editing a
generated file is a silent, guaranteed waste of an agent's whole task. **[F]** `corpus`

**R35. Repo map as task → directory, not as `tree`.** And keep it short — R2 says the layout itself
is derivable; what is not derivable is *which directory a given kind of task belongs in*. **[S]**
`eth-agentsmd` · **[F]** `corpus`

**R36. Point at skills with their triggers.** Names plus a one-line "use when", so the agent knows
what it can load without loading it. `biome`: "Load only the skills relevant to the current task."
**[F]** `corpus` · **[V]** `anthropic-skills`

## D. Portability and file mechanics

**R37. The filename is exactly `AGENTS.md`.** Warp requires all caps; Windsurf and Factory are
lenient. Satisfy the strictest. **[V]** `warp-rules`, `factory-agents-md`

**R38. Delete legacy rule files.** Zed loads exactly one instruction file and ranks `.rules`,
`.cursorrules`, `.windsurfrules`, `.clinerules`, `.github/copilot-instructions.md` and `AGENT.md`
*above* `AGENTS.md`. A leftover `.cursorrules` silently shadows your file there. **[V]** `zed-instructions`

**R39. Bridge to Claude Code with a `@AGENTS.md` import, not duplication.** Claude Code "reads
`CLAUDE.md`, not `AGENTS.md`". A symlink works too, but needs Administrator on Windows and can be
checked out as a plain text file when git's `core.symlinks` is false. **[V]** `anthropic-memory`,
`git-core-config`

**R40. Do not use imports to save context.** "Splitting into `@path` imports helps organization but
doesn't reduce context, since imported files load at launch." Max depth four hops; backtick a path
you mean literally. **[V]** `anthropic-memory`

**R41. Import syntax is not portable.** `@file` is a Claude Code / Gemini CLI / Amp feature. Codex,
opencode, Warp and Copilot treat it as literal text. Do not build the file's structure on it.
**[V]** `codex-agents-md`, `opencode-rules`

**R42. Put maintainer notes in block HTML comments.** They are "stripped before the content is
injected into Claude's context" — free ownership, audit dates and rationale. **[V]** `anthropic-memory`

**R43. Configure the stragglers.** Aider has no auto-discovery at all despite being on the official
compatibility list (`read: AGENTS.md` in `.aider.conf.yml`); Gemini CLI needs
`context.fileName: "AGENTS.md"`. If your team uses them, ship the config. **[V]** `aider-conventions`,
`gemini-cli-context`

## E. Security

**R44. Treat `AGENTS.md` as an executable supply-chain artifact.** Agents load it as operator-level
instruction. A dependency-planted file has been demonstrated instructing an agent to inject a
regression into `main` and hide it from PR summaries. **[S]** `nvidia-injection`

**R45. Gate it in CI.** Diff it as a privileged file; reject or strip it on PRs from untrusted forks.
In review pipelines it "arriv[es] not as user input to be scrutinized but as a trusted configuration
to be followed" — i.e. it runs *before* the review meant to catch it. **[S]** `gitinject`

**R46. Never run an unattended agent against a repo you did not author.** Non-interactive modes
strip approval gates: "Safety controls are mode-dependent rather than invariant." **[S]** `backslash`

**R47. The file is not a security boundary.** Defences against injection show >85% attack success
under adaptive strategies. Do not rely on a prose instruction for anything that matters — use
permissions and filesystem isolation. **[S]** `injection-sok`

## F. Maintenance

**R48. Add a line only after the second failure.** Triggers: the same mistake twice; a review catches
something the agent should have known; you type the same correction you typed last session; a new
teammate would need the same context. **[V]** `anthropic-memory`

**R49. Diagnose by symptom.** Rule ignored despite existing → the file is too long and the rule is
lost. Agent asks what the file answers → the phrasing is ambiguous. **[V]** `anthropic-best-practices`

**R50. Verify every command by executing it during an audit.** Stale commands are the most damaging
failure mode: a rule telling agents to "never modify" a file they routinely need to edit was found in
a real repo. Stale instructions "decay into confident, specific, wrong instructions." **[W]**
`instruction-ablation`

**R51. Review it like code, in the PR that changes the process.** Give it an owner. **[V]**
`anthropic-blog-steering`

**R52. Test a change by observing whether behaviour actually shifted.** Probe the rule in a fresh
session with a task designed to trigger it. **[V]** `anthropic-best-practices` · **[W]** `unblocked-audit`

**R53. Evaluate before deploying an instruction change you expect to improve correctness.** "any
attempts to improve performance should be rigorously evaluated before deployment." **[S]**
`eth-agentsmd`

**R54. Gate agent-authored edits behind human review.** LLM-generated context files measured at or
below baseline — the worst-performing category in the study. **[S]** `eth-agentsmd`

**R55. Expect efficiency, not intelligence.** The honest claim for a good file is median −28.6%
runtime and −16.6% output tokens, not a higher pass rate. Set the team's expectations there. **[S]**
`lulla-efficiency`, `eth-agentsmd`

---

## Review checklist (the short form)

1. Every line passes R1 (deletion) and R2 (derivability).
2. Nothing here is a procedure (R4), a per-file rule (R5), or a must-hold-always rule (R6).
3. No harness restatement (R7), no internal duplication (R8), no copied-and-will-go-stale facts (R9).
4. Every rule is verifiable by a third party (R10) with no vibe checks (R11).
5. Prohibitions have teeth (R15) and closed loopholes (R16).
6. Under 200 lines (R23); under 32 KiB combined on any path (R24); no contradictions (R28).
7. Boundaries block present (R30); verification gate present and narrowest-first (R31).
8. Gotchas section exists and is the best part of the file (R33).
9. Filename exact (R37); legacy rule files deleted (R38); Claude bridge is an import (R39).
10. Owner and audit date in an HTML comment (R42); every command was run today (R50).
