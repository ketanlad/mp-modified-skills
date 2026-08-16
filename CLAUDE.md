# mp-modified-skills

A fork of [mattpocock/skills](https://github.com/mattpocock/skills), trimmed to the engineering flow
we actually run.

**The skills stay generic.** No repo names, paths, commands, or stack assumptions belong in a
`SKILL.md`. Everything project-specific is read at runtime from the consuming repo's own files —
`docs/agents/issue-tracker.md`, `docs/agents/domain.md`, `docs/agents/triage-labels.md`, `CONTEXT.md`,
`docs/adr/`, and the repo's `CLAUDE.md`/`AGENTS.md`. That's what makes this set droppable into any
project, so keep it that way: if a change would only be true here, it belongs in one of those files
instead.

## What survived the fork, and why

23 skills in two buckets. `misc/`, `in-progress/`, `deprecated/`, `teach` and `to-questionnaire` were
deleted, along with the upstream publishing apparatus (`docs/`, changesets, release workflow,
`package.json`, every `agents/openai.yaml` — Codex-only metadata we don't use).

The spine is `setup-matt-pocock-skills` (once per repo) → `grill-with-docs` → `to-spec` →
`to-tickets` → `implement` (drives `tdd`, closes with `code-review`). `grilling`,
`domain-modeling` and `codebase-design` are load-bearing vocabulary layers under it — 3–5 skills
each invoke them, so don't delete them. Everything else is an on-ramp (`triage`, `diagnosing-bugs`,
`wayfinder`) or standalone.

[`ask-matt`](./skills/engineering/ask-matt/SKILL.md) is the router and the map. It hard-names every
user-reachable skill, so **any skill you add, rename, or remove means editing `ask-matt`** — a router
that lies is worse than no router.

## Local deviations from upstream

Three skills differ from upstream, plus one seeded config file. All stay stack-agnostic. Keep them when
merging upstream changes — they're deliberate, not drift.

- **`to-tickets` locks the design into the ticket body** — exact paths, signatures and models stated
  as "implement exactly this", plus a mandatory `## Success Criteria` (each criterion naming the
  command that verifies it), an explicit `## Analysis required` for anything unsettled, and
  `## Out of scope`. Upstream deliberately keeps file paths out of tickets so they don't go stale.
  We invert that because our tickets are worked by fresh agents that never saw the design
  conversation: a stale path costs one correction, an agent re-deciding a settled design costs the
  whole ticket. Only adopt upstream's version if you move to tickets a human implements.
- **`implement` treats the repo's documented gates as hard gates** — it reads
  `docs/agents/issue-tracker.md` for ship mechanics and `CLAUDE.md`/`AGENTS.md`/`CONTRIBUTING.md` for
  the test command, coverage threshold and typechecker, verifies every success criterion by actually
  running it, and reaches for a repo's own scaffolding skill when one fits. Upstream's version is six
  lines and assumes you're watching.
- **The GitHub tracker seed carries our issue conventions** —
  `setup-matt-pocock-skills/issue-tracker-github.md` seeds the design-locked vs `## Analysis required`
  split, mandatory `## Success Criteria` / `## Out of scope`, and the epics-off-the-board rule. It
  deliberately does NOT require every leaf to hang off an epic, and does not archive board items on
  close — both were tried and were pure overhead. Placeholders in the seed are meant to be filled in
  per project; a section left with `<placeholder>` text in it should be deleted, not shipped.
- **`grilling` delivers rounds one answer-effort at a time** — choice-shaped questions go through
  the AskUserQuestion tool (≤4 per call, recommended option first); open-ended questions are asked
  one at a time in chat. Upstream asks the whole frontier as one numbered list. We invert that
  because a human answers every round: clicks are cheap, essays are not.

## Conventions

- Each skill is either **user-invoked** (`disable-model-invocation: true` — only the human can fire
  it; may call model-invoked skills, never another user-invoked one) or **model-invoked** (no flag,
  description carries trigger phrases). See [.agents/invocation.md](./.agents/invocation.md).
- Dependencies between skills are **prose `/skill` invocations**, never `../other-skill/FILE.md`
  cross-references. Shared reference docs live inside the skill that owns them.
- Each bucket's `README.md` and the top-level `README.md` list every skill in it, grouped
  User-invoked / Model-invoked, name linked to its `SKILL.md`.
- `.claude-plugin/plugin.json`'s `skills` array is the install allowlist and must match the folders
  on disk. Run `claude plugin validate . --strict` after touching it.
- Writing or editing a skill? Use the `writing-for-agents` skill.

## Install

This fork is its own single-plugin marketplace. From the workspace:

```bash
claude plugin marketplace add ./mp-modified-skills
```

then install `mattpocock-skills`. `plugin.json` controls exactly what ships. The marketplace is recorded in user settings, so the install is user-scoped.
`scripts/link-skills.sh` is the alternative — it symlinks every skill into `~/.claude/skills`
user-wide, ignoring `plugin.json`. Use one route, not both, or every skill appears twice.
