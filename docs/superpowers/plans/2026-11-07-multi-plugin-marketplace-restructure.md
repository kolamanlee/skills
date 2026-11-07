# kolaman-claude-plugins 多插件化改造 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 把仓库根从"marketplace = 唯一插件"改造为纯 marketplace + `plugins/` 多插件结构，使新增技能插件成为标准化三步操作。

**Architecture:** 仓库根只保留 `.claude-plugin/marketplace.json`（插件目录清单）；每个技能插件是 `plugins/<name>/` 下的自包含目录（自己的 `plugin.json` + `skills/` + 附属文件 + README）。本改造只做文件搬家 + 清单更新 + 文档重组，技能本体零改动。

**Tech Stack:** bash、git（已初始化，`main` 分支，首笔提交 `9a295a1` 为改造前快照）、`jq`（JSON 校验）。

**Spec:** `docs/superpowers/specs/2026-11-07-multi-plugin-marketplace-restructure-design.md`

## Global Constraints

- **内容冻结**：`SKILL.md`、`templates.md`、`plugin.json`、`global-claude-md.md` 四个文件内容一字不动，只允许 `git mv` 改变路径。
- **兼容性承诺**：安装命令 `/plugin install kproject-memory@kolaman-plugins` 语义不变。
- **拒绝覆盖**：`plugins/` 已存在任何内容时立即报错停手，不做任何覆盖。
- `marketplace.json` 中除 `plugins[0].source` 外的字段不动。
- 提交信息以 `Co-Authored-By: Claude Code <noreply@anthropic.com>` 结尾。
- 文档语言：沿用现状——结构/说明英文为主，示例与指南可中英混排。

---

### Task 1: 搬家 — kproject-memory 迁入 plugins/ 并更新 marketplace 清单

**Files:**
- Create（目录）: `plugins/kproject-memory/`
- Move: `.claude-plugin/plugin.json` → `plugins/kproject-memory/.claude-plugin/plugin.json`
- Move: `skills/`（含 `skills/kproject-memory/`）→ `plugins/kproject-memory/skills/`
- Move: `global-claude-md.md` → `plugins/kproject-memory/global-claude-md.md`
- Modify: `.claude-plugin/marketplace.json`（仅 `plugins[0].source`）

**Interfaces:**
- Consumes: 现有仓库布局（git 首笔提交 `9a295a1` 的状态）。
- Produces: Task 2/3 依赖的稳定路径——`plugins/kproject-memory/README.md`（Task 2 写入）、`plugins/kproject-memory/skills/kproject-memory/{SKILL.md,templates.md}`、`plugins/kproject-memory/global-claude-md.md`、`plugins/kproject-memory/.claude-plugin/plugin.json`。

- [ ] **Step 1: 前置守卫 — 确认没有冲突目标**

```bash
cd /home/nvidia/projects/utils/kolaman-claude-plugins
test ! -e plugins && echo "OK: plugins/ 不存在，可以开始" || { echo "ABORT: plugins/ 已存在"; exit 1; }
git status --porcelain  # 应为空（工作区干净）
```

Expected: 输出 `OK: plugins/ 不存在，可以开始`；`git status --porcelain` 无输出。

- [ ] **Step 2: 用 git mv 搬家（保留历史）**

```bash
mkdir -p plugins/kproject-memory/.claude-plugin
git mv skills plugins/kproject-memory/skills
git mv global-claude-md.md plugins/kproject-memory/global-claude-md.md
git mv .claude-plugin/plugin.json plugins/kproject-memory/.claude-plugin/plugin.json
```

Expected: 三条 `git mv` 均无报错；`git status --porcelain` 显示 R（重命名）记录。

- [ ] **Step 3: 更新 marketplace.json 的 source**

把 `.claude-plugin/marketplace.json` 改为（仅 `source` 变化，全文如下）：

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "kolaman-plugins",
  "description": "Personal plugin marketplace — kolaman's reusable Claude Code plugins",
  "owner": {
    "name": "kolaman",
    "email": "kolaman@users.noreply.github.com"
  },
  "plugins": [
    {
      "name": "kproject-memory",
      "description": "Team-shared, git-tracked project memory: bootstrap + write guidance. Distinct from Claude Code's per-user auto-memory.",
      "source": "./plugins/kproject-memory",
      "category": "productivity"
    }
  ]
}
```

- [ ] **Step 4: 结构验证**

```bash
cd /home/nvidia/projects/utils/kolaman-claude-plugins
jq -e '.plugins[0].source == "./plugins/kproject-memory"' .claude-plugin/marketplace.json
jq -e '.name == "kproject-memory"' plugins/kproject-memory/.claude-plugin/plugin.json
test -f plugins/kproject-memory/skills/kproject-memory/SKILL.md
test -f plugins/kproject-memory/skills/kproject-memory/templates.md
test -f plugins/kproject-memory/global-claude-md.md
test "$(ls .claude-plugin)" = "marketplace.json" && echo "根 .claude-plugin/ 只剩 marketplace.json"
test ! -e skills && test ! -e global-claude-md.md && echo "根下旧路径已清空"
```

Expected: 两条 `jq` 输出 `true`；末尾两行 echo 正常打印；无任何 `test` 报错。

- [ ] **Step 5: 提交**

```bash
git add -A
git commit -m "$(cat <<'EOF'
refactor: move kproject-memory into plugins/ as an independent plugin

