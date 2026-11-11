# The report

The user reads it to triage: draft a comment, file a ticket or skip. The saved agent reports hold the evidence for when a comment gets drafted.

- **Opening:** a verdict of at most two sentences, then the tables, then everything else.
- **Each item stands alone.** Write it for a reader who saw no status line and no agent report. Use no agent names and no finding IDs.
- **Complete:** every verified finding gets a row, or a line under *Left to #<child>*.

## From agent reports to items

- **One item per decision.**
  - Split parts the user could judge differently, such as where code lives vs. whether it's needed.
  - Same-kind small slips may share one item.
- **Merge** duplicates across agents. Keep the strongest evidence.
- **Fold** evidence that only strengthens another item into that item. If it is the reason to act now (e.g. a PR already copying the pattern), name it in that item's row and in the verdict.
- **Pending comments:** one that holds goes in *Already covered*, with no section, even when it needs an addition or a correction. One that the evidence refutes stays an item: link it, and say why.
- **Stacked PRs:**
  - When the change belongs only in the stacked PR, give it one line under *Left to #<child>* and no item, so that no comment on this PR asks for that work.
  - When a fix here changes what the stacked PR must implement, keep the item, mark it `affects #<child>` and name it in the verdict.
  - A repeated PR-body row or a mechanical edit (import, rename, mock) gets a short note in the Suggestion and no tag.
- **Cut** what submitted reviews or settled rulings already cover, and what verification refuted.

## Weights

There are exactly four. Priority and blocking are the user's call.

- `change`: a structural choice in the wrong layer or module, the kind that spreads when copied. Cite the rule or the count.
- `simplify`: correct and well placed, but a simpler shape (often the counterpart) meets the same spec. Over-built code stays `simplify` even when the simpler shape is clearly better.
- `nit`: a small idiom, naming or file-location slip.
- `flag`: a behaviour change or risk the PR doesn't state.

## Status

Combine values with a comma.

- `new`: this PR adds it.
- `extends <#parent | existing pattern>`: this PR grows something older.
- `not this PR`: older code that this PR doesn't touch; a ticket candidate.
- `your call`: a judgement call, such as a policy trade-off or a pending comment that the evidence refutes. A trade-off with no finding becomes its own item.
- `affects #<child>`: a fix here changes what the stacked PR must implement.

## Order

1. items that answer a suspicion or refute a pending comment, and judgement calls with no finding;
2. `change`;
3. `simplify`;
4. `nit`;
5. `flag`, with `not this PR` last;
6. *Already covered*.

Number the items 1 to n across both tables, with no gaps. Numbers never change once shown.

## Language

Write in ASD-STE100 (Simplified Technical English), about 75% strict.

- One topic or instruction per sentence, at most 25 words.
- Active voice and simple tenses.
- Commands for suggestions, with the condition first: "If X, do Y."
- Complete sentences with their articles, in bullets too.
- At most three nouns in a row, and one term for one thing across all items.
- Normal engineering words are fine outside the STE dictionary. Table cells and headings may be short claims.

Example: "Since the cap only bites after the deploy, flipping it early would silently truncate bodies" becomes "Set the cap after the deploy. If you set it first, the logger cuts request bodies."

## Template

````markdown
# PR #<n> architecture review (head `<short sha>`)

<Verdict, at most 2 sentences: what should change and why now; how over-built the PR is, in words.> Nothing has been posted to GitHub.

| # | Issue | Weight | Status |
|---|---|---|---|
| 1 | <short claim> | change | new, affects #<child> |
| 2 | <short claim> | simplify | new, your call |
| 3 | <short claim> | flag | not this PR |

**Already covered:**

| # | Issue | Covered in | Add |
|---|---|---|---|
| 4 | <short claim> | [your comment](<link>) | <the addition or correction, if any> |

**Left to #<child>:**
- <What>: <#child does this (link) | #child does not do this yet>.

**Your questions:**
- **<The suspicion, answered as a verdict> (#<item>).** <The fact that settles it.>

### 1. <Issue, in the table's words>
- **Target:** [<file>#L<a>-L<b>](<blob URL at the full head SHA>): <what the code does>.
- **Why:** <the rule, the spec point or the effect>.
- **Suggestion:** <the move or the simpler shape, as a command>.
- **Your call:** <what each option costs; only with that status>.
- **#<child>:** <what the stacked PR must implement differently; only with that status>.

### Nits
- **#<item>.** [<file>#L<a>](<blob URL>): <the change, as a command>.

## Checked and fine
- <What> is fine: <why>.

## Caveats
- <What changes what the user would do.>

Saved: <links>. Reply with your picks, for example: `comment 1 2, ticket 3, skip the rest`.
````

- **Bullets** hold one or two sentences. A `not this PR` item has **Target** and **Why** only.
- **Evidence:** line counts, drift, worst cases and steelmans stay in the saved agent reports. Use one only when it would change the user's call, or when a count is the evidence. Write "not measured" only where it affects the call.
- **Tail:** at most 5 *Checked and fine* bullets, for the points the user or the author will most likely raise. At most 3 *Caveats*, for what changes what the user would do. The rest stays in the saved files.
- **Empty parts:** drop any table, bullet or section that would be empty.
- **Anchors** are blob URLs at the full head SHA. For another PR, use that PR's head SHA.
