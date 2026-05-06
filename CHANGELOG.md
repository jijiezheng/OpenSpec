# @fission-ai/openspec

## 1.3.1

### 补丁变更

- [#995](https://github.com/Fission-AI/OpenSpec/pull/995) [`d1f3861`](https://github.com/Fission-AI/OpenSpec/commit/d1f3861d9ec694cc924b042b5da01963dcf93137) 感谢 [@TabishB](https://github.com/TabishB)！ - ### Bug 修复

  - **规范 artifact 路径** — 工作流 artifact 路径现在通过原生 `realpath` 解析，因此符号链接和不区分大小写的文件系统在应用和归档期间不再导致路径不匹配。
  - **Glob 应用指令** — 带 glob artifact 输出的应用指令现在可以正确解析，文字 artifact 输出被强制为文件路径。
  - **隐藏的主规格需求** — 嵌套在代码块中或以其他方式隐藏在主规格中的需求现在在验证期间被检测到。
  - **干净的 `--json` 输出** — 当传递 `--json` 时，微调器进度文本不再泄漏到 stderr，因此结合 stdout 和 stderr 的 AI agent 可以可靠地解析 JSON。
  - **防火墙环境中的静默遥测** — PostHog 网络错误现在以 1 秒超时和禁用重试/远程配置被吞掉，因此 OpenSpec 不再在封闭网络中显示 `PostHogFetchNetworkError`。遥测选择退出文档在 README、安装指南和 CLI 参考中更早出现。

## 1.3.0

### 次要变更

- [#952](https://github.com/Fission-AI/OpenSpec/pull/952) [`cce787e`](https://github.com/Fission-AI/OpenSpec/commit/cce787ec4083da2b27781f6786f5ce0002909a7b) 感谢 [@TabishB](https://github.com/TabishB)！ - ### 新功能

  - **Junie 支持** — 为 JetBrains Junie 添加了工具和命令生成
  - **Lingma IDE 支持** — 为 Lingma IDE 添加了配置支持
  - **ForgeCode 支持** — 为 ForgeCode 添加了工具支持
  - **IBM Bob 支持** — 添加了对 IBM Bob 编码助手的支持

  ### Bug 修复

  - **Shell 补全可选** — 补全安装现在是可选的，修复了 PowerShell 编码损坏问题
  - **Copilot 自动检测** — 阻止了来自裸 `.github/` 目录的误报 GitHub Copilot 检测
  - **pi.dev 命令生成** — 修复了命令引用转换和模板参数传递

### 补丁变更

- [#760](https://github.com/Fission-AI/OpenSpec/pull/760) [`61eb999`](https://github.com/Fission-AI/OpenSpec/commit/61eb999f7c6c0fc98d2e7f3678756fce6a3f4378) 感谢 [@fsilvaortiz](https://github.com/fsilvaortiz)！ - fix: OpenCode 适配器现在使用 `.opencode/commands/`（复数）来匹配 OpenCode 的官方目录约定。修复了 #748。

- [#759](https://github.com/Fission-AI/OpenSpec/pull/759) [`afdca0d`](https://github.com/Fission-AI/OpenSpec/commit/afdca0d5dab1aa109cfd8848b2512333ccad60c3) 感谢 [@fsilvaortiz](https://github.com/fsilvaortiz)！ - fix: `openspec status` 现在在没有变更存在时优雅退出，而不是抛出致命错误。修复了 #714。

## 1.2.0

### 次要变更

- [#747](https://github.com/Fission-AI/OpenSpec/pull/747) [`1e94443`](https://github.com/Fission-AI/OpenSpec/commit/1e94443a3551b228eecbc89e95d96d3b9600a192) 感谢 [@TabishB](https://github.com/TabishB)！ - ### 新功能

  - **Profile 系统** — 选择 `core`（4 个基本工作流）和 `custom`（选择任何子集）配置文件来控制安装哪些技能。使用新的 `openspec config profile` 命令管理配置文件
  - **Propose 工作流** — 新的一步式工作流从单个请求创建完整的变更提案、设计、specs 和 tasks — 无需分别运行 `new` 然后 `ff`
  - **AI 工具自动检测** — `openspec init` 现在扫描你的项目以查找现有工具目录（`.claude/`、`.cursor/` 等）并预选检测到的工具
  - **Pi (pi.dev) 支持** — Pi 编码 agent 现在是支持的工具，具有 prompt 和技能生成
  - **Kiro 支持** — AWS Kiro IDE 现在是支持的工具，具有 prompt 和技能生成
  - **Sync 修剪取消选择的workflows** — `openspec update` 现在删除你取消选择的workflows 的命令文件和技能目录，保持你的项目干净
  - **配置漂移警告** — `openspec config list` 在全局配置与当前项目不同步时发出警告

  ### Bug 修复

  - 修复了 onboard 预检在全新初始化的项目上给出虚假的"未初始化"错误
  - 修复了归档工作流在同步期间中途停止的问题 — 现在在同步完成后正确恢复
  - 为 onboard shell 命令添加了 Windows PowerShell 替代方案

## 1.1.1

### 补丁变更

- [#627](https://github.com/Fission-AI/OpenSpec/pull/627) [`afb73cf`](https://github.com/Fission-AI/OpenSpec/commit/afb73cf9ec59c6f8b26d0c538c0218c203ba3c56) 感谢 [@TabishB](https://github.com/TabishB)！ - ### Bug 修复

  - **OpenCode 命令引用** — 生成文件中的命令引用现在使用正确的 `/opsx-` 连字符格式而不是 `/opsx:` 冒号格式，确保命令在 OpenCode 中正常工作

## 1.1.0

### 次要变更

- [#625](https://github.com/Fission-AI/OpenSpec/pull/625) [`53081fb`](https://github.com/Fission-AI/OpenSpec/commit/53081fb2a26ec66d2950ae0474b9a56cbc5b5a76) 感谢 [@TabishB](https://github.com/TabishB)！ - ### Bug 修复

  - **Codex 全局路径支持** — Codex 适配器现在正确解析全局路径，修复了在场外项目目录外运行时的 workflow 文件生成（#622）
  - **跨设备或受限路径上的归档操作** — 当重命名因 EPERM 或 EXDEV 错误失败时，归档现在回退到复制+删除，修复了网络/外部驱动器上的失败（#605）
  - **工作流消息中的斜杠命令提示** — 工作流完成消息现在显示有用的斜杠命令提示作为下一步（#603）
  - **Windsurf workflow 文件路径** — 更新了 Windsurf 适配器以使用正确的 `workflows` 目录而不是遗留的 `commands` 路径（#610）

### 补丁变更

- [#550](https://github.com/Fission-AI/OpenSpec/pull/550) [`86d2e04`](https://github.com/Fission-AI/OpenSpec/commit/86d2e04cae76a999dbd1b4571f52fa720036be0c) 感谢 [@jerome-benoit](https://github.com/jerome-benoit)！ - ### 改进

  - **Nix flake 维护** — 版本现在从 package.json 动态读取，减少了手动同步问题
  - **Nix 构建优化** — 源过滤排除 node_modules 和 artifacts，改善了构建时间
  - **update-flake.sh 脚本** — 检测哈希是否已经正确，跳过不必要的重建

  ### 其他

  - 更新了 Nix CI actions 到最新版本（nix-installer v21, magic-nix-cache v13）

## 1.0.2

### 补丁变更

- [#596](https://github.com/Fission-AI/OpenSpec/pull/596) [`e91568d`](https://github.com/Fission-AI/OpenSpec/commit/e91568deb948073f3e9d9bb2d2ab5bf8080d6cf4) 感谢 [@TabishB](https://github.com/TabishB)！ - ### Bug 修复

  - 明确了规范命名约定 — Specs 应该以能力命名（`specs/<capability>/spec.md`），而不是变更
  - 修复了任务复选框格式指南 — 任务现在明确要求 `- [ ]` 复选框格式用于应用阶段跟踪

## 1.0.1

### 补丁变更

- [#587](https://github.com/Fission-AI/OpenSpec/pull/587) [`943e0d4`](https://github.com/Fission-AI/OpenSpec/commit/943e0d41026d034de66b9442d1276c01b293eb2b) 感谢 [@TabishB](https://github.com/TabishB)！ - ### Bug 修复

  - 修复了 onboard 文档中不正确的归档路径 — 模板现在显示正确路径 `openspec/changes/archive/YYYY-MM-DD-<name>/` 而不是错误的 `openspec/archive/YYYY-MM-DD--<name>/`

## 1.0.0

### 主要变更

- [#578](https://github.com/Fission-AI/OpenSpec/pull/578) [`0cc9d90`](https://github.com/Fission-AI/OpenSpec/commit/0cc9d9025af367faa1688a7b2606a2549053cd3f) 感谢 [@TabishB](https://github.com/TabishB)！ - ## OpenSpec 1.0 — OPSX 发布

  工作流从零开始重建。OPSX 用基于动作的系统取代了旧的阶段锁定 `/openspec:*` 命令，其中 AI 了解存在哪些 artifact、什么准备好创建，以及每个动作解锁了什么。

  ### 破坏性变更

  - **旧命令已移除** — `/openspec:proposal`、`/openspec:apply` 和 `/openspec:archive` 不再存在
  - **配置文件已移除** — 工具特定的指令文件（`CLAUDE.md`、`.cursorrules`、`AGENTS.md`、`project.md`）不再生成
  - **迁移** — 运行 `openspec init` 升级。遗留 artifact 被检测到并在确认后清理。

  ### 从静态提示到动态指令

  **之前：** AI 每次都收到相同的静态指令，无论项目状态如何。

  **现在：** 指令从三层动态组装：

  1. **上下文** — 来自 `config.yaml` 的项目背景（技术栈、约定）
  2. **规则** — Artifact 特定约束（例如，"为未知事物提出 spike 任务"）
  3. **模板** — 输出文件的实际结构

  AI 查询 CLI 以获取实时状态：存在哪些 artifact、什么准备好创建、满足哪些依赖、每个动作解锁什么。

  ### 从阶段锁定到基于动作

  **之前：** 线性工作流 — proposal → apply → archive。不能轻易返回或迭代。

  **现在：** 对变更的灵活动作。随时编辑任何 artifact。Artifact 图自动跟踪状态。

  | 命令 | 功能 |
  | -------------------- | ---------------------------------------------------- |
  | `/opsx:explore` | 在承诺变更之前思考想法 |
  | `/opsx:new` | 开始一个新变更 |
  | `/opsx:continue` | 一次创建一个 artifact（逐步） |
  | `/opsx:ff` | 一次创建所有规划 artifact（快进） |
  | `/opsx:apply` | 实现任务 |
  | `/opsx:verify` | 验证实现匹配 artifact |
  | `/opsx:sync` | 同步增量 specs 到主 specs |
  | `/opsx:archive` | 归档已完成的变更 |
  | `/opsx:bulk-archive` | 带冲突检测的批量归档多个变更 |
  | `/opsx:onboard` | 完整工作流 15 分钟引导步行 |

  ### 从文本合并到语义 Spec 同步

  **之前：** Spec 更新需要手动合并或整文件替换。

  **现在：** 增量 specs 使用 AI 理解的语义标记：

  - `## ADDED Requirements` — 要添加的新需求
  - `## MODIFIED Requirements` — 部分更新（添加场景而不复制现有场景）
  - `## REMOVED Requirements` — 删除并提供原因和迁移说明
  - `## RENAMED Requirements` — 重命名保留内容

  归档在需求级别解析这些，而不是脆弱的标题匹配。

  ### 从分散文件到 Agent 技能

  **之前：** 项目根目录有 8+ 个配置文件 + 斜杠命令分散在 21 个工具特定位置，格式不同。

  **现在：** 带有 YAML 前置的 markdown 文件的单一 `.claude/skills/` 目录。由 Claude Code、Cursor、Windsurf 自动检测。跨编辑器兼容。

  ### 新功能

  - **Onboarding 技能** — `/opsx:onboard` 为新用户引导他们完成第一个完整变更，包括基于代码库的任务建议和逐步叙述（11 个阶段，约 15 分钟）

  - **21 个 AI 工具支持** — Claude Code、Cursor、Windsurf、Continue、Gemini CLI、GitHub Copilot、Amazon Q、Cline、RooCode、Kilo Code、Auggie、CodeBuddy、Qoder、Qwen、CoStrict、Crush、Factory、OpenCode、Antigravity、iFlow 和 Codex

  - **交互式设置** — `openspec init` 显示动画欢迎屏幕和可搜索的多选以选择工具。预选已配置的工具以便轻松刷新。

  - **可自定义 schemas** — 在 `openspec/schemas/` 中定义自定义 artifact 工作流，无需触及包代码。团队可以通过版本控制共享工作流。

  ### Bug 修复

  - 修复了当命令名称包含冒号时 Claude Code YAML 解析失败
  - 修复了任务文件解析以处理复选框行上的尾随空格
  - 修复了 JSON 指令输出以将 context/rules 与模板分离 — AI 正在将约束块复制到 artifact 文件中

  ### 文档

  - 新的入门指南、CLI 参考、概念文档
  - 删除了误导性的"飞行中编辑并继续"声明，这些并未实现
  - 添加了从 pre-OPSX 版本升级的迁移指南

## 0.23.0

### 次要变更

- [#540](https://github.com/Fission-AI/OpenSpec/pull/540) [`c4cfdc7`](https://github.com/Fission-AI/OpenSpec/commit/c4cfdc7c499daef30d8a218f5f59b8d9e5adb754) 感谢 [@TabishB](https://github.com/TabishB)！ - ### 新功能

  - **批量归档技能** — 使用 `/opsx:bulk-archive` 在一次操作中归档多个已完成的变更。包括批量验证、spec 冲突检测和合并确认

  ### 其他

  - **简化设置** — 配置创建现在使用带有用注释的有帮助默认值而不是交互式提示

## 0.22.0

### 次要变更

- [#530](https://github.com/Fission-AI/OpenSpec/pull/530) [`33466b1`](https://github.com/Fission-AI/OpenSpec/commit/33466b1e2a6798bdd6d0e19149173585b0612e6f) 感谢 [@TabishB](https://github.com/TabishB)！ - 添加项目级配置、项目本地 schemas 和 schema 管理命令

  **新功能**

  - **项目级配置** — 通过 `openspec/config.yaml` 配置每个项目的 OpenSpec 行为，包括自定义规则注入、上下文文件和 schema 解析设置
  - **项目本地 schemas** — 在项目的 `openspec/schemas/` 目录中定义自定义 artifact schemas 以用于项目特定工作流
  - **Schema 管理命令** — 新的 `openspec schema` 命令（`list`、`show`、`export`、`validate`）用于检查和管理 artifact schemas（实验性）

  **Bug 修复**

  - 修复了配置加载以处理项目配置中 null `rules` 字段

## 0.21.0

### 次要变更

- [#516](https://github.com/Fission-AI/OpenSpec/pull/516) [`b5a8847`](https://github.com/Fission-AI/OpenSpec/commit/b5a884748be6156a7bb140b4941cfec4f20a9fc8) 感谢 [@TabishB](https://github.com/TabishB)！ - 添加反馈命令和 Nix flake 支持

  **新功能**

  - **反馈命令** — 直接从 CLI 提交反馈 `openspec feedback`，创建带自动元数据包含的 GitHub Issues，并在手动提交时优雅降级
  - **Nix flake 支持** — 使用新的 `flake.nix` 安装和开发 openspec，包括自动 flake 维护和 CI 验证

  **Bug 修复**

  - **探索模式保护** — 探索模式现在明确阻止实现，保持思考和发现的焦点，同时仍允许 artifact 创建

  **其他**

  - 改进了 `opsx apply` 中的变更推断 — 在模糊时从对话上下文自动检测目标变更或提示
  - 简化了归档同步评估，具有更清晰的增量 spec 位置指导

## 0.20.0

### 次要变更

- [#502](https://github.com/Fission-AI/OpenSpec/pull/502) [`9db74aa`](https://github.com/Fission-AI/OpenSpec/commit/9db74aa5ac6547efadaed795217cfa17444f2004) 感谢 [@TabishB](https://github.com/TabishB)！ - 添加 `/opsx:verify` 命令并修复 vitest 进程风暴

  **新功能**

  - **`/opsx:verify` 命令** — 验证变更实现是否匹配其规格

  **Bug 修复**

  - 通过限制 worker 并行性修复了 vitest 进程风暴
  - 修复了 agent 工作流以对验证命令使用非交互模式
  - 修复了 PowerShell 补全生成器以删除尾随逗号

## 0.19.0

### 次要变更

- eb152eb: 添加 Continue IDE 支持、shell 补全和 `/opsx:explore` 命令

  **新功能**

  - **Continue IDE 支持** – OpenSpec 现在为 [Continue](https://continue.dev/) 生成斜杠命令，扩展了与 Cursor、Windsurf、Claude Code 等一起的编辑器集成选项
  - **Bash、Fish 和 PowerShell 的 Shell 补全** – 运行 `openspec completion install` 在你喜欢的 shell 中设置制表符补全
  - **`/opsx:explore` 命令** – 一个新的思考伙伴模式，用于在承诺变更之前探索想法和研究问题
  - **Codebuddy 斜杠命令改进** – 更新了 frontmatter 格式以提高兼容性

  **Bug 修复**

  - Shell 补全现在在命令有子命令时正确提供父级标志（如 `--help`）
  - 修复了 Windows 兼容性问题

  **其他**

  - 添加了可选的匿名使用统计以帮助了解 OpenSpec 的使用方式。这是**默认选择退出** – 设置 `OPENSPEC_TELEMETRY=0` 或 `DO_NOT_TRACK=1` 禁用。仅收集命令名称和版本；不收集参数、文件路径或内容。在 CI 环境中自动禁用。

## 0.18.0

### 次要变更

- 8dfd824: 添加 OPSX 实验性工作流命令和增强的 artifact 系统

  **新命令：**

  - `/opsx:ff` - 快进通过 artifact 创建，一次生成所有需要的 artifact
  - `/opsx:sync` - 将变更的增量 specs 同步到主 specs
  - `/opsx:archive` - 带智能同步检查归档已完成的变更

  **Artifact 工作流增强：**

  - 带内联指导和 XML 输出的 schema 感知应用指令
  - 实验性 artifact 工作流的 agent schema 选择
  - 通过 `.openspec.yaml` 文件进行每变更 schema 元数据
  - Agent 技能用于实验性 artifact 工作流
  - 用于模板加载和变更上下文的指令加载器
  - 重构 schemas 为带有模板的目录

  **改进：**

  - 增强的列表命令，带最后修改时间戳和排序
  - 更好的变更创建实用程序以支持工作流

  **修复：**

  - 规范化路径以实现跨平台 glob 兼容性
  - 在创建新 spec 文件时允许 REMOVED 需求

## 0.17.2

### 补丁变更

- 455c65f: 修复 `--no-interactive` 标志在 validate 命令中正确禁用微调器，防止在 pre-commit hooks 和 CI 环境中挂起

## 0.17.1

### 补丁变更

- a2757e7: 修复 config 命令中的 pre-commit hook 挂起问题，使用 @inquirer/prompts 的动态导入

  config 命令导致 pre-commit hooks 由于在模块加载时注册了 stdin 事件监听器而无限挂起。此修复将静态导入转换为动态导入，仅在实际使用 `config reset` 命令时加载 inquirer。

  还添加了 ESLint 规则以防止静态 @inquirer 导入，避免未来回归。

## 0.17.0

### 次要变更

- 2e71835: 添加 `openspec config` 命令和 Oh-my-zsh 补全

  **新功能**

  - 添加 `openspec config` 命令用于管理全局配置设置
  - 实现带 XDG Base Directory 规范支持的全局配置目录
  - 添加 Oh-my-zsh shell 补全支持以增强 CLI 体验

  **Bug 修复**

  - 使用动态导入修复 pre-commit hooks 挂起
  - 在所有平台上尊重 XDG_CONFIG_HOME 环境变量
  - 解决 zsh-installer 测试中的 Windows 兼容性问题
  - 使 cli-completion spec 与实现保持一致
  - 从斜杠命令中移除硬编码的 agent 字段

  **文档**

  - 在 README 中按字母顺序排列 AI 工具列表并使其可折叠

## 0.16.0

### 次要变更

- c08fbc1: 添加新的 AI 工具集成和增强：

  - **feat(iflow-cli)**: 添加 iFlow-cli 集成与斜杠命令支持和文档
  - **feat(init)**: 添加 IDE 重启指令在 init 后通知用户斜杠命令可用性
  - **feat(antigravity)**: 添加 Antigravity 斜杠命令支持
  - **fix**: 为 Qwen Code 生成 TOML 命令（修复 #293）
  - 澄清了脚手架提案文档并增强了提案指南
  - 更新提案指南以强调设计优先方法在实现之前

## 未发布

### 次要变更

- 添加 Continue 斜杠命令支持，以便 `openspec init` 可以生成 `.continue/prompts/openspec-*.prompt` 文件，带 MARKDOWN frontmatter 和 `$ARGUMENTS` 占位符，并在 `openspec update` 时刷新它们。

- 添加 Antigravity 斜杠命令支持，以便 `openspec init` 可以生成 `.agent/workflows/openspec-*.md` 文件，带仅 description 的 frontmatter，并在 `openspec update` 时与 Windsurf 一起刷新现有 workflows。

## 0.15.0

### 次要变更

- 4758c5c: 添加具有原生斜杠命令集成的新 AI 工具支持

  - **Gemini CLI**: 为 Gemini CLI 添加原生基于 TOML 的斜杠命令支持，带 `.gemini/commands/openspec/` 集成
  - **RooCode**: 添加 RooCode 集成与配置器、斜杠命令和模板
  - **Cline**: 修复 Cline 以使用 workflows 而不是 rules 来获取斜杠命令（`.clinerules/workflows/` 路径）
  - **文档**: 更新文档以反映新的集成和工作流变更

## 0.14.0

### 次要变更

- 8386b91: 添加新的 AI 助手支持和配置改进

  - feat: 添加 Qwen Code 支持与斜杠命令集成
  - feat: 添加 $ARGUMENTS 支持到 apply 斜杠命令以进行动态变量传递
  - feat: 添加 Qoder CLI 支持到配置和文档
  - feat: 添加 CoStrict AI 助手支持
  - fix: 在扩展模式下重新创建缺失的 openspec 模板文件
  - fix: 防止工具的误报'已配置'检测
  - fix: 使用 change-id 作为回退标题而不是"无标题变更"
  - docs: 添加填充项目级上下文的指导
  - docs: 在 README 中添加 Crush 到支持的 AI 工具

## 0.13.0

### 次要变更

- 668a125: 添加多个 AI 助手支持并改进验证

  此版本添加了对多个新 AI 编码助手的支持：

  - CodeBuddy Code - AI 驱动的编码助手
  - CodeRabbit - AI 代码审查助手
  - Cline - Claude 驱动的 CLI 助手
  - Crush AI - AI 助手平台
  - Auggie (Augment CLI) - 代码增强工具

  新功能：

  - 归档斜杠命令现在支持参数以获得更灵活的工作流

  Bug 修复：

  - 增量 spec 验证现在处理不区分大小写的标题并正确检测空部分
  - 归档验证现在正确地遵守 --no-validate 标志并忽略元数据

  文档改进：

  - 添加了 VS Code dev 容器配置以便更轻松地开发设置
  - 使用明确的 change-id 符号更新了 AGENTS.md
  - 增强了斜杠命令文档并添加了重启说明

## 0.12.0

### 次要变更

- 082abb4: 为斜杠命令添加工厂函数支持和非交互式 init 选项

  此版本包含两个新功能：

  - **斜杠命令的工厂函数支持**：斜杠命令现在可以定义为返回命令对象的函数，支持动态命令配置
  - **非交互式 init 选项**：添加了 `--tools`、`--all-tools` 和 `--skip-tools` CLI 标志到 `openspec init`，用于在 CI/CD 管道中自动初始化，同时保持与交互模式的向后兼容

## 0.11.0

### 次要变更

- 312e1d6: 添加 Amazon Q Developer CLI 集成。OpenSpec 现在支持 Amazon Q Developer，在 `.amazonq/prompts/` 目录中自动生成 prompt，允许你在 Amazon Q 的 @-语法中使用 OpenSpec 斜杠命令。

## 0.10.0

### 次要变更

- d7e0ce8: 改进 init 向导 Enter 键行为以允许更自然地通过提示继续

## 0.9.2

### 补丁变更

- 2ae0484: 修复跨平台路径处理问题。此版本包括修复 joinPath 行为和斜杠命令路径解析，以确保 OpenSpec 在所有平台上正确工作。

## 0.9.1

### 补丁变更

- 8210970: 修复当选择 Codex 集成时 OpenSpec 在 Windows 上不工作的问题。此版本包括跨平台路径处理和规范化的修复，以确保 OpenSpec 在 Windows 系统上正确工作。

## 0.9.0

### 次要变更

- efbbf3b: 添加 Codex 和 GitHub Copilot 斜杠命令支持与 YAML frontmatter 和 $ARGUMENTS

## 未发布

### 次要变更

- 添加 GitHub Copilot 斜杠命令支持。OpenSpec 现在将 prompts 写入 `.github/prompts/openspec-{proposal,apply,archive}.prompt.md`，带 YAML frontmatter 和 `$ARGUMENTS` 占位符，并在 `openspec update` 时刷新它们。

## 0.8.1

### 补丁变更

- d070d08: 修复 CLI 版本不匹配并添加发布保护，验证打包的 tarball 打印与 package.json 相同的版本 via `openspec --version`。

## 0.8.0

### 次要变更

- c29b06d: 添加 Windsurf 支持。
- 添加 Codex 斜杠命令支持。OpenSpec 现在直接将 prompts 写入 Codex 的全局目录（`~/.codex/prompts` 或 `$CODEX_HOME/prompts`），并在 `openspec update` 时刷新它们。

## 0.7.0

### 次要变更

- 添加原生 Kilo Code 工作流集成，以便 `openspec init` 和 `openspec update` 管理 `.kilocode/workflows/openspec-*.md` 文件。
- 在 init/update 期间始终将托管根 `AGENTS.md` 手握存根脚手架，并重新组合 AI 工具 prompts 以保持指令一致。

## 0.6.0

### 次要变更

- 将生成的根 agent 指令精简为托管手握存根，并更新 init/update 流程以安全地刷新它。

## 0.5.0

### 次要变更

- feat: 使用跨平台 CI 矩阵实现阶段 1 E2E 测试

  - 在 test/helpers/run-cli.ts 中添加共享 runCLI 帮助程序用于生成测试
  - 创建 test/cli-e2e/basic.test.ts 覆盖 help、version、validate 流程
  - 迁移现有 CLI exec 测试以使用 runCLI 帮助程序
  - 扩展 CI 矩阵到 bash（Linux/macOS）和 pwsh（Windows）
  - 拆分 PR 和 main 工作流以优化反馈

### 补丁变更

- 使应用指令更具体

  改进 agent 模板和斜杠命令模板，带更具体和可操作的应用指令。

- docs: 改进文档和清理

  - 记录归档命令的非交互式标志
  - 替换 README 中的 discord 徽章
  - 归档已完成的变更以便更好地组织

## 0.4.0

### 次要变更

- 添加 OpenSpec 变更提案用于 CLI 改进和增强用户体验
- 添加 Opencode 斜杠命令支持用于 AI 驱动的开发工作流

### 补丁变更

- 添加文档改进，包括归档命令模板的 --yes 标志和 Discord 徽章
- 修复 markdown 解析器中的规范化行尾以正确处理 CRLF 文件

## 0.3.0

### 次要变更

- 使用扩展模式、多工具选择和交互式 `AGENTS.md` 配置器增强 `openspec init`。

## 0.2.0

### 次要变更

- ce5cead: - 添加一个 `openspec view` 仪表盘，一目了然地汇总 spec 计数和变更进度
  - 在重命名期间生成和更新 AI 斜杠命令以及重命名的 `openspec/AGENTS.md` 指令文件
  - 删除弃用的 `openspec diff` 命令并引导用户使用 `openspec show`

## 0.1.0

### 次要变更

- 24b4866: 初始版本