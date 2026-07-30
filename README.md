# kady-skills

A Claude Code plugin bundling personal skills for planning and engineering workflows.

## Skills

| Skill | Description |
| :---- | :---------- |
| [`/kady-skills:concise`](skills/concise/SKILL.md) | Enter concise reporting mode — human-facing text gets short and plain; agent-facing documents keep full detail. |
| [`/kady-skills:describe-pr`](skills/describe-pr/SKILL.md) | Write a PR title and description that say only what the diff can't — tickets up top, operational notes surfaced, a signposted reading path through the diff, a diagram only when it earns its place. |
| [`/kady-skills:draft-plan`](skills/draft-plan/SKILL.md) | Draft a self-contained implementation plan in the project's plans directory as a handover document another agent can ship independently. |
| [`/kady-skills:grill-me-simple`](skills/grill-me-simple/SKILL.md) | Interview the user in rounds over a design decision tree, asking only what's ready to be asked, until every branch is decided or explicitly assumed. |
| [`/kady-skills:orchestrate`](skills/orchestrate/SKILL.md) | Enter orchestration mode — delegate discovery, implementation, and verification to subagents and ultracode workflows, and keep your own context clean for planning and synthesis. |

## Install

### Via `npx skills` (recommended)

[`skills.sh`](https://skills.sh) ships skills into any supported agent, including Claude Code:

```bash
# Browse the plugin's skills and pick interactively
npx skills add abdelrahman-elkady/kady-plugins -a claude-code

# Or install a specific skill directly
npx skills add abdelrahman-elkady/kady-plugins --skill draft-plan -a claude-code
```

### As a Claude Code plugin

```
/plugin marketplace add abdelrahman-elkady/kady-plugins
/plugin install kady-skills@kady-plugins
```

### From a local clone (development)

```bash
claude --plugin-dir /path/to/kady-plugins
```

Then invoke a skill, e.g.:

```
/kady-skills:draft-plan
```

After editing a skill, run `/reload-plugins` inside Claude Code to pick up changes without restarting.

## Layout

Skills are auto-discovered from `skills/<skill-name>/SKILL.md`, each with its own YAML frontmatter (`name`, `description`, and invocation flags). Add a new skill by creating a folder under `skills/` with a `SKILL.md` — no manifest update needed. Plugin-level metadata (name, description, version) lives in [`.claude-plugin/plugin.json`](.claude-plugin/plugin.json).
