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

Run `/code-review` via the Skill tool with args `<effort> <target>`, then the brief and the consumers to check, every path absolute. Leave out `--comment` and `--fix`; at `ultra`, add `--no-post`. It runs in the background: once its result arrives, record each finding, and each bot or AI comment it missed, as a row with a priority, or as a `[Q]`. Then sort each row, the context's `[Q]`s included:

- **Dropped** when a human review, or your own earlier one (the user's login, with the `/pr-comments` signature), already raised it, its thread is open and new commits leave that code alone. If ours adds something, note the addition.
- **Side** when the diff neither introduces it nor makes it reachable.
- **Sent to `verify`** otherwise. When a human or your own earlier review raised it, its verifier's brief names the thread, so it tests the fix.

**Done when** `/code-review` has returned and every finding has a row, sorted.

## verify

Brief one fresh agent for each row sent here and each noted addition, at the head SHA from `context`, up to `verify_cap`: P0/P1 first, then `[Q]`s, then P2/P3, then additions. Use Opus, or Sonnet for a much simpler row: a small claim, a plain check. Each returns CONFIRMED, CONFIRMED-WITH-CORRECTION or REFUTED.

| Row | It tries to falsify | Confirmed | Refuted |
|---|---|---|---|
| Finding | each clause, then re-rates the priority, up or down | confirmed at the new priority, with any correction | dropped |
| `[Q]` | each of the four gates | confirmed | dropped, naming the gate, unless it confirms a plain defect: then confirmed at a priority |
| Dropped, with a noted addition | only the addition | an optional reply, with any correction | stays dropped |

A `[Q]` survives only when all four gates hold, so most PRs keep none:

- **Reachable:** a concrete input on a path the diff changes triggers it.
- **Unanswered:** the ticket, PR description, linked specs and threads were read and don't settle it.
- **Consequential:** the plausible answers differ in outcome for users, money, data, or access.
- **Open:** the code is right under one plausible answer and wrong under another.

Each brief points its agent to the session's MCPs, `evidence` first, for when the code can't settle the verdict or the priority: observability for traffic and errors on the path, the database for rows that would trigger it, the issue tracker for past incidents and tickets on the area. It carries these rules:

- **Walk every path:** "impossible before this PR" needs all of them, not only the create path.
- **Read only:** create, edit and send nothing.
- **Production shows the past:** the conditions the change will meet, not the change itself, so a quiet log doesn't prove a path safe.
- **Summaries only:** raw rows and log lines stay out of its reply, so they never reach the comments.
- **Cite the evidence**, naming the MCP, for each verdict or priority change it decides. A finding it shows doesn't matter is REFUTED.

**Done when** each row above has its verdict or is past the cap.

## draft

Drop each confirmed row a person's ruling decides, citing the ruling. Check this PR's threads and search the memory files by file and symbol, not just the index, and count only what a person ruled: memory also holds past agents' verdicts.

Load `/pr-comments` via the Skill tool and draft by it each remaining confirmed row, plus each optional reply under an `Optional` heading, into `drafts-<target>.md` in your session's scratchpad, or a temp folder if it has none. Keep any other file you need there too. The file's first line is the head SHA the verifiers ran at, for `post` to compare. With no confirmed row left, also draft a body-only `COMMENT` review: "Review done, nothing new worth flagging."

**Stop with** the report, giving that file's absolute path, for the user to review and edit it:

1. **Verdict:** one line of counts and the review verdict, e.g. "`REQUEST_CHANGES`: 3 findings (P1, P2, P3), 2 questions, 2 optional replies, 2 out of scope". It quotes any body-only review.
2. **Findings**, highest priority first, each with its anchor and exact text. When an earlier review already raised one, name its thread and what ours focuses on.
3. **Questions:** each `[Q]` with its anchor and exact text.
4. **Optional replies:** thread link, what each adds, and its exact text. Say they're skipped unless picked.
5. **Left out:** one short line per dropped row and why: refuted, already raised, shown not to matter, or a person's ruling. A ruled row cites the ruling and gives its confirmed priority, so the user can overrule it.
6. **Not verified:** one short line per row past the cap, for the user to pick.
7. **Out of scope:** one short line per side row worth a look.
8. **Calls to check:** the brief; the consumers; anything else decided in the user's place. With no profile, a new consumer, or an MCP a verifier used that `evidence` doesn't list, offer to save what the run found.
9. **The ask:** whether to post this set (if there's a PR) and, separately, whether to save. Only a yes to posting runs `post`.

On the reply, a row picked from **Not verified** gets its agent and, once confirmed, its draft, then the report again. A `[Q]` answered as intended leaves the drafts and is dropped.

## post

- If the drafts file is gone, say so, rebuild what the session still holds and show it for a fresh yes.
- If the head SHA moved, rerun `verify` at the new head on the drafts the new commits touch, then update the file, keeping the user's other edits, with the new SHA on its first line. If the set changed, show it for a fresh yes.
- Post the file as it stands, minus its SHA line, each `Optional` reply the user didn't pick and, if they picked one, the body-only review, per `/pr-comments`, which sets the verdict from what's left. The report is its show step, so one yes approves the whole set.

**Stop with** the comment links.

## Profile

`.claude/review-pr.local.md` in the main checkout, the parent of `git rev-parse --path-format=absolute --git-common-dir`, whichever worktree you run from. Its YAML frontmatter holds:

- `consumers` - repos that use this repo's contracts, with local paths
- `evidence` - MCPs holding this repo's real-world facts, with what each holds (e.g. `signoz: prod traces`)
- `verify_cap` - most verifier agents per run (default 15)

Write one only when the user accepts the report's offer, and add it to that checkout's `.gitignore` unless it's already ignored, leaving that edit uncommitted for the user to commit to the default branch.
