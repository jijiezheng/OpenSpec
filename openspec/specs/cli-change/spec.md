# cli-change 规范

## 目的
定义 `openspec change` 命令的行为，用于显示、列出和验证变更提案和增量。

## 需求

### 需求：Change 命令

系统应提供 `change` 命令，包含用于显示、列出和验证变更提案的子命令。

#### 场景：将变更显示为 JSON

- **当** 执行 `openspec change show update-error --json` 时
- **那么** 解析 markdown 变更文件
- **并且** 提取变更结构和增量
- **并且** 将有效 JSON 输出到 stdout

#### 场景：列出所有变更

- **当** 执行 `openspec change list` 时
- **那么** 扫描 openspec/changes 目录
- **并且** 返回所有待处理变更的列表
- **并且** 支持使用 `--json` 标志的 JSON 输出

#### 场景：仅显示需求变更

- **当** 执行 `openspec change show update-error --requirements-only` 时
- **那么** 仅显示需求变更（ADDED/MODIFIED/REMOVED/RENAMED）
- **并且** 排除 why 和 what 变更部分

#### 场景：验证变更结构

- **当** 执行 `openspec change validate update-error` 时
- **那么** 解析变更文件
- **并且** 根据 Zod schema 验证
- **并且** 确保增量格式良好

### 需求：向后兼容性

系统应在显示弃用通知的同时维护与现有 `list` 命令的向后兼容性。

#### 场景：传统 list 命令

- **当** 执行 `openspec list` 时
- **那么** 显示当前变更列表（现有行为）
- **并且** 显示弃用通知："Note: 'openspec list' is deprecated. Use 'openspec change list' instead."

#### 场景：带 --all 标志的传统 list

- **当** 执行 `openspec list --all` 时
- **那么** 显示所有变更（现有行为）
- **并且** 显示相同的弃用通知

### 需求：交互式 show 选择

change show 命令应在未提供变更名称时支持交互式选择。

#### 场景：show 的交互式变更选择

- **当** 执行 `openspec change show` 不带参数时
- **那么** 显示可用变更的交互列表
- **并且** 允许用户选择要显示的变更
- **并且** 显示所选变更内容
- **并且** 保持所有现有 show 选项（--json、--deltas-only）

#### 场景：非交互回退保持当前行为

- **假设** stdin 不是 TTY 或提供了 `--no-interactive` 或环境变量 `OPEN_SPEC_INTERACTIVE=0`
- **当** 执行 `openspec change show` 而不提供变更名称时
- **那么** 不进行交互式提示
- **并且** 打印包含可用变更 ID 的现有提示
- **并且** 设置 `process.exitCode = 1`

### 需求：交互式验证选择

change validate 命令应在未提供变更名称时支持交互式选择。

#### 场景：验证的交互式变更选择

- **当** 执行 `openspec change validate` 不带参数时
- **那么** 显示可用变更的交互列表
- **并且** 允许用户选择要验证的变更
- **并且** 验证所选变更

#### 场景：非交互回退保持当前行为

- **假设** stdin 不是 TTY 或提供了 `--no-interactive` 或环境变量 `OPEN_SPEC_INTERACTIVE=0`
- **当** 执行 `openspec change validate` 而不提供变更名称时
- **那么** 不进行交互式提示
- **并且** 打印包含可用变更 ID 的现有提示
- **并且** 设置 `process.exitCode = 1`