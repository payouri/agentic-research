# Building agentic skills — the guide

Researched 2026-09-10 from primary sources. Companion files: [rulebook.md](rulebook.md)
(the checkable rules) and [sources.md](sources.md) (every source, annotated).

---

## 1. What the standard actually is

Unlike `AGENTS.md`, **Agent Skills has a real specification.** It lives at
[agentskills.io/specification](https://agentskills.io/specification), with source in
`agentskills/agentskills` (`docs/specification.mdx`), Apache-2.0, repo created **2025-12-16**. It
defines six frontmatter fields, three progressive-disclosure tiers, and a directory convention. There
is a **reference validator** in Python (`skills-ref`) you can run against a skill.

Anthropic's own former spec path is now an 87-byte stub:

> `# Agent Skills Spec`
> `The spec is now located at <https://agentskills.io/specification>`

That relocation matters, because a great deal of secondary writing still cites the Anthropic path as
*the* spec. It is not; it is a signpost.

**What the spec is not: neutrally governed.** It has *not* been donated to a foundation. The Linux
Foundation's Agentic AI Foundation announcement (2025-12-09) names MCP, goose and AGENTS.md —
Agent Skills is absent. `CONTRIBUTING.md` says "Logo requests are reviewed by the Anthropic team."
The governance model is a well-run benevolent dictatorship with an open PR surface, and the
contribution bar is deliberately high:

> "We maintain a high bar for additions to the spec — it is much easier to add things to a
> specification than to remove them. Every new feature adds complexity that all implementers must
> understand and support. When in doubt, leave it out."

There is **no JSON Schema** — verified by enumerating every blob in the repo — and no RFC-2119
normative-keywords section. The normative artefact is a prose page plus a validator.

### The gap between having a spec and having a standard

Here is the finding that shapes everything below. **The spec defines six fields. Claude Code
supports about twenty. And Claude Code is one of the very few clients that does not read the
directory the spec recommends for cross-client interoperability.**

The specification's own implementer guide recommends scanning `.agents/skills/` and
`~/.agents/skills/`, mentioning the Anthropic path only as an afterthought:

> "Some implementations **also** scan `.claude/skills/` (both project-level and user-level) **for
> pragmatic compatibility**, since many existing skills are installed there."

Claude Code's documented search paths are `<managed>/.claude/skills/`, `~/.claude/skills/`,
`.claude/skills/`, `<subdir>/.claude/skills/`, `--add-dir` paths, `<plugin>/skills/`, and
`~/.claude/skills/synced/`. `.agents/skills/` is **not among them**.

Meanwhile Cursor, Copilot/VS Code, Codex, Gemini CLI, OpenCode, Crush, goose, Cline, Kilo, Amp,
Zed, Junie and others read `.agents/skills/` — and most of them *also* read `.claude/skills/` for
compatibility. **So portability is one-directional: the rest of the ecosystem will pick up your
Claude Code skills; Claude Code will not pick up theirs.** Zed declines the compatibility scan on
principle: "We do not also scan tool-specific directories that other agent tools sometimes use."

The corpus confirms the field is already voting. Vercel (`next.js`), Supabase, Sentry and Elastic
all ship skills from `.agents/skills/`; several of those repos carry *both* `.agents/skills/` at the
root and `.claude/skills/` deeper in the tree. That is a migration in progress, not a settled
convention.

**Practical consequence:** if you want a skill to work everywhere, put it in `.agents/skills/` and
symlink or copy it to `.claude/skills/`. Writing only to `.claude/skills/` is the *less* portable
choice today, which inverts the advice most blog posts still give.

---

## 2. The six fields, and the fourteen that are not

The spec's complete normative surface:

| Field | Required | Constraint |
|---|---|---|
| `name` | **Yes** | ≤64 chars, lowercase alphanumerics and hyphens, no leading/trailing/consecutive hyphens, **must match the parent directory name** |
| `description` | **Yes** | ≤1024 chars, non-empty |
| `license` | No | License name or reference |
| `compatibility` | No | ≤500 chars, environment requirements |
| `metadata` | No | String→string map |
| `allowed-tools` | No | Space-separated tool list — **"(Experimental)"** |

There is **no `version` field and no `model` field** in the specification. Version is carried inside
`metadata` by convention, per the spec's own example (`metadata: {author: …, version: "1.0"}`).

Three things the prose does not tell you, read out of `skills-ref` source:

- **No regex is used.** The charset check is `all(c.isalnum() or c == "-" for c in name)` — Python's
  `str.isalnum()`, which accepts **any Unicode alphanumeric**. The docstring is explicit: "Skill
  names support i18n characters (Unicode letters) plus hyphens." So the spec's stated `(a-z, 0-9)`
  is *narrower than its own reference implementation*.
- **`skill.md` lowercase is accepted** by the parser (`("SKILL.md", "skill.md")`), despite the spec
  and the implementer guide both saying "a file named exactly `SKILL.md`".
- **Unknown fields are a validation error** in the reference validator — a *closed* field set — yet
  `validate()` returns a list of strings rather than raising. It is a lint, not a gate.

And then Claude Code defines roughly twenty fields: `when_to_use`, `disable-model-invocation`,
`user-invocable`, `disallowed-tools`, `model`, `effort`, `argument-hint`, `arguments`, `context`,
`agent`, `background`, `hooks`, `paths`, `shell`, plus the spec six — of which `license` and
`compatibility` are documented as "Claude Code accepts but doesn't use." Claude Code also does not
require `name` at all; the directory name is the command.

**This is where portability actually breaks, and it breaks asymmetrically.** Zed: "Unknown fields
are silently ignored." Codex, Gemini CLI, goose, Crush, Kilo: parse two or three fields and drop the
rest. **Cline: unknown fields are a hard parse error.** So a Claude Code skill using `model:` or
`context: fork` loads fine in Zed, degrades silently in Codex, and *fails outright* in Cline.

There is also a live cap disagreement. The spec and the API cap `description` at **1024 characters**
and reject over-length. Claude Code truncates `description` + `when_to_use` combined at **1,536
characters** for the listing. Cline, Roo, Kilo and Crush reject over-1024; OpenCode and goose do not
check at all. **The same skill can load everywhere, load truncated, or be dropped.**

Worth noting, since it shows how loose the enforcement really is: Anthropic's own shipped
`claude-api` skill carries a **1,071-character description** — over the spec's own 1024 cap, and it
ships anyway.

---

## 3. What the evidence supports — and the number Anthropic never published

**Anthropic has published no efficacy measurement for skills.** Its engineering post,
"Equipping agents for the real world with Agent Skills", contains no benchmark, no A/B, no
ablation, and no cost or latency comparison. It argues efficiency by analogy — "sorting a list via
token generation is far more expensive than simply running a sorting algorithm" — without measuring
it. Anthropic ships excellent measurement *tooling* (§8) and publishes no measurement *results*.

Third parties have filled the gap, and as of 2026 there is a real benchmark literature.

**Curated skills work, by roughly +16 percentage points.** SkillsBench (arXiv:2602.12670) ran 7,308
trajectories across Claude Code, Gemini CLI and Codex CLI with deterministic verifiers:

> "Curated Skills raise average pass rate by 16.2 percentage points(pp), but effects vary widely by
> domain (+4.5pp for Software Engineering to +51.9pp for Healthcare) and 16 of 84 tasks show
> negative deltas."

Read the two halves together. The mean is large; **software engineering is the domain where skills
help least**, and roughly one task in five got *worse*. The authors' own hypothesis for the negative
tail is that skills "may introduce conflicting guidance or unnecessary complexity."

Two caveats belong beside that number every time it is quoted. First, **the paper's own figures moved
between versions** — v1 reports +16.2pp on a 24.3% baseline; v4 reports "from 33.9% to 50.5%
(+16.6 percentage points)" with a different task and configuration count. Cite the version. Second,
the authors are explicit about what they measured:

> "Our 84 evaluated tasks with high-quality Skills represent an optimistic scenario. Real-world
> Skill usage involves lower-quality Skills."

That is a ceiling, and §6 shows how far below it the real ecosystem sits.

**Models cannot write the skills they benefit from reading.** Same paper: self-generated skills
scored **−1.3pp against the no-skills baseline** — worse than nothing. Only one model of seven
improved. A completely independent method agrees: in a 138,133-file corpus study
(arXiv:2608.08453), AI-generated skills carry a **38% higher defect rate (3.23 vs 2.34), 2.3× more
safety defects and 2.8× more portability defects**. Two methods, one conclusion: *asking the model to
write its own skill is the one authoring approach measurement actively rules out.*

**Skills substitute for capability more than they add to it.** SkillAudit (arXiv:2606.22613)
measured 226 real-world skill packages and found utility "strongly negatively correlated" with
backbone model strength, **r = −0.90**. The procedural-memory work (arXiv:2606.23127) shows the same
shape from a different angle: Gemma 4 E4B gained +14.2 where GPT 5.4 gained +3.1. SkillsBench puts
it plainly — "smaller models with Skills can match larger models without them." **Expect the value
of your skill library to decline as your model improves.**

**Cost is genuinely contested and I am not averaging it.** SkillAudit measures an efficiency gain of
**−0.186** — skills *cost* time and tokens. SkillJuror (arXiv:2606.11543) measures tokens-per-pass
falling from 0.34M (no skill) to ~0.21M (with skill), a ~35% reduction. The plausible reconciliation
— untested — is different denominators: SkillAudit compares skill vs no-skill on raw consumption,
SkillJuror compares cost *per success*, where fewer failed attempts dominate. Both are on the record
in [sources.md](sources.md) as unresolved.

---

## 4. Progressive disclosure is the one design choice that has been isolated

The spec names three tiers, and every runtime implements the first two the same way:

> 1. **Metadata** (~100 tokens): The `name` and `description` fields are loaded at startup for all skills
> 2. **Instructions** (< 5000 tokens recommended): The full `SKILL.md` body is loaded when the skill is activated
> 3. **Resources** (as needed): Files (e.g. those in `scripts/`, `references/`, or `assets/`) are loaded only when required

Anthropic's framing of the payoff: "**No practical limit on bundled content:** Files don't consume
context until accessed."

SkillJuror is the only study that **holds task knowledge fixed and varies only the organisation**,
which makes it the cleanest evidence in the dossier:

| Metric | No skill | Flat baseline | Progressive disclosure |
|---|---|---|---|
| Pass rate | 29.0% | 42.0% | **46.1%** |
| Minutes per pass | 29.8 | 20.1 | **17.8** |
| Tokens per pass | 0.34M | 0.22M | **0.21M** |
| Cost per pass | $2.05 | $1.28 | $1.31 |

> "distinct Skill resources touched per trajectory rise from 1.18 to 3.85, and effective uptake
> events rise from 1.33 to 3.92. It also yields 17 additional verifier-passing trials out of 410
> matched trials (+4.1%) over the normalized flat baseline."

**+4.1pp for +$0.03 per pass.** And the mechanism is the interesting part — the agent does not just
read more, it *returns* to resources mid-task: "Agents do not only read a larger instruction once,
but return to support resources while implementing, checking, and repairing."

SkillsBench independently found "Focused Skills with 2--3 modules outperform comprehensive
documentation." IFScale (arXiv:2507.11538) supplies the mechanism for why density hurts — "even the
best frontier models only achieve 68% accuracy at the max density of 500 instructions," with a
measured bias toward earlier instructions.

**One honesty note.** The universally-repeated argument that progressive disclosure matters *because
of context rot* is an inference, not a measurement. Context Rot (Chroma) and Lost in the Middle
(TACL 2024) measure retrieval of a fact from a long context. **Nobody has measured positional
degradation of the skill listing itself.** The conclusion is well-supported by SkillJuror on its own
merits; the mechanism story is plausible and untested. Say it that way.

---

## 5. The description is the skill

Everything about discovery runs through one string. The spec's reference implementation shows
exactly what the model sees — `name`, `description`, `location`, HTML-escaped inside
`<available_skills>` — which is *why* Anthropic's platform docs forbid XML tags in either field.

Selection is left to the model: "Most implementations rely on the model's own judgment as the
activation mechanism, rather than implementing harness-side trigger matching or keyword detection."

**Under-triggering is the default failure, and the vendor says so.** From Anthropic's own
`skill-creator`:

> "currently Claude has a tendency to 'undertrigger' skills… please make the skill descriptions a
> little bit 'pushy'."

Its worked example appends "**even if they don't explicitly ask for a 'dashboard.'**" And the rule
that follows from it: "All 'when to use' info goes here, not in the body." The body is not loaded
until triggering has already happened, so a trigger condition written in the body is *structurally
unable* to do its job.

The corpus shows this failure at scale. In the 138K-file study, the single most common defect is
**missing trigger guidance, at 52.3%** — and it has a measured cost: "R1-clean skills achieve 88.5%
hit@1 retrieval vs. 82.6% for defective skills," with **67.0% of skills carrying routing defects
affecting discovery.**

### The voice contradiction, settled by the corpus

Two authorities give directly opposing instructions. Anthropic's best-practices, in a warning block:

> "**Always write in third person**… **Good:** 'Processes Excel files and generates reports'
> **Avoid:** 'You can use this to process Excel files'"

agentskills.io's optimizing-descriptions, first bullet:

> "**Use imperative phrasing.** Frame the description as an instruction to the agent: 'Use this
> skill when…' rather than 'This skill does…'"

**The corpus resolves this, and neither page is right alone.** Measured across 61 real skills, 75%
contain an explicit trigger construction, and the strongest descriptions — including Anthropic's own
— are *both*: a third-person clause naming the topic, then an imperative clause naming the trigger.
Anthropic's `docx` is the archetype at 835 characters:

> "Use this skill whenever the user wants to create, read, edit, or manipulate Word documents (.docx
> files) or Word templates (.dotx files). Triggers include: any mention of 'Word doc', 'word
> document', '.docx'… **Do NOT use for PDFs, spreadsheets, Google Docs**, or general coding tasks
> unrelated to document generation."

