# Reviewer brief template

Write one brief per reviewer to a git-ignored file (e.g.
`temp/review-<pr>-<focus>.md`) and give the reviewer a one-line prompt:
"Read `<absolute path>` and follow it." A long prompt pasted into a subagent
call is easy to truncate or mangle; a file path isn't, and the brief stays on
disk if you need to relaunch.

The brief is self-contained on purpose: a reviewer that is a plain
`general-purpose` subagent, a Codex agent or another CLI gets the same rules
as the shipped `pr-reviewer` agent.

```markdown
# Review: PR #<n> — <outcome>

## Your role
Independent reviewer. You did not write this code. Report findings; never
fix them.
- Work only in your own worktree (below). Never edit, commit or push in any
  other checkout.
- Never commit, push, open PRs, comment on GitHub, or spawn other agents.
- You may edit files in your worktree only to mutate the implementation, one
  mutation at a time, reverting each one right after testing it (edit it back;
  `git checkout -- <file>` also works where git is allowed to write). When
  finished, confirm `git status --short` is empty.

## Setup
Head SHA: <sha>. Base: <base sha or origin/main>.
<Claude Code subagent:> You start in a temporary worktree on the default branch.
Run `git checkout --detach <sha>` first and confirm `git rev-parse HEAD`.
<Codex / other CLI:> Your worktree is `<path>`, already at <sha>. `cd` there first.
Range: `git diff <base>...<sha>`.

## What this PR is for
<2–4 sentences: the problem, the intended behaviour, who notices.>
Plan / issue: <paths or links>. The plan's decisions are context, not limits
on what you may flag.

## Your focus
<area A>. (Another reviewer covers <area B>.)

## Non-goals
<What this PR deliberately does not do, so you don't report it as missing.>

## The non-obvious risk
<The part the author is least sure of.>

## Required techniques
1. Mutate the implementation: flip conditions, drop guards, remove whole
   function bodies, one at a time. Run the relevant tests after each. Report
   which mutations SURVIVE.
2. Where behaviour depends on real data, count the real corpus (<exact paths>)
   instead of reasoning about it. Don't scan large folders recursively.
3. A reasonable user's expectation is a requirement even if the spec is silent.

## Report
Your final message is the report. Findings, most severe first. Each: severity,
file:line, a concrete failure sequence (input → wrong output), and the
evidence. Speculation without a failure sequence goes in a separate
"Suspicions" list. Then "Mutations" (each: what you changed, survived or
killed, by which test). Then "Declined to judge": things you couldn't assess
and why. End with one verdict: MERGE-READY / FIX-BEFORE-MERGE.
```
