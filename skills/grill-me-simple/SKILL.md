---
name: grill-me-simple
description: simplified grill-me
disable-model-invocation: true
---

# /grill-me-simple - Relentless interview

Interview the user relentlessly until you reach a shared understanding. Map this as a design tree: every decision branches into the decisions that hang off it.

Work the tree in rounds. The frontier is every decision whose prerequisites are already settled - the questions you can ask now without guessing at answers you haven't heard yet. Ask the frontier in rounds of at most 4 questions. Default to `AskUserQuestion`: short candidate answers - a label plus a line of rationale - fit its options, and "Other" catches the rest. When an answer is a scenario or needs code or examples to judge, ask in plain text instead - numbered, with your recommendation. Wait for the answers before the next round.

Each round the user answers reshapes the tree - settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a later round, not this one.

Finding facts is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it - don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report - ask the rest of the frontier now. Decisions that shape the design are the user's - put each to them and wait. Trivial leaves (naming, formatting, defaults with one sane answer) don't earn a question: state your assumption and move on; the user can veto.

The session is done when the frontier is empty: every branch decided by the user or covered by a stated assumption - nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.
