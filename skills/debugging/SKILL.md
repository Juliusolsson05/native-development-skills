---
name: debugging
description: "Use for any bug, regression, test or CI failure, crash, OOM, CPU/memory/disk problem, hang, or anything called flaky or intermittent; when the user hands over a debug bundle, log, session id or screenshot; or asks did we not fix this, diagnose, or find the root cause. Evidence first, root cause before any fix, regression test from recorded data proven by reverting the fix. Replaces superpowers systematic-debugging."
license: MIT
---

# Debugging

No fix, and no claim of cause, without an artifact that shows it.

Every statement in a diagnosis is either **CONFIRMED** (a concrete input traced
to a wrong output, with the artifact that shows it) or labelled a
**SUSPICION**. If you cannot observe the cause, the next change is
instrumentation, not a fix.

## Diagnose-only means zero edits

"Diagnose", "investigate", "find out why", "do not fix it" mean: change no code,
report the root cause with evidence, and wait for "fix it". Edits made during a
diagnosis are reverted.

## The method

**1. Read everything the user captured, completely, first.** A bundle, log,
session id, recording or screenshot is the primary evidence. Read all of it
before forming a theory. No artifact? Check the project's always-on evidence
sources (its incident journal, debug logs, recorded transcripts — the project's
AGENTS.md or evidence map says where) before asking.

**2. Know the bug's history.** Search issues and PRs for the symptom, read the
git log of the area, read earlier fix attempts and why they didn't hold. A bug
that "we already fixed" means the last fix was fake or partial. Re-diagnose; do
not re-patch.

**3. Rule out the environment and the measurement.**
- Is the number or message itself trustworthy? Tools return sentinels, loaders
  fall back silently, errors lose their signal, profilers show an idle thread
  while GC runs elsewhere. Check the measurement before the thing measured.
- Is it my change? Answer with `git diff --name-only <range>` and a run on a
  clean checkout of main, not intuition. Both directions matter: don't dismiss a
  regression as pre-existing, don't fix a non-regression in the wrong place.
- "It worked before": diff against the last known-good state (uncommitted
  changes, recent commits, stash, bisect across repos and submodules).

**4. Quantify.** Turn the symptom into counts across the whole artifact or
corpus: how many runs, frames, sessions, submits. Tabulate failing versus
working cases that differ in one variable. Source tells shape; the corpus tells
frequency; the installed binary tells the truth (vendored source can be
versions stale).

**5. Hypothesize, then discriminate.** Write ranked hypotheses as suspicions.
For each, name the observation that would kill it, then go get that
observation. Keep a **dead theories** list with the evidence that killed each.
For memory problems, build or use the measuring instrument (heap snapshot,
dominator/owner analysis) before naming an owner.

**6. Reproduce on the real mechanism.** Drive the real entry points with the
exact failing input: a throwaway harness outside the product code that runs the
real process, PTY, API or database. Repeat it (10–20 runs, varied parameters)
and watch the raw state. An isolated test that passes while the product still
fails means the root cause is not proven.

**7. Make "intermittent" deterministic.** Intermittent means an uncontrolled
variable: ordering, chunking, load, clock, wrap width, disk latency. Name it and
pin it (delay one macrotask, replay the recorded order, simulated clock). A
flaky test is a bug in the code or the test until proven otherwise. Reruns are
not a diagnosis. See [reference/flaky-and-timeouts.md](reference/flaky-and-timeouts.md).

**8. State the root cause as "X because Y"**, with the evidence, before
touching code. Say "I don't understand X" rather than pretending.

**9. Fail-first regression test from recorded data.** The fixture is a
captured real input, sanitized only in ids and paths. Drive the real entry
point; mock only true external edges, with stand-ins that behave like reality
(never "more absolute than reality"). Record the failure against the unfixed
code. Fix. Then **revert the fix and confirm the test goes red**; restore it.
Then re-run the original symptom, not just the new test.

**10. Fix at the source.** Fix where the bad value originates or at the one
funnel every caller passes through, not at the symptom site or in each caller.
Add the diagnostic that would have made this obvious next time. Opportunistic
cleanup inside the blast radius is fine.

**11. Verify honestly, then record.** Run the project's check command and
report every failure you saw, classified against main. Anything that needs the
running product and you can't run: write it as a manual step for the user,
marked unverified. Record the finding where it will be found again: issue
(expected / actual / root cause / evidence / impact), PR body (before-fix
failure, what was ruled out), a WHY comment at the fix. Correct any comment,
issue or PR text you found to be false.

## Escalate after two failed fixes

If this is the second failed fix on the same bug family, or each fix reveals a
new symptom somewhere else, the substrate is wrong. Stop, tell the user in one
line, and switch to `staged-decomposition` (evidence → catalog → stages). A
third symptom patch is never the answer.

## Delegated investigation

For multi-symptom incidents, split by area into read-only investigators with
self-contained briefs: goal, artifacts, "no code changes, no app launch",
deliverable = root cause + file:line evidence + a concrete failure sequence.
Their reports are claims: read the evidence yourself before relaying a
diagnosis. A reviewer who reproduces your fix as wrong beats your reasoning —
withdraw the fix and write down what killed it.

## Forbidden

1. Widening a timeout, adding a retry or a sleep, or rerunning CI to make a
   failure go away. A timeout says "the awaited state never arrived"; find out why.
2. Calling anything flaky, pre-existing or environmental without a comparison
   against main or a deterministic reproduction.
3. Fixing from code reading alone when an artifact exists or could be captured.
4. Disabling or masking the feature to make the symptom disappear.
5. `if`-stacking for the case in front of you, regex duct tape, fixes tailored
   to the one failing input.
6. Tests from imagined literals, or test doubles that refuse everything.
7. Weakening, deleting or "adjusting" a failing test that runs on real data.
8. Claiming "fixed" without the reverted-fix red run and the original symptom
   re-checked.
9. Trusting vendored source over the installed binary, or our own comments over
   the code.
10. Relaying a subagent's or reviewer's diagnosis as fact without reading the
    evidence.
11. Leaving the finding only in chat.

## Red flags

| Thought | Reality |
|---|---|
| "It's probably X, let me change it" | Probably isn't a root cause. X because Y, with evidence. |
| "Quick fix now, investigate later" | Later never comes; the patch becomes the substrate. |
| "I'll change a few things and rerun" | Then you can't tell which one mattered. One variable. |
| "Bump the timeout, CI is slow" | It's a race or a real hang. Wait on the condition. |
| "It's flaky" | Name the uncontrolled variable or compare against main. |
| "The test passes now" | Did it fail before the fix? Does it fail with the fix reverted? |
| "The agent says it's fixed" | Read the diff and re-run the symptom yourself. |
| "One more attempt" (after two) | Escalate to staged-decomposition. |
| "Memory keeps growing, it's a leak" | Churn and retention look alike. Measure which. |

## References

- [reference/signatures.md](reference/signatures.md) — symptom → discriminating check → usual cause
- [reference/case-studies.md](reference/case-studies.md) — real misdiagnoses and what corrected them
- [reference/flaky-and-timeouts.md](reference/flaky-and-timeouts.md) — the timeout doctrine and deterministic repro techniques
