# 自定义

OpenSpec 提供三个级别的自定义：

| 级别 | 功能 | 适用于 |
|-------|--------------|----------|
| **项目配置** | 设置默认值、注入上下文/规则 | 大多数团队 |
| **自定义 Schema** | 定义你自己的工作流产物 | 有独特流程的团队 |
| **全局覆盖** | 跨项目共享 schema | 高级用户 |

---

## 项目配置

`openspec/config.yaml` 文件是为你团队自定义 OpenSpec 的最简单方式。它让你：

- **设置默认 schema** - 跳过每个命令的 `--schema`
- **注入项目上下文** - AI 看到你的技术栈、约定等
- **添加每产物规则** - 特定产物的自定义规则

### 快速设置

```bash
openspec init
```

这会引导你交互式创建配置。或者手动创建一个：

```yaml
# openspec/config.yaml
schema: spec-driven

context: |
  技术栈：TypeScript、React、Node.js、PostgreSQL
  API 风格：RESTful，在 docs/api.md 有文档
  测试：Jest + React Testing Library
  我们重视所有公共 API 的向后兼容性

rules:
  proposal:
    - 包含回滚计划
    - 识别受影响的团队
  specs:
    - 使用 Given/When/Then 格式
    - 在发明新模式之前引用现有模式
```

### 工作原理

**默认 schema：**

```bash
# 没有配置
openspec new change my-feature --schema spec-driven

# 有配置 - schema 自动
openspec new change my-feature
```

**上下文和规则注入：**

生成任何产物时，你的上下文和规则被注入到 AI 提示中：

```xml
<context>
技术栈：TypeScript、React、Node.js、PostgreSQL
...
</context>

<rules>
- 包含回滚计划
- 识别受影响的团队
</rules>

<template>
[Schema 内置模板]
</template>
```

- **上下文** 出现在所有产物中
- **规则** 仅出现在匹配的产物中

### Schema 解析顺序

当 OpenSpec 需要 schema 时，它按这个顺序检查：

1. CLI 标志：`--schema <name>`
2. 变更元数据（变更文件夹中的 `.openspec.yaml`）
3. 项目配置（`openspec/config.yaml`）
4. 默认值（`spec-driven`）

---

## 自定义 Schema

当项目配置不够时，创建你自己的 schema，具有完全自定义的工作流。自定义 schema 住在你项目的 `openspec/schemas/` 目录中，并与你的代码版本控制。

```text
your-project/
├── openspec/
│   ├── config.yaml        # 项目配置
│   ├── schemas/           # 自定义 schema 在这里
│   │   └── my-workflow/
│   │       ├── schema.yaml
│   │       └── templates/
│   └── changes/           # 你的变更
└── src/
```

### Fork 现有 Schema

最快的方式是 fork 一个内置 schema：

```bash
openspec schema fork spec-driven my-workflow
```

这将整个 `spec-driven` schema 复制到 `openspec/schemas/my-workflow/`，你可以自由编辑。

**你得到的是：**

```text
openspec/schemas/my-workflow/
├── schema.yaml           # 工作流定义
└── templates/
    ├── proposal.md       # proposal 产物的模板
    ├── spec.md           # specs 的模板
    ├── design.md         # design 的模板
    └── tasks.md          # tasks 的模板
```

现在编辑 `schema.yaml` 改变工作流，或编辑模板改变 AI 生成的内容。

### 从头创建 Schema

对于完全新鲜的工作流：

```bash
# 交互式
openspec schema init research-first

# 非交互式
openspec schema init rapid \
  --description "快速迭代工作流" \
  --artifacts "proposal,tasks" \
  --default
```

### Schema 结构

schema 定义工作流中的产物及其依赖：

