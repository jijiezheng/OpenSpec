# cli-show 规范

## 目的
定义顶级 `openspec show` 命令的行为，用于交互式和直接显示变更和规格内容。

## 需求

### 需求：顶级 show 命令

CLI 应提供顶级 `show` 命令，用于显示变更和规格，并具有智能选择功能。

#### 场景：交互式 show 选择

- **当** 执行 `openspec show` 不带参数时
- **那么** 提示用户选择类型（变更或规格）
- **并且** 显示所选类型的可用项目列表
- **并且** 显示所选项目的内容

#### 场景：非交互环境不提示

- **假设** stdin 不是 TTY 或提供了 `--no-interactive` 或环境变量 `OPEN_SPEC_INTERACTIVE=0`
- **当** 执行 `openspec show` 不带参数时
- **那么** 不提示
- **并且** 打印包含 `openspec show <item>` 或 `openspec change/spec show` 示例的有用提示
- **并且** 以代码 1 退出

#### 场景：直接项目显示

- **当** 执行 `openspec show <item-name>` 时
- **那么** 自动检测项目是变更还是规格
- **并且** 显示项目内容
- **并且** 根据项目类型使用适当的格式

#### 场景：类型检测和歧义处理

- **当** 执行 `openspec show <item-name>` 时
- **那么** 如果 `<item-name>` 唯一匹配变更或规格，则显示该项目
- **并且** 如果同时匹配两者，打印歧义错误并建议使用 `--type change|spec` 或使用 `openspec change show`/`openspec spec show`
- **并且** 如果都不匹配，打印未找到并提供最近匹配建议

#### 场景：显式类型覆盖

- **当** 执行 `openspec show --type change <item>` 时
- **那么** 将 `<item>` 作为变更 ID 处理并显示（跳过自动检测）

- **当** 执行 `openspec show --type spec <item>` 时
- **那么** 将 `<item>` 作为规格 ID 处理并显示（跳过自动检测）

### 需求：输出格式选项

show 命令应支持与现有命令一致的多种输出格式。

#### 场景：JSON 输出

- **当** 执行 `openspec show <item> --json` 时
- **那么** 以 JSON 格式输出项目
- **并且** 包含解析的元数据和结构
- **并且** 与现有的 change/spec show 命令保持格式一致

#### 场景：标志作用域和委托

- **当** 通过顶级命令显示变更或规格时
- **那么** 接受通用标志，如 `--json`
- **并且** 将类型特定标志传递到相应实现
  - 仅变更标志：`--deltas-only`（别名 `--requirements-only` 已弃用）
  - 仅规格标志：`--requirements`、`--no-scenarios`、`-r/--requirement`
- **并且** 对检测类型无关的标志显示警告并忽略

### 需求：交互控制

- CLI 应尊重 `--no-interactive` 以禁用提示。
- CLI 应尊重 `OPEN_SPEC_INTERACTIVE=0` 以全局禁用提示。
- 仅当 stdin 是 TTY 且交互性未禁用时，才显示交互提示。

#### 场景：变更特定选项

- **当** 使用 `openspec show <change-name> --deltas-only` 显示变更时
- **那么** 仅以 JSON 格式显示增量
- **并且** 与现有的 change show 选项保持兼容性

#### 场景：规格特定选项

- **当** 使用 `openspec show <spec-id> --requirements` 显示规格时
- **那么** 仅以 JSON 格式显示需求
- **并且** 支持其他规格选项（--no-scenarios、-r）
- **并且** 与现有的 spec show 选项保持兼容性