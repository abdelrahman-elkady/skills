---
name: orchestrate
description: Enter orchestration mode
disable-model-invocation: true
user-invocable: true
---

# /orchestrate - Coordinate, don't execute

**Orchestration mode is on.** You're the **orchestrator**: decompose, dispatch, synthesize - don't do the work yourself. Reading and editing files burns the context you need for the calls only you can make; if you're reading file after file "just to understand," stop - that's a subagent's job. The usual failure is under-delegating, not over-delegating. (This skill only flips the mode; the user sets the goal.)

Everything below is guidance, not mandate - the judgment calls are yours.

## Keep vs. delegate

**Keep:** decompose the goal · pick the primitive and model · sequence and track · turn results into decisions · talk to the user.

**Delegate almost everything else:** discovery ("read X, report Y") · implementation, refactors, bulk transforms · verification and test runs, adversarial where it matters.

## Pick the primitive

- **A few independent tasks, or a scouting pass** → parallel subagents (one message, multiple `Agent` calls).
- **Structured, multi-stage, or big fan-out** → an ultracode workflow - only when ultracode is on; otherwise subagents, or offer to run a workflow.
- **Usually hybrid:** scout to map the work, then fan out - don't design the orchestration before you know its shape.
- **Too small (≤~3 trivial items)?** Do them inline; past that, batch them out.
- **Match agents to their powers:** read-only (e.g. `Explore`) for pure discovery; write access only where the work edits.
- **Partition parallel edits by file** - nothing to merge. Worktree isolation only when edits or builds/tests genuinely collide (`node_modules`, lockfiles, generated output).

## Briefs and handoffs

Scope what each agent gets and returns to what's needed downstream, not the maximum - a yes/no often beats a full report. The more important or shareable a result, the more it belongs in a file the next agent is pointed at: wire one agent's output location into the next one's prompt so handoffs aren't re-derived.

## Model tiering

Weigh two knobs — **judgment needed** and **cost of being wrong** — and pick the cheapest model that clears both. Three tiers, defined by those knobs, independent of harness:

- **Mechanical** - pattern-following work where mistakes are cheap and easily caught.
- **Workhorse** - the default: anything needing real understanding, or mechanical work whose errors aren't trivial. High fan-out with nontrivial error cost lands here; verifying and repairing many units costs more than dropping a tier saves.
- **Frontier** - high-stakes work: hard reasoning, planning, synthesis, verification, challenging ideas (adversarial critique).

Map the tier to your harness's model:

| Tier           | Claude Code | Codex                                             |
| -------------- | ----------- | ------------------------------------------------- |
| **Mechanical** | Haiku       | `gpt-5.6-luna` (medium effort)                    |
| **Workhorse**  | Sonnet      | `gpt-5.6-terra` (medium effort)                   |
| **Frontier**   | Opus        | `gpt-5.6-sol` (high/xhigh; `max` for the hardest) |

**Don't drop a tier for cost alone if fixing the fallout costs more than you saved**

**On Codex:** reasoning effort is the main dial within a tier - start at `medium` and climb only as the work demands it (`medium → high → xhigh → max → ultra`). Luna is the recommended subagent model for high-volume fan-out. `ultra` spawns its own subagents and burns allowance fast, so it's reserved for very rare, genuinely hard frontier work - **always ask the user for permission before using it.**

## Operating rules

- **Artifacts over chat:** findings, plans, and reports go to files - titles and states, not prose. Use `ai-docs/` if it exists, else wherever the project keeps AI docs, else create one; checkpoints in a `_progress/` subdirectory.
- **Big or slow output → file, then extract:** the running agent pipes to a file, greps what matters, returns a verdict; small output, just read stdout.
- **Verify important findings** with a second, independent agent.
- **Hand off cleanly:** at a boundary, write the remaining work as a plan another agent can pick up (e.g. `/draft-plan`).

Report the synthesis and decisions - not a transcript.
