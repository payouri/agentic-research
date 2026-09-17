# Context smart zone source lexicon

Every source behind [guide.md](guide.md) and [rulebook.md](rulebook.md), keyed for citation.
Gathered 2026-09-17.

Trust tiers: **P1** primary spec/vendor documentation or source code · **P2** peer-reviewed or arXiv
research · **P3** vendor engineering blog / industry research with method · **P4** practitioner
report with measurement · **P5** opinion, anecdote, or unverified secondary.

---

## 1. Primary — what the vendors say

### Anthropic

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `ctx-windows` | [Context windows](https://platform.claude.com/docs/en/build-with-claude/context-windows) | P1 | live, fetched 2026-09-17 | Window sizes; the **"context rot"** admission in product docs; overflow behaviour (400 `invalid_request_error` vs `stop_reason: "model_context_window_exceeded"`); that caching does **not** free window space; the injected `<budget>`/`<system_warning>` context-awareness tags and which models get them |
| `compaction-api` | [Compaction](https://platform.claude.com/docs/en/build-with-claude/compaction) | P1 | live, beta `compact-2026-01-12`, fetched 2026-09-17 | Default trigger **150,000 input tokens**, floor **50,000**, `pause_after_compaction: false`; "response quality degrades" stated as fact |
| `ctx-editing` | [Context editing](https://platform.claude.com/docs/en/build-with-claude/context-editing) | P1 | live, beta `context-management-2025-06-27`, fetched 2026-09-17 | `clear_tool_uses_20250919` defaults: trigger **100,000**, `keep` **3 tool uses**, `clear_tool_inputs: false`; cache-invalidation rule and why `clear_at_least` exists |
| `memory-tool` | [Memory tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool) | P1 | live, `memory_20250818`, fetched 2026-09-17 | The auto-injected "ASSUME INTERRUPTION" system prompt |
| `pricing` | [Pricing](https://platform.claude.com/docs/en/about-claude/pricing) | P1 | live, fetched 2026-09-17 | Long context billed at standard rates (the 1M premium is gone); the **~30% tokenizer inflation** on Claude 4.7+ |
| `eng-context` | [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | P3 | 2025-09-29, fetched 2026-09-17 | The "attention budget" framing; "finite resource with diminishing marginal returns"; the compaction / note-taking / sub-agent triad; subagent returns "often 1,000-2,000 tokens" |
| `eng-harness` | [Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) | P3 | 2025-11-26, fetched 2026-09-17 | That Anthropic considers compaction insufficient alone — "However, compaction isn't sufficient" |
| `cc-best` | [Best practices for Claude Code](https://code.claude.com/docs/en/best-practices) | P1 | live, fetched 2026-09-17 | The bluntest vendor degradation statement — "performance degrades as it fills"; the five named failure patterns; `/clear` between unrelated tasks; "Bloated CLAUDE.md files cause Claude to ignore your actual instructions!" |
| `cc-ctxwin` | [Explore the context window](https://code.claude.com/docs/en/context-window) | P1 | live, refs v2.1.198, fetched 2026-09-17 | What survives compaction; **5,000 tokens/skill, 25,000 total**; five-file re-read; 5,000-token file→path degradation; MEMORY.md **200 lines or 25KB**; tool schemas upfront only within **10% of the window**; subagent isolation and its token arithmetic |
| `cc-model-config` | [Model configuration](https://code.claude.com/docs/en/model-config) | P1 | live, fetched 2026-09-17 | Default auto-compact **~967K** on 1M models; `/autocompact` accepted range **100K–1M**; `CLAUDE_CODE_AUTO_COMPACT_WINDOW`, `autoCompactWindow` |
| `cc-how` | [How Claude Code works](https://code.claude.com/docs/en/how-claude-code-works) | P1 | live, fetched 2026-09-17 | The **two-tier boundary**: clear older tool outputs first, summarise second; the anti-thrash guard; "detailed instructions from early in the conversation may be lost" |

### OpenAI

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `oai-compaction` | [Compaction](https://developers.openai.com/api/docs/guides/compaction) | P1 | live, fetched 2026-09-17 | The two compaction surfaces; the opaque compaction item; `compact_threshold: 200_000` in every example; **the absence of any degradation claim** |
| `oai-models` | [Models](https://developers.openai.com/api/docs/models) | P1 | live, fetched 2026-09-17 | 1.05M combined window / 128K output for the GPT-5.6 and GPT-6 families |
| `codex-config` | [Sample configuration](https://learn.chatgpt.com/docs/config-file/config-sample) | P1 | live, fetched 2026-09-17 | `model_auto_compact_token_limit`, `model_auto_compact_token_limit_scope`; that OpenAI **declines to publish** the per-model default ("unset uses model defaults") |

### Google

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `gem-longctx` | [Long context](https://ai.google.dev/gemini-api/docs/long-context) | P1 | updated 2026-06-22, fetched 2026-09-17 | Google's only degradation concession — multi-needle: "the model does not perform with the same accuracy"; "if you don't need tokens to be passed to the model, it is best to avoid passing them"; put the query last |
| `gem-caching` | [Context caching](https://ai.google.dev/gemini-api/docs/caching) | P1 | updated 2026-09-02, fetched 2026-09-17 | Minimum cacheable tokens (2,048 / 4,096 by model); caching framed as cost, never as context management |

---

## 2. Evidence — what has been measured

### Effective context and its collapse

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `nolima` | [NoLiMa: Long-Context Evaluation Beyond Literal Matching](https://arxiv.org/abs/2502.05167) — Modarressi et al., **ICML 2025** | P2 | v1 2025-02-07, v3 2025-07-09, fetched 2026-09-17 | The definition of **effective length** ("at least 85% of its base score") and the collapse to **2K–16K** for 128K–2M windows; GPT-4o 99.3 → 69.7 at 32K |
| `nolima-repo` | [adobe-research/NoLiMa README](https://raw.githubusercontent.com/adobe-research/NoLiMa/main/README.md) | P2 | fetched 2026-09-17 | The per-model effective-length table (22 models) and NoLiMa-Hard: GPT-o1 99.9 → 31.1 at 32K — **reasoning buys no immunity** |
| `babilong` | [BABILong](https://arxiv.org/abs/2406.10149) — Kuratov et al., **NeurIPS 2024 D&B** | P2 | v1 2024-06-14, v2 2024-11-06, fetched 2026-09-17 | The **10–20%** effective-utilisation figure, and that it declines "sharply with increased reasoning complexity" |
| `ruler` | [RULER](https://arxiv.org/abs/2404.06654) — Hsieh et al., NVIDIA | P2 | 2024-04-09, fetched 2026-09-17 | "despite achieving nearly perfect accuracy in the vanilla NIAH test, almost all models exhibit large performance drops as the context length increases" |
| `helmet` | [HELMET](https://arxiv.org/abs/2410.02694) — Yen et al., **ICLR 2025** | P2 | 2024-10-03, rev 2025-03-06, fetched 2026-09-17 | 59 models: "synthetic tasks like NIAH do not reliably predict downstream performance"; the gap "widens as length increases" |
| `longbench2` | [LongBench v2](https://arxiv.org/abs/2412.15204) — Bai et al., **ACL 2025** | P2 | 2024-12-19, fetched 2026-09-17 | Human experts 53.7% vs best direct model 50.1%; inference-time compute helps (o1-preview 57.7%) |
| `longproc` | [LongProc](https://arxiv.org/abs/2501.05414) — Ye et al., Princeton PLI | P2 | 2025-01, fetched 2026-09-17 | Degradation on **long-form generation**, not just retrieval: GPT-4o "significant degradation on 8K-token tasks" |

### Length as an independent cost

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `length-alone` | [Context Length Alone Hurts LLM Performance Despite Perfect Retrieval](https://arxiv.org/abs/2510.05381) — Du et al., **EMNLP 2025 Findings** | P2 | v1 2025-10-06, fetched 2026-09-17 | The strongest causal result: **13.9%–85%** degradation with retrieval held perfect, persisting when irrelevant content is whitespaced or masked. Recitation mitigates up to 4% |
| `ctx-rot` | [Context Rot](https://www.trychroma.com/research/context-rot) — Hong, Troynikov, Huber (Chroma) | P3 | 2025-07-14, fetched 2026-09-17 (301 from `research.trychroma.com/context-rot`) | 18 models; non-uniform degradation; **"Even a single distractor reduces performance"**; coherent haystacks are *harder* than shuffled ones. Its curves are figure-only — see unverified |
| `lost-middle` | [Lost in the Middle](https://arxiv.org/abs/2307.03172) — Liu et al., **TACL** | P2 | v1 2023-07-06, v3 2023-11-20, fetched 2026-09-17 | The original U-shape: "performance is often highest when relevant information occurs at the beginning or end" |
| `lim-emergent` | [Lost in the Middle: An Emergent Property from Information Retrieval Demands](https://arxiv.org/html/2510.10276v1) — Salvatore, Wang, Zhang (Rutgers) | P2 | 2025-10-11, fetched 2026-09-17 | That the U-shape is **explained, not refuted** — an adaptation to primacy/recency training demands |

### Multi-turn, agentic and long-horizon

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `multiturn` | [LLMs Get Lost In Multi-Turn Conversation](https://arxiv.org/abs/2505.06120) — Laban, Hayashi, Zhou, Neville | P2 | 2025-05-09, fetched 2026-09-17 | **39% average drop** multi-turn vs single-turn; the decomposition into minor aptitude loss + large unreliability rise; "when LLMs take a wrong turn… they get lost and do not recover" |
| `coding-rot` | [When and How Context Rot Appears in Coding Agents](https://arxiv.org/html/2607.17937) — Yue Xue (CertiK) | P2 | v2 2026-08-01, fetched 2026-09-17 | Audit success **80% → 30%** at ~300K chars while coverage held at **92–94%**; authors "cannot identify a universal or monotonic onset" |
| `premature` | [Diagnosing and Mitigating Context Rot in Long-horizon Search](https://arxiv.org/abs/2606.29718) — Xia et al. | P2 | 2026-06-29, rev 2026-08-04, fetched 2026-09-17 | Models "give up… long before exhausting the context window"; premature-termination rates rise with length controlling for query complexity |
| `single-agent` | [Single-Agent LLMs Outperform Multi-Agent Systems… Under Equal Thinking Token Budgets](https://arxiv.org/html/2604.02460) — Tran & Kiela (Stanford) | P2 | v2 2026-04-11, fetched 2026-09-17 | Budget-controlled single-agent ≥ every multi-agent variant — **except** "in highly degraded contexts". The empirical trigger for fan-out |
| `locobench` | [LoCoBench-Agent](https://arxiv.org/html/2511.13998v1) — Qiu et al., Salesforce AI Research | P2 | 2025-11-17, fetched 2026-09-17 | **The counter-witness**: flat 0.71–0.75 comprehension from 10K to 1M, "architectural parity". Confounded by its own admission and graded rather than strict metrics |

### Compaction, measured

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `complexity-trap` | [The Complexity Trap: Simple Observation Masking Is as Efficient as LLM Summarization](https://arxiv.org/abs/2508.21433) — Lindenbauer et al. (JetBrains), **DL4Code @ NeurIPS 2025** | P2 | v3 2025-10-27, fetched 2026-09-17 | Masking "halves cost relative to the raw agent while matching, and sometimes slightly exceeding, the solve rate of LLM summarization" |
| `compactionrl` | [CompactionRL](https://arxiv.org/html/2607.05378) — Li et al. (Tsinghua) | P2 | 2026-07-06, fetched 2026-09-17 | Compaction can **raise** Pass@1 (59.8 → 66.8 SWE-bench Verified); **summariser quality alone is worth 6.5 points** |
| `beyond-compaction` | [Beyond Compaction: Structured Context Eviction for Long-Horizon Agents](https://arxiv.org/abs/2606.11213) — Semenov & Dorofeev | P2 | 2026-05-01, fetched 2026-09-17 | 89 tasks / 80M tokens in one session with "no measurable degradation in task accuracy relative to per-task isolated sessions" |
| `cat` | [Context as a Tool](https://arxiv.org/abs/2512.22087) — Liu et al. | P2 | 2025-12-26, fetched 2026-09-17 | Agent-invoked compaction at milestones reaching 57.6% SWE-Bench-Verified under a bounded budget |

### Composition, safety and cost

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `classifier-rot` | [Classifier Context Rot: Monitor Performance Degrades with Context Length](https://arxiv.org/html/2605.12366v1) — Martin & Roger (Anthropic) | P2 | 2026-05-12, fetched 2026-09-17 | **2×–30×** monitor recall degradation; Opus 4.6 thinking **99.7% → 69%** from 100K to 800K; monitors "particularly weak" in middle positions |
| `longpibench` | [LongPIBench](https://arxiv.org/html/2608.28411) — Liu, Jia, Gong, Jia | P2 | v1 2026-08-28, fetched 2026-09-17 | Injection at middle/end beats front (code review **0.53 → 0.87** ASR); defences decay with length. Long benign content hides an injection rather than diluting it |
| `prompt-scale` | [Prompt Design at Scale](https://arxiv.org/html/2607.19257) — Netanel Eliav | P2 (single-author preprint — weight accordingly) | 2026-07, fetched 2026-09-17 | "Perfect-response rate collapses to zero by N=80"; refusal rate **0% → 89.6%** near context ceilings; format overhead (tables 1.367× plain text) |
| `cache` | [Don't Break the Cache](https://arxiv.org/abs/2601.06007) — Lumer et al. | P2 | 2026-01-09, rev 2026-01-31, fetched 2026-09-17 | Caching cuts cost **41–80%** and TTFT **13–31%**; naive full-context caching backfires; keep dynamic content out of the cached prefix |
| `locodiff` | [LoCoDiff Benchmark](https://abanteai.github.io/LoCoDiff-bench/) — Mentat AI / Abante AI | P4 | released 2025-05-08, fetched 2026-09-17 | A coding-shaped curve on a state-tracking task: Sonnet 4.5 **96% → 64%** from ~2K to ~98K tokens |
| `aa-lcr` | [AA-LCR v1.1](https://artificialanalysis.ai/evaluations/artificial-analysis-long-context-reasoning) — Artificial Analysis | P3 | fetched 2026-09-17, **no last-updated date on page** | Current independent 10K–100K standings (Kimi K3 88.7%) |
| `fictionlive` | [Fiction.liveBench description](https://epoch.ai/benchmarks/fictionlivebench) — Epoch AI | P3 | fetched 2026-09-17 | The methodology — and that Epoch's page carries **no results table**, so per-length scores must come from fiction.live directly |

---

## 3. Corpus — practice in the wild

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `manus` | [Context Engineering for AI Agents: Lessons from Building Manus](https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus) — Yichao 'Peak' Ji | P4 | 2025-07-18, fetched 2026-09-17 | File-system-as-context ("unlimited in size, persistent by nature"); todo.md **recitation**; the 100:1 input:output ratio and KV-cache economics |
| `cognition` | [Don't Build Multi-Agents](https://cognition.com/blog/dont-build-multi-agents) — Walden Yan, Cognition | P5 | 2025-06-12, fetched 2026-09-17 | The "share full agent traces" principle; context engineering as "the #1 job". No numbers |
| `langchain-ce` | [Context Engineering for Agents](https://www.langchain.com/blog/context-engineering-for-agents) — LangChain | P5 | 2025-07-02, fetched 2026-09-17 | The write / select / compress / isolate vocabulary; independently attributes ~95% auto-compact to Claude Code |
| `amp-handoff` | [Handoff (No More Compaction)](https://ampcode.com/news/handoff) — Amp / Sourcegraph | P4 | 2025-10-23, fetched 2026-09-17 | A major harness **removed compaction** — "It's lossy, for one"; compaction "encourage[s] long, meandering threads… stacking summary on top of summary" |
| `amp-guide` | [Context Engineering — Amp](https://github.com/ampcode/amp-examples-and-guides/blob/main/guides/context-management/Context%20Engineering%20-%20Amp.md) | P5 | live, fetched 2026-09-17 | That Amp's own guide deliberately states **no numeric threshold** |
| `cursor-ddc` | [Dynamic context discovery](https://cursor.com/blog/dynamic-context-discovery) — Jediah Katz, Cursor | P3 | 2026-01-06, fetched 2026-09-17 | The cleanest measured practice number: **46.9%** total-token reduction from deferred MCP tool loading; tool output to files |
| `factory` | [Compressing Context](https://factory.com/news/compressing-context) — Theo Luan, Factory | P3 | 2025-07-21, fetched 2026-09-17 (307 from factory.ai) | The anchored rolling-summary pattern; "A 50% increase in average context length translates directly to 50% higher inference cost" |
| `ace-fca` | [Advanced Context Engineering for Coding Agents](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/ace-fca.md) — Dex Horthy, HumanLayer | P5 | 2025-08, fetched 2026-09-17 | The **earliest traceable "40%-60% utilisation"** prescription, hedged "(depends on complexity of the problem)"; research→plan→implement; "a bad line of a **plan** could lead to hundreds of bad lines" |
| `ness` | [One session per task](https://willness.dev/blog/one-session-per-task) — Will Ness | P5 | 2026-01-24, fetched 2026-09-17 | The 0–40 / 40–70 / 70%+ zone model — **and that it is offered as "a mental model" with no citation**, on a simulation whose parameter is "Each file **read** uses 5% of the context window". The folklore exhibit |
| `agentpatterns` | [Context Window Management: the Dumb Zone](https://agentpatterns.ai/context-engineering/context-window-dumb-zone/) | P5 | reviewed 2026-07-22, fetched 2026-09-17 | The one *cited* competing figure — 10–20%, attributed to BABILong — contradicting the 40% folklore |
| `badlogic` | [Context Compaction Research](https://gist.github.com/badlogic/cd2ef65b0697c4dbe2d13fbecb0a0a5f) — Mario Zechner | P4 | 2025-12-02, fetched 2026-09-17 | Third-party cross-harness observation: Claude Code ~95%, Codex CLI 180k–244k, Amp none |
| `cc-6354` | [[BUG] Claude forgets everything in CLAUDE.md after compaction](https://github.com/anthropics/claude-code/issues/6354) — flux627 | P4 | opened 2025-08-22, still open, fetched 2026-09-17 | The cautionary exemplar in the observer's words: "I have to tell it to re-read this every time it compacts, otherwise it starts doing things in 'common sense' ways" |
| `cc-92949` | [[BUG] Auto-compaction re-injects the stale CLAUDE.md copy](https://github.com/anthropics/claude-code/issues/92949) | P4 | 2026-09-08, open, fetched 2026-09-17 | That the post-compaction instruction-loss class is **still live a year later** |
| `ghsearch` | GitHub code search, run 2026-09-17 | P4 | 2026-09-17 | The distribution claim: ~42,112 repos set `permissions` in `.claude/settings.json`; only **199** set `MAX_MCP_OUTPUT_TOKENS` and **4** set `DISABLE_AUTOCOMPACT`. Handoff-flavoured commands (~6,064) outnumber compaction ones (~3,408) |

---

## 4. Implementations — what the harnesses do

Read from source where possible. **Every threshold below carries its denominator**, because no two
harnesses measure against the same quantity (see the conflicts section).

| Key | Source | Tier | Date | What it settles |
|---|---|---|---|---|
| `cc-binary` | `@anthropic-ai/claude-code@2.1.274` → `@anthropic-ai/claude-code-linux-x64` shipped native binary (Bun standalone, ~230MB) | P1 | fetched 2026-09-17 | **The decisive artifact.** `zPe(e,n){let r=e-13000 …}` — the threshold is `effective_window − 13,000` tokens, **absolute, not a percentage**; the `Math.min` clamp making `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` lower-only; the SDK-vs-REPL `precomputeBufferFraction` table; `source: ["env","settings","clientdata","experiment","model-default",…]`; **reactive mode**; the blocking limit at `window − 3,000` |
| `cc-npm` | `package.json` + `install.cjs` of the same package | P1 | fetched 2026-09-17 | The package is now a native-binary installer, not a JS bundle — why third-party `cli.js` greps no longer reproduce |
| `cc-envvars` | [Environment variables](https://code.claude.com/docs/en/env-vars) | P1 | live, fetched 2026-09-17 | `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` is first-party and lower-only — "the variable can't raise the threshold, so values above the default percentage are ignored" |
| `cc-settings` | [Settings reference](https://code.claude.com/docs/en/settings-reference) | P1 | live, fetched 2026-09-17 | `autoCompactEnabled`, `autoCompactWindow` exist as settings keys |
| `cc-86863` | [[BUG] Auto-compact threshold silently regressed — fires at ~73% context used, previously ~83%](https://github.com/anthropics/claude-code/issues/86863) | P4 | fetched 2026-09-17 | A user-observed percentage drift, labelled `bug`+`regression`. The absolute-buffer mechanism in `cc-binary` gives it a plausible cause |
| `gemini-code` | [`chatCompressionService.ts`](https://raw.githubusercontent.com/google-gemini/gemini-cli/main/packages/core/src/context/chatCompressionService.ts), gemini-cli @ `6a466a7` | P1 | read 2026-09-17 | `DEFAULT_COMPRESSION_TOKEN_THRESHOLD = 0.5`, `COMPRESSION_PRESERVE_THRESHOLD = 0.3`; trigger measured against `tokenLimit(model)` |
| `gemini-docs` | [Gemini CLI configuration](https://google-gemini.github.io/gemini-cli/docs/get-started/configuration.html) | P1 | fetched 2026-09-17 | Published default **0.7** — contradicted by the code |
| `gemini-pr` | [PR #13517 "Change default compress threshold to 0.5 for api key users"](https://github.com/google-gemini/gemini-cli/pull/13517) | P1 | merged 2025-11-20, fetched 2026-09-17 | Provenance of the 0.7→0.5 move, and the unresolved question of whether it is scoped to API-key users |
| `opencode-src` | [`session/overflow.ts`](https://github.com/anomalyco/opencode) + `session/compaction.ts` @ `5a83358` | P1 | read 2026-09-17 | `COMPACTION_BUFFER = 20_000`; trigger `count >= usable` — **100% of the input budget**; `TOOL_OUTPUT_MAX_CHARS = 2_000`. Repo moved from `sst/opencode` |
| `cline-src` | `sdk/packages/core/src/extensions/context/compaction-shared.ts`, cline/cline @ `27fe60d` | P1 | read 2026-09-17 | `COMPACTION_TRIGGER_RATIO = 0.9` × `CONTEXT_WINDOW_INPUT_RATIO = 0.9` → **effective ≈0.81**, a number that exists as no constant; `TOOL_RESULT_CHAR_LIMIT = 2_000` |
| `cline-docs` | [Auto Compact](https://docs.cline.bot/features/auto-compact) | P1 | fetched 2026-09-17 | States **no threshold at all**; confirms non-Claude models silently fall back to truncation |
| `cline-legacy` | `.clinerules/cline-overview.md` + `apps/vscode/src/sdk/legacy-task-handling.ts` @ `27fe60d` | P1 | read 2026-09-17 | The old `contextWindow − 40_000` / ×0.8 rule is **legacy** — a live citation hazard |
| `roo-src` | `src/core/context-management/index.ts` + `src/core/condense/index.ts`, RooCodeInc/Roo-Code @ `b867ec9` | P1 | read 2026-09-17 | `TOKEN_BUFFER_PERCENTAGE = 0.1`; dual trigger; **non-destructive** 0.5 truncation fallback; `MIN_CONDENSE_THRESHOLD = 5` vs UI slider `min={10}`. Path moved from `src/core/sliding-window/` |
| `roo-docs` | [Intelligent Context Condensing](https://roocodeinc.github.io/Roo-Code/features/intelligent-context-condensing) | P1 | fetched 2026-09-17 (301 from `docs.roocode.com`) | Slider **defaults to 100%** — the advertised knob is inert out of the box; the ContextWindowProgress bar |
| `codex-src` | `codex-rs/protocol/src/openai_models.rs`, openai/codex @ `fcf0545` | P1 | read 2026-09-17 (delegated, not re-verified) | `(context_window * 9) / 10` and `default_effective_context_window_percent() = 95` — the default OpenAI's docs decline to publish |
| `goose-src` | `crates/goose-context-management/src/lib.rs`, block/goose @ `d213a3b` | P1 | read 2026-09-17 (delegated) | `pub const DEFAULT_COMPACTION_THRESHOLD: f64 = 0.8;` against `current_tokens / context_limit` |
| `zed-src` | `crates/agent_settings/…`, zed-industries/zed @ `b9419ae` | P1 | read 2026-09-17 (delegated) | `Percentage(0.9)` of `max_input_tokens`; `MIN_COMPACTION_CONTEXT_WINDOW: u64 = 80_000` makes the setting **silently inert** below 80K; UI warns at 0.8 |
| `crush-src` | `internal/agent/agent.go`, charmbracelet/crush @ `ef7bae1` | P1 | read 2026-09-17 (delegated) | `largeContextWindowThreshold = 200_000`, `largeContextWindowBuffer = 20_000`, `smallContextWindowRatio = 0.2` — measured in tokens **remaining**; entirely undocumented; strict `>` creates a cliff at exactly 200,000 |
| `continue-src` | `extensions/cli/src/compaction.ts`, continuedev/continue @ `5522c6f` | P1 | read 2026-09-17 (delegated) | `AUTO_COMPACT_BUFFER_CAP = 15_000`; real trip point ≈**60.5%** at 200K while an in-code comment says "80% threshold"; displayed % counts only history, the trigger counts history + system + tool defs |
| `aider-src` | `aider/models.py` + `aider/repomap.py`, Aider-AI/aider @ `5dc9490` | P1 | read 2026-09-17 (delegated) | `max_chat_history_tokens = clamp(max_input/16, 1024, 8192)`; repo map is `clamp(max_input/8, 1024, 4096)` = **4096**, not the documented "1k" |
| `cursor-hooks` | [Hooks](https://cursor.com/docs/hooks) | P1 | fetched 2026-09-17 (308 from `docs.cursor.com`) | `preCompact` exposes `context_usage_percent` / `context_window_size` but **cannot block**; the `85` in the payload is an **example, not a threshold** |
| `windsurf-memories` | [Cascade memories](https://docs.devin.ai/desktop/cascade/memories) | P1 | fetched 2026-09-17 (307 from `docs.windsurf.com`) | Windsurf's docs are now inside Devin's; memories at `~/.codeium/windsurf/memories/`; **"Memories apply to the legacy Cascade agent only"** — not the default agent |
| `copilot-cli` | [GitHub Copilot CLI context management](https://docs.github.com/en/copilot) | P1 | fetched 2026-09-17 (delegated) | ~80% compaction, pause-and-wait ~95%; the coding agent publishes a 59-minute wall clock instead of a token threshold |
| `kilo-src` | [Kilo-Org/kilocode](https://github.com/Kilo-Org/kilocode) @ `a7470d6` — `packages/opencode/src/session/overflow.ts`, `…/kilocode/session/overflow.ts`, `packages/core/src/v1/config/config.ts` | P1 | read 2026-09-17 | **Kilo Code is now a fork of OpenCode, not Roo Code** (stated in its own README) — which is why the Roo constants are gone. `COMPACTION_BUFFER = 20_000`; `threshold_percent` has **no default** and is inert unless set; `FACTOR = 1.3` token-estimate inflation |
| `kilo-docs` | `packages/kilo-docs/pages/getting-started/cost-controls-and-usage-safeguards.md` vs `…/customize/context/context-condensing.md`, same repo | P1 | read 2026-09-17 | Two docs pages in one repo disagree: one says `threshold_percent` is "unset", the other annotates it "(default: ~80%)". **No 80 exists in the compaction path** |
| `roo-default` | `src/core/webview/ClineProvider.ts`, RooCodeInc/Roo-Code @ `b867ec9` | P1 | read 2026-09-17 | `autoCondenseContextPercent: stateValues.autoCondenseContextPercent ?? 100` — the 100% default resolved in code, confirming the slider is inert out of the box |
| `cursor-forum` | [Compaction not happening soon enough](https://forum.cursor.com/t/compaction-not-happening-soon-enough/149490) · [Summarizing at 10-20%](https://forum.cursor.com/t/context-keeps-summarizing-at-10-20-of-total-context-window/163850) | P4 | 2026-01-21/22 and undated, fetched 2026-09-17 | **Cursor staff on record**: "This is a known issue with auto-summarization. It can trigger late or incorrectly", advising manual `/summarize` at "70 to 80%". A separate user measures firing at 10–20%, unreconciled with staff's explanation |
| `cursor-subagents` | [Subagents](https://cursor.com/docs/subagents) · [Context usage breakdown](https://cursor.com/changelog/05-06-26) | P1 | fetched 2026-09-17 | Per-subagent context windows, summary-only return, "roughly five times the tokens" for five in parallel; the context ring's per-category percentage breakdown |
| `swe17` | [SWE-1.7](https://cognition.com/blog/swe-1-7) — Cognition | P3 | 2026-07-08, fetched 2026-09-17 | **Self-compaction trained into the model**: "The model learns to summarize its working state and resume from the summary, extending task horizons past the raw context window"; rollouts "up to six hours" |
| `devin-sonnet` | [Rebuilding Devin for Claude Sonnet 4.5](https://cognition.com/blog/devin-sonnet-4-5-lessons-and-challenges) — Cognition | P3 | 2025-09-29, fetched 2026-09-17 | **"Context anxiety"**: a model aware of its own window "taking shortcuts or leaving tasks incomplete when it believed it was near the end of its window, even when it had plenty of room left" |
| `cognition-2026` | [Multi-Agents: What's Actually Working](https://cognition.com/blog/multi-agents-working) — Cognition | P3 | 2026-04-22, fetched 2026-09-17 | Reverses the 2025 single-agent thesis: "we found this technique to work best when the coding and review agents do not share any context beforehand" |
| `devin-cli` | [Devin CLI changelog](https://docs.devin.ai/cli/changelog/stable) · [Subagents](https://docs.devin.ai/cli/subagents) · [Lifecycle hooks](https://docs.devin.ai/cli/extensibility/hooks/lifecycle-hooks) | P1 | to 2026-09-16, fetched 2026-09-17 | `agent.compaction_threshold_tokens` (**default unpublished**); a **32 KiB cap on always-on rule files**; the `PostCompaction` hook for "re-injecting context that may have been lost during compaction" |
| `windsurf-plugins` | [Windsurf plugins changelog](https://docs.devin.ai/windsurf/plugins/changelog) + plugins Memories page | P1 | fetched 2026-09-17 | Cascade "extends the window by occasionally summarizing messages and clearing history" and concedes context "can be dropped without warning"; doc-vs-doc conflicts on rule caps (6k/12k vs flat 12k) and tool calls (20 vs 25) |
| `amp-200k` | [200k tokens is plenty](https://ampcode.com/notes/200k-tokens-is-plenty) · [Read bigger threads](https://ampcode.com/news/read-bigger-threads) — Amp | P5 | fetched 2026-09-17 | Amp's later position that "auto-compaction makes longer threads work well" — irreconcilable with `amp-handoff` as published |

---

## Conflicts resolved

- **Claude Code's auto-compact threshold: "~83%" vs "~95%".** Two third-party claims circulate and
  contradict each other. **Both are wrong as stated**, and resolved against the shipped binary
  (`cc-binary`): the threshold is `effective_window − 13,000` tokens — **absolute, not a
  percentage**. On a 200K window that is 93.5%; on the 967K effective window it is 98.7% of 967K but
  ~95.4% of a raw 1M. The percentage is an output of the arithmetic, not an input, which is exactly
  why field reports disagree. The `Math.min` clamp the "83%" claim asserted **is real** and is
  confirmed first-party by `cc-envvars`. A P5 rumour predicted a mechanism the P1 artifact confirms,
  while both its numbers were wrong — tiers attached, as the record requires.
- **"Keep under 40%" vs "10–20% effective utilisation".** The corpus carries an uncited 40% rule
  (`ness`) and a cited 10–20% (`agentpatterns`). Fetching the primary settles it: BABILong
  (`babilong`, NeurIPS 2024) states models "effectively utilize only 10-20% of the context". The
  **cited** number survives contact with its source; the popular one has none. See unverified.
- **Gemini CLI's compression threshold: docs 0.7 vs code 0.5.** Resolved to **0.5** against the
  source constant (`gemini-code`), with `gemini-docs` still publishing 0.7 on 2026-09-17. The
  vendor's own test asserts 0.5.
- **Aider's repo-map budget: "1k tokens".** Round 1 of the implementations axis reported the
  constructor default `map_tokens=1024` at face value; round 2 read `get_repo_map_tokens()` and found
  `clamp(max_input/8, 1024, 4096)` = **4096** on any modern model. The documented figure is **4×
  understated**. An axis correcting itself against source, recorded rather than silently amended.
- **Cursor's "85%" threshold.** Appears in a `preCompact` sample payload (`cursor-hooks`). It is an
  **illustrative value in an example**, not a documented rule, and is reported here as such
  specifically so it does not enter circulation as a threshold.
- **Server-tunable boundaries.** Round 1 flagged this as *inferred* for Gemini CLI, from an `async
  getCompressionThreshold()` with a local-first check. Round 2 **verified** the same pattern in
  Claude Code's shipped binary: the resolution order includes `clientdata` and `experiment`, both
  server-side. An inference from one axis-round, confirmed as artifact in the next.
- **Claude Code's documented 967K default vs the binary's `−13,000`.** Not a contradiction: 967K is
  the *effective window*, and the threshold sits 13,000 tokens below it. The docs publish the window
  and never publish the buffer, so a user reading every published page still cannot compute when
  their session compacts.

- **Kilo Code's lineage, and its "~80%" default.** Round 2 reported the Roo-derived constants as *not
  located* after a repo restructure. A later pass found the reason: **Kilo Code is now a fork of
  OpenCode, not Roo Code**, stated in its own README (`kilo-src`) — the constants are not missing,
  they were never carried over. Its real trigger is OpenCode's absolute `input_limit − min(20,000,
  max_output)`, and `threshold_percent` has no default at all. Its own cost-controls doc annotates
  that key "(default: ~80%)", a number that **exists nowhere in the compaction path** (`kilo-docs`).
  Recorded rather than silently amended, because the first report's caution was correct and its
  inference was not.
- **Roo Code's 100% default, confirmed in code.** Round 1 took it from the docs; a later pass found
  the resolution itself — `autoCondenseContextPercent ?? 100` (`roo-default`). Documentation and
  source agree, which in this dossier is worth noting on its own.

## Conflicts left open

- **LoCoBench-Agent vs the rest of the evidence.** `locobench` reports flat 0.71–0.75 comprehension
  from 10K to 1M and "architectural parity", against `nolima`, `babilong`, `length-alone`, `ctx-rot`
  and `coding-rot`. **Not averaged.** Three plausible reasons, none established: it scores a graded
  0–1 comprehension metric rather than strict success (and `coding-rot` shows coverage holding at
  92–94% while strict success falls to 37.5% — the exact shape that would produce this); it is a
  tool-using agent that retrieves rather than holding 1M tokens in-window, so it may measure harness
  quality; and the authors themselves note larger codebases "typically feature more explicit
  architectural documentation", confounding difficulty with length.
- **Amp: has compaction returned?** `amp-handoff` (2025-10-23) announces removal — "We have removed
  compaction from Amp" — and remains live and un-annotated. `amp-200k` (Dec 2025) states
  "auto-compaction makes longer threads work well". No post announces a reinstatement. Amp's manual
  is auth-gated, so the current behaviour could not be verified either way.
- **Does the 0.7→0.5 Gemini change apply to everyone?** `gemini-pr`'s title scopes it to "api key
  users", but the constant read in `gemini-code` is unconditional. Either the conditionality lives in
  the part of `getCompressionThreshold()` that could not be read, or the PR description overstates
  the narrowing.
- **NoLiMa paper vs repo.** The paper (13 models) says "11 models drop below 50%"; the updated repo
  (22 models) says 10. Different populations; do not merge the sentences.
- **LoCoDiff's own page contradicts itself.** It still carries "All models drop to under 50% accuracy
  when prompts are just 25k tokens long" while its current leaderboard shows Sonnet 4.5 at 82% in the
  21–35K quartile. The *shape* is robust; that sentence is stale.
- **Cursor's trigger has regressed in both directions and staff concede it.** Docs say summarisation
  fires when the window "fills up"; staff say "it can trigger late or incorrectly" and advise manual
  intervention at 70–80%; a user measures it firing at 10–20% and staff attribute that to search tools
  flooding context rather than a premature trigger (`cursor-forum`). The user's measurement and the
  staff explanation are **not reconciled**. Cursor's threshold remains unpublished.
- **Cognition has reversed its own multi-agent thesis without retracting it.** `cognition` (2025-06)
  argues against multi-agents; `cognition-2026` argues the opposite and grounds it in context —
  "having a clean context makes the agent smarter because of the math of attention". Both live,
  framed as evolution rather than correction. The 2026 position is the one consistent with
  `single-agent`'s degraded-context exception.
- **Copilot: 80% (CLI, documented) vs a third-party 95% (agent mode).** These are different products,
  so this may be a real per-product difference rather than an error. Unresolved.

## Explicitly unverified

- **The 40% rule has no measurement behind it.** Its hardest statement (`ness`) is introduced as "a
  mental model", carries no citation, and rests on a simulation whose own parameter is "Each file
  **read** uses 5% of the context window". Its earliest traceable ancestor is experiential —
  `ace-fca`'s "40%-60% range", hedged "(depends on complexity of the problem)". **Folklore.** The
  direction it points is supported; the number is not. Do not cite 40% as a finding.
- **Chroma's degradation curves.** `ctx-rot`'s prose is qualitative ("degrades", "non-uniform"); no
  prose sentence giving an accuracy-vs-token-count pair could be extracted. Anyone citing "Chroma
  showed X% at Y tokens" is reading a figure. The qualitative claims and the distractor finding are
  verified; the curves are not.
- **"Summarisation lengthens trajectories 13–15%."** Widely attributed to the JetBrains work
  (`complexity-trap`); not locatable in the abstract and not verified in the full text. Do not cite.
- **Fiction.liveBench per-length scores.** Epoch's page carries methodology but **no results table**;
  the commonly-quoted figures reached the evidence axis only via search snippets. Unverified.
- **Quadratic TTFT-vs-context curves.** Textbook and widely repeated, but no primary paper with a
  measured curve was fetched. The defensible cost/latency numbers are `cache`'s 41–80% and 13–31%.
- **Beyond Compaction's "$55 per run / 20–70% cost reduction"** — search snippet only, not found in
  the fetched abstract.
- **Whether harness auto-compaction specifically costs accuracy.** **No primary source exists.** The
  literature compares *engineered* compaction against *no* compaction (`complexity-trap`,
  `compactionrl`, `beyond-compaction`); nobody has published the negative control a harness user
  actually wants. This is the largest gap in the dossier.
- **Kilo's documented "32,000 token" single-window reserve** — asserted in its docs, not traceable to
  a constant in `overflow.ts`.
- **Cursor's, Windsurf's and Devin Cloud's numeric thresholds** — closed source; none published. The
  only named knob in that family is Devin CLI's `agent.compaction_threshold_tokens`, whose default is
  also unpublished (`devin-cli`).
- **The units of Windsurf's context "meter" and Devin CLI's `/context`** — described in changelogs,
  never specified. Do not infer "percentage" from "meter".
- **Whether Devin Cloud surfaces context usage at all** — no doc or release note describes one; the
  cloud UI surfaces ACUs, which are not context.
- **Devin's "keep sessions under 10 ACUs" guidance** (2025-01) — still live, but published before the
  six-hour self-compacting rollouts of `swe17`. Whether it reflects 2026 behaviour is unverified.
- **Amp's current threshold, config key and usage display** — manual is auth-gated (302 to
  `authapi.ampcode.com`).
- **Cursor's, Windsurf's and Devin's thresholds** — closed source, and no number is published for
  any of them.
- **Claude Code's `precomputeBufferFraction` table values** and the concrete SDK-vs-REPL difference —
  present in the binary, not extracted.
- **Whether `967K` is exact.** Anthropic's own prose hedges: "at about 967K tokens".
- **`cc-86863`'s cause.** The absolute-buffer mechanism plus remotely tunable windows would produce
  exactly the reported symptom, but this is offered as a **hypothesis**, not a confirmed diagnosis.
