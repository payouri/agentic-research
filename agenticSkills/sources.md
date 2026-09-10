# Agent Skills source lexicon

Every source behind [guide.md](guide.md) and [rulebook.md](rulebook.md), keyed for citation.
Gathered 2026-09-10.

Trust tiers: **P1** primary spec/vendor documentation · **P2** peer-reviewed or arXiv research ·
**P3** vendor engineering blog / industry research with method · **P4** practitioner report with
measurement · **P5** opinion, anecdote, or unverified secondary.

---

## 1. The specification

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `spec` | [agentskills.io/specification](https://agentskills.io/specification) | P1 | live, fetched 2026-09-10 | The six frontmatter fields, their caps, the three disclosure tiers, the directory convention. **There is a real spec** — unlike AGENTS.md |
| `spec-repo` | [agentskills/agentskills](https://github.com/agentskills/agentskills) | P1 | created 2025-12-16, pushed 2026-08-09 | Apache-2.0. **No JSON Schema exists** — verified by full tree enumeration. No RFC-2119 keywords section |
| `spec-validator` | [`skills-ref/src/skills_ref/validator.py`](https://raw.githubusercontent.com/agentskills/agentskills/main/skills-ref/src/skills_ref/validator.py) | P1 | fetched 2026-09-10 | `MAX_SKILL_NAME_LENGTH = 64`, `MAX_DESCRIPTION_LENGTH = 1024`, closed `ALLOWED_FIELDS` set. Charset via `str.isalnum()` — **any Unicode alphanumeric**, wider than the spec's stated `a-z, 0-9` |
| `spec-parser` | `skills-ref/src/skills_ref/parser.py` | P1 | fetched 2026-09-10 | Accepts `skill.md` lowercase, contradicting "a file named exactly `SKILL.md`" |
| `spec-prompt` | `skills-ref/src/skills_ref/prompt.py` | P1 | fetched 2026-09-10 | The `<available_skills>` injection format, HTML-escaped — why XML tags are forbidden in name/description |
| `spec-contributing` | [CONTRIBUTING.md](https://raw.githubusercontent.com/agentskills/agentskills/main/CONTRIBUTING.md) | P1 | fetched 2026-09-10 | "Logo requests are reviewed by the Anthropic team"; "When in doubt, leave it out" |
| `spec-impl-guide` | [How to add skills support to your agent](https://agentskills.io/client-implementation/adding-skills-support) | P1 | fetched 2026-09-10 | `.agents/skills/` as the interop path; "project-level skills override user-level skills"; lenient validation ("warn, load anyway") |
| `spec-best-practices` | [Best practices for skill creators](https://agentskills.io/skill-creation/best-practices) | P1 | fetched 2026-09-10 | "teach the agent how to approach a class of problems, not what to produce for a specific instance" |
| `spec-descriptions` | [Optimizing skill descriptions](https://agentskills.io/skill-creation/optimizing-descriptions) | P1 | fetched 2026-09-10 | "**Use imperative phrasing**" — contradicts Anthropic's third-person mandate |
| `spec-clients` | [Client Showcase](https://agentskills.io/clients) | P1 | fetched 2026-09-10 | The official adopter list. **Carries at least one dead entry** (Roo Code) and rotted links (Amp, goose) |
| `anthropic-spec-stub` | [`anthropics/skills/spec/agent-skills-spec.md`](https://raw.githubusercontent.com/anthropics/skills/main/spec/agent-skills-spec.md) | P1 | fetched 2026-09-10 | 87 bytes: "The spec is now located at <https://agentskills.io/specification>". Not a 404 — a deliberate relocation many secondary sources still miss |
| `lf-aaif` | [LF announces formation of the Agentic AI Foundation](https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation) | P1 | 2025-12-09 | Inaugural projects are MCP, goose, AGENTS.md. **Agent Skills is absent** |

## 2. Anthropic product documentation

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `anthropic-overview` | [Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) | P1 | fetched 2026-09-10 | Three levels; "No practical limit on bundled content"; "Treat like installing software"; no XML tags; reserved words |
| `anthropic-best-practices` | [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) | P1 | fetched 2026-09-10 | "**Always write in third person**"; degrees of freedom; gerund naming; one-level references; TOC over 100 lines |
| `anthropic-api-skills` | [Using Agent Skills with the API](https://platform.claude.com/docs/en/build-with-claude/skills-guide) | P1 | fetched 2026-09-10 | `/v1/skills`; 20 skills per request; 30 MB uncompressed; code-execution requirement; no network access |
| `anthropic-managed-agents` | [Skills (Managed Agents)](https://platform.claude.com/docs/en/managed-agents/skills) | P1 | fetched 2026-09-10 | **500 skills per session** — 25× the Messages API limit; repo `.claude/skills/` scanned one level deep; "a mounted repository is part of your agent's trust boundary" |
| `anthropic-enterprise` | [Skills for enterprise](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/enterprise) | P1 | fetched 2026-09-10 | Content scanning is opt-in and "doesn't cover the Claude API"; "limit the number of Skills loaded simultaneously" with no number given |
| `anthropic-engineering` | [Equipping agents for the real world with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) | P3 | pub. 2025-10-16, upd. 2025-12-18 | **Contains no quantitative evaluation of any kind** — verified by fetching the full page. The efficiency claim is argued by analogy, never measured |
| `anthropic-skill-creator` | [`skills/skill-creator/SKILL.md`](https://raw.githubusercontent.com/anthropics/skills/main/skills/skill-creator/SKILL.md) | P1 | fetched 2026-09-10 | 485 lines. "Claude has a tendency to 'undertrigger' skills… make the skill descriptions a little bit 'pushy'"; "All 'when to use' info goes here, not in the body"; the Principle of Lack of Surprise |
| `anthropic-skill-creator-blog` | [Improving skill-creator: test, measure, refine](https://claude.com/blog/improving-skill-creator-test-measure-and-refine-agent-skills) | P3 | 2026-03-03 | The eval harness: subagent per test case, `benchmark.json`, blind A/B, description tuning |
| `anthropic-skills-pdf` | [`skills/pdf/SKILL.md`](https://raw.githubusercontent.com/anthropics/skills/main/skills/pdf/SKILL.md) | P1 | fetched 2026-09-10 | Conditional routing exemplar: "If you need to fill out a PDF form, read FORMS.md" |
| `cc-skills` | [Extend Claude with skills](https://code.claude.com/docs/en/skills) | P1 | fetched 2026-09-10 | ~20 frontmatter fields; the seven search paths; **`.agents/skills/` absent**; personal-over-project precedence; 1,536-char truncation; `allowed-tools` grants but never restricts; "Workspace trust doesn't gate this field"; `/skill-doctor` |
| `cc-sdk-skills` | [Extend agents with skills (Agent SDK)](https://code.claude.com/docs/en/agent-sdk/skills) | P1 | fetched 2026-09-10 | "The SDK doesn't provide a programmatic API for registering them"; `settingSources: []` silently loads zero skills |
| `cc-settings` | [Settings reference](https://code.claude.com/docs/en/settings-reference) | P1 | fetched 2026-09-10 | `skillOverrides`, `disableBundledSkills`, `disableSkillShellExecution` |
| `anthropic-release-notes` | [API release notes](https://platform.claude.com/docs/en/release-notes/api) | P1 | entries 2026-08-19, 2026-08-27 | Skills API **out of beta 2026-08-19**; `BetaSkill` → `BetaContainerSkill` |
| `support-provision` | [Provision and manage skills for your organization](https://support.claude.com/en/articles/13119606) | P1 | fetched 2026-09-10 | Org-wide provisioning on Team/Enterprise — **contradicts the platform overview's "no centralized admin management"** |
| `anthropic-gov-desktop` | [Skills in Claude Desktop (Government)](https://claude.com/docs/government/desktop/skills) | P1 | fetched 2026-09-10 | Text-only admin distribution; "a plugin upload that contains them [scripts, binaries] is rejected" |

## 3. Implementations

Verified from vendor documentation or source. Full comparison in [guide.md](guide.md) §1–2.

| Key | Source | Tier | Reads `.agents/skills/` | `allowed-tools` |
|---|---|---|---|---|
| `zed` | [zed-industries/zed `docs/src/ai/skills.md`](https://zed.dev/docs/ai/skills) (commit 2026-09-07) | P1 | **Only** `.agents/skills/` | **"We parse the field but don't honor it"** |
| `codex` | [openai/codex `codex-rs/skills/src/parser.rs`](https://learn.chatgpt.com/docs/build-skills) | P1 | Yes | Ignored |
| `gemini-cli` | [google-gemini/gemini-cli `skillLoader.ts`](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/skills.md) | P1 | Yes, and it outranks `.gemini/` | Ignored |
| `copilot-vscode` | [Use Agent Skills in VS Code](https://code.visualstudio.com/docs/copilot/customization/agent-skills) | P1 | Yes | — (page silent) |
| `github-docs` | [About agent skills](https://docs.github.com/copilot/concepts/agents/about-agent-skills) | P1 | Yes | **The only surveyed implementer that acts on it** — as pre-approval |
| `cursor` | [Agent Skills](https://cursor.com/docs/skills) | P1 | Yes, recursive | Absent from schema |
| `windsurf` | [Skills — Cascade](https://docs.devin.ai/desktop/cascade/skills) | P1 | Yes | Absent |
| `opencode` | [sst/opencode `src/skill/index.ts`](https://opencode.ai/docs/skills) | P1 | Yes, nested | Ignored; gating via `Permission.evaluate` |
| `crush` | [charmbracelet/crush `internal/skills/skills.go`](https://github.com/charmbracelet/crush) | P1 | Yes, recursive | Ignored |
| `goose` | [block/goose `crates/goose/src/skills/mod.rs`](https://goose-docs.ai) | P1 | Yes, recursive | Ignored |
| `cline` | [cline/cline `skill-directories.ts`](https://docs.cline.bot/customization/skills) | P1 | Yes, one level | Parsed; **unknown fields are a hard parse error** |
| `kilo` | [Kilo-Org/kilocode `packages/core/src/skill.ts`](https://kilo.ai/docs/customize/skills) | P1 | Yes, nested | Ignored |
| `amp` | [Skills](https://ampcode.com/docs/customize/skills) | P1 | Yes | Absent; closed source |
| `roo` | [Roo-Code `SkillsManager.ts`](https://roocodeinc.github.io/Roo-Code/features/skills) | P1 | Yes | Ignored. **Repo archived; "shut down on May 15th"** — yet still listed on the official showcase |
| `junie` | [Agent Skills](https://junie.jetbrains.com/docs/agent-skills.html) | P1 | Yes | Absent |
| `openhands` | [Skills](https://docs.openhands.dev/overview/skills) | P1 | Yes | Absent |
| `factory` | [Skills](https://docs.factory.ai/cli/configuration/skills) | P1 | Yes | Declarative only — **"not a runtime sandbox"** |
| `snowflake` | [Cortex Code extensibility](https://docs.snowflake.com/en/user-guide/cortex-code/extensibility) | P1 | No (`.cortex/`, `.claude/`) | Renamed **`tools:`** |
| `kiro` | [Skills](https://kiro.dev/docs/skills) | P1 | No (`.kiro/` only) | Absent |
| `mistral-vibe` | [mistralai/mistral-vibe README](https://github.com/mistralai/mistral-vibe) | P1 | Yes | Declared as a **YAML list**, not a space-separated string |
| `ms-agent-framework` | [Agent Skills](https://learn.microsoft.com/en-us/agent-framework/agents/skills) | P1 | No fixed paths — passed in code | "Experimental — support may vary" |
| `antigravity` | [Skills](https://antigravity.google/docs/ide/skills) | P1 | Yes | Not documented |
| `letta` | [Skills](https://docs.letta.com/letta-code/skills) | P1 | Yes | Unverified |
| `spring-ai` | [Spring AI generic agent skills](https://spring.io/blog/2026/01/13/spring-ai-generic-agent-skills) | P3 | Configured dirs | Not mentioned |
| `databricks` | [Skills](https://docs.databricks.com/aws/en/assistant/skills) | P1 | No (`.assistant/`) | Absent |
| `aider-negative` | Aider-AI/aider @ 5dc9490 (2026-05-22), full-tree grep | P1 | **No — zero occurrences of `SKILL.md`** | — |
| `npx-skills` | [vercel-labs/skills](https://github.com/vercel-labs/skills) | P1 | Installer, not a loader — symlinks 83 agent directories | Passes through untouched |
| `vercel-authoring-skills` | [`vercel/next.js .agents/skills/authoring-skills/SKILL.md`](https://github.com/vercel/next.js) | P1 | — | Vendor meta-skill: when to use a skill vs `AGENTS.md` |

## 4. Corpus

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `corpus` | **61 SKILL.md files**, all fetched from `raw.githubusercontent.com` and counted by script | P1 | 2026-09-10 | Median 161 lines, median description 317 chars, `allowed-tools` in 1/61, "When NOT to use" as a body section in 1/61, 25% progressive-disclosure shaped, 21% ship executable code |

Composition: `anthropics/skills` (20, complete census) · `anthropics/claude-code` plugins (10,
complete census) · `obra/superpowers` (6 of 14) · `addyosmani/agent-skills` (4 of 25) ·
`wshobson/agents` (3 of **183**) · `vercel/next.js` (5) · `vercel/ai` (1) · `supabase/supabase` (5) ·
`getsentry/sentry` (4) · `getsentry/sentry-javascript` (1) · `elastic/kibana` (2).

**Stated sampling bias:** the corpus is 49% Anthropic and drawn from high-visibility repos. It
describes *notable* skills, not the average skill on GitHub — for which see `skills138k`.

**Exemplars:** `supabase/copywriting` (8 lines — a description plus a pointer) ·
`vercel/react-vendoring` (72 lines, file-name triggers) · `anthropics/pdf` (conditional routing) ·
`obra/test-driven-development` (pre-empts the agent's own rationalizations) ·
`getsentry/analytics` (triggers in user voice) · `vercel/authoring-skills` (the skill/AGENTS.md line).

**Cautionary:** `getsentry/design-system` — **626 lines, zero bundles, 169-char description** ·
`wshobson/api-design-principles` — teaches HTTP verb semantics to a model that knows them ·
`supabase/explorer` — trigger stranded in the body where it cannot fire ·
`anthropics/skills/template` — shipped placeholder description ·
`addyosmani/*` — four monoliths, median 352 lines, nothing bundled.

**Path convention observed:** `.agents/skills/` at Vercel, Supabase, Sentry and part of Elastic;
`.claude/skills/` deeper in some of the same repos. Migration in progress, not settled.

## 5. Evidence — direct measurement of skills

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `skillsbench` | [SkillsBench](https://arxiv.org/abs/2602.12670) (arXiv:2602.12670) | P2 | v1 2026-02-13, v4 2026-06-14 | "Curated Skills raise average pass rate by 16.2 percentage points(pp)… **+4.5pp for Software Engineering** to +51.9pp for Healthcare… 16 of 84 tasks show negative deltas." Self-generated: **−1.3pp**. "Focused Skills with 2--3 modules outperform comprehensive documentation" |
| `skilljuror` | [SkillJuror](https://arxiv.org/html/2606.11543v1) (arXiv:2606.11543) | P2 | 2026-06-11 | **The only clean ablation of progressive disclosure**, holding knowledge fixed: 46.1% vs 42.0% pass, "+4.1%" over flat. Resources touched per trajectory 1.18 → 3.85 |
| `skillaudit` | [SkillAudit](https://arxiv.org/html/2606.22613v1) (arXiv:2606.22613) | P2 | 2026-06-21 | 226 real packages. Utility +0.183; **efficiency −0.186**; "over 7% of skills are at risky status"; utility vs backbone strength "**strongly negatively correlated (r=−0.90)**" |
| `aces` | [ACES](https://arxiv.org/html/2608.20614) (arXiv:2608.20614), NVIDIA | P2 | 2026-08-20 | 947 paired cases, mean Skill Lift **0.2134 [0.1967, 0.2301]**; **87 negative-lift cases "invisible to scanning methods"**; structural vs judge score **Spearman ρ=0.14** |
| `skills138k` | [What Keeps Agent Skills from Being Reusable?](https://arxiv.org/html/2608.08453v1) (arXiv:2608.08453) | P2 | 2026-08-09 | 138,133 files / 20,556 repos: "**89.3% violate the official specification and 91.8% contain at least one detected reusability defect**". Missing trigger guidance **52.3%**. hit@1 88.5% vs 82.6%. AI-generated skills: **38% higher defect rate** |
| `openskilleval` | [OpenSkillEval](https://arxiv.org/pdf/2605.23657) (arXiv:2605.23657) | P2 | v2 2026-05-29 | Documents both over- and under-triggering across marketplaces. **Rates were not extractable from the fetch** — see Unverified |
| `safe-skills-collide` | [When Safe Skills Collide](https://arxiv.org/pdf/2606.00448) (arXiv:2606.00448) | P2 | 2026-06-02 | Individually-safe skills composing into harmful outcomes. **Qualitative only from the fetch; no number asserted** |
| `skills-survey` | [Agent Skills for LLMs: Architecture, Acquisition, Security](https://arxiv.org/html/2602.12430v4) (arXiv:2602.12430) | P2 | v4 2026-06-02 | Survey. Several of its numbers are second-hand — see Unverified |

## 6. Evidence — mechanism, failure modes, security

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `compliance-trap` | [The Compliance Trap](https://arxiv.org/html/2607.10608) (arXiv:2607.10608) | P2 | 2026-07-12 | "**RCR is high and approximately scale-independent (63–72%)**"; divergence by step 7 (0.86–0.94); recovery 7–15%; **Damage Per Compliance −2.1pp to −25.5pp, worse on stronger models** |
| `routing-scale` | [Scaling Enterprise Agent Routing](https://arxiv.org/html/2606.17519) (arXiv:2606.17519), Grammarly/Superhuman | P2 | 2026-06-16 | 110 agents / 584 tools: "Routing F1 on under-specified requests drops **16–23 percentage points**". **≥10pp confusion gap unrecoverable even with perfect retrieval**; oracle ceiling 79% → 69% |
| `samecaprisk` | [Right Family, Wrong Skill / SameCapRisk-Bench](https://arxiv.org/html/2606.10388v2) (arXiv:2606.10388) | P2 | v2 2026-08-21 | Recall@3 0.848–0.888 with **harmful-sibling rate 0.346–0.372**; family-aware selection → **0.007**. "strong helpful ranking does not imply low exposure". **Retitled twice — cite by ID** |
| `after` | [Managing Procedural Memory in LLM Agents](https://arxiv.org/html/2606.23127v1) (arXiv:2606.23127) | P2 | 2026-06-22 | Static skills ~**+2.8 points** (vs SkillsBench's +16pp — different benchmark); weaker models gain more (Gemma +14.2 vs GPT 5.4 +3.1); **cross-role transfer −4.8 to −7.5 points**; refinement +3.7 to +6.7 |
| `ifscale` | [How Many Instructions Can LLMs Follow at Once?](https://arxiv.org/abs/2507.11538) (arXiv:2507.11538) | P2 | 2025-07-15 | "**Even the best frontier models only achieve 68% accuracy at the max density of 500 instructions**"; bias toward earlier instructions |
| `lost-middle` | [Lost in the Middle](https://aclanthology.org/2024.tacl-1.9/) | P2 | TACL vol. 12 (2024) | Retrieval degrades with position; worst in the middle. **Qualitative claim verified from abstract; the widely-quoted 20–30pt magnitude was not verified this session** |
| `context-rot` | [Context Rot](https://www.trychroma.com/research/context-rot) | P3 | 2025-07-14 | 18 models: "performance grows increasingly unreliable as input length grows"; focused ~300-token prompts beat ~113k-token ones. **Vendor-authored (Chroma); not peer-reviewed**. `research.trychroma.com` 301s here |
| `skills-wild` | [Agent Skills in the Wild](https://arxiv.org/abs/2601.10338v1) (arXiv:2601.10338) | P2 | 2026-01-15 | 31,132 skills scanned: **26.1% ≥1 vulnerability**; 13.3% exfiltration; 11.8% privilege escalation; **5.2% high-severity/likely-malicious**; "**Skills bundling executable scripts are 2.12× more likely to contain vulnerabilities**" |
| `payload-less` | [Exploiting LLM Agent Supply Chains via Payload-less Skills](https://arxiv.org/html/2605.14460v1) (arXiv:2605.14460) | P2 | 2026-05-14 | **77.67% leakage, 67.33% RCE, 0.00% detection** — against gateways catching 91–99.81% of conventional attacks |
| `poisoned-skills` | [Supply-Chain Poisoning Attacks](https://arxiv.org/html/2604.03081v1) (arXiv:2604.03081) | P2 | 2026-04-03 | 1,070 adversarial skills: **11.6–33.5% bypass**. Harness matters: Claude Code + Sonnet 4.6 at 2.3% vs OpenHands + GLM-4.7 at 27.1%. Static analysis stops 90.7%; **2.5% evade both detection and alignment** |
| `threat-taxonomy` | [Towards Secure Agent Skills](https://arxiv.org/html/2604.02837v1) (arXiv:2604.02837) | P2 | 2026-04-03 | 7 threat categories, 3 layers, 17 scenarios across Creation → Distribution → Deployment → Execution |
| `clawhavoc` | [ClawHavoc](https://www.antiy.net/p/clawhavoc-analysis-of-large-scale-poisoning-campaign-targeting-the-openclaw-skill-market-for-ai-agents/), Antiy CERT | P4 | 2026-02-06 | "at least **1,184 malicious Skills**" on ClawHub, 12 author IDs, **386 in one day**. **OpenClaw's marketplace, not Anthropic's.** Koi Security's original disclosure is now a dead URL |

---

## Conflicts resolved during this research

1. **`claude plugin eval` — could not be confirmed to exist.** Two axes hit this independently: the
   Evidence agent found the official docs describe the `skill-creator` plugin workflow instead, and
   the Implementations agent found `code.claude.com/docs/en/plugin-eval` returns **404**. Every hit
   was third-party. **Recorded as unverified; the documented eval surface is the `skill-creator`
   plugin.**

2. **`allowed-tools` semantics — four-way convergence.** The spec marks it "(Experimental)";
   Claude Code documents that it "does not restrict which tools are available"; Zed says "We parse
   the field but don't honor it"; the corpus finds it in **1 of 61** files. **Resolved: it is a
   pre-approval grant, never a sandbox, on every implementation surveyed.** The corpus's 1.6% usage
   rate is the field voting correctly.

3. **Precedence — the spec is contradicted by its own originator.** The implementer guide states
   "The universal convention across existing implementations: project-level skills override
   user-level skills." Claude Code documents the opposite (personal over project); Cline documents a
   third ordering (enterprise > global > project). **Resolved: the "universal convention" is not
   universal. Do not depend on precedence.** Structurally identical to the AGENTS.md dossier's
   "nearest file wins" finding.

4. **`.agents/skills/` vs `.claude/skills/` — three axes agree.** The spec recommends `.agents/`;
   Implementations confirms Claude Code does not read it while a dozen others do; the Corpus finds
   Vercel, Supabase, Sentry and Elastic already shipping from `.agents/skills/`. **Resolved:
   portability is one-directional, and writing only to `.claude/skills/` is now the less portable
   choice** — inverting most published advice.

5. **The description-voice contradiction, settled by measurement.** Anthropic mandates third person
   in a warning block; agentskills.io mandates imperative phrasing in its first bullet.
   **Resolved by the corpus:** the strongest real descriptions, Anthropic's own included, are *both* —
   a third-person topic clause plus an imperative trigger clause plus negative scope. Neither page is
   correct alone.

6. **Corpus cleanliness vs the 89.3% violation rate.** The 61-file corpus is 100% compliant on
   required fields; the 138K-file study finds 89.3% non-compliance. **Not a contradiction — different
   sampling depths.** The corpus agent declared its bias toward high-visibility repos before this
   number was in view. Both are reported, with their populations named.

7. **Two independent measurements that self-authored skills fail.** SkillsBench measures **−1.3pp**
   by benchmark; the 138K study measures **+38% defect rate** by static analysis. Different methods,
   same direction. **Resolved as one of the better-supported claims in the dossier.**

8. **Anthropic's docs contradict Anthropic's docs, twice.** (a) The platform overview says "Custom
   Skills do not sync across surfaces", yet Claude Code documents `~/.claude/skills/synced/`, a
   `claude.ai account` path row, and `CLAUDE_CODE_SYNC_SKILLS=1`. (b) The overview says claude.ai
   "does not support centralized admin management", yet the help centre documents org-wide
   provisioning on Team and Enterprise. **Resolved in favour of the more specific, more recently
   maintained pages; the overview's claims are stale.** Plan availability also disagrees (Pro/Max/
   Team/Enterprise vs Free/Pro/Max/Team/Enterprise) and is left open.

9. **The old Anthropic spec path is a stub, not a 404.** `anthropics/skills/spec/agent-skills-spec.md`
   returns 87 bytes pointing at agentskills.io. **Recorded because much secondary writing still cites
   it as the specification.**

10. **A hallucination-shaped claim caught by a negative check.** SEO aggregator lists of
    "27+ compatible agents" include **Aider**; a full-tree grep of `Aider-AI/aider` @ 5dc9490 returns
    **zero occurrences of `SKILL.md`**. **Resolved: Aider does not support skills.** No aggregator was
    used as evidence anywhere in this dossier.

11. **The official Client Showcase carries a dead entry.** Roo Code is listed, but the repo is
    archived read-only with "The Roo Code Extension was shut down on May 15th"; the Amp and goose
    links rot to a missing anchor and a 404. **Recorded: the canonical adopter list is not maintained
    to the standard of the spec itself.**

12. **SkillsBench's own headline moved between versions.** v1: +16.2pp on a 24.3% baseline, 84 tasks,
    7 configurations. v4: "from 33.9% to 50.5% (+16.6 percentage points)", 87 tasks, 18
    configurations. **The delta is stable; the baseline and scope are not. Cite the version.**

13. **Anthropic's own `claude-api` skill exceeds the spec's description cap.** The corpus measured its
    description at **1,071 characters** against a documented maximum of 1024. **Recorded as evidence
    that the cap is unenforced outside the upload boundary**, consistent with the reference
    validator's lint-not-gate behaviour.

## Explicitly unresolved

- **Cost direction.** SkillAudit measures efficiency at **−0.186** (skills cost more); SkillJuror
  measures tokens-per-pass falling **~35%**. **Not averaged.** The plausible reason — untested — is
  different denominators: raw consumption vs cost per success, where avoided retries dominate.
- **Effect size for static skills.** SkillsBench says +16pp; the procedural-memory work says
  ~+2.8 points. Different benchmarks, different task mixes. Both stand.
- **Plan availability for claude.ai Skills** — platform docs and help centre disagree on whether Free
  is included.

## Explicitly unverified

- **`claude plugin eval`** — no official documentation page; the expected URL 404s. Existence and
  flags unconfirmed.
- **Agent Skills' foundation status.** "Not donated" rests on the 2025-12-09 LF press release plus
  the absence of any foundation mention in the spec repo. `aaif.dev` was unreachable (DNS timeout).
  **No evidence of donation is not proof of none.**
- **The "98,380 skills / 157 confirmed malicious / 54.1% one actor" figures.** Cited inside
  arXiv:2602.12430 as "Liu et al. (2026b)" and attributed inconsistently elsewhere. **No primary
  found.** Do not cite.
- **ClawHavoc's "247,000 installations" and "$2.3M stolen".** Only on low-trust aggregators. Antiy —
  the strongest surviving primary — reports 14,285 downloads for one subset. Koi Security's original
  disclosure now 301s to an unrelated vendor page. **Unsubstantiated.**
- **Lost in the Middle's "20–30 point" magnitude** and **IFScale's "near-perfect through ~150
  instructions"** — both from search summaries, not the fetched papers. The qualitative claims and
  the 68%-at-500 figure are verified.
- **OpenSkillEval's over/under-trigger rates** and **"When Safe Skills Collide" numbers** — the papers
  are on-topic but no figures were extractable from the fetches. No number asserted from either.
- **A direct curve of trigger accuracy vs number of installed skills.** Does not exist for SKILL.md.
  The catalogue-degradation claim is **strongly inferred** from tool-routing research, not measured
  on skills.
- **Positional degradation of the skill listing itself.** The context-rot rationale for progressive
  disclosure is inference; SkillJuror supports the conclusion on independent grounds.
- **CVE details** (CVE-2026-24887, -35021, -39861) — from search summaries only; no advisory fetched.
  Confirm against NVD before citing.
- **`allowed-tools` behaviour in Cursor, Windsurf, Antigravity, Junie, OpenHands, Kiro, Databricks,
  Letta** — absent from documented schemas, mostly closed source.
- **Precedence rules in Cursor, Copilot, Windsurf, Antigravity, Letta, Spring AI, Databricks** — not
  documented anywhere reachable.
- **Claude in Slack / Claude Tag as a skills surface** — no Anthropic documentation connects them.
- **~35 further tools** appearing on the official showcase or in `npx skills`' 83-target path table.
  **An installer path proves where files are written, not that the agent parses them.** None are
  claimed as compatible here.
- **GitHub star counts.** Two agents independently fetched the same implausible-looking figures
  (`anthropics/skills` ~175.6k, `obra/superpowers` ~284.6k). Reported as the API returned them,
  **disputed, and load-bearing on nothing.**
