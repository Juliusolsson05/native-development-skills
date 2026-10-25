# Plan template

Copy the parts your size needs. Sections marked **(all)** are required even in
a short plan. The appended sections at the bottom are added during the PR, only
when they happen.

```markdown
# <Area>: <outcome as a sentence>                                   (all)

Fixes #N | Refs #N.                                                  (all)
Branch `<type>/<outcome>` · Worktree `.worktrees/<outcome>` · Base `origin/main` @ <sha> (<date>)
Status: draft for review | approved <date> (<how>) | implemented, PR #N open, merge needs explicit approval
[Spec: <path> — the spec wins where they differ.]  [Decomposition: <path> — tasks are its stages.]

## Outcome                                                           (all)
The observable end state in the user's terms. What someone will see or be able
to do when this merges.

## Evidence (verified, do not re-derive)
What reading established: file:line at the current sha, recordings, log rows,
counts over the real corpus, installed-binary behaviour, prior issues/PRs and
why earlier attempts didn't converge. For a bug: the root cause as a causal
chain (see the debugging skill).

## Decisions
1. **<question>** — *Default: <x>*, because <why>. Alternative: <y>.
2. …
Answered <date>: all defaults | per item. Unanswered and shipped → UNCONFIRMED.

## Design                                            (all; "Change" in a short plan)
Contracts: files, functions, signatures, data shapes, ownership boundaries.
Invariants: what must hold, and what breaks if it doesn't. Each one becomes a
WHY comment at the code that protects it.
Why this shape and not the obvious alternative.

## Files
- Create `path` — one-line responsibility
- Modify `path` — what changes and why

## Tests                                                             (all)
Per test: file, fixture provenance (recording / extracted from <source> /
justified literal), the observable assertion, what it shows before the fix.

## Verification                                                      (all)
Scoped checks while iterating; the project's full check once at the end; CI.
Verification boundary: what cannot be proven here and how it will be reported.

## Out of scope                                                      (all)
Each item with why it isn't here and where it goes (follow-up issue, other repo).

## Tasks                                            (long plans and goal loops)
- [ ] 1. <deliverable> — produces <artifact>; hands <names/types> to task 2; verified by <check>
- [ ] 2. …
Coarse, contract-level boxes, ticked as they land. No per-step choreography.

## Execution notes                                  (long plans; append-only, dated)
- <date> task 1: commits <sha..sha>; verified <what>; confirm-red observed | NOT observed; deviations <what, why>.

<!-- Appended during the PR, only when they happen: -->
## Rulings
- Ruling: <decision> — <why> — <cost if wrong>

## Scope correction (<date>)
What the user changed, and what is now groundwork versus completion.

## Review (<date>, one round)
| Finding | Verdict | Disposition |
|---|---|---|
| … | valid / overstated / already fixed | fixed in <sha> / declined because … / follow-up #N |

## Plan vs built
| Section | MATCH / DRIFT / MISSING | Note |
```

## Sizing guide

- **Short (15–80 lines):** header, Outcome, Root cause or Evidence, Change,
  Tests, Verification, Out of scope.
- **Standard (80–500):** add Decisions, Design, Files.
- **Long:** add Tasks and Execution notes. This is the form a goal loop
  resumes from: its prompt should say "re-read <plan>, continue from the first
  unticked task, append Execution notes before stopping".
- **Many small fixes under one loop:** one living ledger is the plan; each fix
  PR cites its ledger item instead of getting its own plan file. If the ledger
  and GitHub disagree, GitHub wins.
