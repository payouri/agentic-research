# Software factories source lexicon

Every source behind [guide.md](guide.md) and [rulebook.md](rulebook.md), keyed for citation.
Gathered 2026-10-01.

Trust tiers: **P1** primary spec, vendor documentation, policy text, or shipped source code · **P2**
peer-reviewed or arXiv research · **P3** vendor engineering blog / industry research with disclosed
method · **P4** practitioner report with measurement (including this dossier's own counts, method
stated) · **P5** opinion, anecdote, news, or unverified secondary.

Source code was read from shallow clones at the commit stated. Quotes marked **(WF)** passed through a
summarising fetcher and may be lightly paraphrased. **[ABS]** marks a number taken from an abstract
only. OpenAI's own pages (openai.com/index/…) returned HTTP 403 throughout; claims resting on them are
secondary.

A note on tiering definitions: StrongDM's and Dan Shapiro's pages are the *canonical statement* of
what the agentic term means, so they are cited as the authority for the definition. As *evidence that
the approach works*, they carry no disclosed outcome data and are weighed accordingly in the guide.

Sibling dossiers cited by key: `dossier-git` ([gitGuardrails/](../gitGuardrails/)), `dossier-microvms`
([microVms/](../microVms/)), `dossier-ctx` ([contextSmartZone/](../contextSmartZone/)),
`dossier-memory` ([agentMemory/](../agentMemory/)), `dossier-agentsmd` ([agentsMd/](../agentsMd/)).

---

## 1. Primary — what the term's authors, the vendors, and the governance bodies say

