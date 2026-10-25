# Reviewer brief template

Write this to a git-ignored file (e.g. `temp/review-<pr>-<focus>.md`) and give
the reviewer a one-line prompt: "Read `<path>` and do what it says." Long
prompts pasted directly into some agent CLIs get truncated or mangled; a file
path doesn't.

```markdown
# Review: PR #<n> — <outcome>

## Your role
Independent, read-only reviewer. Do not edit files, commit, or spawn agents.
Do not run commands that clean or rebuild shared output in this worktree.

## What this PR is for
<2–4 sentences: the problem, the intended behaviour, who notices.>
Plan / issue: <paths or links>. The plan's decisions are context, not limits
on what you may flag.

## What to review
Worktree: <path>. Range: `git diff $(git merge-base origin/main HEAD)..HEAD`.
Your focus: <area A>. (Another reviewer covers <area B>.)

## Non-goals
<What this PR deliberately does not do, so you don't report it as missing.>

## The non-obvious risk
<The part the author is least sure of.>

## Required techniques
1. Mutate the implementation: flip conditions, drop guards, remove whole
   function bodies, one at a time. Report which mutations SURVIVE the tests.
2. Where behaviour depends on real data, count the real corpus (<path>)
   instead of reasoning about it.
3. A reasonable user's expectation is a requirement even if the spec is silent.

## Report
Findings, most severe first. Each: severity, file:line, a concrete failure
sequence (input → wrong output), and the evidence. Speculation without a
failure sequence goes in a separate "Suspicions" list.
Then a "Declined to judge" list: things you couldn't assess and why.
End with one verdict: MERGE-READY / FIX-BEFORE-MERGE.
```
