# cli-completion 规范

## 目的
为 OpenSpec CLI 提供 shell 补全脚本，为多个 shell 支持命令、标志和动态值（变更 ID、规格 ID）的制表符补全。支持 Zsh、Bash、Fish 和 PowerShell。

## 需求

### 需求：原生 Shell 行为集成

补全系统应尊重并集成每个支持 shell 的原生补全模式和用户交互模型。

#### 场景：Zsh 原生补全

- **当** 生成 Zsh 补全脚本时
- **那么** 使用 Zsh 补全系统，包括 `_arguments`、`_describe` 和 `compadd`
- **并且** 补全应在单个 TAB 上触发（标准 Zsh 行为）
- **并且** 显示为用户使用 TAB/箭头键导航的交互菜单
- **并且** 自动支持 Oh My Zsh 的增强菜单样式

#### 场景：Bash 原生补全

- **当** 生成 Bash 补全脚本时
- **那么** 使用 Bash 补全，包括 `complete` 内置命令和 `COMPREPLY` 数组
- **并且** 补全应在双 TAB 上触发（标准 Bash 行为）
- **并且** 显示为空格分隔列表或列格式
- **并且** 支持 bash-completion v1 和 v2 模式

#### 场景：Fish 原生补全

- **当** 生成 Fish 补全脚本时
- **那么** 使用 Fish 的 `complete` 命令和条件
- **并且** 补全应在单个 TAB 上触发，并带有自动建议预览
- **并且** 使用 Fish 的原生着色和描述对齐显示
- **并且** 自动利用 Fish 的内置缓存

#### 场景：PowerShell 原生补全

- **当** 生成 PowerShell 补全脚本时
- **那么** 使用 `Register-ArgumentCompleter` 和 scriptblock
- **并且** 补全应在 TAB 上触发并具有循环行为
- **并且** 使用 PowerShell 的原生补全 UI 显示
- **并且** 支持 Windows PowerShell 5.1 和 PowerShell Core 7+

#### 场景：没有自定义 UX 模式

- **当** 为任何 shell 实现补全时
- **那么** 不尝试自定义补全触发行为
- **并且** 不覆盖 shell 特定的导航模式
- **并且** 确保补全对于该 shell 的有经验用户感觉原生

### 需求：命令结构

补全命令应遵循用于生成和管理补全脚本的子命令模式。

#### 场景：可用的子命令

- **当** 用户执行 `openspec completion --help` 时
- **那么** 显示可用的子命令：
  - `generate [shell]` - 为 shell 生成补全脚本（输出到 stdout）
  - `install [shell]` - 为 Zsh 安装补全（自动检测或需要显式 shell）
  - `uninstall [shell]` - 为 Zsh 移除补全（自动检测或需要显式 shell）

### 需求：Shell 检测

补全系统应自动检测用户的当前 shell 环境。

#### 场景：从环境检测 Zsh

- **当** 未显式指定 shell 时
- **那么** 读取 `$SHELL` 环境变量
- **并且** 从路径中提取 shell 名称（例如 `/bin/zsh` → `zsh`）
- **并且** 验证 shell 是以下之一：`zsh`、`bash`、`fish`、`powershell`
- **并且** 如果 shell 不支持则抛出错误

#### 场景：从环境检测 Bash

- **当** `$SHELL` 路径中包含 `bash` 时
- **那么** 检测 shell 为 `bash`
- **并且** 继续使用 bash 特定的补全逻辑

#### 场景：从环境检测 Fish

- **当** `$SHELL` 路径中包含 `fish` 时
- **那么** 检测 shell 为 `fish`
- **并且** 继续使用 fish 特定的补全逻辑

#### 场景：从环境检测 PowerShell

- **当** `$PSModulePath` 环境变量存在时
- **那么** 检测 shell 为 `powershell`
- **并且** 继续使用 PowerShell 特定的补全逻辑

#### 场景：不受支持的 shell 检测

