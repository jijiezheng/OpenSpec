# schema-validate-command 规范

## 目的
定义 `openspec schema validate` 行为，用于验证 schema 语法、结构、模板和依赖图。

## 需求

### 需求：Schema validate 检查 schema 结构
CLI 应提供 `openspec schema validate [name]` 命令，验证 schema 配置并报告错误。

#### 场景：验证特定 schema
- **当** 用户运行 `openspec schema validate my-workflow` 时
- **那么** 系统使用解析顺序定位 schema
- **并且** 根据 schema Zod 类型验证 `schema.yaml`
- **并且** 显示验证结果（有效或错误列表）

#### 场景：验证所有项目 schema
- **当** 用户不带名称运行 `openspec schema validate` 时
- **那么** 系统验证 `openspec/schemas/` 中的所有 schema
- **并且** 显示每个 schema 的结果
- **并且** 如果任何 schema 无效则退出并返回非零代码

#### 场景：Schema 未找到
- **当** 用户运行 `openspec schema validate nonexistent` 时
- **那么** 系统显示未找到 schema 的错误
- **并且** 退出并返回非零代码

### 需求：Schema validate 检查 YAML 语法
CLI 应在可能时报告带有行号的 YAML 解析错误。

#### 场景：无效的 YAML 语法
- **当** 用户运行 `openspec schema validate my-workflow` 且 `schema.yaml` 有语法错误时
- **那么** 系统显示带有行号的 YAML 解析错误
- **并且** 退出并返回非零代码

#### 场景：有效的 YAML 但缺少必需字段
- **当** `schema.yaml` 是有效的 YAML 但缺少 `name` 字段时
- **那么** 系统显示缺少必需字段的 Zod 验证错误
- **并且** 识别具体缺失的字段

### 需求：Schema validate 检查模板存在
CLI 应验证产物引用的所有模板文件是否存在。

#### 场景：缺少模板文件
- **当** 产物引用 `template: proposal.md` 但文件在 schema 目录中不存在时
- **那么** 系统报告错误："Template file 'proposal.md' not found for artifact 'proposal'"
- **并且** 退出并返回非零代码

#### 场景：所有模板存在
- **当** 所有产物模板存在时
- **那么** 系统报告模板有效
- **并且** 模板存在包含在验证摘要中

### 需求：Schema validate 检查依赖图
CLI 应验证产物依赖是否形成有效的有向无环图。

#### 场景：有效的依赖图
- **当** 产物依赖形成有效的 DAG（例如 tasks → specs → proposal）时
- **那么** 系统报告依赖图有效

#### 场景：检测到循环依赖
- **当** 产物 A 需要 B 且产物 B 需要 A 时
- **那么** 系统报告循环依赖错误
- **并且** 识别循环中涉及的具体产物
- **并且** 退出并返回非零代码

#### 场景：未知依赖引用
- **当** 产物需要 `nonexistent-artifact` 时
- **那么** 系统报告错误："Artifact 'x' requires unknown artifact 'nonexistent-artifact'"
- **并且** 退出并返回非零代码

### 需求：Schema validate 输出 JSON 格式
CLI 应支持 `--json` 标志用于机器可读的验证结果。

#### 场景：有效 schema 的 JSON 输出
- **当** 用户运行 `openspec schema validate my-workflow --json` 且 schema 有效时
- **那么** 系统输出带有 `valid: true`、`name` 和 `path` 字段的 JSON

#### 场景：无效 schema 的 JSON 输出
- **当** 用户运行 `openspec schema validate my-workflow --json` 且 schema 有错误时
- **那么** 系统输出带有 `valid: false` 和 `issues` 数组的 JSON
- **并且** 每个问题包含 `level`、`path` 和 `message` 字段
- **并且** 格式匹配现有 `openspec validate` 输出结构

### 需求：Schema validate 支持详细模式
CLI 应支持 `--verbose` 标志以获取详细的验证信息。

#### 场景：详细输出显示所有检查
- **当** 用户运行 `openspec schema validate my-workflow --verbose` 时
- **那么** 系统在运行时显示每个验证检查
- **并且** 显示 YAML 解析、Zod 验证、模板存在、依赖图 的通过/失败状态