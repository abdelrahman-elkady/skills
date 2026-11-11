# Briefs

The agent knows only its brief and the context file. Fill every `<...>` with real paths, SHAs and names, and drop clauses that don't apply.

**Rule out the naive answer.**
- Name the counterpart with the gaps you saw in it, and say why the obvious reuse fails when it does.
- Handed only the user's hypothesis, an agent confirms it. Handed the real alternative and its costs, it weighs them.

## Shared setup

> You are reviewing <the **<subsystem>** | **the placement of every file**> in PR #<n> (<repo>, <stack>). <Goal in one sentence.>
>
> **Read-only.** Your final reply is your only output.
> - Read the PR with `git show <head>:<path>`, `git grep -n <pattern> <head>` and `git diff <base>...<head>`.
> - Probe installed modules with one-off scripts.
> - No edits, commits, branch switches, GitHub writes, tests, installs or subagents. Never fable or forks.
>
> - **Repo:** `<path>`, on `<branch>` with the user's work in it. Leave it as is.
> - **Base:** `<base>` @ `<sha>`<, the open parent PR #<parent>: its code is context, not a finding>. Tag each finding **new in #<n>** or **extends <#parent's | an existing> pattern** (check with `git show <base>:<path>`).
> - <**Built on this PR:** #<child> @ `<sha>`. Read it for what it copies, what it already does, and what it uses from this PR. Its code is not a finding. Also tag each finding by the PR that must change:
>   - **here**;
>   - **here, affects #<child>**: say what #<child> must implement differently;
>   - **left to #<child>**: cite where #<child> does it, or say that it does not yet.>
> - **Exclude from searches:** `<stale copies>`.
> - **Context:** `<context file>`. Read it first.
>   - Already-raised items: reference them, don't re-report them.
>   - Pending-review comments: hypotheses to test.
> - **Conventions:** `<instruction files>`, what they point to, and the sibling files in `<folders>`.
> - **Evidence:**
>   - A convention claim cites the rule or a sibling count ("the only one of 41 files in `<dir>/` that …"), and says whether the convention is written or observed.
>   - A library claim cites the installed version's source line.
> - **Others:** <agent> covers <scope>. Where your scope touches theirs, give it one line.

## Deep dive

> **Files:** `<path>` (<new | changed>; <role>), and how `<callers>` use them.
>
> **Questions.** The first <k> are the reviewer's hypotheses: test them, and the answer may be no.
> 1. …

Write one question per suspicion, pending comment and inventory entry, from these patterns:

- **Placement:** what `<layer>/` otherwise holds, and whether anything like X lives there. Ask for a verdict and a concrete target.
- **Necessity**, for every machinery entry:
  - First ask for the exact semantics, step by step. Name the subtle points: ordering, what a rejected request still consumes, expiry, error paths.
  - Then ask, for each alternative:
    - how the semantics drift, and in which direction;
    - whether the spec still holds;
    - what keeping today's behaviour would cost;
    - the worst case, in one sentence.

    The alternatives:
    - (a) the counterpart `<path>`, with its gaps, and how the installed version behaves on <the failure the spec cares about>;
    - (b) the bluntest option (<plain calls | a library | no abstraction>), with its drift priced in user-visible terms ("one or two extra emails under a race");
    - (c) keep X, but relocate it.
- **Separate verdicts** for placement and necessity: the user may block on one and defer the other.
- **Invariants:** the spec rules every alternative keeps. Ask the agent to trace how the PR meets each one today.
- **Copies:** does <#child | the next ticket> copy or need this? Does that argue for shared infrastructure over a one-off? Would the fix here change what <#child> must implement?
- **Moved code:** did the move change behaviour? Compare line by line.
- **Base-PR edits:** is the edit needed?
- **New patterns:** is it house style? Compare with the sibling <source | tests> (`<paths>`).

End every deep dive with: the simplest design for your area that meets the spec, with its files, the layer of each, and LOC against the PR's.

> **Output** (complete; another agent reads it): a verdict per question; per finding `path:line` at `<head>`, tag, what's off, a steelman, the simpler alternative as a sketch, spec constraints kept or dropped, worst case, LOC delta, confidence; the simplest-design sketch; checked and fitting; could not verify. Mark each claim measured or reasoned.

## Placement sweep

> Scan PR #<n> for code in unconventional layers or modules: <the instruction files' layer rules, phrased as the violations to look for>. The reviewer's examples: <verbatim>.
>
> Go for breadth over depth: every non-test file, plus where each test sits. For files <other agents> own, give placement facts in one line.
>
> Per file, check:
> - the layer, and whether the content matches it;
> - dependency direction, including values a higher layer branches on;
> - the domain folder: feature code in shared places, or shared infrastructure under a feature;
> - the homes for shared values, errors, config and logging, by file count;
> - a new idiom in a file that already has one;
> - whether entry points, mounts, registries and config follow their existing pattern;
> - request validation, where the sibling routes have it;
> - tests: in the right project, at the mirrored path;
> - <PR-specific checks from the inventory>.
>
> **Output** (complete):
> - a table of every non-test file: layer, verdict (fits / unconventional / wrong place), tag, reason;
> - the same table for test files;
> - per flagged item: `path:line`, the broken convention with its rule or count, the target, a steelman, your confidence;
> - what you checked and found fitting;
> - what you could not verify.
