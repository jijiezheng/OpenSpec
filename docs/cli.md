# CLI 参考

OpenSpec CLI（`openspec`）提供用于项目设置、验证、状态检查和管理的终端命令。这些命令补充了 AI 斜杠命令（如 `/opsx:propose`），文档见[命令](commands.md)。

## 概要

| 类别 | 命令 | 用途 |
|----------|----------|---------|
| **设置** | `init`、`update` | 在你的项目中初始化和更新 OpenSpec |
| **浏览** | `list`、`view`、`show` | 探索变更和规格 |
| **验证** | `validate` | 检查变更和规格的问题 |
| **生命周期** | `archive` | 完成已完成的变更 |
| **工作流** | `status`、`instructions`、`templates`、`schemas` | 产物驱动的工作流支持 |
| **Schema** | `schema init`、`schema fork`、`schema validate`、`schema which` | 创建和管理自定义工作流 |
| **配置** | `config` | 查看和修改设置 |
| **实用工具** | `feedback`、`completion` | 反馈和 shell 集成 |

---

## 人 vs 代理命令

大多数 CLI 命令设计为**人类在终端中使用**。一些命令也通过 JSON 输出支持**代理/脚本使用**。

### 仅人类命令

这些命令是交互式的，设计用于终端：

| 命令 | 用途 |
|---------|---------|
| `openspec init` | 初始化项目（交互式提示） |
| `openspec view` | 交互式仪表板 |
| `openspec config edit` | 在编辑器中打开配置 |
| `openspec feedback` | 通过 GitHub 提交反馈 |
| `openspec completion install` | 安装 shell 补全 |

### 代理兼容命令

这些命令支持 `--json` 输出，用于 AI 代理和脚本的编程使用：

| 命令 | 人类使用 | 代理使用 |
|---------|-----------|-----------|
| `openspec list` | 浏览变更/规格 | `--json` 获取结构化数据 |
| `openspec show <item>` | 读取内容 | `--json` 用于解析 |
| `openspec validate` | 检查问题 | `--all --json` 用于批量验证 |
| `openspec status` | 查看产物进度 | `--json` 获取结构化状态 |
| `openspec instructions` | 获取下一步 | `--json` 用于代理指令 |
| `openspec templates` | 查找模板路径 | `--json` 用于路径解析 |
| `openspec schemas` | 列出可用 schema | `--json` 用于 schema 发现 |

---

## 全局选项

这些选项适用于所有命令：

| 选项 | 描述 |
|--------|---------|
| `--version`、`-V` | 显示版本号 |
| `--no-color` | 禁用彩色输出 |
| `--help`、`-h` | 显示命令帮助 |

---

## 设置命令

### `openspec init`

在你的项目中初始化 OpenSpec。创建文件夹结构并配置 AI 工具集成。

默认行为使用全局配置默认值：配置文件 `core`、交付 `both`、工作流 `propose、explore、apply、sync、archive`。

```
openspec init [路径] [选项]
```

**参数：**

| 参数 | 必需 | 描述 |
|----------|----------|-------------|
| `路径` | 否 | 目标目录（默认：当前目录） |

**选项：**

| 选项 | 描述 |
|--------|---------|
| `--tools <列表>` | 非交互式配置 AI 工具。使用 `all`、`none` 或逗号分隔列表 |
| `--force` | 不提示自动清理旧版文件 |
| `--profile <配置文件>` | 覆盖此 init 运行的全局配置文件（`core` 或 `custom`） |

`--profile custom` 使用全局配置中当前选择的工作流（`openspec config profile`）。

**支持的工具 ID（`--tools`）：** `amazon-q`、`antigravity`、`auggie`、`bob`、`claude`、`cline`、`codex`、`forgecode`、`codebuddy`、`continue`、`costrict`、`crush`、`cursor`、`factory`、`gemini`、`github-copilot`、`iflow`、`junie`、`kilocode`、`kimi`、`kiro`、`opencode`、`pi`、`qoder`、`lingma`、`qwen`、`roocode`、`trae`、`windsurf`

**示例：**

```bash
# 交互式初始化
openspec init

# 在特定目录中初始化
openspec init ./my-project

# 非交互式：为 Claude 和 Cursor 配置
openspec init --tools claude,cursor

# 为所有支持的工具配置
openspec init --tools all

# 覆盖此运行的配置文件
openspec init --profile core

# 跳过提示并自动清理旧版文件
openspec init --force
```

**创建的内容：**

