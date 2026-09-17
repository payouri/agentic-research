# Context smart zone rulebook

Checkable rules for managing an agent's context. Each rule states the test, the evidence strength,
and the source key (see [sources.md](sources.md)).

Evidence: **[S]** peer-reviewed or controlled · **[V]** vendor documentation or shipped source ·
**[F]** field pattern across many real artifacts · **[W]** practitioner report with some measurement ·
**[O]** opinion.

Use it two ways: as a working discipline, and as a review gate — a session that fails R1, R6 or R21
is producing output you should not trust without re-checking.

---

## A. Calibration — where the zone actually is

**R1. Never treat the harness threshold as a quality boundary.** Compaction fires to prevent an API
overflow error, not to protect output quality. Test: can you state, for your harness, the trigger
*and* its denominator, and do you understand that nothing intervenes before it? **[V]** `cc-binary`,
`gemini-code`, `codex-src` · **[S]** `nolima`, `babilong`

**R2. Assume the effective zone is an order of magnitude below the window.** Measured effective
length is 2K–16K for 128K–2M windows; effective utilisation is 10–20%. Test: is your working context
closer to tens of thousands of tokens than to hundreds of thousands? **[S]** `nolima`, `babilong`

**R3. Do not cite "40%".** It is folklore — offered as "a mental model", uncited, on an arbitrary
simulation. Test: if you state a threshold number, can you name the study? **[O]** `ness` · **[S]**
`babilong` (the cited alternative, and stricter)

**R4. Expect gradual decay, not a cliff.** There is no knee in the curve to sit below; every token
costs a little. Test: does your plan assume a safe zone followed by a drop-off? If so it is wrong.
**[S]** `ruler`, `locodiff`, `ctx-rot`

**R5. Do not trust a needle-in-a-haystack result as evidence of capability.** Across 59 models, NIAH
"do[es] not reliably predict downstream performance". Test: is the long-context claim you are relying
on measured on retrieval or on reasoning? **[S]** `helmet`, `ruler`

**R6. Short beats clean.** Length degrades performance 13.9%–85% even with perfect retrieval and
irrelevant content masked out. Test: when reducing context, are you removing tokens or merely
tidying them? **[S]** `length-alone`

---

## B. Session hygiene — the highest-leverage actions

**R7. One task per session; clear at the boundary.** Test: does the current session contain more than
one unrelated objective? **[V]** `cc-best` · **[S]** `multiturn` · **[F]** `ness`, `ace-fca`

**R8. Restart after two failed corrections.** "If you've corrected Claude more than twice on the same
issue in one session, the context is cluttered with failed approaches." Test: count the corrections.
**[V]** `cc-best`

**R9. Prefer a fresh session over a longer conversation when quality wobbles.** The multi-turn damage
is unreliability, not lost capability, and models "do not recover" after a wrong turn. Test: are you
retrying in place, or restarting with a re-specified task? **[S]** `multiturn`

**R10. Scope every investigation.** Unscoped "look into this" is the named failure mode: "Claude reads
hundreds of files, filling the context." Test: does the instruction bound what may be read? **[V]**
`cc-best`

**R11. Split spec from implementation across sessions.** Write the plan to a file, then execute it in
a clean session. Test: did the implementing session inherit the exploration transcript? **[V]**
`cc-best` · **[W]** `ace-fca`

---

## C. Keeping material out of context

**R12. Put durable instructions on disk, not in conversation.** They are the first casualty of
compaction, by vendor admission. Test: would the rule survive a compaction? The API's own memory tool ships the
assumption as its system prompt — "ASSUME INTERRUPTION: Your context window might be reset at any
moment" (`memory-tool`). **[V]** `cc-how`, `cc-best`, `memory-tool` · **[W]** `cc-6354`, `cc-92949`

**R13. Keep the instruction file short.** Perfect-response rate collapses to zero by N=80
instructions; "Bloated CLAUDE.md files cause Claude to ignore your actual instructions!" Test: is
each rule earning its place, and is the file under a few hundred lines? **[S]** `prompt-scale` ·
**[V]** `cc-best`

**R14. Route bulk output to files, then read slices.** The most widely adopted convention in the
corpus. Test: does any single tool result exceed a few thousand tokens? **[F]** `manus`, `cursor-ddc`,
`langchain-ce` · **[V]** `cc-ctxwin`

**R15. Load tool schemas on demand.** Measured at a 46.9% reduction in total agent tokens. Test: are
all MCP schemas resident at turn one? **[S]** `cursor-ddc` · **[V]** `cc-ctxwin`