Three components, and the third is the one almost everybody omits.

### Negative scope is the highest-value thing you can write

**"When NOT to use" appears as a body section in exactly 1 of 61 measured skills.** But the *good*
skills do express it — in the description, where it actually runs. `xlsx`: "Do NOT trigger when the
primary deliverable is a Word document." Sentry's `django-models`: "Not for Pydantic models,
dataclasses, ML models, or Protobuf."

The evidence says this is not a nicety. SameCapRisk-Bench (arXiv:2606.10388) found retrievers hitting
Recall@3 of 0.848–0.888 while surfacing a *harmful same-capability sibling* in **34.6–37.2%** of
cases — "strong helpful ranking does not imply low exposure." Family-aware selection cut that to
**0.007** for a 1.5pp recall cost. Negative scope in your description is the cheap, author-side
version of that fix.

### Write triggers in the user's vocabulary, not the maintainer's

The single best sentence in the corpus, from Supabase's `safe-sql-execution`:

> "Use whenever code will build, return, fetch, or execute SQL that runs against a user's real
> Postgres database — **even when the request reads like an ordinary feature or bug fix and never
> says 'security,' 'injection,' or 'SafeSqlFragment.'**"

That targets the exact case where the user's words will not contain the skill's words. Sentry's
`analytics` does the same by listing literal user phrasings: "'how many people', 'is anyone
clicking', 'who is using'." Vercel goes further and triggers on **file names** — "Use when editing
`entry-base.ts`, `$$compiled.internal.d.ts`, `compiled/react*` packages" — which makes activation
mechanical rather than semantic. That is the highest-signal pattern in the corpus.

