---
name: implement
description: Implement a ticket or spec end to end — read the parent chain, follow the repo's issue-tracker and quality gates, drive /tdd, and close out with /code-review. Use when the user asks for a ticket, issue or spec to be implemented, or when an agent is handed one to work autonomously.
---

Implement the work described by the user in the spec or tickets.

Never reopen the plan. Whatever was settled upstream is the input, and your job is to turn it into a
commit. If a ticket is genuinely ambiguous, stop and ask — don't guess, and don't widen scope.

## Scope: full or staged

By default you own the whole flow below, steps 1–5 — the standalone behaviour, unchanged. An
orchestrating pipeline may instead hand you a narrower scope, keeping review, full-suite
validation, and shipping for its own later stages. Its stage instructions win over steps 4–5;
steps 1–3 — read the chain, find the repo's rules, build with `/tdd` — always apply, in every
scope.

A typical narrowed scope asks for just the code, its tests, and a quick sanity check: do steps
1–3 exactly as written, then typecheck and run only the test files you touched, and stop once the
commit the caller asked for (for example, a single WIP commit on the current branch) sits on the
branch with a clean tree. The pipeline's later stages own the full suite, the code review, the
shipping, and — unless the caller asks you to verify them — the success criteria.

## 1. Read the ticket, and everything above it

Read the ticket **and its full parent chain** — your work has to serve the parent's intent, not just
the leaf. Note its success/acceptance criteria and anything it declares out of scope.

## 2. Find this repo's rules before writing code

Three sources, all of which may or may not exist. Read what's there:

- **`docs/agents/issue-tracker.md`** — owns the branch, PR, and board mechanics. Follow it; don't
  invent your own. Run `/setup-matt-pocock-skills` if it's missing.
- **The repo's agent instructions** (`CLAUDE.md` / `AGENTS.md`) and any standards doc
  (`CONTRIBUTING.md`, `CODING_STANDARDS.md`) — these carry the **quality gates**: the test command,
  the coverage threshold, the typechecker, whether docs precede code. Whatever they state is a hard
  gate, not a suggestion.
- **A scaffolding skill for this kind of work** — many repos ship one per component type (a new
  module, endpoint, page, plugin). If one fits, reach for it; it owns the layout conventions and
  saves you re-deriving them.

If the repo documents no gates at all, default to: typecheck clean, full test suite green, new code
covered by tests, bug fixes covered by a regression test.

## 3. Build it

Use `/tdd` where possible, at pre-agreed seams. Typecheck regularly and run single test files
regularly — don't save all feedback for the end.

## 4. Verify, don't assert

Run the full suite and the gates from step 2 once at the end. Then verify **every** success criterion
by actually running the command or test it names. Reporting a criterion as met without executing it
is a failure, and a red suite or a missed gate means the work isn't done.

## 5. Review, then ship

Run `/code-review` against the branch point and act on its findings before committing.

Then ship it the way `docs/agents/issue-tracker.md` says — branch, PR referencing the issue, merge,
board state. Commit to the current branch if that file says nothing about branching.

Close out by telling the user what changed, the test and gate results, and anything that now needs a
human — a service to restart, a migration to run, a secret to seed.
