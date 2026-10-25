# native-development-skills

How we build software with coding agents, as skills for plain **Claude Code**
and **Codex**. Install once and your agent follows the same conventions and
process: an issue and a plan before a PR, evidence before a bug fix, staged work
for risky changes, and two independent reviewer subagents before anything
merges. Merging always waits for a human.

| Skill | Use it for |
|---|---|
| [`development-conventions`](skills/development-conventions/SKILL.md) | Every task: issues, worktrees and branches, plans, Conventional Commits, PRs, testing, the completion standard |
| [`writing-plans`](skills/writing-plans/SKILL.md) | Any feature, fix or refactor that lands as a PR: read first, confirm understanding, decisions with defaults, plan as the first commit, build it all, keep the plan true |
| [`debugging`](skills/debugging/SKILL.md) | Bugs, regressions, flaky tests, crashes: evidence first, root cause before fix, regression test from recorded data proven by reverting the fix |
| [`staged-decomposition`](skills/staged-decomposition/SKILL.md) | Risky work: several sources of truth, a stalled 40–70% attempt, an unrecorded subsystem. Evidence → catalog → stages, tests from real recordings |
| [`pr-review`](skills/pr-review/SKILL.md) | One round of two independent reviewer subagents, findings checked against the code, merge only on explicit approval |

Plus one reviewer agent, `pr-reviewer`, which `pr-review` launches twice in
parallel: [Claude Code version](agents/pr-reviewer.md),
[Codex version](codex/agents/pr-reviewer.toml).

## How they fit

```
request
  ├─ trivial ─────────────────────────► just do it
  ├─ bug / flaky / crash ─────────────► debugging ──(2 failed fixes)──┐
  ├─ risky / unknown case set ────────► staged-decomposition ◄────────┘
  │                                          │ approved
  └─ feature / refactor, known shape ─► writing-plans (tasks = stages)
                                             │
                        read a lot → recite + decisions → plan commit
                        → build all of it → tests from real data
                        → pr-review (2 subagents, one round) → PR → wait for "merge"

development-conventions applies throughout.
```

## Install

### Claude Code

In a Claude Code session:

```text
/plugin marketplace add Juliusolsson05/native-development-skills
/plugin install native-dev@native-development-skills
```

Or in one step from the shell (Claude Code 2.1.292 or later):

```bash
claude plugin install native-dev --marketplace Juliusolsson05/native-development-skills --scope user
```

Skills show up as `native-dev:<skill>` and the reviewer as
`native-dev:pr-reviewer`. The agent picks skills up automatically; you can also
call one directly, e.g. `/native-dev:writing-plans`.

**Updates.** Auto-update is off by default for marketplaces like this one.
Turn it on in `/plugin` → Marketplaces → `native-development-skills` →
Enable auto-update, or update by hand:

```bash
claude plugin marketplace update native-development-skills
claude plugin update native-dev@native-development-skills
```

### Codex

Skills, into `~/.agents/skills`:

```bash
npx skills add Juliusolsson05/native-development-skills -g -a codex
```

The reviewer agent, which Codex reads from `~/.codex/agents/`:

```bash
mkdir -p ~/.codex/agents
curl -fsSL https://raw.githubusercontent.com/Juliusolsson05/native-development-skills/main/codex/agents/pr-reviewer.toml \
  -o ~/.codex/agents/pr-reviewer.toml
```

Codex only starts subagents when asked; `pr-review` tells it exactly what to
ask for.

### Anything else

Copy `skills/<name>/` into your agent's skill folder (`~/.claude/skills/`,
`~/.agents/skills/`). Without the reviewer agent, `pr-review` falls back to a
general-purpose subagent in a worktree; the reviewer brief carries the full
rules.

## Want the conventions on every turn?

Skills load when the agent decides they're relevant. If you'd rather have the
conventions always in context, import them from your personal instructions
file. For Claude Code, add a line to `~/.claude/CLAUDE.md` pointing at the
installed `SKILL.md`, e.g. `@~/.agents/skills/development-conventions/SKILL.md`
if you also installed with `npx skills`. A repo's own `CLAUDE.md` /
`AGENTS.md` still wins where it disagrees.

## What's deliberately not here

- **Autonomous loops** (manager + workers, overnight runs). The versions we use
  depend on Agent Code's orchestration tools; a plain Claude Code version is a
  follow-up.
- **Goal and status reporting tools.** They need Agent Code. The skills still
  ask for a short TLDR at the end of each piece of work.

## Where this comes from

`writing-plans`, `debugging` and `staged-decomposition` are copied unchanged
from [Juliusolsson05/agent-skills](https://github.com/Juliusolsson05/agent-skills)
@ `25e206f`, the version for [Agent Code](https://github.com/Juliusolsson05/agent-code)
users. `pr-review` is rewritten here to use native subagents, and
`development-conventions` is the conventions text Agent Code otherwise injects.
When agent-skills changes, sync the three copies by hand and update the commit
above.

The staged-decomposition method is explained in the essay
[Misadventures in Agentic Development](https://www.juliusolsson.com/blog/agentic-development).

## License

MIT
