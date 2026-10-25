---
name: pr-review
description: "Use when a PR or branch is built and needs review before merge, and whenever the user says review this, review and merge, get two agents to review this, or is this ready to merge; also when receiving review findings from another agent or a pasted review. One round of two independent reviewer subagents, findings triaged against the code, valid ones fixed, merge only on explicit approval. Replaces superpowers requesting-code-review and receiving-code-review."
license: MIT
---

# PR review

One round. Two independent reviewers running as subagents. Findings are claims
to check, not orders. Merge only when the user says so, for that PR.

## Requesting review

1. **Scope.** The PRs built in this conversation, unless the user names others.
   Say which PRs are in scope.
2. **Commit and push first.** Reviewers read commits, not your working tree:
   anything uncommitted is invisible to them. Note the head SHA.
3. **Brief.** One brief per reviewer, self-contained, written to a git-ignored
   file (e.g. `temp/review-<pr>-<focus>.md`; add `temp/` to `.gitignore` if it
   isn't there). Use [reference/reviewer-brief.md](reference/reviewer-brief.md).
   It carries the goal, the head SHA and merge-base range, the plan/issue path,
   the non-goals, the non-obvious risk, and the report shape. The two briefs
   differ in focus area, and the focus areas don't overlap (e.g.
   correctness and failure paths / contracts, tests and integration). They
   also differ where the setup does: a Codex reviewer's brief names its own
   worktree, and a read-only reviewer's brief drops the mutation technique.
4. **Launch two reviewers in parallel.** See "Launching reviewers" below for
   Claude Code and Codex. Never review your own PR in your own context instead:
   the point is a reader who didn't write the code.
5. **Reviewers never touch your tree.** Each works in its own worktree at the
   head SHA, so mutation testing can't leak into your branch. They never
   commit, push, open PRs, comment on GitHub, or spawn their own reviewers.
6. **The two instructions that find the most bugs** (already in the brief
   template):
   - *Mutate the implementation and report which mutations survive.* One
     reviewer doing this beat two who only read.
   - *Count the real corpus* when the change depends on what real data looks
     like, instead of reasoning about it.
7. Never tell a reviewer what not to flag or pre-set a severity ("at most
   minor", "the plan already chose this"). Adjudicate afterwards.
8. Don't push new commits to the PR until both reviewers have reported, or
   their findings will point at code that moved.

## Launching reviewers

### Claude Code

This plugin ships a reviewer subagent, `native-dev:pr-reviewer`. It runs in a
temporary git worktree (`isolation: worktree`), checks out the head SHA from
its brief, and its final message is the report.

- Make **two Agent tool calls in the same message** so they run concurrently,
  both with `subagent_type: "native-dev:pr-reviewer"`.
- Each prompt is one line: `Read <absolute path to brief> and follow it.` Use
  an absolute path: the reviewer's working directory is its own worktree.
- Name the model. The default inherits yours; you may give the second reviewer
  a different strong model (`model` on the call) to catch different bugs.
  Cheap models argue for defects that aren't there.
- If the plugin agent isn't installed (skills copied by hand), use a
  `general-purpose` subagent with `isolation: "worktree"` on the call and the
  same one-line prompt; the brief template contains the full reviewer rules.

The temporary worktree starts from the default branch, which is why the brief
tells the reviewer to `git checkout --detach <sha>` before reading anything.

**Clean up after reading both reports.** Claude Code removes a subagent's
worktree only when it sees no changes, and a reviewer that moved HEAD to the
PR's commit counts as changed: in testing, both review worktrees and their
`worktree-agent-*` branches were left behind. Remove them yourself:

```bash
git worktree list                                   # find .claude/worktrees/agent-*
git worktree remove .claude/worktrees/agent-<id>    # for each reviewer
git branch -D worktree-agent-<id>
```

Add `.claude/worktrees/` to the repo's `.gitignore` so leftovers never show up
in `git status`.

### Codex

Codex starts subagents only when asked explicitly. Install the reviewer agent
from `codex/agents/pr-reviewer.toml` in this repo into `~/.codex/agents/` (or
the project's `.codex/agents/`).

Know your sandbox first. Under the usual `workspace-write` sandbox, files
inside the session's working directory are writable but `.git` is protected,
so **any git command that writes** (`worktree add`, `checkout`, `worktree
remove`) fails or asks for approval. Subagents inherit the same sandbox, and
an agent file can't change it.

- Create one worktree per reviewer at the head SHA, somewhere inside the
  session's working directory, approving the git write when Codex asks (or
  running it in your own terminal):
  `git worktree add --detach .worktrees/review-<pr>-<focus> <sha>`.
  Keep `.worktrees/` git-ignored so the worktrees stay out of `git status`.
  Put the worktree path in each brief.
- Prompt: `Spawn two pr-reviewer agents. Give the first "Read <brief A> and
  follow it." and the second "Read <brief B> and follow it." Wait for both and
  return both reports in full.` Name the model in the prompt or set `model` in
  the agent file; cheap models argue for defects that aren't there.
- Reviewers undo each mutation by editing the file back, not with git, because
  git can't write inside the sandbox. The brief confines their writes to their
  own worktree; nothing else enforces it.
- Remove the review worktrees after reading the reports (another git write to
  approve): `git worktree remove --force .worktrees/review-<pr>-<focus>`.

### A second provider (optional)

Mixing providers catches different bugs, but it is never required. Either
way, create the reviewer's worktree first:
`git worktree add --detach .worktrees/review-<pr>-<focus> <sha>`.

- From Claude Code, one reviewer can be Codex:
  `codex exec -s read-only -C <review worktree> -o <report file> "Read <brief> and follow it."`
- From Codex, one can be Claude Code:
  `claude -p "Read <brief> and follow it." > <report file>`, run inside the
  review worktree. This needs network access and writes under `~/.claude`, so
  it only works outside the sandbox: approve the escalation when Codex asks.
  Without permission flags `claude -p` refuses edits, so treat it as read-only.

A read-only reviewer can't mutate: drop the mutation technique from its brief
and give it to the other reviewer. Remove the worktree afterwards.

## Receiving review

Applies to your reviewers, a pasted review, or another agent's report.

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
  main; check mergeability (`gh pr view --json mergeable,mergeStateStatus`)
  before waiting.
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
| "I'll just review it myself, it's faster" | You wrote it. Launch the two reviewers. |
| "I'll launch one, then the other" | Two Agent calls in one message, or they don't run in parallel. |
| "The reviewer can see my changes" | Only committed ones. Commit and push first. |
| "The reviewer says it's a bug, fixing it" | Check it against the code first. |
| "Let me get one more review to be safe" | One round. File an issue instead. |
| "I'll tell the reviewer to ignore X, it's intentional" | Don't steer. Adjudicate after. |
| "The reviewer can just fix it" | Reviewers never commit. Mutations are reverted. |
| "The review worktrees clean themselves up" | They didn't in testing. Check `git worktree list` and remove them. |
| "CI is green and the user approved the last PR" | Approval is per PR. Ask. |
| "I'll summarize the reviews without reading them fully" | Read every word first. |
