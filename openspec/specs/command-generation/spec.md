# command-generation 规范

## 目的
定义与工具无关的命令内容和适配器合约，用于生成特定于工具的 OpenSpec 命令文件。

## 需求

### 需求：CommandContent 接口

系统应定义与工具无关的 `CommandContent` 接口用于命令数据。

#### 场景：CommandContent 结构

- **当** 定义要生成的命令时
- **那么** `CommandContent` 应包括：
  - `id`：字符串标识符（例如 'explore'、'apply'）
  - `name`：人类可读名称（例如 'OpenSpec Explore'）
  - `description`：命令目的的简要描述
  - `category`：分组类别（例如 'OpenSpec'）
  - `tags`：标签字符串数组
  - `body`：命令指令内容

### 需求：ToolCommandAdapter 接口

系统应定义 `ToolCommandAdapter` 接口用于每个工具的格式化。

#### 场景：适配器接口结构

- **当** 实现工具适配器时
- **那么** `ToolCommandAdapter` 应要求：
  - `toolId`：匹配 `AIToolOption.value` 的字符串标识符
  - `getFilePath(commandId: string)`：返回命令的文件路径（对于项目根目录为相对路径，对于像 Codex 这样的全局作用域工具为绝对路径）
  - `formatFile(content: CommandContent)`：返回带有 frontmatter 的完整文件内容

#### 场景：Claude 适配器格式化

- **当** 为 Claude Code 格式化命令时
- **那么** 适配器应输出带有 `name`、`description`、`category`、`tags` 字段的 YAML frontmatter
- **并且** 文件路径应遵循模式 `.claude/commands/opsx/<id>.md`

#### 场景：Cursor 适配器格式化

- **当** 为 Cursor 格式化命令时
- **那么** 适配器应输出带有 `name` 作为 `/opsx-<id>`、`id`、`category`、`description` 字段的 YAML frontmatter
- **并且** 文件路径应遵循模式 `.cursor/commands/opsx-<id>.md`

#### 场景：Windsurf 适配器格式化

- **当** 为 Windsurf 格式化命令时
- **那么** 适配器应输出带有 `name`、`description`、`category`、`tags` 字段的 YAML frontmatter
- **并且** 文件路径应遵循模式 `.windsurf/workflows/opsx-<id>.md`

### 需求：命令生成器函数

系统应提供将内容与适配器组合的 `generateCommand` 函数。

#### 场景：生成命令文件

- **当** 调用 `generateCommand(content, adapter)` 时
- **那么** 它应返回带有以下内容的对象：
  - `path`：来自 `adapter.getFilePath(content.id)` 的文件路径
  - `fileContent`：来自 `adapter.formatFile(content)` 的格式化内容

#### 场景：生成多个命令

- **当** 为工具生成所有 opsx 命令时
- **那么** 系统应迭代命令内容并使用该工具的适配器生成每个命令

### 需求：CommandAdapterRegistry

系统应提供用于查找工具适配器的注册表。

#### 场景：通过工具 ID 获取适配器

- **当** 调用 `CommandAdapterRegistry.get('cursor')` 时
- **那么** 它应返回 Cursor 适配器，如果未注册则返回 undefined

#### 场景：获取所有适配器

- **当** 调用 `CommandAdapterRegistry.getAll()` 时
- **那么** 它应返回所有已注册适配器的数组

#### 场景：适配器未找到

- **当** 查找未注册工具的适配器时
- **那么** `CommandAdapterRegistry.get()` 应返回 undefined
- **并且** 调用者应适当处理缺失的适配器

### 需求：共享命令正文内容

命令的正文内容应在所有工具之间共享。

#### 场景：跨工具相同指令

- **当** 为 Claude 和 Cursor 生成 'explore' 命令时
- **那么** 两者应使用相同的 `body` 内容
- **并且** 只有 frontmatter 和文件路径应不同