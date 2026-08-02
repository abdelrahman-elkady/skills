---
name: describe-pr
description: Write or rewrite a pull request description and title — concise and high-signal, tickets named up top, operational notes surfaced, a how-to-review block handing the reviewer a reading order, a mermaid diagram only when it shows what prose can't. Use whenever a PR body is being drafted or edited, including `gh pr create` / `gh pr edit`, "open a PR", "write the PR description", "update the PR body". Not for reviewing someone else's PR.
user-invocable: true
---

# /describe-pr — say only what the diff can't

Two people read a PR body: a reviewer deciding where to look, and whoever deploys it deciding what to do by hand. Write for those two and nobody else. **The diff already lists the files, the functions and the tests; the tickets already hold the backstory. Your only job is what neither of them shows.** Roughly a line of prose per file changed is the right order of magnitude — past ~20 lines you are padding. The reading path is navigation, not prose — it sits outside that budget, under a cap of its own.

## Load concise first

The `concise` skill governs the title and the body. Read its `SKILL.md` before drafting — `.claude/skills/concise/`, `~/.claude/skills/concise/`, or `~/.claude/plugins/**/skills/concise/`. Not via the Skill tool: concise sets `disable-model-invocation`, so the call is rejected.

Not on any of them: it isn't installed. Write the PR anyway — a hint, never a gate — and tell the user once in chat, never in the PR body: `concise` isn't installed, the body reads better with it — `npx skills add abdelrahman-elkady/kady-plugins --skill concise -a claude-code`.

## Facts come from the diff and the tickets

Read the real diff (`gh pr diff`, or `git diff <base>...HEAD`) and fetch every ticket you cite before writing a word. Never invent a ticket key, never claim tests pass unless you ran them, never describe work that isn't in the diff.

## What earns a line

**Tickets first, unmistakable.** Key, full link, and what each one *is*. Say the relationship out loud when there is one — two tickets on the same defect, or a ticket that left a decision open and which way you went.

> Fixes **[ABC-1234](https://tracker.example.com/browse/ABC-1234)** (Task) and **[ABC-1240](https://tracker.example.com/browse/ABC-1240)** (Bug) — same defect, filed twice.

**One line of why.** What the old code did wrong, in the present tense of the old code.

**Behavior, not files.** What the system does differently now — one line per real decision, naming the calls a reviewer would otherwise question. In prose, name a file only when the name is the news: a rename, a new module, a moved boundary.

**What a human must do that merging won't do for them.** Pre- and post-merge steps, new env vars, migrations, feature flags, config and infra changes, breaking changes. These are the highest-value lines in the body — a reviewer who misses one breaks prod — and none of them are inferable from the diff, so none of them get cut.

**What the PR leaves undone.** A ticket only partly closed says so, in a closing line. Work you didn't do stays named as not done.

Anything that doesn't apply is omitted, heading and all: no `## Testing` reading "tests added", no empty screenshots section, no "N/A". Silence means none.

**Out entirely:** a file-by-file change list · a test-by-test breakdown · line counts · restating the ticket · a summary of the summary · anything a reviewer reads faster in the diff itself. A fold launders none of these — what's cut is cut, collapsed or not.

## The diagram test

Add a mermaid diagram **only if it makes a relationship visible that the prose does not** — several distinct paths converging on one outcome, a race, a state machine, an ordering that has to hold. Then delete the prose it supersedes: a diagram substitutes, it never accumulates. Can't name in one sentence the non-obvious thing it carries? Default to none.

- **Earns it:** *every route out has to exit non-zero* — arrows from the handler, from the flush timeout, and from a signal that races the flush, all landing on one exit node. The reader sees the invariant; prose can only assert it.
- **Decoration, omit:** a box per bullet · a linear A→B→C a sentence already states · one caller and one callee · a redrawn file list.

## The reading path

GitHub sorts the diff alphabetically, which is never the order that makes a change comprehensible. Close the body with the order that is — last, below the diagram and the open items, folded. A bare grey triangle at the end of a long body is exactly what a reviewer scrolls past, so the summary is a heading: it renders at `h2` size, and the thing a reviewer can't miss is the same thing they click.

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

Write one only if you can name the file a reviewer should open **second**. When that's "any of them" — one file changed, or six independent leaves — there's no order to teach and no block.

**Two paths, three at the outside**, load-bearing first, then a different angle: the failure route, the other entry points, the change walked back from its consumer.

**One idea per paragraph, a blank line between every one.** The `➞` is what holds a path together once the blank lines have pulled it apart: everything from one marker to the next is one path.

- **Label** — `➞`, then a bold name, alone on its line. A trailing `— gloss` earns its place only by saying what shape follows: `— read in order`, `— all three point at config/error-policy.js`.
- **Route** — files and symbols in the order they run, `→` between hops, nothing else: no clause hanging off a hop, no aside in parentheses, never line numbers, which rot on the next push. It *skips* files, and that is what makes it a route and not the change list.
- **Reason** — one paragraph per thing worth knowing, not one per path. The invariant the walk proves is one; the guard that makes it hold at the edge is another.
- **Fan-in** — sites reaching the same target for *different* reasons get a bullet each: the path, an em dash, its own reason. One reason covering all of them stays a comma list on the route line.

A spec earns a hop only at the end of a route, where it pins the invariant faster than the code states it.

**Nothing load-bearing hides behind a fold.** An operational step, an open item, a question you want answered — those stay visible, or go in an inline comment where a reviewer can reply.

Mechanics: the summary is always `<summary><h2>🧭 How to review</h2></summary>`, verbatim — HTML because `##` inside `<summary>` renders as literal `##`, and GitHub styles `summary h2` inline-block so it sits beside the triangle rather than below it · blank line after `</summary>`, or GitHub renders the inside raw · `➞` is the literal character, not an entity · every break is a blank line, never `<br>` — GitHub doubles it into `<br><br>` in a PR body.

**Out of the block:** a checklist to tick · "look carefully at X" with no route.

## The title

One line, no padding: the repo's commit convention (often `type(scope): summary`), the ticket key where the repo puts it, and the change stated as behavior — `fix(ABC-1234): exit non-zero on uncaught exception`.
