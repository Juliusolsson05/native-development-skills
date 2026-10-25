# Worked examples

Two problems in the same system, solved by the same decomposition. Agent Code is an Electron-based AI-native IDE acting as a UI and feature middleman for Claude Code, Codex, and OpenCode. It runs the real CLIs in a PTY rather than using the SDKs, so native behaviour — compaction, all commands — is preserved. Roughly 300k lines, zero human-written.

## Contents

- Case 1: Rendering the agent feed
- Case 2: Conversation ownership
- The shared pattern
- Case 3: A stalled bug solved by a labeler (prompt delivery)
- Case 4: The counterexample (skipping capture)
- Case 5: Stage 0 with hypotheses (a new provider)
- Anti-example: review loops that became stages

---

## Case 1: Rendering the agent feed

**A (exists, trusted):** a sanitized object representing the conversation — what should appear on screen.

**D (goal):** React elements painted for every operation shape, across every provider.

### Why this is hard

The upstream ownership system deliberately produces a sanitized object *without* categorising shapes and without locking to a fixed list of renderable things. So the set of elements to draw, and their structures, was genuinely unknown at the start.

Each provider has different shapes for a bash command, a file edit, and so on. Claude, Codex, OpenCode, Pi.dev all differ. Basic shapes are easy. The edge-case shapes are where it gets tricky — and they are exactly the ones absent from any given context window.

### The shortcut, and what it produced

Point a model at the object, say "render the feed," and you get `feed.tsx`. It handles the shapes that happened to be in its context. **That is your 40%.**

Then a week of bug prompts, each adding an `if` statement, crawling toward 70% before collapsing under its own weight. The common shapes render well. The less common ones are simply unsupported, and the structure has hardened enough that adding them means fighting it.

### The decomposition actually used

| Stage | Produces | Verified by | Why separate |
|---|---|---|---|
| **A** Feed object | Sanitized conversation object | Already trusted upstream | — |
| **B** Recorder | Capture of every shape encountered during real use, with frequency | Compare against live sessions | Shapes cannot be enumerated without observing them. Renders nothing. |
| **C** Catalog | `shapes.ts` per provider — every operation, its shape, its data | Recordings type-check against it | Fixes the contract before any component exists. Renders nothing. |
| **D** Rendering | Components, one shape at a time | Each shape rendered against its recorded fixture | Implementation is now enumerable rather than open-ended |

B and C produce nothing on screen. That is exactly why an unguided model drives through them — and exactly why the shortcut version could pass 481/481 tests while being broken.

The recorder was built *before* implementation began, while using the app normally. Frequency data mattered: it distinguished load-bearing shapes from rare-but-real ones, both of which need support, but which warrant different attention.

---

## Case 2: Conversation ownership

The stage upstream of Case 1 — producing object A in the first place. Same granularity of difficulty.

### The situation

Agent Code runs a simulated terminal (PTY) with Claude Code inside it, replacing the terminal rendering with React. Doing that requires **five channels**:

1. A **proxy** reading Claude Code's API requests to Anthropic in real time — used for streaming
2. **JSONL transcript files** that Claude Code and Codex write — the actual source of truth for conversations
3. **Screen parsing** — needed to react to user questions and permission prompts
4. and 5. further channels feeding the same picture

### What went wrong

With the model making architectural decisions, all five channels ended up fighting each other **inside the rendering layer**. Days of bugs: duplicated messages, conversation entries disappearing, inconsistent ordering.

Every fix was a conditional inside a component that had no business arbitrating between data sources. The bugs presented as rendering bugs. They were ownership bugs.

### What actually fixed it

1. **Isolate the hard part.** Conversation-object construction was pulled completely out of rendering into its own ownership system. Rendering became a pure consumer of one object.
2. **Establish explicit ownership.** Rules for which channel owns which entry under which conditions, producing a single sanitized object stating exactly what is true and what should be shown.
3. **TDD against real fixtures.** Because the ownership rules are genuinely complex, they were built test-first — using fixtures collected from real sessions, never invented ones.

The output of this became input **A** of Case 1. The stages compose.

---

## The shared pattern

**The model could not get there from where it stood.** Roughly **50% of the structural groundwork came from the human**, both times. Not a capability gap at the granular level — the models are excellent there. Arriving at "this substrate is wrong, tear it out" requires declaring current work a dead end, and forward-patching never has to make that call.

**The winning move was deliberately slower.** Recorder, then catalog, then rendering is visibly slower than writing `feed.tsx` on day one — right up until `feed.tsx` stops absorbing changes, at which point the shortcut has cost far more than it saved.

**Isolation was the enabling decision in both cases.** Confine the hard part to one place with a single consumer. Before this was done, features were spread like weed across ten times more files than necessary, so a change in one file broke a completely different domain.

**Green tests proved nothing.** 481 of 481 passing, product broken, because the tests came from the same imagination as the bug. What made the second attempt's tests meaningful was fixtures recorded from real sessions.

**The same cycle applied at both layers.** This is not a one-off trick for rendering. It is the shape the work takes every time a genuinely hard problem appears: isolate it, instrument reality, catalog what is actually there, then implement one case at a time against recorded fixtures.

---

## Case 3: A stalled bug solved by a labeler (prompt delivery)

Prompts were intermittently refused by the composer. Many fixes had been
attempted, each a plausible theory. The decomposition started from the
recorded journals instead: a labeler classified every real refusal. The first
leading theory (orphaned writes) explained 5 of 57 refusals; the rest had a
different cause entirely (a single-witness delivery verdict plus a text-gated
cache latch). One recording showed the durable entry existed **316 ms** after
submit while the waiter had **19.7 seconds** and never fired.

The doc kept a section titled "dead theories, kept so they stay dead" and one
called "rejected as tape". The narrow fix shipped within an hour of the labeler
running. **Lesson:** for a stalled bug, Stage 0 is a labeler over data you
already have, not a new recorder.

## Case 4: The counterexample (skipping capture)

A provider-condition feature was built from a corpus capture plus reasoning,
without a live probe. Its author later wrote: *"precisely the failure mode this
document warned about, committed by its author … I invented the indentation
rule. The 51-test suite passed the whole time."* A live probe then disproved a
hypothesis ("wrapped markers break detection" — they don't) that would
otherwise have been "fixed".

**Lesson:** the method fails exactly where you skip it, even when you wrote the
method. Green tests built on an invented rule are the 481/481 story again.

## Case 5: Stage 0 with hypotheses (a new provider)

Integrating a new agent CLI started with a census script, a live probe and
sanitized fixtures. Invariants were written as hypotheses H1–H10 before any
implementation. Results: five held, three held only in refined forms, and one
design choice became *proven rather than chosen*: turn boundaries could only
come from the live channel, because a transcript-derived rule would have been
wrong about 30% of the time. The refined hypotheses were pinned by a corpus
test. **Lesson:** writing invariants as hypotheses turns design arguments into
measurements.

## Anti-example: review loops that became stages

One decomposition absorbed every review round as a new stage: **~2,000 lines,
48 stages, 12 review gates, 35 commits in three days.** It escaped only by
inventing a bounded final gate: *"another open-ended bug hunt would restart the
same unbounded review loop"*, limiting review to recorded behaviours and
treating unrecorded speculation as an Unknown, not work.

**Lesson:** findings that fit inside a stage go to the PR. Review is bounded.
Past ~25 stages or ~1,000 lines, split into a successor doc.
