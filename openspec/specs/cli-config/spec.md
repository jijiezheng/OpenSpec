# cli-config 规范

## 目的
提供友好的 CLI 接口，用于查看和修改全局 OpenSpec 配置设置，而无需手动编辑 JSON 文件。

## 需求

### 需求：命令结构

config 命令应为所有配置操作提供子命令。

#### 场景：可用的子命令

- **当** 用户执行 `openspec config --help` 时
- **那么** 显示可用的子命令：
  - `path` - 显示配置文件位置
  - `list` - 显示所有当前设置
  - `get <key>` - 获取特定值
  - `set <key> <value>` - 设置值
  - `unset <key>` - 删除键（恢复默认值）
  - `reset` - 重置配置为默认值
  - `edit` - 在编辑器中打开配置

### 需求：配置路径

config 命令应显示配置文件位置。

#### 场景：显示配置路径

- **当** 用户执行 `openspec config path` 时
- **那么** 打印配置文件的绝对路径
- **并且** 以代码 0 退出

### 需求：配置列表

config 命令应显示所有当前配置值。

#### 场景：以人类可读的格式列出配置

- **当** 用户执行 `openspec config list` 时
- **那么** 以 YAML 格式显示所有配置值
- **并且** 用缩进显示嵌套对象

#### 场景：以 JSON 列出配置

- **当** 用户执行 `openspec config list --json` 时
- **那么** 将完整配置作为有效 JSON 输出
- **并且** 仅输出 JSON（无其他文本）

### 需求：配置获取

config 命令应检索特定的配置值。

#### 场景：获取顶级键

- **当** 用户使用有效的顶级键执行 `openspec config get <key>` 时
- **那么** 仅打印原始值（无标签或格式）
- **并且** 以代码 0 退出

#### 场景：使用点号表示法获取嵌套键

- **当** 用户执行 `openspec config get featureFlags.someFlag` 时
- **那么** 使用点号表示法遍历嵌套结构
- **并且** 打印该路径的值

#### 场景：获取不存在的键

- **当** 用户使用不存在的键执行 `openspec config get <key>` 时
- **那么** 不打印任何内容（空输出）
- **并且** 以代码 1 退出

#### 场景：获取对象值

- **当** 用户执行的值是对象的 `openspec config get <key>` 时
- **那么** 将对象打印为 JSON

### 需求：配置设置

config 命令应以自动类型转换方式设置配置值。

#### 场景：设置字符串值

- **当** 用户执行 `openspec config set <key> <value>` 时
- **并且** 值不匹配布尔值或数字模式
- **那么** 将值存储为字符串
- **并且** 显示确认消息

#### 场景：设置布尔值

- **当** 用户执行 `openspec config set <key> true` 或 `openspec config set <key> false` 时
- **那么** 将值存储为布尔值（不是字符串）
- **并且** 显示确认消息

#### 场景：设置数值

- **当** 用户执行 `openspec config set <key> <value>` 时
- **并且** 值是有效的数字（整数或浮点数）
- **那么** 将值存储为数字（不是字符串）

#### 场景：使用 --string 标志强制字符串

- **当** 用户执行 `openspec config set <key> <value> --string` 时
- **那么** 无论内容如何都将值存储为字符串
- **并且** 这允许将字面量"true"或"123"存储为字符串

#### 场景：设置嵌套键

- **当** 用户执行 `openspec config set featureFlags.newFlag true` 时
- **那么** 如果不存在则创建中间对象
- **并且** 在嵌套路径设置值

### 需求：配置取消设置

config 命令应删除配置覆盖。

#### 场景：取消设置现有键

- **当** 用户执行 `openspec config unset <key>` 时
- **并且** 键存在于配置中
- **那么** 从配置文件中删除该键
- **并且** 该值恢复为其默认值
- **并且** 显示确认消息

#### 场景：取消设置不存在的键

- **当** 用户执行 `openspec config unset <key>` 时
- **并且** 键不存在于配置中
- **那么** 显示指示键未设置的消息
- **并且** 以代码 0 退出

### 需求：配置重置

config 命令应将配置重置为默认值。

#### 场景：带确认的重置全部

- **当** 用户执行 `openspec config reset --all` 时
- **那么** 在继续之前提示确认
- **并且** 如果确认，删除配置文件或重置为默认值
- **并且** 显示确认消息

#### 场景：使用 -y 标志重置全部

- **当** 用户执行 `openspec config reset --all -y` 时
- **那么** 不提示确认直接重置

