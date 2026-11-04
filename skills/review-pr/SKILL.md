---
name: review-pr
description: Unattended PR review built on /code-review that drafts every comment, then pauses once for approval before posting, with findings tracked in a per-PR file.
disable-model-invocation: true
user-invocable: true
argument-hint: "<PR|branch> [effort]"
---

# /review-pr - run it, come back to one report

The stages run `context → hunt → verify → draft → post → settle`. First check that a `pr-comments` skill is in your skill list; without it, say so and stop, since `draft` needs it. Then read the profile and start at `context`, or after the tracker's `stage`, and run straight through `draft`, whose report is the one pause: the user may be away until then, so make any call you'd ask them about and log it there. `post` runs on their yes, `settle` on a later run. A branch with no PR ends at `draft`.

## context

- No profile: write one from the repo and its siblings, seed `settled` from the review rulings in the project's memory, and save both. Add the profile and `review_dir` to `.gitignore` unless they're already ignored, leave that edit uncommitted, and run on the new profile.
- Read the PR, its linked ticket, the specs and TDDs either links, and every existing review and comment. Bot and AI comments are leads, not decisions.
- Write a 3-line brief: what the PR must do, what's risky, what's out of scope.
- Record each place the diff departs from the ticket or a spec as a `new` `[Q]` row: a product call for the author, which the code can't make.
- When the diff changes a contract (routes, response shapes, enums, events, schema, exports), find its consumers: the profile first, else same-org sibling repos or repos linked from the PR or ticket. Take the best match.
- The effort is the user's argument as given, else `xhigh` if the diff touches a `risk` area, else `high`.

**Done when** the tracker holds the head SHA, the brief, the effort, the consumers and the `[Q]` rows.

## hunt

Run `/code-review` via the Skill tool with args `<effort> <target>`, then the brief, the consumers to check, and the work dir `<review_dir>/<target>/cr/`, every path absolute. Leave out `--comment` and `--fix`; at `ultra`, add `--no-post`. It runs in the background: once its result arrives, record each finding, and each bot or AI comment it missed, as a row with a priority, or as a `[Q]`. Then give each row, the context's `[Q]`s included, its status:

- `new` by default.
- `dropped` when `settled` already decides it, citing the ruling.
- `dropped` when a human review already raised it. If ours adds something, note the addition.
- `side` when the diff neither introduces it nor makes it reachable.

**Done when** `/code-review` has returned and every finding has a row and a status.

## verify

Brief one fresh agent per row below, at the recorded head SHA, up to `verify_cap`: P0/P1 first, then `[Q]`s, then P2/P3, then additions. Rows past the cap keep their status. Use Opus, or Sonnet for a much simpler row: a small claim, a plain check. Each returns CONFIRMED, CONFIRMED-WITH-CORRECTION or REFUTED.

| Row | It tries to falsify | Confirmed | Refuted |
|---|---|---|---|
| `new` finding | each clause, then re-rates the priority, up or down | `validated` at the new priority, with any correction | `wrong` |
| `new` `[Q]` | each of the four gates | `validated` | `wrong`, naming the gate, unless it confirms a plain defect: then `validated` at a priority |
| `dropped` with a noted addition | only the addition | an optional reply, with any correction | stays `dropped` |

A `[Q]` survives only when all four gates hold, so most PRs keep none:

- **Reachable:** a concrete input on a path the diff changes triggers it.
- **Unanswered:** the ticket, PR description, linked specs, threads and `settled` were read and don't settle it.
- **Consequential:** the plausible answers differ in outcome for users, money, data, or access.
- **Open:** the code is right under one plausible answer and wrong under another.

Each brief points its agent to the session's MCPs, `evidence` first, for when the code can't settle the verdict or the priority: observability for traffic and errors on the path, the database for rows that would trigger it, the issue tracker for past incidents and tickets on the area. It carries these rules:

