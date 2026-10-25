# Symptom signatures

Each row: what you see, the one check that tells the candidates apart, and the
usual cause. These came from real incidents (see case-studies.md); treat the
cause column as a suspicion until the check confirms it.

| Symptom | Discriminating check | Usual cause |
|---|---|---|
| Heap grows / "memory leak" | Force-GC heap snapshot. Small snapshot while heapUsed swings hugely → churn. Dominator tree names one owner → retention. `large_object_space` dominant → big transient strings. | Churn from a hot loop serializing too often; retention by one cache or watcher; whole-file re-reads in a poll loop |
| High CPU, profiler shows the main thread idle | GC runs on background threads; look at GC time and allocation rate | Allocation churn, not computation |
| Heap fine for hours, then spirals | Shape of the curve: sudden, not gradual | A concrete trigger (one input, one loop), not a slow leak |
| Every spawn fails with a generic spawn error | Check the native module's architecture (`file`), which build the loader actually picked (`lsof`) | Native module rebuilt for the wrong arch; silent fallback to a prebuild |
| Process "starts" then exits in ~0 ms, all layers report success | Check the working directory still exists (`git worktree list`, `ls`) | Deleted cwd; the child chdir fails after fork |
| Works the second time | Diff what the first attempt does that the retry skips | Identity or ordering bug on the first path (e.g. an object replaced during wake) |
| "Intermittent" timeout in a test | Run on a clean main checkout; replay the recorded ordering deterministically | A real race in the product or the test, not slowness |
| Garbled or missing characters in a terminal view | Copy the corrupted text. Clipboard clean and a repaint fixes it → renderer. Clipboard wrong → data path. | GPU texture atlas bug vs dimension/data mismatch |
| Tool reports an absurd value (0 bytes free, 0 ms) | Move the underlying quantity and see if the number moves | The API or tool is wrong, not the system |
| Disk fills, `du` of the usual places is small | Check dot-config and app-support dirs; find the writer in source | Unbounded debug/trace output that never rotates |
| Files "revert themselves" | Inspect mounts and sync tools touching the path | A bind mount or copy step overwriting the worktree |
| CI fails on Linux only, passes on macOS | Compare fixture file modes / uid against the container user | Umask or uid mismatch hidden by the desktop VM |
| PR shows "no checks reported" | `gh pr view --json mergeable,mergeStateStatus` | Merge conflict with main; CI never started |
| Our own WHY comment contradicts behaviour | Test the claim against the installed binary | The comment is stale; fix it |
