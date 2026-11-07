# 设计：kolaman-claude-plugins 多插件化改造

日期：2026-11-07
状态：已确认（用户批准方案 A）

## 背景与问题

仓库当前只含一个技能插件 `kproject-memory`，但仓库根**身兼两职**：

- 根 `.claude-plugin/marketplace.json` 声明 marketplace，其中 `plugins[0].source: "."` 指向仓库根；
- 根 `.claude-plugin/plugin.json` 又把仓库根本身定义为 `kproject-memory` 插件。

这个"根 = marketplace = 唯一插件"的模式只在恰好一个插件时成立：根 `plugin.json` 只能描述一个插件，`skills/` 下所有目录都会归入同一个插件，无法新增第二个插件。附属文件（`global-claude-md.md`）也只能堆在根上。

## 目标

把仓库改造成可扩展的 skills 插件项目空间：仓库根是纯 marketplace，每个技能是一个独立插件（独立安装 / 卸载 / 版本），新增技能插件的成本低且步骤明确。用户已确认：未来扩展以**纯 skills** 为主。

## 方案（已选 A：marketplace + plugins/ 目录）

### 目标布局

```
kolaman-claude-plugins/                 # 仓库根 = 纯 marketplace
├── .claude-plugin/
│   └── marketplace.json                # 每个插件一条记录
├── plugins/
│   └── kproject-memory/
│       ├── .claude-plugin/
│       │   └── plugin.json             # 原根 plugin.json，内容不变
│       ├── skills/
│       │   └── kproject-memory/
│       │       ├── SKILL.md            # 原样搬
│       │       └── templates.md        # 原样搬
│       ├── global-claude-md.md         # 原样搬（归插件自己）
│       └── README.md                   # 新文件：kproject-memory 完整说明（自根 README 拆出）
├── README.md                           # 重写：marketplace 总览 + 新增插件指南
└── docs/superpowers/specs/             # 本设计文档所在
```

未选方案 B（单一大插件打包所有 skills）：无法按需安装单个技能、插件名与技能名脱钩会破坏已发布的安装命令、附属文件仍堆根目录、未来混入 commands/hooks 会绑死所有技能。

### 具体变更

| 对象 | 动作 |
|---|---|
| `marketplace.json` | `plugins[0].source`: `"."` → `"./plugins/kproject-memory"`，其余字段不动 |
| 根 `plugin.json` | 移动到 `plugins/kproject-memory/.claude-plugin/plugin.json`，内容不变（版本 0.1.0） |
| 根 `skills/` | 整体移动到 `plugins/kproject-memory/skills/` |
| `global-claude-md.md` | 移动到 `plugins/kproject-memory/`，内容不动 |
| 根 `README.md` | 重写为 marketplace 级：插件清单表格、通用安装方式、**新增插件三步走**（① `plugins/<name>/.claude-plugin/plugin.json` ② `plugins/<name>/skills/<name>/SKILL.md` ③ `marketplace.json` 加一条）、本地开发测试方式 |
| 新 `plugins/kproject-memory/README.md` | 承接原 README 的 kproject-memory 全部内容（安装/用法/FAQ），仅两处修正：文件结构图更新为新布局；本地测试由 `enabledPlugins` 路径改为 `/plugin marketplace add <本地路径>`（根不再是插件，旧写法失效） |
| git | 先 `git init` + 提交原始结构快照（含本设计文档），再重构、再提交 |

### 兼容性承诺（不变的的东西）

- 已发布/已写入文档的安装命令 `/plugin install kproject-memory@kolaman-plugins` 语义完全不变。
- `SKILL.md`、`templates.md`、`plugin.json` 内容一字不动——纯搬家 + 文档重组，技能本体零改动。
- `global-claude-md.md` 内容不动（其中指向 GitHub 仓库的链接仍然有效）。

## 边界处理

- `plugins/` 或任一目标路径已存在非预期内容 → 报错停手，绝不覆盖。
- 重构完成后用 `jq` 校验两个 JSON 均可解析，且 `source` 指向的路径存在。
- 根 README 中指向仓库内文件的相对链接全部随搬家更新，不留死链。

## 验证

1. 自动：`jq .` 两个 JSON 通过；`test -f plugins/kproject-memory/skills/kproject-memory/SKILL.md` 通过；根 `.claude-plugin/` 下只剩 `marketplace.json`，根下不再有 `skills/`、`global-claude-md.md`。
2. 人工（需用户配合，新 Claude Code 会话）：
   `/plugin marketplace add ~/projects/utils/kolaman-claude-plugins` →
   `/plugin install kproject-memory@kolaman-plugins` → 确认技能可列出。
3. `git log`：首笔为改造前快照（`9a295a1`），其后为重构提交（按任务细粒度提交）。

## 范围外

- 不新增 LICENSE 文件（根 README 版权行维持现状，发布前再定）。
- 不改动技能本体的任何行为/文案。
- 不处理 monorepo / 多 marketplace 分发等未来话题。
