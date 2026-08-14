---
name: to-tickets
description: Break a plan, spec, or the current conversation into a set of tracer-bullet tickets, each declaring its blocking edges, published to the configured tracker — edges as text in one file per ticket locally, or native blocking links on a real tracker.
disable-model-invocation: true
---

# To Tickets

Break a plan, spec, or conversation into a set of **tickets** — tracer-bullet vertical slices, each declaring the tickets that **block** it.

The issue tracker and triage label vocabulary should have been provided to you — run `/setup-matt-pocock-skills` if not.

## Process

### 1. Gather context

Work from whatever is already in the conversation context. If the user passes a reference (a spec path, an issue number or URL) as an argument, fetch it and read its full body and comments.

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so to understand the current state of the code. Ticket titles and descriptions should use the project's domain glossary vocabulary, and respect ADRs in the area you're touching.

Look for opportunities to prefactor the code to make the implementation easier. "Make the change easy, then make the easy change."

### 3. Draft vertical slices

Break the work into **tracer bullet** tickets.

<vertical-slice-rules>

- Each slice cuts a narrow but COMPLETE path through every layer (schema, API, UI, tests) — vertical, NOT a horizontal slice of one layer
- A completed slice is demoable or verifiable on its own
- Each slice is sized to fit in a single fresh context window
- Any prefactoring should be done first

</vertical-slice-rules>

Give each ticket its **blocking edges** — the other tickets that must complete before it can start. A ticket with no blockers can start immediately.

**Wide refactors are the exception to vertical slicing.** A **wide refactor** is one mechanical change — rename a column, retype a shared symbol — whose **blast radius** fans across the whole codebase, so a single edit breaks thousands of call sites at once and no vertical slice can land green. Don't force it into a tracer bullet; sequence it as **expand–contract**. First expand: add the new form beside the old so nothing breaks. Then migrate the call sites over in batches sized by blast radius (per package, per directory), each batch its own ticket blocked by the expand, keeping CI green batch to batch because the old form still exists. Finally contract: delete the old form once no caller remains, in a ticket blocked by every migrate batch. When even the batches can't stay green alone, keep the sequence but let them share an integration branch that all block a final integrate-and-verify ticket — green is promised only there.

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each ticket, show:

- **Title**: short descriptive name
- **Blocked by**: which other tickets (if any) must complete first
- **What it delivers**: the end-to-end behaviour this ticket makes work

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the blocking edges correct — does each ticket only depend on tickets that genuinely gate it?
- Should any tickets be merged or split further?

Iterate until the user approves the breakdown.

### 5. Publish the tickets to the configured tracker

Publish the approved tickets the way `docs/agents/issue-tracker.md` says. The tickets are the same
either way — only the shape of the blocking edges changes:

- **A real issue tracker (GitHub, GitLab, Linear, …)** → one issue per ticket, in dependency order
  (blockers first) so each ticket's edges can reference real identifiers. Use the platform's native
  blocking / sub-issue relationship where it has one; otherwise set each ticket's "Blocked by" to the
  blocking issues. Apply the `ready-for-agent` triage label unless instructed otherwise — these
  tickets are agent-grabbable by construction.
- **Local files** → one file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`,
  numbered from `01` in dependency order, each file's "Blocked by" listing the numbers it depends on.
  One ticket per file, never a single combined file.

Where the tracker has a parent/sub-issue model, work happens on **leaf tickets only**: a ticket with
open children is a container, never a unit of work.

Work the **frontier**: any ticket whose blockers are all done. For a purely linear chain that means top to bottom.

Do NOT close or modify any parent issue.

Both forms carry the same sections. Use `<issue-template>` on a tracker; on local files, put the
title as an `# <NN> — <title>` heading and keep the rest identical.

<issue-template>

## Parent

A reference to the parent issue on the tracker (if the source was an existing issue, otherwise omit this section).

## What to build

The end-to-end behaviour this ticket makes work, from the user's perspective — not layer-by-layer implementation.

## Design

The decisions already settled upstream, stated as instructions: exact module paths, signatures,
models, and file locations. Write them as "implement exactly this". This section is mandatory.

## Analysis required

Anything NOT yet settled, called out explicitly at the top of this section. Omit the section entirely
when nothing is open — never leave an open question implicit or buried in prose.

## Success Criteria

- [ ] Criterion 1 — with the exact command or test that verifies it
- [ ] Criterion 2

## Out of scope

- What this ticket deliberately does not touch.

## Blocked by

- A reference to each blocking ticket, or "None — can start immediately".

</issue-template>

**The ticket body carries the design.** These tickets are implemented by separate agents that never
see the conversation that produced them, so decisions already made get LOCKED into the description —
exact models, signatures, and paths, "implement exactly this". This deliberately overrides the usual
advice to keep file paths out of tickets: a stale path costs one correction, whereas an agent
re-deciding a settled design costs the whole ticket.

If a prototype produced a snippet that encodes a decision more precisely than prose can (state
machine, reducer, schema, type shape), inline it and note that it came from a prototype. Trim to the
decision-rich parts — not a working demo.
