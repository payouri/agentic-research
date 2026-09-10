# Agent Skills rulebook

Checkable rules for writing and reviewing a `SKILL.md`. Each rule states the test, the evidence
strength, and the source key (see [sources.md](sources.md)).

Evidence: **[S]** peer-reviewed or controlled · **[V]** vendor documentation or specification ·
**[F]** field pattern across many real files · **[W]** practitioner report with some measurement ·
**[O]** opinion.

Use it two ways: as a checklist when writing, and as a review gate — a skill that fails R1, R2 or
R30 does not ship.

---

## A. Admission — should this be a skill at all?

**R1. The measurement test.** Before writing the body, define how you will know it worked: paired
with-skill / without-skill runs on the same tasks, multiple trials, deterministic verifiers where
possible. Test: does an eval exist that could fail? **[V]** `anthropic-skill-creator`,
`spec-best-practices` · **[S]** `skillsbench`, `aces`

**R2. The derivability test.** Delete anything the model already knows. HTTP verb semantics, "use
plural nouns for collections", general engineering advice. Test: would a competent model do this
correctly with no skill at all? If yes, delete it. **[V]** `anthropic-best-practices` · **[F]**
`corpus`

**R3. The altitude test.** A skill teaches *how to approach a class of problems*, not *what to
produce for one instance*. "A skill should teach the agent how to approach a class of problems, not
what to produce for a specific instance." **[V]** `spec-best-practices`

**R4. Not a fact — a procedure.** A standing fact about the repo belongs in `AGENTS.md`/`CLAUDE.md`,
which loads once. A multi-step procedure needed only for specific tasks belongs in a skill. Test:
"Is it a one-liner rule or guardrail every session needs?" → not a skill. **[V]** `cc-skills`,
`vercel-authoring-skills`

**R5. Do not have the model author it.** Model-generated skills measure **−1.3pp against no skills at
all**, and carry a **38% higher defect rate** with 2.3× more safety defects. Have the model help you
*edit* a skill grounded in expertise you supply. Test: can you name the human source of the domain
knowledge? **[S]** `skillsbench`, `skills138k` · **[V]** `spec-best-practices`

**R6. Expect the value to decay as models improve.** Skill utility correlates with backbone strength
at **r = −0.90**. Test: has this skill been re-benchmarked since your last model upgrade? **[S]**
`skillaudit`, `after`

---

## B. The description — the entire discovery surface

**R7. Everything about *when* goes in the description.** The body does not load until triggering has
already happened, so a trigger condition in the body is structurally unable to fire. "All 'when to
use' info goes here, not in the body." Test: grep the body for "use this skill when" — if it's
there and not in the description, it's in the wrong file. **[V]** `anthropic-skill-creator` · **[F]**
`corpus`

**R8. Three components: topic, trigger, negative scope.** Third-person clause naming what it does,
imperative "Use when…" clause naming when, explicit "Do NOT use for…" clause naming when not. The two
vendor pages contradict each other on voice; the best real descriptions do both. Test: can you point
at all three clauses? **[V]** `anthropic-best-practices`, `spec-descriptions` · **[F]** `corpus`

**R9. Write triggers in the user's vocabulary, not the maintainer's.** List literal phrasings a user
would type ("how many people", "is anyone clicking"), not internal abstractions. Best-in-corpus
example anticipates the gap explicitly: fires "even when the request reads like an ordinary feature
or bug fix and never says 'security,' 'injection,' or 'SafeSqlFragment.'" **[F]** `corpus`

**R10. Name concrete files and extensions where you can.** File-name triggers make activation
mechanical rather than semantic — "Use when editing `entry-base.ts`, `compiled/react*` packages".
Test: does the description contain at least one literal filename, extension, or command? **[F]**
`corpus` · **[V]** `vercel-authoring-skills`

**R11. Be pushy.** Under-triggering is the documented default failure: "Claude has a tendency to
'undertrigger' skills… make the skill descriptions a little bit 'pushy'." **52.3%** of public skills
have missing trigger guidance, the single most common defect. **[V]** `anthropic-skill-creator` ·
**[S]** `skills138k`

**R12. Negative scope is not optional.** Retrievers with Recall@3 of 0.848–0.888 surface a harmful
same-capability sibling in **34.6–37.2%** of cases; family-aware disambiguation cuts that to 0.007.
Your description is the author-side version of that fix. Test: does it name at least one thing it is
*not* for? **[S]** `samecaprisk` · **[F]** `corpus`

**R13. Put the key use case first.** Claude Code truncates `description` + `when_to_use` at **1,536
characters** in the listing; the spec and API reject over **1024**. Test: does the first sentence
survive truncation intact? **[V]** `cc-skills`, `spec`, `anthropic-api-skills`

