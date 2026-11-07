# Global CLAUDE.md — Project Memory Section

**Purpose**: This file contains the `## Project Memory` section that should be appended to your global `~/.claude/CLAUDE.md` (or `~/.config/claude/CLAUDE.md` if you use XDG config). It complements the `kproject-memory` skill — the skill teaches Claude **how to write** memories; this CLAUDE.md section tells Claude **that this convention exists** and enforces the hard rules (never mirror to auto-memory, one fact per file, etc.).

**Why both?**
- Skill = triggered on-demand when writing a memory → detailed workflow, templates, common mistakes.
- CLAUDE.md = loaded every session → persistent reminder of the convention + routing rules.

**Installation**: See README.md § Installation for exact steps. This file is the **content to copy**, not a file to place somewhere.

---

## Project Memory

Each project keeps a **shared** memory store at `<repo-root>/memory/` (checked into git so the whole team shares it), using the same file-per-fact + index mechanism as Claude's per-machine auto-memory.

- **Location:** `memory/` at the repo root. It MUST be git-tracked — don't put it under a gitignored path (many repos ignore `.claude/`). Verify with `git check-ignore memory/` (no output ⇒ trackable).
- **File naming:** `YYYY-MM-DD-short-kebab-case-slug.md` (date prefix = when the fact was recorded). Example: `2026-06-23-npz-quaternion-convention.md`.
- **Structure:** one Markdown file per fact, plus an index `memory/MEMORY.md`.
- **Index (`MEMORY.md`):** one bullet per memory — `- [Title](YYYY-MM-DD-slug.md) — hook`. The hook **can be multi-line** (3-5 lines OK) and includes the load-bearing rule itself (the ≤150-char ceiling is auto-memory-only, does NOT apply here). Read it at session start and before working in an area with known pitfalls; open a fact file when its hook is relevant.
- **Each fact file** uses YAML frontmatter + a short body:

  ```yaml
  ---
  name: short-kebab-case-slug
  description: one-line summary — used to decide relevance during recall
  metadata:
    type: decision | pitfall | rule | reference
  ---
  ```

  The fact in a sentence or short paragraph. For decision/pitfall/rule, follow with `**Why:**` (motivation) and `**How to apply:**` (trigger conditions) lines. Link related memories with `[[other-slug]]`.

- **When to write:** after a non-obvious fix, decision, or rule — anything a teammate should know before touching the same code.
- **Hygiene:** one fact per file; update an existing file rather than duplicate; delete stale entries. Don't duplicate what's already in the repo's `CLAUDE.md` (build/run commands, architecture) — `memory/` is for decisions, pitfalls, and rules.
- **Hard rule:** This repo `memory/` is the team source of truth; **never mirror the same facts into a private per-machine auto-memory** (`~/.claude/projects/<...>/memory/`). Pick one location per fact. Mirroring causes drift and contradictory updates.
- **Routing test:** *Would a new hire on this repo need to know this to avoid a regression?* → yes = project memory. Auto-memory is for "about this user / about this machine" facts (role, preferences, single-machine quirks).

---

## Maintenance Notes

**This section was generated from the `kproject-memory` plugin** (see https://github.com/kolamanlee/skills). If you update the plugin, re-copy this section to keep your global CLAUDE.md in sync.

**Differences from auto-memory:**

| | Project memory `<repo>/memory/` | Auto-memory `~/.claude/.../memory/` |
|---|---|---|
| Scope | Team-shared, git-tracked | Per-user, per-machine, untracked |
| Type taxonomy | `decision \| pitfall \| rule \| reference` | `user \| feedback \| project \| reference` |
| Index hook length | Multi-line OK (load-bearing rule inline) | ≤150 chars (one-liner teaser) |
| Body structure | `**Why:**` + `**How to apply:**` required for decision/pitfall/rule | Optional structure |
| When loaded | Session start (via MEMORY.md) + on-demand | Session start (via MEMORY.md) |

**If you also use auto-memory:** keep them strictly separate. Project facts go in `<repo>/memory/`; personal/machine facts go in `~/.claude/.../memory/`. Never duplicate.
