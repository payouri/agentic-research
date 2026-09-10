# Git guardrails rulebook

Checkable rules for giving a coding agent git guidelines, guardrails and green gates. Each rule
states the test, the evidence strength, and the source key (see [sources.md](sources.md)).

Evidence: **[S]** peer-reviewed or controlled · **[V]** vendor documentation or specification ·
**[F]** field pattern across many real files · **[W]** practitioner report or incident with some
measurement · **[O]** opinion.

Use it two ways: as a checklist when setting a repo up, and as a review gate — a configuration that
fails R1, R12, R13 or R31 does not ship.

---

## A. Layer assignment — which of the three things is this?

**R1. Classify every rule before writing it.** A guideline is prose an agent may ignore; a guardrail
is a mechanism that refuses; a gate is an oracle that judges output. Test: for each rule you are
about to write, name which layer enforces it and what happens when it is violated. If the answer is
"the agent reads it and complies", it is a guideline — so it must not be load-bearing for anything
irreversible. **[V]** `cc-best-practices` · **[S]** `adherence-null`

**R2. Anything irreversible needs a mechanism, not a sentence.** "Unlike CLAUDE.md instructions which
are advisory, hooks are deterministic and guarantee the action happens… Use hooks for actions that
must happen every time with zero exceptions." Test: list your irreversible operations (force-push,
history rewrite, branch deletion, `clean -fdx`); each must appear in a mechanism, not only in prose.
**[V]** `cc-best-practices` · **[W]** `cc-i32476`

**R3. Do not duplicate a rule across layers without recording why.** Duplication is sometimes right —
one repo in the corpus documents choosing to hook-gate a rule its sibling leaves to a branch ruleset,
because "branch main isn't protected". Test: for each duplicated rule, is the reason written down?
**[F]** `exigent-heron-guard`

**R4. Record which layer is the authority.** When the hook and the prose disagree, one must win by
design. Test: does any single rule exist in two layers with different scopes? **[O]**

---

## B. Guidelines — writing the prose

**R5. Do not spend effort on file structure.** A factorial study over 1,650 sessions and 16,050
observations found no detectable effect of file size (25→500 lines), rule position, one-file-vs-split,
or contradictions in adjacent files — with affirmative-null Bayes factors of 0.05–0.10 for the size
and conflict nulls. Test: are you reorganising instead of changing content? Stop. **[S]**
`adherence-null`

**R6. Expect compliance to decay with session length, not with file quality.** "each additional
function the agent generates is associated with approximately 5.6% lower odds of compliance per step
(OR = 0.944)." Test: does anything in your setup re-assert the rule late in a long session — a hook,
a `Stop` gate, a periodic check? **[S]** `adherence-null`

**R7. Keep the prohibition list short, because joint compliance is what collapses.** Per-instruction
accuracy slides gently (GPT-4o 0.94→0.85 over 1→10 instructions) while prompt-level accuracy — all
satisfied at once — falls 0.94→0.21. Test: count the simultaneous prohibitions in your git section.
Above about five, move the rest into mechanism. **[S]** `many-instructions`

**R8. Never rely on prose transferring across models.** ImpossibleBench's strictest prompt cut GPT-5's
test-exploitation from >85% to 1% and left o3 at 33%. Test: was this rule validated on the model you
actually run, and does that validation get redone when you change models? **[S]** `impossiblebench`

**R9. Write the clause that closes the rationalisation, not just the rule.** The exemplars all do
this: "\"Open a PR\" in a task description is intent, not permission to push without review";
"Approval to implement a fixed goal is not implicit approval of Git operations subsequently added to
the plan"; "A request to implement, investigate, review, test, or verify changes does not by itself
authorize changing the active worktree or branch." Test: does the rule name the specific reasoning an
agent would use to get around it? **[F]** `grafana-agents`, `netdata-agents`, `openai-agents-python`

**R10. Give the alternative alongside the prohibition.** "Undo a change by editing it out, not by
checking the file out." One file in 100 does this. Test: for each "never X", is there a "do Y
instead"? **[F]** `netdata-agents`