- **当** shell 路径指示不受支持的 shell 时
- **那么** 抛出错误："Shell '<name>' is not supported. Supported shells: zsh, bash, fish, powershell"

### 需求：补全生成

补全命令应按需为所有支持的 shell 生成补全脚本。

#### 场景：生成 Zsh 补全

- **当** 用户执行 `openspec completion generate zsh` 时
- **那么** 将完整的 Zsh 补全脚本输出到 stdout
- **并且** 包含所有命令的补全：init、list、show、validate、archive、view、update、change、spec、completion
- **并且** 包含所有命令特定的标志和选项
- **并且** 使用 Zsh 的 `_arguments` 和 `_describe` 内置函数
- **并且** 支持变更和规格 ID 的动态补全

#### 场景：生成 Bash 补全

- **当** 用户执行 `openspec completion generate bash` 时
- **那么** 将完整的 Bash 补全脚本输出到 stdout
- **并且** 包含所有命令和子命令的补全
- **并且** 使用 `complete -F` 和自定义补全函数
- **并且** 用适当的建议填充 `COMPREPLY`
- **并且** 通过 `openspec __complete` 支持变更和规格 ID 的动态补全

#### 场景：生成 Fish 补全

- **当** 用户执行 `openspec completion generate fish` 时
- **那么** 将完整的 Fish 补全脚本输出到 stdout
- **并且** 使用 `complete -c openspec` 和条件
- **并且** 包含带有 `--condition` 谓词的命令特定补全
- **并且** 通过 `openspec __complete` 支持变更和规格 ID 的动态补全
- **并且** 为每个补全选项包含描述

#### 场景：生成 PowerShell 补全

- **当** 用户执行 `openspec completion generate powershell` 时
- **那么** 将完整的 PowerShell 补全脚本输出到 stdout
- **并且** 使用 `Register-ArgumentCompleter -CommandName openspec`
- **并且** 实现处理命令上下文的 scriptblock
- **并且** 通过 `openspec __complete` 支持变更和规格 ID 的动态补全
- **并且** 返回 `[System.Management.Automation.CompletionResult]` 对象

### 需求：动态补全

补全系统应为项目特定的值提供上下文感知的动态补全。

#### 场景：补全变更 ID

- **当** 为接受变更名称的命令补全参数时（show、validate、archive）
- **那么** 从 `openspec/changes/` 目录发现活动变更
- **并且** 排除 `openspec/changes/archive/` 中的已归档变更
- **并且** 将变更 ID 作为补全建议返回
- **并且** 仅在 OpenSpec 启用项目内提供建议

#### 场景：补全规格 ID

- **当** 为接受规格名称的命令补全参数时（show、validate）
- **那么** 从 `openspec/specs/` 目录发现规格
- **并且** 将规格 ID 作为补全建议返回
- **并且** 仅在 OpenSpec 启用项目内提供建议

#### 场景：补全缓存

- **当** 请求动态补全时
- **那么** 缓存发现的变更和规格 ID 2 秒
- **并且** 在缓存窗口内重用缓存的值
- **并且** 在过期后自动刷新缓存

#### 场景：项目检测

- **当** 用户在 OpenSpec 项目外请求补全时
- **那么** 跳过动态变更/规格 ID 补全
- **并且** 仅建议静态命令和标志

### 需求：安装自动化

补全命令应为所有支持的 shell 自动将补全脚本安装到 shell 配置文件。

#### 场景：为 Oh My Zsh 安装

- **当** 用户执行 `openspec completion install zsh` 时
- **那么** 通过检查 `$ZSH` 环境变量或 `~/.oh-my-zsh/` 目录检测 Oh My Zsh 是否安装
- **并且** 如果不存在则创建自定义补全目录 `~/.oh-my-zsh/custom/completions/`
- **并且** 将补全脚本写入 `~/.oh-my-zsh/custom/completions/_openspec`
- **并且** 如果需要，通过更新 `~/.zshrc` 确保 `~/.oh-my-zsh/custom/completions` 在 `$fpath` 中
- **并且** 显示成功消息，包含运行 `exec zsh` 或重启终端的说明

