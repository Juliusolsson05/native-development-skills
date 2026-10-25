# Case studies

Real misdiagnoses from [Agent Code](https://github.com/Juliusolsson05/agent-code),
each as: symptom → wrong first theory → decisive evidence → what it was → lesson.

## "Flaky CI test" that was a real client bug (#847 → PR #911)

- **Symptom:** a websocket feed integration test failed intermittently in CI,
  seven sightings.
- **Wrong first theory:** the default 1 s wait was too short; raise it.
- **Decisive evidence:** the standing rule "don't raise timeouts to silence
  flakes" held; a deterministic replay of the message ordering reproduced it.
- **What it was:** a stale session list evicted a mounted view's state and
  listeners depending on TCP chunking. The same race hit phones on reconnect.
- **Lesson:** a timeout is a symptom. Seven sightings of "flaky" was seven
  sightings of a bug.

## "Memory leak" that was GC churn (#390)

- **Symptom:** main process at 80% CPU, heap swinging to 1.2 GB.
- **Wrong first theory:** a leak.
- **Decisive evidence:** a force-GC heap snapshot showed ~45 MB reachable while
  heapUsed swung 46 MB ↔ 1.2 GB; the CPU sampler showed the main thread idle
  because GC runs on background threads.
- **What it was:** a 60 Hz four-way screen serialization per session.
- **Lesson:** measure retention versus churn before hunting an owner.

## Three wrong OOM owners (#288 → PRs #319, #322)

- **Symptom:** out-of-memory crashes.
- **Wrong first theories:** full transcripts in RAM; a sidecar doubling data;
  terminal buffers. Each got a "shrink" fix.
- **Decisive evidence:** a dominator/retainer analysis tool was built first.
  It showed 7 watcher instances retaining ~85% of the heap, plus two V8 traps
  (a sliced string pinning its parent, bulk in an un-walked field).
- **What it was:** retention by one watcher's map, fixed by a derive-and-drop
  redesign.
- **Lesson:** build the instrument before naming the owner.

## "Regression from the latest merges" that was deleted worktrees

- **Symptom:** every agent session came up as Error after a batch of merges.
- **Wrong first theory:** one of the merged PRs broke startup.
- **Decisive evidence:** the incident journal showed `start ok` in 15 ms then
  `exited`; `git diff --name-only` showed the spawn path untouched;
  `git worktree list` showed the session directories as prunable.
- **What it was:** sessions pointing at deleted worktrees; the PTY library
  chdirs inside the forked child and exits, so every layer reported success.
- **Lesson:** "is it my change?" is answered by the diff, not by timing.

## "Delivery failure" that was an ownership break (#632)

- **Symptom:** prompts appeared stuck in a queue.
- **Wrong first theory:** terminal input wasn't being delivered.
- **Decisive evidence:** recordings showed 4 successful submits, thousands of
  semantic events, and zero transcript entries.
- **What it was:** the upstream CLI stopped writing the event our ownership
  logic keyed on.
- **Lesson:** count what the recordings contain before theorizing about the path.

## "Intermittent" absorption timeout that was deterministic (#1113 → PR #1115)

- **Symptom:** image prompts sometimes timed out waiting to be absorbed.
- **Wrong first theory:** timing.
- **Decisive evidence:** counting literal versus whitespace-normalized matches
  across every recorded frame.
- **What it was:** the TUI word-wrapped `[Image #1]` at its only space; the text
  path normalized whitespace, the image path didn't.
- **Lesson:** "intermittent" meant an uncontrolled variable (wrap width).

## Equivalent mutant claims (3 of 4 wrong)

- **Symptom:** code comments claimed some mutations were "equivalent" (not
  detectable by any test).
- **Decisive evidence:** an adversarial reviewer built the distinguishing state
  for three of them.
- **Lesson:** "no surviving mutants" is a claim about the test suite you chose,
  not about the code.
