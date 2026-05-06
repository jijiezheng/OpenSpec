# schema-resolution 规范

## 目的
定义项目本地的 schema 解析行为，包括优先级顺序（项目本地、然后用户覆盖、然后包内置），以及当未提供 `projectRoot` 时的向后兼容回退。

## 需求

### 需求：项目本地 schema 解析

当提供 `projectRoot` 时，系统应从项目本地目录（`./openspec/schemas/<name>/`）以最高优先级解析 schema。

#### 场景：项目本地 schema 优先于用户覆盖

- **当** 名为 "my-workflow" 的 schema 存在于 `./openspec/schemas/my-workflow/schema.yaml`
- **并且** 名为 "my-workflow" 的 schema 存在于 `~/.local/share/openspec/schemas/my-workflow/schema.yaml`
- **并且** 调用 `getSchemaDir("my-workflow", projectRoot)` 时
- **那么** 系统应返回项目本地路径

#### 场景：项目本地 schema 优先于包内置

- **当** 名为 "spec-driven" 的 schema 存在于 `./openspec/schemas/spec-driven/schema.yaml`
- **并且** "spec-driven" 是包内置 schema
- **并且** 调用 `getSchemaDir("spec-driven", projectRoot)` 时
- **那么** 系统应返回项目本地路径

#### 场景：当没有项目本地 schema 时回退到用户覆盖

- **当** `./openspec/schemas/my-workflow/` 中不存在名为 "my-workflow" 的 schema
- **并且** 名为 "my-workflow" 的 schema 存在于 `~/.local/share/openspec/schemas/my-workflow/schema.yaml`
- **并且** 调用 `getSchemaDir("my-workflow", projectRoot)` 时
- **那么** 系统应返回用户覆盖路径

#### 场景：当没有项目本地或用户 schema 时回退到包内置

- **当** `./openspec/schemas/spec-driven/` 中不存在名为 "spec-driven" 的 schema
- **并且** `~/.local/share/openspec/schemas/spec-driven/` 中也不存在名为 "spec-driven" 的 schema
- **并且** "spec-driven" 是包内置 schema
- **并且** 调用 `getSchemaDir("spec-driven", projectRoot)` 时
- **那么** 系统应返回包内置路径

#### 场景：当未提供 projectRoot 时的向后兼容性

- **当** 调用 `getSchemaDir("my-workflow")` 时不带 `projectRoot` 参数
- **那么** 系统应仅检查用户覆盖和包内置位置
- **并且** 系统不应检查项目本地位置

### 需求：项目 schemas 目录辅助函数

系统应提供 `getProjectSchemasDir(projectRoot)` 函数，返回项目本地的 schemas 目录路径。

#### 场景：返回正确路径

- **当** 调用 `getProjectSchemasDir("/path/to/project")` 时
- **那么** 系统应返回 `/path/to/project/openspec/schemas`

### 需求：列出 schema 时包含项目本地

当提供 `projectRoot` 时，系统应在列出可用 schema 时包含项目本地的 schema。

#### 场景：项目本地 schema 出现在列表中

- **当** 名为 "team-flow" 的 schema 存在于 `./openspec/schemas/team-flow/schema.yaml`
- **并且** 调用 `listSchemas(projectRoot)` 时
- **那么** 返回的列表应包含 "team-flow"

#### 场景：项目本地 schema 在列表中遮蔽同名的用户 schema

- **当** 名为 "custom" 的 schema 同时存在于项目本地和用户覆盖位置
- **并且** 调用 `listSchemas(projectRoot)` 时
- **那么** 返回的列表应仅包含一次 "custom"

#### 场景：listSchemas 的向后兼容性

- **当** 调用 `listSchemas()` 时不带 `projectRoot` 参数
- **那么** 系统应仅包含用户覆盖和包内置 schema

### 需求：Schema 信息包含项目源

系统应在 `listSchemasWithInfo()` 结果中为项目本地 schema 指示 `source: 'project'`。

#### 场景：项目本地 schema 显示项目源

- **当** 名为 "team-flow" 的 schema 存在于 `./openspec/schemas/team-flow/schema.yaml`
- **并且** 调用 `listSchemasWithInfo(projectRoot)` 时
- **那么** "team-flow" 的 schema 信息应具有 `source: 'project'`

#### 场景：用户覆盖 schema 显示用户源

- **当** 名为 "my-custom" 的 schema 仅存在于 `~/.local/share/openspec/schemas/my-custom/`
- **并且** 调用 `listSchemasWithInfo(projectRoot)` 时
- **那么** "my-custom" 的 schema 信息应具有 `source: 'user'`

