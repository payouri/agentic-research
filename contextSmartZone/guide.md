# The context smart zone — the guide

Researched 2026-09-17 from primary sources: vendor documentation and API references, the shipped
source of 17 agent harnesses, the 2023–2026 long-context measurement literature, and the published
practice of the teams building these tools. Companion files: [rulebook.md](rulebook.md) (the
checkable rules) and [sources.md](sources.md) (every source, tiered and annotated).

---

## 1. The finding that organises everything else

**The zone in which an agent works well is roughly an order of magnitude smaller than the point at
which any harness intervenes.**

That sentence is the dossier. Both halves are measured, from independent sources, and nobody
publishes them side by side.

The **measured** zone, from peer-reviewed work: NoLiMa defines *effective length* as "the longest
context where a model maintains at least 85% of its base score" and reports it at **2K–16K tokens**
for models advertising 128K to 2M (`nolima`). BABILong, independently, finds that models "effectively
utilize only 10-20% of the context and their performance declines sharply with increased reasoning
complexity" (`babilong`).

The **enforced** boundary, read out of shipped source:

| Harness | Fires at | Of what | Source |
|---|---|---|---|
| Gemini CLI | 0.5 | the model's token limit | `gemini-code` |
| Continue (CLI) | ≈0.605 @200K | total input incl. system + tool defs | `continue-src` |
| Goose | 0.8 | `current_tokens / context_limit` | `goose-src` |
| Crush | 0.8 or ~0.9 | tokens *remaining* (20K, or 0.2×ctx) | `crush-src` |
| Copilot CLI | ~0.8 | context window capacity | `copilot-cli` |
| Cline | ≈0.81 | 0.9 of (window × 0.9) | `cline-src` |
| Zed | 0.9 | `max_input_tokens` | `zed-src` |
| Codex CLI | 0.9, hard cap 0.95 | resolved context window | `codex-src` |
| Roo Code | 1.0 by default | slider ships at 100% | `roo-docs` |
| **Claude Code** | **window − 13,000 tokens** | absolute; ≈0.935 @200K | `cc-binary`, `cc-model-config` |
| Kilo Code | 1.0 | `input_limit − min(20K, max_output)` | `kilo-src` |
| opencode | 1.0 | the usable input budget | `opencode-src` |

Not one harness intervenes inside the measured zone. The *lowest* trigger in the field, Gemini CLI's
0.5, sits at roughly three to five times the effective length NoLiMa measures. The most common
design, clustering at 0.8–0.95, sits an order of magnitude past it.

**This is not a criticism of the harnesses, and reading it as one is the mistake.** A compaction
threshold is *overflow protection*. Its job is to stop the API returning `400 invalid_request_error`
(`ctx-windows`), and at that job every one of these numbers is correct. The error is in what users
infer from it. A progress bar that stays green until 90% is reporting headroom, not health — and
nothing in any of these products tells you that the two are different quantities.

And two lineages make the point structurally rather than numerically. Roo Code resolves
`autoCondenseContextPercent ?? 100` (`roo-default`) and Kilo Code's `threshold_percent` has no default
at all (`kilo-src`) — **both ship the percentage trigger disabled and fall back to an absolute
overflow buffer.** The quality knob exists, and out of the box it does nothing. That is the clearest
statement any of these products makes about what compaction is for.

So: **the harness will not keep you in the smart zone, because it is not trying to.** Managing the
zone is the user's job, and it starts long before any of these numbers is reached.

---

## 2. What the vendors will and will not say

Anthropic now names the phenomenon in its own product documentation:

> "A larger context window allows the model to handle more complex and lengthy prompts, but more
> context isn't automatically better. As token count grows, accuracy and recall degrade, a phenomenon
> known as *context rot*. This makes curating what's in context just as important as how much space
> is available." — `ctx-windows`

Claude Code's best-practices page makes it the organising constraint of the entire document:

> "Most best practices are based on one constraint: Claude's context window fills up fast, and
> performance degrades as it fills." — `cc-best`

