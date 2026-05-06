# schema-which-command 规范

## 目的
定义 `openspec schema which` 行为，用于报告解析后的 schema 源、位置和回退详情。

## 需求

### 需求：Schema which 显示解析结果
CLI 应提供 `openspec schema which <name>` 命令，显示 schema 从何处解析。

#### 场景：Schema 从项目解析
- **当** 用户运行 `openspec schema which my-workflow` 且 schema 存在于 `openspec/schemas/my-workflow/` 中时
- **那么** 系统显示源为"project"
- **并且** 显示 schema 目录的完整路径

#### 场景：Schema 从用户目录解析
- **当** 用户运行 `openspec schema which my-workflow` 且 schema 仅存在于用户数据目录中时
- **那么** 系统显示源为"user"
- **并且** 显示包含 XDG 数据目录的完整路径

#### 场景：Schema 从包解析
- **当** 用户运行 `openspec schema which spec-driven` 且不存在覆盖时
- **那么** 系统显示源为"package"
- **并且** 显示包 schemas 目录的完整路径

#### 场景：Schema 未找到
- **当** 用户运行 `openspec schema which nonexistent` 时
- **那么** 系统显示未找到 schema 的错误
- **并且** 列出可用的 schema
- **并且** 退出并返回非零代码

### 需求：Schema which 显示遮蔽信息
CLI 应指示 schema 何时遮蔽较低优先级级别的另一个 schema。

#### 场景：项目 schema 遮蔽包
- **当** 用户运行 `openspec schema which spec-driven` 且项目和包都有 `spec-driven` 时
- **那么** 系统显示项目 schema 处于活动状态
- **并且** 指示它遮蔽了包版本
- **并且** 显示被遮蔽的包 schema 的路径

#### 场景：无遮蔽
- **当** schema 仅存在于一个位置时
- **那么** 系统不显示遮蔽信息

#### 场景：多个遮蔽
- **当** 项目 schema 同时遮蔽用户和包 schema 时
- **那么** 系统按优先级顺序列出所有被遮蔽的位置

### 需求：Schema which 输出 JSON 格式
CLI 应支持 `--json` 标志用于机器可读的输出。

#### 场景：JSON 输出基本
- **当** 用户运行 `openspec schema which spec-driven --json` 时
- **那么** 系统输出带有 `name`、`source` 和 `path` 字段的 JSON

#### 场景：带遮蔽的 JSON 输出
- **当** 用户运行 `openspec schema which spec-driven --json` 且 schema 有遮蔽时
- **那么** JSON 包含 `shadows` 数组，每个被遮蔽的 schema 有 `source` 和 `path`

### 需求：Schema which 支持列表模式
CLI 应支持列出所有 schema 及其解析源。

#### 场景：列出所有 schema
- **当** 用户运行 `openspec schema which --all` 时
- **那么** 系统显示所有可用的 schema，按源分组
- **并且** 指示哪些 schema 遮蔽其他

#### 场景：以 JSON 格式列出
- **当** 用户运行 `openspec schema which --all --json` 时
- **那么** 系统输出带有每个 schema 解析信息的 JSON 数组