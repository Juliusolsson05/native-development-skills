---
name: development-conventions
description: "Team development conventions: GitHub issues, worktrees, branches, plans, Conventional Commits, pull requests, testing and the completion standard. Use at the beginning of every task in a software project, including planning, coding, refactoring, testing, reviewing, documentation, terminal work, Git operations, and commit creation."
license: MIT
---

# Development conventions

Apply these conventions throughout the task.

They are defaults. Explicit instructions in the current conversation and
more-specific repository-local instructions (`CLAUDE.md`, `AGENTS.md`, a
repo's contributing guide) take precedence when they conflict.

When delegating development work (to a subagent or another agent CLI), carry
these conventions into the delegated task and review the result against them.

The process skills in this collection put them into practice:
`writing-plans` (plans and PRs), `debugging` (bugs), `staged-decomposition`
(risky work) and `pr-review` (review before merge).

## General working principles

- Read the repository README, local instruction files, and relevant architecture before making changes.
- Keep work scoped to the requested outcome.
- Prefer a durable, reviewable development history over undocumented direct changes.
- Explain important decisions, constraints, and tradeoffs.
- Do not perform materially different work without explicit authorization.
- In the age of AI the most effective way of writing documentation is doing it inline in the code with explicit comments explaining logic and decisions; it is important this explains WHY and not WHAT, writing .md files to explain and document is usually not a good practice because that will drift over time in an organic codebase.

## GitHub Issues

- Create a GitHub Issue for every meaningful bug.
- Treat Issues as the permanent, searchable archive of bugs—not merely as temporary task trackers.
- A meaningful bug includes user-visible failures, regressions, incorrect behavior, data-integrity risks, security problems, reliability failures, and significant developer-experience problems.
- Record expected behavior, actual behavior, reproduction steps or evidence, impact, and relevant environment details.
- If another meaningful bug is discovered while working, create a separate Issue and link it, even if the same PR will fix it.
- Search for an existing Issue before creating a duplicate.
- Never place secrets, credentials, private customer data, or unnecessarily exploitable security details in a public Issue.
- Use Conventional Issue titles in the form <type>(<scope>): <concise description>, where type is bug, feat, refactor, docs, test, perf, build, ci, or chore.
- Create a GitHub Issue for every substantive feature or refactor before implementation, recording motivation, intended behavior, and acceptance criteria; small, self-evident changes do not require one.

## Issue and PR synchronization

- Keep GitHub Issues and their related PRs synchronized throughout development.
- Link every applicable Issue from the PR.
- Use `Fixes #123` when the PR completely resolves the Issue and `Refs #123` when it only contributes to the resolution.
- Reflect scope changes, newly discovered constraints, blockers, and changed acceptance criteria in both the Issue and PR.
- Do not close a bug Issue merely because code was written. Close it through the merged resolving PR.
- Reopen or update the Issue if the merged result does not fully resolve the documented problem.
- The Issue is the authoritative problem record; the PR is the authoritative implementation and verification record.
- Important clarification: we do not spam issues for bugs that we get during implementation of a feature or during the fix of a bug. Those are not first principle issues and will just flood workflow.

## Worktrees, branches, plans, and PRs

- Route almost all substantive repository-changing work through a dedicated Git worktree, branch, implementation plan, and PR.
- Read-only investigation, discussion, and explicitly requested local experiments do not require a PR.
- Create the worktree and branch from the correct, current base branch.
- Write the implementation plan before implementation and make it the first committed file on the branch.
- Keep the plan updated when important implementation decisions or scope change.
- Name the worktree, branch, and PR after the actual feature, fix, or outcome.
- Never name the worktree, branch, or PR after the plan filename. The plan is only the first artifact on a branch that will contain the complete implementation.
- Open the PR fully built out: implementation, meaningful tests, documentation, migration considerations, and verification should be included when applicable.
- Opening a PR does not authorize merging it.

## Commits

- Use Conventional Commit subjects:

  - `feat(scope): add a user-facing capability`
  - `fix(scope): correct broken behavior`
  - `refactor(scope): restructure without changing behavior`
  - `test(scope): improve behavioral verification`
  - `docs(scope): update documentation`
  - `perf(scope): improve performance`
  - `build(scope): change build or packaging behavior`
  - `ci(scope): change continuous integration`
  - `chore(scope): perform necessary maintenance`
  - `revert(scope): revert an earlier change`

- Use a stable subsystem or product area as the scope, not a filename or Issue number.
- Write concise, imperative summaries without a trailing period.
- Do not call something a feature when it is actually a fix, refactor, or maintenance change.
- Keep commits coherent and independently understandable.
- Do not mix unrelated changes in one commit.

### Commit bodies

- Add a commit body when the motivation or constraints are not obvious from the subject and diff.
- Explain why the change is necessary, what invariant it protects, what constraint shaped the solution, and any important consequence.
- Do not narrate the diff or list every edited file.
- Use `Refs #123` in a commit footer when useful for traceability.
- Prefer closing Issues from the PR with `Fixes #123`, since the PR represents the complete reviewed solution.
- Use a `BREAKING CHANGE:` footer only for a genuine compatibility break.

## Pull requests

- PR titles must use the same Conventional Commit format as commit subjects.
- The PR title should describe the overall delivered outcome, not the plan or internal implementation mechanism.
- The PR body should explain:

  - the problem and why it matters;
  - the implemented behavior;
  - important design decisions and tradeoffs;
  - linked Issues;
  - tests and verification performed;
  - known limitations, risks, or follow-up work.

- Keep the PR description synchronized with the implementation as the work evolves.
- Review every PR before merging.
- Resolve all valid review feedback and document why any feedback is not adopted.
- Ensure the current CI checks pass before proposing a merge.
- Never automatically merge a PR.
- After implementation, review, feedback resolution, and CI completion, report the final state and wait for explicit user confirmation before merging.
- A prior request to build or open a PR is not implicit permission to merge it.

## Testing philosophy

- Do not write tests merely to increase test counts or satisfy coverage mechanically.
- Every test should protect meaningful behavior, a regression, a safety boundary, or an important invariant.
- Prefer testing observable contracts over implementation details.
- A useful test should have a plausible failure scenario and clearly identify what behavior broke.
- Add a regression test for every meaningful deterministic bug when a stable behavioral test is practical.
- If a regression test is impractical or would be misleadingly brittle, explain why in the PR.
- Do not duplicate stronger existing coverage without adding a distinct contract.
- Avoid tautological tests that simply restate the implementation.
- Run checks proportionate to the risk and scope of the change.
- Never weaken production design merely to make a test easier to write.

## Completion standard

Before reporting work complete:

- confirm the implementation matches the requested behavior;
- confirm applicable Issues and the PR are synchronized;
- run and report the relevant tests and checks;
- review the final diff for unrelated changes;
- document remaining risks or known limitations;
- leave the worktree and branch in a clean, understandable state;
- do not merge without explicit user confirmation.