And the compaction docs state the quality claim as a plain fact: "as a conversation grows, response
quality degrades, so compaction replaces older content with a concise summary" (`compaction-api`).
The engineering blog supplies the mechanism — "LLMs have an 'attention budget'", context is "a finite
resource with diminishing marginal returns" (`eng-context`).

**What no vendor page contains is a number.** Not a percentage, not a token count, not a per-model
effective-context table. Anthropic names the decay, builds three separate product features against it
and never says where it begins. OpenAI does not make the claim at all: its compaction guide frames
the same feature as a way to "balance quality, cost, and latency as conversations grow"
(`oai-compaction`), with no degradation statement anywhere on the page, against a stated 1.05M combined window
(`oai-models`). Its Codex CLI goes further and declines to publish the per-model default at all —
"unset uses model defaults" (`codex-config`). Google's admission is
carefully scoped to concede the multi-fact case while defending the single-fact one — "In cases where
you might have multiple 'needles' … the model does not perform with the same accuracy"
(`gem-longctx`).

Vendors publish *capacity*. Nobody publishes *effective* capacity. That silence is the reason the
community invented a number, and §5 is about the number it invented.

### The tell is in the defaults

The vendors do not state a threshold, but they set them, and the API defaults are dramatically
tighter than the harness defaults from the same company:

| Anthropic surface | Trigger | Of a 1M window |
|---|---|---|
| `clear_tool_uses_20250919` | 100,000 input tokens | **10%** |
| Server-side compaction | 150,000 input tokens (floor 50,000) | **15%** |
| Claude Code auto-compact | ~967,000 − 13,000 | **~95%** |

The same company, the same models, the same month. The API behaves as though the comfortable zone is
narrow; the coding harness behaves as though it is nearly the whole window. OpenAI's compaction
examples use `compact_threshold: 200_000` against a 1.05M window — ~19%, strikingly close to
Anthropic's 15% (`oai-compaction`), though the page never labels it a default.

Neither company explains the gap. The most defensible reading, and it is **inference, not vendor
intent**: the API default is tuned for quality on long-running agents, and the harness default is
tuned so that interactive users are not interrupted. Those are different objectives, and only one of
them is about the smart zone.

---

## 3. What is actually measured

### Degradation is gradual from the start, not a cliff at the limit

This is the most consistent result in the literature, and it defeats the intuition the progress bar
encourages. RULER, across 17 models: "despite achieving nearly perfect accuracy in the vanilla NIAH
test, almost all models exhibit large performance drops as the context length increases" (`ruler`).
LoCoDiff, on a genuinely agent-shaped task — reconstruct a file's final state from its commit history
— shows Sonnet 4.5 falling monotonically from **96% at ~2K tokens to 64% at ~98K** (`locodiff`).
There is no knee in that curve to stay below. Every token costs a little.

Nor does a reasoning model buy immunity: on NoLiMa-Hard, GPT-o1 falls from 99.9 at base to **31.1 at
32K** (`nolima-repo`). Nor is this only a retrieval problem — LongProc finds "significant
degradation on 8K-token tasks" in long-form *generation* (`longproc`), and on LongBench v2 the best
direct model scores 50.1% against human experts at 53.7% (`longbench2`).

### Length hurts even when nothing is wrong with the content

The single strongest causal result, and the one that should change behaviour: *Context Length Alone
Hurts LLM Performance Despite Perfect Retrieval* (EMNLP 2025 Findings) finds that

> "even when models can perfectly retrieve all relevant information, their performance still degrades
> substantially (13.9%--85%) as input length increases." — `length-alone`

The control is what makes it load-bearing: the degradation **persisted when the irrelevant content
was replaced with whitespace or masked entirely.** So this is not only a distraction effect. Sequence
length is itself a cost.

That matters because it rules out the comfortable conclusion. "Keep the context clean" is not
sufficient advice. Clean and long still loses to clean and short. The instruction that follows is
not *curate* but *truncate*.

Distraction is real too, and cheaper to trigger than anyone expects. Chroma, across 18 models: "Even
a single distractor reduces performance relative to the baseline (needle only)" — and, against
intuition, models "perform better on shuffled haystacks than on logically structured ones"
(`ctx-rot`).

