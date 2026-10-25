---
name: writing-plans
description: "Use before any feature, fix or refactor that will land as a PR, and whenever the user says plan this, read a lot and plan, write a plan file, breakdown, or do the full thing. Covers investigating first, confirming understanding, decisions with defaults, the plan as the branch's first commit, executing it to the end, and keeping it true through the PR. Replaces superpowers brainstorming, writing-plans and executing-plans."
license: MIT
---

# Writing plans

A plan is the contract for a branch. It is written after reading, agreed in
chat, committed first, executed to the end, and kept true until merge.

The rules below come from months of real sessions. Each one exists because its
absence caused a specific, repeated failure.

## 1. Pick the size out loud

Say which one applies and why, in one line, so the user can override it:

| Situation | Do this |
|---|---|
| Trivial, known shape (typo, one-liner, config value) | No plan. Just do it and say so. |
| Bug with a known root cause, or a small bounded change | **Short plan** (15–80 lines). Bugs without a known cause go to `debugging` first. |
| Feature or refactor with a known shape but real design choices | **Standard plan** (80–500 lines). |
| Multi-day, cross-repo, or run by a goal loop | **Long plan**: standard + Tasks + Execution notes. |
| Unknown case set, several sources of truth must agree, an attempt stalled at 40–70%, conditionals piling up | **`staged-decomposition` first.** Then a long plan whose tasks are its stages. |

When unsure, pick the heavier form. If hidden complexity shows up mid-task,
upgrade and record it as a scope correction. Never downgrade mid-task.

## 2. Read a lot before writing anything

Under-reading is the most common reason plans get rejected.

- Read the code the change touches, whole files, not search hits.
- Read the history: related issues, PRs, review comments, earlier plans and
  decomposition docs, git log of the area. Know why the current code exists.
- Read real data: recordings, logs, transcripts, the installed binary's actual
  behaviour. Source tells shape; the corpus tells frequency. Count, don't guess.
- Subagents may fan out the reading. You still read the key material yourself.
  A plan built from someone else's summary is a plan built on a claim.

Put what you established in the plan's **Evidence** section, marked
"verified, do not re-derive", so the next session doesn't redo it.

## 3. Show your understanding, then ask once

Before the plan file exists, reply in chat with:

1. A TLDR of the outcome in the user's terms, separating what they said from
   what you assumed.
2. An ASCII sketch when there is a UI, a flow, or a before/after file tree.
3. **Decisions**, numbered, all at once, each with a recommended default and
   the reason. Offer 2–3 real alternatives only where they exist.

Rules:

- Ask in chat. Never one question at a time, never via a question/picker tool.
  The user usually answers "all defaults" or changes one number.
- Approval covers only what was shown. Agreeing to a scope is not approval of
  a plan they haven't read.
- If the user doesn't answer a decision and you proceed, mark it **UNCONFIRMED**
  in the plan and the PR so the open product call stays findable.
- Don't invent product semantics. Unknown design choices stay as decisions,
  not silent picks.

## 4. Know which mode you are in

These two modes are the most common source of correction, in both directions.

- **Plan only** ("plan this", "breakdown", "do not implement"): write the plan,
  commit it, report, and **stop**. No implementation code.
- **Full build** ("do the full thing", "go", "build it"): the plan is the first
  commit, then implement **everything** in the same run. Do not deliver a spec
  instead of the build. Do not stop at phase boundaries to check in.

If the request is ambiguous, ask in the same message as the decisions.

## 5. Write the plan

- Location: the repo's plan folder (default `docs/plans/YYYY-MM-DD-<outcome>.md`).
  A repo-local convention wins.
- It is the **first commit** on a fresh branch and worktree from the current
  remote main. Name the worktree, branch and PR after the outcome, never after
  the plan file. Commit: `docs(<scope>): plan <outcome>`.
- Use [reference/template.md](reference/template.md). Required in every size:
  header, Outcome, Change/Design, Tests, Verification, Out of scope.
- **Contracts, not code.** Name files, functions, signatures, data shapes,
  invariants and observable behaviour. Pasted implementation drifts from what
  ships and bloats the plan.
- Name what each task hands to the next (exact names and types), so a fresh
  session or subagent can execute any task cold.
- Tests name their fixture's provenance (a real recording, not an invented
  literal) and the failure they would show before the fix.
- State the **verification boundary**: what cannot be verified here (running
  the app, native/packaged behaviour, long soak) and how that will be reported.
- No length caps. Short when the change is short, exhaustive when the design is
  large. No padding, no ceremony sections nobody will read.
- Before handing it over, check once yourself: every requirement maps to a
  task, names match across tasks, no "TBD / handle edge cases / similar to
  Task N". Fix inline. No plan-reviewer subagent.

A separate spec file is only for designs the user must approve before a plan
can even be written (new subsystem, UX with open product calls).

## 6. Execute it

- Implement task by task, in coherent commits. Tests from real recorded data,
  written to fail first; prove each regression test by reverting the fix and
  watching it fail.
- **Decide instead of stalling.** Small ambiguities get a ruling recorded in
  the plan: `Ruling: <decision> — <why> — <cost if wrong>`. An unrecorded
  deviation is a secret decision.
- Stop and ask only for: a destructive or irreversible action, anything
  security-sensitive, a side effect outside the worktree (merge, push to a
  shared branch, publish), or a plan so broken every path is a guess.
- Update the plan **in the same commit** as any deviation, scope change, stage
  completion or merge with main.
- After a context compaction or in a goal loop: re-read the plan and `git log`;
  trust them over memory; continue from the first unfinished task; append to
  Execution notes before stopping.
- Run scoped checks while iterating and the full project check once at the end.

## 7. Finish

1. **Plan vs built**: walk the plan section by section and mark each MATCH /
   DRIFT / MISSING. Fix drift or record it with the reason. Silent drift from
   an agreed plan is the worst correction this workflow has recorded.
2. Review with `pr-review` (one round).
3. Open the PR with the verification you ran and the boundary you couldn't.
4. Report in a short TLDR: what is implemented, what was only investigated,
   what is UNCONFIRMED. Then **stop**. Opening a PR never authorizes a merge.

After merge the plan is left as-is; the PR and the project's memory hold the
shipped state.

## Deadline mode

If the user explicitly declares deadline mode (hackathon, demo tonight), the
review round and full test gate may be skipped. Reading first, the plan commit,
and evidence before claims still apply.

## Red flags

| Thought | Reality |
|---|---|
| "I've read enough, I'll plan now" | Have you read the history and the real data, or just the code? |
| "I'll ask the first question and wait" | Batch every decision with a default. |
| "I'll paste the implementation so the plan is complete" | It will drift. Name the contract and the invariant. |
| "Step 1 write test, step 2 run it, step 3…" | Choreography nobody ticks. Name the test, its fixture, its pre-fix failure. |
| "I'll write a realistic fixture" | Record or extract a real one. |
| "The plan is written, I'm done" | In full-build mode the plan is the first commit, not the deliverable. |
| "I'll check in after phase 1" | Full build means finish. Stop only for the four stop conditions. |
| "The code works, the plan says something else, fine" | Record the ruling or fix the drift. |
| "Let's do another review round" | One round. Anything bigger is a follow-up issue. |
| "I'll add a CI check so this never regresses" | No enforcement scaffolding. Write the WHY comment. |
| "Tests pass, so it works" | Did you run the thing the user will run? State the verification boundary. |
