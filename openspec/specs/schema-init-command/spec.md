# schema-init-command 规范

## 目的
定义 `openspec schema init` 命令的行为，用于在交互式和非交互式模式下创建项目本地的 schema 骨架。

## 需求

### 需求：Schema init 命令创建项目本地的 schema
CLI 应提供 `openspec schema init <name>` 命令，在 `openspec/schemas/<name>/` 下创建新的 schema 目录，包含有效的 `schema.yaml` 文件和默认模板文件。

#### 场景：使用有效名称创建 schema
- **当** 用户运行 `openspec schema init my-workflow`
- **那么** 系统创建目录 `openspec/schemas/my-workflow/`
- **并且** 创建包含 name、version、description 和 artifacts 数组的 `schema.yaml`
- **并且** 创建 artifact 引用的模板文件
- **并且** 显示包含创建路径的成功消息

#### 场景：拒绝无效的 schema 名称
- **当** 用户运行 `openspec schema init "My Workflow"`（包含空格）
- **那么** 系统显示关于无效 schema 名称的错误
- **并且** 建议使用 kebab-case 格式
- **并且** 以非零代码退出

#### 场景：Schema 名称已存在
- **当** 用户运行 `openspec schema init existing-schema` 且 `openspec/schemas/existing-schema/` 已存在
- **那么** 系统显示 schema 已存在的错误
- **并且** 建议使用 `--force` 覆盖或使用 `schema fork` 复制
- **并且** 以非零代码退出

### 需求：Schema init 支持交互式模式
在交互式终端中运行且没有显式标志时，CLI 应提示 schema 配置。

#### 场景：交互式提示输入 description
- **当** 用户在交互式终端中运行 `openspec schema init my-workflow`
- **那么** 系统提示输入 schema description
- **并且** 在生成的 `schema.yaml` 中使用提供的 description

#### 场景：交互式提示选择 artifact
- **当** 用户在交互式终端中运行 `openspec schema init my-workflow`
- **那么** 系统显示带有常用 artifact（proposal、specs、design、tasks）的多选提示
- **并且** 每个选项包含简要说明
- **并且** 在生成的 `schema.yaml` 中使用选中的 artifact

#### 场景：使用标志的非交互式模式
- **当** 用户运行 `openspec schema init my-workflow --description "My workflow" --artifacts proposal,tasks`
- **那么** 系统不提示直接创建 schema
- **并且** 使用标志值进行配置

### 需求：Schema init 支持设置项目默认值
CLI 应提供将新创建的 schema 设置为项目默认值的选项。

#### 场景：交互式设置为默认值
- **当** 用户在交互模式下运行 `openspec schema init my-workflow`
- **并且** 用户确认设置为默认值
- **那么** 系统使用 `defaultSchema: my-workflow` 更新 `openspec/config.yaml`

#### 场景：通过标志设置为默认值
- **当** 用户运行 `openspec schema init my-workflow --default`
- **那么** 系统创建 schema 并使用 `defaultSchema: my-workflow` 更新 `openspec/config.yaml`

#### 场景：跳过设置默认值
- **当** 用户运行 `openspec schema init my-workflow --no-default`
- **那么** 系统创建 schema 但不修改 `openspec/config.yaml`

### 需求：Schema init 输出 JSON 格式
CLI 应支持 `--json` 标志以提供机器可读的输出。

#### 场景：成功时输出 JSON
- **当** 用户运行 `openspec schema init my-workflow --json --description "Test" --artifacts proposal`
- **那么** 系统输出包含 `created: true`、`path` 和 `schema` 字段的 JSON
- **并且** 不显示交互提示或旋转动画

#### 场景：错误时输出 JSON
- **当** 用户运行 `openspec schema init "invalid name" --json`
- **那么** 系统输出包含描述问题的 `error` 字段的 JSON
- **并且** 以非零代码退出