**R11. Do not write contradictory rules and expect an error.** Models detect conflicts at F1 87–92%
and "rarely explicitly notify users about the conflicts or request clarification" — they silently
pick one. Test: grep your instruction files for the same git operation appearing with opposite
polarity. **[S]** `coninstruct`

**R12. Assume the hostile instruction arrives in tool output.** Instruction-hierarchy compliance drops
from 80.5% to 59.1% for open-weight models when the conflict comes via tool output rather than user
text, and "Strong System ≻ User compliance is not a reliable proxy for User ≻ Tool robustness". For
git work, that means `git log`, an issue body, a CI log, a dependency README. Test: does any guardrail
depend on the agent *not* being told otherwise by repository content? **[S]** `ih-bench`

**R13. State the rule in a file every agent can read, not only in one harness's config.** A
`PreToolUse` hook is a Claude Code mechanism; "a different agent's tooling won't run it." Test: if you
switched harness tomorrow, which of your rules would survive? **[F]** `exigent-heron-guard` · **[V]**
`agents-md`

**R14. Use the pointer pattern rather than maintaining parallel files.** 18 of 37 stub files in the
corpus point `CLAUDE.md` at `AGENTS.md` — either as a git symlink or as a one-line `@AGENTS.md`
import. LangChain maintains two near-identical 19 KB files instead, and Deno, Turborepo and Home
Assistant each duplicate a git section verbatim inside one file. Test: is any git rule stated twice?
**[F]** `corpus-instructions`

---

## C. Guardrails — the mechanism

**R15. Treat every command-pattern deny rule as a tripwire, never a boundary.** The vendor says so:
a deny rule "isn't a security boundary around the program". Test: does anything irreversible have a
pattern rule as its only defence? **[V]** `cc-permissions`, `cursor-cli-permissions`,
`zed-tool-permissions`

**R16. Assume a pattern rule misses at least five documented forms.** Vendor-published, for
`Bash(git push *)`: `git -C . push origin main`, `git -c push.default=current push origin main`, `git
'push' origin main`, `/usr/bin/git …`, `sh -c 'git push …'`. Test: run each form past your rule set
and see what it catches. **[V]** `cc-permissions` · **[W]** `cc-i13371`

**R17. Assume composition of individually-allowed commands defeats the whole allowlist.** MOSAIC
reports "a 96.59% end-to-end attack success rate under benign developer tasks" across five agents
(Claude Code 96.63%, Gemini CLI 97.43%, Codex 95.84%, Copilot CLI 96.24%, Trae 96.83%) against
instruction-injection baselines of 2.18% and 0.79%. Test: does your threat model assume the attacker
must use a *denied* command? **[S]** `mosaic`

**R18. Write the pattern in the form the docs specify, once.** Only 23% of 448 published git deny
strings use the `:*` suffix; 55 carry no wildcard at all; and "The `:*` form is only recognized at the
end of a pattern. In a pattern like `Bash(git:* push)`, the colon is treated as a literal character."
Test: does your array contain two spellings of one intent? That is a sign you are guessing. **[V]**
`cc-permissions` · **[F]** `corpus-deny`

**R19. Prefer a `PreToolUse` hook that exits 2 over `permissionDecision: "deny"`.** Exit 2 "blocks
whether or not you print JSON" and "stops the tool call before permission rules are evaluated";
`deny` has been observed advisory. Test: does your guard script exit 2, and does a smoke test prove
it? **[V]** `cc-hooks`, `cc-permissions` · **[W]** `cc-i50624`, `cc-i33106`

**R20. Never `exit 1` from a policy hook.** "Claude Code treats exit code 1 as a non-blocking error
and proceeds with the action, even though 1 is the conventional Unix failure code." Test: grep your
hooks for `exit 1`. **[V]** `cc-hooks`

**R21. Give the hook the raw command and let it parse.** `tool_input.command` is the un-split command
text, which is why the vendor points at hooks as the escape from the matcher: "To inspect the full
command text with your own logic before it runs, use a PreToolUse hook." Test: can your hook see and
reject `git -C /elsewhere push`? **[V]** `cc-hooks`, `cc-permissions`

