# Issue tracker: GitHub

Issues and specs for this repo live as GitHub issues. Use the `gh` CLI for all operations.

> **Seed template.** Replace every `<placeholder>` with this project's real values, and delete any
> section that doesn't apply. A section left with placeholders in it is worse than a deleted one —
> agents will try to run it.

## Where

- Owner: `<owner>` (user or org)
- Repo: inferred from `git remote -v` — `gh` does this automatically inside a clone
- Project board: `<board name>`, number `<n>`, id `<PVT_...>` — _delete this line if there's no board_

## The model

- **One issue per requirement**, filed in the repo whose code changes.
- **Grouping is native sub-issues only.** An "epic" is a parent issue whose children are sub-issues;
  children may live in other repos. A parent closes when all its children close. Don't add a second
  grouping mechanism (labels, milestones, custom fields) — one is enough and two always diverge.
- **Work leaf issues only.** An issue with open sub-issues is a container, not a unit of work.
- A leaf does not need a parent. Group under an epic when the work genuinely spans several tickets;
  file a standalone leaf when it doesn't.

## Mandatory body sections

Every issue body carries:

- `## Success Criteria` — binary and command-verifiable: the test name or command, and the expected
  output or file path. Behavioural, not implementational. Keep it to ~6 bullets.
- `## Out of scope` — what this issue deliberately doesn't touch.

**The body is the only design channel.** Issues are implemented by separate agents that never see the
conversation that produced them, so every issue must be in exactly one of two states:

1. **Design locked** — every decision made while planning is written into the body: exact models and
   signatures, algorithms as code rather than prose, constants and their values, file paths, verbatim
   schemas or manifests, explicit delete lists. Say so plainly: _"All design below is LOCKED —
   implement exactly this; do not redesign, rename, or improve."_ Design questions that surface
   during implementation go back to the creator as a comment, not resolved ad hoc in the PR.
2. **Analysis required** — if the design wasn't settled at creation time, the body opens with an
   `## Analysis required` section listing what's undecided, where to look, and the instruction to do
   that analysis first and record the decisions in the issue body before writing code.

Never leave an issue between the two. Silent ambiguity ("figure out the details") is a defect in the
issue, not a task for the implementer.

## Working an issue

1. Read the **full parent chain** — the work serves the epic's intent, not just the leaf.
   `gh issue view <n> --repo <owner>/<repo> --comments`
2. Set board Status → **In Progress**. _(Skip if there's no board.)_
3. Branch from `main`. Open a **PR whose body ends with `Closes #<n>`** so the merge auto-closes the
   issue. `<any trailer conventions or prohibitions>`
4. Verify **every** success criterion by running the command or test it names.
5. Merge → the issue auto-closes. Set board Status → **Done**. Add a completion-summary comment if
   the PR body doesn't cover it.
6. When all sub-issues of a parent are closed, close the parent with a one-line comment and mark it
   Done too.

## Creating an issue

One issue per requirement: what, why, success criteria. Write it for an agent working independently —
background, implementation detail, file paths, no open questions. Size it into sub-issues rather than
splitting one requirement across several. After creating a **leaf**, link it under its parent and add
it to the board with Status → Todo; an **epic** gets its children linked but stays off the board.

## Conventions

- **Create an issue**: `gh issue create --title "..." --body-file body.md`. Use a file or heredoc for multi-line bodies.
- **Read an issue**: `gh issue view <number> --comments`, filtering comments by `jq` and also fetching labels.
- **List issues**: `gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'` with appropriate `--label` and `--state` filters.
- **Comment on an issue**: `gh issue comment <number> --body "..."`
- **Apply / remove labels**: `gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **Close**: `gh issue close <number> --comment "..."`
- **Sub-issue link** (works cross-repo; needs node ids, not numbers):
  `gh api graphql -f query='mutation{addSubIssue(input:{issueId:"<parent-node-id>",subIssueId:"<child-node-id>"}){issue{number}}}'`
  where a node id comes from `gh api repos/<owner>/<repo>/issues/<n> --jq .node_id`.

### Board writes

_Delete this section if there's no board._ Projects v2 is not covered by the GitHub MCP server —
board reads and writes go through `gh project ...`, which needs the `project` token scope.

```bash
gh project item-add <n> --owner <owner> --url <issue-url> --format json   # → item id (PVTI_...)
gh project item-edit --project-id <PVT_...> --id <item-id> \
  --field-id <field-id> --single-select-option-id <option-id>
```

Record the field and option ids here once, so no one re-queries them:

| Field | Field ID | Options |
|---|---|---|
| `<Status>` | `<PVTSSF_...>` | `<option-id>` Todo · `<option-id>` In Progress · `<option-id>` Done |

> ⚠️ Mutating a single-select field's options **regenerates every option id** for that field
> (`updateProjectV2Field` takes no option `id`), orphaning already-tagged items. Don't mutate them;
> if unavoidable, pass the full existing option set and re-query the ids afterwards.

## Pull requests as a triage surface

**PRs as a request surface: no.** _(Set to `yes` if this repo treats external PRs as feature requests; `/triage` reads this flag.)_

When set to `yes`, PRs run through the same labels and states as issues, using the `gh pr` equivalents:

- **Read a PR**: `gh pr view <number> --comments` and `gh pr diff <number>` for the diff.
- **List external PRs for triage**: `gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments` then keep only `authorAssociation` of `CONTRIBUTOR`, `FIRST_TIME_CONTRIBUTOR`, or `NONE` (drop `OWNER`/`MEMBER`/`COLLABORATOR`).
- **Comment / label / close**: `gh pr comment`, `gh pr edit --add-label`/`--remove-label`, `gh pr close`.

GitHub shares one number space across issues and PRs, so a bare `#42` may be either — resolve with `gh pr view 42` and fall back to `gh issue view 42`.

## When a skill says "publish to the issue tracker"

Create a GitHub issue.

## When a skill says "fetch the relevant ticket"

Run `gh issue view <number> --comments`.

## Wayfinding operations

Used by `/wayfinder`. The **map** is a single issue with **child** issues as tickets.

- **Map**: a single issue labelled `wayfinder:map`, holding the Notes / Decisions-so-far / Fog body. `gh issue create --label wayfinder:map`.
- **Child ticket**: an issue linked to the map as a GitHub sub-issue (`gh api` on the sub-issues endpoint). Where sub-issues aren't enabled, add the child to a task list in the map body and put `Part of #<map>` at the top of the child body. Labels: `wayfinder:<type>` (`research`/`prototype`/`grilling`/`task`). Once claimed, the ticket is assigned to the driving dev.
- **Blocking**: GitHub's **native issue dependencies** — the canonical, UI-visible representation. Add an edge with `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`, where `<blocker-db-id>` is the blocker's numeric **database id** (`gh api repos/<owner>/<repo>/issues/<n> --jq .id`, _not_ the `#number` or `node_id`). GitHub reports `issue_dependencies_summary.blocked_by` (open blockers only — the live gate). Where dependencies aren't available, fall back to a `Blocked by: #<n>, #<n>` line at the top of the child body. A ticket is unblocked when every blocker is closed.
- **Frontier query**: list the map's open children (`gh issue list --state open`, scoped to the map's sub-issues / task list), drop any with an open blocker (`issue_dependencies_summary.blocked_by > 0`, or an open issue in the `Blocked by` line) or an assignee; first in map order wins.
- **Claim**: `gh issue edit <n> --add-assignee @me` — the session's first write.
- **Resolve**: `gh issue comment <n> --body "<answer>"`, then `gh issue close <n>`, then append a context pointer (gist + link) to the map's Decisions-so-far.