### The benchmark you have seen is the wrong one

HELMET evaluated 59 long-context models and concluded that "synthetic tasks like NIAH do not reliably
predict downstream performance" (`helmet`). Near-perfect needle-in-a-haystack coexists with large
drops on anything requiring reasoning across the whole context. Every "we support 1M tokens, here is
our NIAH chart" claim is measuring the easiest task in the space.

### The failure mode is not what you are watching for

You are watching for the agent to get confused. That is not what happens first.

A white-box study of a coding agent on a 24-check code-audit workflow found success falling from
**80% clean to 30% in extended context (~300K chars)** — while **requirement coverage held at 92–94%**
and strict task success fell to 37.5% of baseline (`coding-rot`). The agent keeps doing the bulk of
the work and quietly drops the sparse critical obligations. That is precisely the failure a human
reviewer skims past, because the output looks complete.

Three more failure shapes, all measured, none of which look like confusion:

- **Premature termination.** Models "give up or provide uncertain incorrect answers long before
  exhausting the context window", and the rate rises with length even controlling for query
  complexity (`premature`). A separate study measured refusal rates climbing **0% → 89.6%** as
  context approached model ceilings (`prompt-scale`).
- **Unreliability, not incapacity.** Multi-turn conversation costs a **39% average drop** across six
  tasks, and the decomposition is the interesting part: minor aptitude loss, large *unreliability*
  increase. "when LLMs take a wrong turn in a conversation, they get lost and do not recover"
  (`multiturn`). The remedy for variance is a retry from a clean context, not a longer conversation.
- **Instruction dilution.** Perfect-response rate "collapses to zero by N=80" instructions, for every
  model and both placements tested (`prompt-scale`). Anthropic says the same thing about its own
  memory file: "Bloated CLAUDE.md files cause Claude to ignore your actual instructions!" (`cc-best`).

### The middle is still the worst place to put anything

"Lost in the Middle" (TACL) has not been refuted — it has been *explained*, as an emergent adaptation
to primacy and recency demands in pre-training (`lim-emergent`). And three independent 2026
measurements still find the penalty: retrieval, injection defence (`longpibench`), and safety
monitoring, where monitors are "particularly weak" in middle positions of long transcripts
(`classifier-rot`).

Two of those deserve separate billing, because they are about safety rather than quality:

- **Long benign content hides an injection rather than diluting it.** LongPIBench finds injection at
  the middle or end beats the front — code-review attack success **0.53 → 0.87** — and that defences
  decay as the document grows (`longpibench`). The position that is worst for *your* content is best
  for an attacker's.