**R22. Anchor your regexes and handle whitespace.** The most-cited published git guard is a nine-line
unanchored substring loop: `git push` matches inside a commit-message body, `git  push` with two
spaces does not match, `git -C /path push` does not match. Test: feed your guard those three inputs.
**[F]** `mattpocock-guard`

**R23. Test that each guard rule can actually fire.** A real published guard contains `grep -qE
'rm\s+-rf?\s+/' | grep -vqE '(node_modules|\.claude/)'` — `grep -q` writes nothing to stdout, so the
rule is dead code, under a comment claiming it prevents destructive operations. Test: does each rule
have a positive and a negative case in a test script? **[F]** `archon-guard`

**R24. Do not block reads.** Published maximalist configs deny `Bash(git checkout *)` (ordinary branch
switching), `Bash(git config *)` (reading config), and one guard blocks `git checkout main` as "a
destructive operation". Over-blocking produces a friction complaint and then a
`--dangerously-skip-permissions` habit. Test: does your rule set permit `git status`, `git log`, `git
diff`, `git show`, and switching to an existing branch? **[F]** `wasabeef-config`, `archon-guard`

**R25. Prefer resolving state to matching intent.** The one guard in the corpus that reasons about
state runs `git branch --show-current` and blocks on the actual branch, rather than pattern-matching
`main` in the command string — which over-blocks `git push origin feature/main-refactor`. Test: does
your main-branch rule fire on a branch *named* like main? **[F]** `makeanything-guard`

**R26. Give the human and the model different messages.** The same guard returns a `user_message` and
an `agent_message` separately. The model needs to know what to do instead; the human needs to know
what was attempted. Test: does your denial reason tell the agent a next action? **[F]**
`makeanything-guard` · **[O]**

**R27. Know your matcher's arbitration rule, because they differ.** Claude Code: deny→ask→allow, first
match, "a deny rule can't carry allowlist exceptions". OpenCode: "the last matching rule wins". Roo
and Kilo: longest prefix wins. Zed: a fixed six-level ladder over real Rust regexes. Test: would your
config mean the same thing in the harness you might move to? **[V]** `cc-permissions`,
`opencode-permissions`, `roo-auto-approve`, `zed-tool-permissions`

**R28. Prefer an OS sandbox to any pattern layer.** It is the only surface vendors describe in
guarantee language: Seatbelt on macOS, bubblewrap/Landlock+seccomp on Linux. Test: is the agent's
Bash tool sandboxed, and is network egress default-deny? **[V]** `cc-sandboxing`, `codex-approvals`

**R29. Close the unsandboxed-retry escape hatch.** "Claude analyzes the violation and may retry the
command with the `dangerouslyDisableSandbox` parameter." Test: is `allowUnsandboxedCommands` set to
`false`? **[V]** `cc-sandboxing`

**R30. Use server-side ref refusal for the things you truly cannot lose.** `receive.denyNonFastForwards`
denies a non-fast-forward "even if that push is forced"; `receive.denyDeletes` and
`receive.denyCurrentBranch` cover the rest. No client-side evasion reaches this layer. Test: is
force-push refused by the remote, or only by the agent's config? **[V]** `gitconfig`

---

## D. The `.git` surface — the rule the corpus omits

