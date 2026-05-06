# 迁移到 OPSX

本指南帮助你从旧版 OpenSpec 工作流迁移到 OPSX。迁移设计为平滑的 — 你现有工作会被保留，新系统提供更多灵活性。

## 有什么变化？

OPSX 用流动的、基于动作的方法替代旧的阶段锁定工作流。关键转变如下：

| 方面 | 旧版 | OPSX |
|--------|--------|------|
| **命令** | `/openspec:proposal`、`/openspec:apply`、`/openspec:archive` | 默认：`/opsx:propose`、`/opsx:apply`、`/opsx:sync`、`/opsx:archive`（扩展工作流命令可选） |
| **工作流** | 一次创建所有产物 | 增量或一次创建 — 由你选择 |
| **返回** | 尴尬的阶段门 | 自然 — 随时更新任何产物 |
| **自定义** | 固定结构 | Schema 驱动，完全可 hack |
| **配置** | `CLAUDE.md` 标记 + `project.md` | `openspec/config.yaml` 中的清晰配置 |

**理念改变：** 工作不是线性的。OPSX 停止假装它是。

---

## 开始之前

### 你现有工作是安全的

迁移过程设计为保留：

- **`openspec/changes/` 中的活动变更** — 完全保留。你可以用 OPSX 命令继续。
- **归档的变更** — 原样保留。你的历史保持完整。
- **`openspec/specs/` 中的主规格** — 原样保留。这是你的事实来源。
- **你在 `CLAUDE.md`、`AGENTS.md` 等中的内容** — 保留。只有 OpenSpec 标记块被删除；你写的内容保持不变。

### 什么被删除

只有被替换的 OpenSpec 管理的文件：

| 内容 | 为什么 |
|------|-----|
| 旧版斜杠命令目录/文件 | 被新的技能系统取代 |
| `openspec/AGENTS.md` | 过时的工作流触发器 |
| `CLAUDE.md`、`AGENTS.md` 等中的 OpenSpec 标记 | 不再需要 |

**各工具的旧版命令位置**（示例 — 你的工具可能不同）：

- Claude Code: `.claude/commands/openspec/`
- Cursor: `.cursor/commands/openspec-*.md`
- Windsurf: `.windsurf/workflows/openspec-*.md`
- Cline: `.clinerules/workflows/openspec-*.md`
- Roo: `.roo/commands/openspec-*.md`
- GitHub Copilot: `.github/prompts/openspec-*.prompt.md`（仅 IDE 扩展；Copilot CLI 不支持）

以及其他（Augment、Continue、Amazon Q 等）。

迁移检测你配置的工具并清理它们的旧版文件。

删除列表看起来可能很长，但这些都是 OpenSpec 最初创建的文件。你自己的内容永远不会被删除。

### 需要你注意的事项

一个文件需要手动迁移：

**`openspec/project.md`** — 此文件不会自动删除，因为它可能包含你写的项目上下文。你需要：

1. 查看其内容
2. 将有用内容移至 `openspec/config.yaml`（见下文指导）
3. 准备好时删除该文件

**为什么我们做出这个改变：**

旧的 `project.md` 是被动的 — 代理可能读取它，可能不读取，可能忘记他们读取了什么。我们发现可靠性不一致。

新的 `config.yaml` 上下文**被主动注入到每个 OpenSpec 规划请求中**。这意味着你的项目约定、技术栈和规则在 AI 创建产物时始终存在。更高的可靠性。

**权衡：**

因为上下文被注入到每个请求中，你会希望它简洁。专注于真正重要的内容：
- 技术栈和关键约定
- 代理需要知道的非显而易见的约束
- 以前经常被忽略的规则

不要担心做得完美。我们仍在学习什么最有效，我们将随着实验改进上下文注入的工作方式。

---

## 运行迁移

`openspec init` 和 `openspec update` 都会检测旧版文件并引导你完成相同的清理过程。使用适合你情况的任何一个：

- 新安装默认为配置文件 `core`（`propose`、`explore`、`apply`、`sync`、`archive`）。
- 迁移的安装通过在需要时写入 `custom` 配置文件来保留你之前安装的工作流。

### 使用 `openspec init`

如果你想添加新工具或重新配置哪些工具被设置，运行这个：

```bash
openspec init
```

init 命令检测旧版文件并引导你完成清理：

