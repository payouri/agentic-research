# AGENTS.md source lexicon

Every source behind [guide.md](guide.md) and
[rulebook.md](rulebook.md), keyed for citation. Gathered 2026-09-10.

Trust tiers: **P1** primary spec/vendor documentation · **P2** peer-reviewed or arXiv research ·
**P3** vendor engineering blog / industry research with method · **P4** practitioner report with
measurement · **P5** opinion, anecdote, or unverified secondary.

---

## 1. The standard

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `agentsmd-site` | [agents.md](https://agents.md/) | P1 | live | The whole normative surface: 4 FAQ answers, 5 suggested sections, nesting, README split |
| `agentsmd-repo` | [agentsmd/agents.md](https://github.com/agentsmd/agents.md) | P1 | pushed 2026-08-25 | That there is **no spec** — no `SPEC.md`, no schema; `/spec` and `/schema.json` 404. Canonical org is `agentsmd`, **not** `openai` |
| `agentsmd-faq` | [FAQSection.tsx](https://raw.githubusercontent.com/agentsmd/agents.md/main/components/FAQSection.tsx) | P1 | — | "No. AGENTS.md is just standard Markdown"; "The closest AGENTS.md to the edited file wins" |
| `agentsmd-howto` | [HowToUseSection.tsx](https://raw.githubusercontent.com/agentsmd/agents.md/main/components/HowToUseSection.tsx) | P1 | — | Nesting guidance; "the main OpenAI repo has 88 AGENTS.md files" |
| `agentsmd-why` | [WhySection.tsx](https://raw.githubusercontent.com/agentsmd/agents.md/main/components/WhySection.tsx) | P1 | — | The README-vs-AGENTS.md division of labour |
| `agentsmd-compat` | [CompatibilitySection.tsx](https://raw.githubusercontent.com/agentsmd/agents.md/main/components/CompatibilitySection.tsx) | P1 | — | The authoritative 23-tool list. **Anthropic is not on it** |
| `lf-aaif` | [LF: Agentic AI Foundation](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation) | P1 | 2025-12-09 | Governance: donated to the Linux Foundation alongside MCP and goose |

## 2. Tool implementations (merge semantics, caps, filenames)

| Key | Source | Tier | What it settles |
|---|---|---|---|
| `codex-agents-md` | [Custom instructions with AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md) | P1 | **The 32 KiB cap** (`project_doc_max_bytes`), root→cwd concatenation, one file per directory, `AGENTS.override.md`, filename allowlist. No import mechanism. Never mentions CLAUDE.md |
| `anthropic-memory` | [How Claude remembers your project](https://code.claude.com/docs/en/memory) | P1 | "Claude Code reads `CLAUDE.md`, not `AGENTS.md`"; 4-scope hierarchy; concatenation not override; `@` imports don't save context, depth 4; delivered as a **user message after** the system prompt; 4 MiB skip; 200-line target; HTML comments stripped; `/import` (v2.1.213+) |
| `cursor-rules` | [Cursor — Rules](https://cursor.com/docs/rules) | P1 | Reads AGENTS.md **and** CLAUDE.md; merges, "more specific instructions taking precedence" |
| `amp-agents-md` | [Amp — AGENTS.md](https://ampcode.com/docs/customize/agents-md) | P1 | AGENTS.md → `AGENT.md` → `CLAUDE.md` fallback chain; lazy subtree loading; `globs:` frontmatter |
| `zed-instructions` | [Zed — Instructions](https://zed.dev/docs/ai/instructions) | P1 | **First match of 9 filenames wins** — `.cursorrules` ranks above `AGENTS.md`, `AGENT.md` above it too |
| `warp-rules` | [Warp — Rules](https://docs.warp.dev/knowledge-and-collaboration/rules) | P1 | **ALL CAPS required**; nearest-wins precedence; `WARP.md` beats `AGENTS.md` in the same dir |
| `opencode-rules` | [opencode — Rules](https://opencode.ai/docs/rules/) | P1 | "The first matching file wins in each category" — AGENTS.md and CLAUDE.md mutually exclusive; does not parse `@` references |
| `factory-agents-md` | [Factory — AGENTS.md](https://docs.factory.ai/cli/configuration/agents-md) | P1 | 80,000 / 40,000 char caps; case-lenient filenames |
| `windsurf-agents-md` | [Devin Desktop — AGENTS.md](https://docs.devin.ai/desktop/cascade/agents-md) | P1 | Case-**insensitive**; 6,000 / 12,000 char rule caps (application to AGENTS.md itself is inference) |
| `devin-cli-rules` | [Devin CLI — Rules](https://docs.devin.ai/cli/extensibility/rules) | P1 | Reads AGENTS.md, AGENTS.local.md, CLAUDE.md and `~/.claude/CLAUDE.md`; "treated identically… always-on rules" |
| `gemini-cli-context` | [Gemini CLI — context files](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/gemini-md.md) | P1 | Default is `GEMINI.md`; AGENTS.md only via `context.fileName` |
| `copilot-repo-instructions` | [GitHub — repository custom instructions](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions) | P1 | **"the nearest `AGENTS.md`… will take precedence"** — one of only two tools honouring the FAQ |
| `vscode-custom-instructions` | [VS Code — Custom instructions](https://code.visualstudio.com/docs/copilot/customization/custom-instructions) | P1 | Nested support is experimental (`chat.useNestedAgentsMdFiles`); "no specific order is guaranteed" |
| `aider-conventions` | [Aider — coding conventions](https://aider.chat/docs/usage/conventions.html) | P1 | **No auto-discovery of any instruction file**, despite being on the compatibility list. Needs `read: AGENTS.md` |
| `charlie-docs` | [Charlie — AGENTS.md instructions](https://docs.charlielabs.ai/AGENTS.md-instructions) | P1 | "Make overrides explicit instead of requiring the agent to infer them" |
| `goose-hints` | [goose — providing hints](https://goose-docs.ai/docs/guides/context-engineering/using-goosehints/) | P1 | Genuine AGENTS.md default; `CONTEXT_FILE_NAMES` override |
| `junie-guidelines` | [Junie — guidelines and memory](https://junie.jetbrains.com/docs/guidelines-and-memory.html) | P1 | `.junie/AGENTS.md` precedence chain; dedupes identical content |
| `roo-instructions` | [Roo Code — custom instructions](https://roocodeinc.github.io/Roo-Code/features/custom-instructions) | P1 | Docs say root-only; source has undocumented `enableSubfolderRules` (default false) — docs/source conflict |
| `cline-rules` | [Cline — Rules](https://docs.cline.bot/customization/cline-rules) | P1 | Merges; workspace wins |
| `jules-docs` | [Jules — getting started](https://jules.google/docs/) | P1 | Thinnest docs of any listed tool: root AGENTS.md only, no nesting/merge/limit documented |
| `git-core-config` | [git `core.symlinks`](https://raw.githubusercontent.com/git/git/master/Documentation/config/core.adoc) | P1 | Why a `CLAUDE.md → AGENTS.md` symlink can check out as a plain text file |

**URLs that have moved** (relevant if anything is cached): `developers.openai.com/codex/guides/agents-md`
→ 308 → `learn.chatgpt.com/...`; `zed.dev/docs/ai/rules` → 404 → `/docs/ai/instructions`;
`docs.windsurf.com/...` → 307 → `docs.devin.ai/desktop/...`; `docs.claude.com/en/docs/claude-code/*`
→ 301 → `code.claude.com/docs/en/*`; the April-2025 "Claude Code best practices" engineering post no
longer exists standalone — it redirects into the docs.

## 3. Anthropic guidance

| Key | Source | Tier | What it settles |
|---|---|---|---|
| `anthropic-best-practices` | [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) | P1 | **The deletion test**; the include/exclude table; "Bloated CLAUDE.md files cause Claude to ignore your actual instructions!"; emphasise one line only; the two symptom diagnostics; "The over-specified CLAUDE.md" |
| `anthropic-features` | [Extend Claude Code](https://code.claude.com/docs/en/features-overview) | P1 | The context-cost table (CLAUDE.md every request vs skills low vs hooks zero); "a request, not a guarantee" — rules vs hooks |
| `anthropic-skills` | [Skills](https://code.claude.com/docs/en/skills) | P1 | When a CLAUDE.md section has "grown into a procedure rather than a fact"; body loads on invocation; 500-line SKILL.md target; description truncated at 1,536 chars |
| `anthropic-context-eng` | [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | P3 | The "attention budget"; context rot; "smallest possible set of high-signal tokens"; **the "right altitude" framing**; just-in-time / progressive disclosure; the overlapping-tools failure mode |
| `anthropic-tools` | [Writing effective tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents) | P3 | "think of how you would describe your tool to a new hire"; surface implicit context; small wording changes yield large gains |
| `anthropic-skills-eng` | [Equipping agents for the real world with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) | P3 | The canonical 3-level progressive-disclosure model; "start with evaluation" |
| `anthropic-prompting` | [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) | P1 | **The colleague test**; "tell Claude what to do instead of what not to do"; attach the reason; 3–5 tagged examples; longform data at top ("up to 30 percent" on 20k+ inputs) |
| `anthropic-blog-steering` | [Steering Claude Code](https://claude.com/blog/steering-claude-code-skills-hooks-rules-subagents-and-more) | P3 | 2026-06-18. "Keep CLAUDE.md under 200 lines, give it an owner, and review changes to it like code"; the per-team cost framing; procedures → skills |

## 4. Research

| Key | Paper | Tier | Date | Finding |
|---|---|---|---|---|
| `eth-agentsmd` | [Evaluating AGENTS.md](https://arxiv.org/abs/2602.11988) — Gloaguen, Mündler, Müller, Raychev, Vechev (ETH Zurich SRI Lab) | P2 | 2026-02-12, rev 2026-06-23 | **No general gain in task success; >20% higher inference cost. Repository overviews measurably unhelpful. Instructions themselves are well followed.** Caveats: Python-only, issue-resolution as the sole metric |
| `lulla-efficiency` | [On the Impact of AGENTS.md Files on the Efficiency of AI Coding Agents](https://arxiv.org/abs/2601.20404) — Lulla et al. | P2 | 2026-01 (ICSE 2026 JAWs) | 10 repos, 124 PRs: median runtime **−28.64%**, output tokens **−16.58%**, comparable completion. Percentages taken from the abstract, not a PDF fetch |
| `lost-in-middle` | [Lost in the Middle](https://arxiv.org/abs/2307.03172) — Liu, Lin, Hewitt, Paranjape, Bevilacqua, Petroni, Liang | P2 | 2023 (TACL) | U-shaped retrieval by position; the middle is the worst place for a rule |
| `ifscale` | [How Many Instructions Can LLMs Follow at Once?](https://arxiv.org/abs/2507.11538) — Jaroslawicz, Whiting et al. | P2 | 2025 | 500 instructions, 20 models: best reach **68%**; explicit "bias towards earlier instructions"; three degradation shapes |
| `ifeval` | [IFEval](https://arxiv.org/abs/2311.07911) — Zhou et al. | P2 | 2023 | The original verifiable instruction-following benchmark |
| `multi-if` | [Multi-IF](https://arxiv.org/abs/2410.15553) | P2 | 2024 | Adherence decays across turns — the regime an instruction file actually lives in |
| `prime` | [PRIME](https://arxiv.org/abs/2606.22470) | P2 | 2026 | Models "seldom recognize contradictions or request clarification" — contradictions are invisible failures |
| `ih-benchmark` | [IH-Benchmark](https://arxiv.org/abs/2607.25987) | P2 | 2026 | Instruction-hierarchy compliance ranges 98.2%→20.5% across 37 model variants |
| `many-tier-ih` | [Many-Tier Instruction Hierarchy](https://arxiv.org/abs/2604.09443) | P2 | 2026 | Adherence degrades as instruction tiers increase; best models <50% |
| `negation` | [How Language Models Process Negation](https://arxiv.org/abs/2605.03052) — Zhou, Zhou, Jia, May | P2 | ICML 2026 | Negation failures come from "late-layer attention behavior that promotes simple shortcuts". Evidence about *comprehension*, **not** about prohibitions backfiring |
| `found-in-middle` | [Found in the Middle](https://arxiv.org/abs/2406.16008) | P2 | 2024 | Positional-attention recalibration as mitigation |
| `context-rot` | [Context Rot](https://www.trychroma.com/research/context-rot) — Hong, Troynikov, Huber (Chroma) | P3 | 2025-07-14 | 18 models: "Even a single distractor reduces performance relative to the baseline" |
| `arize-prompt-learning` | [CLAUDE.md best practices from prompt learning](https://arize.com/blog/claude-md-best-practices-learned-from-optimizing-claude-code-with-prompt-learning/) | P3 | 2025-11 | +5.19% held-out / +10.87% within-repo from optimising the prompt via a failure-explanation loop. Vendor blog, no CIs |
| `github-2500` | [How to write a great AGENTS.md — lessons from over 2,500 repositories](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/) — Matt Nigh | P3 | 2025-11-19 | "Most agent files fail because they're too vague"; grow through iteration. No sampling method or metrics disclosed |

## 5. Security

| Key | Source | Tier | Date | Finding |
|---|---|---|---|---|
| `nvidia-injection` | [Mitigating indirect AGENTS.md injection attacks](https://developer.nvidia.com/blog/mitigating-indirect-agents-md-injection-attacks-in-agentic-environments/) — D. Teixeira, NVIDIA AI Red Team | P3 | 2026-04-20 | Working PoC: compromised Go dependency writes an AGENTS.md that injects a regression into `main` and hides it from PR summaries. Mitigations list |
| `backslash` | [OpenAI Codex injection in AGENTS.md — exfiltrating credentials](https://www.backslash.security/blog/openai-codex-injection-in-agents-md-exfiltrating-credentials) — A. Waizman | P3 | 2026-07-06 | `exec` mode reads `~/.aws/credentials`, `~/.npmrc`, `~/.gitconfig` with no prompt. "Safety controls are mode-dependent rather than invariant." Specific payload since blocked |
| `injection-sok` | [Prompt Injection Attacks on Agentic Coding Assistants](https://arxiv.org/abs/2601.17548) — Maloyan, Namiot | P2 | 2026-01-24 | SoK, 78 studies, 42 techniques: >85% ASR against SOTA defences under adaptive attack; most defences <50% mitigation |
| `gitinject` | [GitInject](https://arxiv.org/html/2606.09935v1) | P2 | 2026 | A PR-branch AGENTS.md loads "as a trusted configuration to be followed" before the review meant to catch it |
| `livepi` | [LivePI](https://arxiv.org/pdf/2605.17986) | P2 | 2026 | Realistic indirect-injection benchmarking |
| `iclr-ih` | [Improving LLM Safety with Instruction Hierarchy](https://proceedings.iclr.cc/paper_files/paper/2025/file/ea13534ee239bb3977795b8cc855bacc-Paper-Conference.pdf) | P2 | ICLR 2025 | Training models to prioritise higher-privilege instructions |

## 6. The corpus of real files

Key `corpus` throughout. 37 files fetched from default branches on 2026-09-10.
**Median ≈ 155 lines / ~1200 words**; cluster 60–250.

**Exemplary (cited in the guide and rulebook):**

- [openai/codex](https://github.com/openai/codex/blob/main/AGENTS.md) — 320 lines. The 800/500-line diff budget
- [apache/airflow](https://github.com/apache/airflow/blob/main/AGENTS.md) — 537 lines. Boundaries block; the "Dag" rule with its exception list
- [vercel/next.js](https://github.com/vercel/next.js/blob/canary/AGENTS.md) — 560 lines. "Context-Efficient Workflows"; instruction-shaped headings; nested `packages/next/AGENTS.md`; `CLAUDE.md` symlink
- [huggingface/transformers](https://github.com/huggingface/transformers/blob/main/.ai/AGENTS.md) — 40 lines. **The best gotchas in the corpus**; root symlinks documented
- [astral-sh/uv](https://github.com/astral-sh/uv/blob/main/AGENTS.md) — 32 lines, no headings, 30 ALWAYS/NEVER/PREFER/AVOID lines. Rule+rationale density
- [ghostty-org/ghostty](https://github.com/ghostty-org/ghostty/blob/main/AGENTS.md) — 39 lines. Narrow-test reason inline; the tripwire idiom
- [rust-lang/rust-analyzer](https://github.com/rust-lang/rust-analyzer/blob/master/CLAUDE.md) — 60 lines. "Validation"; narrowest-first; the `UPDATE_EXPECT=1` rule
- [biomejs/biome](https://github.com/biomejs/biome/blob/main/AGENTS.md) — 79 lines. "Implementation Gate"; the evidence standard; fresh-subagent review; skills delegation
- [colinhacks/zod](https://github.com/colinhacks/zod/blob/main/AGENTS.md) — 183 lines. Named forbidden code shapes + stated consequence
- [cloudflare/agents](https://github.com/cloudflare/agents/blob/main/AGENTS.md) — 196 lines. "Boundaries"; documented nested files; `pnpm run check` gate
- [cloudflare/workers-sdk](https://github.com/cloudflare/workers-sdk/blob/main/AGENTS.md) — 153 lines. **"Prefer authoritative configuration… copied versions become stale"**
- [getsentry/sentry](https://github.com/getsentry/sentry/blob/master/AGENTS.md) — 137 lines. Names the exact failure modes agents hit
- [prisma/prisma](https://github.com/prisma/prisma/blob/main/AGENTS.md) — 130 lines. "Golden Rules" + "Ask First"; prohibition + enforcement plugin
- [grafana/grafana](https://github.com/grafana/grafana/blob/main/AGENTS.md) — 172 lines. The push-approval loophole closer (and some platitudes)
- [pytorch/pytorch](https://github.com/pytorch/pytorch/blob/main/CLAUDE.md) — 343 lines. "AI Policy — MANDATORY"; git-ignored `agent_space/`
- [vitest-dev/vitest](https://github.com/vitest-dev/vitest/blob/main/AGENTS.md) — 240 lines. Comment policy; social-pressure prohibitions
- [withastro/astro](https://github.com/withastro/astro/blob/main/AGENTS.md) — 165 lines. "Every changed line should trace directly to the user's request"
- [supabase/supabase](https://github.com/supabase/supabase/blob/master/AGENTS.md) — 75 lines. Nested app files; do-not-edit globs + regen command
- [danny-avila/LibreChat](https://github.com/danny-avila/LibreChat/blob/main/AGENTS.md) — the false-green warning
- [openai/openai-agents-js](https://github.com/openai/openai-agents-js/blob/main/AGENTS.md) — verification tiers done well, policy prose done badly

**Cautionary:**

- [All-Hands-AI/OpenHands](https://github.com/All-Hands-AI/OpenHands/blob/main/AGENTS.md) — **714 lines / 14,476 words.** Right content, wrong file
- [temporalio/temporal](https://github.com/temporalio/temporal/blob/main/AGENTS.md) — a pasted CLI system prompt; nothing Temporal-specific
- [TanStack/router](https://github.com/TanStack/router/blob/main/AGENTS.md) — product marketing under "Code style"; triple-duplicated commands
- [microsoft/vscode](https://github.com/microsoft/vscode/blob/main/AGENTS.md) — 5 lines, pure redirect, zero commands
- [oven-sh/bun](https://github.com/oven-sh/bun/blob/main/CLAUDE.md) — "Important Development Notes" junk-drawer heading

**Also measured:** astral-sh/ruff (203) · openai/openai-python · home-assistant/core (56) ·
modelcontextprotocol/python-sdk (166) · temporalio sdk-go (163) / sdk-dotnet (155) / sdk-python (117) /
sdk-java (59) · getsentry/sentry-docs (113) · TanStack/query (10) · vercel/turborepo (27) ·
nushell/nushell (21).

## 7. Practitioner and opinion

| Key | Source | Tier | Use |
|---|---|---|---|
| `instruction-ablation` | [evolsb/claude-instruction-ablation](https://github.com/evolsb/claude-instruction-ablation) | P4 | Six-repo case study, Aug 2026: 63,572 → 18,200 words (−71%); staleness beats volume as the problem |
| `unblocked-audit` | [How to audit a bloated CLAUDE.md in 7 steps](https://getunblocked.com/blog/audit-fix-bloated-claude-md/) | P4 | The audit loop: run every command, probe every rule in a fresh session |
| `pink-elephant` | [The pink elephant: negative instructions](https://eval.16x.engineer/blog/the-pink-elephant-negative-instructions-llms-effectiveness-analysis) | P5 | The negatives-backfire claim — self-admittedly "anecdotal evidence and not controlled experiments" |
| `augment-good-agents` | [How to write good AGENTS.md files](https://www.augmentcode.com/blog/how-to-write-good-agents-dot-md-files) | P5 | "A good AGENTS.md is a model upgrade. A bad one is worse than no docs at all" |
| `crosley-patterns` | [AGENTS.md patterns](https://blakecrosley.com/blog/agents-md-patterns) | P5 | Author's own 10+-run A/B per pattern; no numbers published |
| `osmani-init` | [Stop using /init for AGENTS.md](https://addyosmani.com/blog/agents-md/) | P5 | Argues against generated files |
| `kerneltalks-mixed` | [AGENTS.md just turned one — the evidence is mixed](https://kerneltalks.com/ai/agents-md-just-turned-one-the-evidence-on-whether-it-works-is-mixed/) | P5 | Useful roundup of the correctness-vs-efficiency split |

---

## Conflicts resolved during this research

1. **The 32 KiB Codex cap** was reported second-hand and unverified by one line of research, and
   confirmed from OpenAI's own docs (`project_doc_max_bytes`) by another. **Confirmed, P1.**
2. **"Closest AGENTS.md wins"** — the standard's own FAQ is contradicted by most implementers. Only
   Copilot coding agent and Warp behave that way; twelve tools concatenate; Zed and opencode load
   exactly one file. **The FAQ is aspirational.**
3. **Claude on the compatibility list** — one summary asserted "OpenAI Codex, Claude (Anthropic)".
   The source array contains no Anthropic entry, and Claude Code's docs say it reads CLAUDE.md, not
   AGENTS.md. **Not on the list.**
4. **`openai/agents.md` as the canonical repo** — it is `agentsmd/agents.md`. No rename event found;
   noted as current location, not as history.
5. **"Models can follow ~150–200 instructions; Claude Code's system prompt uses ~50"** — circulates
   widely, **no primary source found.** The direction is supported by IFScale; the numbers are folklore.
6. **Negative-instruction backfire** — folklore, and in mild tension with Anthropic's own docs, which
   list `"never do X" rules` as appropriate CLAUDE.md content. Handled in R14 as "prefer positive
   where a positive form exists", not "purge negatives".
7. **Cost direction** — ETH measures +20% inference cost, Lulla measures −16.6% output tokens.
   Unresolved; plausibly reasoning tokens on benchmark tasks vs. output tokens on real PRs.

## Explicitly unverified

- Windsurf's 12,000-char cap **as applied to AGENTS.md itself** — inference from "Processed by the
  same Rules engine"; no page states it.
- Roo Code nested support — present in source (`enableSubfolderRules`, default false), contradicted
  by the docs.
- Phoenix *reads* AGENTS.md — it demonstrably *writes* one via `phx.new`; reading behaviour undocumented.
- Jules' nesting/merge behaviour — claims circulating in blog posts are not vendor-documented.
- Lulla et al.'s exact percentages — from the abstract, not a PDF fetch.
- Devin cloud vs Devin CLI diverge sharply; do not generalise CLI behaviour to cloud sessions.