#### 场景：没有 --all 标志的重置

- **当** 用户执行 `openspec config reset` 而不提供 `--all` 时
- **那么** 显示错误，指示需要 `--all`
- **并且** 以代码 1 退出

### 需求：配置编辑

config 命令应在用户的编辑器中打开配置文件。

#### 场景：成功打开编辑器

- **当** 用户执行 `openspec config edit` 时
- **并且** 设置了 `$EDITOR` 或 `$VISUAL` 环境变量
- **那么** 在该编辑器中打开配置文件
- **并且** 如果不存在则使用默认值创建配置文件
- **并且** 在返回之前等待编辑器关闭

#### 场景：未配置编辑器

- **当** 用户执行 `openspec config edit` 时
- **并且** 既未设置 `$EDITOR` 也未设置 `$VISUAL`
- **那么** 显示错误消息，建议设置 `$EDITOR`
- **并且** 以代码 1 退出

### 需求：Profile 配置流程

`openspec config profile` 命令应提供动作优先的交互流程，允许用户独立修改传递和工作流设置。

#### 场景：首先显示当前 profile 摘要

- **当** 用户在交互式终端中运行 `openspec config profile` 时
- **那么** 显示带有以下内容的当前状态标题：
  - 当前传递值
  - 带 profile 标签的工作流计数（core 或 custom）

#### 场景：动作优先菜单提供可跳过路径

- **当** 用户以交互方式运行 `openspec config profile` 时
- **那么** 第一个提示应提供：
  - `更改传递 + 工作流`
  - `仅更改传递`
  - `仅更改工作流`
  - `保持当前设置（退出）`

#### 场景：传递提示标记当前选择

- **当** 在 `openspec config profile` 中显示传递选择时
- **那么** 当前配置的传递选项应在标签中包含 `[current]`
- **并且** 该值应默认预选

#### 场景：无操作退出而不保存或应用提示

- **当** 用户选择`保持当前设置（退出）`或做出不改变有效配置值的选择时
- **那么** 命令应打印 `No config changes.`
- **并且** 不应写入配置更改
- **并且** 不应询问是否将更新应用到当前项目

#### 场景：当前项目不同步时无操作警告

- **当** `openspec config profile` 在 OpenSpec 项目内以 `No config changes.` 退出时
- **并且** 项目文件与当前全局 profile/传递不同步
- **那么** 显示非阻塞警告，提示全局配置尚未应用于此项目
- **并且** 包含运行 `openspec update` 来同步项目文件的指导

#### 场景：应用提示受实际更改限制

- **当** 配置值已被更改并保存时
- **并且** 当前目录是 OpenSpec 项目
- **那么** 提示 `立即将此更改应用到该项目？`
- **并且** 如果确认，为当前项目运行 `openspec update`

### 需求：键命名约定

config 命令应使用与 JSON 结构匹配的 camelCase 键。

#### 场景：键与 JSON 结构匹配

- **当** 通过 CLI 访问配置键时
- **那么** 使用与实际 JSON 属性名匹配的 camelCase
- **并且** 支持嵌套访问的点号表示法（例如 `featureFlags.someFlag`）

### 需求：Schema 验证

config 命令应使用 zod 对配置写入进行 schema 验证，同时拒绝 `config set` 的未知键，除非明确覆盖。

#### 场景：默认拒绝未知键

- **当** 用户执行 `openspec config set someFutureKey 123` 时
- **那么** 显示描述该键无效的错误消息
- **并且** 不修改配置文件
- **并且** 以代码 1 退出

#### 场景：使用覆盖接受未知键

- **当** 用户执行 `openspec config set someFutureKey 123 --allow-unknown` 时
- **那么** 值成功保存
- **并且** 以代码 0 退出

#### 场景：拒绝无效的功能标志值

- **当** 用户执行 `openspec config set featureFlags.someFlag notABoolean` 时
- **那么** 显示描述性错误消息
- **并且** 不修改配置文件
- **并且** 以代码 1 退出

### 需求：保留的 Scope 标志

config 命令应为未来可扩展性保留 `--scope` 标志。

#### 场景：Scope 标志默认为全局

- **当** 用户执行任何 config 命令而不提供 `--scope` 时
- **那么** 对全局配置进行操作（默认行为）

#### 场景：项目 scope 尚未实现

- **当** 用户执行 `openspec config --scope project <subcommand>` 时
- **那么** 显示错误消息："Project-local config is not yet implemented"
- **并且** 以代码 1 退出