# kproject-memory

A two-part system for **team-shared project memory** that lives in `<repo>/memory/` (git-tracked) instead of Claude Code's default per-user auto-memory (`~/.claude/projects/<...>/memory/`, untracked).

Part of the [kolaman-claude-plugins](../../README.md) marketplace.

**What it does:**
- **Bootstrap mode** — Sets up `memory/` + `MEMORY.md` + CLAUDE.md section in a new project (one command).
- **Write mode** — Guides Claude to write decision/pitfall/rule/reference facts with proper frontmatter, `**Why:**` / `**How to apply:**` sections, and cross-links.
- **Routing** — Enforces the hard rule: project facts go in `<repo>/memory/` (team-shared), personal/machine facts go in auto-memory (per-user). **Never mirror both.**

**Origin:** Extracted from `tonly_robot2/memory/` (28 working fact files over 6 months of RL robot development). Battle-tested conventions.

---

## Installation

Two parts: install the **plugin** (skill), then optionally append the **global CLAUDE.md section** (persistent reminder).

### Part 1: Install the plugin

**From GitHub** (after this repo is pushed):

```bash
/plugin marketplace add kolaman/kolaman-claude-plugins
/plugin install kproject-memory@kolaman-plugins
```

**From local checkout** (for testing before pushing):

```bash
git clone <this-repo-url> ~/kolaman-claude-plugins
# In any Claude Code session:
/plugin marketplace add ~/kolaman-claude-plugins
/plugin install kproject-memory@kolaman-plugins
```

Verify installation:
```bash
# In any Claude Code session:
/skill list
# Should show "kproject-memory" in the list
```

### Part 2: Append the global CLAUDE.md section (recommended)

The skill teaches Claude **how to write** memories; the global CLAUDE.md section tells Claude **that this convention exists** in your projects.

**Why both?**
- **Skill** = triggered on-demand when writing a memory → detailed workflow, templates, common mistakes.
- **CLAUDE.md** = loaded every session → persistent reminder + hard rules (never mirror, one fact per file, routing test).

**Install:**

```bash
# Backup your existing global CLAUDE.md
cp ~/.claude/CLAUDE.md ~/.claude/CLAUDE.md.backup-$(date +%Y%m%d)

# Append the Project Memory section
cat >> ~/.claude/CLAUDE.md << 'EOF'

---

EOF
cat global-claude-md.md >> ~/.claude/CLAUDE.md
```

Or manually: open `~/.claude/CLAUDE.md`, copy the `## Project Memory` section from `global-claude-md.md` (in this plugin directory), paste at the end.

**Verify:**

```bash
grep -c "## Project Memory" ~/.claude/CLAUDE.md
# Should return 1
```

---

## Usage

### Bootstrap a new project

In any project directory (git repo) that doesn't have `memory/` yet:

```
You: 把这个项目记忆机制用到这个项目里
```

Claude will:
1. Verify `memory/` doesn't exist (refuse to clobber if it does).
2. Create `memory/` + `memory/MEMORY.md` (from `templates.md` § `bootstrap-header`).
3. Check `git check-ignore memory/` (warn if gitignored).
4. Append `## Project Memory` section to `CLAUDE.md` (if missing).
5. Report what was created.

### Write a memory

After fixing a bug, making a decision, or discovering a pitfall:

```
You: 记住这个事：process_motor_ 没给 velocity 乘 direction，NEG 极性关节喂策略的 joint_vel 反号了
```

Claude will:
1. Route to `<repo>/memory/` (not auto-memory).
2. Create `memory/YYYY-MM-DD-process-motor-direction-sign.md` with proper frontmatter (`type: pitfall`).
3. Body includes `**Why:**` + `**How to apply:**` + cross-links `[[other-slug]]`.
4. Append bullet to `memory/MEMORY.md` with multi-line hook (load-bearing rule inline).

### Read memories

At session start, Claude reads `memory/MEMORY.md` (loaded into context). When working in an area with known pitfalls, open the specific fact file.

---

## File Structure

```
plugins/kproject-memory/
  .claude-plugin/
    plugin.json              # Plugin manifest
  skills/
    kproject-memory/
      SKILL.md               # Main skill doc (<500 words)
      templates.md           # bootstrap-header + fact-file-skeleton
  global-claude-md.md        # Content to append to ~/.claude/CLAUDE.md
  README.md                  # This file
```

---

## Uninstall

```bash
/plugin uninstall kproject-memory@kolaman-plugins
/plugin marketplace remove kolaman-plugins

# Remove the CLAUDE.md section manually:
# Open ~/.claude/CLAUDE.md, delete the "## Project Memory" section
```

---

## FAQ

**Q: Do I need both the plugin AND the CLAUDE.md section?**

A: Plugin is required (contains the skill). CLAUDE.md section is **strongly recommended** — without it, Claude won't know the convention exists until it triggers the skill (which requires recognizing the task). With it, Claude sees the routing rules every session.

**Q: Can I use this alongside auto-memory?**

A: Yes, but keep them **strictly separate**. Project facts → `<repo>/memory/`. Personal/machine facts → `~/.claude/.../memory/`. The skill enforces this with a routing test: *"Would a new hire need to know this?"* → yes = project memory.

**Q: What if my project already has a `memory/` directory?**

A: Bootstrap mode refuses to clobber. If `memory/` exists but uses a different convention, migrate manually or adapt the skill's templates to match.

**Q: Can I customize the templates?**

A: Yes. Edit `plugins/kproject-memory/skills/kproject-memory/templates.md` in this repo. Reinstall the plugin to pick up changes (`/plugin update kproject-memory@kolaman-plugins`).

**Q: Does this work with monorepos / submodules?**

A: Convention is one `memory/` per repo root. Monorepo subprojects can each have their own `memory/` if independently versioned, but that's untested. Flag for future iteration.

**Q: Why not just use auto-memory for everything?**

A: Auto-memory is per-user, per-machine, untracked. Project memory is team-shared, git-tracked, follows the repo. If you're working solo on one machine forever, auto-memory is fine. If you have teammates, work across machines, or want facts versioned with code → project memory.

---

## Origin & References

- **Origin project:** `tonly_robot2/memory/` — 28 fact files over 6 months of RL robot development (decision/pitfall/rule/reference taxonomy, multi-line hooks, cross-link conventions all battle-tested).
- **Design doc:** See `~/.claude/plans/ancient-snacking-sparkle.md` (in origin author's setup) for full design rationale.
- **Claude Code docs:** https://docs.claude.com/en/docs/claude-code/skills

---

## License

MIT (or your preferred license — update this line before publishing)
