# cli-artifact-workflow 规范

## 目的
定义产物工作流 CLI 行为（`status`、`instructions`、`templates` 和设置流程），用于脚手架化和活动变更。

## 需求

### 需求：Status 命令

系统应显示变更的产物完成状态，包括脚手架化（空）变更。

> **修复错误**：以前通过 `getActiveChangeIds()` 要求 `proposal.md` 存在。

#### 场景：显示所有状态的状态

- **当** 用户运行 `openspec status --change <id>` 时
- **那么** 系统显示每个带有状态指示器的产物：
  - `[x]` 表示已完成的产物
  - `[ ]` 表示就绪的产物
  - `[-]` 表示被阻塞的产物（列出缺失的依赖）

#### 场景：状态显示完成摘要

- **当** 用户运行 `openspec status --change <id>` 时
- **那么** 输出包括完成百分比和计数（例如 "2/4 artifacts complete"）

#### 场景：状态 JSON 输出

- **当** 用户运行 `openspec status --change <id> --json` 时
- **那么** 系统输出带有 changeName、schemaName、isComplete 和 artifacts 数组的 JSON

#### 场景：状态 JSON 包含 apply 要求

- **当** 用户运行 `openspec status --change <id> --json` 时
- **那么** 系统输出带有以下内容的 JSON：
  - `changeName`、`schemaName`、`isComplete`、`artifacts` 数组
  - `applyRequires`：apply 阶段需要的产物 ID 数组

#### 场景：脚手架化变更上的状态

- **当** 用户在没有产物的变更上运行 `openspec status --change <id>` 时
- **那么** 系统显示所有带有状态的产物
- **并且** 根产物（无依赖）显示为就绪 `[ ]`
- **并且** 依赖的产物显示为被阻塞 `[-]`

#### 场景：缺少 change 参数

- **当** 用户在没有 `--change` 的情况下运行 `openspec status` 时
- **那么** 系统显示包含可用变更列表的错误
- **并且** 包括脚手架化变更（没有 proposal.md 的目录）

#### 场景：未知的变更

- **当** 用户运行 `openspec status --change unknown-id` 时
- **并且** 目录 `openspec/changes/unknown-id/` 不存在
- **那么** 系统显示列出所有可用变更目录的错误

### 需求：下一个产物发现

工作流应使用 `openspec status` 输出确定下一步可以创建什么，而不是单独的 next-command 界面。

#### 场景：从状态输出发现下一个产物

- **当** 用户需要知道下一步创建哪个产物时
- **那么** `openspec status --change <id>` 识别带有 `[ ]` 的就绪产物
- **并且** 不需要专门的"next 命令"来继续工作流

### 需求：Instructions 命令

系统应输出用于创建产物的富指令，包括用于脚手架化变更的指令。

#### 场景：显示富指令

- **当** 用户运行 `openspec instructions <artifact> --change <id>` 时
- **那么** 系统输出：
  - 产物元数据（ID、输出路径、描述）
  - 模板内容
  - 依赖状态（已完成/缺失）
  - 已解锁的产物（完成后哪些将可用）

#### 场景：Instructions JSON 输出

- **当** 用户运行 `openspec instructions <artifact> --change <id> --json` 时
- **那么** 系统输出匹配 ArtifactInstructions 接口的 JSON

#### 场景：未知的产物

- **当** 用户运行 `openspec instructions unknown-artifact --change <id>` 时
- **那么** 系统显示列出该 schema 有效产物 ID 的错误

#### 场景：具有未满足依赖的产物

- **当** 用户请求被阻塞产物的指令时
- **那么** 系统显示带有缺失依赖警告的指令

#### 场景：脚手架化变更上的 Instructions

- **当** 用户在脚手架化变更上运行 `openspec instructions proposal --change <id>` 时
- **那么** 系统输出用于创建 proposal 的模板和元数据
- **并且** 不需要任何产物已存在

### 需求：Templates 命令
系统应显示 schema 中所有产物的已解析模板路径。

#### 场景：使用默认 schema 列出模板路径

- **当** 用户运行 `openspec templates` 时
- **那么** 系统使用默认 schema 显示每个产物及其已解析模板路径

#### 场景：使用自定义 schema 列出模板路径

- **当** 用户运行 `openspec templates --schema tdd` 时
- **那么** 系统显示指定 schema 的模板路径

#### 场景：Templates JSON 输出

- **当** 用户运行 `openspec templates --json` 时
- **那么** 系统输出将产物 ID 映射到模板路径的 JSON

#### 场景：模板解析源

- **当** 显示模板路径时
- **那么** 系统指示每个模板是来自用户覆盖还是包内置

### 需求：New Change 命令
系统应创建带有验证的新变更目录。

#### 场景：创建有效变更

- **当** 用户运行 `openspec new change add-feature` 时
- **那么** 系统创建 `openspec/changes/add-feature/` 目录

#### 场景：无效的变更名称

- **当** 用户运行 `openspec new change "Add Feature"` 使用无效名称时
- **那么** 系统显示带有指导的验证错误

#### 场景：重复的变更名称

- **当** 用户为已存在的变更运行 `openspec new change existing-change` 时
- **那么** 系统显示指示变更已存在的错误

#### 场景：创建时带描述

- **当** 用户运行 `openspec new change add-feature --description "Add new feature"` 时
- **那么** 系统创建带 README.md 中描述的变更目录

