---
name: describe-pr
description: Write or rewrite a pull request description and title — concise, high-signal, tickets up top, operational notes surfaced, a how-to-review reading order, a mermaid diagram only when it shows what prose can't. Use whenever a PR body is being drafted or edited, including `gh pr create` / `gh pr edit`, "open a PR", "write the PR description", "update the PR body". Not for reviewing someone else's PR.
user-invocable: true
---

# /describe-pr — say only what the diff can't

Two people read a PR body: a reviewer deciding where to look, and whoever deploys it deciding what to do by hand. Write for those two and nobody else — and write it backwards: settle the diagram on the diff, draft the reading path, write the body into what's left, title last.

**Every fact has one home, and the body is the home of last resort.** The diff holds the files, the mechanism, the tests; the ticket holds the backstory and the proof the problem was real; the reading path holds the reasons a reviewer needs mid-review. The body keeps only what nothing else carries: the one-line why, the steps a human must take, what's left undone and still matters, a decision no artifact records. Two things have no home anywhere and stay out — **the alternative you rejected**, and **what you checked and found clean**. A fact outside its home is a copy, and the copy is the cut; a fold launders nothing. Length tracks decisions, not files.

Read the real diff (`gh pr diff`, or `git diff <base>...HEAD`) and fetch every ticket you cite before writing a word. Never invent a ticket key, never claim tests pass unless you ran them, never describe work that isn't in the diff.

## Load concise first

Read its `SKILL.md` before drafting — `.claude/skills/concise/`, `~/.claude/skills/concise/`, or `~/.claude/plugins/**/skills/concise/`. Not via the Skill tool: concise sets `disable-model-invocation`, so the call is rejected. **It governs the register and the separation; this file governs what goes in and how tight it gets.** Its *give each distinct idea its own visible spot* binds here in full: the separation lands in bold lead-ins and bullets under `##` headings, not concise's `###` device.

Not installed: write the PR anyway — a hint, never a gate — and say so once in chat, never in the body: `npx skills add abdelrahman-elkady/kady-plugins --skill concise -a claude-code`.

## The diagram test

Add a mermaid diagram when it makes a relationship visible that prose can only list — several distinct paths converging on one outcome, a race, a state machine, an ordering that has to hold, a wiring that differs before and after. One picture of the shape beats three paragraphs, and it is the one place the holistic view belongs.

**Settle it on the diff, before a line of prose exists**: name in one sentence the non-obvious thing a picture would carry. Named, draw it and never write that prose at all — **the diagram buys a paragraph back, it does not cost one**. Unnamed, there is no diagram. What follows the picture states what it proves and moves on; prose that re-walks the arrows is the paragraph the diagram already bought.

- **Earns it:** *every route out has to exit non-zero* — arrows from the handler, from the flush timeout, and from a signal that races the flush, all landing on one exit node.
- **Earns it:** *the retry moved from the caller into the worker* — before-and-after wiring where the boxes are the same and only the arrows moved.
- **Decoration, omit:** a box per bullet · an unchanged, unbranching A→B→C a sentence already states · a redrawn file list.

## The reading path — drafted first, rendered last

GitHub sorts the diff alphabetically, never the order that makes the change comprehensible. Close the body with the order that is, folded — but only if you can name the file a reviewer should open **second**; when that's "any of them", there's no order to teach and no block. **The path is where reasons live**, each written here once; a body cut that orphans a promise the path makes moves the reason into the path, not out of the PR.

```markdown
<details>
<summary><h2>🧭 How to review</h2></summary>

➞ **The write path** — read in order

`routes/thing.js` `create()` → `services/thing.js` `enqueue()` → the `things` insert

Every branch has to leave the row and the job agreeing.

The duplicate-key early return is the one that doesn't — it commits the row and skips the enqueue.

➞ **Who else enqueues** — both bypass the route above

- `jobs/backfill.js` — same contract, no retry; a failed enqueue is silent here.
- `cli/import.js` — same, and it runs unsupervised.

</details>
```

**Two paths, three at the outside**, load-bearing first, then a different angle. A route is files and symbols in run order and nothing else — no clause hanging off a hop, no aside in parentheses, never line numbers, which rot on the next push; it *skips* files, and that is what keeps it a route and not the change list. Nothing load-bearing hides behind the fold: an operational step, an open item, a question you want answered stays visible, or goes in an inline comment a reviewer can reply to.

Mechanics: the summary is `<summary><h2>🧭 How to review</h2></summary>`, verbatim — `##` inside `<summary>` renders literally, and GitHub styles `summary h2` inline-block, beside the triangle · blank line after `</summary>`, or the inside renders raw · `➞` is the literal character, not an entity · every break is a blank line, never `<br>`, which GitHub doubles.

## The body — written last

Default is nothing. **A block exists for something a reviewer would still get wrong after reading the whole diff — not something they would reach slower.** *True* and *worth knowing* are neither. A slot with nothing in it disappears, heading and all: no `## Testing` reading "tests added", no "N/A". Silence means none.

```markdown
[ABC-1234](https://tracker.example.com/browse/ABC-1234) caps importer retries and sends failed rows to a dead-letter queue.

## Changes

- An unparseable row used to retry until someone purged the queue by hand. It stops after three attempts now.
- Failed rows carry the batch id and the parse error.
- Retry limits are per batch, not per worker:
  - `IMPORT_MAX_RETRIES` — no default
  - `IMPORT_DLQ_TOPIC` — defaults to `rows.dead`

## Before merging

Set `IMPORT_MAX_RETRIES` in staging and prod.

## Not covered

- **The ticket's third condition** is unverified — it needs a batch large enough to page.
- [ABC-1240](https://tracker.example.com/browse/ABC-1240): the reconciler still reads the old retry column.
```

**`## Before merging` never has to earn its place** — steps, env vars, migrations, feature flags, config and infra changes, breaking changes, none of them inferable from the diff. The action and who takes it, imperative, with no argument for it. Confirming that nothing needs doing is not a step.

**`## Not covered` earns every line.** Only a gap that changes what someone does next — a ticket that can't close, a follow-up to file, a case a reviewer would assume was handled. Skipped spec detail is not a gap when the intent is met and nothing is at risk; leave it out.

Two tells that a cut is leaking back in:

- **The trailing clause.** Cut material returns riding a dash or a *rather than* on a sentence that earned its place — the mechanism, the rejected alternative, the reason the path already carries. End the sentence where its claim ends.
- **Imported evidence.** A ticket-backed claim links the ticket and repeats none of its numbers; only a claim no ticket holds may carry its own proof — a log line, a query result — as its own short sentence, never a clause chained onto the first.

## The title

One line, no padding: the repo's commit convention (often `type(scope): summary`), the ticket key where the repo puts it, and the change stated as behavior — `fix(ABC-1234): exit non-zero on uncaught exception`.
