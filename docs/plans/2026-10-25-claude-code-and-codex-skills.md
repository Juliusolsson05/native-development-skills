# Skills: colleagues follow the same development principles in plain Claude Code and Codex

Fixes #1.
Branch `feat/claude-code-and-codex-skills` · Worktree `.worktrees/claude-code-and-codex-skills` · Base `origin/main` @ 0ea0270 (2026-10-25)
Status: approved 2026-10-25 (Julius: "give it a go and set up the repo and just try to do all of this stuff"), implemented, PR open, merge needs explicit approval

## Outcome
A colleague who uses plain Claude Code adds this repo as a plugin marketplace,
installs one plugin, and gets the development conventions plus `writing-plans`,
`debugging`, `staged-decomposition` and `pr-review`, with a reviewer subagent
that `pr-review` launches twice in parallel. A colleague who uses Codex gets the same
skills in `~/.agents/skills` and the reviewers as Codex custom agents. No file
refers to Agent Code or its tools.

## Evidence (verified, do not re-derive)
- The four process skills in `Juliusolsson05/agent-skills` @ 25e206f call no
  Agent Code tool. Agent Code appears only as the setting of two background
  stories (`debugging/reference/case-studies.md:3`,
  `staged-decomposition/reference/worked-examples.md:3`). grep, 2026-10-25.
- The conventions exist only as Agent Code's generated
  `~/.claude/skills/agent-code-conventions/SKILL.md` (marker
  `agent-code-managed:v1`, user-authored section below it).
- `pr-review` @ 25e206f says "two reviewers in parallel ... use the reviewer
  providers the user has chosen" but not how to start them; Agent Code users get
  that from its orchestration tools. Plain Claude Code and Codex users need the
  mechanism spelled out.
- Codex (official docs, learn.chatgpt.com/docs/agent-configuration/subagents,
  fetched 2026-10-25): custom agents are TOML files in `~/.codex/agents/` or a
  repo's `.codex/agents/`; required `name`, `description`,
  `developer_instructions`; optional `model`, `model_reasoning_effort`. Agents
  cannot be made read-only by their own file: they inherit the parent's sandbox
  and approval settings. Subagents start only when the prompt (or a skill /
  AGENTS.md) asks for them explicitly.
- Codex user skills live in `$HOME/.agents/skills` (developers.openai.com/codex/skills).
- Claude Code plugin and subagent formats: see "Claude Code facts" below,
  checked against the current docs before writing the manifests.

## Decisions
1. **Where do the shared skills live?** *Ruling: a new repo with copies*, not a
   rename of agent-skills — the user said pr-review is "very different" for
   native users, so one shared copy would no longer fit Agent Code users too —
   cost if wrong: three identical skills kept in two repos, synced by hand
   (README records the source commit).
2. **Tools:** Claude Code first (plugin marketplace), Codex second (`npx skills
   add` + agent TOML files to copy).
3. **Conventions delivery:** a skill that triggers at the start of every task,
   as in Agent Code; README offers an optional always-on `CLAUDE.md` / `AGENTS.md`
   line.
4. **Conventions source of truth:** this repo. Agent Code users skip this skill.
5. **Conventions text:** obvious typos fixed only; no rewording.
6. **`/simple-code`, `/no-overengineering`:** left out (authorship and licence
   unknown). UNCONFIRMED — the owner can add them.
7. **`gen-split`, `autonomous-loops`, goal/TLDR:** left out (Agent Code-specific
   or not a principle). Follow-up issue for a native loops skill.
8. **Owner/visibility:** `Juliusolsson05`, public, MIT.

## Design
- `skills/development-conventions/SKILL.md` — frontmatter `name`,
  `description` ("use at the beginning of every task in a software project ...");
  body: precedence note (conversation and repo-local instructions win), the
  conventions text, the delegation rule (carry conventions into subagent briefs).
- `skills/{writing-plans,debugging,staged-decomposition}` — verbatim copies of
  agent-skills @ 25e206f.
- `skills/pr-review/SKILL.md` — same doctrine (one round, independent read-only
  reviewers, findings are claims, disposition table, merge only on explicit
  approval). "Requesting review" is rewritten around native subagents:
  - Claude Code: two `pr-reviewer` subagent calls in **one message** (parallel),
    each prompt a one-line pointer to its brief file; each in an isolated git
    worktree so mutation testing never touches the author's tree.
  - Codex: ask explicitly to "spawn two pr-reviewer agents, wait for both".
    The session stays in its usual `workspace-write` sandbox, because reviewers
    must write to mutate. `.git` is protected there, so the author creates and
    removes the review worktrees (approving the git write) and reviewers undo
    mutations by editing, not with git. Nothing enforces "only your worktree"
    beyond the brief — stated in the skill. (Revised after review; the draft
    said "read-only sandbox", which would have made mutation impossible.)
  - A second provider as reviewer is optional (e.g. `codex exec` from Claude
    Code), never required.
- `agents/pr-reviewer.md` — Claude Code subagent: reviewer role, required
  techniques (mutation, counting the corpus), report shape. Tools limited to
  what a reviewer needs; edits allowed only because it runs in its own worktree
  for mutations, and it never commits.
- `codex/agents/pr-reviewer.toml` — the same reviewer for Codex, to copy into
  `~/.codex/agents/` (or a repo's `.codex/agents/`).
- `.claude-plugin/marketplace.json` + `.claude-plugin/plugin.json` — one
  marketplace whose single plugin is the repo root.
- `README.md` — who it's for, install per tool, how the skills fit (diagram from
  agent-skills), what's deliberately missing, source commit of the copies.

Invariant: no shipped file names an Agent Code tool. Checked by grep before the PR.

