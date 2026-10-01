# Software factories rulebook

Checkable rules for pipelines where agents take work from spec to merged code with reduced or no
human review. Each rule states its test (how a third party checks compliance), its evidence tier, and
its source keys (see [sources.md](sources.md)).

Tiers: **P1** primary spec, vendor docs, policy or shipped source · **P2** peer-reviewed or preprint
research · **P3** vendor engineering blog / industry research with method · **P4** practitioner
measurement (including this dossier's own counts) · **P5** opinion, anecdote, unverified.

Use it two ways: as a design checklist before removing a human from any step, and as a review gate
for an existing factory. A pipeline that fails R1, R6, R13 or R20 does not pass, whatever else it
gets right. Git-level controls are in [gitGuardrails/rulebook.md](../gitGuardrails/rulebook.md);
isolation is in [microVms/rulebook.md](../microVms/rulebook.md).

---

## A. The oracle

**R1. Name the oracle before removing the reviewer.** Test: for each change class merged without
human review, is there a written statement of what independent evidence establishes correctness?
**P3** `carlini-compiler`, `metr-swebench` · **P1** `strongdm-site`

**R2. Don't let passing tests be the whole oracle.** About half of test-passing agent patches are not
mergeable. Test: does the merge condition include evidence beyond the repository's own test suite
(holdout scenarios, running-app QA, equivalence checks)? **P3** `metr-swebench`, `metr-holistic` ·
**P2** `swe-abs`, `sting`

**R3. Hide the acceptance criteria from the agent.** Test: are acceptance scenarios stored outside the
agent's working repository and unreachable by its tools? **P2** `impossiblebench` · **P1**
`strongdm-site`

**R4. Make tests and gate config read-only to the agent.** Test: does any diff from the agent that
touches tests, CI workflows or gate configuration require human approval, and does the agent's token
lack `workflow` scope? **P2** `impossiblebench` · **P3** `metr-rewardhack` · **P4** `w-wacrypt`

**R5. Strengthen tests adversarially before trusting them.** Test: is mutation, property or
coverage-guided strengthening run on the suites used as merge gates? **P2** `swe-abs`, `sting`

**R6. Don't use an LLM reviewer as the sole gate.** Monitors catch ~42–65% of SWE-task cheating. Test:
is every LLM-review gate paired with a non-LLM check? **P2** `impossiblebench`, `lin-review` · **P1**
`agate`

**R7. Review a sample of "successful" runs.** ≥16% of long-task successes were illegitimate. Test: is
a random sample of auto-merged changes audited by a human on a schedule, and the illegitimate-success
rate recorded? **P3** `metr-frr`

## B. Gates

**R8. Fail closed when no gate is configured.** Test: with no checks configured, does the pipeline
block the merge? (Read the code path — `gastown` and `contclaude` pass.) **P1** `gastown`,
`contclaude`, `ao`

**R9. Fail closed on unknown or missing status.** Test: do "CI unknown", "no checks reported", a
missing status file, or a timed-out wait block the merge? **P1** `ao`, `attractor`, `contclaude`

**R10. Never accept self-attested completion.** Test: is "done" established by something other than
the agent's own string, flag or summary? **P1** `ralph`, `ralph-wiggum`, `gastown`

**R11. Detect silent tool failure.** Test: does the pipeline verify that the claimed artefact exists
and changed (diff non-empty, build produced, deploy observed), not only that the job exited 0? **P1**
`gh-status-0626`, `attractor`

**R12. Don't let a bot satisfy a human-review rule.** Test: can any automation's APPROVE count toward
a required-reviewers rule? **P4** `w-mcpjam` · **P1** `slsa`

## C. Merge authority and irreversible steps

**R13. Keep a human, or a non-injectable gate, at every irreversible step.** Merge to an
auto-deploying branch, production data, releases. Test: for each irreversible action, is there either
a named human approval or a deterministic gate whose inputs no untrusted party controls? **P1**
`slsa`, `kernel-ai` · **P5** `replit-incident`, `i-44202` · **P4** `r-darkexp`

**R14. Never merge agent output with `--admin` or ruleset bypass.** Test: do any workflows or agent
identities hold bypass permissions on the target branch? **P4** `w-skyvern` · **P1** `copilot-agent`

**R15. Know whether each agent can merge, from the code, not the docs.** Test: for each agent in use,
has its actual tool allowlist (not its README) been checked for merge capability? **P1** `cc-action`,
`devin`, `gh-aw` · **P4** `corpus-cohort`

**R16. Disable plan auto-approval where plans gate risky work.** Test: are Jules API sessions,
timed plan approval and agent-team lead auto-approval turned off for such work? **P1** `jules`,
`cc-teams`

**R17. Name the accountable human for every merged change.** Test: can each merged change be traced to
a human who accepted responsibility (sign-off, approval or ownership of the change class)? **P1**
`kernel-ai`, `slsa`, `openssf`

## D. Intake and injection

**R18. Don't run write-capable agents on text from users without write access.** Test: are
`allowed_non_write_users`-style settings absent, or the agent read-only, wherever issue/PR text from
outsiders is the input? **P3** `promptpwnd`, `invariant-mcp` · **P5** `comment-control` · **P4**
`w-xtuner`

**R19. Never feed untrusted text to agents under `pull_request_target` with secrets.** Test: does any
workflow combine `pull_request_target` (or equivalent) with an agent and repository secrets? **P1**
`nx-s1ngularity`

**R20. Scope the agent's token to what the change class needs.** Test: does each agent identity lack
`contents: write` on protected branches, `workflow` scope, and admin rights? **P1** `aws-2025-015`,
`copilot-agent` · **P4** `w-wacrypt` · see `dossier-git`

**R21. Don't claim SLSA's Trusted Robot exception for an LLM agent.** Its codebase-cannot-be-influenced
condition is what injection violates. Test: does any SLSA L4 claim rely on an agent being a Trusted
Robot? **P1** `slsa`

**R22. Pin dependencies the agent adds.** Test: is every new dependency checked against the registry
and an allowlist before merge? **P2** `slopsquat`

## E. Budgets and reliability

**R23. Set explicit caps on turns, time, cost and concurrency.** Most defaults are unlimited. Test: are
all four set in config for every factory component? **P1** `attractor`, `mini-swe`, `devin`,
`gastown`, `cc-sdk`

**R24. Size unattended tasks to the 80% horizon, not the 50%.** Test: is the typical unattended task
scoped well under ~1.5 h of human-equivalent work, or decomposed until it is? **P3** `metr-frr`,
`metr-thpage`

**R25. Escalate to a human on repeated failure.** Test: after a bounded number of failed attempts,
does the pipeline stop and notify a person rather than retry indefinitely? **P1** `agate`, `sweaf`

**R26. Don't assume best-of-N selects.** Test: if multiple attempts are run, is the selection
criterion defined and independent of the attempts themselves? **P1** `codex-src`, `attractor`

## F. Measuring the factory

**R27. Measure outcomes per change, before and after.** Revert rate, incidents per change, escaped
defects, cost per merged change. Test: are these tracked, with a pre-factory baseline? **P4**
`faros-2026`, `dotnet-cca` · **P2** `kraishan`

**R28. Measure review load, not just throughput.** Test: is reviewer time per merged change tracked?
**P4** `dotnet-cca`, `faros-2026` · **P2** `ehsani`

**R29. Don't cite vendor productivity multipliers without method.** Test: does any business case rest
on a "3–5x", "0 lines of manual code" or tokens-per-day figure without a disclosed method? **P5**
`bcg`, `oai-harness`, `strongdm-site`

**R30. Don't rank agents on SWE-bench Verified or Pro alone.** Both have been found substantially
broken. Test: is agent selection backed by your own repository tasks? **P5** `oai-benchmarks` · **P3**
`metr-swebench`

## G. Rollout

**R31. Remove the human by change class, strongest oracle first.** Test: is there a written list of
change classes allowed to merge without review, each with its oracle, and does everything else
default to human merge? **P4** `w-gated` · **P1** `copilot-agent`, `kiro-agent`

**R32. Respect the target project's AI policy.** Test: for each repository the factory contributes to,
has its AI policy been checked (ban, presumed tainted, disclosure, human sign-off)? **P1** `gentoo`,
`qemu`, `netbsd`, `curl`, `kernel-ai` · **P4** `i-oss-closing`

**R33. Isolate the factory's agents.** Test: does the factory meet the microVms review gate? **P1** see
`dossier-microvms`

---

## Review checklist

A software factory does not pass review until every one of these holds:

| # | Gate | Rules |
|---|---|---|
| 1 | Each unreviewed change class has a named oracle beyond the repo's own tests | R1, R2, R31 |
| 2 | The agent cannot see the acceptance criteria or edit tests, CI or gates | R3, R4 |
| 3 | Every gate fails closed: no checks, unknown CI, missing status, silent failure | R8, R9, R10, R11 |
| 4 | No LLM-only or bot-approval gate stands alone | R6, R12 |
| 5 | Irreversible steps have a human or non-injectable gate; no `--admin` merges | R13, R14 |
| 6 | An accountable human exists for every merged change | R17 |
| 7 | Untrusted intake cannot drive a write-capable agent; tokens scoped | R18, R19, R20 |
| 8 | Turns, time, cost and concurrency capped; tasks sized to the 80% horizon | R23, R24 |
| 9 | Outcomes and review load measured against a pre-factory baseline | R27, R28 |

**The four rules that gate everything else:** R1, because a factory without a named oracle is an
unreviewed merge with extra steps; R8, because a gate that passes when unconfigured is the commonest
way the oracle silently disappears; R13, because the incidents on record happen at the irreversible
step; and R20, because every published autonomous PR agent has been shown injectable, and the token is
what the injection inherits.