```
openspec/
├── specs/              # 你的规格（事实来源）
├── changes/            # 提议的变更
└── config.yaml         # 项目配置

.claude/skills/         # Claude Code 技能（如果选择了 claude）
.cursor/skills/         # Cursor 技能（如果选择了 cursor）
.cursor/commands/       # Cursor OPSX 命令（如果交付包含命令）
...（其他工具配置）
```

---

### `openspec update`

升级 CLI 后更新 OpenSpec 指令文件。使用你当前的全局配置文件、选择的工作流和交付模式重新生成 AI 工具配置文件。

```
openspec update [路径] [选项]
```

**参数：**

| 参数 | 必需 | 描述 |
|----------|----------|-------------|
| `路径` | 否 | 目标目录（默认：当前目录） |

**选项：**

| 选项 | 描述 |
|--------|---------|
| `--force` | 即使文件是最新的也强制更新 |

**示例：**

```bash
# npm 升级后更新指令文件
npm update @fission-ai/openspec
openspec update
```

---

## 浏览命令

### `openspec list`

列出项目中的变更或规格。

```
openspec list [选项]
```

**选项：**

| 选项 | 描述 |
|--------|---------|
| `--specs` | 列出规格而非变更 |
| `--changes` | 列出变更（默认） |
| `--sort <顺序>` | 按 `recent`（默认）或 `name` 排序 |
| `--json` | 输出为 JSON |

**示例：**

```bash
# 列出所有活动变更
openspec list

# 列出所有规格
openspec list --specs

# 用于脚本的 JSON 输出
openspec list --json
```

**输出（文本）：**

```
活动变更：
  add-dark-mode     UI 主题切换支持
  fix-login-bug     会话超时处理
```

---

### `openspec view`

显示用于探索规格和变更的交互式仪表板。

```
openspec view
```

打开基于终端的界面来导航项目的规格和变更。

---

### `openspec show`

显示变更或规格的详情。

```
openspec show [项目名称] [选项]
```

**参数：**

| 参数 | 必需 | 描述 |
|----------|----------|-------------|
| `项目名称` | 否 | 变更或规格的名称（如果省略则提示） |

**选项：**

| 选项 | 描述 |
|--------|---------|
| `--type <类型>` | 指定类型：`change` 或 `spec`（如果无歧义则自动检测） |
| `--json` | 输出为 JSON |
| `--no-interactive` | 禁用提示 |

**变更特定选项：**

| 选项 | 描述 |
|--------|---------|
| `--deltas-only` | 仅显示增量规格（JSON 模式） |

**规格特定选项：**

| 选项 | 描述 |
|--------|---------|
| `--requirements` | 仅显示需求，排除场景（JSON 模式） |
| `--no-scenarios` | 排除场景内容（JSON 模式） |
| `-r, --requirement <id>` | 按 1-based 索引显示特定需求（JSON 模式） |

**示例：**

```bash
# 交互式选择
openspec show

# 显示特定变更
openspec show add-dark-mode

# 显示特定规格
openspec show auth --type spec

# 用于解析的 JSON 输出
openspec show add-dark-mode --json
```

---

## 验证命令

### `openspec validate`

验证变更和规格的结构问题。

```
openspec validate [项目名称] [选项]
```

**参数：**

| 参数 | 必需 | 描述 |
|----------|----------|-------------|
| `项目名称` | 否 | 要验证的特定项目（如果省略则提示） |

**选项：**

| 选项 | 描述 |
|--------|---------|
| `--all` | 验证所有变更和规格 |
| `--changes` | 验证所有变更 |
| `--specs` | 验证所有规格 |
| `--type <类型>` | 当名称有歧义时指定类型：`change` 或 `spec` |
| `--strict` | 启用严格验证模式 |
| `--json` | 输出为 JSON |
| `--concurrency <n>` | 最大并行验证数（默认：6，或 `OPENSPEC_CONCURRENCY` 环境变量） |
| `--no-interactive` | 禁用提示 |

**示例：**

```bash
# 交互式验证
openspec validate

# 验证特定变更
openspec validate add-dark-mode

# 验证所有变更
openspec validate --changes

# 用 JSON 输出验证所有内容（用于 CI/脚本）
openspec validate --all --json

# 严格验证并增加并行度
openspec validate --all --strict --concurrency 12
```

**输出（文本）：**

```
正在验证 add-dark-mode...
  ✓ proposal.md 有效
  ✓ specs/ui/spec.md 有效
  ⚠ design.md：缺少"技术方法"节

发现 1 个警告
```