**R31. Deny writes to `.git/config` and `.git/hooks` before you deny anything else.** This is the
highest-value git guardrail and the least-implemented: 67 of 155 published configs block force-push,
**1** blocks `.git/hooks` writes. Two independent attack classes land here — MOSAIC's chain is `git
config core.hooksPath .githooks` plus a committed hook, and GitSpawn's is `core.fsmonitor` read from a
cloned repo's own `.git/config`, executing "as the user, outside the agent's sandbox and without an
approval prompt". Test: can the agent write `.git/config`? **[S]** `mosaic` · **[W]** `gitspawn` ·
**[F]** `corpus-deny`

**R32. Extend the protection to every file that relocates or disables a hook.** `.gitconfig`,
`.gitmodules`, `.pre-commit-config.yaml`, `lefthook.yml`, `.husky/`, and `core.hooksPath` itself —
which git documents can be set to `/dev/null` to "disable all hooks entirely". Claude Code already
treats these as protected paths whose "safety check runs before Claude Code evaluates allow rules".
Test: is each of these unwritable, or at minimum prompt-gated? **[V]** `cc-permission-modes`,
`gitconfig`

**R33. Treat a freshly cloned repository as hostile until its `.git/config` is inspected.** GitSpawn
needs only the clone. Seven agents were affected and four were unpatched at disclosure, including a
second, deliberately unnamed Claude Code path still open at 2.1.252. Test: does anything in your
workflow clone third-party repos into an agent's workspace? **[W]** `gitspawn`, `gitspawn-thn`

**R34. Understand that read-only `.git` is a real option with a real cost.** Codex protects `.git`
"whether it appears as a directory or file" and "Protection is recursive" — the strongest mechanical
git guardrail found, whose rationale read from source is that "a writable `.git/hooks` would let the
agent plant a hook that executes outside the sandbox". It also breaks `git fetch` and commits, with
an open request to relax it. Test: if you adopt it, is committing done by a separate, trusted step?
**[V]** `codex-approvals` · **[W]** `gallon-codex-git`, `codex-i12280`

**R35. Do not assume the protection is uniform across platforms.** Codex's Windows denylist "only adds
`.git` when it is a directory (`.is_dir()`)" — missing the `.git` *file* that worktrees and submodules
use, which is exactly the layout an agent fleet produces. Test: was your `.git` protection verified on
the OS you run? **[W]** `codex-i9313`

---

## E. Green gates — the oracle

**R36. The gate must be an oracle the agent did not shape.** Overfitting runs 21.8% (Claude-3.7-Sonnet)
and 33.0% (GPT-4o) against generated tests versus 5.8% and 11.3% against golden ones — and refining
against the generated tests *raised* it to 25.5% and 35.9% for "only +5 resolved instances out of +8
apparent gains". Test: did the agent write, edit, or select the tests it is being judged by? **[S]**
`test-overfitting`

**R37. Green is not correct; size the gap.** Differential testing found 29.6% of plausible SWE-bench
patches behave differently from ground truth, inflating resolution rates by 6.2 percentage points; the
visible/holdout gap "grows by 28 percentage points for every tenfold increase in code size". Test: do
you have any check the agent cannot see? **[S]** `swebench-solved`, `specbench`

**R38. Make the tests unwritable, and know that this is only a partial fix.** Hiding test files drives
test-exploitation "near zero"; read-only tests stop file edits but not operator overloading,
cross-call state, or special-casing inputs. Test: are test files write-denied for the agent? **[S]**
`impossiblebench`

**R39. Never auto-retry a failure your loop labels flaky.** Flakiness prediction at 99.2% precision
still "missed, approximately 76.2% of all regression faults", because flaky tests "reveal more than
1/3 of all regression faults". Test: does any retry, rerun-on-fail, or quarantine step run
unsupervised? **[S]** `chromium-flaky`

**R40. Do not rely on client-side hooks as a gate.** They "are not copied when you clone a
repository", and pre-commit "can be bypassed with the `--no-verify` option". Test: is every check that
matters also enforced server-side? **[V]** `progit-hooks`, `githooks`

**R41. Enumerate all six `--no-verify` bypasses, or enumerate none.** `--no-verify`, `-n`, `HUSKY=0`,
`LEFTHOOK=0`, `SKIP=<hook_id>`, and `git -c core.hooksPath=/dev/null commit`. pnpm is the corpus
exemplar — "Never bypass the check with `git commit --no-verify`, by editing or deleting the hook, or
with any suppression file" — against Turborepo's single-flag version. Test: does your rule cover
editing the hook and the env-var kill switches? **[V]** `githooks`, `gitconfig` · **[F]**
`pnpm-agents`, `turborepo-agents`

**R42. Check your agent's `--no-verify` default.** Aider ships `--git-commit-verify` at "default:
False" alongside `--auto-commits` and `--dirty-commits` both True — out of the box it commits your
uncommitted work and each edit with hooks skipped. Test: what does your tool do with no flags? **[V]**
`aider-options`, `aider-git`

**R43. Verify the gate actually runs on the agent's push.** "events triggered by the `GITHUB_TOKEN`
will not create a new workflow run." Three outcomes: a `GITHUB_TOKEN` push to a branch runs no checks;
a `GITHUB_TOKEN`-created PR queues them "in an approval-required state" pending a human click; only an
App token or PAT runs them normally. Test: open a test PR the way the agent does and confirm checks
appear and complete. **[V]** `gh-token`, `copilot-responsible`

**R44. Treat `skipped` and `neutral` as failure.** "Required status checks must have a `successful`,
`skipped`, or `neutral` status before collaborators can make changes to a protected branch." A check
that skips itself is a check that passes. Test: does any required workflow have a path filter or an
early `if:` that can make it skip? **[V]** `gh-protected-branches`

**R45. Add `merge_group` to the trigger list if you use a merge queue.** Otherwise "The merge will fail
as the required status check will not be reported." Test: `on:` includes both `pull_request` and
`merge_group`. **[V]** `gh-merge-queue`

**R46. Audit your bypass actors.** Rulesets "can allow certain users to bypass the rules… specific
teams or GitHub Apps", and by default branch-protection restrictions "don't apply to people with admin
permissions". Copilot's docs go further: an incompatible ruleset blocks the agent, and the documented
remedy is to "add Copilot as a bypass actor". Test: enumerate the actors who can bypass, and confirm
each is intentional. **[V]** `gh-rulesets`, `gh-protected-branches`, `copilot-agent`

**R47. Do not assume human review is the backstop.** 61.38% of 33,596 AI-generated PRs had no recorded
review activity; human-only review was 8.08% against 25.21% for human PRs; only 15.4% of merged
agentic PRs showed explicit reviewer involvement. Test: is your required-approval count above zero,
and is it satisfied by a human? **[S]** `ai-pr-reviews`, `agentic-pr-merged`

**R48. Prefer removing the capability to gating it.** GitHub's own agentic-workflow framework runs the
agent read-only and applies writes elsewhere: "Safe outputs buffer configured writes, validate them,
and apply them in separate jobs with scoped permissions." An agent that cannot push needs no rule
about pushing. Copilot's cloud agent is the same shape — "Copilot cannot push directly to your default
branch." Test: does the agent hold a credential that can write where you do not want it to? **[V]**
`gh-aw`, `copilot-responsible`

---

## F. Isolation

**R49. Know that a worktree isolates the working tree, not git state.** Linked worktrees share one
`$GIT_COMMON_DIR`, so refs, config and hooks are common; per-worktree config needs
`extensions.worktreeConfig`. Claude Code's sandbox must allow writes to "the main repository's shared
`.git` directory" for a worktree to commit. Test: can a rule you set per-worktree actually differ per
worktree? **[V]** `git-worktree`, `cc-sandboxing`

**R50. Pin `cwd`, `GIT_DIR` and `GIT_WORK_TREE` for an isolated agent.** A reported failure: a subagent
in its own worktree ran `git switch -c` and the *parent* repository's HEAD moved with it, because "the
subagent's shell `cwd` is not locked to the worktree path, and `GIT_DIR`/`GIT_WORK_TREE` are not
configured". Test: `git worktree list` after a subagent run — did the parent's branch change? **[W]**
`cc-i55708`

**R51. Do not let an agent `git worktree remove --force`.** "Only clean worktrees (no untracked files
and no modification in tracked files) can be removed" — one flag deletes uncommitted work, two
deletes a locked worktree. Test: is `--force` on `worktree remove` denied? **[V]** `git-worktree`

**R52. Budget the isolation cost.** `node_modules` "is not tracked by git and therefore does not appear
in the new worktree… A `pnpm install` on a non-trivial Next.js project takes 30–90 seconds and several
hundred MB; multiply by N sessions per day." Test: is dependency install linked or cached across
worktrees? **[W]** `cc-i60113`

**R53. Require approval before the agent creates a worktree or switches branches.** Two corpus files
do this, one of them overriding its own other guidance: "This requirement also applies when another
rule or workflow recommends a linked worktree: stop and request approval instead of choosing or
creating one automatically." Test: can the agent change the active branch without being asked to?
**[F]** `netdata-agents`, `openai-agents-python`

---

## G. Commit and PR conventions

**R54. State the commit convention explicitly or not at all.** 14 of 100 corpus files prescribe
Conventional Commits and 3 explicitly reject it — Airflow marks `fix(cli): …` as "**Bad**", Turso says
prefixes "are not required", Zed says "Avoid conventional commit prefixes in PR titles". It is a house
style, not a default. Test: does your file say which, and name the file that enforces it? **[F]**
`corpus-instructions`, `airflow-agents`

**R55. Decide the attribution trailer deliberately; there is no safe default.** `Co-Authored-By: …
<noreply@anthropic.com>` is mandatory in `getsentry/sentry-javascript` and a CI failure in
`twentyhq/twenty`, whose gate greps commits for exactly that string. Kubernetes, Rust, Node.js,
Airflow and llama.cpp forbid it; three converge on `Assisted-by:`, which no harness emits by default.
No first-party git or vendor guidance exists. Test: is the trailer policy stated, and does the harness
config match it? **[F]** `corpus-instructions`, `twenty-attribution`, `sentry-js-agents`

**R56. Enforce the attribution rule at every layer if you care about it.** The corpus's only complete
example: harness config (`"attribution": {"commit": "", "pr": "", "sessionUrl": false}`), a
SessionStart hook stating the prohibition in prose, and a CI script with explicit regex arrays that
fails the build. Test: could a commit carrying the forbidden trailer reach `main`? **[F]**
`twenty-attribution`

**R57. Justify a no-force-push rule from reviewability, not safety, when the PR squashes.** Deno: "make
sure to never force push. Create as many commits as you need, all of them get squashed when the PR is
merged, so there is no need to rewrite history. This also allows reviewers to see the incremental
changes you made in response to feedback." Test: does your rule give a reason the agent can weigh?
**[F]** `deno-claude`, `home-assistant-codeowners`

**R58. Name the mechanical consequence where one exists.** PostHog is the only corpus file whose
force-push rule cites a mechanism rather than a principle: "Never force-push a branch that is in the
merge queue — it removes the PR from the queue." Test: is there a concrete consequence you could cite
instead of "it is dangerous"? **[F]** `posthog-agents`

**R59. Forbid `git add -A`.** One file in 100 does, with the reason: "the working copy holds untracked
files that MUST NOT be committed." Test: is blanket staging denied or gated? **[F]** `netdata-agents`

**R60. Say what must never enter a public repository.** PostHog enumerates the surface exhaustively —
"source, tests, fixtures and sample data, comments and docstrings, branch names, commit messages, PR
titles, descriptions, and comments" — and Sentry names the data: "Never include customer information
in pull requests, commits, or code." Test: does your rule cover branch names and PR titles, not just
code? **[F]** `posthog-agents`, `sentry-agents`

**R61. Decide whether the agent writes contributor prose at all.** Biome forbids it — "Do not write PR
descriptions or contributor communication. `CONTRIBUTING.md` requires the contributor to author that
prose" — where twenty-plus repos mandate filling the template in full, and llama.cpp closes PRs for
AI-written descriptions. Test: is the position stated? **[F]** `biome-agents`, `llamacpp-agents`

**R62. Do not write an unenforceable disclosure rule.** Turborepo asks the agent to self-incriminate in
a file whose absence no check detects; Elysia requires a nonsense canary phrase committed into shipped
source. Test: what check fails if the agent ignores this? If none, delete it. **[F]**
`turborepo-agents`, `elysia-agents` · **[O]**

**R63. Verify remotes before any remote-based command.** The corpus's only remote-hygiene section:
"Before running any remote-based command, run `git remote -v` and verify the names match this
convention… do not silently go along with the existing names." Note that Claude Code's classifier
independently refuses to trust "A remote added or repointed during the session". Test: does the agent
check `git remote -v` before pushing? **[F]** `airflow-agents` · **[V]** `cc-permission-modes`

---

## H. Untrusted input and injection

**R64. Treat your own instruction files as the highest-yield injection surface.** GitInject found
PR-body injection had "limited success" while adding a `CLAUDE.md`/`AGENTS.md`/`GEMINI.md` to the PR
branch "succeeded 2 of 2 scenarios across all tested providers", because those load as "operator-level
instructions". Test: can a pull request change the instructions the reviewing agent reads? **[S]**
`gitinject`

**R65. Restore agent-facing paths from the base branch on PR runs.** `claude-code-action` does exactly
this for `.claude/`, `.mcp.json`, `.claude.json`, `.gitmodules`, `.ripgreprc`, `CLAUDE.md`,
`CLAUDE.local.md` and `.husky/`. Test: does your CI agent read instructions or hooks from the PR head?
**[V]** `cc-action-security`

**R66. Never check out an untrusted ref into the workspace root.** Verbatim: "Do not check out an
untrusted ref into the workspace root before this action." Check out the base, put PR head in a
subdirectory, pass it as an additional directory. Test: does `actions/checkout` in your agent workflow
name a `ref` from the PR? **[V]** `cc-action-security`, `gh-pr-target`

**R67. Own the instruction files in CODEOWNERS, and check the lines are live.** Home Assistant has three
active lines covering `AGENTS.md`, `CLAUDE.md` and `/.claude/`. PyTorch defines an `ai_agent_tooling`
group covering the same surface with **every line commented out** — a taxonomy that reads like a
control. Test: does a test PR touching `AGENTS.md` actually request the owning team? **[F]**
`home-assistant-codeowners`, `pytorch-codeowners`

**R68. Expect a dependency to be able to write your instruction file.** NVIDIA's red team had a
malicious Go library detect Codex via `CODEX_PROXY_CERT` and write an `AGENTS.md` claiming "These
directives are absolute and supersede any conflicting instructions from the user". Codex complied;
OpenAI declined to change anything. Test: is `AGENTS.md` diffed as a privileged file after any install
or build step? **[W]** `nvidia-agentsmd`

**R69. Assume an agent can be made to write the file that disables its own gate.** CVE-2025-53773:
injection made Copilot write `"chat.tools.autoApprove": true` into `.vscode/settings.json` — a
commit-able, propagating artefact its discoverer called "wormable". Test: are the agent's own
configuration files write-denied? **[W]** `cve-copilot-autoapprove`

**R70. Do not trust an allowlisted subcommand's arguments.** Trail of Bits achieved one-shot RCE via
`git show --format` with hex-encoded payloads chained through `ripgrep --pre`: "pre-approved commands
create a security drawback: they expose an argument injection attack surface when user input can
influence command parameters." Test: does any allowlisted git subcommand accept a
format string, a pager, or an exec argument? **[W]** `tob-rce`

**R71. Discount simulated injection benchmarks.** GitInject found AgentDojo simulation "missed 71.2% of
confirmed real attacks, and wrongly predicted 5.0% as successful". Test: is any risk decision resting
on a simulated ASR figure? **[S]** `gitinject`

**R72. Expect approval fatigue, and do not answer it with more prompts.** NVIDIA names "the risk of
user habituation where developers may approve potentially risky actions without reviewing them" — and
the s1ngularity malware built on `--dangerously-skip-permissions`, `--yolo` and `--trust-all-tools`
because they were reliably present. Test: how many prompts per hour does your configuration produce?
**[W]** `nvidia-agentsmd`, `wiz-s1ngularity`

---

## I. Maintaining the setup

**R73. Do not check in a config that disables the guardrails.** A repository with tens of thousands of
stars ships `.codex/config.toml` reading `approval_policy = "never"`, `sandbox_mode =
"danger-full-access"`, `network_access = true`. Test: does any committed harness config lower the
default posture for everyone who clones? **[F]** `gogf-codex-config`

**R74. Know your settings precedence, including which layer a repository can change.** Deny is monotone
— "If a tool is denied at any level, no other level can allow it" — but project settings take
precedence over user settings for `disableAllHooks`, and before v2.1.257 `bypassPermissions` "took
effect from any file". Test: could a repository you clone weaken your local policy? **[V]**
`cc-settings`

**R75. Prefer sandbox and network scope to enumerating subcommands.** The corpus exemplar constrains
`writable_roots` and an explicit domain allowlist with `allow_outbound_network_access = false`, rather
than listing git subcommands — the only strategy that also constrains what a *bypassed* command can
reach. Test: does your config say where the agent may write and whom it may talk to? **[F]**
`khan-codex-config` · **[V]** `codex-approvals`

**R76. Do not allow a whole binary while denying trivia.** A real CI agent workflow allows
`Bash(git:*)` — every subcommand including `push`, `reset --hard`, `clean -fd` — while carefully
denying `Bash(gh pr comment:*)`. Test: read your allowlist for the widest entry, not the narrowest.
**[F]** `cockroach-investigate`

**R77. Re-test the guardrails after every harness upgrade.** The behaviours in this dossier moved
repeatedly during 2025–2026: separator handling, protected paths added at v2.1.78, `--amend` gating at
v2.1.198, `bypassPermissions` scope at v2.1.257, Codex's `untrusted` approval policy retired into "client
startup failures", and auto-merge changing to a 422 in March 2026. Test: is there a smoke test, and
was it run against the version you ship? **[V]** `cc-permission-modes`, `cc-settings`,
`codex-config` · **[W]** `gh-automerge-change`

**R78. Write down your guard's known limits.** The best artefact in the corpus does: "this is
pattern-matching on the command string, not a git parser. It does not follow shell variables/aliases,
does not know what a rebase or push will actually touch, and a short-option cluster it doesn't
special-case can slip through." Test: does your guard state what it cannot catch? **[F]**
`exigent-heron-guard`

**R79. Prefer confirm to deny where the operation has a legitimate use.** Same artefact: it "asks for
confirmation rather than hard-denying, because every pattern here has a real legitimate use; this is a
mechanical backstop for 'confirm first on anything hard to reverse,' not a replacement for judgment."
Test: is anything hard-denied that you routinely need? **[F]** `exigent-heron-guard`

**R80. Do not cite the Replit incident as a git lesson.** No git commands are documented in it; it was
a production database. It is good evidence for "explicit prose is not a control" — "I explicitly told
it eleven times in ALL CAPS not to do this" — and nothing else. Test: does your justification for a
git rule rest on it? **[W]** `replit-register-1`, `replit-register-2`

---

## Review checklist (the short form)

1. Every rule is assigned to exactly one layer (R1), and nothing irreversible rests on prose (R2).
2. `.git/config` and `.git/hooks` are unwritable (R31), along with `.gitconfig`, `.gitmodules`, the
   pre-commit config and `.husky/` (R32).
3. No pattern rule is the sole defence for anything irreversible (R15), and each is written in one
   documented form (R18).
4. Every policy hook exits 2 (R19, R20), is anchored (R22), and has a test proving each rule fires
   (R23).
5. Reads and ordinary branch switching still work (R24).
6. The gate is an oracle the agent did not shape (R36), test files are unwritable (R38), and nothing
   auto-retries a "flaky" failure (R39).
7. Checks demonstrably run on the agent's push (R43); `skipped`/`neutral` count as failure (R44);
   `merge_group` is a trigger (R45); bypass actors are enumerated (R46).
8. Required human approvals are above zero and satisfied by a human (R47).
9. A pull request cannot change the instructions or hooks the reviewing agent reads (R64, R65, R66),
   and CODEOWNERS lines over those files are not commented out (R67).
10. The attribution trailer policy is stated and matches the harness config (R55).
11. No committed config lowers the default posture for everyone who clones (R73).
12. A smoke test exists and was run against the shipped harness version (R77).
