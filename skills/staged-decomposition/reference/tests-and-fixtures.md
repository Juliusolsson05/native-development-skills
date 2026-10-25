# Tests and fixtures

## Contents

- Why AI-written TDD fails
- The 481/481 case
- What a real fixture is
- The fixture pipeline
- Where human judgement is required
- Getting the evidence (Stage 0 methods)
- Labels, provenance and privacy
- Rules learned in practice
- Checklist before trusting a suite

---

## Why AI-written TDD fails

TDD is the correct approach for complex, deeply integrated logic. Both hard problems referenced in this skill were ultimately solved with it. The methodology is not in question.

What fails is TDD performed by a model without real inputs.

The mechanism is specific and worth stating precisely:

1. The model decides what the code should do
2. The model writes the implementation
3. The model writes tests asserting the behaviour it just implemented
4. The suite passes

Step 3 is the problem. The tests are derived from the same understanding that produced the implementation. If that understanding was wrong — if it missed a shape, misjudged which source of truth wins, misread an edge case — the tests encode the identical error and confirm it.

**A green suite built this way proves only that the code does what the code does.**

Models are drawn to this because passing tests are a legible output. It is a **vanity metric**: it produces the visible artifact of rigour without the substance. The same optimisation that skips the recorder produces the tests-that-bless-the-implementation.

## The 481/481 case

A rendering implementation, built by taking the A → D shortcut, passed **481 out of 481 tests**.

The product was broken. Common shapes rendered; uncommon shapes were unsupported entirely. The tests were green because they were written from the same imagination that wrote the bug — they tested the shapes the implementation already knew about, which was precisely the set of shapes that had been in context.

The tests could not fail. There was no input in the entire suite that the implementation had not already been built around.

**Test count is not evidence.** Neither is pass rate, when the inputs are invented.

## What a real fixture is

| Real fixture | Not a fixture |
|---|---|
| A captured payload from an actual session | A plausible-looking literal typed into the test file |
| A recorded sequence including its ordering and timing | An idealised sequence with clean ordering |
| The malformed case that actually occurred in production | The malformed case someone imagined might occur |
| Output from stage-one instrumentation | Output from the model's expectations |

The distinction is not about realism of appearance. An invented fixture can look completely realistic and still be worthless, because it was generated from the same assumptions as the implementation. What makes a fixture valuable is **causal independence from the implementer's beliefs.**

## The fixture pipeline

This is why instrumentation is stage one. The stages exist in this order for a reason:

```
Stage B: Recorder    -> captures real inputs during actual use, with frequency
Stage C: Catalog     -> enumerates the observed shapes and their data
Stage D: Implement   -> tests written from B's recordings, against C's contract
```

Concretely:

1. **Instrument first.** Capture real inputs during genuine use of the system. Record frequency — it tells you which cases are load-bearing and which are rare-but-real.
2. **Store recordings as fixtures**, verbatim. Do not clean them up. The ugliness is the point.
3. **Catalog what was observed.** Every distinct shape, its structure, its data. This becomes the contract.
4. **Write tests against the recordings before implementing.** They should fail.
5. **Implement one shape at a time** until they pass.

If a recording is inconvenient — malformed, out of order, contradicting the model you had in mind — that recording is the most valuable item in the corpus. It is reality disagreeing with your design, before you built on it.

## Where human judgement is required

Writing meaningful tests for a deeply integrated component requires understanding the system's own semantics. These questions cannot be answered by inspecting the code, because the code is what is in question:

- When two sources of truth disagree, which one wins, and why?
- What *should* own this piece of state?
- Is this observed behaviour correct, or is it a bug that has been present long enough to look normal?
- Which of these recorded cases are permanent, and which are artifacts of the current implementation?

**Ask. Do not infer.** If you invent the semantics, you will write tests that bless your invention, and you will have reproduced the exact failure this document describes — one layer deeper, where it is harder to see.

