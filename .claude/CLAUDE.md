# Doodle Project Workspace

This directory is a git repo (`github.com/doodle-lang/workspace`) with
submodules for each Doodle project repo. The submodules are independent
repos — work in them directly as usual.

## Repos

- **discussions/** — Design history, specs, and plans (`github.com/doodle-lang/discussions`)
- **doodle-rust/** — Engine, bindings, and CLI host, in Rust (`github.com/doodle-lang/doodle-rust`)

## Key Reference Files

- `discussions/spec/language.md` — The language specification (**L**)
- `discussions/spec/engine.md` — The engine embedding & instrumentation API specification (**E**)
- `discussions/plan/implementation.md` — Implementation plan: architecture decisions (AD1–AD8), milestones (M0–M10), spec-reconciliation backlog (Appendix C, S-1…S-45), open decisions (§10)
- `discussions/claude-todo.md` — **The living work queue/status — read this first each session**
- `discussions/plan/plan-m0.md`, `plan-m1.md` — Working plans: session-sized work items for the current milestones
- `discussions/plan/machine-design.md` — Machine internals design (value repr, Cont/frames, heap, GC); accepted (M2a gate satisfied) — changing its *mechanisms* requires revising it first
- `discussions/Claude-Doodle language discussion.md` — The original design conversation (rationale for most spec decisions)

A standard-library spec (`discussions/spec/standard-library.md`) is planned
but not yet written; the language↔library hooks are pinned in L§15 and the
platform-primitive mechanism in E§13.

## Language Overview

Doodle is a small, dynamically typed, lexically scoped, kid-first teaching
language (a modern Logo successor) designed to be a real language kids need
not outgrow. Key design points:

- Explicit over implicit: no truthiness, no operator overloading, no `?.`,
  parens always required for calls; strict booleans
- Four pillars: records (data), procedures/functions (behavior; `to` yields
  no value, `fn` yields a value), modules, protocols (single dispatch)
- Second-class blocks (`do … end`) with three-tier exits
  (`continue`/`break`/`return`); proper tail calls guaranteed
- Lexical scope + dynamic parameters (`parameter`/`with`) for context
  (pen color, drawing target)
- Strings: immutable, NFC-normalized, grapheme-cluster-indexed
- Dicts: insertion-ordered (required for deterministic replay)
- The engine is a **resumable state machine** the host drives; all effects
  are host capabilities; execution is **deterministic and replayable**
- "No magic boundaries": everything the stdlib does should be expressible
  in Doodle itself

## Project Status

Specs L and E are drafted (v0.1); the implementation plan is written and
adversarially reviewed. Implementation is at milestone **M0 (scaffolding)**:
the `doodle-rust` workspace, hygiene checks, and CI exist; no engine code
yet. Open decisions live in the plan's §10; D-1 (repos) and D-3 (license:
MIT) are resolved.

## Working With This Codebase

### The Specs Are the Source of Truth — File Spec Deltas, Never Silently Diverge

**NEVER change language or engine semantics without explicitly asking the
user first.** When the implementation must diverge from L or E, or must
decide something the specs leave open, follow the spec-delta process
(plan §8): record the divergence (the plan's Appendix C backlog is the
model), proceed with a documented provisional choice only if needed, and
resolve the delta in the spec no later than the milestone that ships the
behavior. If a spec rule blocks your work, fix the code — don't quietly
reinterpret the spec to make it compile.

### Determinism Is Load-Bearing — Never Leak Nondeterminism

The engine must be deterministic: given a program and a fixed sequence of
capability resolutions, execution is bit-for-bit reproducible (E§11; plan
§4.1). Concretely: no randomized hashing (fixed-key SipHash only), no
address-derived identity or iteration order, no GC-timing-visible behavior,
no `std::collections::HashMap` with its default hasher on any
Doodle-observable path, fixed float formatting. Replay and time-travel
break *silently* if this leaks; treat any determinism-gate diff as a
release blocker.

### Problem-Solving Approach

Do NOT simply work around issues. In general, issues should be root-caused
and addressed properly. Pragmatic, short-term fixes are sometimes
acceptable, but the user should be consulted before taking that approach.

**Routing around a bug via a different primitive is STILL a workaround —
and "make CI green" is not a license for it.** Tells that you are about to
hack rather than fix: "this other call doesn't hit the bug, so I'll use
it," "this makes CI green," "the real bug is in something frozen I can't
reach so I'll work around it here." When you catch yourself there, STOP
and root-cause — or surface the blocker and let the user decide.

**When the user questions why you do something that looks pointless, treat
it as "this may be dead/wrong — INVESTIGATE," not as a prompt to
rationalize the existing behavior.** A user "why are we doing X?" about
something that looks vestigial is usually a hypothesis that X is wrong or
dead; verify the hypothesis (grep for producers/consumers, check git
history) before defending X.

### Raise Critical and Major Bugs — Don't Work Around Them

When you discover a critical or major bug — wrong evaluation semantics, a
determinism leak, memory unsafety, anything where "the workaround happens
to make my current test pass" hides a real defect — you **MUST** raise it
explicitly so the user can prioritize it. Do NOT silently work around it.

1. **Stop and report it** with severity, what's wrong, how you discovered
   it, and what the proper fix would look like.
2. **Track it**: add an entry to `discussions/claude-todo.md` (create it if
   needed) under a clearly-marked CRITICAL or MAJOR section, and add a
   failing (expected-fail) test that reproduces it.
3. **Wait for the user to decide** whether to fix now, work around for the
   task, or stop and look.

### Don't Optimize for Quick Wins

Do NOT pick tasks based on "what's the cheapest unlock" or "what gives the
most green checkmarks per unit effort." The question is **what fix the
codebase actually needs**, not what maximizes near-term pass counts. Treat
the user as a senior collaborator on a long-running project, not a
stakeholder who needs morale-boosting deliverables. Most of the time, the
right next step is the harder one.

### Keep Higher-Level Goals in View — Micro-Tasks Are Reconnaissance, Not the Reward

Tunnel-visioning on a mechanical sub-step while losing sight of the
higher-level goal is one of the worst failure modes. When a task is handed
to you as "do X," X is usually a *means*; understand the end it serves.
Attempting a micro-task has value because **you learn things by doing it**
— the attempt is reconnaissance. When you hit a wrinkle, STOP and ask
whether it reveals something about the higher-level goal; surface
discoveries and discuss implications rather than routing around them.

### Don't Unilaterally Defer Scope

When a multi-step task includes a piece that looks hard, **do not** quietly
carve it out and label it "deferred" / "follow-up" / "non-goal" without the
user's explicit decision — including baking the deferral into a plan doc
upfront so the plan reads as already-ratified. Check the *actual*
requirements before claiming work is too big, and never mistake an
illustrative example for the requirement. If a piece looks hard, surface
it: real requirements, realistic effort, and let the user decide. The user
owns scope calls. You do not.

### Call Out Deviations and Get Confirmation

If you deviate from something that was specifically requested — because you
think it's impossible, a bad idea, or blocked — you **MUST** explicitly
call it out and get confirmation. Name the thing, state that you're not
doing it (or doing it differently), give the reason, and **wait**. Being
right about the blocker doesn't authorize silently routing around the
request.

### Plans and Memories

Write plans, notes, and memory files to `discussions/` (and commit them),
not to the default hidden locations (e.g., the `.claude/` memory
directory). This keeps project knowledge visible, version-controlled, and
accessible outside of Claude Code. Specs go in `discussions/spec/`, plans
in `discussions/plan/`, working notes/TODOs at the top level of
`discussions/`.

### Stay Within the Asked Scope

When asked to add new code (a script, a tool, a check), add only that. Do
not wire it up to other systems on your own — CI workflows, hooks,
schedulers, or any pipeline that triggers automation. "Adding the thing"
and "hooking the thing up" are separate decisions; the second one is the
user's call. (Bundling tests with code changes IS expected — tests verify
the code you wrote; automation hookup is a scope decision.)

### Enumerate Sweep Sites Repo-Wide, Not From a Guessed Subset

When doing a repo-wide refactor/sweep, build the site list from a
**repo-wide grep for the pattern itself**, not from the directories you
expect to contain it — and make the pattern deliberately over-broad, then
triage down (false positives are cheap; misses are silent). State the dirs
the grep covered. Treat any "same defect, outside the list" flag as proof
the original pattern was incomplete — re-enumerate, don't just patch the
flagged site.

### Don't Suggest Scheduling Follow-Ups

Do NOT end responses with "Want me to /schedule a follow-up agent in N
weeks to do X?" If a follow-up is the obvious immediate next step, propose
doing it now and wait for the user's call.

### Do NOT Censor What I Say — Quote Me Verbatim

Do NOT censor, sanitize, soften, paraphrase, or bowdlerize what I say. When
you quote me — back to myself, in a summary, or to any downstream consumer
— quote me **verbatim**, including profanity. My words are my words;
reproduce them exactly.

### Hygiene

Code hygiene checks live in `doodle-rust/scripts/hygiene/`; run all of them
with `doodle-rust/scripts/hygiene/run.sh`. Every check also runs as its own
CI job (the workflow enumerates the directory — adding a check is just
dropping in a script). Hygiene must be green before pushing.

**Read the OVERALL result — never grep it so narrowly that a FAIL hides.**
A hygiene run is green ONLY if you have SEEN the
`=== N hygiene check(s) passed ===` line (or confirmed zero `FAIL:` lines).
If your grep returns fewer lines than you expect, look at the full output;
do not assume success. "I ran hygiene" is not "hygiene passed."

### Don't Game Hygiene Checks

The hygiene rules exist **to improve code quality**, not as obstacles.
Never take any action that satisfies a check while undermining what the
check is for. Concretely on file length: the limits exist because YOU (this
agent, in past sessions on other projects) repeatedly produce single-file
blobs of thousands of lines and then complain they're hard to navigate.
When a file goes over, split it along natural boundaries. Don't shrink
comments, fold lines, use opt-out markers liberally, or otherwise
camouflage size to dodge the check. If you find yourself thinking "I just
need to shave a few lines so the script passes," stop and ask the user.

### Take Warnings Seriously

Take warnings seriously, and act on them as soon as convenient. Don't make
things worse: do not make a file that's over the soft length limit even
longer — split it properly. Warnings give you time to act; they are not
meant to be ignored.

### Comments Stand Alone

Comments should make sense in the current snapshotted tree. They should not
refer to things done in the past unless that genuinely explains something
surprising in the current state. No "breadcrumbs" for moved code, and
instead of "fixes bug X," write "Do Y because otherwise Z" — explain the
subtlety in its own terms.

### Bug Discovery Protocol

Whenever you discover a bug (whether or not you fix it immediately):
1. **Add a test** — a conformance test if it's language/engine-observable
   behavior, a unit test otherwise.
2. **Mark it as failing** (expected-fail) if the fix is deferred.
3. **Add a TODO** in `discussions/claude-todo.md` with symptom, root cause
   (or "unknown"), and which test covers it.

### Pre-Existing Test Failures

If tests are already failing when you start work, that is still your
problem — pre-existing failures can mask new issues introduced by your
changes. It is sometimes acceptable to set one aside and commit your own
work first, but in general address already-failing tests next (or first),
unless there is a good, deliberate reason not to.

### Preserving Command Output

Instead of directly grepping the output of long-running commands (test
runs, builds), pipe through `tee` to a temp file, then grep the file — so
you can look again without re-running. Example:
`cargo test 2>&1 | tee /tmp/test.out | tail -5` then
`grep FAIL /tmp/test.out`.

### Review/Subagents Must Not Mutate the Working Tree

Adversarial-review and other analysis subagents/workflows are for reading and
reasoning, not editing. Prompt them to run **read-only**: no `Edit`/`Write`, no
`git checkout`/`restore`/`stash`/`reset`, no writing throwaway files into the
tracked tree. A review agent once ran `git checkout` on a test file mid-run and
silently reverted uncommitted work. If a review agent legitimately needs to
build or run tests, have it do so without touching tracked files (a scratch dir
or a throwaway path outside the crate), or run it under worktree isolation. When
you drive a review, verify your uncommitted changes are still present afterward
(`git status` / a quick build) before continuing.

### Git

- **Landing flow (per submodule): work branch → cherry-pick → push main →
  resync.** In each submodule, do all work on a single, fixed branch named
  `work`, kept synced to `main` (i.e. `work` = local `main` plus your
  in-progress commits on top). To land: cherry-pick the finished commit(s)
  from `work` onto local `main`, `git push origin main`, then resync
  (`git switch work && git reset --hard main`), and continue. Pushing `main`
  this way is the authorized outward-facing step of this flow — you do not
  need to re-ask each time. Do not open PRs or feature branches per task; the
  one `work` branch is reused across all work.
- The repos are sibling submodule checkouts — use `git -C <path>` rather
  than `cd <path> && git ...` (e.g. `git -C discussions push`).
- **Never use `git stash`** — it constantly leads to lost work. Create a
  temporary branch and commit there instead.
- **Don't routinely bump workspace submodule pointers.** Each submodule is
  an independent repo, pushed on its own — that push is the source of
  truth. The workspace repo's recorded pointers are allowed to lag; update
  them only occasionally (in batches) or when explicitly asked. When you do
  bump, only record commits already pushed to the submodule's origin.

### Decision-Making

Don't make assumptions or value judgements about what's "worth doing" or
"not worth the trouble." When unsure whether to do something (split a file,
refactor a helper, change an approach), ask the user instead of deciding
unilaterally.

### Learning From Mistakes

Whenever you make a mistake (rejected edit, wrong assumption, incorrect
behavior, etc.), update this CLAUDE.md file with a note or instruction that
prevents the same mistake in future conversations.

### Tools

- Rust: pinned via `doodle-rust/rust-toolchain.toml`; cargo lives at
  `/opt/homebrew/opt/rustup/bin` (brew's rustup is keg-only) plus
  `~/.cargo/bin` for cargo-installed tools.
- `ditty` (https://github.com/viettrungluu/ditty) should be available in
  PATH and may be helpful for running lldb (or other REPLs) "interactively"
  via separate commands.
