# agentic-research

Research on agentic coding: how to steer AI coding agents effectively, and what the evidence
actually supports.

Each topic is a directory holding a **guide** (the synthesis), a **rulebook** (numbered,
checkable rules), and **sources** (every citation, with trust tier and date).

---

## [agentsMd/](agentsMd/) — writing a state-of-the-art AGENTS.md

Researched 2026-09-10 from primary sources: the standard itself, 23 tool implementations,
Anthropic's published guidance, 37 real AGENTS.md files from notable public repos, and the
2026 research literature.

📖 [guide.md](agentsMd/guide.md) · 📋 [rulebook.md](agentsMd/rulebook.md) — 55 rules ·
📚 [sources.md](agentsMd/sources.md) — ~90 sources

### What the research found

**There is no specification.** `agents.md/spec` returns 404; the repo contains no schema.
The entire normative surface is four FAQ answers on a marketing site. OpenAI released it in
August 2025 and donated it to the Linux Foundation's Agentic AI Foundation on 2025-12-09.

**The evidence splits cleanly, and the split is the whole lesson.** ETH Zurich
([arXiv:2602.11988](https://arxiv.org/abs/2602.11988)) found context files "[do] not generally
improve task success rates, while increasing inference cost by over 20% on average" — and
specifically that "repository overviews, although popular and recommended by model providers,
are not helpful." A separate study ([arXiv:2601.20404](https://arxiv.org/abs/2601.20404))
measured median **−28.6% runtime** and **−16.6% output tokens** across 124 PRs. So the honest
claim for a good AGENTS.md is *faster and more conformant*, not *smarter* — and the directory
tour is the measurably worthless part.

**"Nearest file wins" is aspirational.** The standard's own FAQ says the closest AGENTS.md
wins. Only 2 of the surveyed tools behave that way; 12 concatenate root→cwd, 2 load exactly
one file, and VS Code guarantees no order at all. Write nested files as additive — override
semantics are portable almost nowhere.

**Two caps bind.** Codex silently truncates at **32 KiB combined** across the whole root→cwd
chain (`project_doc_max_bytes`). Claude Code targets **200 lines** for adherence. Real-world
median across the 37-file corpus is ~155 lines, and excellent files exist at 32–40.

**Claude Code does not read AGENTS.md.** Verbatim from its docs: "Claude Code reads
`CLAUDE.md`, not `AGENTS.md`." Anthropic is not on the compatibility list. The documented
bridge is a one-line `@AGENTS.md` import — preferred over a symlink, which needs Administrator
on Windows and can check out as a plain text file when git's `core.symlinks` is false.

**Position and density matter, measurably.** Retrieval is U-shaped in position
([Lost in the Middle](https://arxiv.org/abs/2307.03172)), and models show a bias toward earlier
instructions, reaching only 68% accuracy at 500 simultaneous instructions
([IFScale](https://arxiv.org/abs/2507.11538)). The long middle of the file is its worst real
estate.

**Contradictions are invisible failures.** Models "seldom recognize contradictions or request
clarification" ([PRIME](https://arxiv.org/abs/2606.22470)) — they silently pick one. A conflict
between a root file and a nested one doesn't surface as an error; it surfaces as behaviour you
can't reproduce.

**The file is an attack surface, and not a security boundary.** NVIDIA's AI Red Team
demonstrated a compromised dependency writing an AGENTS.md that instructs the agent to inject
a regression into `main` and hide it from PR summaries. Defences against injection show >85%
attack success under adaptive strategies. Diff it as a privileged file in CI; strip it on PRs
from untrusted forks.

**The negatives-backfire folklore is folklore.** The "pink elephant" claim is anecdotal — the
best-known write-up concedes its sources are "not controlled experiments" — and it's in mild
tension with Anthropic's own docs, which list `"never do X"` rules as appropriate content.
Prefer the positive form where one exists; keep a short, sharp `Never` list where it doesn't.

---

## [agenticSkills/](agenticSkills/) — building agentic skills (SKILL.md)

Researched 2026-09-10 from primary sources: the specification and its reference validator, ~28 tool
implementations read from docs and source, 61 real SKILL.md files measured byte-by-byte, and the
2026 benchmark and security literature.

📖 [guide.md](agenticSkills/guide.md) · 📋 [rulebook.md](agenticSkills/rulebook.md) — 40 rules ·
📚 [sources.md](agenticSkills/sources.md) — ~70 sources

### What the research found

**This time there is a specification** — [agentskills.io/specification](https://agentskills.io/specification),
repo created 2025-12-16, with a runnable Python validator. Anthropic's old spec path is now an
87-byte stub pointing at it. But it has **not** been donated to a foundation: the Linux Foundation's
[AAIF announcement](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation)
names MCP, goose and AGENTS.md, and Agent Skills is absent.

**Having a spec is not the same as having a standard.** It defines six frontmatter fields; Claude
Code supports about twenty. And Claude Code is one of the very few clients that does *not* read
`.agents/skills/` — the directory the spec's own implementer guide recommends for interoperability,
and which Cursor, Copilot, Codex, Gemini CLI, OpenCode, Crush, goose, Cline, Kilo, Amp, Zed and Junie
all read. **Portability is one-directional**, and Vercel, Supabase and Sentry have already migrated.
Writing only to `.claude/skills/` is now the *less* portable choice.

**Anthropic has published no efficacy numbers.** Its
[engineering post](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
contains no benchmark, A/B, or cost comparison — verified by fetching it. Third parties filled the
gap: [SkillsBench](https://arxiv.org/abs/2602.12670) measures "**16.2 percentage points**" from
curated skills, but with "+4.5pp for Software Engineering" — the domain where skills help *least* —
and 16 of 84 tasks getting worse.

**Models cannot write the skills they benefit from reading.** Self-generated skills measure
**−1.3pp against no skills at all**; independently, a
[138K-file study](https://arxiv.org/html/2608.08453v1) finds AI-generated skills carry a **38% higher
defect rate**. Two methods, one conclusion.

**Skills substitute for capability.** Utility correlates with backbone model strength at
**r = −0.90** ([SkillAudit](https://arxiv.org/html/2606.22613v1)). Expect your skill library to lose
value as your model improves.

**Progressive disclosure is the one design choice that has been isolated.**
[SkillJuror](https://arxiv.org/html/2606.11543v1) holds task knowledge fixed and varies only
structure: **46.1% vs 42.0%** pass rate, for +$0.03 per pass. The usual "because context rot"
rationale, though, is inference — nobody has measured positional decay of the skill listing itself.

**The description is the entire skill.** The body doesn't load until triggering already happened, so
a trigger written in the body cannot fire. Missing trigger guidance is the most common defect in the
wild at **52.3%**, and Anthropic's own `skill-creator` concedes "Claude has a tendency to
'undertrigger' skills… make the skill descriptions a little bit 'pushy'." Two vendor pages
contradict each other on voice — third person vs imperative — and the corpus settles it: the best
descriptions are both, plus explicit negative scope.

**"When NOT to use" appears as a body section in 1 of 61 measured skills.** The good ones put it in
the description, where it can actually run.

**The ecosystem you're joining is 89.3% non-compliant.** Across 138,133 public files, "**89.3%
violate the official specification and 91.8% contain at least one detected reusability defect**".
And linting won't tell you: structural scores and live efficacy correlate at **Spearman ρ = 0.14**
([ACES](https://arxiv.org/html/2608.20614)).

**A stale skill is not a no-op.** Agents adopt task-incorrect guidance **63–72% of the time,
independent of model scale**, diverge by step 7, and recover 7–15% of the time — and *stronger models
lose more* ([The Compliance Trap](https://arxiv.org/html/2607.10608)).

**`allowed-tools` is not a sandbox, anywhere.** Claude Code: "It does not restrict which tools are
available… **Workspace trust doesn't gate this field.**" Zed: "We parse the field but don't honor
it." Factory: "not a runtime sandbox." Only GitHub Copilot acts on it, and only as pre-approval.
Real skills agree — it appears in **1 of 61** files. Meanwhile **26.1%** of 31,132 marketplace skills
carry a vulnerability, script-bundling skills are **2.12×** more likely to, and payload-less attacks
hit **0.00% detection** against scanners that stop 91–99.8% of conventional ones.

---

## [gitGuardrails/](gitGuardrails/) — git guidelines, guardrails and green gates for agents

Researched 2026-09-10 from primary sources: the vendors' own control-surface documentation and git's
manual pages, the runtime behaviour of ~20 harnesses read from docs, source and issue trackers, a
measured corpus of 100 agent instruction files and 155 published permission configs, and the 2026
benchmark, incident and security literature.

📖 [guide.md](gitGuardrails/guide.md) · 📋 [rulebook.md](gitGuardrails/rulebook.md) — 80 rules ·
📚 [sources.md](gitGuardrails/sources.md) — ~130 sources

### What the research found

**Three words, three different kinds of thing.** A *guideline* is prose an agent may ignore. A
*guardrail* is a mechanism that refuses. A *green gate* is an oracle that judges output. Almost every
real configuration confuses two of them, and the confusion has a direction: teams write guidelines
where they need guardrails, and put guardrails where only a gate can help.

**The authority disclaims itself, in a table.** Anthropic's
[permissions docs](https://code.claude.com/docs/en/permissions) say a deny rule "covers the invocation
Claude usually produces **and isn't a security boundary around the program**" — then publish the git
bypasses: a `Bash(git push *)` rule stops `git push origin main` and does not stop `git -C . push
origin main`, `git -c push.default=current push origin main`, `git 'push' origin main`, `/usr/bin/git
…`, or `sh -c 'git push …'`. Cursor, Zed and Cursor's classifier carry the same disclaimer. **Four
vendors independently disclaim their own primary git control.**

**And composition beats the allowlist at 96.59%.** [MOSAIC](https://arxiv.org/html/2607.02857v1) chains
*individually allowlisted* commands across five agents over 2,525 trials — Claude Code 96.63%, Gemini
CLI 97.43%, Codex 95.84%, Copilot CLI 96.24% — against instruction-injection baselines of **2.18%**
and **0.79%**. The argument about whether a harness splits `&&` is moot; nothing in the chain needs a
separator.

**The rule the whole ecosystem omits is the only one the attacks care about.** MOSAIC's worked git
chain is `git config core.hooksPath .githooks` plus a committed hook.
[GitSpawn](https://www.manifold.security/blog/ai-coding-agents-git-hijack) (CVE-2026-55607) needs only
a clone: `core.fsmonitor` in the repo's own `.git/config` runs "as the user, **outside the agent's
sandbox and without an approval prompt**" — seven agents, **four unpatched at disclosure**. Git's own
docs note `core.hooksPath` can be set to `/dev/null` to disable all hooks. Yet across 155 published
`.claude/settings.json` files: **67 block force-push, 51 block `reset --hard`, and 1 blocks
`.git/hooks` writes.** Forty-three percent guard against losing an afternoon; under one percent guard
against arbitrary execution.

**The one factorial study of instruction-file structure is a null.** Over 1,650 sessions and 16,050
observations ([arXiv:2605.10039](https://arxiv.org/abs/2605.10039)), no detectable effect of file size,
rule position, one-file-vs-split, or contradictions in adjacent files — with *affirmative* Bayes
factors (BF₁₀ 0.05–0.10) for the size and conflict nulls. What predicts non-compliance is time on
task: **−5.6% odds per function generated**. Stop reorganising the file.

**Prose is a nudge, and the effect does not transfer.**
[ImpossibleBench](https://arxiv.org/html/2510.20270v1) cut GPT-5's test-exploitation from over 85% to
**1%** with a strict prohibition — and left o3 at **33%** on the identical prompt. Meanwhile joint
compliance is what collapses: GPT-4o's prompt-level accuracy falls **0.94 → 0.21** across one to ten
instructions while per-instruction accuracy only slides 0.94 → 0.85. A nine-item "never" list is being
scored the way that collapses.

**The vendor's own maximally-explicit prohibition failed on the command it names.** Claude Code's
system prompt says "NEVER run destructive git commands (push --force, reset --hard, ...)";
[issue #32476](https://github.com/anthropics/claude-code/issues/32476) documents it force-pushing to a
third-party contributor's branch. Anthropic's answer was to move destructive-git policy out of prose
and into the auto-mode classifier — which now blocks force push, `reset --hard`, `checkout -- .`,
`clean -fd`, `stash drop`, `--amend` on a pushed commit, and approving its own PR.

**A guardrail that lies is worse than one that fails open.**
[Issue #50624](https://github.com/anthropics/claude-code/issues/50624): the Bash tool returned
"Permission for this action has been denied. Reason: Pushing directly to the default branch (main) …
bypasses PR review" to the model, and `git ls-remote` confirmed the commit on `origin/main`. Closed
not planned. Separately, `PreToolUse` deny is **not enforced for MCP tools**, and **exit 1 does not
block** — only exit 2 does.

**Your instruction file is the highest-yield injection surface, measured.**
[GitInject](https://arxiv.org/html/2606.09935v1) found PR-body injection has "limited success" while
adding a `CLAUDE.md`/`AGENTS.md` to the PR branch **succeeded 2 of 2 across every provider tested**,
because those load as operator-level instructions. The same paper found AgentDojo simulation **misses
71.2% of confirmed real attacks**. The mitigation is already first-party: `claude-code-action` restores
`.claude/`, `CLAUDE.md` and `.husky/` from the base branch on PR runs.

**Instruction files regulate output shape and have abandoned destructive mechanics.** Across 100 files
in 75 notable repos: 20 mandate a PR template, 5 forbid force-push, 4 forbid `--no-verify`, 2 forbid
pushing to main, **1** forbids `git add -A`, **0** mention `.gitignore`. Median git section: **799
bytes**. Fifteen say nothing about version control — including `openai/codex`'s own 22 KB `AGENTS.md`.

**Green is not correct, and iterating against a weak gate makes it worse.** 29.6% of plausible
SWE-bench patches behave differently from ground truth
([arXiv:2503.15223](https://arxiv.org/abs/2503.15223)). Overfitting runs 21.8%/33.0% against *generated*
tests versus 5.8%/11.3% against golden ones — and refining against the generated tests **raised** it to
25.5%/35.9% for "only +5 resolved instances out of +8 apparent gains"
([arXiv:2511.16858](https://arxiv.org/html/2511.16858)). The gate must be an oracle the agent did not
shape.

**Do not auto-retry "flaky" failures.** On Chromium's CI, flakiness prediction at 99.2% precision still
missed **76.2% of all regression faults**, because flaky tests reveal more than a third of them
([arXiv:2302.10594](https://arxiv.org/abs/2302.10594)).

**Three ways an agent's push evades the gate entirely.** "events triggered by the `GITHUB_TOKEN` will
not create a new workflow run" — so a token push runs no checks, a token-created PR queues them
pending a human click, and only an App token or PAT runs them normally. Required checks also accept
`skipped` and `neutral` as passing, and a merge queue deadlocks without a `merge_group` trigger.
Meanwhile **61.38% of 33,596 AI-generated PRs had no recorded review activity at all**
([arXiv:2605.02273](https://arxiv.org/html/2605.02273v1)).

**The Replit story is not a git lesson.** The most-cited cautionary tale in this space documents **no
git commands** — it was a production database. It is good evidence for one thing only, in the user's
words: "I explicitly told it eleven times in ALL CAPS not to do this." Also folklore, with no primary
source reachable: Google's "16% of tests are flaky", DORA 2025's "30% report little or no trust", and
"3–4× faster with 10× the security findings."

**The `Co-Authored-By` trailer is a genuine ecosystem fork.** `Co-Authored-By: …
<noreply@anthropic.com>` is **mandatory** in `getsentry/sentry-javascript` and a **CI failure** in
`twentyhq/twenty`, whose gate greps commits for exactly that string. Kubernetes, Rust, Node.js,
Airflow and llama.cpp forbid it; three converge on `Assisted-by:`, which no harness emits. No
first-party git or vendor guidance on AI attribution trailers exists at all.

---

## [contextSmartZone/](contextSmartZone/) — the zone where an agent still works well

Researched 2026-09-17 from primary sources: vendor documentation and API references, the shipped
source of 17 agent harnesses (including the Claude Code native binary), the 2023–2026 long-context
measurement literature, and the published practice of the teams building these tools.

📖 [guide.md](contextSmartZone/guide.md) · 📋 [rulebook.md](contextSmartZone/rulebook.md) — 40 rules ·
📚 [sources.md](contextSmartZone/sources.md) — ~80 sources

### What the research found

**The smart zone is an order of magnitude smaller than the point where any harness intervenes.**
NoLiMa (ICML 2025) measures *effective length* — "the longest context where a model maintains at
least 85% of its base score" — at **2K–16K tokens** for models advertising 128K–2M
([arXiv:2502.05167](https://arxiv.org/abs/2502.05167)), and BABILong finds models "effectively
utilize only 10-20% of the context"
([arXiv:2406.10149](https://arxiv.org/abs/2406.10149)). Meanwhile the *lowest* compaction trigger in
the field is Gemini CLI's 0.5, and most cluster at 0.8–0.95. **Not one harness intervenes inside the
measured zone** — because a compaction threshold is overflow protection, not quality management. The
progress bar reports headroom, not health.

**Vendors name the decay and never quantify it.** Anthropic's own docs say "As token count grows,
accuracy and recall degrade, a phenomenon known as context rot", and Claude Code's best-practices
page is blunter: "performance degrades as it fills." No vendor page anywhere states a number. The
tell is in the defaults: Anthropic's **API** compacts at 150k of 1M (**15%**) and clears tool results
at 100k (**10%**), while **Claude Code** runs to **~95%**. Same company, same models, same month.
OpenAI makes no degradation claim at all.

**Claude Code's threshold is absolute, which settles a live dispute.** Two contradictory third-party
claims circulate ("~83% with a clamp", "~95%"). Read out of the shipped native binary, the function
is `let r = e - 13000` — **`effective_window − 13,000` tokens, not a percentage**. That is 93.5% of a
200K window but ~95.4% of a raw 1M, so the percentage moves with the window, which is exactly why
field reports disagree. The `Math.min` clamp the "83%" claim asserted **is real**;
`CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` can only compact *earlier*. Both numbers wrong, the mechanism
right.

**Length hurts even when the context is perfect.** The strongest causal result: performance degrades
**13.9%–85%** as input grows *even when models retrieve all relevant information perfectly*, and the
effect **persists when irrelevant content is replaced with whitespace**
([arXiv:2510.05381](https://arxiv.org/abs/2510.05381), EMNLP 2025 Findings). "Keep the context clean"
is not sufficient advice. The goal is not a clean context — it is a short one.

**The failure doesn't look like failure.** In a coding-agent audit study, success fell from **80% to
30%** in extended context while **requirement coverage held at 92–94%**
([arXiv:2607.17937](https://arxiv.org/html/2607.17937)). The agent keeps doing the bulk and drops the
sparse critical obligations — the failure a reviewer skims past. Related: refusal rates climb
**0% → 89.6%** near context ceilings, and Anthropic's own research finds safety monitors degrade
**2×–30×**, with Opus 4.6 recall falling **99.7% → 69%** between 100K and 800K
([arXiv:2605.12366](https://arxiv.org/html/2605.12366v1)).

**"Keep under 40%" is folklore.** Its hardest statement is introduced by its author as "a mental
model", carries no citation, and rests on a simulation whose own parameter is "Each file read uses 5%
of the context window". The competing 10–20% figure *does* have a primary source (BABILong). The
cited number survives contact with its source and the popular one does not — and the measured figure
is **stricter** than the folklore, not looser.

**The number your harness shows you is usually not the number that binds.** Five different
denominators are in use across the field, so "90%" in Codex, Zed and Claude Code are different
quantities. Gemini CLI's docs say `0.7` where its code says `0.5`. Cline's effective trigger is
≈0.81 and exists as no constant. Continue's UI understates because it counts fewer tokens than the
trigger does. Roo Code and Kilo Code both **ship the percentage trigger disabled**. Cursor's staff
concede theirs "can trigger late or incorrectly". And Claude Code's window resolves through
server-side `clientdata` and `experiment` sources — **two vendors can move your boundary without
shipping a release**, and neither documents it.

**Nobody has measured the question you actually have.** Whether *harness auto-compaction specifically*
costs accuracy is unpublished — the literature compares engineered compaction against none, never
on-versus-off. What is measured: simply dropping old tool results matches LLM summarisation's solve
rate at **half the cost** ([arXiv:2508.21433](https://arxiv.org/abs/2508.21433)), and summariser
quality alone is worth **6.5 points** ([arXiv:2607.05378](https://arxiv.org/html/2607.05378)).
Sub-agents, under equal token budgets, **lose** to a single agent — except "in highly degraded
contexts" ([arXiv:2604.02460](https://arxiv.org/html/2604.02460)), which makes context rot the
empirical trigger for fan-out rather than a reason to prefer it.

---

## Conventions

- **Every claim carries a source.** Verbatim quotes where the wording matters.
- **Evidence strength is labelled, always.** Peer-reviewed, vendor documentation, field
  pattern, practitioner measurement, and opinion are not the same thing and are never
  presented as if they were.
- **Conflicts are recorded, not smoothed over.** Where sources disagree — or where a standard's
  own documentation contradicts its implementers — that is the finding.
- **Unverified is stated as unverified.** Each topic ends with what could not be confirmed.
- **Findings are dated.** This field moves fast; a claim without a date is a claim with an
  unknown shelf life.
