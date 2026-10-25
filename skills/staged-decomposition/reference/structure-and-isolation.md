# Structure and isolation

## Contents

- Vibecoding vs agentic development
- The two shapes a codebase can take
- LOC thresholds
- What isolation actually means
- Untangling as continuous work
- Directory structure as a real constraint

---

## Vibecoding vs agentic development

These are different activities and the distinction matters for how you behave.

**Vibecoding:** no knowledge of structure or design principles. Fixing an issue means describing the symptom visible on screen and asking for a fix. Structural decisions are made implicitly by the model, one prompt at a time.

**Agentic development:** the human is the engineer guiding and deciding structural patterns, but does not write the lines themselves. The model handles granular implementation — which it is genuinely excellent at — while the human owns where things live and what may depend on what.

The models are strong enough at the granular level that structure is where the remaining human leverage is concentrated.

## The two shapes a codebase can take

**Vibecoded — a living organism.** Every part of the tree imports and stretches over every other part. A single feature touches 40 of 100 files. There are no seams. A change in one file breaks a completely different domain, and there is no way to predict which, because the dependency graph is dense and undocumented.

**Agentically developed — bounded modules.** Complicated features are confined, in code and in directory layout. Each has a small, explicit surface. Changes stay local. New features do not add to a compounding tech-debt mountain, because they attach at known seams.

The main job of an agentic developer is to **untangle as the models work.** Models produce correct-in-isolation code that gradually spreads a feature across far more files than it needs. Left alone this converges on the first shape.

## LOC thresholds

Background, not the trigger. The skill applies on risk (several sources of
truth, a stalled attempt, an unrecorded subsystem, a boundary crossing); LOC
only describes how fast debt compounds once it exists. Tiny decomposition docs
in small repos have paid off; a large repo with a known shape doesn't need one.

Rough, empirical, from a ~300k-line codebase written this way:

| Size | What happens |
|---|---|
| **Under 20-35k** | Raw model capability handles it. Tech debt too small to matter. Skip this methodology; it is overhead. |
| **~30k** | Nightmare territory for pure vibecoding, though it is survivable if you push. |
| **~70k** | Theoretical ceiling for surviving with no structural discipline. Life support only. |
| **Well beyond** | Only reachable with deliberate isolation. Reference point: ~300k lines maintained this way, versus a plausible 800k+ had LOC reduction not been active work. |

LOC is not a productivity metric here. A large fraction of engineering time goes into *reducing* it. The number matters only as a proxy for how much surface area exists to tangle.

## What isolation actually means

Isolation is not "the file is in its own folder." Test it:

- **Single consumer.** Exactly one thing downstream depends on this module's output. If two consumers both reach into internals, the boundary is decorative.
- **Explicit forbidden list.** You can name what may *not* import this, and that constraint is written down somewhere.
- **Independently verifiable.** You can confirm the module is correct without running anything downstream of it.
- **Replaceable.** You could reimplement the internals without touching consumers.
- **Directory-level containment.** The feature lives in its own directory. Locating everything that implements it does not require searching the whole tree.

If a subsystem fails these, it is not isolated regardless of how it is organised on disk.

## The reconciliation rule

When several sources of truth must produce one answer, the reconciliation gets its own isolated layer that emits a single clean object.

The failure mode when this is skipped: the reconciliation ends up distributed across consumers, each making local decisions about which source wins. The consumers then fight. The bugs look like presentation bugs — duplicated entries, items disappearing, ordering that shifts between runs — but they are ownership bugs, and no amount of work in the presentation layer fixes them.

Symptoms that this is the actual problem:

- The same entity appears twice, inconsistently
- Items vanish under specific timing
- Fixes in one consumer break a different consumer
- Nobody can state which source of truth wins in a given case

### Pipeline form: one owner per signal

For multi-source work, an isolation table is often clearer as a pipeline where
every signal has exactly one owner:

| Signal | Owner | Mechanism |
|---|---|---|
| Turn boundaries | Live channel | Only the live channel may declare a turn ended |
| Committed history | Durable transcript | Tail + project onto the active branch |
| Screen text | Nobody | Observed for prompts only; never becomes history |

Rules that make it work:

- **A "Nobody" row** for signals you deliberately ignore, so no consumer quietly
  starts depending on them.
- **The doorbell rule**: a fast channel may *wake* the owner ("something
  changed"), but only the owner decides what is true.
- A signal with two owners is the bug, whatever the symptom looks like.

### Hard rules for code in someone else's process

When part of the system runs inside a foreign process (an extension loaded
into a CLI, a script injected into a page), write its hard rules down: what it
may never block, never throw, never write. That code can't be debugged like
your own.

### Enforcing boundaries

A structural test that fails when a forbidden importer appears is an option,
not a requirement. Offer it when a boundary keeps getting crossed; otherwise
the WHY comment at the boundary is enough.

## Untangling as continuous work

Untangling is not a phase. It is ongoing maintenance performed while the models work.

Practical signals that something needs untangling now:

- A feature's implementation is spread across many more files than its conceptual size suggests
- Changing one file breaks a different domain
- You cannot state the boundary of a feature without listing files
- The same logic appears in several consumers with small variations

The corrective prompt is structural, not symptomatic:

> "It feels like our rendering system is spread out over the codebase. Should we not take this and isolate it into its own `/rendering` directory?"

This is what prompt engineering actually is. Not phrasing rituals — decisions about where things live and what may depend on what.
