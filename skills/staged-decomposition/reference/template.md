# Decomposition templates

## Full

```markdown
# <Subsystem> decomposition

Status: draft | approved <date>, <who>, <scope>, <mode> | stage N verified | shipped <PR/sha> | superseded by <path> | abandoned after stage N — <why>
Approval mode: hard gate | pre-authorized (<standing instruction>, stop triggers: <H-ids / conditions>)

## Why this applies
Which risk trigger(s) fired.

## Measured problem
What is wrong, with numbers from real data. For a stalled effort: how many
prior attempts, and why they didn't converge.

## Prior attempts and graveyard
Approaches tried and the evidence that killed them. Never delete entries.

## A — what exists and is trusted
| Artifact | Establishes | Does NOT establish |
|---|---|---|

## D — end state
Observable behaviours, ≤ ~10. Invariants belong in tests or the isolation contract.

## Decisions
1. **<question>** — default <x> because <why>. Answer (verbatim, date): …

## Stage 0 — evidence
Method: extract | record | probe | not recordable (substitute: …)
Hypotheses until checked:
- H1 — <claim>. Verdict: HOLDS / REFINED (<new form>) / DISPROVEN. Evidence: <path/test>.
Stage 0 results: <counts, tables>.

## Stages
### Stage 1 — <name>
- Produces:
- Verified by:
- Why separate:
- Reality check:

## Isolation
| Hard part | Lives in | Single consumer | Forbidden importers |
|---|---|---|---|
or, for multi-source work, one owner per signal:
| Signal | Owner | Mechanism |
|---|---|---|
(Include a "Nobody" row for signals deliberately ignored.)

## Unknowns
1. <unknown> — answered by: Stage N | the user | out of scope. (Resolved <date>: …)

## Fixture and evidence plan
| Family | Real source | Label | Redaction | Consumer |
|---|---|---|---|---|
Not permitted: <e.g. invented transcripts, editing a captured fixture to pass>.

## Outcome and deviations (filled at close)
Deviations by cause: recording / test / review / owner.
```

## Bug

```markdown
# <Bug family> decomposition

Status: …   Approval mode: …

## Facts
What the recordings show, with counts. CONFIRMED only.

## Prior attempts
Each fix, why it didn't hold.

## Hypotheses
- H1 — <claim>. Would be killed by: <observation>. Verdict: …

## Stage 0 — labeler / census
The script or query that classifies real occurrences, and its results.

## Fix
The substrate change, where it lives, the fail-first test from recorded data.

## Graveyard
Dead theories, kept so they stay dead.
```

## Small (under ~80 lines)

```markdown
# <Subsystem> decomposition

Status: …   Approval mode: …

A: <what exists and is trusted>. D: <observable end state>.

Stages (1–4), each with Produces / Verified by / Why separate / Reality check.
Isolation: <hard part, where, single consumer>.
Unknowns: <numbered>.
Validation record: <what was checked, with what data>.
```
