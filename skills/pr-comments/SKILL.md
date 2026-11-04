---
name: pr-comments
description: Write and post GitHub PR review comments. Use whenever review feedback is headed for a PR, like, "comment on the PR", "add it to the pending review", a reply in a review thread, `gh pr review`. Not for finding the issues (/code-review) or writing the PR body (describe-pr).
user-invocable: true
---

# /pr-comments - the author reads it cold

The author reads every comment cold: the chat, the digging and the dropped drafts never reach them. The three comments below are the template; keep their frame and swap in the point.

## The template

**A finding.**

- The bold lead opens with its priority, `[P0]`-`[P3]`, and names the problem by its cause, in the domain's words: "A basic hash helper lives in one service under a use-specific name". The consequence (for that helper, the duplication) goes in the body.
- Understate a trade-off ("a bit of", "no huge"); state a bug or a blocking ask plainly.
- One fact per paragraph, stopping at the consequence. One supporting number is enough.
- The fix goes under `**Suggestion:**`. When it moves code, it says who owns what.

```markdown
**[P2] Stock `reconcile` also places purchase orders, beyond updating counts**

The job registry documents `reconcile` as updating stock counts from the warehouse feed, but this one also writes `purchase_orders` and `supplier_ledger`. So anything that reconciles stock can order from a supplier and book the cost.

The reorder rule is split as a result: raised here, cancelled in [`discontinue-product.js`](https://github.com/<org>/<repo>/blob/<full-sha>/services/catalog/discontinue-product.js#L40-L41).

**Suggestion:**
move the reorder and the ledger booking into a purchasing service that keeps them in one transaction, so the cost lands in the period the order was placed.

The flows that can trigger a reorder call that service as part of their normal business logic, and `reconcileStock` owns only the counts.

---
🤖 Generated with Claude Code
```

**A small point.**

- The same frame at its shortest. When the user asks for a "concise" or "small" comment, cut to this length and keep the frame.
- A convention point gives the project's own convention as the reason, shown by one existing file that follows it.
- `[nit]` may follow the priority on a naming, placement or convention point.

```markdown
**[P3] [nit] This spec sits outside the `billing/` folder**

Unit specs mirror the source path, and the sibling `require-billing-account.spec.js` lives in `unit/middlewares/billing/`.

**Suggestion:**
move it to `unit/middlewares/billing/require-billing-session.spec.js`.

---
🤖 Generated with Claude Code
```

**A product question** (`[Q]`), for a call the code can't make. The lead is the question, then today's behavior, then what each answer changes. It ends there: the answer is the author's.

```markdown
**[Q] Should a cancelled order still count toward the monthly limit?**

Today it does, so a member who cancels and re-orders hits the limit one order early.

If that's intended, this is fine as is. If not, `countMonthlyOrders` has to skip cancelled orders, and members who hit the limit that way get the slot back.

---
🤖 Generated with Claude Code
```

## Every comment

- **One idea, one anchor.** Two concerns are two comments, even on the same line.
- **Only what the diff does.** Every sentence is about this change. On the reread, cut each sentence that recaps the chat, guards against a change nobody made, or announces structure ("two parts: one settled, one open").
- **Stands alone.** Finding IDs and draft names live only in the chat, so a cross-reference names the file or links the thread. It still reads right if the other comment is dropped.
- **Settled stays settled.** Before drafting, read the project's rulings: the file the caller names, else the project's memory. An issue the author deferred stays out of the review, in this PR and the next.
- **References:**
  - code: short text over a full-SHA blob link, as in the first example.
  - a person to coordinate with: their GitHub @handle.
  - a quoted foreign-language string: its English in parentheses right after it.
- **Closers come from the user.** When they scope a point one of these ways, end it with the matching closer:
  - can wait: "This can be a follow-up after this PR merges."
  - a side point inside a blocking comment, after the suggestion: `**Not for this PR:** <point>. That can be a separate refactor after this merges.`
- **The signature** ends every comment, reply and non-empty review body, exactly as in the template. Keep the blank line above `---`; without it the last paragraph renders as a heading.

## The review

- **One review per PR.** Its comments sit on lines the diff shows, with `start_line` + `line` for a range.
  - A point on code outside the diff anchors to the nearest line the diff shows and names the true location.
  - A point about a whole file is a file-level comment.
  - A point with no line goes in the review body. Otherwise the body stays empty.
- **Verdict:** `COMMENT` when every comment is `[P3]` or `[Q]`, or on the user's own PR, where GitHub rejects `REQUEST_CHANGES`; otherwise `REQUEST_CHANGES`.
- **A reply** goes in its thread, alone. One that carries a finding leads with its priority.

## Posting - a hard stop

1. **Show the exact set:** anchor, text, verdict. Post only what the user approves in this turn, on the set as they saw it; show anything changed since again. "Post this" means the comment under discussion, not the list.
2. **Check for the user's own PENDING review** (`GET /repos/{owner}/{repo}/pulls/{n}/reviews`). GitHub allows one per user per PR, so creating a second fails. If one exists, ask before touching it: add the threads to it, or post after they submit it.
3. **Post through the API**, byte for byte as approved (`gh pr review` posts only a body). If the user edited a draft, re-read it from where they edited it.
   1. **New review:** `POST /repos/{owner}/{repo}/pulls/{n}/reviews` with `commit_id` (the current head; re-anchor if it moved since drafting) and `comments[]`. Leave out `event` so the review stays PENDING while file-level threads are added.
   2. **File-level points, and every thread for the user's existing pending review:** GraphQL `addPullRequestReviewThread`, with `subjectType: FILE` for a file-level point.
   3. **Submit:** `POST /repos/{owner}/{repo}/pulls/{n}/reviews/{id}/events` with the verdict as `event`. The user's own pending review is theirs to submit: threads added to it take its verdict, and you submit it only on their word.
   4. **Replies:** after any submit, each alone: `POST /repos/{owner}/{repo}/pulls/{n}/comments/{comment_id}/replies`, with the thread's top comment as `comment_id`.

**Editing a pending comment:** right before the edit, re-read the review's state and stop if it is no longer PENDING. The edit call also rewrites submitted comments, whose notifications already carried the old text.