- **Walk every path:** "impossible before this PR" needs all of them, not only the create path.
- **Read only:** create, edit and send nothing.
- **Production shows the past:** the conditions the change will meet, not the change itself, so a quiet log doesn't prove a path safe.
- **Summaries only:** raw rows and log lines stay out of the tracker and the comments.
- **Cite the evidence**, naming the MCP, for each verdict or priority change it decides. A finding it shows doesn't matter is `dropped`.

**Done when** each row above has its verdict or is past the cap.

## draft

Load `/pr-comments` via the Skill tool, naming the `settled` file as the project's rulings, and draft by it into `<review_dir>/<target>/drafts.md` each `validated` row, plus each optional reply under an `Optional` heading. With no `validated` row, also draft a body-only `COMMENT` review, posted unless the user picks a reply: "Review done, nothing found worth flagging."

**Stop with** the report, for the user to review and edit `drafts.md`:

1. **Verdict:** one line of counts and the review verdict, e.g. "`REQUEST_CHANGES`: 3 findings (P1, P2, P3), 2 questions, 2 optional replies, 2 out of scope". It quotes any body-only review.
2. **Findings**, highest priority first, each with its anchor and exact text. When a bot or AI review already raised one, name its thread and what ours focuses on.
3. **Questions:** each `[Q]` with its anchor and exact text.
4. **Optional replies:** thread link, what each adds, and its exact text. Say they're skipped unless picked.
5. **Left out:** one short line per `wrong` or `dropped` row and why.
6. **Not verified:** one short line per row past the cap, for the user to pick.
7. **Out of scope:** one short line per `side` finding worth a look.
8. **Calls to check:** the brief; the effort; the consumers, offering to save a new match; a new profile and `settled`, shown in full; a new `.gitignore` edit, for the user to commit to the default branch; anything else decided in the user's place.
9. Any MCP a verifier used that `evidence` doesn't list, with an offer to add it.

On the reply, a row picked from **Not verified** gets its agent and, once confirmed, its draft, then the report again. A `[Q]` answered as intended goes to `settled`, leaves `drafts.md` and is `waived`.

## post

- If the head SHA moved, rerun `verify` on the findings the new commits touch, update `drafts.md` keeping the user's other edits, and record the new SHA. If a comment changed, show it for a fresh yes.
- Post `drafts.md` as it stands, minus each `Optional` reply the user didn't pick, per `/pr-comments`, loading it if this session hasn't. The report is its show step, so one yes approves the whole set; a run resuming here shows the report first.
- Mark each posted row `posted`, with the comment link, each `validated` row the user cut `dropped`, and each row still `new` `dropped` (not picked).

**Stop with** the comment links.

## settle

Run once the PR is merged or closed, or the author has replied; before that, say so and stop. Update each row the author answered: declined, deferred, or a `[Q]` answered as intended → `waived`; shown wrong → `wrong`; fixed or accepted → stays `posted`. Add each waived ruling and answered `[Q]` not already there to `settled`, with the PR and the reason. `settle` stays unfinished until the PR merges or closes, so later replies rerun it.

**Stop with** the updated rows.

## Profile

The YAML frontmatter of `.claude/review-pr.local.md`:

- `consumers` - repos that use this repo's contracts, with local paths
- `review_dir` - per-PR files (default `.reviews/`)
- `settled` - past rulings, the one source for them (default `<review_dir>/settled.md`)
- `risk` - high-risk areas
- `evidence` - MCPs holding this repo's real-world facts, with what each holds (e.g. `signoz: prod traces`)
- `verify_cap` - most verifier agents per run (default 15)

Keep the profile, `review_dir` and the `.gitignore` edit in the main checkout, the parent of `git rev-parse --path-format=absolute --git-common-dir`, whichever worktree you run from. The review itself stays where it started.

## Tracker

`<review_dir>/<target>/review.md`, where `<target>` is the PR number, else the branch name. It holds `stage` (the last stage finished), the head SHA, the calls to check, and one row per finding: priority, anchor, status, a short note, and a link to any thread that already raised it. Update it as each stage ends, as each verifier returns, and whenever the user rules on a row.

Statuses: `new`, `validated`, `posted`, `waived` (ruled fine as is), `wrong`, `dropped` (the note says why) and `side`.
