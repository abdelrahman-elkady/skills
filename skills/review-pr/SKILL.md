---
name: review-pr
description: Unattended PR review built on /code-review that drafts every comment, then pauses once for approval before posting.
disable-model-invocation: true
user-invocable: true
argument-hint: "<PR|branch> [effort]"
---

# /review-pr - run it, come back to one report

The stages run `context → hunt → verify → draft → post`, all in one session. First check that a `pr-comments` skill is in your skill list; without it, say so and stop, since `draft` needs it. Then read the profile, if there is one, and run straight through `draft`, whose report is the one pause: the user may be away until then, so make any call you'd ask them about and log it there. `post` runs on their yes. A branch with no PR ends at `draft`. `<target>` is the PR number, else the branch name.

## context

- Read the PR, its linked ticket, the specs and TDDs either links, and every existing review and comment. Bot and AI comments are leads, not decisions.
- Write a 3-line brief: what the PR must do, what's risky, what's out of scope.
- Record each place the diff departs from the ticket or a spec as a `[Q]` row: a product call for the author, which the code can't make.
- When the diff changes a contract (routes, response shapes, enums, events, schema, exports), find its consumers: the profile first, else same-org sibling repos or repos linked from the PR or ticket.
- The effort is the user's argument as given, else `xhigh`.

**Done when** you have the head SHA, the brief, the effort, the consumers and the `[Q]` rows.

## hunt

Run `/code-review` via the Skill tool with args `<effort> <target>`, then the brief and the consumers to check, every path absolute. Leave out `--comment` and `--fix`; at `ultra`, add `--no-post`. It runs in the background: once its result arrives, record each finding, and each bot or AI comment it missed, as a row rated P0-P3 by `## Priorities`, or as a `[Q]`. Then sort each row, the context's `[Q]`s included:

- **Dropped** when a human review, or your own earlier one (the user's login, with the `/pr-comments` signature), already raised it, its thread is open and new commits leave that code alone. If ours adds something, note the addition.
- **Side** when the diff neither introduces it nor makes it reachable.
- **Sent to `verify`** otherwise. When a human or your own earlier review raised it, its verifier's brief names the thread, so it tests the fix.

**Done when** `/code-review` has returned and every finding has a row, sorted.

## verify

Brief one fresh agent for each row sent here and each noted addition, at the head SHA from `context`, up to `verify_cap`: P0-P2 first, then `[Q]`s, then P3s, then additions. Use Opus, or Sonnet for a much simpler row: a small claim, a plain check. Each returns CONFIRMED, CONFIRMED-WITH-CORRECTION or REFUTED.

| Row | It tries to falsify | Confirmed | Refuted |
|---|---|---|---|
| Finding | each clause, then rates it by `## Priorities` | confirmed at that priority, with any correction | dropped |
| `[Q]` | each of the four gates | confirmed | dropped, naming the gate, unless it confirms a plain defect: then confirmed at a priority |
| Dropped, with a noted addition | only the addition | an optional reply, with any correction | stays dropped |

A `[Q]` survives only when all four gates hold, so most PRs keep none:

- **Reachable:** a concrete input on a path the diff changes triggers it.
- **Unanswered:** the ticket, PR description, linked specs and threads were read and don't settle it.
- **Consequential:** the plausible answers differ in outcome for users, money, data, or access.
- **Open:** the code is right under one plausible answer and wrong under another.

Each brief carries `## Priorities` and a one-line summary of every other row, and leaves out this row's level, so its agent rates it fresh. It points the agent to the session's MCPs, `evidence` first, for when the code can't settle the verdict or the priority: observability for traffic and errors on the path, the database for rows that would trigger it, the issue tracker for past incidents and tickets on the area. It carries these rules:

- **Walk every path:** "impossible before this PR" needs all of them, not only the create path.
- **Read only:** create, edit and send nothing.
- **Production shows the past:** the conditions the change will meet, not the change itself, so a quiet log doesn't prove a path safe.
- **Summaries only:** raw rows and log lines stay out of its reply, so they never reach the comments.
- **Cite the evidence**, naming the MCP, for each verdict or priority it decides.

**Done when** each row above has its verdict or is past the cap.

## draft

Drop each confirmed row a person's ruling decides, citing the ruling; the PR's author on record counts as a person. Check this PR's threads and ticket, and search the memory files by file and symbol, not just the index, and count only what a person ruled: memory also holds past agents' verdicts.

Load `/pr-comments` via the Skill tool and draft by it each remaining confirmed row, with P3s under a `P3` heading and each optional reply under an `Optional` heading, into `drafts-<target>.md` in your session's scratchpad, or a temp folder if it has none. Keep any other file you need there too. The file's first line is the head SHA the verifiers ran at, for `post` to compare. With no row left outside those headings, also draft a body-only `COMMENT` review: "No blocking issues found." if a P3 is left, else "Review done, nothing new worth flagging."

**Stop with** the report, giving that file's absolute path, for the user to review and edit it:

1. **Verdict:** one line of counts and the review verdict, e.g. "`REQUEST_CHANGES`: 2 findings (P1, P2), 1 P3, 2 questions, 2 optional replies, 2 out of scope". It quotes any body-only review.
2. **Findings**, P0-P2, highest priority first, each with its anchor and exact text. When an earlier review already raised one, name its thread and what ours focuses on. When a verifier moved its priority, give both levels and why: `P1 → P2: <evidence>`.
3. **P3s:** the same, for each P3. Say they're skipped unless picked.
4. **Questions:** each `[Q]` with its anchor and exact text.
5. **Optional replies:** thread link, what each adds, and its exact text. Say they're skipped unless picked.
6. **Left out:** one short line per dropped row and why: refuted, already raised, shown not to matter, or a person's ruling. A ruled row cites the ruling and gives its confirmed priority, so the user can overrule it.
7. **Not verified:** one short line per row past the cap, with its priority, highest first, for the user to pick.
8. **Out of scope:** one short line per side row worth a look.
9. **Calls to check:** the brief; the consumers; anything else decided in the user's place. With no profile, a new consumer, or an MCP a verifier used that `evidence` doesn't list, offer to save what the run found.
10. **The ask:** whether to post this set (if there's a PR) and, separately, whether to save. Only a yes to posting runs `post`.

On the reply, a row picked from **Not verified** gets its agent and, once confirmed, its draft, then the report again. A `[Q]` answered as intended leaves the drafts and is dropped. A P2 the user marks can-wait gets the can-wait closer from `/pr-comments`.

## post

- If the drafts file is gone, say so, rebuild what the session still holds and show it for a fresh yes.
- If the head SHA moved, rerun `verify` at the new head on the drafts the new commits touch, then update the file, keeping the user's other edits, with the new SHA on its first line. If the set changed, show it for a fresh yes.
- Post the file as it stands, minus its SHA line, each P3 and each `Optional` reply the user didn't pick and, if they picked any, the body-only review, per `/pr-comments`, which sets the verdict from what's left. If that verdict differs from the report's, show it for a fresh yes first. The report is its show step, so one yes approves the whole set.

**Stop with** the comment links.

## Priorities

Rate the harm if the diff merged as is and the target branch deployed, with the PR's other P0-P2 findings fixed. P0-P2 hold the merge. If you can't tell whether production reaches the code, assume it does. Rate code behind an off flag as if the flag were on, except that a P0 or P1 runtime defect becomes P2. Take the first line that fits.

- **Refuted:** code, a spec or a convention shows the claim false or the behaviour intended; a production count of the conditions it needs shows nothing would trigger it, and no fix would simplify the code; or it's taste, with no convention or named harm behind it.
- **P0:** on deploy, working software breaks for everyone exposed, or production data is destroyed at scale, and only gating or a new release contains it.
- **P1:** a feature always fails, a flow strands users until someone else steps in, or someone unauthorized reaches private data or a protected action.
- **P2:** a path in routine use, internal use included, misbehaves or locks out unrelated writes; a failure or race stores wrong state or loses data; money or anything sent to outside parties comes out wrong, even rarely; or a deploy, load spike or crafted input can take the service down, even briefly.
- **Working code with a design cost:** P1 when a stored name needs a migration or dual reads to correct, or an open PR already builds on misplaced code; P2 when it adds a stored copy, a new module in the wrong layer, or machinery other code must follow for little gain; otherwise P3.
- **P3:** anything else real: a race that leaves nothing wrong, a step that touches few records ever, test, dev or staging-only code, tidying, hardening with no path to harm today.

## Profile

`.claude/review-pr.local.md` in the main checkout, the parent of `git rev-parse --path-format=absolute --git-common-dir`, whichever worktree you run from. Its YAML frontmatter holds:

- `consumers` - repos that use this repo's contracts, with local paths
- `evidence` - MCPs holding this repo's real-world facts, with what each holds (e.g. `signoz: prod traces`)
- `verify_cap` - most verifier agents per run (default 15)

Write one only when the user accepts the report's offer, and add it to that checkout's `.gitignore` unless it's already ignored, leaving that edit uncommitted for the user to commit to the default branch.