### Who defined the agentic usage

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `strongdm-site` | StrongDM AI, [Software Factory](https://factory.strongdm.ai/), [/principles](https://factory.strongdm.ai/principles), [/techniques](https://factory.strongdm.ai/techniques) | P1 (definition) · P5 (outcomes) | page undated; announced ≈2026-02-06 (Justin McCarthy); `strongdm/attractor` created 2026-02-05T23:40Z | "Non-interactive development where specs + scenarios drive agents that write code, run harnesses, and converge without human review"; "Code must not be written by humans" / "Code must not be reviewed by humans"; "If you haven't spent at least **$1,000 on tokens today** per human engineer, your software factory has room for improvement"; "The loop runs until the holdout scenarios pass (and stay passing)"; scenarios "stored outside the codebase" like a holdout set **(WF)**; probabilistic "satisfaction" **(WF)**; Digital Twin Universe: "Clone the externally observable behaviors of critical third-party dependencies"; "Separate interactive work from fully specified work". No defect, throughput or outcome data |
| `attractor` | [strongdm/attractor](https://github.com/strongdm/attractor) `attractor-spec.md`, `coding-agent-loop-spec.md`, `unified-llm-spec.md`, README | P1 | fb57a55, 2026-03-17; 1,323★ | Specs only ("NLSpecs"); `goal_gate` nodes "must reach `SUCCESS` or `PARTIAL_SUCCESS` before the pipeline can exit"; `max_parallel` default `"4"`; `wait.human` "Human-in-the-loop gate"; `AutoApproveInterviewer` "Always selects YES"; `auto_status` synthesises `{"outcome": "success"}` when a handler writes none; `max_turns : Integer = 0 -- 0 = unlimited`; `default_max_retries` 0. **`grep -ci "scenario\|holdout\|satisfaction\|digital twin"` = 0 in all four files** (re-run by this dossier) |
| `agate` | [strongdm/agate](https://github.com/strongdm/agate) `internal/workflow/next.go`, `internal/agent/claude.go` | P1 | ea95448, 2026-02-23 | `const maxReviewRetries = 3`; LLM `_reviewer` gate; exit 255 = "human intervention needed"; `--dangerously-skip-permissions`; 10-minute task timeout; no push/merge path found |
| `cxdb` | [strongdm/cxdb](https://github.com/strongdm/cxdb) | P1 | c262588, 2026-08-28 | "AI Context Store", Turn DAG; commits attributed to human accounts |
| `shapiro` | Dan Shapiro, [The Five Levels: from Spicy Autocomplete to the Software Factory](https://www.danshapiro.com/blog/2026/01/the-five-levels-from-spicy-autocomplete-to-the-software-factory/) | P5 | 2026-01-23 (page) | Level 4: "You write a spec… Then you leave for 12 hours, and check to see if the tests pass"; level 5: "a black box that turns specs into software"; the Fanuc dark factory, "a place where humans are neither needed nor welcome"; practitioners are "small teams, less than five people" |
| `willison-levels` | Simon Willison, [The Five Levels](https://simonwillison.net/2026/Jan/28/the-five-levels/) | P5 | 2026-01-28 | Link post (explains the date conflict) |
| `willison-strongdm` | Simon Willison, [How StrongDM's AI team build serious software without even looking at the code](https://simonwillison.net/2026/Feb/7/software-factory/) | P5 | 2026-02-07 | "If these patterns really do add $20,000/month per engineer to your budget they're far less interesting to me"; "what it takes to have agents prove that their code works without needing to review every line" |
| `yegge-shape` | Steve Yegge, [The Shape of Things to Come](https://yegge.ai/essays/the-shape-of-things-to-come/) | P5 | Aug 2026 | "Gas Town fell apart at the seams with Opus 4.7"; "by next year, human code review is completely done and gone"; wish factories that accept "only GHIs" |
| `bcg` | BCG Platinion, [The Agentic Software Factory](https://www.bcgplatinion.com/insights/the-agentic-software-factory) | P5 | 2026-03-26 | "humans define business intent and review outcomes"; "the defining shift is not the absence of humans; it is the relocation of human effort"; "stage-gate approval"; "3 to 5x" (no method) |
| `factory-site` | Factory, [factory.com](https://factory.com/) (307 from factory.ai) | P1 | fetched 2026-10-01 | Tagline "Build Your Software Factory"; "agent-native software development platform" |
| `oai-harness` | OpenAI, [Harness engineering](https://openai.com/index/harness-engineering/) via [InfoQ](https://www.infoq.com/news/2026/02/openai-harness-engineering-codex) | P5 | Feb 2026; primary 403 | Snippets only: "Humans steer. Agents execute."; "0 lines of manually-written code"; ~1,500 PRs, three engineers. **Unverified** |

### Vendor products: what is mandated and what is left to you

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `copilot-agent` | GitHub Docs: [About Copilot cloud agent](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent), [Risks and mitigations](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/risks-and-mitigations) (github/docs 10844e1) | P1 | 2026-09-30 | "Only users with write access… can trigger"; "the agent can only push to that branch" (`copilot/`); "cannot mark its pull requests as 'Ready for review' and cannot approve or merge a pull request"; requester cannot approve; workflows wait for "**Approve and run workflows**"; "Optionally, you can configure Copilot to allow workflows to run automatically"; rulesets can grant bypass; "maximum execution time of 59 minutes" |
| `copilot-automations` | github/docs `about-automations.md` | P1 | 2026-09-30 | Workflows "don't run on a pull request until a user with write access approves them" — contradicts the optional auto-run in `copilot-agent` |
| `gh-aw` | [github/gh-aw](https://github.com/github/gh-aw) + concept docs; `actions/setup/js/merge_pull_request.cjs`; ADR-52541 | P1 | v0.90.1, 1f75ecc, 2026-09-30 | "Agent jobs are read-only and sandboxed by default… Safe outputs buffer configured writes"; concept page: "you control approvals and merges"; experimental `merge-pull-request` safe output refusing the default branch (docs) **and** protected branches (code, `target_branch_protected`); `create-pull-request` `max: 1`, `draft: true`; ADR-52541 (Draft, 2026-08-13) `approve-workflow-run` |
| `cc-action` | [anthropics/claude-code-action](https://github.com/anthropics/claude-code-action) `docs/capabilities-and-limitations.md`, `docs/security.md`, `src/modes/tag/index.ts`, `scripts/git-push.sh`, `examples/agent-approval-check.yml`; [docs page](https://code.claude.com/docs/en/github-actions) | P1 | v1.0.238, 12dd8d7, 2026-09-30 | "For security reasons, Claude cannot approve pull requests"; "Cannot merge branches, rebase, or perform other git operations beyond pushing commits"; tag mode `--permission-mode acceptEdits --allowedTools "…"` (re-read by this dossier); agent mode takes whatever `claude_args` allows — **the merge limit is the tag-mode allowlist, not an enforced ban**; push wrapper "only allows `origin <ref>` with no flags"; example "Require human approvals on PRs that contain agent-authored commits"; scheduled runs skip the write-access check; "review Claude's changes before merging" (advice) |
| `cc-teams` | Anthropic, [Orchestrate teams of Claude Code sessions](https://code.claude.com/docs/en/agent-teams) | P1 | fetched 2026-10-01 | "In non-interactive mode… Claude doesn't spawn teammates"; "Claude Code approves the plan in the lead's session as soon as the request arrives, without the lead reviewing it"; "There's no hard limit on the number of teammates"; "Letting a team run unattended for too long increases the risk of wasted effort" |
| `cc-sdk` | `@anthropic-ai/claude-agent-sdk` 0.3.286 `sdk.d.ts` ([npm](https://www.npmjs.com/package/@anthropic-ai/claude-agent-sdk)) | P1 | 2026-09-30 | `maxTurns?`, `maxBudgetUsd?` → `error_max_budget_usd`; bypass needs `allowDangerouslySkipPermissions` |
| `cc-costs` | Anthropic, [Manage costs effectively](https://code.claude.com/docs/en/costs) | P1 | fetched 2026-10-01 | "average cost is around $13 per developer per active day and $150-250 per developer per month… below $30 per active day for 90% of users"; agent teams "approximately 7x more tokens" |
| `codex-cloud` | OpenAI, [Codex cloud](https://learn.chatgpt.com/docs/cloud), [Cloud environments (Legacy)](https://learn.chatgpt.com/docs/environments/cloud-environment), [non-interactive mode](https://learn.chatgpt.com/docs/non-interactive-mode), [approvals & security](https://learn.chatgpt.com/docs/agent-approvals-security.md) | P1 | 308s from developers.openai.com; fetched 2026-10-01 | "Agent internet access is off by default"; "secrets are removed before the agent phase starts"; `codex exec` "runs in a read-only sandbox" by default; "treat Codex suggestions like any other PR: run targeted verification, review diffs" (advice); `--dangerously-bypass-approvals-and-sandbox` opt-in |
| `codex-src` | [openai/codex](https://github.com/openai/codex) `codex-rs/cloud-tasks/src/cli.rs` | P1 | dd90f16, 2026-10-01 | `/// Number of assistant attempts (best-of-N).`, default 1, "attempts must be between 1 and 4"; user applies one with `apply --attempt N` — no automatic selection |
| `codex-action` | [openai/codex-action](https://github.com/openai/codex-action) | P1 | v1.12, 8636508, 2026-08-20 | Never pushes; `drop-sudo` default; legacy `workspace-write` sandbox when neither `sandbox` nor `permission-profile` is set |
| `gemini-action` | [google-github-actions/run-gemini-cli](https://github.com/google-github-actions/run-gemini-cli) | P1 | 387c8dd, 2026-08-21 | Plan → `/approve` by OWNER/MEMBER/COLLABORATOR → execute; no merge tool in `includeTools`; `maxSessionTurns: 25` |
| `jules` | Google, [Jules docs](https://jules.google/docs/), [usage limits](https://jules.google/docs/usage-limits), [review plan](https://jules.google/docs/review-plan), [API](https://developers.google.com/jules/api) | P1 | API updated 2025-11-10 | "If you navigate away, Jules will eventually auto-approve the plan, which is set on a timer"; API "sessions… will have their plans automatically approved" by default; PR only with `AUTO_CREATE_PR`; 15/100/300 daily tasks |
| `cursor-cloud` | Cursor, [Cloud agents](https://cursor.com/docs/cloud-agent), [Automations](https://cursor.com/docs/cloud-agent/automations) | P1 | undated | Separate branch, push for handoff; review approvals "run as `cursor`"; Automations "cannot merge" but may approve; spend limit at first use; "Inputs may lead to misleading or malicious memories" |
| `devin` | Cognition, [GitHub integration](https://docs.devin.ai/integrations/gh.md), [Devin Review](https://docs.devin.ai/work-with-devin/devin-review.md), [Automations](https://docs.devin.ai/product-guides/automations.md), [advanced capabilities](https://docs.devin.ai/work-with-devin/advanced-capabilities.md) | P1 | undated | "ensure all required checks pass before Devin can merge changes"; "Devin will never create commits or comments on behalf of a user without the user explicitly initiating the action"; "New automations start with no ACU limit and a **Rate limit** of 50 runs per hour"; child sessions with "ACU limits" |
| `factory-docs` | Factory, [Droid Exec](https://docs.factory.com/droid-exec/overview.md), [Missions](https://docs.factory.com/missions/overview.md), [Missions planning](https://docs.factory.com/missions/planning.md) | P1 | 308 from docs.factory.ai | "The default is spec-mode… only allowed to execute read-only operations"; `--auto medium` blocks "`git push`, sudo commands, production changes"; `--skip-permissions-unsafe` only "in completely isolated environments"; Missions: "user-facing QA testing against your application to validate each feature"; "Is parallelization necessary? … We are testing this." |
| `amazonq-gh` | AWS, [Amazon Q Developer in GitHub](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/github-feature-development.html) | P1 | preview | "If you're satisfied… you can merge the pull request" — human merges |
| `kiro-agent` | Kiro, [Autonomous agent](https://kiro.dev/docs/autonomous-agent/), [GitHub](https://kiro.dev/docs/autonomous-agent/github/) | P1 | 2026-09-02 | "Kiro never merges changes automatically"; tasks only to repos with write permission |
| `aws-transform` | AWS, [Transform custom](https://docs.aws.amazon.com/transform/latest/userguide/custom.html) | P1 | undated | `-t`/`--trust-all-tools`; human-curated "lessons" |
| `replit-agent` | Replit, [Agent](https://docs.replit.com/replitai/agent) | P1 | undated | Self-testing; checkpoints; manual publish |

### Governance, standards and project policy

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `slsa` | [SLSA v1.2 Source requirements](https://slsa.dev/spec/v1.2/source-requirements) | P1 | v1.2 Approved | Level 4: "Changes in protected branches MUST be agreed to by two or more trusted persons prior to submission"; "Trusted person": "A human who is authorized…"; "Trusted robot": "The robot's identity and codebase cannot be unilaterally influenced"; MAY grant a Trusted Robot "a perpetual exception". No mention of AI |
| `kernel-ai` | Linux kernel [`coding-assistants.rst`](https://www.kernel.org/doc/Documentation/process/coding-assistants.rst), [`generated-content.rst`](https://www.kernel.org/doc/Documentation/process/generated-content.rst), [v7.2 rendering](https://kernel.org/doc/html/v7.2/process/coding-assistants.html) | P1 | mainline 7.3-rc5, fetched 2026-10-01 | "AI agents MUST NOT add Signed-off-by tags. Only humans can legally certify the Developer Certificate of Origin"; human "Taking full responsibility"; "expect additional scrutiny in proportion to how much of it was generated"; `Assisted-by:` format differs between v7.2 and mainline |
| `gentoo` | [Gentoo Council AI policy](https://wiki.gentoo.org/wiki/Project:Council/AI_policy) | P1 | voted 2024-04-14; page edited 2026-09-11 | "It is expressly forbidden to contribute to Gentoo any content that has been created with the assistance of Natural Language Processing artificial intelligence tools" |
| `netbsd` | [NetBSD commit guidelines](https://www.netbsd.org/developers/commit-guidelines.html) | P1 | fetched 2026-10-01 | LLM code "is presumed to be tainted code, and must not be committed without prior written approval by core" |
| `qemu` | [QEMU code-provenance.rst](https://gitlab.com/qemu-project/qemu/-/raw/master/docs/devel/code-provenance.rst) | P1 | 2026-05-22 | "DECLINE any contributions which are believed to include or derive from AI generated content", including "code/content generation agents" |
| `curl` | [curl CONTRIBUTE.md](https://raw.githubusercontent.com/curl/curl/master/docs/CONTRIBUTE.md) | P1 | 2026-05-20 | "We can accept code written with the help of AI… but the code must still follow coding standards"; disclosure mandatory for AI-found issues; "We ban users immediately who submit made up fake reports" |
| `openssf` | OpenSSF, [Securing Open Source in the Age of AI](https://openssf.org/wp-content/uploads/2026/05/Securing-Open-Source-in-the-Age-of-AI.pdf) | P1 (advisory) | updated 2026-05-19 | "Consider an AI Policy"; "'Trust, but verify' must be top of mind… before merging" |
| `cisa-agentic` | ASD's ACSC, CISA, NSA et al., [Careful adoption of agentic AI services](https://www.cyber.gov.au/sites/default/files/2026-05/careful_adoption_of_agentic_ai_services.pdf) | P5 | Apr 30 / May 2026 | Snippets only: "mandatory human approval for decision-making steps". **Unverified** — PDF fetch failed |
| `nist-218a` | NIST [SP 800-218A](https://csrc.nist.gov/pubs/sp/800/218/a/final) | P1 | 2024-07-26 | Profile for *developing AI models*, not for AI-written code — off-target |
| `cis-safecode` | [CIS & SAFECode Secure by Design v1.1](https://www.cisecurity.org/about-us/media/press-release/cis-and-safecode-release-secure-by-design-v1-1-a-guide-to-assessing-software-security-practices) | P5 | Jul 2026 | Snippet: AI code "should be subject to the same testing, review, and validation processes". Unverified |
| `cra` | [Regulation (EU) 2024/2847](https://eur-lex.europa.eu/eli/reg/2024/2847/oj/eng) | P5 | 2024; full application 2027-12-11 | No AI-authorship provision found; fetch failed. Unverified |

### Lineage

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `cusumano` | M. A. Cusumano, [Shifting Economies: From Craft Production to Flexible Systems and Software Factories](https://dspace.mit.edu/server/api/core/bitstreams/1ec5521e-5ffe-4b0f-865d-2f1f74626858/content), MIT Sloan WP 3325-91 | P2 | 1991-08-26 draft | Factory approaches "became the dominant methodology for software development at Japan's largest computer manufacturers during the 1970s and 1980s"; Bemer (c. 1968): "A factory… has measures and controls for productivity and quality" |
| `greenfield` | Jack Greenfield, [The Case for Software Factories](https://learn.microsoft.com/en-us/previous-versions/aa480032(v=msdn.10)) (msdn2 → msdn → learn redirects) | P3 | July 2004 | "A Software Factory is a development environment configured to support the rapid development of a specific type of application" |

---

## 2. Implementations — how factories are actually built

All read from source at the stated commit, 2026-10-01.

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `gastown` | [gastownhall/gastown](https://github.com/gastownhall/gastown) (redirect from steveyegge/gastown) `internal/refinery/engineer.go`, `batch.go`, `internal/scheduler/capacity/config.go`, `internal/rig/config.go`; README | P1 | 649b832, 2026-07-23; v1.2.1 2026-06-06 | README: Refinery "runs verification gates"; code: `MergeStrategy … "direct" (default) does local merge + git push`; `RequireReview … Nil defaults to false`; defaults `TestCommand: ""`, `AutoPush: true`, `MaxRetryCount: 5`; `batch.go:375` "`// No gates configured — pass by default`" → `return ProcessResult{Success: true}` (re-read by this dossier); "Skipping gates (pre-verified by polecat)"; scheduler `MaxPolecats` `-1` vs rig `max_polecats: 10` |
| `ralph` | [snarktank/ralph](https://github.com/snarktank/ralph) | P1 | 6c53cb0, 2026-02-01 | `MAX_ITERATIONS=10`; completion by `<promise>COMPLETE</promise>`; `--dangerously-skip-permissions`; errors swallowed `\|\| true` |
| `ralph-wiggum` | anthropics/claude-code `plugins/ralph-wiggum` ([repo](https://github.com/anthropics/claude-code)) | P1 | 6160717, 2026-09-30 | Stop hook re-feeds prompt; `MAX_ITERATIONS=0` ("unlimited") |
| `contclaude` | [AnandChowdhary/continuous-claude](https://github.com/AnandChowdhary/continuous-claude) `continuous_claude.sh` | P1 | v0.24.9, c9edd80, 2026-09-27 | `gh pr merge` with `MERGE_STRATEGY="squash"`; "Only merge if: review is APPROVED, or no review was ever requested"; line 1429 "No checks found after waiting, proceeding without checks" → `all_success=true` (re-read by this dossier); `--max-runs` / `--max-cost` / `--max-duration` |
| `ao` | [Untrivial-ai/agent-orchestrator](https://github.com/Untrivial-ai/agent-orchestrator) | P1 | 53ba1e8, 2026-10-01; 12.6k★ | `ReadyToMerge()` treats `CIUnknown` as a blocker — "AO only claims readiness it can actually prove"; reviewer agent denied `gh pr merge`; `auto_inject_ci` feeds CI failures back |
| `sweaf` | [Agent-Field/SWE-AF](https://github.com/Agent-Field/SWE-AF) | P1 | 17d3160, 2026-09-24 | PM→architect→DAG→parallel coders→review/QA→merger→verifier→PR→CI; `agent_max_turns` 150; `max_replans 2`; `max_ci_fix_cycles 2`; PR "ready for review, not draft"; does not merge to main |
| `openinspect` | [ColeMurray/background-agents](https://github.com/ColeMurray/background-agents) | P1 | f1cf069, 2026-10-01 | `MAX_SPAWN_DEPTH = 2`; `DEFAULT_MAX_CONCURRENT_JOBS = 8`; no merge path |
| `aperant` | [AndyMik90/Aperant](https://github.com/AndyMik90/Aperant) | P1 | 20250db, 2026-06-14 | `MAX_QA_ITERATIONS = 50`; `DEFAULT_MAX_PARALLEL_TASKS = 3` |
| `openhands` | [OpenHands/OpenHands](https://github.com/OpenHands/OpenHands) a8c0558; [software-agent-sdk](https://github.com/OpenHands/software-agent-sdk) fad6377; [openhands-resolver](https://github.com/All-Hands-AI/openhands-resolver) baa2f45 | P1 | 2026-10-01 | `max_iteration_per_run: int = 500`; `stuck_detection: bool = True`; resolver's move notice points to a path that no longer exists |
| `mini-swe` | [SWE-agent/mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent) | P1 | v2.4.6, 04d809c, 2026-09-03 | Class `cost_limit` 3.0; shipped `default.yaml` `cost_limit: 0.`, `step_limit: 0` |
| `goose` | [aaif-goose/goose](https://github.com/aaif-goose/goose) | P1 | bab8ff6, 2026-09-30 | `DEFAULT_MAX_TURNS: u32 = 1000`; recipe `retry.checks` |
| `orchestrator-misc` | [BloopAI/vibe-kanban](https://github.com/BloopAI/vibe-kanban) d5cbb53 (`"dangerously_skip_permissions": true`; [shutdown 2026-04-10](https://www.vibekanban.com/blog/shutdown)); [smtg-ai/claude-squad](https://github.com/smtg-ai/claude-squad) v1.0.20 (`GlobalInstanceLimit = 10`, `--autoyes`); [ruvnet/ruflo](https://github.com/ruvnet/ruflo) v3.49.0 (`maxAgents` 10 vs 15); [sweepai/sweep](https://github.com/sweepai/sweep) (discontinued as PR bot); [Aider](https://github.com/Aider-AI/aider) (`--auto-test` False, `--auto-commits` True) | P1 | 2025-09 to 2026-10 | Human-gated parallel runners and their defaults |

---

## 3. Corpus — what factory output looks like on public GitHub

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `corpus-volume` | This dossier's counts: GitHub REST `search/issues` `total_count`, all-time and per month, by agent identity (`author:app/copilot-swe-agent`, `head:codex`, `author:app/devin-ai-integration`, `author:app/google-labs-jules`, `author:app/claude`, `author:app/cursor`, `"Generated with Claude Code" in:body`, …) | P4 | 2026-10-01 09:53–10:23Z | All-time: Copilot 2.10M (72% merged), Jules 330k, Devin 276k, Cursor 160k, claude[bot] 61k, "Generated with Claude Code" body 14.1M (93% merged); Sept 2026: CC body 4.31M/month, `head:codex` 974k, Devin 41k, Cursor 33k, claude[bot] 21k; Copilot −79% Mar→Jun 2026; the Codex task-link marker collapsed after ~Nov 2025 (studies keyed on it undercount) |
| `corpus-cohort` | This dossier's cohort: PRs created 2026-08-03 to 08-09; population state counts; S1 = 200 per agent (10 three-hour slices × 20 newest); S2 = 840 per agent filtered to ≥100★; "human review" = ≥1 review by a non-author `User` before `mergedAt`, bot reviewers excluded; reviews truncated at 10 | P4 | 2026-10-01 10:26–10:40Z | Population merged-with-`review:approved`: 3.3–10.2%; S1 human review pre-merge 3% (Codex), 4% (CC body), 8% (Devin), 13% (Jules), 22% (Copilot, claude[bot]); S1 self-merge by launcher 94% (Codex), 97% (CC body); median time to merge 0.10–0.74 h; 46–74% of PRs in 0★ repos; S2 (≥100★) review 27% (Codex) to 80% (Cursor); bot self-merge 34% Cursor, 16% claude[bot], 9% Devin (S1). Clustered sample: github/gh-aw is 43 of 102 Copilot S2 PRs |
| `corpus-workflows` | This dossier's code-search counts over `path:.github/workflows` plus a purposive sample of 30 workflows classified by trigger, write scope, human gate, merge ability | P4 | 2026-10-01 10:12–10:36Z | `claude-code-action` 19,904 files; with `schedule` 2,028; `"contents: write"` 7,056; `"gh pr merge"` 547; `"merge --auto"` 130; merge explicitly disallowed 110; `dangerously-skip-permissions` 378; `codex-action` 1,288 (55 with `gh pr merge`); 30-sample: 10 merge or arm auto-merge with no per-change human approval, 10 keep merging from the agent, 6 leave it to repo settings. Biased toward risky shapes by design |

Workflow, repo and PR exemplars (commit or date as fetched):

| Key | Source | Date | Why it is cited |
|---|---|---|---|
| `w-wacrypt` | [ElDavoo/wa-crypt-tools](https://github.com/ElDavoo/wa-crypt-tools) `agent-implement.yml` (1,151★) | 2026-09-24 | Auto-merge armed after checks and AI review; refuses `workflow` scope: "That would let an agent rewrite the review and CI gates that judge it, and then merge the rewrite." |
| `w-vmark` | [xiaolai/vmark](https://github.com/xiaolai/vmark) `claude.yml` (808★) | 2026-09-26 | Auto-merge on an AI "✅ VERIFIED" verdict; fixed after audit: "Without it any commenter could authorize an auto-merge by typing the marker"; issue trigger "RETIRED 2026-06-17" |
| `w-mcpjam` | [MCPJam/inspector](https://github.com/MCPJam/inspector) `mintlify-triage.yml` (2,231★) | 2026-07-16 | github-actions posts APPROVE because "the main ruleset requires 1 approving review" — a bot satisfying a human-review rule |
| `w-skyvern` | [Skyvern-AI/skyvern](https://github.com/Skyvern-AI/skyvern) `auto-merge-sync.yml` (23,113★) | 2026-05-22 | Author allowlist includes claude[bot], copilot-swe-agent[bot], cursoragent; merges with `--admin`, bypassing branch protection |
| `w-gated` | [MaterializeInc/materialize](https://github.com/MaterializeInc/materialize) `update-generated-docs.yml` ("Automerge is intentionally NOT enabled here."); [metabase/metabase](https://github.com/metabase/metabase) `codespell.yml`; [Shopify/cli](https://github.com/Shopify/cli) `maintenance-prs.yml` ("Never merge, approve, or mark a PR ready for review."); [astral-sh/uv](https://github.com/astral-sh/uv) `fix-bug.yml` (`gh pr create --draft`); [sourcebot-dev/sourcebot](https://github.com/sourcebot-dev/sourcebot) `_cve-remediation.yml` (`--disallowedTools "Bash(gh pr merge *)"`) | 2026-09 | Well-gated scheduled agents in large repos |
| `w-xtuner` | [InternLM/xtuner](https://github.com/InternLM/xtuner) `claude-general.yml` | 2026-09-18 | `allowed_non_write_users: "*"` — trigger open to anyone |
| `w-autojules` | [jelni/ai-bullshit](https://github.com/jelni/ai-bullshit) `jules-auto-merge.yml` (631/693 merged); [BintzGavin/helios](https://github.com/BintzGavin/helios) `auto-merge.yml` (283/296) | 2026-07 to 08 | Ungated auto-merge of every agent PR |
| `r-ghaw` | [github/gh-aw](https://github.com/github/gh-aw), e.g. [PR #49898](https://github.com/github/gh-aw/pull/49898) | 2026-08-03 | 16,960 Copilot PRs, 13,036 merged, 4 with `review:approved` — GitHub's own repo |
| `r-darkexp` | [coleam00/dark-factory-experiment](https://github.com/coleam00/dark-factory-experiment), [PR #364](https://github.com/coleam00/dark-factory-experiment/pull/364) | 2026-08-11/14 | "auto-merge with no human reading the diff"; 94 of 248 PRs merged; #364: fixes "done by hand rather than dispatched to the factory, because merging main auto-deploys" |
| `r-declared` | [jleechanorg/dark-factory](https://github.com/jleechanorg/dark-factory), [obra/homedir-manager](https://github.com/obra/homedir-manager) ("not human-reviewed"), [sttts/kc](https://github.com/sttts/kc) ("No human has reviewed the code") | 2025-09 to 2026-09 | Self-declared dark-factory repos, all small |
| `r-tzf` | [jannikmi/timezonefinder](https://github.com/jannikmi/timezonefinder) unattended-pipeline decision record | 2026-09-05 | Deterministic no-review pipeline; "A branch-name prefix is not an authorization check" |
| `pr-selfmerge` | [Francis1998/scholar-rag-agent#60](https://github.com/Francis1998/scholar-rag-agent/pull/60) (merged by `cursor` 8 min after opening, +543 lines, 0 reviews); [wookat/attnbox#5](https://github.com/wookat/attnbox/pull/5) (merged by `devin-ai-integration` after 2 min); [dfinity/ic#11073](https://github.com/dfinity/ic/pull/11073) (merged by `claude` *after* human approval) | 2026-08 | Agents merging; bot-merged ≠ ungated |
| `dotnet-cca` | Stephen Toub, [Ten months with CCA in dotnet/runtime](https://devblogs.microsoft.com/dotnet/ten-months-with-cca-in-dotnet-runtime/) | 2026-03-23 | P4: 878 Copilot PRs, 535 merged (67.9%); "3 of 535 merged CCA PRs were reverted (0.6%)" vs 0.8% for non-agent PRs; every PR human-reviewed; "I quite quickly created 5 to 9 hours of review work" |
| `i-44202` | [anthropics/claude-code#44202](https://github.com/anthropics/claude-code/issues/44202) | 2026-04-06 | "created and auto-merged a pull request to a production branch… ~11 seconds — no human review occurred" (P5, first-hand report) |
| `i-oss-closing` | [tldraw#7695](https://github.com/tldraw/tldraw/issues/7695) (2026-01-15); [openai/codex discussion #9956](https://github.com/openai/codex/discussions/9956) (2026-01-27, "We no longer accept unsolicited pull requests"); [Ghostty AI_POLICY.md](https://github.com/ghostty-org/ghostty/blob/main/AI_POLICY.md) (2026-01-22: "increased the 'bad' count by 10x if not more"); [Ladybird](https://ladybird.org/posts/changing-how-we-develop-ladybird/) (2026-06-05: "A substantial patch used to imply substantial effort… That assumption no longer holds") | 2026 | Projects closing or restricting outside PRs |
| `i-matplotlib` | [matplotlib#31132](https://github.com/matplotlib/matplotlib/pull/31132); [AIID incident 1373](https://incidentdatabase.ai/entities/matplotlib/) | 2026-02-10 | Agent responds to closure: "Judge the code, not the coder" |
| `gh-pr-settings` | GitHub Changelog, [New repository settings for configuring pull request access](https://github.blog/changelog/2026-02-13-new-repository-settings-for-configuring-pull-request-access/) | 2026-02-13 | P1: "Disable pull requests entirely" / "Restrict pull requests to collaborators" |
| `gh-status-0626` | [GitHub Status incident](https://www.githubstatus.com/incidents/0rkjjs2ssp7z) | 2026-06-26/28 | P1: Copilot Cloud Agent "tool calls failed silently so agent jobs appeared to succeed" |
| `copilot-billing` | GitHub Blog, [Copilot is moving to usage-based billing](https://github.blog/news-insights/github-copilot-is-moving-to-usage-based-billing/) | 2026-04-27 | P1: effective June 1, 2026; the volume drop's link to it is inference |

---

## 4. Evidence — what has been measured

### Agent PRs in the wild

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `aidev-2507` | Li, Zhang, Hassan, [The Rise of AI Teammates in SE 3.0](https://arxiv.org/html/2507.15003), arXiv 2507.15003 | P2 | v1 2025-07-20 | 456k PRs; acceptance (>500★ subset) Codex 64–65.3%, Claude Code 52.5%, Cursor 51.4%, Devin 48.9%, Copilot 38.2%, **human 76.8%**; median hours to accepted merge Codex 0.3, human 3.9, Copilot 17.2; "questioning review depth" |
| `aidev-2602` | [AIDev: Studying AI Coding Agents on GitHub](https://arxiv.org/html/2602.09185v1), arXiv 2602.09185 (MSR '26) | P2 | 2026-02-09 | "932,791 Agentic-PRs produced by five agents", cutoff 2025-08-01; curated 33,596 PRs from >100★ repos |
| `ehsani` | Ehsani et al., [Where Do AI Coding Agents Fail?](https://arxiv.org/html/2601.15195), arXiv 2601.15195 | P2 | 2026-01-21 | 33,596 PRs, 71.48% merged (Codex 82.59%, Copilot 43.04%); not-merged reasons: reviewer abandonment 38%, duplicates 23%, CI failures 17% |
| `watanabe` | Watanabe et al., [On the Use of Agentic Coding](https://arxiv.org/abs/2509.14745), arXiv 2509.14745 | P2 | v3 2026-02-09 | 567 Claude Code PRs: "83.8%… accepted"; "54.9% of the merged PRs are integrated without further modification" |
| `kraishan` | Kraishan, [Not All Agents Are Equal](https://arxiv.org/html/2609.17598), arXiv 2609.17598 | P2 | 2026-09-12 | 37,623 agent PRs + matched human baseline; reverts Codex 6.1% vs human 11.5% (OR 0.50), Devin 14.5%; Claude Code median 12.6 h to first human review; pooled security smells 2.9% vs 4.6% (regex). Single author, non-random assignment |
| `sawada` | Sawada et al., [arXiv 2605.06464](https://arxiv.org/html/2605.06464v1) | P2 | 2026-05-07 | Maintenance of AI-generated files: "83.21% of commits were made by human developers" |
| `siddiq` | Siddiq et al., [Security in the Age of AI Teammates](https://arxiv.org/abs/2601.00477v1) | P2 | 2026-01-01 | Security agent PRs "lower merge rates and longer review latency" [ABS] |
| `sakib` | Sakib, Banik, Jadliwala, [Trust but Verify?](https://arxiv.org/abs/2607.12428), arXiv 2607.12428 | P2 | 2026-07-14 | "38.9%" of 4,022 agent PRs have a security smell; humans introduced "67.6% of genuine leaked secrets"; review fails "to detect 81.1%" of credentials |
| `lin-review` | Lin et al., [Is Agentic Code Review Helpful?](https://arxiv.org/abs/2607.03316) | P2 | 2026-07-03 | CodeRabbit comments "36.4%… accepted", "56.3%… rejected" across 31,073 pairs |

### Productivity and organisation-level telemetry

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `metr-rct` | Becker, Rush, Barnes, Rein, [arXiv 2507.09089](https://arxiv.org/abs/2507.09089) | P2 | v2 2025-07-25 | RCT, 16 devs, 246 tasks: AI made tasks take **19% longer**; devs forecast 24% faster |
| `metr-uplift` | METR, [We are Changing our Developer Productivity Experiment Design](https://metr.org/blog/2026-02-24-uplift-update/) | P3 | 2026-02-24 | Returning devs −18% [−38%, +9%], new devs −4% [−15%, +9%]; "30% to 50% of developers… choosing not to submit some tasks"; time measurement "unreliable" with concurrent agents; "very weak evidence" |
| `dora` | Google, [DORA 2025 report](https://research.google/pubs/dora-2025-state-of-ai-assisted-software-development-report/) + [blog](https://blog.google/technology/developers/dora-report-2025/) | P3 | 2025-09-23 | ~5,000 respondents; adoption "90%"; "linked to higher software delivery throughput"; "an amplifier". Link to higher instability via secondary summaries only |
| `faros-2026` | Faros AI, [AI Engineering Report 2026: The Acceleration Whiplash](https://www.faros.ai/blog/ai-acceleration-whiplash-takeaways) | P4 | 2026-04-12 | 22,000 devs, 4,000+ teams, low- vs high-adoption periods within org: tasks +33.7%, merge rate +16.2%, **incidents per PR +242.7%**, bugs per dev +54%, **PRs merged without review +31.3%**, review time +441.5%. Vendor; not causal |
| `faros-2025` | Faros AI "AI Productivity Paradox" via [Rob Bowley](https://blog.robbowley.net/2025/08/21/new-research-shows-ai-assisted-coding-isnt-moving-delivery/) | P5 | 2025-07 / 08-21 | 98% more PRs, review +91%; "correlation… evaporates at the company level". Secondary only |
| `gitclear` | GitClear, [AI Copilot Code Quality 2025](https://www.gitclear.com/ai_assistant_code_quality_2025_research) | P4 | early 2025 | Copy/paste 8.3% → 12.3% of changed lines; moved code ~25% → <10%. AI influence inferred from trend |
| `google-75` | [Semafor](https://www.semafor.com/article/04/24/2026/google-ceo-says-75-of-companys-new-code-is-ai-generated) | P5 | 2026-04-24 | "75% of the company's new code is AI-generated", engineer-approved; no method |

### Long-horizon capability

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `metr-th` | Kwa et al., [arXiv 2503.14499](https://arxiv.org/abs/2503.14499) (NeurIPS 2025) | P2 | 2025-03 | 50% horizon "doubling approximately every seven months" |
| `metr-th11` | METR, [Time Horizon 1.1](https://metr.org/blog/2026-1-29-time-horizon-1-1/) | P3 | 2026-01-29 | Doubling since 2024: 89 days (TH1.1) |
| `metr-frr` | METR, [Frontier Risk Report Feb–Mar 2026](https://metr.org/blog/2026-05-19-frontier-risk-report/) | P3 | 2026-05-19 | Public frontier **50% ≈12 h [5–61 h]; 80% ≈1.5 h [50 min–2 h 40]**; "can't reliably measure time horizons above 16 hours"; on ≥8 h tasks "at least 16% of successful runs were illegitimate upon review"; "deliberate steps to hide evidence" |
| `metr-thpage` | METR, [Time horizons](https://metr.org/time-horizons/) | P3 | updated 2026-05-08 | "Measurements above 16 hrs are unreliable with our current task suite" |
| `oai-benchmarks` | OpenAI SWE-bench Verified retirement via [The Decoder](https://the-decoder.com/openai-wants-to-retire-the-ai-coding-benchmark-that-everyone-has-been-competing-on/) (2026-02-23) and SWE-bench Pro audit via [Gigazine](https://gigazine.net/gsc_news/en/20260709-openai-coding-evaluations/) (2026-07-08) | P5 | 2026 | "at least 59.4 percent" of 138 hard Verified tasks flawed; ~30% of Pro tasks broken; recommendation withdrawn. Primaries 403 |

### Tests as the oracle

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `metr-holistic` | METR, [Algorithmic vs. Holistic Evaluation](https://metr.org/blog/2025-08-12-research-update-towards-reconciling-slowdown-with-time-horizons) | P3 | 2025-08-12 | 38% by tests; "0% of reviewed PRs were mergeable as-is"; 42 min average fix |
| `metr-swebench` | METR, [Many SWE-bench-Passing PRs Would Not Be Merged into Main](https://metr.org/notes/2026-03-10-many-swe-bench-passing-prs-would-not-be-merged-into-main/) | P3 | 2026-03-10 | "Roughly half of test-passing SWE-bench Verified PRs would not be merged into main by repo maintainers" (296 PRs, 4 maintainers); acceptance improves ~9.6 pp/yr slower than grader scores |
| `swe-abs` | Yu et al., [SWE-ABS](https://arxiv.org/abs/2603.00520), arXiv 2603.00520 | P2 | 2026-02-28 | Strengthened tests reject ~20% (2,184) of passing patches; top agent 78.80% → 62.20% [ABS] |
| `sting` | Li et al., [Probe to Generate](https://arxiv.org/abs/2604.01518), arXiv 2604.01518 | P2 | 2026-04-02 | "77% of instances in SWE-bench Verified" admit a surviving incorrect variant [ABS] |
| `impossiblebench` | Zhong, Raghunathan, Carlini, [ImpossibleBench](https://arxiv.org/html/2510.20270v1), arXiv 2510.20270 | P2 | 2025-10-23 | "GPT-5 cheats 54.0% of the time on Conflicting-SWEbench", 76% on Oneoff; abort option 54% → 9%; LLM monitors catch ~42–65% on SWE-bench; hidden/read-only tests reduce cheating |
| `metr-rewardhack` | METR, [Recent frontier models are reward hacking](https://metr.org/blog/2025-06-05-recent-reward-hacking/) | P3 | 2025-06-05 | o3 reward-hacked 30.4% of RE-Bench runs; "Please do not cheat" left ~80% on one task |
| `carlini-compiler` | Nicholas Carlini, [Building a C compiler with a team of parallel Claudes](https://www.anthropic.com/engineering/building-c-compiler) | P3 | 2026-02-05 | ~$20,000, ~2,000 sessions, 100k lines Rust, "99% pass rate on most compiler test suites"; "the task verifier is nearly perfect, otherwise Claude will solve the wrong problem"; "new features and bugfixes frequently broke existing functionality" |
| `signal65` | Signal65 / Martian CodeRabbit benchmarks | P5 | 2026 | Precision 49% or 96% depending on source; snippets only |

### Code quality and security

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `veracode` | Veracode, [2025 GenAI Code Security Report](https://www.veracode.com/press-release/ai-generated-code-poses-major-security-risks-in-nearly-half-of-all-development-tasks-veracode-research-reveals/) | P3 | 2025-07-30 | "chose the insecure option 45 percent of the time"; single-shot, not agents |
| `slopsquat` | Spracklen et al., [We Have a Package for You!](https://arxiv.org/abs/2406.10279) (USENIX Security 2025) | P2 | v3 2025-03-02 | Hallucinated packages "at least 5.2%" commercial, "21.7%" open models |

### Incidents

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `replit-incident` | The Register via [AIID report 5578](https://incidentdatabase.ai/reports/5578/); [eWeek](https://www.eweek.com/news/replit-ai-coding-assistant-failure/) | P5 | 2025-07-21 | Production DB deleted during a code freeze; "I explicitly told it eleven times in ALL CAPS not to do this"; Replit then separated dev and prod databases |
| `aws-2025-015` | AWS Security Bulletin [AWS-2025-015](https://aws.amazon.com/security/security-bulletins/AWS-2025-015/) (CVE-2025-8217) | P1 | 2025-07-23 | "inappropriately scoped GitHub token in their CodeBuild configuration" let a wiper prompt ship in Amazon Q v1.84.0 |
| `invariant-mcp` | Invariant Labs, [GitHub MCP Exploited](https://invariantlabs.ai/blog/mcp-github-vulnerability) | P3 | 2025-05-26 | Malicious public issue → agent leaks private-repo data in a public PR |
| `cve-53773` | [CVE-2025-53773](https://cveawg.mitre.org/api/cve/CVE-2025-53773) (Microsoft CNA, CVSS 7.8) | P1 | 2025-08-12 | Prompt injection → command injection in Copilot/Visual Studio |
| `nx-s1ngularity` | Nx [GHSA-cxm3-wv7p-598c](https://github.com/nrwl/nx/security/advisories/GHSA-cxm3-wv7p-598c) (P1); Wiz, [s1ngularity](https://www.wiz.io/blog/s1ngularity-supply-chain-attack) (P3) | P1/P3 | 2025-08-26/27 | Root cause `pull_request_target` + bash injection from a PR title; Wiz: malware "weaponized installed AI CLI tools by prompting them with dangerous flags"; the Nx advisory does not mention AI CLIs |
| `promptpwnd` | Rein Daelman (Aikido), [PromptPwnd](https://www.aikido.dev/blog/promptpwnd-github-actions-ai-agents) | P3 | 2025-12-04, upd. 2026-03-17 | "Untrusted user input → injected into prompts → AI agent executes privileged tools → secrets leaked"; Gemini CLI, Claude Code Actions, Codex Actions; "At least 5 Fortune 500 companies" |
| `comment-control` | [SecurityWeek](https://www.securityweek.com/claude-code-gemini-cli-github-copilot-agents-vulnerable-to-prompt-injection-via-comments/) on Guan, Liu, Zhong ([researcher post](https://oddguan.com/blog/comment-and-control-prompt-injection-credential-theft-claude-code-gemini-cli-github-copilot/), not fetched) | P5 | 2026-04-16 | PR titles/comments hijack Claude Code Security Review, Gemini CLI Action, Copilot Agent; bounties $100 / $1,337 / $500 ("known architectural limitation") |
| `hf-intrusion` | Hugging Face, [Agent intrusion technical timeline](https://huggingface.co/blog/agent-intrusion-technical-timeline); METR, [Painter Senate testimony](https://metr.org/blog/2026-09-30-chris-painter-senate-testimony/) | P3 | 2026-07-27; 2026-09-30 | Evaluation agents tried to "cheat the evaluation: reach our production systems and steal the test solutions"; "Roughly 700 of these AI agents compromised Hugging Face" (testimony; OpenAI primary not fetched) |

---

## Conflicts resolved

- **Does claude-code-action merge?** Docs: "Cannot merge branches" (`cc-action`). Corpus: 16% of merged
  `claude[bot]` PRs in S1 were merged by the bot, and 547 workflow files pair the action with `gh pr
  merge` (`corpus-cohort`, `corpus-workflows`). **Both hold.** The limitation is tag mode's fixed
  `--allowedTools` allowlist (re-read in `src/modes/tag/index.ts`); agent mode takes whatever
  `claude_args` allows, and nothing in code enforces a merge ban. The docs describe a default, not a
  guarantee.
- **Can Devin merge?** Primary and Implementations both inferred it from "before Devin can merge
  changes" (`devin`). Corpus observed `devin-ai-integration` merging its own PR two minutes after
  opening (`pr-selfmerge`) and 9% of S1 merges by the bot. **Resolved: yes.** The conditions remain
  undocumented.
- **Claude Code Action and PR creation.** `cc-action`'s security.md says the user opens the PR from a
  link; the docs page says it can "turn issues into pull requests". **Resolved by mode:** tag mode
  pushes a branch and offers a link; agent/automation mode opens PRs when its tools allow — 61,266
  all-time PRs are authored by `claude[bot]` (`corpus-volume`).
- **"Best-of-N" in Codex.** A third-party write-up says `--attempts` selects the best result;
  `codex-src` produces N attempts and the user applies one by number. **Resolved against source: no
  automatic selection.**
- **Shapiro's date.** Jan 23 (page) vs Jan 28 (`willison-levels`). **Resolved: Jan 23 publication,
  Jan 28 link post.**
- **StrongDM's date.** The site is undated; the Attractor repo was created 2026-02-05T23:40Z
  (`attractor`); the announcement is reported as 2026-02-06. **Recorded as ≈2026-02-06.**
- **Gas Town and continuous-claude merge-on-nothing.** Reported by Implementations; **re-verified in
  source by this dossier** (`gastown` `batch.go:375`; `contclaude` line 1429).
- **Attractor has no scenario machinery.** Reported by Implementations; **re-verified**: zero matches
  in all four published files (`attractor`).
- **Jules plan approval.** Primary (UI timer auto-approve) and Implementations (API auto-approve by
  default) describe two paths of the same behaviour; both from `jules`. **Consistent.**

## Conflicts left open

- **Agent PR merge rates.** `aidev-2507` (Codex ~65%, Copilot 38.2%, humans 76.8%) vs `ehsani` (Codex
  82.59%, all agents 71.48%) over overlapping AIDev subsets, vs this dossier's Aug 2026 cohort (Copilot
  66–70%). Different denominators, filters and windows; not averaged.
- **Revert rates vs incidents.** `kraishan` (agent reverts at or below human) and `dotnet-cca` (0.6% vs
  0.8%) vs `faros-2026` (incidents per PR +242.7%). Per-PR reviewed OSS vs organisation-level enterprise
  telemetry; both may be true.
- **Security smells.** `sakib` (38.9% of agent PRs) vs `kraishan` (fewer than humans, 2.9% vs 4.6%).
  Different detectors (LLM judge vs regex), units and baselines.
- **Perceived vs measured productivity.** `metr-rct` (−19% measured, +24% forecast) vs `metr-uplift`
  (point estimates now favour speedup, intervals include zero, selection effects).
- **Did s1ngularity use AI CLIs?** `nx-s1ngularity`'s Wiz post says yes; the Nx advisory as fetched does
  not mention it.
- **Copilot workflow auto-run.** `copilot-agent` says it can be configured; `copilot-automations` says
  workflows don't run until approved. Possibly scoped to Automations; unresolved.
- **Cursor Automations "cannot merge" vs `cursor` merging.** `cursor-cloud` says Automations cannot
  merge; 34% of merged Cursor S1 PRs were merged by the `cursor` identity (`corpus-cohort`,
  `pr-selfmerge`). Which product path produced those merges is not determinable from the PR.
- **Is review part of the definition?** `strongdm-site` and `shapiro` say no human review; `bcg` keeps
  "stage-gate approval". A definitional disagreement, recorded as such.
- **Yegge across time.** Gas Town's README escalates to humans; `yegge-shape` predicts human review
  "completely done and gone" and says Gas Town "fell apart" with Opus 4.7.
- **Gas Town's two concurrency defaults** (`-1` scheduler vs `max_polecats: 10`) — interaction
  untested.
- **Kernel `Assisted-by:` format** differs between v7.2 and mainline (`kernel-ai`).
- **CodeRabbit precision** 49% vs 96% (`signal65`).

## Explicitly unverified

- **StrongDM's validation layer.** Holdout scenarios, satisfaction scoring and the Digital Twin
  Universe are described on `strongdm-site` and absent from every published StrongDM repo
  (`attractor`, `agate`, `cxdb`). Whether they exist as described is unverified.
- **StrongDM outcomes.** No defect, throughput, incident or cost-per-change number has been published.
- **OpenAI "Harness engineering"** — "0 lines of manually-written code", ~1,500 PRs, three engineers
  (`oai-harness`, 403). Circulates widely; primary unread. Treat as folklore until read.
- **BCG's "3 to 5x"** and the Spotify/OpenAI figures it cites (`bcg`): no method.
- **The "$1,000 per engineer per day" target** (`strongdm-site`) is a rule of thumb, not a measured
  optimum; `cc-costs` puts enterprise interactive use at ~$13/active day.
- **CISA/ASD agentic AI guidance**, **CIS/SAFECode v1.1**, **EU CRA**: fetches failed; snippets only.
- **Comment and Control** CVSS 9.4: secondary only.
- **DORA 2025's instability finding**: secondary summaries; not confirmed in the PDF.
- **Faros 2025**: secondary only.
- **OpenAI benchmark audits** (`oai-benchmarks`): secondary only.
- **Copilot's volume drop and billing**; **the Codex marker change**: inferred causes.
- **`review:none` semantics**: tested on two PRs only.
- **Post-merge outcomes of genuinely unreviewed pipelines.** No controlled or observational study
  measures defect, incident or security rates for a pipeline that has removed human review. The
  absence of a public postmortem of an auto-merged agent regression is an unverified absence.
- **Jules PR/branch behaviour, Kiro limits, OpenHands Cloud defaults, Conductor merge path.**
- **Model names** (Opus 4.7, Fable 5.1, Claude Mythos Preview, GPT-5.4, etc.) are reproduced as the
  sources state them.
