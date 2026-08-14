# mp-modified-skills

A fork of [mattpocock/skills](https://github.com/mattpocock/skills), trimmed to the engineering flow
the Loom workspace actually runs and adapted to its conventions. **This repo holds skills only — it
is not the workspace.** Workspace rules live in `/home/klad/loom/CLAUDE.md`.

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

Keep these when merging upstream changes; they are deliberate, not drift.

- **`to-tickets` locks the design into the ticket body** — exact paths, signatures, models,
  "implement exactly this", plus a mandatory `## Success Criteria` and an explicit
  `## Analysis required` for anything unsettled. Upstream says to keep file paths out of tickets;
  our tickets are worked by agents that never saw the design conversation, so a stale path costs one
  correction while a re-decided design costs the whole ticket. Local-file tracker mode is removed.
- **`implement` enforces the workspace hard gates** — `uv run pytest --cov` at ≥80%, pyright,
  docs-before-code, branch → PR ending `Closes #<n>` → merge → board Status → Done, and it reaches
  for the repo's own scaffolding skill (`new-loom-module` and friends) rather than re-deriving layout.
- **`setup-matt-pocock-skills` is GitHub-only** — GitLab and local-markdown templates are gone. Every
  repo here is under the `ketanlad` owner on the **Loom Agent** board.

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
claude plugin marketplace add /home/klad/loom/mp-modified-skills
```

then install `mattpocock-skills`. `plugin.json` controls exactly what ships, and it's project-scoped.
`scripts/link-skills.sh` is the alternative — it symlinks every skill into `~/.claude/skills`
user-wide, ignoring `plugin.json`. Use one route, not both, or every skill appears twice.
