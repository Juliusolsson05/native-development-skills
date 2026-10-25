---
name: staged-decomposition
description: "Use before implementing when several sources of truth must agree on one output, when an earlier attempt stalled at 40-70% or bugs are being fixed by adding a second conditional, when a subsystem has no recorded behaviour yet (new protocol, provider or integration), when work crosses a repo, process or trust boundary, when debugging escalates after two failed fixes, or when the user says duct tape, untangle, isolate or redesign. Also use for its testing rules alone: tests from recorded real data, fail-first, no vanity tests."
license: MIT
---

# Staged Decomposition

Implementation may not start until a written decomposition exists and is
approved under its declared approval mode.

This skill encodes a methodology, not a checklist. Read the reasoning; the
rules only work if you understand what they defend against.
Source essay: [Misadventures in Agentic Development](https://www.juliusolsson.com/blog/agentic-development).

## When this applies

Apply on **risk**, not size. Any one of these is enough:

- Two or more sources of truth must agree on one output.
- An earlier attempt stalled at 40–70%, or a second conditional is being added
  for the same family of bugs.
- The subsystem has no recorded behaviour yet: a new protocol, provider, or
  integration whose real shapes nobody has enumerated.
- The change crosses a repo, process or trust boundary.
- `debugging` escalated after two failed fixes.
- The user says "duct tape", "untangle", "isolate", "redesign".

Codebase size is a weak hint: tech debt compounds faster past ~35k LOC, but a
small repo with an unknown case set still needs this, and a large repo with a
known shape does not.

**Skip when** the shape is known and nothing above holds. Say you are skipping
and why, then use `writing-plans`. A skill that fires on trivial work gets
ignored on real work.

For **just the testing rules** (a feature that doesn't need stages but does
need real tests), read [reference/tests-and-fixtures.md](reference/tests-and-fixtures.md)
and apply it; no decomposition doc required.

## The failure this prevents

Asked to get from A to D, models go straight to D.

What comes back looks great and is roughly **40% correct**. It handles the
cases that happened to be in context. Then a bug, an `if`. Another bug, another
`if`. 50, 60, maybe 70%. Then it stops working: in a system that isn't CRUD,
each change breaks something buried elsewhere. There is no sequence of bug
reports that converts a wrong substrate into a right one.

**This is a workflow problem, not an intelligence problem.** Fill the middle:
A → B → C → D.

## Why the middle gets skipped

**Intermediate stages produce nothing visible.** A recorder renders nothing. A
catalog renders nothing. To a process optimising for visible output, the middle
looks like empty air.

**Forward-patching never has to admit defeat.** It will squeeze the next fix
into the existing structure forever. What it will not do on its own is say
"this whole substrate is wrong; build the boring invisible thing first".

So roughly half the structural groundwork comes from the human, and the
highest-value intervention in agentic development is **deciding to do
something slower in order to do it correctly.**

## The method

### Stage 0: evidence

The first stage establishes what really happens, with frequencies. Pick the
cheapest method that yields real data, in this order:

1. **Extract** from data already on disk (logs, recordings, transcripts,
   journals, databases). This is the usual case.
2. **Record**: build instrumentation only when the signal isn't on disk yet.
3. **Probe**: drive the real system deliberately to observe a behaviour.
4. **Not recordable**: say so, name the substitute, and label everything built
   on it.

Publish a **Stage 0 results** block with numbers before later stages proceed.
List invariants as **hypotheses until checked** (H1, H2, …). Each gets a
verdict in place: HOLDS / REFINED / DISPROVEN, with the evidence path. Killed
hypotheses move to a **graveyard** that is never deleted, so they stay dead.

Stage 0 is the single highest-value step and the one most likely to be skipped,
because it produces nothing visible.

### What counts as a stage

Every stage declares four fields. A stage that can't fill all four is not a stage:

| Field | Requirement |
|---|---|
| **Produces** | A named, inspectable artifact: a file, dataset, catalog, module. Not "understanding". |
| **Verified by** | How you know it's correct without depending on any later stage. |
| **Why separate** | What breaks if it's merged into the next stage. |
| **Reality check** | What real, collected evidence it's built from, or "none — reason". |

### Isolate the hard part

Name the genuinely hard component and confine it: its own directory or module,
a single consumer, an explicit list of what may not import it.

When several sources of truth must be reconciled, reconciliation gets its own
isolated layer that emits one clean object, with one owner per signal. Spread
across consumers, the sources fight, and ownership bugs masquerade as
presentation bugs (duplicates, vanishing items, ordering that changes between
runs). See [reference/structure-and-isolation.md](reference/structure-and-isolation.md).

## Tests and fixtures

**AI-written TDD against imagined inputs is worse than no tests: it
manufactures false confidence.** A model decides what the code does, writes
tests asserting exactly that, and the suite goes green. A fundamentally broken
renderer passed **481 of 481** such tests.

- Fixtures come from recordings of real behaviour (Stage 0 produces them).
  Label every fixture: recording / mechanical mutation of a recording /
  contract probe / schema-derived / invented literal (justified).
- Write tests before the implementation, against those fixtures.
- Prove each guard: revert or mutate it and watch a test go red. A surviving
  mutant is a claim about the suite.
- Never delete or weaken a failing test against a real fixture. It is the most
  valuable artifact you have: reality disagreeing with your design.
- The semantics (which source wins, what should own a piece of state) come from
  the human. Ask; don't infer. The oracle is never the reader.

Full rules in [reference/tests-and-fixtures.md](reference/tests-and-fixtures.md).

## The document

Write `docs/decomposition/<subsystem>.md` (a repo-local convention wins) before
any implementation code. Name it after the subsystem, not a date: it is revised
in place when a stage disproves it. Pick a size:

- **Full**: subsystems and multi-source work.
- **Bug**: a stalled bug. Facts → prior attempts → hypotheses with the
  observation that would kill each → Stage 0 → verdicts → fix → graveyard.
- **Small** (under ~80 lines): A→D in two paragraphs, 1–4 stages with the four
  fields, isolation, unknowns. Never drop A and D, even here.

Templates in [reference/template.md](reference/template.md). Every size states:
a status line, A (what exists and is trusted, and what it does *not*
establish), D (observable end state, ≤ ~10 behaviours), the stages, what is
isolated, **Unknowns** (numbered, each with the stage or person that answers
it; an empty list means you haven't looked), the fixture plan, and the approval
mode.

On a branch that requires a plan as its first commit, the decomposition doc is
that commit, or the plan points to it as its spec and its tasks are the stages.

## Workflow

1. Confirm the skill applies (or say why not).
2. Read the current state: code, history, earlier attempts, real data.
3. Write the decomposition.
4. **Approval, per the declared mode:**
   - **Hard gate**: stop and present it. No implementation code until approved.
   - **Pre-authorized** (the user gave a standing go): publish Stage 0 results
     before later stages, and stop at the named stop-and-ask triggers.
   - Never waivable in any mode: recordings over imagination; never weaken a
     failing real test; revise the doc rather than patch forward; semantics
     come from the human (or record the default taken and why).
   - "Keep fixing" is not approval of a revised decomposition.
5. Implement one stage at a time. Confirm each stage's artifact exists and is
   independently verified before the next.
6. If a later stage shows the decomposition was wrong, stop and **revise the
   document**. Do not patch forward. Findings that fit inside a stage go to the
   PR, not into new stages.
7. **Close**: when the work merges, update the status line, resolve the
   Unknowns (answered by / still open / out of scope), and add a short
   "Outcome and deviations". See [reference/lifecycle-and-ledgers.md](reference/lifecycle-and-ledgers.md).

Review is bounded: one adversarial review per PR or stage boundary, limited to
the recorded invariants and accepted findings. "Zero findings from an
open-ended hunt" is not a merge bar; a finding with no recording or reachable
trace becomes an Unknown, not work.

Size guard: past ~25 stages or ~1,000 lines, re-decompose into a successor doc.

## Red flags

| Thought | Reality |
|---|---|
| "I can see the whole thing, I'll just build it" | You see the shapes in context, not the rest. |
| "The data exists, but I'll build a recorder" | Extract it. Record only what isn't on disk. |
| "I'll add the recorder afterwards if needed" | Afterwards the implementation is the substrate. |
| "The tests pass" | Written by whom, from what data? Did any go red when you reverted the fix? |
| "I'll write fixtures that look like real data" | Then you're testing your imagination. |
| "This stage produces nothing visible" | Correct. That's why it's the one being skipped. |
| "I'll handle that edge case with a conditional" | A second conditional means the substrate is wrong. |
| "This test is wrong, I'll adjust it" | Against a real fixture, the test is right and the code is wrong. |
| "A reviewer found something, add Stage N+1" | PR fix, unless the substrate changed. |
| "Autonomous run, so skip the stop" | Declare the mode and name the stop triggers. |
| "It merged, the doc is fine" | Close it: status, unknowns, deviations. |

## What good steering sounds like

Phrasing rituals do nothing. Structural direction changes outcomes:

> "It feels like our rendering system is spread out over the codebase. Should
> we not take this and isolate it into its own `/rendering` directory?"

Expect and invite this kind of correction.

## References

- [reference/template.md](reference/template.md): full, bug and small doc templates
- [reference/tests-and-fixtures.md](reference/tests-and-fixtures.md): the fixture methodology and test rules in full
- [reference/structure-and-isolation.md](reference/structure-and-isolation.md): isolation, pipelines, untangling
- [reference/lifecycle-and-ledgers.md](reference/lifecycle-and-ledgers.md): status, closing, specs vs plans vs ledgers, goal loops
- [reference/worked-examples.md](reference/worked-examples.md): real cases, including an anti-example
