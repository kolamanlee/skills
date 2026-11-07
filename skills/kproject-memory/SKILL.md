---
name: kproject-memory
description: Use when bootstrapping a team-shared memory directory in a new project ("set up project memory", "init shared memory"), or when about to write/record a fact, decision, pitfall, rule, gotcha, or post-mortem into project memory or MEMORY.md. Also use when deciding whether a fact belongs in git-tracked project memory vs Claude Code's per-user auto-memory.
---

# Project Memory

## Overview

Team-shared, git-tracked fact store at `<repo>/memory/`. Distinct from Claude Code's built-in **auto-memory** (`~/.claude/projects/<...>/memory/`, per-user, untracked). Two modes: **bootstrap** (one-time setup) + **write** (every fact).

## When to Use

- **Mode A — bootstrap**: user asks to "set up project memory" / "init shared memory" / "把这个 memory 机制用到这个项目里" in a repo with no `memory/` yet.
- **Mode B — write**: you're about to record a decision, pitfall, rule, reference, gotcha, or post-mortem.
- **Routing**: deciding whether a fact belongs in `<repo>/memory/` (team-shared) vs `~/.claude/.../memory/` (per-user).

## Bootstrap Workflow (Mode A)

1. Verify `<repo>/memory/` does NOT exist — refuse to clobber if it does.
2. `mkdir memory/`.
3. Write `memory/MEMORY.md` from `templates.md` § `bootstrap-header`.
4. `git check-ignore memory/` — if it returns the path (gitignored), warn the user and offer to ungitignore. **Ask, don't auto-fix.**
5. Append a `## Project Memory` section to the project's `CLAUDE.md` IF that heading is missing (3-line summary + skill pointer). **Never edit existing CLAUDE.md prose.**
6. Report what was created.

## Writing a Memory (Mode B)

- **File**: `memory/YYYY-MM-DD-<slug>.md` (date = when written; slug = frontmatter `name`, kebab-case).
- **Frontmatter**: `name` (kebab-slug, NO date) / `description` (one-line recall hook) / `metadata.type` ∈ `{decision, pitfall, rule, reference}`. **NOT** auto-memory's `user|feedback|project|reference` taxonomy.
- **Body order**: lead with the fact (1-2 paragraphs) → `**Why:**` (motivation) → `**How to apply:**` (when/where this kicks in) → cross-links `[[other-slug]]` (slug = other memory's `name`; dangling links OK — they mark "worth writing later").
- **MEMORY.md bullet**: `- [Title](YYYY-MM-DD-slug.md) — hook`. The hook **can be multi-line** (3-5 lines) and includes the load-bearing rule itself. **The ≤150-char ceiling is auto-memory-only — does NOT apply here.**
- **Stale fact**: update in place; don't duplicate. Delete the file + MEMORY.md entry if obsolete.
- Copy-paste skeleton: `templates.md` § `fact-file-skeleton`.

## Choosing Project vs Auto-memory

| Project memory `<repo>/memory/` | Auto-memory `~/.claude/.../memory/` |
|---|---|
| Team-shared, git-tracked, follows the repo | Per-user, per-machine, untracked |
| `decision \| pitfall \| rule \| reference` | `user \| feedback \| project \| reference` |
| Any teammate must know to work this repo | "About this user / about this machine" |

**Hard rule: never mirror the same fact into both.** Pick one. Mirroring causes drift and contradictory updates.

Routing test: *would a new hire on this repo need to know this to avoid a regression?* → yes = project memory.

## Common Mistakes

| Rationalization | Correct behavior |
|---|---|
| "This fact is small, I'll fold it into an existing file" | One fact per file. Small facts still get their own dated file + MEMORY.md bullet. |
| "Auto-memory index is shorter, I'll put it there" | Team-shared fact → project memory only. Auto-memory is for the user, not the team. |
| "I'll skip the date prefix, slug is enough" | Date prefix is load-bearing — records when written, orders the directory. Always `YYYY-MM-DD-slug.md`. |
| "This is my workflow preference, so auto-memory" | If a teammate would need it to work this repo → project memory. Test: would a new hire need to know it? |
| "MEMORY.md hook should be ≤150 chars per the auto-memory rules" | Project memory hooks are deliberately long — include the load-bearing rule. ≤150 chars is auto-memory-only. |
| "I'll mirror to both so it's not lost" | Never mirror. Pick one. Mirroring causes drift. |
| "Auto-memory types apply here (user/feedback/project)" | No — use `decision\|pitfall\|rule\|reference`. Different taxonomy on purpose. |

## References

- `templates.md` — `bootstrap-header` + `fact-file-skeleton` for copy-paste.
- Origin project: `tonly_robot2/memory/` (28 working fact files; reference for hook style + body structure).