#### 场景：为标准 Zsh 安装

- **当** 用户执行 `openspec completion install zsh` 且未检测到 Oh My Zsh 时
- **那么** 如果不存在则创建补全目录 `~/.zsh/completions/`
- **并且** 将补全脚本写入 `~/.zsh/completions/_openspec`
- **并且** 如果 `fpath=(~/.zsh/completions $fpath)` 尚不存在，则将其添加到 `~/.zshrc`
- **并且** 如果 `autoload -Uz compinit && compinit` 尚不存在，则将其添加到 `~/.zshrc`
- **并且** 显示成功消息，包含运行 `exec zsh` 或重启终端的说明

#### 场景：为带 bash-completion 的 Bash 安装

- **当** 用户执行 `openspec completion install bash` 时
- **那么** 通过检查 `/usr/share/bash-completion` 或 `/etc/bash_completion.d` 检测 bash-completion 是否安装
- **并且** 如果 bash-completion 可用，写入 `/etc/bash_completion.d/openspec`（使用 sudo）或 `~/.local/share/bash-completion/completions/openspec`
- **并且** 如果 bash-completion 不可用，写入 `~/.bash_completion.d/openspec` 并从 `~/.bashrc` 获取源
- **并且** 如果需要，使用基于标记的更新将源行添加到 `~/.bashrc`
- **并且** 显示成功消息，包含运行 `exec bash` 或重启终端的说明

#### 场景：为 Fish 安装

- **当** 用户执行 `openspec completion install fish` 时
- **那么** 如果不存在则创建 Fish 补全目录 `~/.config/fish/completions/`
- **并且** 将补全脚本写入 `~/.config/fish/completions/openspec.fish`
- **并且** Fish 自动从此目录加载补全（无需修改配置文件）
- **并且** 显示成功消息，指示补全立即可用

#### 场景：为 PowerShell 安装

- **当** 用户执行 `openspec completion install powershell` 时
- **那么** 通过 `$PROFILE` 环境变量或默认路径检测 PowerShell profile 位置
- **并且** 如果不存在则创建 profile 目录
- **并且** 使用基于标记的更新将补全脚本导入添加到 profile
- **并且** 将补全脚本写入 PowerShell 模块目录或与 profile 一起
- **并且** 显示成功消息，包含重启 PowerShell 或运行 `. $PROFILE` 的说明

#### 场景：自动检测 shell 进行安装

- **当** 用户执行 `openspec completion install` 而不指定 shell 时
- **那么** 使用 shell 检测逻辑检测当前 shell
- **并且** 为检测到的 shell 安装补全（zsh、bash、fish 或 powershell）
- **并且** 显示检测到的是哪个 shell

#### 场景：已安装

- **当** 目标 shell 的补全已安装时
- **那么** 显示指示补全已安装的消息
- **并且** 提供通过覆盖现有文件重新安装/更新的选项
- **并且** 以代码 0 退出

### 需求：卸载

补全命令应为所有支持的 shell 移除已安装的补全脚本和配置。

#### 场景：卸载 Zsh 补全

- **当** 用户执行 `openspec completion uninstall zsh` 时
- **那么** 在继续之前提示确认（除非提供 `--yes` 标志）
- **并且** 如果用户拒绝，取消卸载并显示"Uninstall cancelled."
- **并且** 如果用户确认，如果检测到 Oh My Zsh，则移除 `~/.oh-my-zsh/custom/completions/_openspec`
- **并且** 如果检测到标准 Zsh 设置，则移除 `~/.zsh/completions/_openspec`
- **并且** 使用基于标记的移除从 `~/.zshrc` 移除 fpath 修改
- **并且** 显示成功消息

#### 场景：卸载 Bash 补全

