---
name: review-tdd
description: Understand the problem behind a technical design document (TDD), independently assess the design, then compare judgments with the user.
disable-model-invocation: true
user-invocable: true
argument-hint: "[context|assess|compare]"
---

# /review-tdd - understand, assess, compare

Treat the user as an engineer unfamiliar with this system. Investigate enough to explain the context; go deeper during assessment. Explain briefly. Start at the stage the user names; default to context.

Start from whatever the user provides: a ticket, document, link, file, attachment, or pasted context. Follow relevant links and attachments to find the problem and TDD wherever they live, including a TDD attached to a Jira story. Don't require both upfront.

## 1. Context - explain the problem, not the proposal

- Establish the problem and scope from available sources; verify related tickets when present.
- Explain the affected users, today's flow, the problem, and the required outcome. Use examples, diagrams, or snippets where they help.
- Don't present or judge the proposal yet.
- Close by offering to start an independent assessment. Wait for the go-ahead. If the user defers, don't repeat the offer on every follow-up.

## 2. Assess - form a judgment without asking for theirs

- Check requirements, current code, preserved behavior, realistic failures, and rollout. Don't solicit the user's verdict.
- Test added complexity against simpler alternatives, and consequential performance claims against available usage evidence.
- Deliver a concise judgment and prioritized findings, each with its consequence, source, and suggested resolution.
- Label each finding a defect, an unspecified decision, or an optional improvement. State where the evidence runs out.
- Progress updates carry no findings while the user is still reading. The findings arrive together in the completed review, unless the user asks to hold them.

## 3. Compare - once the user shares their judgment

- Test agreements, disagreements, and omissions against evidence.
- Challenge both judgments. Explain any conclusion that changed, and what would resolve the remaining uncertainty.

Review and explain. Don't implement changes or post review comments unless asked.