Repo root is now a pure marketplace; kproject-memory lives in
plugins/kproject-memory/ with its own manifest, skills, aux files.
marketplace.json now points at ./plugins/kproject-memory.
Skill contents unchanged — moves only. Install command unchanged:
/plugin install kproject-memory@kolaman-plugins

Co-Authored-By: Claude Code <noreply@anthropic.com>
EOF
)"
```

---

### Task 2: kproject-memory 插件级 README

**Files:**
- Create: `plugins/kproject-memory/README.md`

**Interfaces:**
- Consumes: Task 1 产出的路径（`plugins/kproject-memory/...`）；原根 README 的 kproject-memory 章节（内容素材在下方全文给出，无需回读旧文件）。
- Produces: `plugins/kproject-memory/README.md` — Task 3 根 README 的插件表格将链接到它（`plugins/kproject-memory/`）。

- [ ] **Step 1: 写入插件 README（全文）**

创建 `plugins/kproject-memory/README.md`，内容如下（两处相对原文的修正已包含：文件结构图、本地安装方式；FAQ 中 templates 路径同步更新）：

````markdown
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
````

- [ ] **Step 2: 验证无旧路径残留 + 链接有效**

```bash
cd /home/nvidia/projects/utils/kolaman-claude-plugins/plugins/kproject-memory
grep -n "enabledPlugins" README.md && echo "FAIL: 旧本地安装写法残留" || echo "OK: 无 enabledPlugins 旧写法"
test -f global-claude-md.md && test -f ../../README.md && echo "OK: 相对链接目标存在"
```

Expected: 两条 `OK`。

- [ ] **Step 3: 提交**

```bash
git add plugins/kproject-memory/README.md
git commit -m "$(cat <<'EOF'
docs: add plugin-level README for kproject-memory

Full usage/FAQ carried over from the old root README, with two fixes:
updated file-structure diagram, and local-testing now via
/plugin marketplace add <local path> (root is no longer a plugin,
the old enabledPlugins path method no longer works).

Co-Authored-By: Claude Code <noreply@anthropic.com>
EOF
)"
```

---

### Task 3: 重写根 README — marketplace 总览 + 新增插件指南

**Files:**
- Modify: `README.md`（整文件替换）

**Interfaces:**
- Consumes: Task 1 的路径布局、Task 2 的 `plugins/kproject-memory/README.md`（链接目标）。
- Produces: marketplace 级 README——未来新插件的登记处与"新增插件三步走"指南。

- [ ] **Step 1: 整体替换根 README（全文）**

````markdown
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
````

- [ ] **Step 2: 验证链接与残留**

```bash
cd /home/nvidia/projects/utils/kolaman-claude-plugins
test -f plugins/kproject-memory/README.md && echo "OK: 插件表格链接有效"
grep -n "enabledPlugins\|source.*\"\.\"" README.md && echo "FAIL: 有旧写法残留" || echo "OK: 无旧写法"
```

Expected: 两条 `OK`。

- [ ] **Step 3: 提交**

```bash
git add README.md
git commit -m "$(cat <<'EOF'
docs: rewrite root README as marketplace overview + add-plugin guide

Root README now covers: plugin catalog table, install/uninstall,
repo layout, and the 3-step "adding a new plugin" guide.
Plugin-specific docs live in plugins/<name>/README.md.

Co-Authored-By: Claude Code <noreply@anthropic.com>
EOF
)"
```

---

### Task 4: 终验（对照设计的验证清单）

**Files:**
- 无新建/修改（纯验证）。若任何一步失败，回到对应 Task 修复后重跑本任务。

**Interfaces:**
- Consumes: Task 1–3 的全部产出。
- Produces: 验证通过的最终状态 + 给用户的人工验证命令。

- [ ] **Step 1: 自动验证全清单**

```bash
cd /home/nvidia/projects/utils/kolaman-claude-plugins
set -e
jq -e '.plugins | length == 1' .claude-plugin/marketplace.json > /dev/null
jq -e '.plugins[0].source == "./plugins/kproject-memory"' .claude-plugin/marketplace.json > /dev/null
jq -e '.name == "kproject-memory" and .version == "0.1.0"' plugins/kproject-memory/.claude-plugin/plugin.json > /dev/null
test "$(ls .claude-plugin)" = "marketplace.json"
test ! -e skills && test ! -e global-claude-md.md && test ! -e .claude-plugin/plugin.json
test -f plugins/kproject-memory/skills/kproject-memory/SKILL.md
test -f plugins/kproject-memory/skills/kproject-memory/templates.md
test -f plugins/kproject-memory/global-claude-md.md
test -f plugins/kproject-memory/README.md
! grep -rn "enabledPlugins" README.md plugins/
test -z "$(git status --porcelain)" && echo "OK: 工作区干净"
echo "=== ALL CHECKS PASSED ==="
```

Expected: 最后打印 `=== ALL CHECKS PASSED ===`。

- [ ] **Step 2: 确认提交历史符合设计**

```bash
git log --oneline
```

Expected: 首笔为 `9a295a1 chore: preserve pre-restructure snapshot + multi-plugin design doc`，其后依次为 Task 1–3 的三笔提交（refactor → docs × 2）。

- [ ] **Step 3: 输出用户人工验证命令（不改文件）**

向用户报告以下命令，请其在**新的 Claude Code 会话**中执行：

```
/plugin marketplace add ~/projects/utils/kolaman-claude-plugins
/plugin install kproject-memory@kolaman-plugins
/skill list        # 应能看到 kproject-memory
```

---