#### 场景：包内置 schema 显示包源

- **当** "spec-driven" 仅作为包内置存在
- **并且** 调用 `listSchemasWithInfo(projectRoot)` 时
- **那么** "spec-driven" 的 schema 信息应具有 `source: 'package'`

### 需求：Schemas 命令显示源

`openspec schemas` 命令应显示每个 schema 的源。

#### 场景：显示格式包含源

- **当** 用户运行 `openspec schemas` 时
- **那么** 输出应显示每个 schema 及其源标签（project、user 或 package）

### 需求：使用 config schema 作为新变更的默认值

系统在创建新变更时，应使用 `openspec/config.yaml` 中的 schema 字段作为默认值，而不带显式 `--schema` 标志。

#### 场景：创建不带 --schema 标志的变更且配置存在

- **当** 用户运行 `openspec new change foo` 且配置包含 `schema: "tdd"` 时
- **那么** 系统使用 "tdd" schema 创建变更

#### 场景：创建不带 --schema 标志的变更且没有配置

- **当** 用户运行 `openspec new change foo` 且不存在配置文件时
- **那么** 系统使用默认 schema "spec-driven" 创建变更

#### 场景：创建带显式 --schema 标志的变更

- **当** 用户运行 `openspec new change foo --schema custom` 且配置包含 `schema: "tdd"` 时
- **那么** 系统使用 "custom" schema 创建变更（CLI 标志覆盖配置）

### 需求：使用更新后的优先级顺序解析 schema

系统应使用以下优先级顺序解析变更的 schema：CLI 标志、变更元数据、项目配置、硬编码默认值。

#### 场景：提供了 CLI 标志

- **当** 用户运行带 `--schema custom` 的命令时
- **那么** 系统使用 "custom"，无论变更元数据或配置如何

#### 场景：变更元数据指定 schema

- **当** 变更有带 `schema: bound` 的 `.openspec.yaml` 且配置有 `schema: tdd` 时
- **那么** 系统使用变更元数据中的 "bound"

#### 场景：仅有项目配置指定 schema

- **当** 没有 CLI 标志或变更元数据，但配置有 `schema: tdd` 时
- **那么** 系统使用项目配置中的 "tdd"

#### 场景：任何地方都没有指定 schema

- **当** 没有 CLI 标志、变更元数据或项目配置时
- **那么** 系统使用硬编码默认值 "spec-driven"

### 需求：在配置中支持项目本地 schema 名称

系统应允许配置 schema 字段引用 `openspec/schemas/` 中定义的项目本地 schema。

#### 场景：配置引用项目本地 schema

- **当** 配置包含 `schema: "my-workflow"` 且 `openspec/schemas/my-workflow/` 存在时
- **那么** 系统解析到项目本地 schema

#### 场景：配置引用不存在的 schema

- **当** 配置包含 `schema: "nonexistent"` 且该 schema 不存在时
- **那么** 系统在尝试加载 schema 时显示错误，并提供模糊匹配建议和所有有效 schema 列表

### 需求：为无效 schema 提供有用的错误消息

系统应显示带有模糊匹配建议、可用 schema 列表和修复说明的 schema 错误。

#### 场景：Schema 名称有拼写错误（接近匹配）

- **当** 配置包含 `schema: "spce-driven"`（拼写错误）时
- **那么** 错误消息包括 "Did you mean: spec-driven (built-in)" 作为建议

#### 场景：Schema 名称没有接近匹配

- **当** 配置包含 `schema: "completely-wrong"` 时
- **那么** 错误消息显示所有可用的内置和项目本地 schema 列表

#### 场景：错误消息包含修复说明

- **当** 配置引用无效 schema 时
- **那么** 错误消息包括 "Fix: Edit openspec/config.yaml and change 'schema: X' to a valid schema name"

#### 场景：错误区分内置 vs 项目本地 schema

- **当** 错误列出可用 schema 时
- **那么** 输出清楚地将每个标记为 "built-in" 或 "project-local"

### 需求：保持现有变更的向后兼容性

系统应继续使用没有项目配置的现有变更。

#### 场景：没有配置的现有变更

- **当** 变更在配置功能之前创建且不存在配置文件时
- **那么** 系统使用现有逻辑（变更元数据或硬编码默认值）解析 schema

#### 场景：稍后添加配置的现有变更

- **当** 带有现有变更的项目添加了配置文件时
- **那么** 现有变更继续使用 `.openspec.yaml` 中的绑定 schema