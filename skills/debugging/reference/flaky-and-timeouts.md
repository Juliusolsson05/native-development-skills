# Flaky tests and timeouts

## The doctrine

A timeout or `waitFor` failure means "the state I was waiting for never
arrived". It is a symptom. It is never a diagnosis and never a reason to raise
the budget.

- Do not widen the timeout.
- Do not add a retry around the test or the product call.
- Do not rerun CI and call it green.
- Do not label it flaky, pre-existing or environmental until it has been run on
  a clean main checkout, or reproduced deterministically.

A retry is legitimate only as a designed recovery path in the product, with its
own reasoning and tests, never as a way to make a failure disappear.

## Is it mine?

1. `git diff --name-only origin/main...HEAD` — does the branch touch the path?
2. Run the failing test on a detached worktree of `origin/main`.
3. Both fail → pre-existing, file an issue with the evidence. Only yours fail →
   it's your regression, whatever the error message says. A guard that refuses
   to start something surfaces downstream as "the thing I waited for never
   happened".

## Making intermittent deterministic

Name the uncontrolled variable, then pin it:

| Variable | Pin it with |
|---|---|
| Event / message ordering | Replay the recorded order; delay exactly one macrotask to force the other interleaving |
| Network chunking | Feed the recorded chunks in their recorded boundaries |
| Wall clock / timers | Fake or simulated clock |
| Process start readiness | An explicit readiness gate, not a sleep |
| Machine load | Reproduce under load, or remove the load dependence |
| Terminal width / wrapping | Fix the width to the recorded value |
| Disk latency / fs events | Await the condition that follows the write |

## Waiting correctly

Wait on the condition that proves the state arrived (an event, a file, a
flag), with a generous upper bound only as a safety net. A fixed delay needs a
known timing source and a WHY comment explaining it.

## Test pollution

If a test only fails in the full run, bisect by running test files one at a
time (or halving the set) until the polluter appears. Fix the leaked state at
its source.

## Budget sanity

If a test needed a 20-second budget for something that takes 300 ms in
reality, the budget was hiding a hang. Measure the real duration and investigate
the gap.
