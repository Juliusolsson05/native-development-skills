# Lifecycle, and how decompositions relate to plans and ledgers

## Status line

One line, fixed values:

`draft` · `approved <date>, <who>, <scope>, <mode>` · `stage N verified` ·
`shipped <PR/sha>` · `superseded by <path>` · `abandoned after stage N — <why>`

The most common rot in real docs was a status that contradicted reality ("none
are merged" after merge, "awaiting approval" after the fix shipped). Whoever
merges the PR that completes a stage updates the status in that PR.

## Closing

When the work merges:

1. Status → `shipped <PR/sha>`.
2. Unknowns: each marked answered-by / still open / out of scope, dated.
3. Hypotheses: every verdict filled in; stale pre-evidence text corrected so the
   doc doesn't contradict itself.
4. Add **Outcome and deviations**: what shipped differently from the doc and
   why, grouped by cause (a recording disproved it / a test did / review did /
   the owner changed it).
5. A superseded doc gets a forward link to its successor. A stage that is
   waiting on calendar time gets a trigger or its own doc.

## What goes where

| Artifact | Holds | Changes when |
|---|---|---|
| Decomposition doc | Substrate, stages, isolation, hypotheses, unknowns | A stage, the substrate, isolation or a decision changes |
| Plan | Tasks (= stages), rulings, execution notes, review dispositions | Every working session |
| PR | Implementation and verification evidence | Every push |
| Issue | The problem record | Scope or acceptance changes |
| Ledger | Many small items under one loop | Every iteration |

Findings that fit inside a stage belong in the PR or plan. Only changes to the
substrate, stages, isolation or decisions come back to the decomposition.
This keeps the doc a plan, not a log. The anti-example: a doc that absorbed
every review round grew to ~2,000 lines and 48 stages in three days.

## Goal loops

A long-running loop needs a file it re-reads each iteration:

- Point the loop prompt at the plan (tasks = stages) or the ledger.
- Each iteration: re-read it and `git log`, continue from the first unfinished
  item, append dated Execution notes before stopping.
- Collect owner-only questions in one place and ask once.

## Ledgers

For many small fixes under one loop, one living ledger replaces per-fix plans:

- A current-state block rebuilt each iteration; a newest-first log below it.
- Archive finished items instead of letting the file grow unbounded.
- Checkboxes are not truth. If the ledger disagrees with GitHub, GitHub wins.
- Ledgers are not decompositions: keep them out of `docs/decomposition/`.
