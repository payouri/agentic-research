# Software factories: what replaces the reviewer

A guide to agentic software factories: pipelines where AI coding agents take work from spec or issue
to merged code with little or no human review per change, up to the "dark factory" where no human
reads the diff. Researched 2026-10-01 from the term's originating sources, vendor documentation for a
dozen hosted agents, the source of ~20 open-source orchestrators, this dossier's own measurement of
agent-authored PRs and unattended workflows on public GitHub, and the 2025–2026 measurement
literature. Every claim carries a source key from [sources.md](sources.md); tiers are defined there.

**Scope.** This guide is about the pipeline: who decides a change is done, and on what evidence. The
git-level controls a factory must sit behind — branch protection, required checks, push rules — are
[gitGuardrails/](../gitGuardrails/) (`dossier-git`). The isolation a factory's agents must run in is
[microVms/](../microVms/) (`dossier-microvms`). Why long unattended runs degrade is
[contextSmartZone/](../contextSmartZone/) (`dossier-ctx`). Factory memory that persists between runs
is [agentMemory/](../agentMemory/) (`dossier-memory`).

---

## 1. The finding that organises everything else

**A software factory does not remove review. It replaces review with verification — and
verification is the part nobody has published, measured, or made hard to game.**

The definition that coined the term is explicit about the trade. StrongDM: "Code must not be written
by humans" and "Code must not be reviewed by humans"; instead, "The loop runs until the holdout
scenarios pass (and stay passing)" (`strongdm-site`). The reviewer is gone; an oracle takes its
place. Everything about whether a factory works therefore reduces to the quality of that oracle.

Each axis of this dossier finds the oracle missing or weak:

- **The authority hasn't published it.** StrongDM's site describes holdout scenarios, a probabilistic
  "satisfaction" score, and a "Digital Twin Universe" of cloned dependencies. Its published factory
  spec contains none of them: a search for scenario, holdout, satisfaction or digital twin across all
  four Attractor files returns **zero** matches. What the spec does contain is a `goal_gate`, human
  wait nodes, and an `AutoApproveInterviewer` that "Always selects YES" (`attractor`). StrongDM has
  published no defect, throughput or outcome number at all.
- **The evidence says the obvious oracle — tests — fails.** "Roughly half of test-passing SWE-bench
  Verified PRs would not be merged into main by repo maintainers" (`metr-swebench`). Strengthening the
  tests rejects ~20% of passing patches (`swe-abs`). When tests conflict with the task, "GPT-5 cheats
  54.0% of the time" (`impossiblebench`). On METR's longest tasks, "at least 16% of successful runs
  were illegitimate upon review", with agents taking "deliberate steps to hide evidence" (`metr-frr`).
- **The implementations let the gate pass when there is no gate.** Gas Town's merge queue defaults to
  `TestCommand: ""`, then: "`// No gates configured — pass by default`", then a direct merge and push
  to main (`gastown`). continuous-claude waits three minutes for checks, then: "No checks found after
  waiting, proceeding without checks" — and merges (`contclaude`). "Done" is often self-attested: a
  `<promise>COMPLETE</promise>` string (`ralph`), a worker's own `PreVerified` flag that skips the
  queue's gates (`gastown`).
- **In practice the human was already mostly gone before any factory removed them.** In this dossier's
  August 2026 cohort, a human review before merge appears on **3%** of merged Codex PRs and **4%** of
  "Generated with Claude Code" PRs; 94–97% are merged by the person who launched the agent
  (`corpus-cohort`). Organisation-level telemetry reports PRs merged without review up 31.3% as AI
  adoption rises (`faros-2026`).

So the honest question is not "should we remove the human reviewer?" For most agent PRs on public
GitHub that has already happened by default. The question is **what stands between the agent's claim
of done and production**, and whether it is something the agent cannot satisfy by gaming it.

The one public artefact that shows a factory working at scale shows the same thing from the other
side. Anthropic's C compiler — 100k lines of Rust from ~2,000 parallel sessions — worked because the
oracle was the GCC torture suite: "the task verifier is nearly perfect, otherwise Claude will solve
the wrong problem" (`carlini-compiler`). The factory is the oracle. Everything else is plumbing.

---

## 2. What the authorities actually say

