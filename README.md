# kolaman-claude-plugins

Personal plugin **marketplace** for kolaman's reusable Claude Code plugins. The repo root is the marketplace; each skill lives in its own plugin under `plugins/`.

## Plugins

| Plugin | Description | Install |
|---|---|---|
| [`kproject-memory`](plugins/kproject-memory/) | Team-shared, git-tracked project memory: bootstrap + write guidance. Distinct from Claude Code's per-user auto-memory. | `/plugin install kproject-memory@kolaman-plugins` |

## Installation

**From GitHub** (after this repo is pushed):

```bash
/plugin marketplace add kolaman/kolaman-claude-plugins
/plugin install <plugin-name>@kolaman-plugins
```

**From local checkout** (development / testing):

```bash
git clone <this-repo-url> ~/kolaman-claude-plugins
# In any Claude Code session:
/plugin marketplace add ~/kolaman-claude-plugins
/plugin install <plugin-name>@kolaman-plugins
```

Verify: `/skill list` should show the plugin's skills.

## Repo Layout

```
kolaman-claude-plugins/          # repo root = marketplace
  .claude-plugin/
    marketplace.json             # plugin catalog; one entry per plugin
  plugins/
    <plugin-name>/               # one self-contained directory per plugin
      .claude-plugin/
        plugin.json              # plugin manifest
      skills/
        <skill-name>/
          SKILL.md               # skill doc (+ optional aux files)
      README.md                  # plugin-level docs
  docs/superpowers/              # specs & implementation plans
```

## Adding a New Plugin（三步走）

1. Create the plugin manifest at `plugins/<name>/.claude-plugin/plugin.json`:

```json
{
  "name": "<name>",
  "description": "<one-line description>",
  "version": "0.1.0",
  "author": {
    "name": "kolaman",
    "email": "kolaman@users.noreply.github.com"
  },
  "keywords": ["<keywords>"]
}
```

2. Create the skill at `plugins/<name>/skills/<skill-name>/SKILL.md`:

```markdown
---
name: <skill-name>
description: Use when <trigger conditions — this is what Claude matches on>
---

# <Skill Title>

<skill body>
```

3. Register the plugin in `.claude-plugin/marketplace.json` → `plugins` array:

```json
{
  "name": "<name>",
  "description": "<one-line description>",
  "source": "./plugins/<name>",
  "category": "productivity"
}
```

Commit + push, and it's installable: `/plugin install <name>@kolaman-plugins`.

> Tip: copy `plugins/kproject-memory/` as a skeleton — fastest path to a working plugin.

## Uninstall

```bash
/plugin uninstall <plugin-name>@kolaman-plugins
/plugin marketplace remove kolaman-plugins
```

## License

MIT (or your preferred license — update this line before publishing)
