# cli-spec 规范

## 目的
定义 `openspec spec` 命令的行为，用于列出、显示和验证真实性规格。

## 需求

### 需求：交互式规格显示

spec show 命令应在未提供 spec-id 时支持交互式选择。

#### 场景：show 的交互式规格选择

- **当** 执行 `openspec spec show` 不带参数时
- **那么** 显示可用规格的交互列表
- **并且** 允许用户选择要显示的规格
- **并且** 显示所选规格内容
- **并且** 保持所有现有 show 选项（--json、--requirements、--no-scenarios、-r）

#### 场景：非交互回退保持当前行为

- **假设** stdin 不是 TTY 或提供了 `--no-interactive` 或环境变量 `OPEN_SPEC_INTERACTIVE=0`
- **当** 执行 `openspec spec show` 而不提供 spec-id 时
- **那么** 不进行交互式提示
- **并且** 打印缺少 spec-id 的现有错误消息
- **并且** 设置非零退出代码

### 需求：Spec 命令

系统应提供 `spec` 命令，包含用于显示、列出和验证规格的子命令。

#### 场景：将规格显示为 JSON

- **当** 执行 `openspec spec show init --json` 时
- **那么** 解析 markdown 规格文件
- **并且** 提取标题和内容层次结构
- **并且** 将有效 JSON 输出到 stdout

#### 场景：列出所有规格

- **当** 执行 `openspec spec list` 时
- **那么** 扫描 openspec/specs 目录
- **并且** 返回所有可用规格的列表
- **并且** 支持使用 `--json` 标志的 JSON 输出

#### 场景：过滤规格内容

- **当** 执行 `openspec spec show init --requirements` 时
- **那么** 仅显示需求名称和 SHALL 语句
- **并且** 排除场景内容

#### 场景：验证规格结构

- **当** 执行 `openspec spec validate init` 时
- **那么** 解析规格文件
- **并且** 根据 Zod schema 验证
- **并且** 报告任何结构问题

### 需求：JSON Schema 定义

系统应定义 Zod schemas，准确表示用于运行时验证的规格结构。

#### 场景：Schema 验证

- **当** 将规格解析为 JSON 时
- **那么** 使用 Zod schemas 验证结构
- **并且** 确保存在所有必填字段
- **并且** 为验证失败提供清晰的错误消息

### 需求：交互式规格验证

spec validate 命令应在未提供 spec-id 时支持交互式选择。

#### 场景：验证的交互式规格选择

- **当** 执行 `openspec spec validate` 不带参数时
- **那么** 显示可用规格的交互列表
- **并且** 允许用户选择要验证的规格
- **并且** 验证所选规格
- **并且** 保持所有现有验证选项（--strict、--json）

#### 场景：非交互回退保持当前行为

- **假设** stdin 不是 TTY 或提供了 `--no-interactive` 或环境变量 `OPEN_SPEC_INTERACTIVE=0`
- **当** 执行 `openspec spec validate` 而不提供 spec-id 时
- **那么** 不进行交互式提示
- **并且** 打印缺少 spec-id 的现有错误消息
- **并且** 设置非零退出代码