```
升级到新的 OpenSpec

OpenSpec 现在使用代理技能，这是跨编码代理的新兴标准。
这简化了你的设置，同时保持一切像以前一样工作。

要删除的文件
无需保留的用户内容：
  • .claude/commands/openspec/
  • openspec/AGENTS.md

要更新的文件
OpenSpec 标记将被删除，你的内容保留：
  • CLAUDE.md
  • AGENTS.md

需要你注意
  • openspec/project.md
    我们不会删除此文件。它可能包含有用的项目上下文。

    新的 openspec/config.yaml 有一个 "context:" 部分用于规划
    上下文。这包含在每个 OpenSpec 请求中，比旧的
    project.md 方法更可靠地工作。

    查看 project.md，将任何有用内容移至 config.yaml 的
    context 部分，准备好时删除该文件。

? 升级并清理旧版文件？（Y/n）
```

**当你说是时会发生什么：**

1. 删除旧版斜杠命令目录
2. 从 `CLAUDE.md`、`AGENTS.md` 等中剥离 OpenSpec 标记（你的内容保留）
3. 删除 `openspec/AGENTS.md`
4. 在 `.claude/skills/` 中安装新技能
5. 用默认 schema 创建 `openspec/config.yaml`

### 使用 `openspec update`

如果你只想迁移并刷新你现有工具到最新版本：

```bash
openspec update
```

update 命令也检测并清理旧版产物，然后刷新生成的技能/命令以匹配你当前的配置文件和交付设置。

### 非交互式 / CI 环境

对于脚本化迁移：

```bash
openspec init --force --tools claude
```

`--force` 标志跳过提示并自动接受清理。

---

## 将 project.md 迁移到 config.yaml

旧的 `openspec/project.md` 是一个用于项目上下文的自由格式 markdown 文件。新的 `openspec/config.yaml` 是结构化的 — 关键是通过**注入到每个规划请求**，因此你的约定在 AI 工作时始终存在。

### 之前（project.md）

```markdown
# 项目上下文

这是一个使用 React 和 Node.js 的 TypeScript monorepo。
我们使用 Jest 进行测试，遵循严格的 ESLint 规则。
我们的 API 是 RESTful 的，在 docs/api.md 中有文档。

## 约定

- 所有公共 API 必须保持向后兼容
- 新功能应包含测试
- 规格使用 Given/When/Then 格式
```

### 之后（config.yaml）

```yaml
schema: spec-driven

context: |
  技术栈：TypeScript、React、Node.js
  测试：Jest + React Testing Library
  API：RESTful，在 docs/api.md 有文档
  我们为所有公共 API 保持向后兼容

rules:
  proposal:
    - 为风险变更包含回滚计划
  specs:
    - 使用 Given/When/Then 格式编写场景
    - 引用现有模式再发明新模式之前
  design:
    - 复杂流程包含序列图
```

### 关键差异

| project.md | config.yaml |
|------------|-------------|
| 自由格式 markdown | 结构化 YAML |
| 一大块文本 | 分开的 context 和每产物规则 |
| 不清楚何时使用 | 上下文出现在所有产物中；规则仅出现在匹配的产物中 |
| 无 schema 选择 | 明确的 `schema:` 字段设置默认工作流 |

### 保留什么、删除什么

迁移时要有选择性。问自己："AI 需要这个来*每个*规划请求吗？"

**适合 `context:` 的好东西**
- 技术栈（语言、框架、数据库）
- 关键架构模式（monorepo、微服务等）
- 非显而易见的约束（"我们不能使用库 X 因为..."）
- 经常被忽略的关键约定

**改为移到 `rules:`**
- 特定于产物的格式（"在规格中使用 Given/When/Then"）
- 审查标准（"提案必须包含回滚计划"）
- 这些只出现在匹配的产物中，保持其他请求更轻

**完全省略**
- AI 模型已经知道的通用最佳实践
- 可以总结的冗长解释
- 不影响当前工作的历史上下文

### 迁移步骤

1. **创建 config.yaml**（如果 init 还没有创建）：
   ```yaml
   schema: spec-driven
   ```

2. **添加你的上下文**（要简洁 — 这会进入每个请求）：
   ```yaml
   context: |
     你的项目背景在这里。
     专注于 AI 真正需要知道的。
   ```

3. **添加每产物规则**（可选）：
   ```yaml
   rules:
     proposal:
       - 你的提案特定指导
     specs:
       - 你的规格编写规则
   ```

4. **一旦你移动了所有有用的内容，删除 project.md。**

