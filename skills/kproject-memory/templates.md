# kproject-memory templates

Two copy-paste artifacts: § `bootstrap-header` for Mode A, § `fact-file-skeleton` for Mode B.

---

## bootstrap-header

Copy this verbatim into `<repo>/memory/MEMORY.md` during Mode A bootstrap. Then leave one example bullet (commented or live) so the structure is self-documenting.

```markdown
# Memory Index

Team-shared, git-tracked. One bullet per fact. Hook 可以多行（3-5 行 OK），
包含 load-bearing 规则本身（不受 auto-memory 的 ≤150-char 约束）。

打开方式：session 启动时 + 触及有 pitfall 的区域前，先读这个文件，
按 hook 提示打开对应的事实文件全文。

- [Example fact](2026-01-01-example-slug.md) — 一行 hook 说明这是什么事实、什么时候触发、load-bearing 规则是什么。
```

After bootstrap, the directory layout is:

```
<repo>/
  memory/
    MEMORY.md          # this index
    YYYY-MM-DD-slug.md # one file per fact (created on demand)
  CLAUDE.md            # appended with "## Project Memory" section
```

---

## fact-file-skeleton

Copy this skeleton when writing a new fact in Mode B. Replace EVERY placeholder (`<...>`) — don't leave any in the committed file.

```markdown
---
name: <kebab-case-slug-without-date>
description: <one-line summary — used for relevance recall at session start; include the symptom/trigger so future-you can decide whether to open the full file>
metadata:
  type: <decision | pitfall | rule | reference>
---

# <Title>

<1-2 段陈述事实本身。第一句应该是结论 / load-bearing 规则；后续段落补充上下文。
不要写叙事（"我们曾经遇到 X 然后..."），写断言（"X 必须如此因为..."）。>

**Why:** <动机 — 过往事故 / 物理约束 / deadline / stakeholder 要求 / 为什么不能改。
这一段回答"将来有人想'优化'掉这个规则时，凭什么阻止他"。>

**How to apply:**
- <触发场景 1：什么情况下这个规则生效>
- <触发场景 2：什么操作前要查这条>
- <边界 case：什么时候这个规则不适用>

相关：[[<other-slug>]]、[[<another-slug>]]。
```

### Frontmatter field rules

| Field | Rule |
|---|---|
| `name` | kebab-case, NO date prefix, NO spaces. Matches the slug part of the filename (`YYYY-MM-DD-<slug>.md`). Other memories cross-link to this via `[[<name>]]`. |
| `description` | One line. Used for recall at session start (loaded into MEMORY.md context). Should contain the **trigger** ("when X happens, ...") not just the topic. Mixed Chinese/English OK. |
| `metadata.type` | Exactly one of `decision` (architecture/API choice), `pitfall` (bug/gotcha with non-obvious fix), `rule` (workflow/process rule), `reference` (pointer to external system). **NOT** auto-memory's `user\|feedback\|project\|reference`. |

### Body section rules

| Section | Required? | What goes here |
|---|---|---|
| Title (`# ...`) | yes | Short, scannable. Not the same as `description` — title is the headline, description is the recall hook. |
| Lead paragraphs | yes | The fact itself. Lead with the conclusion. 1-2 paragraphs. |
| `**Why:**` | yes for `decision`/`pitfall`/`rule` | The motivation. Past incident, physical constraint, deadline, stakeholder ask. Future-proof against "let me just simplify this" regressions. Optional for `reference`. |
| `**How to apply:**` | yes for `pitfall`/`rule` | Trigger conditions. When does this kick in? What should the reader check before touching the related code? Bulleted list. |
| `相关：[[...]]` | encouraged | Cross-links to other memories. Slug = other memory's `name`. Dangling links (pointing to a not-yet-written memory) are OK — they mark "worth writing later". |

### MEMORY.md bullet rule

After writing the fact file, append ONE bullet to `memory/MEMORY.md`:

```markdown
- [<Short title>](YYYY-MM-DD-<slug>.md) — <hook>
```

The `<hook>` can be **multi-line** (3-5 lines OK). It should contain:
- WHAT the fact is (one phrase)
- WHEN it triggers (the symptom / code area)
- The LOAD-BEARING RULE itself (so a future reader can decide whether to open the full file from the index alone)

**The ≤150-char ceiling from auto-memory does NOT apply here.** Long hooks are intentional — the index is the team's first-aid reference, not a TOC teaser.