---

## 6. What real skills actually look like

61 SKILL.md files from Anthropic, Vercel, Supabase, Sentry, Elastic and major community collections,
every count computed from fetched bytes:

| Measure | Median | Range |
|---|---|---|
| SKILL.md length | **161 lines** | 6 → 834 |
| Word count | 1,209 | 21 → 11,698 |
| Description length | **317 chars** | 68 → 1,071 |

Length is roughly trimodal — 28% under 100 lines, 36% between 100–299, **36% at 300+**. There is no
settled "short skill" convention, and Anthropic itself ships 400–834-line skills.

**Progressive disclosure is preached more than practised.** Only **25%** of the corpus is genuinely
progressive-disclosure shaped (short body + bundled files). **16% are monoliths** — 250+ lines with
nothing bundled at all. 46% are a single file with nothing beside it. Only **21% ship executable
code**; the rest is prose.

**`allowed-tools` is functionally dead: 1 of 61 files (1.6%).** Which is the correct response to
§9 — it does not do what its name suggests anywhere.

The section-frequency table's interesting end is the bottom:

| Section | Files | % |
|---|---|---|
| Overview / Purpose | 20 | 33% |
| Anti-patterns / Red flags | 14 | 23% |
| Verification / Checklist | 13 | 21% |
| Quick reference | 12 | 20% |
| **Output/report format** | **2** | **3%** |
| **When NOT to use** | **1** | **2%** |

