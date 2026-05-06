# schema-fork-command 规范

## 目的
定义 `openspec schema fork` 行为，用于将现有 schema 克隆到项目本地 schema，并带有安全的覆盖控制。

## 需求

### 需求：Schema fork 复制现有 schema
CLI 应提供 `openspec schema fork <source> [name]` 命令，将现有 schema 复制到项目的 `openspec/schemas/` 目录。

#### 场景：带显式名称的 Fork
- **当** 用户运行 `openspec schema fork spec-driven my-custom` 时
- **那么** 系统使用解析顺序（项目 → 用户 → 包）定位 `spec-driven` schema
- **并且** 将所有文件复制到 `openspec/schemas/my-custom/`
- **并且** 将 `schema.yaml` 中的 `name` 字段更新为 `my-custom`
- **并且** 显示带有源和目标路径的成功消息

#### 场景：带默认名称的 Fork
- **当** 用户运行 `openspec schema fork spec-driven` 而不指定名称时
- **那么** 系统复制到 `openspec/schemas/spec-driven-custom/`
- **并且** 将 `schema.yaml` 中的 `name` 字段更新为 `spec-driven-custom`

#### 场景：源 schema 未找到
- **当** 用户运行 `openspec schema fork nonexistent` 时
- **那么** 系统显示未找到 schema 的错误
- **并且** 列出可用的 schema
- **并且** 退出并返回非零代码

### 需求：Schema fork 防止意外覆盖
当目标 schema 已存在时，CLI 应要求确认或 `--force` 标志。

#### 场景：目的地存在且无 force
- **当** 用户运行 `openspec schema fork spec-driven my-custom` 且 `openspec/schemas/my-custom/` 存在时
- **那么** 系统显示目的地已存在的错误
- **并且** 建议使用 `--force` 覆盖
- **并且** 退出并返回非零代码

#### 场景：目的地存在且带 force 标志
- **当** 用户运行 `openspec schema fork spec-driven my-custom --force` 且目的地存在时
- **那么** 系统移除现有目标目录
- **并且** 将源 schema 复制到目的地
- **并且** 显示成功消息

#### 场景：覆盖的交互式确认
- **当** 在交互模式下运行 `openspec schema fork spec-driven my-custom` 且目的地存在时
- **那么** 系统提示确认覆盖
- **并且** 根据用户响应继续

### 需求：Schema fork 保留所有 schema 文件
CLI 应复制完整的 schema 目录，包括模板、配置和任何附加文件。

#### 场景：复制包括模板文件
- **当** 用户 fork 带模板文件（例如 `proposal.md`、`design.md`）的 schema 时
- **那么** 所有模板文件被复制到目的地
- **并且** 模板文件内容不变

#### 场景：复制包括嵌套目录
- **当** 用户 fork 带嵌套目录（例如 `templates/specs/`）的 schema 时
- **那么** 嵌套目录结构被保留
- **并且** 所有嵌套文件被复制

### 需求：Schema fork 输出 JSON 格式
CLI 应支持 `--json` 标志用于机器可读的输出。

#### 场景：成功时的 JSON 输出
- **当** 用户运行 `openspec schema fork spec-driven my-custom --json` 时
- **那么** 系统输出带有 `forked: true`、`source`、`destination` 和 `sourcePath` 字段的 JSON

#### 场景：JSON 输出显示源位置
- **当** 用户运行 `openspec schema fork spec-driven --json` 时
- **那么** JSON 输出包含 `sourceLocation` 字段，指示"project"、"user"或"package"