**不要想太多。** 从 essentials 开始，然后迭代。如果你注意到 AI 遗漏了重要的东西，添加它。如果上下文感觉臃肿，削减它。这是一个活的文档。

### 需要帮助？使用这个提示

如果你不确定如何提炼你的 project.md，问你的 AI 助手：

```
我正在从 OpenSpec 旧的 project.md 迁移到新的 config.yaml 格式。

这是我当前的 project.md：
[paste your project.md content]

请帮我创建一个 config.yaml，包含：
1. 一个简洁的 `context:` 部分（这会被注入到每个规划请求中，所以保持紧凑 — 专注于技术栈、关键约束和经常被忽略的约定）
2. 如果任何内容是特定于产物的，为特定产物添加 `rules:`（例如，"在规格中使用 Given/When/Then" 属于规格规则，不是全局上下文）

省略任何 AI 模型已经知道的通用内容。要 ruthless 关于简洁。
```

AI 会帮助你识别什么是 essential 的，什么可以修剪。

---

## 新命令

命令可用性取决于配置文件：

**默认（`core` 配置文件）：**

| 命令 | 用途 |
|---------|---------|
| `/opsx:propose` | 一步创建变更并生成规划产物 |
| `/opsx:explore` | 无结构地思考想法 |
| `/opsx:apply` | 实施 tasks.md 中的任务 |
| `/opsx:archive` | 完成并归档变更 |

**扩展工作流（自定义选择）：**

| 命令 | 用途 |
|---------|---------|
| `/opsx:new` | 启动新的变更脚手架 |
| `/opsx:continue` | 一次创建一个产物 |
| `/opsx:ff` | 快进 — 一次创建规划产物 |
| `/opsx:verify` | 验证实施符合规格 |
| `/opsx:sync` | 预览/规格合并但不归档 |
| `/opsx:bulk-archive` | 一次归档多个变更 |
| `/opsx:onboard` | 引导式端到端入职工作流 |

用 `openspec config profile` 启用扩展命令，然后运行 `openspec update`。

### 从旧版命令映射

| 旧版 | OPSX 等价 |
|--------|-----------------|
| `/openspec:proposal` | `/opsx:propose`（默认）或 `/opsx:new` 然后 `/opsx:ff`（扩展） |
| `/openspec:apply` | `/opsx:apply` |
| `/openspec:archive` | `/opsx:archive` |

### 新能力

这些能力是扩展工作流命令集的一部分。

**细粒度产物创建：**
```
/opsx:continue
```
基于依赖一次创建一个产物。当你想在继续之前审查每一步时使用。

**探索模式：**
```
/opsx:explore
```
在承诺变更之前与伙伴一起思考想法。

---

## 理解新架构

### 从阶段锁定到流动

旧版工作流强制线性进度：

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   规划       │ ───► │    实施      │ ───► │    归档      │
│    阶段      │      │    阶段      │      │    阶段      │
└──────────────┘      └──────────────┘      └──────────────┘

如果你在实施中意识到设计错了？
运气不好。阶段门不让你轻易回去。
```

OPSX 使用动作，不是阶段：

```
         ┌───────────────────────────────────────────────┐
         │           动作（不是阶段）                    │
         │                                               │
         │     new ◄──► continue ◄──► apply ◄──► archive │
         │      │          │           │             │   │
         │      └──────────┴───────────┴─────────────┘   │
         │                    任意顺序                   │
         └───────────────────────────────────────────────┘
```

### 依赖图

产物形成一个有向图。依赖是赋能者，不是门：

```
                         proposal
                        （根节点）
                             │
               ┌─────────────┴─────────────┐
               │                           │
               ▼                           ▼
            specs                       design
         (requires:                  (requires:
          proposal)                   proposal)
               │                           │
               └─────────────┬─────────────┘
                             │
                             ▼
                          tasks
                      (requires:
                      specs, design)
```

当你运行 `/opsx:continue` 时，它检查什么准备好了并提供下一个产物。你也可以按任意顺序创建多个就绪的产物。

### 技能 vs 命令

旧版系统使用特定于工具的命令文件：

```
.claude/commands/openspec/
├── proposal.md
├── apply.md
└── archive.md
```

OPSX 使用新兴的**技能**标准：

```
.claude/skills/
├── openspec-explore/SKILL.md
├── openspec-new-change/SKILL.md
├── openspec-continue-change/SKILL.md
├── openspec-apply-change/SKILL.md
└── ...
```

技能在多个 AI 编码工具中被识别并提供更丰富的元数据。

---

## 继续现有变更

你的进行中的变更与 OPSX 命令无缝协作。

**有旧版工作流的active变更？**

```
/opsx:apply add-my-feature
```

OPSX 读取现有产物并从你停止的地方继续。

**想为现有变更添加更多产物？**

```
/opsx:continue add-my-feature
```

显示基于已存在的内容准备创建什么。

**需要查看状态？**

```bash
openspec status --change add-my-feature
```

---

## 新配置系统

### config.yaml 结构

```yaml
# 必需：新变更的默认 schema
schema: spec-driven