```yaml
# openspec/schemas/my-workflow/schema.yaml
name: my-workflow
version: 1
description: 我团队的自定义工作流

artifacts:
  - id: proposal
    generates: proposal.md
    description: 初始提案文档
    template: proposal.md
    instruction: |
      创建一个解释为什么需要这个变更的提案。
      专注于问题，而不是解决方案。
    requires: []

  - id: design
    generates: design.md
    description: 技术设计
    template: design.md
    instruction: |
      创建一个解释如何实施的设计文档。
    requires:
      - proposal    # 在 proposal 存在之前不能创建 design

  - id: tasks
    generates: tasks.md
    description: 实施检查清单
    template: tasks.md
    requires:
      - design

apply:
  requires: [tasks]
  tracks: tasks.md
```

**关键字段：**

| 字段 | 用途 |
|-------|---------|
| `id` | 唯一标识符，用于命令和规则 |
| `generates` | 输出文件名（支持 glob 如 `specs/**/*.md`） |
| `template` | `templates/` 目录中的模板文件 |
| `instruction` | 创建这个产物的 AI 指令 |
| `requires` | 依赖 — 哪些产物必须先存在 |

### 模板

模板是 markdown 文件，在创建该产物时注入到提示中。

```markdown
<!-- templates/proposal.md -->
## 为什么

<!-- 解释这个变更的动机。解决什么问题？ -->

## 什么变更

<!-- 描述将要变更什么。具体说明新能力或修改。 -->

## 影响

<!-- 受影响的代码、API、依赖、系统 -->
```

模板可以包含：
- AI 应填写的小节标题
- AI 的 HTML 注释指导
- 显示预期结构的示例格式

### 验证你的 Schema

使用自定义 schema 之前，验证它：

```bash
openspec schema validate my-workflow
```

这检查：
- `schema.yaml` 语法正确
- 所有引用的模板存在
- 没有循环依赖
- 产物 ID 有效

### 使用你的自定义 Schema

创建后，用你的 schema：

```bash
# 在命令上指定
openspec new change feature --schema my-workflow

# 或在 config.yaml 中设置为默认值
schema: my-workflow
```

### 调试 Schema 解析

不确定使用哪个 schema？检查：

```bash
# 查看特定 schema 从哪里解析
openspec schema which my-workflow

# 列出所有可用 schema
openspec schema which --all
```

输出显示它是来自你的项目、用户目录还是包：

```text
Schema: my-workflow
Source: project
Path: /path/to/project/openspec/schemas/my-workflow
```

---

> **注意：** OpenSpec 也支持用户级 schema 在 `~/.local/share/openspec/schemas/`，用于跨项目共享，但项目级 schema 在 `openspec/schemas/` 中被推荐，因为它们与你的代码版本控制。

---

## 示例

### 快速迭代工作流

一个最小化开销的快速迭代工作流：

```yaml
# openspec/schemas/rapid/schema.yaml
name: rapid
version: 1
description: 最小化开销的快速迭代

artifacts:
  - id: proposal
    generates: proposal.md
    description: 快速提案
    template: proposal.md
    instruction: |
      为这个变更创建一个简洁的提案。
      专注于是什么和为什么，跳过详细规格。
    requires: []

  - id: tasks
    generates: tasks.md
    description: 实施检查清单
    template: tasks.md
    requires: [proposal]

apply:
  requires: [tasks]
  tracks: tasks.md
```

### 添加审查产物

Fork 默认的并添加审查步骤：

```bash
openspec schema fork spec-driven with-review
```

然后编辑 `schema.yaml` 添加：

```yaml
  - id: review
    generates: review.md
    description: 实施前审查检查清单
    template: review.md
    instruction: |
      基于设计创建审查检查清单。
      包含安全性、性能和测试考虑。
    requires:
      - design

  - id: tasks
    # ... existing tasks config ...
    requires:
      - specs
      - design
      - review    # 现在 tasks 也需要 review
```

---

## 另见

- [CLI 参考：Schema 命令](cli.md#schema-commands) — 完整的命令文档