**R16. Clear old tool results before summarising anything.** Observation masking matches LLM
summarisation's solve rate at half the cost, and both the API and Claude Code implement it as a
distinct, earlier mechanism. Test: is tool-result eviction enabled and firing before compaction?
**[S]** `complexity-trap` · **[V]** `ctx-editing`, `cc-how`

**R17. Remember that caching does not buy space.** "Cached prompt prefixes still occupy the context
window." Test: is prompt caching being treated as a context-management strategy? It is a cost
strategy — worth 41–80% of cost and 13–31% of TTFT, and it backfires if dynamic content sits inside
the cached prefix. **[V]** `ctx-windows`, `gem-caching` · **[S]** `cache`

---

## D. Compaction

**R18. Compact deliberately, at a phase boundary you choose.** Not because auto-compaction is proven
harmful — it is unmeasured — but because summariser quality is worth ~6.5 points and a clean moment
produces a better summary. Test: did the last compaction happen mid-task? **[S]** `compactionrl` ·
**[V]** `compaction-api`

**R19. Do not claim auto-compaction costs N points.** No study measures it. Test: can you cite the
negative control? There isn't one — the literature compares engineered compaction against *no*
compaction, never auto-compaction on versus off. **[S]** `complexity-trap`, `compactionrl`,
`beyond-compaction` (all measure engineered compaction; none is the negative control)

**R20. Re-state critical constraints immediately after a compaction.** The instruction-loss bug has
been open since 2025-08-22 with a live 2026 successor. Test: after a compact, does the agent still
name your constraints without prompting? Some harnesses give you a place to automate this — Devin's
`PostCompaction` hook exists for "re-injecting context that may have been lost during compaction"
(`devin-cli`). **[W]** `cc-6354`, `cc-92949` · **[V]** `cc-how`, `devin-cli`

**R21. Review long-context output for omissions, not errors.** The measured failure is 92–94%
coverage with the sparse critical obligations dropped — the output looks complete. Test: check
against the requirement list, not against the output. **[S]** `coding-rot`

**R22. Treat an early give-up as a context symptom.** Premature termination and refusal rise with
length (refusals 0% → 89.6% near ceilings), independent of task difficulty. Test: did it stop early,
or did it actually finish? Note this can be self-inflicted: a model aware of its own window has been
observed "leaving tasks incomplete when it believed it was near the end of its window, even when it
had plenty of room left" (`devin-sonnet`). **[S]** `premature`, `prompt-scale` · **[W]** `devin-sonnet`

---

## E. Delegation

**R23. Delegate to protect context, not to parallelise.** Budget-controlled, single-agent beat every
multi-agent architecture tested — except in already-degraded contexts. Test: would the sub-agent's
work otherwise drag bulk through the main context? If not, do it inline — five parallel subagents cost
"roughly five times the tokens of a single agent" (`cursor-subagents`). **[S]** `single-agent` ·
**[V]** `cursor-subagents` · **[O]** `cognition`, `cognition-2026`

**R24. Expect a sub-agent to return a summary, not a transcript.** Typical returns are 1,000–2,000
tokens against far larger reads. Test: is the sub-agent's output bounded? **[V]** `cc-ctxwin`,
`eng-context`

**R25. Account for the sub-agent's own overhead.** It loads instruction files against its own window,
and it starts without your history — so it needs its task fully specified. Test: is the brief
self-contained? **[V]** `cc-ctxwin`

---

## F. Reading your instruments

**R26. Know your harness's denominator.** Five different quantities appear under one "%" sign: raw
window, derived input budget, tokens remaining, total input including system and tool definitions, or
a sub-budget that never bounds the prompt. Test: can you name yours? **[V]** `cc-binary`,
`continue-src`, `crush-src`, `cline-src`, `aider-src`, `kilo-src`

**R27. Do not compare thresholds across harnesses without normalising.** Codex's 90%, Zed's 90% and
Claude Code's ≈93.5% are four different measurements. Test: same denominator? **[V]** `codex-src`,
`zed-src`, `cc-binary`, `opencode-src` 
**R28. Verify a threshold against source, not documentation.** Gemini CLI ships 0.5 while its docs
publish 0.7; Aider's repo map is 4096 where its docs say "1k"; Continue's comment says 80% where the
code trips at ≈60.5%. Test: did you read the constant, at a ref you can name? Beware also of
constants that are real but retired — Cline's `contextWindow − 40_000` rule is legacy
(`cline-legacy`). **[V]** `gemini-code`/`gemini-docs`, `aider-src`, `continue-src`, `cline-legacy`