- **当** 用户执行 `openspec completion uninstall bash` 时
- **那么** 提示确认（除非提供 `--yes` 标志）
- **并且** 如果用户确认，从 bash-completion 目录或 `~/.bash_completion.d/` 移除补全文件
- **并且** 使用基于标记的移除从 `~/.bashrc` 移除源行
- **并且** 显示成功消息

#### 场景：卸载 Fish 补全

- **当** 用户执行 `openspec completion uninstall fish` 时
- **那么** 提示确认（除非提供 `--yes` 标志）
- **并且** 如果用户确认，移除 `~/.config/fish/completions/openspec.fish`
- **并且** 显示成功消息（无需修改配置文件）

#### 场景：卸载 PowerShell 补全

- **当** 用户执行 `openspec completion uninstall powershell` 时
- **那么** 提示确认（除非提供 `--yes` 标志）
- **并且** 如果用户确认，使用基于标记的移除从 PowerShell profile 移除补全导入
- **并且** 移除补全脚本文件
- **并且** 显示成功消息

#### 场景：自动检测 shell 进行卸载

- **当** 用户执行 `openspec completion uninstall` 而不指定 shell 时
- **那么** 检测当前 shell 并为该 shell 卸载补全

#### 场景：未安装

- **当** 尝试卸载未安装的补全时
- **那么** 显示指示补全未安装的错误消息
- **并且** 以代码 1 退出

### 需求：架构模式

补全实现应遵循干净的架构原则和 TypeScript 最佳实践，通过基于插件的模式支持多个 shell。

#### 场景：特定于 shell 的生成器

- **当** 实现补全生成器时
- **那么** 为每个 shell 创建生成器类：`ZshGenerator`、`BashGenerator`、`FishGenerator`、`PowerShellGenerator`
- **并且** 实现带有方法的公共 `CompletionGenerator` 接口：
  - `generate(commands: CommandDefinition[]): string` - 返回完整的 shell 脚本
- **并且** 每个生成器处理特定于 shell 的语法、转义和模式
- **并且** 所有生成器从命令注册表消费相同的 `CommandDefinition[]`

#### 场景：特定于 shell 的安装程序

- **当** 实现补全安装程序时
- **那么** 为每个 shell 创建安装程序类：`ZshInstaller`、`BashInstaller`、`FishInstaller`、`PowerShellInstaller`
- **并且** 实现带有方法的公共 `CompletionInstaller` 接口：
  - `install(script: string): Promise<InstallationResult>` - 安装补全脚本
  - `uninstall(): Promise<{ success: boolean; message: string }>` - 移除补全
- **并且** 每个安装程序处理特定于 shell 的路径、配置文件和安装模式

#### 场景：shell 选择的工厂模式

- **当** 选择特定于 shell 的实现时
- **那么** 使用 `CompletionFactory` 类及其静态方法：
  - `createGenerator(shell: SupportedShell): CompletionGenerator`
  - `createInstaller(shell: SupportedShell): CompletionInstaller`
- **并且** 工厂使用带有 TypeScript 穷尽性检查的 switch 语句
- **并且** 添加新 shell 需要更新 `SupportedShell` 类型和工厂用例

#### 场景：动态补全提供程序

- **当** 实现动态补全时
- **那么** 创建封装项目发现逻辑的 `CompletionProvider` 类
- **并且** 实现方法：
  - `getChangeIds(): Promise<string[]>` - 发现活动变更 ID
  - `getSpecIds(): Promise<string[]>` - 发现规格 ID
  - `isOpenSpecProject(): boolean` - 检查当前目录是否启用 OpenSpec
- **并且** 使用类属性实现 2 秒 TTL 的缓存

#### 场景：命令注册表

- **当** 定义可补全命令时
- **那么** 创建带有属性集中化的 `CommandDefinition` 类型：
  - `name: string` - 命令名称
  - `description: string` - 帮助文本
  - `flags: FlagDefinition[]` - 可用标志
  - `acceptsPositional: boolean` - 命令是否接受位置参数
  - `positionalType: string` - 位置类型（change-id、spec-id、path、shell）
  - `subcommands?: CommandDefinition[]` - 嵌套子命令