**Exemplar worth copying.** Supabase's `copywriting` — **8 lines**, entire body:

> "Source of truth: `apps/design-system/content/docs/copywriting.mdx`. Read it before writing or
> auditing any UI copy — it covers voice and tone, buttons, error messages, empty states, and
> confirmation dialogs with good/bad examples."

Progressive disclosure at its limit: a description that does all the triggering work and a pointer
that cannot drift from the document it points at. It duplicates nothing.

**Cautionary case.** Sentry's `design-system`: **626 lines, zero bundled files, gated by a
169-character description** — a component reference manual that should be two files behind a short
body, triggered by a string so vague ("Use when implementing UI components, layouts, or typography")
that it will match nearly every frontend task in the repo. Under-triggering and over-triggering at
once.

**And the one that teaches nothing.** `wshobson/agents` `api-design-principles` spends its 110 lines
on "`GET`: Retrieve resources (idempotent, safe)" and "Use plural nouns for collections (`/users`,
not `/user`)". No model needs HTTP verb semantics loaded into context. That repo ships **183**
SKILL.md files; the volume is the signal.

**A positive hygiene result worth stating.** Grepping all 61 files for credentials (`sk-`, `ghp_`,
`api_key:`) returned **zero hits**; absolute paths (`/Users/`, `/home/`, `C:\Users`) returned exactly
**one**, and it was an anti-pattern *warning*, not a bug. Caveat: this corpus is drawn from
high-visibility repos, which is precisely where such mistakes get caught.

