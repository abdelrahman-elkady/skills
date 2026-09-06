---
name: draft-plan
description: Draft a self-contained implementation plan in the project's plans directory as a handover document another agent can ship independently.
disable-model-invocation: true
user-invocable: true
argument-hint: "[slug]"
allowed-tools: Read Glob Bash(ls *) Bash(find *) Bash(mkdir *) Write
---

# /draft-plan - Create a handover plan

Convert a completed design discussion (typically a grilling or brainstorming session) into a self-contained markdown plan that another agent can pick up and ship without further context.

A handover plan is read once by an agent who wasn't in the room when the design was decided. Anything ambiguous becomes a question that bounces back to the user. The structure below is tuned for that handoff: capture *why*, *what*, *how*, and *done when*, drop everything else, and refuse to leave open questions implicit.

## Procedure

### 1. Detect cold start

If the conversation has no meaningful prior design context (no grilling, brainstorm, or substantive design discussion in the recent turns), surface this single line before proceeding:

> No prior design context detected. For best results, try [`/grill-me`](https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md) first. Continuing with gap-check…

Then keep going. Do not gate, refuse, or block - this is informational only.

### 2. Resolve target directory

Apply in order, stop at the first match:

1. If `ai-docs/plans/` exists at the repo root → use it silently.
2. Otherwise, search for any directory literally named `plans` at depth ≤ 2 from the repo root, e.g. via `find . -maxdepth 2 -type d -name plans -not -path '*/node_modules/*' -not -path '*/.git/*'`.
3. If one or more such directories are found, use `AskUserQuestion` to let the user pick one OR opt to create `ai-docs/plans/`.
4. If none are found → create `ai-docs/plans/` silently (`mkdir -p ai-docs/plans`).

The reason for the auto-detection: many projects already keep plans under `docs/plans/` or similar. Silently writing to `ai-docs/plans/` when another plans dir exists would scatter related work across two places.

### 3. Gap-check

Read the relevant conversation context and decide whether the four required sections can be filled with substance:

- **Context** - why this work exists, what triggered it
- **Goal** - what "done" looks like, in one paragraph
- **Approach** - ordered, concrete implementation steps
- **Acceptance Criteria** - verifiable checks that prove completion

If any of these is missing, vague, or in conflict, ask targeted questions using `AskUserQuestion`. Ask as many as you need - there is no cap. Stop when you can write each section with real substance, not filler. The cost of a few extra questions now is much smaller than the cost of a downstream agent bouncing back because the plan was thin.

### 4. Resolve the filename slug

- If the user passed a slug as an argument (`$ARGUMENTS`), use it as the basis.
- Otherwise, derive a 3–6 word slug from the Goal sentence.

Apply these slugify rules:

- Lowercase only.
- ASCII alphanumerics and hyphens only - strip everything else.
- Collapse runs of hyphens to a single hyphen.
- Trim leading and trailing hyphens.
- Truncate at 60 characters maximum, on a word boundary (do not cut a word in half).
- If the slug ends up empty after slugification, ask the user for one.

### 5. Resolve the next number

Scan `<target-dir>/*.md`. For each file, parse the leading run of digits (of any width, so `001-foo.md` and `0042-bar.md` both parse to 1 and 42 respectively). Take `max + 1`. Format as a 4-digit zero-padded number. An empty directory yields `0001`.

### 6. Write the file

Path: `<target-dir>/<NNNN>-<slug>.md`

Use this template. Conditional sections (`Out of scope`, `Open questions`) must be omitted entirely — heading and all — when they have no content. Do not emit empty placeholders.

```markdown
---
status: proposed
---

# <H1 title - real prose derived from the Goal, not a humanized slug>

## Context

<Why this exists, what triggered it. Inline links to Jira tickets, prior plans, design docs, related PRs go here.>

## Goal

<One paragraph: exactly what "done" looks like.>

## Approach

1. <Ordered step. Reference any file paths inline in backticks, e.g. `src/auth/validator.ts`.>
2. <Step>
3. <Step>

## Acceptance Criteria

- [ ] <Verifiable claim with the check inline, e.g. "Endpoint returns 200 on /health (`curl localhost:3000/health`)">
- [ ] <…>

## Out of scope

- <non-goal>

## Open questions

- <unresolved item>
```

Rules for the body:

- Frontmatter contains **only** `status: proposed`. No `created` date, no other fields.
- There is **no** separate `Files` section. Approach steps reference file paths inline in backticks. One source of truth means Approach and Files cannot drift apart.
- Acceptance Criteria merges "definition of done" with "how to verify": each checkbox is the verifiable outcome with the check (a command, a test name, or a manual step) included inline.
- The H1 title is real prose, distinct from the slug. Slug `implement-auth` could pair with title `Implement OAuth login for admin users`.

### 7. Report back

After writing, respond with exactly this single line:

```
Created <path> - Goal: <one-line goal>. Steps: N. Acceptance criteria: M.
```

Where `N` is the number of items in the Approach list and `M` is the number of Acceptance Criteria checkboxes. No additional commentary, no rendered preview - the file on disk is the artifact.

## Status transitions

Files are always created with `status: proposed`. The user flips to `status: implemented` by hand when the plan ships. This skill is creation-only and does not manage status changes - adding a flip subcommand would muddy the single purpose.