## Claude Code facts
Checked against code.claude.com/docs (plugins/marketplace-reference,
plugins/manifest-reference, plugins/install, sub-agents, plugins/cli-reference)
on 2026-10-25, Claude Code 2.1.310 installed locally.
- `marketplace.json` needs `name`, `owner.name`, `plugins[]` (each `name`,
  `source`); `"source": "./"` makes the repo root the plugin.
- `plugin.json` needs only `name`. `skills/` and `agents/` at the plugin root
  are auto-discovered. Setting `version` pins users until it is bumped.
- Install: `/plugin marketplace add <owner>/<repo>`, then
  `/plugin install <plugin>@<marketplace>`; shell `claude plugin ...` mirrors it.
  Auto-update is off by default for third-party marketplaces.
- Plugin skills are `<plugin>:<skill>`; plugin agents are `<plugin>:<agent>`.
- Subagent frontmatter: `name`, `description` required; `tools` allowlist,
  `model`, `effort`, `isolation: worktree`, `color`. Unknown fields are ignored
  silently. Plugin agents ignore `hooks`, `mcpServers`, `permissionMode`,
  `initialPrompt`.
- `isolation: worktree` branches from the **default branch**, so a reviewer
  must check out the PR head itself before reading. Not explicitly confirmed for
  plugin agents → verified by a real run.
- Validate with `claude plugin validate . --strict`.

Rulings from these facts:
- Ruling: plugin name `native-dev`, marketplace `native-development-skills` —
  skills and agent read as `native-dev:pr-reviewer`, short enough to type —
  cost if wrong: a rename, which breaks existing installs once.
- Ruling: no `version` in the manifests — every pushed commit is the release,
  which suits a small team repo of prose — cost if wrong: no way to pin a known
  good version.
- Ruling: reviewer tools `Read, Grep, Glob, Bash, Edit` with
  `isolation: worktree` — mutation testing needs edits, the worktree keeps them
  away from the author's tree, and `permissionMode` can't be used by plugin
  agents — cost if wrong: a reviewer could still `git push` through Bash; the
  agent instructions forbid it.

## Tests
These are prose skills; there is no code to unit-test. Verification instead:
- Plugin validation command passes on the repo.
- Install into a throwaway Claude Code config from the local path: plugin
  installs, five skills and the `pr-reviewer` agent are listed.
- `npx skills add <local path> --list` (or equivalent) sees the five skills.
- grep for `goal_set|tldr_update|orchestration_|ac_agents|Agent Code tools` in
  shipped files returns nothing outside the background stories.

## Verification
Verification boundary: I cannot run it on a colleague's machine, run a real
Codex review round end to end, or see whether the conventions skill triggers
reliably on every task in someone else's sessions. These go in the PR as manual
steps.

## Out of scope
- Native autonomous-loops skill → follow-up issue.
- Changing agent-skills (it stays the Agent Code-flavoured home).
- `/simple-code`, `/no-overengineering` (decision 6).

## Execution notes
- 2026-10-25: plan cff284b; copies cb894df; conventions 778478c; pr-review +
  agents 870af63; manifests 82142fa; README aa64ca8. `claude plugin validate .`
  passes (warnings: no version — deliberate, falls back to the commit SHA; no
  icon). Clean-config install from the local path lists 5 skills + `pr-reviewer`.
  `npx skills add <path> -g -a codex` into a temp HOME wrote all five to
  `~/.agents/skills`.
- 2026-10-25: pr-review dogfooded on this branch: headless `claude -p
  --plugin-dir .` launched two `native-dev:pr-reviewer` subagents in one
  message. Both ran in their own `.claude/worktrees/agent-*` worktree starting
  at main and checked out the head SHA as instructed (plugin `isolation:
  worktree` works). Both worktrees and their `worktree-agent-*` branches were
  left behind although clean → cleanup step added to the skill.

## Review (2026-10-25, one round)
| Finding | Verdict | Disposition |
|---|---|---|
| Codex: git-ignored folder doesn't make it writable; `.git` is protected under workspace-write, so worktree add / checkout / remove fail or need approval | valid | fixed: author creates/removes worktrees with approval, reviewers revert by editing |
| Codex → `claude -p` needs network and `~/.claude` writes, fails inside the sandbox | valid | fixed: documented as needing escalation, read-only |
| Claude Code → `codex exec` uses a worktree nobody created; read-only reviewer still told to mutate | valid | fixed: create-worktree step, drop mutation from read-only briefs |
| Plan says Codex review runs read-only; skill doesn't | valid | fixed: plan Design revised |
| "Briefs differ only in focus area" contradicts per-reviewer setup | valid | fixed |
| Conventions wrapper reworded / delegation rule narrowed to subagents | overstated (wrapper was planned) | delegation rule widened back; README says there's a wrapper |
| "Name the model" missing for Codex; "Replaces superpowers" dropped | valid | fixed |
| README `@`-import only for npx installs | valid | plugin path added (marketplace clone path, not verified for GitHub installs) |
| Plugin worktrees may not auto-clean | valid, confirmed by the run | fixed: cleanup step + red flag; `.claude/worktrees/` ignored |
| Agent TOML accepts `sandbox_mode` | wrong | declined: official docs say it doesn't override the parent |

## Plan vs built
| Section | Status | Note |
|---|---|---|
| Outcome | MATCH | |
| Decisions 1–8 | MATCH | 6 stays UNCONFIRMED |
| Design: conventions, copies, pr-review, agents, manifests, README | MATCH | Codex bullet revised after review (see Review) |
| Tests / Verification | MATCH | Codex end-to-end spawn not run (boundary) |
| Out of scope | MATCH | loops follow-up is #2 |