# 可选：项目上下文（最大 50KB）
# 注入到所有产物指令中
context: |
  你的项目背景、技术栈、
  约定和约束。

# 可选：每产物规则
# 仅注入到匹配的产物中
rules:
  proposal:
    - 包含回滚计划
  specs:
    - 使用 Given/When/Then 格式
  design:
    - 记录回退策略
  tasks:
    - 分解成最多 2 小时的大块
```

### Schema 解析

在确定使用哪个 schema 时，OPSX 按顺序检查：

1. **CLI 标志**： `--schema <name>`（最高优先级）
2. **变更元数据**：变更目录中的 `.openspec.yaml`
3. **项目配置**： `openspec/config.yaml`
4. **默认值**： `spec-driven`

### 可用 Schema

| Schema | 产物 | 适用于 |
|--------|-----------|----------|
| `spec-driven` | proposal → specs → design → tasks | 大多数项目 |

列出所有可用 schema：

```bash
openspec schemas
```

### 自定义 Schema

创建你自己的工作流：

```bash
openspec schema init my-workflow
```

或 fork 一个现有的：

```bash
openspec schema fork spec-driven my-workflow
```

详情参见[自定义](customization.md)。

---

## 故障排除

### "在非交互模式检测到旧版文件"

你在 CI 或非交互环境中运行。使用：

```bash
openspec init --force
```

### 迁移后命令没有出现

重启你的 IDE。技能在启动时检测。

### "规则中未知的产物 ID"

检查你的 `rules:` 键是否与你的 schema 的产物 ID 匹配：

- **spec-driven**：`proposal`、`specs`、`design`、`tasks`

运行这个查看有效的产物 ID：

```bash
openspec schemas --json
```

### 配置未应用

1. 确保文件在 `openspec/config.yaml`（不是 `.yml`）
2. 验证 YAML 语法
3. 配置更改立即生效 — 无需重启

### project.md 未迁移

系统故意保留 `project.md`，因为它可能包含你的自定义内容。手动查看，将有用部分移到 `config.yaml`，然后删除它。

### 想看看什么会被清理？

运行 init 并拒绝清理提示 — 你会看到完整的检测摘要，而不进行任何更改。

---

## 快速参考

### 迁移后的文件

```
project/
├── openspec/
│   ├── specs/                    # 不变
│   ├── changes/                  # 不变
│   │   └── archive/              # 不变
│   └── config.yaml               # 新：项目配置
├── .claude/
│   └── skills/                   # 新：OPSX 技能
│       ├── openspec-propose/     # 默认 core 配置文件
│       ├── openspec-explore/
│       ├── openspec-apply-change/
│       ├── openspec-sync-specs/
│       └── ...                   # 扩展配置文件添加 new/continue/ff 等
├── CLAUDE.md                     # OpenSpec 标记已删除，你的内容保留
└── AGENTS.md                     # OpenSpec 标记已删除，你的内容保留
```

### 什么被删除

- `.claude/commands/openspec/` — 被 `.claude/skills/` 取代
- `openspec/AGENTS.md` — 过时
- `openspec/project.md` — 迁移到 `config.yaml`，然后删除
- `CLAUDE.md`、`AGENTS.md` 等中的 OpenSpec 标记块

### 命令速查

```text
/opsx:propose      快速开始（默认 core 配置文件）
/opsx:apply        实施任务
/opsx:archive      完成并归档

# 扩展工作流（如果启用）：
/opsx:new          脚手架变更
/opsx:continue     创建下一个产物
/opsx:ff           创建规划产物
```

---

## 获取帮助

- **Discord**：[discord.gg/YctCnvvshC](https://discord.gg/YctCnvvshC)
- **GitHub Issues**：[github.com/Fission-AI/OpenSpec/issues](https://github.com/Fission-AI/OpenSpec/issues)
- **文档**：[docs/opsx.md](opsx.md) 获取完整 OPSX 参考