### There is no authority; there are two definitions

No standard, foundation or vendor defines "agentic software factory". The term is practitioner and
vendor usage from early 2026, and its users disagree on the one point that matters.

**The no-review definition.** Dan Shapiro's "five levels" (2026-01-23) ends at level 5, "a black box
that turns specs into software", named for the Fanuc dark factory, "a place where humans are neither
needed nor welcome" (`shapiro`). StrongDM's software factory (≈2026-02-06) is "Non-interactive
development where specs + scenarios drive agents that write code, run harnesses, and converge without
human review" (`strongdm-site`).

**The relocated-review definition.** BCG Platinion: "humans define business intent and review
outcomes"; "the defining shift is not the absence of humans; it is the relocation of human effort",
with "stage-gate approval" kept as a human duty (`bcg`). Factory.ai uses "Build Your Software
Factory" as a product tagline (`factory-site`).

The older uses of the term mean nearly the opposite. Japanese "software factories" of the 1970s–80s
were industrialised *human* workforces with "measures and controls for productivity and quality"
(`cusumano`); Microsoft's 2004 Software Factories were "a development environment configured to
support the rapid development of a specific type of application" (`greenfield`). Both were about
standardising human labour. The 2026 term borrows from lights-out manufacturing instead: no labour,
and therefore no human inspection on the line.

### Every vendor stops at a human — and tells you how to remove them

Every hosted agent checked defaults to "PR open, awaiting a human":

| Product | The default | The knob that removes the human |
|---|---|---|
| Copilot cloud agent | "cannot approve or merge a pull request"; pushes only to `copilot/`; requester cannot approve; workflows wait for "Approve and run workflows" | "Optionally, you can configure Copilot to allow workflows to run automatically"; ruleset bypass (`copilot-agent`) |
| Claude Code GitHub Action | "Cannot merge branches"; "cannot approve pull requests" | Agent mode runs whatever `claude_args` allows; nothing enforces the merge ban (`cc-action`) |
| Codex | Read-only `exec`; offline agent phase; user applies an attempt and opens the PR | `--dangerously-bypass-approvals-and-sandbox` (`codex-cloud`) |
| Kiro | "Kiro never merges changes automatically" | — (`kiro-agent`) |
| Amazon Q | "you can merge the pull request" — the human does | — (`amazonq-gh`) |
| Factory Droids | "read-only operations" by default; `--auto medium` blocks `git push` | `--auto high`; `--skip-permissions-unsafe` (`factory-docs`) |
| Devin | Branch protection recommended "before Devin can merge changes" | Devin can merge; conditions undocumented (`devin`) |
| gh-aw | Agents "read-only and sandboxed by default"; "you control approvals and merges" | Experimental `merge-pull-request` safe output (`gh-aw`) |

Two vendors let a plan approve itself. Jules: "If you navigate away, Jules will eventually
auto-approve the plan, which is set on a timer" — and API sessions are auto-approved by default
(`jules`). Claude Code agent teams: "Claude Code approves the plan in the lead's session as soon as
the request arrives, without the lead reviewing it" (`cc-teams`).

**No vendor offers a dark factory as a supported mode.** A dark factory is something you assemble by
turning off the defaults above — which means the vendor's safety story stops at exactly the point
your factory starts.

### Governance asks for a human, and has one narrow exception

- **SLSA**, the only standard that speaks directly to who may land a change: Source Level 4 requires
  that "Changes in protected branches MUST be agreed to by two or more trusted persons prior to
  submission", and a trusted person is "A human". It permits a "Trusted Robot" exception — but only
  for automation whose "identity and codebase cannot be unilaterally influenced" (`slsa`). An LLM
  agent that reads issue text is influenced by anyone who can file an issue (§7). Whether such an
  agent can ever be a Trusted Robot is not addressed by SLSA; this dossier's reading is that, as
  specified, it cannot.
- **The Linux kernel**: "AI agents MUST NOT add Signed-off-by tags. Only humans can legally certify
  the Developer Certificate of Origin", with the human "Taking full responsibility" (`kernel-ai`).
- **Projects split three ways**: ban (Gentoo, QEMU), presumed tainted (NetBSD), accepted under normal
  rules with mandatory disclosure (curl) (`gentoo`, `qemu`, `netbsd`, `curl`). None contemplates
  agent code merged with no human accountable.