This is the point in agentic development where human input has the highest leverage, and it is the point most often skipped, because asking is slower than assuming.

## Getting the evidence (Stage 0 methods)

A recorder is not always the first move. In most real cases the data already
existed on disk. Choose, in order:

1. **Extract** from what exists: logs, transcripts, journals, debug bundles,
   databases. Write an extractor; if the raw sources rotate or are private,
   commit the sanitized output and give the extractor a `--verify-checked-in`
   mode.
2. **Record**: add instrumentation only when the signal isn't captured anywhere.
3. **Probe**: drive the real system deliberately (a live PTY, a real API call)
   to observe one behaviour.
4. **Not recordable**: state it, name the substitute, and label everything
   built on it as derived.

Read the source for the *shape*; count the corpus for *frequency*. Reasoning
from source alone has shipped bugs that a count would have caught
("136 turns against 105" ended an argument source-reading got wrong).

## Labels, provenance and privacy

Every fixture carries a label, in its manifest or README:

| Label | Meaning |
|---|---|
| recording | Captured verbatim from real use (ids/paths sanitized) |
| mechanical mutation | A recording with one deliberate, scripted change; label it a mutation, not a recording |
| contract probe / fault injection | Produced by driving the real system into a state on purpose |
| schema-derived | Built from a published schema, not observed |
| user-authored expectation | The human's statement of correct semantics (e.g. an `expectations.json` oracle) |
| invented literal | Allowed only with a written justification |

Privacy: tokenize personal data instead of normalizing it away; never normalize
away the behaviour under test. Do not edit a captured fixture to make a test
pass: re-record it, or revise the decomposition.

## Rules learned in practice

- **Mutation-prove every guard.** Revert the fix or mutate the guard and watch a
  test go red. Mutate every call site, not just one, and try removing whole
  function bodies. Apply mutations one at a time.
- **A surviving mutant is a claim about the suite**, not about the code.
  "Equivalent mutant" claims were wrong 3 times out of 4 when someone built the
  distinguishing state.
- **Fail-first is tri-state**: observed red / not observed / not applicable.
  Never tick "confirm it fails" when you didn't see it fail.
- **Pin corpus and fixture counts** in the test, so a silently shrinking corpus
  fails loudly.
- **Drive the real entry point.** Tests go through the real routing, reducer or
  painter; mock only true external edges. A test named "end to end" must justify
  the name.
- **Stand-ins must be as messy as reality.** A fake that refuses everything, or
  never produces the real shape (a line-wrapped token, a partial chunk), passes
  against broken code.
- **No test reads the developer's home directory or live corpora.** Extract,
  sanitize, commit.
- **The oracle is never the reader.** Expected values never come from the code
  under test or its helpers; they come from the recording or the human.
- **Each test names the break it catches.** Which production change fails this
  test? If you can't name one, redesign the test.
- **No change detectors** (asserting constants, exact wording, private
  structure), and never assert on a mock.
- **Suspiciously good results are bugs until proven otherwise.**

## Visual and perceptual work

Recorded browser sessions and screenshots with provenance ids are the fixtures.
Math and GPU checks are labelled synthetic. Perceptual acceptance ("it looks
right") is a named gate that stays open until the human closes it. A green
check never means "verified in the installed app". If the capture tool
disconnects, say so; never fabricate the evidence.

## Checklist before trusting a suite

- [ ] Every fixture traces back to a recording of real behaviour
- [ ] Tests were written before the implementation they cover
- [ ] At least one test failed for a real reason before it passed
- [ ] No test was weakened, skipped, or deleted to reach green
- [ ] The semantics being asserted were confirmed by a human, not inferred from the code
- [ ] Rare-but-real recorded cases are covered, not just the frequent ones
- [ ] The suite would fail if the implementation silently dropped an uncommon shape
- [ ] Every guard was mutation-checked and at least one test went red
- [ ] Every fixture carries a provenance label

If the last item is not true, the suite is measuring the implementation against itself.