- **并且** 导出包含所有命令定义的 `COMMAND_REGISTRY` 常量
- **并且** 所有生成器消费此注册表以确保跨 shell 的一致性

#### 场景：类型安全的 shell 检测

- **当** 实现 shell 检测时
- **那么** 将 `SupportedShell` 类型定义为字面量类型：`'zsh' | 'bash' | 'fish' | 'powershell'`
- **并且** 在 `src/utils/shell-detection.ts` 中实现 `detectShell()` 函数
- **并且** 返回检测到的 shell 或抛出带有支持 shell 列表的错误

### 需求：错误处理

补全命令应为常见失败场景提供清晰的错误消息。

#### 场景：不受支持的 shell

- **当** 用户请求不受支持的 shell 的补全（例如 ksh、csh、tcsh）时
- **那么** 显示错误消息："Shell '<name>' is not supported yet. Currently supported: zsh, bash, fish, powershell"
- **并且** 以代码 1 退出

#### 场景：安装期间的权限错误

- **当** 由于文件权限问题导致安装失败时
- **那么** 显示指示权限问题的清晰错误消息
- **并且** 建议使用适当的权限或替代安装方法
- **并且** 以代码 1 退出

#### 场景：缺少 shell 配置目录

- **当** 预期的 shell 配置目录不存在时
- **那么** 自动创建目录（并通知用户）
- **并且** 继续安装

#### 场景：未检测到 shell

- **当** `openspec completion install` 无法检测当前 shell 时
- **那么** 显示错误："Could not auto-detect shell. Please specify shell explicitly."
- **并且** 显示使用提示："Usage: openspec completion <operation> [shell]"
- **并且** 以代码 1 退出

### 需求：输出格式

补全命令应提供机器可解析和人类可读的输出。

#### 场景：脚本生成输出

- **当** 将补全脚本生成到 stdout 时
- **那么** 仅输出补全脚本内容（无额外消息）
- **并且** 允许重定向到文件：`openspec completion generate zsh > /path/to/_openspec`

#### 场景：安装成功输出

- **当** 安装成功完成时
- **那么** 显示格式化的成功消息，包含：
  - 复选标记指示器
  - 安装位置
  - Next steps（shell 重新加载说明）
- **并且** 当终端支持时使用颜色（除非设置 `--no-color`）

#### 场景：详细安装输出

- **当** 用户在安装期间提供 `--verbose` 标志时
- **那么** 显示详细步骤：
  - Shell 检测结果
  - 目标文件路径
  - 配置修改
  - 文件创建确认

### 需求：测试支持

补全实现应可通过所有支持 shell 的单元和集成测试进行测试。

#### 场景：模拟 shell 环境

- **当** 为 shell 检测编写测试时
- **那么** 允许覆盖 `$SHELL` 和 `$PSModulePath` 环境变量
- **并且** 对文件系统操作使用依赖注入
- **并且** 独立测试所有四个 shell 的检测

#### 场景：生成器输出验证

- **当** 测试补全生成器时
- **那么** 为每个 shell 生成器（zsh、bash、fish、powershell）创建测试套件
- **并且** 验证生成的脚本包含该 shell 的预期模式
- **并且** 测试命令注册表被正确消费
- **并且** 确保存在动态补全占位符
- **并且** 验证特定于 shell 的语法和转义

#### 场景：安装程序模拟

- **当** 测试安装逻辑时
- **那么** 为每个 shell 安装程序创建测试套件
- **并且** 使用临时测试目录而不是实际的 home 目录
- **并且** 验证文件创建而不修改真实的 shell 配置
- **并且** 独立测试路径解析逻辑
- **并且** 模拟文件系统操作以避免副作用

#### 场景：跨 shell 一致性

- **当** 测试补全行为时
- **那么** 验证所有 shell 支持相同的命令和标志
- **并且** 验证动态补全在 shell 间一致工作
- **并且** 确保错误消息在 shell 间一致