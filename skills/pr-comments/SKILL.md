---
name: pr-comments
description: Write and post GitHub PR review comments - one idea per comment, address only what the diff does, bold lead + one fact per paragraph, AI attribution on every comment, hard stop before posting. Use whenever review feedback is about to land on a PR: drafting inline comments, replying to a review thread, `gh pr review`, "post these findings", "comment on the PR". Not for finding the bugs (/code-review) or writing the PR body (describe-pr).
user-invocable: true
---

# /pr-comments - the author reads it cold

Every comment lands in front of someone who has none of the reasoning that led to it - not the chat, not the ticket digging, not the discarded drafts. Framing, hedges, and preemptive defenses cost their time and dilute the one thing being asked. Everything below follows from that.

## Three rules

**One idea per comment.** A naming question and a correctness constraint are two comments, anchored at their own lines - never one long comment with sections. If the ideas share an anchor, the split still wins.

**Only address what the diff actually does.** Never argue against a position the author never took. A "keep this NOT NULL" warning on a column nobody proposed changing reads as a misread of the migration. What was agreed in chat is not context the author has; what you were tempted to warn about is not a change they made.

**Bold one-line claim first**, then one fact per paragraph with blank lines between. No headers inside a comment. Anything the author can re-derive from their own diff gets trimmed by the reader anyway - trim it first.

## Scope is the author's call

A pre-existing issue the author has explicitly deferred — "known, not this PR, owned by a separate discussion" — is settled. Don't re-raise it as a finding, in this PR or the next one that touches the same file. Review what the changed code controls (its own error isolation, idempotency, atomicity) and leave the surrounding machinery's semantics to the discussion that owns them. Project-specific rulings live in that project's memory; check it before drafting.

## The cut pass

Draft the comment, then delete every sentence that:

- recaps something already agreed in chat,
- guards against a change the PR doesn't make, or
- is scaffolding ("two parts - one settled, one open").

Park a deferred point until its prerequisite question is answered instead of including it conditionally. A question that needs a product or business decision is flagged as one in the comment itself - it isn't a defect and must not read like one.

## Posting - a hard stop, every time

**Never post without explicit approval in the current turn.** Show the draft, discuss, revise. Approval for one comment is not approval for the rest; approval last session is not approval now.

On approval, new findings go out as **one review, not a pile of comments** - a single GitHub review whose inline comments anchor to the code each addresses, never standalone top-level comments. A reply to an existing thread is the one exception: it lands in its thread, alone.

## Signature

Every comment — inline, reply, and review body alike — ends with:

```markdown
---

🤖 _AI-generated review comment from [Claude Code](https://claude.com/claude-code)._
```

It is not a default and it gets missed; add it before posting, not after.
