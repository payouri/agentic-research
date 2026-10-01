# Agent memory rulebook

Checkable rules for persistent agent memory. Each rule states its test (how a third party checks
compliance), its evidence tier, and its source keys (see [sources.md](sources.md)).

Tiers: **P1** primary spec, vendor docs or shipped source · **P2** peer-reviewed or preprint
research · **P3** vendor engineering blog / industry research with method · **P4** practitioner
measurement (including this dossier's own counts) · **P5** opinion, anecdote, unverified.

Use it two ways: as a design checklist when enabling or building agent memory, and as a review gate
for a memory directory or a memory integration. A setup that fails R1, R6, R19 or R24 does not pass,
whatever else it gets right.

---

## A. Whether to use memory, and for what

**R1. Keep rules out of memory.** Anything that must always apply goes in a reviewed instruction file
(CLAUDE.md, AGENTS.md, GEMINI.md, rules), not in learned memory. Test: does any memory entry state an
imperative ("always", "never", "must") that is not also in an instruction file? **P1** `codex-docs`,
`windsurf`, `cc-memory` · **P4** `corpus`

**R2. Don't enable memory expecting the agent to get smarter.** Off-the-shelf memory systems did not
beat memory-off on real coding tasks. Test: is there a measured, task-level reason for enabling it, on
your workload, against a memory-off baseline? **P2** `vibemembench`, `agentsmd-eth`

**R3. Know your tool's default.** Claude Code, Copilot (individual), Letta and consumer Claude/M365
have memory on by default; Codex, Gemini Auto Memory, OpenHands, Goose and Kilo off. Test: can you
state, per tool in use, whether memory is on and where it is stored? **P1** `cc-memory`, `cc-binary`,
`copilot-mem`, `codex-docs`, `gemini-src`, `m365-mem`

**R4. Prefer full context when the history fits the effective zone.** Retrieval-based memory wins
only once the history outgrows what the model reads well. Test: for conversational recall, has the
memory layer been compared to simply reading the history, at your history length? **P2**
`mem0-paper`, `zep-paper`, `longmemeval` · see `dossier-ctx`

**R5. Turn user-profile memory off where disagreement matters.** Profiles measurably increase
agreement sycophancy. Test: is user-level memory disabled, or justified, for agents whose job includes
review or critique? **P2** `sycophancy` [ABS]

## B. What to write

**R6. Write only what is durable, non-derivable, and not already written down.** No architecture,
file paths, change summaries, bug-fix narratives or task state. Test: sample ten entries; can any be
reconstructed from the repo, its README/instruction files, or git log? **P1** `cc-memory`,
`gemini-src` · **P2** `expfollow` · **P4** `corpus`

**R7. Filter before you store.** A strict admission policy beat store-everything by up to 25 points.
Test: does every write path (agent, background extractor) pass through a selection criterion stronger
than "this happened"? **P2** `expfollow`, `vibemembench`, `evomemory`

**R8. Prefer verified experience over volunteered experience.** Test: does any automated write require
a passing check (test, build, user confirmation) on the thing it records? **P2** `vibemembench`,
`reasoningbank`

**R9. Distil; don't store transcripts.** Short curated summaries beat full trajectories; verbatim
storage causes instruction pollution. Test: is any memory entry a raw transcript, log or tool output?
**P2** `swecontextbench`, `vibemembench`, `memtransfer` · **P4** `c-macroflows`

**R10. Don't let the agent pick its own summary unchecked.** Self-selected summaries scored below no
memory. Test: is an agent-written summary reviewed or validated before it is reused? **P2**
`swecontextbench`

**R11. Write corrections first.** The highest-value entry type is "X, not Y" for a mistake that would
otherwise recur. Test: does the memory contain the corrections from the last sessions in which the
agent was corrected? **P4** `c-quorum`, `corpus` · **P1** `cc-memory`

**R12. Attach provenance to each fact.** Date and method, so a later session can re-check rather than
trust. Test: does each factual entry say when, and how, it was established? **P1** `cc-binary`,
`cc-memory` (`modified` timestamp) · **P4** `c-bubbly`

**R13. Point to authority instead of copying it.** Test: where an entry summarises a doc, config or
log, does it link to the source and name which source wins? **P4** `c-warden`, `c-qc1c`

**R14. No secrets, ever.** Test: does a secret scanner over the memory store return zero hits, and does
the write path redact? **P1** `codex-src`, `letta-code`, `mt-docs`, `ma-memory`

**R15. No personal data in shared or committed memory.** Test: is the memory directory free of names,
emails and behavioural notes about identifiable people, or is it machine-local and excluded from VCS?
**P4** `c-pii`, `corpus` · **P1** `cc-memory`, `claude-app`

## C. How to structure it

**R16. Keep a lean index; push detail into topic files.** Test: is the always-loaded file a list of
one-line pointers, each under ~200 characters? **P1** `cc-binary`, `cc-memory`, `ma-memory`,
`gemini-src`

**R17. Stay under the load cap with margin.** Claude Code 200 lines / 25,000 bytes (head kept);
OpenHands 6,000 chars (tail kept); Codex 2,500 tokens summary, 8,900 bytes per item; Kilo 8,192 bytes.
Test: is the index below 80% of its tool's cap? **P1** `cc-binary`, `openhands-sdk`, `codex-src`,
`kilo-src`

**R18. One fact, one place.** Update in place; never append a second version. Test: grep for the same
subject in two entries or two tiers — any hits? **P1** `gemini-src`, `cc-binary` · **P2**
`memagentbench`

## D. Keeping it true

**R19. Expire or re-verify on a schedule.** Most coding tools never expire anything. Test: is there a
documented pass — automatic or calendar — that deletes or re-checks entries unused beyond a set age
(vendor defaults on record: 28 and 30 days)? **P1** `copilot-mem`, `codex-src`, `mt-docs`,
`memorybank` · **P4** `corpus`

**R20. Treat memory as a claim about the past.** Test: does the agent's read-path instruction say to
verify a remembered file, function or flag before acting on it? **P1** `codex-src`, `cc-binary`

**R21. Don't rely on a memory layer to overwrite changed facts.** Commercial layers scored 2–3% on
multi-hop updates; several are append- or ADD-only. Test: when a fact changes, is the old entry
deleted or edited by a person or a deterministic process, not left for the layer to reconcile? **P2**
`memagentbench` · **P1** `mem0-src`, `graphiti-src`, `mcp-memory`, `goose-src`

**R22. Purge frozen task state.** Test: does any entry describe work "in progress", "next steps", or a
PR/branch that is merged or gone? **P4** `corpus`, `c-macroflows`, `c-chatrelay`, `c-onejs`

**R23. Know what writes between your sessions.** Codex and Claude Code run background consolidation
that applies (and, in Claude Code, may delete) without review. Test: can you name every process that
writes to the memory store, and does a human see its changes before they take effect? **P1**
`codex-src`, `cc-binary`, `gemini-src`

## E. Security

**R24. Make memory read-only or off for agents that read untrusted content.** Test: for any agent
processing issues, web pages, PR text or third-party READMEs, is the memory store read-only at the
filesystem/API level, or disabled? **P1** `ma-memory`, `cursor-auto` · **P2** `pmpa`, `memorygraft`,
`minja`

**R25. Frame memory as data, not instructions, on read.** Test: is memory content delivered fenced and
labelled as untrusted context, outside the system prompt? **P1** `codex-src`, `kilo-src`,
`gemini-src`, `cc-memory`

**R26. Don't treat moving memory out of the system prompt as a fix.** Claude Code still injects it
every session. Test: does your threat model assume a poisoned memory reaches the model? **P1**
`cc-memory`, `cc-binary` · **P3** `cisco-cc`

**R27. Monitor the memory directory as a persistence path.** Test: are writes to
`~/.claude/projects/*/memory/`, `~/.codex/memories/`, `~/.gemini/` and equivalents covered by the same
integrity monitoring as hooks and shell rc files? **P3** `cisco-cc` · see `dossier-git`

**R28. Validate every memory-tool path.** If you implement Anthropic's memory tool, canonicalise and
reject traversal (`../`, `..\`, `%2e%2e%2f`) in every command, and refuse delete/rename of the root.
Test: does a `view /memories/../../etc/passwd` call fail? **P1** `mt-docs`

**R29. Make the memory tool do what the model is told.** The model expects `create` to overwrite and
`view` to truncate at 16,000 characters; the SDK helper does neither. Test: does your handler either
match the description or return an error message that says exactly what happened? **P1** `mt-docs`,
`sdk-py-memory`

**R30. Version the store.** Test: can you roll the memory store back to its state before a given
session (git, immutable versions)? **P1** `ma-memory`, `letta-code`, `codex-src`

**R31. Don't rely on detection defences alone.** Published guards contradict each other and
prompt-level detection is inconsistent across agents. Test: is there a structural control (R24, R30,
review) in addition to any detector? **P2** `amemguard`, `farma`, `minja`

## F. Sharing and portability

**R32. Never commit auto-memory without reading it.** Test: is every committed memory file
human-reviewed, and is the tool's machine-local memory directory excluded from VCS by default? **P1**
`cc-memory` · **P4** `c-pii`, `c-brakeza`

**R33. Assume memory is not portable.** No standard exists. Test: is anything you need to survive a tool
change kept in a file you own rather than a vendor store? **P1** `aaif`, `w3c-cg`, `claude-app`,
`cursor-forum`

**R34. Scope deliberately.** Tools key memory by repo, project id, user, agent or workspace. Test: can
you say which scope each memory lives in, and is project state absent from user-global memory? **P1**
`cc-memory`, `gemini-src`, `codex-src`, `letta-code` · **P4** `corpus`

## G. Evaluating memory products

**R35. Don't rank on LoCoMo.** Answer key, judge leniency and public harness are all disputed. Test: is
any purchasing or design decision justified by a LoCoMo score alone? **P4** `locomo-audit` · **P1**
`mem0-harness` · **P3** `zep-rebuttal`, `letta-bench`

**R36. Demand a memory-off baseline on coding tasks.** Test: does the claim compare against the same
agent with memory disabled, on repository tasks? **P2** `vibemembench`, `swecontextbench`

**R37. Discount undisclosed internal evals.** Test: does the cited number name its dataset, size,
model and baseline? **P5** `ae-ctxmgmt`, `mem0-2026`

**R38. Read the shipped defaults, not the documented ones.** Codex's docs state 16 and 30 where the
code ships 2 and 10. Test: are memory-related defaults you depend on set explicitly in config? **P1**
`codex-config`, `codex-src`

---

## Review checklist

A memory setup does not pass review until every one of these holds:

| # | Gate | Rules |
|---|---|---|
| 1 | Rules live in instruction files, not memory | R1 |
| 2 | Writes are filtered: durable, non-derivable, verified, distilled | R6, R7, R9 |
| 3 | No secrets and no personal data in any shared or committed store | R14, R15, R32 |
| 4 | Index lean and under its cap; one fact, one place | R16, R17, R18 |
| 5 | Scheduled expiry or re-verification exists | R19, R22 |
| 6 | Every writer is known; background writes are reviewable or versioned | R23, R30 |
| 7 | Read-only or off for agents processing untrusted input | R24 |
| 8 | Memory treated as untrusted data on read; directory monitored | R25, R26, R27 |
| 9 | Product claims backed by a coding-task, memory-off baseline | R35, R36 |

**The four rules that gate everything else:** R1, because a rule stored as memory is a rule nobody
reviews; R6, because unfiltered writes are the measured way memory makes agents worse; R19, because a
memory that never expires is a memory that is eventually wrong; and R24, because persistence turns a
one-session injection into a standing one.