**输出（JSON）：**

```json
{
  "version": "1.0.0",
  "results": {
    "changes": [
      {
        "name": "add-dark-mode",
        "valid": true,
        "warnings": ["design.md: 缺少'技术方法'节"]
      }
    ]
  },
  "summary": {
    "total": 1,
    "valid": 1,
    "invalid": 0
  }
}
```

---

## 生命周期命令

### `openspec archive`

归档已完成的变更并将增量规格合并到主规格。

```
openspec archive [变更名称] [选项]
```

**参数：**

| 参数 | 必需 | 描述 |
|----------|----------|-------------|
| `变更名称` | 否 | 要归档的变更（如果省略则提示） |

**选项：**

| 选项 | 描述 |
|--------|---------|
| `-y, --yes` | 跳过确认提示 |
| `--skip-specs` | 跳过规格更新（用于基础设施/工具/仅文档变更） |
| `--no-validate` | 跳过验证（需要确认） |

**示例：**

```bash
# 交互式归档
openspec archive

# 归档特定变更
openspec archive add-dark-mode

# 无提示归档（用于 CI/脚本）
openspec archive add-dark-mode --yes

# 归档不影响规格的工具变更
openspec archive update-ci-config --skip-specs
```

**功能：**

1. 验证变更（除非 `--no-validate`）
2. 提示确认（除非 `--yes`）
3. 将增量规格合并到 `openspec/specs/`
4. 将变更文件夹移动到 `openspec/changes/archive/YYYY-MM-DD-<name>/`

---

## 工作流命令

这些命令支持产物驱动的 OPSX 工作流。它们对人类检查进度和代理确定下一步都有用。

### `openspec status`

显示变更的产物完成状态。

```
openspec status [选项]
```

**选项：**

| 选项 | 描述 |
|--------|---------|
| `--change <id>` | 变更名称（如果省略则提示） |
| `--schema <名称>` | Schema 覆盖（从变更的配置自动检测） |
| `--json` | 输出为 JSON |

**示例：**

```bash
# 交互式状态检查
openspec status

# 特定变更的状态
openspec status --change add-dark-mode

# 用于代理使用的 JSON
openspec status --change add-dark-mode --json
```

**输出（文本）：**

```
变更：add-dark-mode
Schema：spec-driven
进度：2/4 产物完成

[x] proposal
[ ] design
[x] specs
[-] tasks（被阻止：design）
```

**输出（JSON）：**

```json
{
  "changeName": "add-dark-mode",
  "schemaName": "spec-driven",
  "isComplete": false,
  "applyRequires": ["tasks"],
  "artifacts": [
    {"id": "proposal", "outputPath": "proposal.md", "status": "done"},
    {"id": "design", "outputPath": "design.md", "status": "ready"},
    {"id": "specs", "outputPath": "specs/**/*.md", "status": "done"},
    {"id": "tasks", "outputPath": "tasks.md", "status": "blocked", "missingDeps": ["design"]}
  ]
}
```

---

### `openspec instructions`

获取创建产物或应用任务的丰富指令。用于 AI 代理理解下一步要创建什么。

```
openspec instructions [产物] [选项]
```

**参数：**

| 参数 | 必需 | 描述 |
|----------|----------|-------------|
| `产物` | 否 | 产物 ID：`proposal`、`specs`、`design`、`tasks` 或 `apply` |

**选项：**

| 选项 | 描述 |
|--------|---------|
| `--change <id>` | 变更名称（非交互模式必需） |
| `--schema <名称>` | Schema 覆盖 |
| `--json` | 输出为 JSON |

**特殊 case：** 使用 `apply` 作为产物获取任务实施指令。

**示例：**

```bash
# 获取下一个产物的指令
openspec instructions --change add-dark-mode

# 获取特定产物指令
openspec instructions design --change add-dark-mode

# 获取 apply/实施指令
openspec instructions apply --change add-dark-mode

# 用于代理消费的 JSON
openspec instructions design --change add-dark-mode --json
```

**输出包括：**

- 产物的模板内容
- 来自配置的项目上下文
- 来自依赖产物的内容
- 来自配置的每产物规则

---

### `openspec templates`

显示 schema 中所有产物的解析模板路径。

```
openspec templates [选项]
```

**选项：**

| 选项 | 描述 |
|--------|---------|
| `--schema <名称>` | 要检查的 schema（默认：`spec-driven`） |
| `--json` | 输出为 JSON |

**示例：**