**R29. Check whether your context setting is actually active.** Roo Code's slider ships at 100%
(inert); Zed's auto-compact is silently inert below an 80K window; Cline silently falls back from
summarisation to truncation on non-Claude models; Windsurf's documented memories "apply to the legacy
Cascade agent only", not the default one. Two whole lineages ship the
percentage trigger disabled — Roo resolves it to 100, Kilo leaves it unset. Test: has the setting ever
been observed to fire? **[V]** `roo-docs`, `roo-default`, `kilo-src`, `zed-src`, `cline-docs`,
`windsurf-memories`, `windsurf-plugins`

**R30. Do not treat a displayed percentage as a fraction of the model window.** Claude Code computes
it against the threshold-adjusted window; Continue counts only chat history while the trigger counts
more. Test: does the number on screen reconcile with a token count you computed yourself? **[V]**
`cc-binary`, `continue-src`

**R31. Expect the boundary to move without a release.** Claude Code resolves its window through
server-side `clientdata` and `experiment` sources; Gemini CLI's getter is async and local-first. Test:
is your rule pinned to a number the vendor controls? **[V]** `cc-binary` · **[V, inferred]**
`gemini-code`

**R32. Know whether compaction is even enforced.** Claude Code has a *reactive mode* in which it does
not compact proactively and waits for the API to reject the prompt. Test: countdown or bare
percentage? **[V]** `cc-binary`

**R33. Do not quote an example payload as a threshold.** Cursor's `85` appears in a sample `preCompact`
body; its actual threshold is unpublished — and its vendor concedes the trigger "can trigger late or
incorrectly" (`cursor-forum`). Beware equally of a documented default that does not exist: Kilo's
"(default: ~80%)" appears in no code path (`kilo-docs`). Test: is the number from a normative
statement, or from a sample? **[V]** `cursor-hooks`, `kilo-docs` ·
**[W]** `cursor-forum`

**R33b. Verify a harness setting after any upgrade or fork change.** Lineages move: Kilo Code stopped
being a Roo Code fork and became an OpenCode one, taking its condensing settings with it. Test: does
the key you configured still exist? **[V]** `kilo-src`, `cc-settings`

**R34. Re-check your numbers when a model changes.** Claude 4.7+ tokenizers produce "approximately
30% more tokens for the same text", so the same repository consumes a different share of the same
window. Test: was the budget last calibrated on the model you are now running? **[V]** `pricing`

---

## G. Position and safety

**R35. Put the important thing first or last, never the middle.** Still measured in 2026 across
retrieval, injection defence and monitoring. Test: where does the constraint sit in the prompt?
**[S]** `lost-middle`, `lim-emergent`, `longpibench`, `classifier-rot`

**R36. Put the query after the context.** Vendor guidance, and consistent with the above. Test: does
the instruction precede the material it applies to? **[V]** `gem-longctx`

**R37. Treat long context as an elevated injection risk, not a diluted one.** Attack success rises
from 0.53 to 0.87 when the injection sits at the end rather than the front, and defences decay with
length. Test: is untrusted tool output accumulating in a long session? **[S]** `longpibench`

**R38. Do not rely on automated oversight over a long transcript.** Monitor recall degrades 2×–30×;
Opus 4.6 falls 99.7% → 69% between 100K and 800K. Test: is the review of a long session automated?
**[S]** `classifier-rot`

**R39. Assume a single distractor costs you.** "Even a single distractor reduces performance relative
to the baseline." Test: is anything in context that the task does not need? **[S]** `ctx-rot`

---

## Review checklist

A long-running session's output does not ship until every one of these passes:

| # | Gate | Rule |
|---|---|---|
| 1 | The session covers one task, and started clean | R7, R8 |
| 2 | Durable constraints live on disk and are short | R12, R13 |
| 3 | No single tool result or file dump dominates context | R14, R15, R16 |
| 4 | Output was checked against the requirement list for **omissions** | R21 |
| 5 | Any compaction was followed by re-stating critical constraints | R20 |
| 6 | An early stop was investigated as a context symptom, not accepted | R22 |
| 7 | Delegation was chosen to protect context, not to parallelise | R23 |
| 8 | The harness threshold's denominator is known, and the setting verified active | R26, R29 |
| 9 | No threshold was quoted from documentation, an example, or a blog | R28, R33 |
| 10 | Critical instructions sit at the start or end, not the middle | R35 |
| 11 | Long sessions carrying untrusted tool output were reviewed by a human | R37, R38 |
| 12 | No claim was made about auto-compaction's accuracy cost | R19 |

**The two that gate everything else:** R1 — the harness is not keeping you in the smart zone and was
never trying to — and R21, because the characteristic failure leaves coverage high and quietly drops
the things that mattered, which means it passes a casual review by construction.
