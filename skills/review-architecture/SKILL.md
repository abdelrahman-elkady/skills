---
name: review-architecture
description: Architecture and simplification review of a PR with agents, ending at a verdict and a findings table. Read-only.
disable-model-invocation: true
user-invocable: true
argument-hint: "<PR> [what looks off to you]"
---

# /review-architecture

For every mechanism the PR adds, judge three things against its spec and this codebase:

- **Placement:** does it live where this repo puts such code, written this repo's way?
- **Necessity:** does the spec need this machinery?
- **Simplest shape:** what is the leanest design that still meets the spec?

This is a design review that ends at the report.

The user's suspicions are **hypotheses**. Agents test them, the verdict may be no, and the report answers each one.

## Guardrails

- **Read-only; the user decides what reaches GitHub.**
  - Read the PR at its head SHA (`git show`, `git grep`). `git fetch` is the only git write.
  - Verify with read-only `git`/`gh`, and with one-off scripts against installed modules.
  - No edits, branch switches, tests or installs.
- **Every Agent call:** `subagent_type: "general-purpose"`, an explicit `model: "opus"` (on another harness, its strongest model), in the background. Never a fork or fable.
- **Cap:** use the number the user names. Otherwise use up to 5 without asking.
  - A sixth agent, at any point, needs the user's yes. Propose it in one line with its scope.
  - Ask rather than merge briefs to fit.

## 1. Orient

Gather what makes the briefs concrete; the agents do the deep reading.

1. **PR and stack:** the head SHA, base and files.
   - If the base isn't the default branch, find the parent PR. Its code is context, not a finding.
   - Find PRs and branches built on this one (**stacked PRs**). Note what each one takes on and what it uses from this PR. They show where the pattern is **copied**, and which work belongs there.
2. **Diff:** read the whole non-test diff. Group its files into **subsystems**, one mechanism each.
3. **Conventions:** read the repo's instruction files and what they point to.
   - Those files and the sibling code are the only yardstick.
   - Note which conventions are written down and which are only observed.
4. **Inventory:** list every new mechanism:
   - a client or connection, an SDK or provider call, a script, a new kind of file in a layer;
   - a post-response or async pattern, a test seam, moved code, an edit to a base-PR file.

   Pair each with its nearest existing **counterpart**. Read the counterpart, and note its defaults and exemptions that would change behaviour on reuse. Suspicions join the inventory.
5. **Spec:** the linked ticket and design doc, verbatim. If either is unreachable, say so in Caveats.
6. **Prior review**, in two groups:
   - **Already raised:** submitted reviews, other people's and bots' threads, and rulings settled in notes or memory. These are excluded from findings.
   - **The user's pending review:** their unsent drafts. Each comment is a hypothesis, like a suspicion.
7. **Context file** `<scratchpad>/pr-<n>-context.md`:
   - the PR and stack SHAs;
   - per stacked PR: what it takes on, and what it uses from this PR;
   - the spec text;
   - one **forced / chosen** line per inventory entry: what the spec requires against what the PR chose;
   - prior review in its two groups, with links;
   - the stale-copy directories that searches exclude.

**Done when** every inventory entry has a counterpart or "none", and every suspicion and pending comment belongs to one agent.

## 2. Split and brief

- **Deep dive:** one per subsystem. Suspected subsystems come first, then the main remaining flow.
- **Placement sweep:** exactly one, on every run. It covers every non-test file, plus where each test sits.

The usual split is two deep dives plus the sweep. Fill one brief for each from [references/briefs.md](references/briefs.md).

**Done when:**
- no `<placeholder>` remains;
- every inventory entry and suspicion is a question in exactly one brief;
- each brief names what the others cover.

## 3. Dispatch and wait

- Launch all the agents in one message. Tell the user each agent's scope, what's excluded, and the context-file link.
- While they run, send one status line per finished agent. A finding shared early still appears in full in the report.

## 4. Verify each report as it lands

- **Load-bearing claims:** a claim is load-bearing if, when false, the finding dies or its weight changes. Examples: census counts, copies in other PRs, what a stacked PR does or needs, behaviour changes, "unused" claims, library behaviour. Re-run the cheapest decisive check yourself, at head (a stacked PR's claims at that PR's head).
- **"Could not verify":** settle what a short check can. True becomes a finding; false goes to *Checked and fine*.
- **Agents disagree:** check it yourself. A judgement call gets a **Your call** bullet on its item.
- **Before writing:**
  - take line ranges from `git show <head>:<path>`;
  - re-read the head SHA;
  - re-verify the files any new commits touched.

**Done when** every item rests on a check at head, or is marked "reasoned, not measured".

## 5. Report

- Write the report in chat, following [references/report.md](references/report.md).
- Save it and each agent's hand-back to the scratchpad, and link them.
