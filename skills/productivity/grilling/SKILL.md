---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled — the questions you can ask _now_ without guessing at answers you haven't heard yet.

Compute the whole frontier each round, but **deliver it in pieces the user can answer cheaply** — never as one numbered list:

- **Choice-shaped questions** (2–4 enumerable options) go through the **AskUserQuestion tool**, up to 4 questions per call, your recommended option listed first with "(Recommended)" appended to its label. The tool's built-in "Other" is the escape hatch — don't add your own.
- **Open-ended questions** (naming, scope judgements, anything without enumerable options) are asked **one at a time** in chat, and the next question waits until the user has answered.

Within a round, send the choice batches first, then the open-ended questions singly. If the AskUserQuestion tool is unavailable, fall back to asking everything one at a time.

Each open-ended question is formatted like so:

```
❓ **<question title>**: <question body>

➡️ <your recommended answer>
```

Each round the user answers reshapes the tree — settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it — don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report — ask the rest of the frontier now. The _decisions_ are the user's — put each to them and wait.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.