```bash
# 显示默认 schema 的模板路径
openspec templates

# 显示自定义 schema 的模板
openspec templates --schema my-workflow

# 用于程序使用的 JSON
openspec templates --json
```

**输出（文本）：**

```
Schema：spec-driven

模板：
  proposal  → ~/.openspec/schemas/spec-driven/templates/proposal.md
  specs     → ~/.openspec/schemas/spec-driven/templates/specs.md
  design    → ~/.openspec/schemas/spec-driven/templates/design.md
  tasks     → ~/.openspec/schemas/spec-driven/templates/tasks.md
```

---

### `openspec schemas`

列出具有描述和产物流的可用水工作流 schema。

```
openspec schemas [选项]
```

**选项：**

| 选项 | 描述 |
|--------|---------|
| `--json` | 输出为 JSON |

**示例：**

```bash
openspec schemas
```

**输出：**

```
可用 schema：

  spec-driven (package)
    默认的规格驱动开发工作流
    流：proposal → specs → design → tasks

  my-custom (project)
    此项目的自定义工作流
    流：research → proposal → tasks
```

---

## Schema 命令

用于创建和管理自定义工作流 schema 的命令。

### `openspec schema init`

创建新的项目本地 schema。

```
openspec schema init <名称> [选项]
```

**参数：**

| 参数 | 必需 | 描述 |
|----------|----------|-------------|
| `名称` | 是 | Schema 名称（短横线分隔） |

**选项：**

| 选项 | 描述 |
|--------|---------|
| `--description <文本>` | Schema 描述 |
| `--artifacts <列表>` | 逗号分隔的产物 ID（默认：`proposal,specs,design,tasks`） |
| `--default` | 设置为项目默认 schema |
| `--no-default` | 不提示设置为默认 |
| `--force` | 覆盖现有 schema |
| `--json` | 输出为 JSON |

**示例：**

```bash
# 交互式 schema 创建
openspec schema init research-first

# 非交互式，特定产物
openspec schema init rapid \
  --description "快速迭代工作流" \
  --artifacts "proposal,tasks" \
  --default
```

**创建的内容：**

```
openspec/schemas/<名称>/
├── schema.yaml           # Schema 定义
└── templates/
    ├── proposal.md       # 每个产物的模板
    ├── specs.md
    ├── design.md
    └── tasks.md
```

---

### `openspec schema fork`

复制现有 schema 到你的项目以进行自定义。

```
openspec schema fork <源> [名称] [选项]
```

**参数：**

| 参数 | 必需 | 描述 |
|----------|----------|-------------|
| `源` | 是 | 要复制的 Schema |
| `名称` | 否 | 新 schema 名称（默认：`<源>-custom`） |

**选项：**

| 选项 | 描述 |
|--------|---------|
| `--force` | 覆盖现有目标 |
| `--json` | 输出为 JSON |

**示例：**

```bash
# Fork 内置的 spec-driven schema
openspec schema fork spec-driven my-workflow
```

---

### `openspec schema validate`

验证 schema 的结构和模板。

```
openspec schema validate [名称] [选项]
```

**参数：**

| 参数 | 必需 | 描述 |
|----------|----------|-------------|
| `名称` | 否 | 要验证的 schema（如果省略则验证所有） |

**选项：**

| 选项 | 描述 |
|--------|---------|
| `--verbose` | 显示详细验证步骤 |
| `--json` | 输出为 JSON |

**示例：**

```bash
# 验证特定 schema
openspec schema validate my-workflow

# 验证所有 schema
openspec schema validate
```

---

### `openspec schema which`

显示 schema 从哪里解析（用于调试优先级）。

```
openspec schema which [名称] [选项]
```

**参数：**

| 参数 | 必需 | 描述 |
|----------|----------|-------------|
| `名称` | 否 | Schema 名称 |

**选项：**

| 选项 | 描述 |
|--------|---------|
| `--all` | 列出所有 schema 及其来源 |
| `--json` | 输出为 JSON |

**示例：**

```bash
# 检查 schema 从哪里来
openspec schema which spec-driven
```

**输出：**

```
spec-driven 从以下位置解析：package
  来源：/usr/local/lib/node_modules/@fission-ai/openspec/schemas/spec-driven
```

**Schema 优先级：**

1. 项目：`openspec/schemas/<名称>/`
2. 用户：`~/.local/share/openspec/schemas/<名称>/`
3. 包：内置 schema

---

## 配置命令

### `openspec config`

查看和修改全局 OpenSpec 配置。

