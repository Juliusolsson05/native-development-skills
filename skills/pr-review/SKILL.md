---
name: pr-review
description: "Use when a PR or branch is built and needs review before merge, and whenever the user says orch review, review and merge, get two agents to review this, or is this ready to merge; also when receiving review findings from another agent or a pasted review. One round of independent read-only reviewers, findings triaged against the code, valid ones fixed, merge only on explicit approval. Replaces superpowers requesting-code-review and receiving-code-review."
license: MIT
---

# PR review

One round. Independent reviewers. Findings are claims to check, not orders.
Merge only when the user says so, for that PR.

## Requesting review

1. **Scope.** The PRs built in this conversation, unless the user names others.
   Say which PRs are in scope.
2. **Reviewers.** Two in parallel per PR, with non-overlapping focus areas
   (e.g. correctness and failure paths / contracts, tests and integration).
   Use the reviewer providers the user has chosen (their standing preference
   wins; mixing providers catches different bugs). Name the model; cheap
   reviewers argue for defects that aren't there.
3. **Brief.** Self-contained, written to a git-ignored file, with a one-line
   prompt pointing at it. Use [reference/reviewer-brief.md](reference/reviewer-brief.md).
   It carries the goal, the merge-base range, the plan/issue path, the
   non-goals, the non-obvious risk, and the report shape.
4. **Read-only.** Reviewers never edit code, never spawn their own reviewers,
   and don't run build or test commands that clean shared output in your worktree.
5. **The two instructions that find the most bugs:**
   - *Mutate the implementation and report which mutations survive.* One
     reviewer doing this beat two who only read.
   - *Count the real corpus* when the change depends on what real data looks
     like, instead of reasoning about it.
6. Never tell a reviewer what not to flag or pre-set a severity ("at most
   minor", "the plan already chose this"). Adjudicate afterwards.
7. Don't edit the worktree until every reviewer has finished reading it.

## Receiving review

Applies to your orchestrated reviewers, a pasted review, or another agent's report.

1. Read every report in full before acting on any item. Reviewer output is gold,
   but it is a set of claims.
2. For each finding, check it against the actual code and mark it:
   **valid** / **overstated** / **already fixed** / **wrong**. Reproduce the
   serious ones. Push back with evidence; no performative agreement.
3. When reviewers conflict, pick one and record why.
4. Fix the valid findings that matter (correctness, regressions, data loss,
   security, real user-facing breakage). Edge-case polish and nitpicks become
   follow-up issues, not more work on this PR.
5. Regression-test each fix and mutation-check it (revert, watch it go red).
6. Record a disposition table in the PR body (and the plan, if there is one):

   | Finding | Verdict | Disposition |
   |---|---|---|
   | … | valid | fixed in `<sha>` |
   | … | overstated | declined: <reason> |
   | … | valid, large | follow-up #N |

## No round two

Do not dispatch a re-review. Do not run round 2, 3, … 10. Anything a second
round would catch can be a follow-up issue; throughput matters more than
exhausting a reviewer's imagination. If a finding is big enough to need its own
change, file an issue and leave it out of this PR.

If review narrowed the PR's scope, check every commit's trailer: a `Fixes #N`
in any commit closes the issue on merge, even if the PR body says `Refs #N`.

## Merge

- Wait for green CI. "No checks reported" usually means the PR conflicts with
  main; check mergeability before waiting.
- Report the final state in a short TLDR (fixed / declined / follow-ups / CI)
  and **stop**.
- Merge only on the user's explicit go-ahead for **this** PR. An earlier
  approval, or "build and open a PR", never authorizes a merge.
- When authorized, use the repository's merge style (merge commits unless the
  user says otherwise).

## Deadline mode

If the user explicitly declares deadline mode, review can be skipped or reduced
to one light pass. Merge still needs their say-so.

## Red flags

| Thought | Reality |
|---|---|
| "The reviewer says it's a bug, fixing it" | Check it against the code first. |
| "Let me get one more review to be safe" | One round. File an issue instead. |
| "I'll tell the reviewer to ignore X, it's intentional" | Don't steer. Adjudicate after. |
| "The reviewer can just fix it" | Reviewers are read-only. |
| "CI is green and the user approved the last PR" | Approval is per PR. Ask. |
| "I'll summarize the reviews without reading them fully" | Read every word before closing the reviewers. |