**R14. Stay under 1024 characters, target 200–500.** Corpus median is **317**; the 200–500 band holds
66% of real skills. Over 1024 is rejected by the spec validator, the API, Cline, Roo, Kilo and Crush,
and silently accepted elsewhere. **[V]** `spec`, `spec-validator` · **[F]** `corpus`

**R15. No XML tags, no reserved words.** `name` and `description` are injected HTML-escaped into an
`<available_skills>` block; Anthropic's platform additionally forbids "anthropic" and "claude" in
`name`. The reference validator enforces neither — so this fails only at upload. **[V]**
`anthropic-overview`, `spec-prompt`

**R16. Test the description separately from the body.** "Seeing a skill trigger tells you Claude
found it, not that it did what you intended." Measure should-trigger and should-not-trigger prompts
in a fresh session. **[V]** `anthropic-skill-creator-blog`, `cc-skills`

---

## C. Structure — progressive disclosure

**R17. Keep the body under 500 lines.** Spec: "Keep your main `SKILL.md` under 500 lines" and
"Instructions (< 5000 tokens recommended)". Corpus median is 161 lines. **[V]** `spec`,
`anthropic-best-practices` · **[F]** `corpus`

**R18. Split the overflow into `references/`, not into a longer body.** Progressive disclosure beats
a normalised flat file by **+4.1pp pass rate** (46.1% vs 42.0%) at +$0.03/pass, holding knowledge
fixed. **[S]** `skilljuror`

**R19. Route conditionally, don't list.** "If you need to fill out a PDF form, read FORMS.md and
follow its instructions" — a condition and a destination. A bare "see also `references/details.md`"
is boilerplate. Test: does each pointer say *when* to follow it? **[V]** `anthropic-skills-pdf` ·
**[F]** `corpus`

**R20. One level deep, forward slashes, relative paths.** "Keep file references one level deep from
`SKILL.md`. Avoid deeply nested reference chains" — Claude may `head -100` a file reached through
another reference rather than reading it. **[V]** `spec`, `anthropic-best-practices`

**R21. Table of contents on reference files over 100 lines.** **[V]** `anthropic-best-practices`

**R22. Prefer a pointer to a copy.** The best file in the corpus is 8 lines: a description plus
"Source of truth: `apps/design-system/content/docs/copywriting.mdx`. Read it before writing or
auditing any UI copy." It cannot drift from what it points at. **[F]** `corpus`

**R23. Two or three focused modules beat one comprehensive document.** "Focused Skills with 2--3
modules outperform comprehensive documentation," and instruction-following degrades with density —
68% accuracy at 500 simultaneous instructions, with a bias toward earlier ones. **[S]**
`skillsbench`, `ifscale`

---

## D. Frontmatter and portability

**R24. Only the six spec fields are portable.** `name`, `description`, `license`, `compatibility`,
`metadata`, `allowed-tools`. Everything else — `model`, `context`, `when_to_use`, `hooks`, `paths` —
is Claude Code surface area. **Cline treats unknown fields as a hard parse error**; Zed ignores them
silently; the reference validator errors on them. Test: does it load in a non-Anthropic runtime?
**[V]** `spec`, `spec-validator`, `cline`, `zed`

**R25. `name` must match the parent directory, lowercase-kebab.** ≤64 chars, no leading/trailing or
consecutive hyphens. Note the reference implementation accepts any Unicode alphanumeric via
`str.isalnum()`, which is *wider* than the spec's stated `a-z, 0-9`. Anthropic's own plugin skills
violate the casing convention (`Hook Development`). **[V]** `spec`, `spec-validator` · **[F]**
`corpus`

**R26. File named `SKILL.md`, uppercase, at the root of its own folder.** 61/61 in the corpus. The
reference parser also accepts `skill.md`, but nothing else does reliably. No nesting: one skill per
folder. **[V]** `spec`, `spec-parser` · **[F]** `corpus`

**R27. Write to `.agents/skills/`, mirror to `.claude/skills/`.** `.agents/skills/` is the spec's
recommended interop path and is read by Cursor, Copilot, Codex, Gemini CLI, OpenCode, Crush, goose,
Cline, Kilo, Amp, Zed and Junie. **Claude Code does not read it.** Portability is one-directional
unless you write both. **[V]** `spec-impl-guide`, `cc-skills`, `zed`, `codex`, `gemini-cli` · **[F]**
`corpus`

**R28. Do not rely on nested-skill discovery.** Recursive in Cursor, OpenCode, Kilo, goose, Crush;
one level only in Cline, Roo, Claude Code and the Managed Agents repo scan; refused outright by Zed.
**[V]** `cursor`, `cline`, `zed`, `anthropic-managed-agents`

**R29. Do not rely on precedence.** The spec says "project-level skills override user-level skills."
**Claude Code says personal overrides project. Cline says enterprise > global > project.** Three
answers to one question. Test: does anything break if the other file wins? **[V]** `spec-impl-guide`,
`cc-skills`, `cline`, `zed`

