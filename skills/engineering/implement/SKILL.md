---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

The issue tracker workflow should have been provided to you in `docs/agents/issue-tracker.md` — read it
first. It owns the branch, PR, and board mechanics for this repo; do not invent your own.

Never reopen the plan. Whatever was settled upstream is the input. If a ticket is genuinely
ambiguous, stop and ask — do not guess and do not widen scope.

## Process

1. **Read the ticket and its full parent chain.** Your work must serve the parent's vision, not just
   the leaf. Note the ticket's Success Criteria and its Out of scope section.
2. **Reach for the repo's scaffolding skill if one fits** — `new-loom-module`, `new-rest-endpoint`,
   `new-web-feature`, `new-persona`, `new-mcp-connector`. They own the layout conventions; don't
   re-derive them.
3. **Build it.** Use `/tdd` where possible, at pre-agreed seams. Docs first, code second — a new or
   changed module needs its design doc (`module-docs`) before implementation.
4. **Run the gates as you go**, not just at the end:
   - typecheck regularly (`pyright`), single test files regularly
   - the full suite once at the end: `uv run pytest --cov --cov-report=term-missing`
   - **≥80% line coverage is a hard gate.** New code needs tests; bug fixes need a regression test.
     A red suite or a coverage miss means the work is not done — fix it or report failure.
5. **Verify every Success Criterion** by actually running the stated command or test. Reporting a
   criterion as met without running it is a failure.
6. **Review before committing.** Use `/code-review` against the branch point. Act on its findings.
7. **Ship it** per `docs/agents/issue-tracker.md`: branch from `main`, PR whose body ends with
   `Closes #<n>`, merge, board Status → Done. No `Co-authored-by` trailers. Push every touched repo.

If the change is a runtime change to a service, say so at the end — uat needs bouncing.
