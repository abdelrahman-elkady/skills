---
name: review-pr
description: PR review built on /code-review that verifies every finding, stops at a table for you to filter, then drafts and posts what you keep. With `unattended`, drafts everything first and pauses once before posting.
disable-model-invocation: true
user-invocable: true
argument-hint: "<PR|branch> [effort] [unattended]"
---

# /review-pr - run it, come back to a table

The stages run `context → hunt → verify → triage → draft → post`, all in one session. First check that a `pr-comments` skill is in your skill list; without it, say so and stop, since `draft` needs it. Then read the profile, if there is one, and run straight through `verify`: the user may be away until then, so make any call you'd ask them about and log it under **Calls to check**. A branch with no PR ends at `draft`. `<target>` is the PR number, else the branch name.

From `triage` on, the run takes one of two paths:

- **Driven**, the default: `triage` stops with its report and the user filters the rows with you. `draft` runs when they name the rows to draft, and stops for them to read or edit the text.
- **Unattended**, when the arguments include the word `unattended`: the run goes straight through `draft`, whose report is the one pause.

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

## triage

Mark each confirmed row a person's ruling decides, citing the ruling; the PR's author on record counts as a person. Check this PR's threads and ticket, and search the memory files by file and symbol, not just the index, and count only what a person ruled: memory also holds past agents' verdicts. A ruled row gets no draft unless the user overrules it.

Number the table's rows from 1. Each keeps its number for the whole session, dropped or not, so the user can say "drop 3, draft 1 and 5".

**Stop with** the report, unless the run is unattended:

1. **Headline:** whether it should merge and the main reason in bold, then the counts and the head SHA, e.g. "**#12 shouldn't merge yet: a retried webhook refunds twice.** 1 P1, 3 P3s, 1 question. Nothing is posted, and head is `abc1234`."
2. **Table:** one row for each confirmed finding and `[Q]`, ruled ones included, each optional reply and each row past the cap. Order them P0-P2, then `[Q]`s, then P3s, with ruled and `Not verified` rows last in each group. `Initial` is the `hunt` rating; `Final` is the verifier's, in bold when it holds the merge. `Status` is a few words: how it was confirmed, why it moved, the ruling it cites, any doubt about it, or `Not verified`.

   | # | Issue | Initial | Final | Status |
   |---|---|---|---|---|
   | 1 | A retried webhook refunds the order twice | P2 | **P1** | Reproduced locally |
   | 2 | Should a cancelled order count toward the limit? | `[Q]` | `[Q]` | Confirmed, the author's call |
   | 3 | Export holds the whole file in memory | P2 | P3 | Largest export is 2 MB |
   | 4 | Stock `reconcile` re-reads the feed on retry | P3 | P3 | Ruled fine on the ticket |
   | 5 | Comment still names the old queue | P3 | P3 | Not verified |

3. **Reasoning:** each unruled P0-P2 and `[Q]`, highest first, under a `#1: <issue>` line with its anchor link, in short labeled bullets: **Cause**, **Effect**, **Evidence** (naming the MCP or the run), **Fix**, plus **Why this priority** when a verifier moved it or a stated condition would move it. When an earlier review already raised it, name its thread and what ours focuses on.
4. **P3s and optional replies:** each unruled P3 and optional reply, one line with its anchor or thread: why it's a P3, or what the reply adds. Say they're skipped unless picked.
5. **Left out:** one short line per dropped row and why: refuted or already raised. Confirmed rows stay in the table for the user to filter.
6. **Out of scope:** one short line per side row worth a look.
7. **Calls to check:** the brief; the consumers; anything else decided in the user's place.
8. **The ask:** which rows to draft, naming your pick (by default the unruled P0-P2 and `[Q]`s); each decision only the user can make, with your recommendation; and, separately, whether to save what the run found, when there's no profile, a new consumer, or an MCP a verifier used that `evidence` doesn't list.

On the reply, work through the rows with the user: check what they question, drop, add or re-rate what they say, and show the table again if the discussion goes on after a change. A `Not verified` row they pick gets its agent first. A `[Q]` answered as intended is dropped. A P2 they mark can-wait gets the can-wait closer from `/pr-comments`. A reply that names the rows to draft goes straight to `draft`.

**Done when** the user has named the rows to draft, or the run is unattended.

## draft

Load `/pr-comments` via the Skill tool and draft by it into `drafts-<target>.md` in your session's scratchpad, or a temp folder if it has none. Keep any other file you need there too. The file's first line is the head SHA the verifiers ran at, for `post` to compare. Each draft sits under a heading with its row number and anchor.

- **Driven:** draft the rows the user named.
- **Unattended:** draft every confirmed row that isn't ruled, with P3s under a `P3` heading and each optional reply under an `Optional` heading.

`triage` already applied the rulings and the user's picks stand, so these rows are the whole set, whatever `/pr-comments` finds in the project's memory.

When nothing would post without a further pick, because no row was named or every row sits under those headings, also draft a body-only `COMMENT` review: "No blocking issues found." if a P3 is left, else "Review done, nothing new worth flagging."

**Stop with** the file's absolute path and the review verdict, for the user to read and edit, plus:

- **Driven:** each draft's anchor and exact text, any body-only review quoted, each confirmed P0-P2 left undrafted, each open decision a draft assumes, and the ask: whether to post this set, if there's a PR.
- **Unattended:** the `triage` report, with each row's anchor and exact text in place of its reasoning and any body-only review quoted. Its ask replaces which rows to draft with whether to post this set, if there's a PR.

On the reply, `triage`'s reply rules still hold: make the changes in the file and show what changed. Only a yes to posting the set runs `post`.

## post

- If the drafts file is gone, say so, rebuild what the session still holds and show it for a fresh yes.
- If the head SHA moved, rerun `verify` at the new head on the drafts the new commits touch, then update the file, keeping the user's other edits, with the new SHA on its first line. If the set changed, show it for a fresh yes.
- Post the file as it stands, minus its SHA line and headings, each draft under a `P3` or `Optional` heading the user didn't pick and, if they picked any, the body-only review, per `/pr-comments`, which sets the verdict from what's left. If that verdict differs from the one `draft` gave, show it for a fresh yes first. What `draft` showed is its show step, so one yes to the set approves all of it.

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