```
openspec config <子命令> [选项]
```

**子命令：**

| 子命令 | 描述 |
|------------|-------------|
| `path` | 显示配置文件位置 |
| `list` | 显示所有当前设置 |
| `get <键>` | 获取特定值 |
| `set <键> <值>` | 设置值 |
| `unset <键>` | 移除键 |
| `reset` | 重置为默认值 |
| `edit` | 在 `$EDITOR` 中打开 |
| `profile [预设]` | 交互式配置或通过预设配置工作流配置文件 |

**示例：**

```bash
# 显示配置文件路径
openspec config path

# 列出所有设置
openspec config list

# 获取特定值
openspec config get telemetry.enabled

# 设置值
openspec config set telemetry.enabled false

# 显式设置字符串值
openspec config set user.name "My Name" --string

# 移除自定义设置
openspec config unset user.name

# 重置所有配置
openspec config reset --all --yes

# 在编辑器中编辑配置
openspec config edit

# 使用基于向导的配置文件
openspec config profile

# 快速预设：切换到 core 工作流（保持交付模式）
openspec config profile core
```

`openspec config profile` 从当前状态摘要开始，然后让你选择：
- 更改交付 + 工作流
- 仅更改交付
- 仅更改工作流
- 保持当前设置（退出）

如果你保持当前设置，不会写入任何更改，也不会显示更新提示。
如果配置没有更改但当前项目文件与全局配置文件/交付不同步，OpenSpec 会显示警告并建议运行 `openspec update`。
按 `Ctrl+C` 也干净地取消流程（无堆栈跟踪）并以代码 `130` 退出。
在工作流检查列表中，`[x]` 表示工作流在全局配置中被选中。要将这些选择应用到项目文件，运行 `openspec update`（或在项目中被提示时选择`立即应用更改到此项目？`）。

**交互式示例：**

```bash
# 仅更新交付
openspec config profile
# 选择：仅更改交付
# 选择交付：仅技能

# 仅更新工作流
openspec config profile
# 选择：仅更改工作流
# 在检查列表中切换工作流，然后确认
```

---

## 实用工具命令

### `openspec feedback`

提交关于 OpenSpec 的反馈。创建 GitHub issue。

```
openspec feedback <消息> [选项]
```

**参数：**

| 参数 | 必需 | 描述 |
|----------|----------|-------------|
| `消息` | 是 | 反馈消息 |

**选项：**

| 选项 | 描述 |
|--------|---------|
| `--body <文本>` | 详细描述 |

**要求：** GitHub CLI（`gh`）必须已安装并认证。

**示例：**

```bash
openspec feedback "添加对自定义产物类型的支持" \
  --body "我想定义自己的产物类型，超出内置类型。"
```

---

### `openspec completion`

管理 OpenSpec CLI 的 shell 补全。

```
openspec completion <子命令> [shell]
```

**子命令：**

| 子命令 | 描述 |
|------------|-------------|
| `generate [shell]` | 输出补全脚本到 stdout |
| `install [shell]` | 为你的 shell 安装补全 |
| `uninstall [shell]` | 移除已安装的补全 |

**支持的 shell：** `bash`、`zsh`、`fish`、`powershell`

**示例：**

```bash
# 安装补全（自动检测 shell）
openspec completion install

# 为特定 shell 安装
openspec completion install zsh

# 生成脚本用于手动安装
openspec completion generate bash > ~/.bash_completion.d/openspec

# 卸载
openspec completion uninstall
```

---

## 退出代码

| 代码 | 含义 |
|------|---------|
| `0` | 成功 |
| `1` | 错误（验证失败、文件缺失等） |

---

## 环境变量

| 变量 | 描述 |
|----------|---------|
| `OPENSPEC_TELEMETRY` | 设置为 `0` 禁用遥测 |
| `DO_NOT_TRACK` | 设置为 `1` 禁用遥测（标准 DNT 信号） |
| `OPENSPEC_CONCURRENCY` | 批量验证的默认并发数（默认：6） |
| `EDITOR` 或 `VISUAL` | 用于 `openspec config edit` 的编辑器 |
| `NO_COLOR` | 设置时禁用彩色输出 |

---

## 相关文档

- [命令](commands.md) - AI 斜杠命令（`/opsx:propose`、`/opsx:apply` 等）
- [工作流](workflows.md) - 常见模式和每个命令的使用时机
- [自定义](customization.md) - 创建自定义 schema 和模板
- [快速入门](getting-started.md) - 首次设置指南