---

## 7. The population you are joining is overwhelmingly defective

The corpus above measures *notable* skills. The long tail is a different world. From 138,133 public
SKILL.md files across 20,556 repositories (arXiv:2608.08453):

> "**89.3% violate the official specification and 91.8% contain at least one detected reusability
> defect**"

Mean 2.5 defects per skill, robust across lenient (88.8%) and strict (94.6%) thresholds. This is not
in tension with the clean 61-file corpus — it is the same finding at a different sampling depth, and
the corpus agent flagged its own bias toward notable repos before seeing this number.

**Static quality scores barely predict whether a skill works.** NVIDIA's ACES (arXiv:2608.20614)
scored 145 skills structurally and by LLM-judge rubric and found them correlated at **Spearman
ρ = 0.14**. In 947 paired live cases, mean Skill Lift was **0.2134 [95% CI 0.1967–0.2301]**, positive
in 72.8% of cases — and **87 cases had negative lift**, "invisible to scanning methods."

**Linting your SKILL.md tells you almost nothing about whether it helps.** You have to run it.

---

## 8. Failure modes

**The compliance trap is the one to fear.** When an agent retrieves guidance that is wrong for the
task (arXiv:2607.10608):

> "**RCR is high and approximately scale-independent (63–72% across the five models).**"

