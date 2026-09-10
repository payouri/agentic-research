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