### 需求：Schema 选择
系统应支持工作流命令的自定义 schema 选择。

#### 场景：默认 schema

- **当** 用户运行没有 `--schema` 的工作流命令时
- **那么** 系统使用"spec-driven" schema

#### 场景：自定义 schema

- **当** 用户运行 `openspec status --change <id> --schema tdd` 时
- **那么** 系统使用指定的 schema 作为产物图

#### 场景：未知 schema

- **当** 用户指定未知 schema 时
- **那么** 系统显示列出可用 schema 的错误

### 需求：输出格式
系统应提供一致的输出格式。

#### 场景：彩色输出

- **当** 终端支持颜色时
- **那么** 状态指示器使用颜色：绿色（完成）、黄色（就绪）、红色（阻塞）

#### 场景：无颜色输出

- **当** 使用 `--no-color` 标志或设置 `NO_COLOR` 环境变量时
- **那么** 输出使用纯文本指示器，不带 ANSI 颜色

#### 场景：进度指示

- **当** 加载变更状态需要时间时
- **那么** 系统在加载期间显示旋转动画

### 需求：实验性隔离
系统应在隔离中实现产物工作流命令，以便于移除。

#### 场景：单文件实现

- **当** 实现产物工作流功能时
- **那么** 所有命令都在 `src/commands/artifact-workflow.ts` 中

#### 场景：帮助文本标记

- **当** 用户在任何产物工作流命令上运行 `--help` 时
- **那么** 帮助文本指示该命令是实验性的

### 需求：Schema Apply 块

系统应支持 schema 定义中的 `apply` 块，控制实现何时以及如何开始。

#### 场景：带 apply 块的 Schema

- **当** schema 定义了 `apply` 块时
- **那么** 系统使用 `apply.requires` 确定 apply 前必须存在哪些产物
- **并且** 使用 `apply.tracks` 识别进度跟踪文件（如果没有则为空）
- **并且** 使用 `apply.instruction` 显示给代理的指导

#### 场景：没有 apply 块的 Schema

- **当** schema 没有 `apply` 块时
- **那么** 系统要求所有产物流 程在 apply 前存在
- **并且** 使用默认指令："All artifacts complete. Proceed with implementation."

### 需求：Apply Instructions 命令

系统应通过 `openspec instructions apply` 生成 schema 感知的 apply 指令。

#### 场景：生成 apply 指令

- **当** 用户运行 `openspec instructions apply --change <id>` 时
- **并且** 所有必需产物（根据 schema 的 `apply.requires`）存在
- **那么** 系统输出：
  - `contextFiles` 将产物 ID 映射到所有现有产物的具体路径数组
  - Schema 特定的指令文本
  - 进度跟踪文件路径（如果设置了 `apply.tracks`）

#### 场景：Apply 被缺失产物阻塞

- **当** 用户运行 `openspec instructions apply --change <id>` 时
- **并且** 必需的产物缺失
- **那么** 系统指示 apply 被阻塞
- **并且** 列出必须首先创建哪些产物

#### 场景：Apply Instructions JSON 输出

- **当** 用户运行 `openspec instructions apply --change <id> --json` 时
- **那么** 系统输出带有以下内容的 JSON：
  - `contextFiles`：将产物 ID 映射到现有产物的具体路径数组的对象
  - `instruction`：apply 指令文本
  - `tracks`：进度文件路径或 null
  - `applyRequires`：必需的产物 ID 列表

### 需求：工具选择标志

`artifact-experimental-setup` 命令应接受 `--tool <tool-id>` 标志以指定目标 AI 工具。

#### 场景：通过标志指定工具

- **当** 用户运行 `openspec artifact-experimental-setup --tool cursor` 时
- **那么** 技能文件在 `.cursor/skills/` 中生成
- **并且** 命令文件使用 Cursor 的 frontmatter 格式生成

#### 场景：缺少工具标志

- **当** 用户在没有 `--tool` 的情况下运行 `openspec artifact-experimental-setup` 时
- **那么** 系统显示需要 `--tool` 标志的错误
- **并且** 在错误消息中列出有效的工具 ID

#### 场景：未知的工具 ID

- **当** 用户运行 `openspec artifact-experimental-setup --tool unknown-tool` 时
- **并且** 工具 ID 不在 `AI_TOOLS` 中
- **那么** 系统显示列出有效工具 ID 的错误

#### 场景：没有 skillsDir 的工具

- **当** 用户指定没有配置 `skillsDir` 的工具时
- **那么** 系统显示错误，指示该工具不支持技能生成

#### 场景：没有命令适配器的工具

- **当** 用户指定有 `skillsDir` 但没有注册命令适配器的工具时
- **那么** 技能文件成功生成
- **并且** 命令生成被跳过，并显示信息消息

### 需求：输出消息

设置命令应显示关于生成内容的清晰输出。

#### 场景：在输出中显示目标工具

- **当** 设置命令成功运行时
- **那么** 输出包括目标工具名称（例如 "Setting up for Cursor..."）

#### 场景：显示生成的路径

- **当** 设置命令完成时
- **那么** 输出列出所有生成的技能文件路径
- **并且** 列出所有生成的命令文件路径（如果适用）

#### 场景：显示跳过的命令消息

- **当** 由于缺少适配器而跳过命令生成时
- **那么** 输出包含消息："Command generation skipped - no adapter for <tool>"