---

## E. Safety

**R30. `allowed-tools` is not a sandbox. Never treat it as one.** Claude Code: "It does not restrict
which tools are available: every tool remains callable" — and "**Workspace trust doesn't gate this
field.**" Zed: "We parse the field but don't honor it." Codex, Gemini CLI, OpenCode, goose, Crush,
Kilo drop it. Factory: "not a runtime sandbox." Only GitHub Copilot acts on it, and only as
pre-approval. Use real permission settings instead. **[V]** `cc-skills`, `zed`, `factory`,
`github-docs`, `spec`

**R31. Bundle executable scripts only when determinism earns it.** Skills bundling scripts are
**2.12× more likely to contain vulnerabilities** (p<0.001). Only 21% of notable skills do it. **[S]**
`skills-wild` · **[F]** `corpus`

**R32. Treat an installed skill like installed software.** "Use Skills only from trusted sources…
a malicious Skill can direct Claude to invoke tools or execute code in ways that don't match the
Skill's stated purpose." **26.1%** of 31,132 marketplace skills carry ≥1 vulnerability; 5.2% show
"patterns strongly suggesting malicious intent." **[V]** `anthropic-overview` · **[S]** `skills-wild`

**R33. Do not rely on scanning to catch a malicious skill.** Payload-less attacks — harmful intent
expressed as abstract "compliance guidelines", with the agent writing the code itself — achieve up to
**77.67% leakage and 67.33% RCE at 0.00% detection**, against gateways that stop 91–99.81% of
conventional attacks. **[S]** `payload-less`

**R34. Gate project-scope skills on trust.** "This prevents untrusted repositories from silently
injecting instructions into the agent's context." Org-level: `disableSkillShellExecution` replaces
every command with `[shell command execution disabled by policy]`. **[V]** `spec-impl-guide`,
`cc-skills`, `anthropic-enterprise`

**R35. Review a skill diff as privileged.** A skill is an instruction-injection surface reachable
through a dependency, a marketplace, or a PR. The ClawHavoc campaign put 1,184 malicious skills on
one marketplace, 386 in a single day. **[W]** `clawhavoc` · **[S]** `threat-taxonomy`

---

## F. Maintenance

**R36. Prune on a schedule; staleness is worse than absence.** Agents adopt task-incorrect retrieved
guidance **63–72% of the time, independent of model scale**, diverge by step 7, and recover only
7–15% of the time. **Stronger models lose more** (Damage Per Compliance −2.1pp to −25.5pp). **[S]**
`compliance-trap`

**R37. Account for the standing cost of every installed skill.** "Every skill in the skill listing
adds to your context on every turn, whether or not Claude ever uses it." Run `/skill-doctor`. **[V]**
`cc-skills`

**R38. Keep the catalogue small and the descriptions non-overlapping.** Routing F1 drops **16–23pp**
as a catalogue scales; embedding shortlisting recovers 10–11pp of it, but a **≥10pp confusion gap is
unrecoverable even with perfect retrieval** and must be fixed by making options genuinely distinct.
**[S]** `routing-scale`

**R39. Do not assume a skill transfers.** Applying a skill evolved for one role to another cost
**−4.8 to −7.5 points**. **[S]** `after`

**R40. Re-run the eval, don't re-read the file.** Structural quality scores and LLM-judge rubrics
correlate at **Spearman ρ = 0.14**; 87 of 947 paired cases had *negative* lift that was "invisible to
scanning methods." **[S]** `aces`

---

## Review checklist

A skill does not ship until every one of these passes:

| # | Gate | Rule |
|---|---|---|
| 1 | An eval exists that could fail, and it has been run with and without the skill | R1, R40 |
| 2 | Nothing in the body is knowledge the model already has | R2 |
| 3 | The description carries topic + trigger + negative scope | R8, R12 |
| 4 | Triggers use the user's vocabulary, and name at least one literal file/extension/command | R9, R10 |
| 5 | Description under 1024 chars, key use case first | R13, R14 |
| 6 | Body under 500 lines; overflow is in `references/` with conditional routing | R17, R18, R19 |
| 7 | Frontmatter uses only spec fields, or you accept Claude-Code-only portability | R24 |
| 8 | `name` matches the directory; file is `SKILL.md` at the folder root | R25, R26 |
| 9 | Nothing depends on precedence or nested discovery resolving your way | R28, R29 |
| 10 | No one is treating `allowed-tools` as a restriction | R30 |
| 11 | Bundled scripts are justified, or absent | R31 |
| 12 | The skill was not authored by the model | R5 |

**The two that gate everything else:** a skill with no eval (R1) is unfalsifiable, and a skill whose
description omits its trigger (R7) will never run to be judged.