Agents adopt task-incorrect guidance roughly two-thirds of the time, *regardless of model strength*.
It happens fast — "P(t_entry≤t) reaches 0.86–0.94 by step 7" — and recovery is poor: conflicting
trajectories re-align only **7–15%** of the time versus 27–42% for helpful ones. Worst of all,
**stronger models lose more**: Damage Per Compliance ranges from −2.1pp on weaker models to −25.5pp
on Qwen3.5-27B, "because higher-baseline models start higher but land on a similar low floor."

**A stale skill is not a no-op. It is a two-thirds-probability trajectory hijack that your best model
is most exposed to.** Combined with a 91.8% ecosystem defect rate, that is the mechanism by which a
large installed skill library becomes actively harmful.

**Skills do not transfer.** The procedural-memory work measured **−4.8 to −7.5 points** when a skill
evolved for one professional role was applied to another. That is the quantified version of "this
skill is not as general as you think."

**Catalogue size degrades selection, but the direct curve does not exist.** Nobody has published
"trigger accuracy vs number of installed skills" for SKILL.md. The nearest proxy is production tool
routing at Grammarly/Superhuman (arXiv:2606.17519), scaling 10 → 110 agents over 584 tools:

> "Routing F1 on under-specified requests drops 16–23 percentage points across models."

Decomposed: a **16pp retrieval gap** (recoverable — embedding shortlisting wins back +10–11pp) and a
**≥10pp confusion gap** that is *unrecoverable even with perfect retrieval*, arising from genuinely
similar options. The oracle ceiling itself falls from 79% to 69% as the catalogue grows.

**Treat "more skills degrade selection" as strongly inferred from adjacent evidence, not directly
measured on skills.** But note the confusion gap: part of the damage cannot be fixed by better
retrieval and must be fixed by making descriptions genuinely non-overlapping — which is author-side
work, i.e. yours.

---

## 9. `allowed-tools` is not a sandbox, on any implementation surveyed

Four axes converged on this independently, so it goes in bold. The spec marks the field
**"(Experimental)"**. Claude Code:

> "The `allowed-tools` field grants permission for the listed tools during the turn that invokes the
> skill… **It does not restrict which tools are available: every tool remains callable.**"

And the part that should worry you:

> "**Workspace trust doesn't gate this field.** Claude Code applies a project skill's `allowed-tools`
> whenever you or Claude invoke the skill, including in a `-p` run in a folder you've never trusted."

Zed states its position outright — "We parse the field but don't honor it." Codex, Gemini CLI,
OpenCode, goose, Crush and Kilo drop it silently. Snowflake renamed it `tools:`. Mistral Vibe takes a
YAML list where the spec says space-separated string. Factory says it plainly: "not a runtime
sandbox." **GitHub Copilot is the only surveyed implementer that acts on it — and only as
pre-approval**, never restriction.

So `allowed-tools` in an untrusted skill is not a limit. It is a **pre-approved capability grant to
whoever wrote the skill.**

### The rest of the security picture

Of 31,132 skills scanned from two marketplaces (arXiv:2601.10338): **26.1% contain at least one
vulnerability**; 13.3% data exfiltration; 11.8% privilege escalation; **5.2% "exhibit high-severity
patterns strongly suggesting malicious intent."** And the single most actionable ratio in the
dossier:

> "**Skills bundling executable scripts are 2.12× more likely to contain vulnerabilities**"

**Scanning does not save you.** Payload-less attacks (arXiv:2605.14460) express harmful objectives as
abstract "compliance guidelines" and let the agent write the malicious code itself — up to **77.67%
complete leakage rate**, **67.33% RCE**, and **0.00% detection**, against gateways that catch
"91-99.81% against traditional attacks." Supply-chain poisoning (arXiv:2604.03081) measured
**11.6–33.5% bypass rates**, with harness choice mattering enormously: Claude Code + Sonnet 4.6 at
2.3% direct execution versus OpenHands + GLM-4.7 at 27.1%.

This is not hypothetical. The ClawHavoc campaign put "at least 1,184 malicious Skills" on OpenClaw's
ClawHub across 12 author IDs, 386 of them deployed on a single day. (That was OpenClaw's marketplace,
not Anthropic's — evidence about marketplaces as a class.)

Anthropic's own guidance is refreshingly blunt: "**Treat like installing software.**" Its documented
controls are `disableSkillShellExecution` (org-level kill switch), enterprise content scanning
(opt-in, and it "doesn't cover the Claude API"), and — for Claude for Government Desktop — text-only
admin-distributed skills where "a plugin upload that contains them [scripts, binaries] is rejected."

---

## 10. How to actually build one

**Start with an eval, not a document.** This is the one point where vendor guidance and academic
practice align exactly: paired with-skill/without-skill runs on the same tasks, multiple trials,
deterministic verifiers. Anthropic's `skill-creator` plugin implements it — spawns a subagent per
test case for clean context, records tokens and duration, writes `benchmark.json` comparing
with-skill against without-skill, runs blind A/B between versions, and generates should-trigger and
should-not-trigger prompts to tune the description. Its own methodology note is the important part:

> "Seeing a skill trigger tells you Claude found it, not that it did what you intended."

Use `/skill-doctor` for the standing cost: "Every skill in the skill listing adds to your context on
every turn, whether or not Claude ever uses it."

**Then, in order:**

1. **Write the description first and last.** It is the entire discovery surface. Topic clause +
   "Use when…" trigger clause + explicit "Do NOT use for…". Put the key use case first — Claude Code
   truncates at 1,536 chars, the spec rejects over 1024. Aim well under both; the corpus median is
   317 characters and the best files sit between 200 and 500.
2. **Name concrete triggers in the user's vocabulary** — file names, literal phrasings, extensions —
   not the maintainer's abstractions.
3. **Be pushy.** Under-triggering is the documented default failure and the most common defect in
   the wild at 52.3%.
4. **Keep the body under ~500 lines and split the rest into `references/`** with conditional routing
   ("If you need to fill out a PDF form, read FORMS.md"), not a flat "see also". +4.1pp, measured.
5. **Delete anything the model already knows.** HTTP verb semantics do not belong in a skill.
6. **Do not have the model write the skill for you.** −1.3pp, and 38% more defects. Have it help you
   *edit* one grounded in real expertise you supply.
7. **Bundle scripts only when the determinism is worth it** — 2.12× the vulnerability rate, and only
   21% of notable skills do it.
8. **Write to `.agents/skills/` and mirror to `.claude/skills/`** if portability matters.
9. **Never treat `allowed-tools` as a restriction.** Use real permission settings and
   `disableSkillShellExecution`.
10. **Prune on a schedule.** Skills go stale, staleness is adopted 63–72% of the time, and your
    strongest model is the one most damaged by it.

**And the honest summary of what a good skill buys you:** roughly +16pp on tasks it actually covers,
least of all in software engineering, declining as your model gets stronger, at a cost that is
either slightly positive or slightly negative depending on whose denominator you use. It is a real
effect, it is not magic, and the difference between a skill that helps and one that hurts is almost
entirely in the description and in whether you measured it.