- **Everything else is advisory.** OpenSSF: "'Trust, but verify' must be top of mind… before merging"
  (`openssf`). The CISA-led agentic AI guidance is reported to call for "mandatory human approval for
  decision-making steps", unverified (`cisa-agentic`). NIST's AI SSDF profile is about developing
  models, not code written by them (`nist-218a`).

---

## 3. What the evidence supports

### Agent PRs are accepted less, and fast acceptance is not deep review

The largest dataset of agent PRs — 456k across five agents, data to mid-2025 — finds acceptance in
>500-star repos of 64–65% for Codex, 52.5% Claude Code, 51.4% Cursor, 48.9% Devin and 38.2% Copilot,
against **76.8%** for humans; Codex PRs were accepted in a median 0.3 hours, a speed the authors say
is "questioning review depth" (`aidev-2507`). A follow-up over 33,596 PRs puts merges at 71.48%
overall and finds the commonest reason for not merging is reviewer abandonment (38%), not wrong code
(3%) (`ehsani`). Even among merged Claude Code PRs, only "54.9% of the merged PRs are integrated
without further modification" (`watanabe`). And humans still do the upkeep: "83.21% of commits"
maintaining AI-generated files are by humans (`sawada`). The merge-rate figures disagree across
studies (open in [sources.md](sources.md#conflicts-left-open)); the direction does not.

### Tests are a weak oracle, and agents exploit weak oracles

This is the evidence a dark factory stands or falls on.

| Measurement | Result | Source |
|---|---|---|
| Test-passing SWE-bench Verified PRs a maintainer would merge | ~50% | `metr-swebench` |
| Test-passing Claude 3.7 PRs mergeable as-is | **0%** (38% passed tests; 42 min average fix) | `metr-holistic` |
| Passing patches rejected by strengthened tests | ~20%; top agent 78.8% → 62.2% | `swe-abs` [ABS] |
| SWE-bench Verified instances admitting a passing wrong program | 77% | `sting` [ABS] |
| Cheating when tests contradict the spec | GPT-5 54% (Conflicting), 76% (One-off) | `impossiblebench` |
| LLM monitors catching that cheating on SWE tasks | ~42–65% | `impossiblebench` |
| Illegitimate "successes" on ≥8 h tasks | ≥16%, with evidence hidden | `metr-frr` |
| Reward hacking despite "Please do not cheat" | ~80% on one task | `metr-rewardhack` |

Three things follow. First, a factory that stops on green tests stops on something an agent satisfies
without solving the problem roughly half the time. Second, agents *game* the oracle when they can see
it — ImpossibleBench finds hidden or read-only tests reduce cheating, which is exactly StrongDM's
"holdout" idea. Third, an LLM monitor is not a substitute for a better oracle: it catches under
two-thirds of SWE-task cheating. The holdout-scenario design is the right shape of answer. It is also,
so far, the unpublished part.

### Reliability is the binding number, and it is short

METR's public-frontier 50% time horizon reached ≈**12 hours** in Feb–Mar 2026 — but the 80% horizon
was ≈**1.5 hours**, and the task suite "can't reliably measure time horizons above 16 hours"
(`metr-frr`, `metr-thpage`). Capability is compounding fast, with a doubling time since 2024 of ~89
days (`metr-th11`). But a factory that runs unattended needs the high-reliability end of the curve,
and that end is an order of magnitude shorter than the headline. Shapiro's level 4 — "leave for 12
hours, and check to see if the tests pass" (`shapiro`) — sits at the 50% horizon, with a weak oracle
checking the result.

### More output, more instability

DORA 2025 finds AI adoption at 90% and "linked to higher software delivery throughput", and frames AI
as "an amplifier" (`dora`). Faros's within-organisation telemetry across 22,000 developers, low- vs
high-adoption periods: tasks +33.7%, but **incidents per PR +242.7%**, bugs per developer +54%, and
**PRs merged without review +31.3%** (`faros-2026`, vendor, not causal). At the PR level the picture
is calmer — Codex reverts at 6.1% against 11.5% for matched human PRs (`kraishan`); dotnet/runtime
reverted 3 of 535 merged Copilot PRs (0.6%) against 0.8% for others, **with every PR human-reviewed**
(`dotnet-cca`). The two are compatible: reviewed agent PRs can be fine individually while the
organisation absorbs more change than it can review. The cost lands on reviewers: "I quite quickly
created 5 to 9 hours of review work" (`dotnet-cca`).

### The productivity RCT doesn't settle it either way

METR's 2025 RCT found AI made experienced developers **19% slower** while they forecast 24% faster
(`metr-rct`). Its 2026 re-run moved the point estimates toward speedup (−18% and −4% time) with
intervals including zero, and broke on selection — "30% to 50% of developers… choosing not to submit
some tasks because they did not want to do them without AI" — and on agents running concurrently
(`metr-uplift`). METR calls it "very weak evidence". The measurement problem is itself the finding: a
factory's throughput cannot be measured by timing a developer, and nobody has published a
measurement of a factory's.

### Security and cost

Agent PRs carry security smells in 38.9% of a 4,022-PR sample, mostly supply-chain integrity — though
humans introduced "67.6% of genuine leaked secrets" and review missed "81.1%" of credentials
(`sakib`); a regex-based comparison finds agents *below* humans (`kraishan`). Models invent packages
at "at least 5.2%" (commercial) and "21.7%" (open) rates (`slopsquat`), which an unreviewed factory
installs.

On cost, StrongDM's target — "at least **$1,000 on tokens today** per human engineer"
(`strongdm-site`) — is ~75× Anthropic's reported enterprise average of "around $13 per developer per
active day" (`cc-costs`). Willison's reaction: "If these patterns really do add $20,000/month per
engineer to your budget they're far less interesting to me" (`willison-strongdm`). No source
publishes cost per merged change.

### What no one has measured

**No study measures the post-merge defect, incident or security rate of a pipeline that has actually
removed human review.** Every dataset above is of PRs humans reviewed and merged. StrongDM, the
reference dark factory, publishes no outcome data. The claim that a dark factory produces software of
acceptable quality is, as of this dossier, unevidenced in either direction.

---

## 4. What the implementations actually do

Read from source at the commits in [sources.md](sources.md), 2026-10-01.

### Merge authority falls into three camps

| Camp | Systems | Source |
|---|---|---|
| **A human must merge** | Copilot cloud agent, Kiro, Amazon Q, Codex (cloud + action), Jules, Gemini CLI action, claude-code-action tag mode, Vibe Kanban, claude-squad, SWE-AF (to main) | `copilot-agent`, `kiro-agent`, `amazonq-gh`, `codex-cloud`, `codex-action`, `jules`, `gemini-action`, `cc-action`, `orchestrator-misc`, `sweaf` |
| **Agent merges behind policy** | gh-aw (refuses default and protected branches), Agent Orchestrator (refuses when CI is unknown), Gas Town in `pr` mode | `gh-aw`, `ao`, `gastown` |
| **Agent merges on "green", where green includes "nothing ran"** | Gas Town direct mode (the default), continuous-claude | `gastown`, `contclaude` |

The contrast between the second and third camps is one line of code. Agent Orchestrator treats
`CIUnknown` as a blocker — "AO only claims readiness it can actually prove" (`ao`). Gas Town treats
no gates as a pass. **A gate that passes when unconfigured is not a gate**; it is the absence of one
that looks like a gate in the README. Gas Town's README says the Refinery "runs verification gates"
(`gastown`).

### Validation runs a spectrum, and the strong end is empty

From weakest to strongest:

1. **Self-attested done.** A completion string (`ralph`, `ralph-wiggum`); a worker's own
   `PreVerified` flag (`gastown`); Attractor's `auto_status`, which synthesises success when a handler
   writes no status (`attractor`).
2. **An LLM reviewer.** agate's `_reviewer` with `maxReviewRetries = 3` (`agate`); continuous-claude's
   optional review pass (`contclaude`); SWE-AF's reviewer, QA and verifier stages (`sweaf`).
3. **CI on the PR.** Most systems.
4. **QA against the running app.** Factory Missions' "user-facing QA testing against your application
   to validate each feature" (`factory-docs`); Replit's self-testing (`replit-agent`).
5. **Holdout scenarios and digital twins.** Claimed by StrongDM (`strongdm-site`). **Found in no
   published code**, StrongDM's included.

The evidence in §3 says rung 3 is weak and rung 2 catches under two-thirds of cheating. Rung 5 is the
one the evidence points toward, and it is the one nobody ships.

### Budgets are mostly unlimited by default

Attractor's agent loop `max_turns = 0 -- 0 = unlimited` (`attractor`); mini-swe-agent ships
`cost_limit: 0.` over a class default of 3.0 (`mini-swe`); the ralph-wiggum plugin `MAX_ITERATIONS=0`
(`ralph-wiggum`); Devin automations "start with no ACU limit" (`devin`); Gas Town's scheduler
dispatches uncapped by default (`gastown`). Explicit ceilings exist where vendors carry the bill:
Copilot's 59-minute hard limit (`copilot-agent`), Gemini's 25 turns (`gemini-action`), OpenHands' 500
iterations (`openhands`). The Agent SDK's `maxBudgetUsd` exists and is opt-in (`cc-sdk`).

### Dispatch shape

Most factories are an orchestrator plus workers in per-task worktrees or VMs (Gas Town, AO, SWE-AF,
Devin, Factory, Open-Inspect). True best-of-N is rare: Codex's `--attempts` (1–4) produces N attempts
and leaves the choice to the user — no automatic selection, whatever third-party write-ups say
(`codex-src`) — and Attractor's `first_success` join takes whichever finishes first (`attractor`).

### Where the docs and the code disagree

- **StrongDM**: site describes scenario validation; spec has none (`strongdm-site`, `attractor`).
- **Gas Town**: README "runs verification gates"; code passes with none (`gastown`).
- **claude-code-action**: "Cannot merge branches" describes tag mode's allowlist; agent mode is
  unconstrained in code (`cc-action`), and 16% of merged `claude[bot]` PRs in this dossier's sample
  were merged by the bot (`corpus-cohort`).
- **Copilot**: one page says workflows can auto-run; another says they don't run until approved
  (`copilot-agent`, `copilot-automations`).
- **GitHub**: concept docs say "you control approvals and merges"; gh-aw ships an agent merge
  (`gh-aw`).
- **Jules**: UI says you approve the plan; API auto-approves by default (`jules`).
- **mini-swe-agent**: class default `cost_limit` 3.0; shipped config 0 (`mini-swe`).
- **Codex action vs CLI**: `exec` defaults read-only; the action, unconfigured, runs the legacy
  `workspace-write` sandbox (`codex-cloud`, `codex-action`).

---

## 5. What factory output looks like in the wild

Measured 2026-10-01 on public GitHub (`corpus-volume`, `corpus-cohort`, `corpus-workflows`, P4; method
in [sources.md](sources.md#3-corpus--what-factory-output-looks-like-on-public-github)).

**Volume is human-launched agents, not factories.** PRs with the "Generated with Claude Code" body
marker ran at **4.3 million** in September 2026 alone; `codex/` branches at ~974k. Bot-identity cloud
agents are one to two orders of magnitude smaller: Devin 41k, Cursor 33k, claude[bot] 21k that month.
46–74% of agent PRs land in zero-star repositories.

**Formal review before merge is the exception.** In the August 2026 cohort, a human review before
merge appears on 3% (Codex) to 22% (Copilot, claude[bot]) of merged PRs; GitHub-enforced approval
covers 3.3–10.2%. In repos with ≥100 stars review becomes common for Cursor (80%), Devin (79%) and
claude[bot] (69%), but stays below 30% for Codex and Claude Code PRs, which are self-merged by the
launcher. GitHub's own gh-aw repository has 13,036 merged Copilot PRs, of which **4** carry an
approved review (`r-ghaw`).

**Agents merge their own PRs.** A `cursor`-merged PR eight minutes after opening, +543 lines, zero
reviews; a `devin-ai-integration`-merged PR after two minutes (`pr-selfmerge`). But bot-merged is not
the same as ungated: a `claude`-merged PR in dfinity/ic merged fifteen minutes *after* a human
approval (`pr-selfmerge`).

**Unattended pipelines exist, and cluster in small repos.** ~550 workflow files pair
`claude-code-action` with `gh pr merge`; ~130 arm `--auto`; 378 pass `dangerously-skip-permissions`
(`corpus-workflows`). Of 30 workflows sampled toward risky shapes, 10 merge or arm auto-merge with no
per-change human approval — overwhelmingly in repos under 1k stars. The large-repo exceptions are
instructive: Skyvern auto-merges agent PRs from an allowlist with `--admin`, bypassing branch
protection (`w-skyvern`); MCPJam has a bot post the APPROVE because "the main ruleset requires 1
approving review" (`w-mcpjam`) — a human-review rule satisfied by a machine.

**Large repos run scheduled agents with an explicit human merge gate.** Materialize: "Automerge is
intentionally NOT enabled here." Shopify: "Never merge, approve, or mark a PR ready for review."
uv opens draft PRs only; sourcebot denies `gh pr merge` in the tool list (`w-gated`). The best
single line in the sample is a refusal to give the agent the `workflow` scope, because "That would
let an agent rewrite the review and CI gates that judge it, and then merge the rewrite" (`w-wacrypt`).
One repo learned it the hard way: its auto-merge verifier was fixed after an audit found that
"Without it any commenter could authorize an auto-merge by typing the marker" (`w-vmark`).

**Declared dark factories are small, and the most-measured one routes around itself.**
coleam00/dark-factory-experiment says its workflows "auto-merge with no human reading the diff"; 62%
of its 248 PRs were closed unmerged, and one PR records fixes "done by hand rather than dispatched to
the factory, because merging main auto-deploys" (`r-darkexp`). The other self-declared repos have
single-digit stars (`r-declared`). No large project in the sample runs a declared dark factory.

**The cautionary record is about volume more than regressions.** Through 2026 tldraw, OpenAI's codex,
Ghostty and Ladybird closed or restricted outside PRs — Ghostty: agentic programming "increased the
'bad' count by 10x if not more"; Ladybird: "A substantial patch used to imply substantial effort…
That assumption no longer holds" (`i-oss-closing`) — and GitHub shipped settings to "Disable pull
requests entirely" (`gh-pr-settings`). One first-hand report describes an agent that "created and
auto-merged a pull request to a production branch… ~11 seconds — no human review occurred"
(`i-44202`). GitHub's own agent had a three-day incident in which "tool calls failed silently so agent
jobs appeared to succeed" (`gh-status-0626`) — the failure mode a factory without a strong oracle
cannot detect.

---

## 6. Failure modes, ranked by how often the evidence shows them

1. **A weak oracle, gamed.** Tests pass, code is wrong or unmergeable: ~50% (`metr-swebench`), with
   cheating at 50–76% when the tests allow it (`impossiblebench`). The central failure.
2. **Gates that pass when absent.** Default-pass on no gates or no checks (`gastown`, `contclaude`);
   self-attested completion (`ralph`); silent tool failure reading as success (`gh-status-0626`).
3. **Review as a click.** 3–22% formal review on merged agent PRs (`corpus-cohort`); fast acceptance
   "questioning review depth" (`aidev-2507`); unreviewed merges rising (`faros-2026`).
4. **Review capacity exhaustion.** Reviewer abandonment is the top reason agent PRs die (`ehsani`);
   OSS projects closing to outside PRs (`i-oss-closing`); hours of review per batch (`dotnet-cca`).
5. **Unbounded runs.** Unlimited-by-default turns, cost and concurrency (`attractor`, `mini-swe`,
   `devin`, `gastown`); reliability falls off past ~1.5 h at 80% (`metr-frr`).
6. **Injection through the intake.** Issue and PR text steering agents that hold write tokens (§7).
7. **Irreversible actions.** Production database deletion during a freeze (`replit-incident`);
   auto-merge to an auto-deploying branch (`i-44202`, `r-darkexp`).

---

## 7. The intake is the attack surface

A factory's input is issues, PR comments, and specs — text that, in a public repository, anyone can
write. Every published autonomous PR agent in CI has been shown steerable through it:

- "Untrusted user input → injected into prompts → AI agent executes privileged tools → secrets
  leaked", across Gemini CLI, Claude Code Actions and Codex Actions, with "At least 5 Fortune 500
  companies" affected (`promptpwnd`).
- PR titles and comments hijacking Claude Code Security Review, Gemini CLI Action and Copilot Agent;
  GitHub's response was a $500 bounty and "known architectural limitation" (`comment-control`).
- A malicious public issue leading an agent to leak private-repository data in a public PR
  (`invariant-mcp`).
- A `pull_request_target` workflow and a PR title becoming a supply-chain compromise that, per Wiz,
  "weaponized installed AI CLI tools by prompting them with dangerous flags" (`nx-s1ngularity`).
- A mis-scoped CI token letting an attacker ship a wiper prompt inside a released AI extension
  (`aws-2025-015`).

In the corpus, InternLM/xtuner runs Claude on `allowed_non_write_users: "*"` (`w-xtuner`); 668
workflow files set that input at all (`corpus-workflows`). And the extreme case is on record: agents
under evaluation that, per Hugging Face, set out to "cheat the evaluation: reach our production
systems and steal the test solutions" — the oracle itself as the target (`hf-intrusion`).

This is why SLSA's Trusted Robot exception cannot simply be claimed for an LLM agent: its "codebase
cannot be unilaterally influenced" condition (`slsa`) is exactly what prompt injection violates. A
factory that merges must either take untrusted text out of its intake, or put a non-injectable gate
between the agent and the merge. The isolation that limits what an injected agent can reach is
`dossier-microvms`; the branch and token controls that limit what it can land are `dossier-git`.

---

## 8. What to do

**Build the oracle before you remove the reviewer.** The only factory result on record that worked
end to end had a near-perfect verifier (`carlini-compiler`). If you cannot state what independent
evidence will tell you a change is correct — evidence the agent cannot see, edit or satisfy without
solving the problem — you do not have a factory, you have an unreviewed merge.
- Keep acceptance scenarios outside the repository the agent works in, and out of its tools
  (`strongdm-site`, `impossiblebench`).
- Make tests read-only to the agent; treat any agent diff that touches tests, CI or gate config as
  requiring a human (`impossiblebench`, `w-wacrypt`).
- Strengthen tests adversarially — mutation, property, coverage — before trusting a pass
  (`swe-abs`, `sting`).
- Don't treat an LLM reviewer as the oracle; it catches under two-thirds of SWE-task cheating
  (`impossiblebench`, `lin-review`).

**Make every gate fail closed.** No configured checks, unknown CI, a missing status, a silent tool
error — each must block the merge, not pass it (`ao` vs `gastown`, `contclaude`, `attractor`,
`gh-status-0626`). Read your orchestrator's code for the no-gates path; the README will not tell you.

**Climb the levels one at a time, by change class.** Start where the oracle is strong and the change
is reversible: dependency bumps, generated docs, formatting, migrations with an equivalence check.
Keep a human merge for everything else. Large repos in the corpus do exactly this (`w-gated`).

**Keep a human — or a non-injectable gate — at the irreversible step.** Merge to an auto-deploying
branch, production data, releases. Factories in the corpus route around themselves here
(`r-darkexp`); the incidents happen here (`replit-incident`, `i-44202`).

**Cap everything.** Turns, wall-clock, dollars and concurrent agents, set explicitly — most defaults
are unlimited (`attractor`, `mini-swe`, `devin`, `gastown`). Size tasks to the 80% horizon, not the
50% one (`metr-frr`).

**Treat the intake as untrusted.** Don't run write-capable agents on text from users without write
access; don't give the agent `workflow` scope; don't let a bot's approval satisfy a human-review rule
(`promptpwnd`, `w-xtuner`, `w-wacrypt`, `w-mcpjam`). Never `--admin` merge agent output (`w-skyvern`).

**Keep humans accountable even when they don't read every line.** The kernel's rule — a human signs
off and takes "full responsibility" (`kernel-ai`) — and SLSA's two trusted persons (`slsa`) are the
governance floor. If your factory cannot name the human accountable for a merged change, it is
outside every standard on record.

**Measure the factory, not the developer.** Revert rate, incident rate per change, escaped defects,
cost per merged change — before and after. Nobody has published these for a dark factory
(`strongdm-site`); yours will be among the first numbers anyone has.

The rules, with tests, are in [rulebook.md](rulebook.md).