- **Monitoring degrades hardest of anything measured.** Claude Opus 4.6 with thinking: needle recall
  **99.7% at 100K → 69% at 800K**, with overall degradation of **2×–30×** when a malicious action
  follows a long benign prefix (`classifier-rot`, Anthropic's own research). The oversight fails
  before the agent does.

### One honest counter-witness

LoCoBench-Agent reports flat comprehension (0.71–0.75) from 10K to 1M tokens and concludes that
"context management capabilities have reached architectural parity" (`locobench`). It points the
opposite way to everything above and it is **not averaged away** here.

Three plausible reasons it differs, none established: it scores a graded 0–1 metric rather than
strict success — and `coding-rot` shows exactly the shape where graded coverage holds at 92–94% while
strict success collapses; it is a tool-using agent that *retrieves* rather than holding 1M tokens
in-window, so it may be measuring harness quality; and its authors themselves note that larger
codebases "typically feature more explicit architectural documentation", confounding difficulty with
length. Recorded as open in [sources.md](sources.md).

---

## 4. The number you are shown is not the number that binds

Reconciling the harness sources produced a result nobody has published: **the thresholds are not
comparable, and several are not what the product tells you.**

### No two harnesses share a denominator

Five different quantities appear under the same "%" sign:

1. **Raw model window** — Gemini CLI, Goose, Codex.
2. **A derived input budget** — Cline (0.9 of window×0.9), Zed (`max_input_tokens`), opencode
   (`limit.input − reserved`).
3. **Tokens remaining, counted downward** — Crush, and **Claude Code**.
4. **Total input including system prompt and tool definitions** — Continue.
5. **A sub-budget that never bounds the prompt at all** — Aider's repo-map and chat-history budgets.

Putting Codex's "90%" beside Zed's "90%" beside Claude Code's "≈93.5%" in one column compares four
different things. Any comparison table you have seen, including ones built from these same repos,
is almost certainly wrong in this specific way.

### Claude Code's threshold is absolute, and that settles a live dispute

Two contradictory third-party claims circulate — "~83% with a `Math.min()` clamp" and "~95%". The npm
package is no longer a JS bundle (it ships a native binary, which is why `cli.js` greps stopped
working), so this was settled by reading the shipped artifact (`cc-npm`, `cc-binary`):

```js
function zPe(e,n){
  let r=e-13000, s=n.testPctOverride;
  if(s!==void 0&&!isNaN(s)&&s>0&&s<=100)
    return Math.min(Math.floor(e*(s/100)),r);
  return r
}
```

The threshold is **`effective_window − 13,000` tokens — absolute, not a percentage.** On a 200K window
that is 93.5%; on the 967K effective window it is 98.7% of 967K but ~95.4% of a raw 1M. **The
percentage is an output of the arithmetic, not an input** — which is exactly why field reports
disagree with each other. Both circulating numbers are wrong as stated; the *mechanism* the "83%"
claim asserted (a one-directional `Math.min` clamp) is real, and is confirmed first-party:
`CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` "can't raise the threshold, so values above the default percentage
are ignored" (`cc-envvars`).

A P5 rumour predicted the mechanism correctly and both its numbers wrongly. Tiers attached, as the
record requires.

### The boundary can move without a release

Claude Code's resolution order for the effective window includes `clientdata` and `experiment` —
both server-side (`cc-binary`). Gemini CLI's `getCompressionThreshold()` is `async` with a local-first
check, strongly suggesting the same (`gemini-code`, inferred). **Two of the largest harnesses can move
your boundary without shipping anything**, and neither documents it. There is an open issue reporting
exactly the symptom this would produce — "[BUG] Auto-compact threshold silently regressed — fires at
~73% context used, previously ~83%" (`cc-86863`) — offered here as a hypothesis, not a diagnosis.

### Sometimes compaction does not fire at all

Also from the binary, verbatim from its telemetry schema:

> `enforced: "Whether threshold-triggered compaction will actually fire at `threshold`. False when
> the worker defers to the API's prompt-too-long (reactive mode) …"`

In **reactive mode**, Claude Code does no proactive compaction and waits for the API to reject the
prompt; the UI shows a percentage instead of a countdown. Users in reactive mode and users in
threshold mode are observing genuinely different machines, which explains a great deal of the
contradictory field reporting.

### Four more places the label lies

- **Gemini CLI: docs say `0.7`, code says `0.5`.** `DEFAULT_COMPRESSION_TOKEN_THRESHOLD = 0.5`
  (`gemini-code`) against a live docs page still publishing 0.7 (`gemini-docs`). A user reading the
  documentation believes they have 20 percentage points more runway than they have.
- **Roo Code's advertised knob is inert.** The condensing slider ships at **100%** (`roo-docs`), so
  the real boundary is the buffer formula `contextWindow × 0.9 − reservedTokens`. Core accepts a
  minimum of 5, the UI slider stops at 10, the docs state neither bound (`roo-src`).
- **Continue's displayed percentage understates.** The UI counts chat history; the trigger counts
  history *plus* system prompt *plus* tool definitions — and an in-code comment says "80% threshold"
  where the real trip point is ≈60.5% (`continue-src`).
- **Cursor's vendor concedes the trigger is unreliable.** The docs say summarisation fires when the
  window "fills up". Staff, on the record: "This is a known issue with auto-summarization. It can
  trigger late or incorrectly" — advising users to run `/summarize` manually at "70 to 80%"
  (`cursor-forum`). A separate user measures it firing at 10–20%. The number is unpublished and has
  regressed in both directions.
- **Kilo Code's docs advertise a default that does not exist.** One page annotates
  `threshold_percent` "(default: ~80%)"; another says "unset"; **no 80 appears anywhere in the
  compaction path** (`kilo-docs`, `kilo-src`).
- **Cline's widely-cited rule is code it no longer runs.** The `contextWindow − 40_000` / ×0.8
  formula that circulates in write-ups is preserved in-repo as **legacy** (`cline-legacy`); current
  Cline summarises at ≈0.81. A live citation hazard.
- **Zed's setting is silently inert below an 80K window** (`MIN_COMPACTION_CONTEXT_WINDOW`), and its
  UI warns at 0.8 while compaction fires at 0.9 (`zed-src`).

And Claude Code's displayed `pctLeft` is computed against the *threshold-adjusted* window, not the
raw model window (`cc-binary`) — so the percentage on screen is not a percentage of your context
window in any product listed here that you might assume it is.

One harness resisted verification entirely — Amp's manual is auth-gated. A second, Kilo Code, first
looked restructured beyond reach; it turned out to have **changed lineage**, from a Roo Code fork to
an OpenCode fork (`kilo-src`), which is why its old constants are absent rather than moved. A third-party cross-harness survey (`badlogic`) remains the only observation
covering several of these at once, and its figures predate the 2026 restructures.

**The practical consequence:** do not build a personal rule on a displayed percentage. Build it on
something you control — a task boundary, a file count, a phase transition.

---

## 5. What practitioners do, and the one number to stop repeating

The field converged on advice long before it had evidence, and the advice is mostly right for reasons
its authors did not have.

**The 40% rule is folklore.** Its hardest statement — quality zones at 0–40% / 40–70% / 70%+ — is
introduced by its author as "a mental model", carries no citation, and rests on a simulation whose own
parameter is "Each file **read** uses 5% of the context window" (`ness`). Its earliest traceable
ancestor is experiential and appropriately hedged: "keeping utilization in the 40%-60% range …
(depends on complexity of the problem)" (`ace-fca`).

The competing figure in circulation — 10–20% — *does* have a source, and fetching it settles the
matter: BABILong, NeurIPS 2024 (`babilong`). **The cited number survives contact with its source and
the popular one does not.** The honest position is that the direction of the 40% advice is well
supported and the number is invented, and that the measured figure is considerably *stricter* than
the folklore, not looser.

**What practitioners agree on, and the evidence supports:**

- **Write it to a file instead of holding it.** The most consistently adopted convention in the
  corpus: Manus treats "the file system as the ultimate context … unlimited in size, persistent by
  nature" (`manus`); Cursor sends long tool output to disk so "the agent calls `tail` to check the
  end" (`cursor-ddc`); Anthropic's own advice is to write a spec and start a fresh session
  (`cc-best`).
- **`/clear` between unrelated tasks**, vendor-endorsed: "Long sessions with irrelevant context can
  reduce performance" (`cc-best`). And the sharper heuristic: "If you've corrected Claude more than
  twice on the same issue in one session, the context is cluttered with failed approaches."
- **Scope investigations.** Anthropic names the failure directly — "The infinite exploration. You ask
  Claude to 'investigate' something without scoping it. Claude reads hundreds of files, filling the
  context" (`cc-best`).
- **Defer what you might not need.** Cursor measured a **46.9% reduction in total agent tokens** from
  loading MCP tool schemas on demand (`cursor-ddc`) — the cleanest measured practice number in the
  corpus. Claude Code does the same, loading schemas upfront only within 10% of the window
  (`cc-ctxwin`).

**And a gap between what people say and what they configure.** Across GitHub: ~42,112 repos set
`permissions` in `.claude/settings.json`, but only **199** cap MCP output tokens (~0.5%) and **4**
disable autocompact (`ghsearch`). The convention lives in prose instructions and custom commands, not
in anything a harness enforces. Handoff-flavoured commands outnumber compaction-flavoured ones about
2:1 — consistent with where the tooling is moving.

**Because the tooling is moving.** Amp removed compaction outright: "It's lossy, for one", and it
"encourage[s] long, meandering threads, in which you just compact once you run out of context window,
stacking summary on top of summary" (`amp-handoff`). Whether it has since returned is left open in
[sources.md](sources.md) — a later Amp note says "auto-compaction makes longer threads work well"
with no post announcing a reinstatement, and the removal post is still live and un-annotated. Amp's
own context-engineering guide, notably, states **no numeric threshold at all** (`amp-guide`) — a
deliberate abstention that looks better in light of §4 than most of the numbers do.

---

## 6. Compaction: what the evidence actually licenses

Three results, and they are more interesting together than apart.

- **The cheap strategy matches the expensive one.** Simple observation masking — just dropping old
  tool outputs — "halves cost relative to the raw agent while matching, and sometimes slightly
  exceeding, the solve rate of LLM summarization" on SWE-bench Verified (`complexity-trap`).
- **Compaction quality is a large free variable.** Trained compaction *raises* Pass@1 (59.8 → 66.8),
  and "Changing only the summary agent leads to a large difference in final task performance" —
  **6.5 absolute points** between a good and a weak summariser (`compactionrl`).
- **Agent-invoked compaction works under a bounded budget**, reaching 57.6% on SWE-Bench-Verified by
  compacting at task milestones the agent itself chooses (`cat`).
- **Structured eviction can be lossless in practice.** Dependency-graph eviction ran 89 sequential
  tasks and 80M tokens in one session with "no measurable degradation in task accuracy relative to
  per-task isolated sessions" (`beyond-compaction`).

Read together: **compaction is not inherently lossy in a way that must cost you accuracy — but the
thing doing the compacting is a real variable, and dropping old tool results is a shockingly strong
baseline.** Which is, notably, exactly what Claude Code does first: "It clears older tool outputs
first, then summarizes the conversation if needed" (`cc-how`), and what the API exposes as a separate
primitive at a *tighter* trigger than compaction itself (`ctx-editing`). Anthropic's own position is
that none of this is enough on its own — "However, compaction isn't sufficient" (`eng-harness`).

**The honest gap:** nobody has measured whether *harness auto-compaction specifically* costs accuracy.
The literature compares engineered compaction against no compaction. The negative control a harness
user actually wants — same task, compaction on versus off — is unpublished. Anyone who tells you
auto-compact costs you N points is not citing a study, because there isn't one.

A fourth strategy is emerging that none of the above measures: **compaction trained into the model
itself.** Cognition reports that "The model learns to summarize its working state and resume from the
summary, extending task horizons past the raw context window", with rollouts reaching "up to six
hours" (`swe17`). The same team documents the failure mode that comes with it — a model aware of its
own window showed "context anxiety", "taking shortcuts or leaving tasks incomplete when it believed
it was near the end of its window, even when it had plenty of room left" (`devin-sonnet`). Which is
the premature-termination result of §3, arriving from the opposite direction.

What compaction demonstrably does cost is **instructions**. Claude Code's docs say it plainly:
"detailed instructions from early in the conversation may be lost. Put persistent rules in CLAUDE.md
rather than relying on conversation history" (`cc-how`). And the corresponding bug has been open since
2025-08-22, in the reporter's words: "I have to tell it to re-read this every time it compacts,
otherwise it starts doing things in 'common sense' ways that deviate from the 'correct way' that is
defined" (`cc-6354`), with a live successor filed 2026-09-08 (`cc-92949`).

---

## 7. Sub-agents have an empirical trigger

The strongest available result is a corrective to enthusiasm. Holding *intermediate reasoning tokens
constant* — the control most multi-agent comparisons omit — single-agent matched or beat **every** one
of five multi-agent architectures across three model families (`single-agent`). Cognition argued the
same from experience (`cognition`) — and has since reversed itself, on grounds that belong to this
dossier: "having a clean context makes the agent smarter because of the math of attention", with
coding and review agents that "do not share any context beforehand" (`cognition-2026`). Both posts
are live and the reversal is framed as evolution, never as retraction.

But the exception is the whole point for this topic. Multi-agent became competitive **only** "in
highly degraded contexts (heavy substitution or masking of information)."

**So fan-out is not a general-purpose improvement; it is a context remedy.** It pays when the single
agent's context has already rotted, and costs you when it has not. That gives a criterion rather than
a preference: delegate when the work would otherwise drag a large volume of material through your
main context, not because parallelism sounds efficient.

The mechanism is well documented where it exists. A Claude Code subagent runs with "a fresh, separate
context window… without your conversation history", and "Only the subagent's final text response comes
back to your context" — the docs' own worked example reads 6,100 tokens of files and returns 420
(`cc-ctxwin`). Anthropic's engineering post puts typical returns at "often 1,000-2,000 tokens"
(`eng-context`). The pattern is near-universal where it exists: Cursor's subagents each hold their own
window and return only a summary — with the honest caveat that "Running five subagents in parallel
uses roughly five times the tokens of a single agent" (`cursor-subagents`) — and Devin's run as full
child sessions that cannot see the parent's files (`devin-cli`).

---

## 8. What to actually do

Ordered by how much of the problem each one removes.

1. **Treat the harness threshold as a fire alarm, not a speed limit.** It exists to prevent an API
   error, and every product's number is tuned for that. Nothing in the tooling is trying to keep you
   in the smart zone.
2. **Start fresh at every task boundary.** This is the single highest-leverage action, it is
   vendor-endorsed, and the multi-turn evidence explains why: the damage is unreliability, and a
   clean restart resets variance in a way that continuing cannot (`multiturn`).
3. **Put durable instructions on disk, never in conversation.** They are the first thing compaction
   loses, by the vendor's own admission, and the bug has been open for a year. Keep them short —
   instruction adherence collapses with instruction count (`prompt-scale`), and a bloated memory file
   gets ignored (`cc-best`).
4. **Make the big things not enter context at all.** Files over inline output; deferred tool schemas
   (46.9% measured); scoped investigations rather than "look into this". The cheapest token is the
   one you never read.
5. **Count the cost, not just the quality.** "A 50% increase in average context length translates
   directly to 50% higher inference cost" (`factory`), and caching cuts cost 41–80% and
   time-to-first-token 13–31% — but only if dynamic content stays out of the cached prefix
   (`cache`). Independent standings for calibration: `aa-lcr`, and `fictionlive` for method.
6. **Clear old tool results aggressively.** It is the strongest cost/benefit intervention measured —
   equal solve rate at half the cost (`complexity-trap`) — and both the API and Claude Code implement
   it as a distinct, earlier-firing mechanism than summarisation.
7. **Compact deliberately, at a phase boundary you choose.** Not because auto-compaction is proven
   harmful — it isn't, and nobody has measured it — but because a summary you trigger at a clean
   moment is a better summary, and summariser quality is worth ~6.5 points (`compactionrl`).
8. **Delegate to a sub-agent when the alternative is dragging bulk through your own context** — and
   not otherwise (`single-agent`).
9. **Do not trust the percentage on screen.** Different denominators, inert settings, stale docs,
   absolute-not-percentage thresholds, and two vendors able to move the boundary server-side.
10. **Put the important thing first or last, never in the middle** — still true in 2026, across
   retrieval, injection defence and monitoring.
11. **Review long-context output for what is missing, not for what is wrong.** The measured failure is
    92–94% coverage with the sparse critical obligations dropped. It looks finished. That is the
    danger.

**And the honest summary of what this buys you.** The evidence supports *faster, more reliable, and
more complete* — not *smarter*. Degradation is gradual from the first token, so there is no threshold
to sit just below and no configuration that makes the problem go away; there is only the discipline of
keeping the context short, current and yours. The strongest single result in the literature is that
length hurts **even when everything in the context is relevant and perfectly retrievable**
(`length-alone`). That is the finding to carry: the goal is not a clean context. It is a short one.
