# OpenSpec

<p align="center">
  <a href="https://github.com/Fission-AI/OpenSpec">
    <picture>
      <source srcset="assets/openspec_pixel_dark.svg" media="(prefers-color-scheme: dark)">
      <source srcset="assets/openspec_pixel_light.svg" media="(prefers-color-scheme: light)">
      <img src="assets/openspec_pixel_light.svg" alt="OpenSpec logo" height="64">
    </picture>
  </a>

</p>
<p align="center">AI 编码助手的规格驱动开发。</p>
<p align="center">
  <a href="https://github.com/Fission-AI/OpenSpec/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/Fission-AI/OpenSpec/actions/workflows/ci.yml/badge.svg" /></a>
  <a href="https://www.npmjs.com/package/@fission-ai/openspec"><img alt="npm version" src="https://img.shields.io/npm/v/@fission-ai/openspec?style=flat-square" /></a>
  <a href="https://nodejs.org/"><img alt="node version" src="https://img.shields.io/node/v/@fission-ai/openspec?style=flat-square" /></a>
  <a href="./LICENSE"><img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square" /></a>
  <a href="https://conventionalcommits.org"><img alt="Conventional Commits" src="https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg?style=flat-square" /></a>
  <a href="https://discord.gg/YctCnvvshC"><img alt="Discord" src="https://img.shields.io/badge/Discord-Join%20the%20community-5865F2?logo=discord&logoColor=white&style=flat-square" /></a>
</p>

<p align="center">
  <img src="assets/openspec_dashboard.png" alt="OpenSpec 仪表盘预览" width="90%">
</p>

<p align="center">
  在 <a href="https://x.com/0xTab">X 上关注 @0xTab</a> 获取更新 · 加入 <a href="https://discord.gg/YctCnvvshC">OpenSpec Discord</a> 寻求帮助和问题解答。
</p>

<p align="center">
  <sub>🧪 <strong>新功能：</strong> <a href="docs/opsx.md">OPSX 工作流</a> — schema 驱动、可 hack、流畅。迭代工作流而无需代码更改。</sub>
</p>

# OpenSpec

OpenSpec 使人类和 AI 编码助手与规格驱动开发保持一致，以便你在写任何代码之前就同意要构建什么。**无需 API 密钥。**

## 为什么选择 OpenSpec？

AI 编码助手很强大，但当需求仅存在于聊天历史中时，它们是不可预测的。OpenSpec 添加了一个轻量级的规格工作流，在实现之前锁定意图，给你确定的、可审查的输出。

关键成果：
- 人类和 AI 利益相关者在工作开始前同意规格。
- 结构化的变更文件夹（提案、任务和规格更新）保持范围明确和可审计。
- 共享可见性，了解什么是已提议、活动或已归档。
- 与你已经使用的 AI 工具配合工作：在支持的地方使用自定义斜杠命令，在其他地方使用上下文规则。

## OpenSpec 如何比较（一目了然）

- **轻量级**：简单工作流，无需 API 密钥，最小设置。
- **Brownfield 优先**：在 0→1 之外也能很好地工作。OpenSpec 将真相来源与提案分开：`openspec/specs/`（当前真相）和 `openspec/changes/`（提议的更新）。这使差异保持明确和可管理跨功能。
- **变更跟踪**：提案、任务和规格增量生活在一起；归档将批准的更新合并回规格。
- **与 spec-kit & Kiro 比较**：这些在全新的功能（0→1）上表现出色。在修改现有行为（1→n）时，OpenSpec 也表现出色，特别是当更新跨越多个规格时。

参见 [OpenSpec 如何比较](#how-openspec-compares) 中的完整比较。

## 它是如何工作的

```
┌────────────────────┐
│ 起草变更           │
│ 提案               │
└────────┬───────────┘
         │ 与你的 AI 分享意图
         ▼
┌────────────────────┐
│ 审查与对齐         │
│ (编辑 specs/任务)  │◀──── 反馈循环 ──────┐
└────────┬───────────┘                      │
         │ 批准的计划                        │
         ▼                                  │
┌────────────────────┐                      │
│ 实现任务           │──────────────────────────┘
│ (AI 编写代码)      │
└────────┬───────────┘
         │ 交付变更
         ▼
┌────────────────────┐
│ 归档并更新         │
│ Specs (真相来源)   │
└────────────────────┘

1. 起草一个捕获你想要的规格更新的变更提案。
2. 与你的 AI 助手审查提案，直到每个人都同意。
3. 实现引用已同意规格的任务。
4. 归档变更以将批准的更新合并回真相来源规格。
```

## 入门

### 支持的 AI 工具

<details>
<summary><strong>原生斜杠命令</strong>（点击展开）</summary>

这些工具有内置的 OpenSpec 命令。在提示时选择 OpenSpec 集成。

| 工具 | 命令 |
|------|----------|
| **Amazon Q Developer** | `@openspec-proposal`、`@openspec-apply`、`@openspec-archive`（`.amazonq/prompts/`） |
| **Antigravity** | `/openspec-proposal`、`/openspec-apply`、`/openspec-archive`（`.agent/workflows/`） |
| **Auggie (Augment CLI)** | `/openspec-proposal`、`/openspec-apply`、`/openspec-archive`（`.augment/commands/`） |
| **Claude Code** | `/openspec:proposal`、`/openspec:apply`、`/openspec:archive` |
| **Cline** | `.clinerules/workflows/` 目录中的 Workflows（`.clinerules/workflows/openspec-*.md`） |
| **CodeBuddy Code (CLI)** | `/openspec:proposal`、`/openspec:apply`、`/openspec:archive`（`.codebuddy/commands/`） — 参见[文档](https://www.codebuddy.ai/cli) |
| **Codex** | `/openspec-proposal`、`/openspec-apply`、`/openspec-archive`（全局：`~/.codex/prompts`，自动安装） |
| **Continue** | `/openspec-proposal`、`/openspec-apply`、`/openspec-archive`（`.continue/prompts/`） |
| **CoStrict** | `/openspec-proposal`、`/openspec-apply`、`/openspec-archive`（`.cospec/openspec/commands/`） — 参见[文档](https://costrict.ai)|
| **Crush** | `/openspec-proposal`、`/openspec-apply`、`/openspec-archive`（`.crush/commands/openspec/`） |
| **Cursor** | `/openspec-proposal`、`/openspec-apply`、`/openspec-archive` |
| **Factory Droid** | `/openspec-proposal`、`/openspec-apply`、`/openspec-archive`（`.factory/commands/`） |
| **Gemini CLI** | `/openspec:proposal`、`/openspec:apply`、`/openspec:archive`（`.gemini/commands/openspec/`） |
| **GitHub Copilot** | `/openspec-proposal`、`/openspec-apply`、`/openspec-archive`（`.github/prompts/`） |
| **iFlow (iflow-cli)** | `/openspec-proposal`、`/openspec-apply`、`/openspec-archive`（`.iflow/commands/`） |
| **Kilo Code** | `/openspec-proposal.md`、`/openspec-apply.md`、`/openspec-archive.md`（`.kilocode/workflows/`） |
| **OpenCode** | `/openspec-proposal`、`/openspec-apply`、`/openspec-archive` |
| **Qoder** | `/openspec:proposal`、`/openspec:apply`、`/openspec:archive`（`.qoder/commands/openspec/`） — 参见[文档](https://qoder.com) |
| **Qwen Code** | `/openspec-proposal`、`/openspec-apply`、`/openspec-archive`（`.qwen/commands/`） |
| **RooCode** | `/openspec-proposal`、`/openspec-apply`、`/openspec-archive`（`.roo/commands/`） |
| **Windsurf** | `/openspec-proposal`、`/openspec-apply`、`/openspec-archive`（`.windsurf/workflows/`） |

Kilo Code 自动发现团队 workflows。将生成的文件保存在 `.kilocode/workflows/` 下，并使用 `/openspec-proposal.md`、`/openspec-apply.md` 或 `/openspec-archive.md` 从命令面板触发它们。

</details>

<details>
<summary><strong>AGENTS.md 兼容</strong>（点击展开）</summary>

这些工具自动从 `openspec/AGENTS.md` 读取工作流指令。如果他们需要提醒，请让他们遵循 OpenSpec 工作流。了解有关 [AGENTS.md 约定](https://agents.md/) 的更多信息。

| 工具 |
|-------|
| Amp • Jules • 其他 |

</details>

### 安装和初始化

#### 先决条件
- **Node.js >= 20.19.0** - 使用 `node --version` 检查你的版本

#### 步骤 1：全局安装 CLI

**选项 A：使用 npm**

```bash
npm install -g @fission-ai/openspec@latest
```

验证安装：
```bash
openspec --version
```

**选项 B：使用 Nix（NixOS 和 Nix 包管理器）**

直接运行 OpenSpec 而无需安装：
```bash
nix run github:Fission-AI/OpenSpec -- init
```

或安装到你的 profile：
```bash
nix profile install github:Fission-AI/OpenSpec
```

或在 `flake.nix` 中添加到你的开发环境：
```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    openspec.url = "github:Fission-AI/OpenSpec";
  };

  outputs = { nixpkgs, openspec, ... }: {
    devShells.x86_64-linux.default = nixpkgs.legacyPackages.x86_64-linux.mkShell {
      buildInputs = [ openspec.packages.x86_64-linux.default ];
    };
  };
}
```

验证安装：
```bash
openspec --version
```

#### 步骤 2：在你的项目中初始化 OpenSpec

导航到你的项目目录：
```bash
cd my-project
```

运行初始化：
```bash
openspec init
```

**初始化期间会发生什么：**
- 系统会提示你选择任何原生支持的 AI 工具（Claude Code、CodeBuddy、Cursor、OpenCode、Qoder 等）；其他助手始终依赖共享的 `AGENTS.md` 存根
- OpenSpec 自动为你选择的工具配置斜杠命令，始终在项目根目录编写一个托管的 `AGENTS.md` 手握存根
- 在你的项目中创建一个新的 `openspec/` 目录结构

**设置后：**
- 主要 AI 工具可以触发 `/openspec` 工作流而无需额外配置
- 运行 `openspec list` 验证设置并查看任何活动变更
- 如果你的编码助手没有立即显示新的斜杠命令，请重启它。斜杠命令在启动时加载，
  因此全新启动确保它们出现

### 可选：填充项目上下文

`openspec init` 完成后，你将收到一个建议的提示来帮助你填充项目上下文：

```text
填充你的项目上下文：
"请阅读 openspec/project.md 并帮助我填写关于我的项目、技术栈和约定的详细信息"
```

使用 `openspec/project.md` 定义项目级约定、标准、架构模式以及应该跨所有变更遵循的其他指导方针。

### 创建你的第一个变更

这是一个显示完整 OpenSpec 工作流的真实示例。这适用于任何 AI 工具。具有原生斜杠命令的工具会自动识别快捷方式。

#### 1. 起草提案
首先要求你的 AI 创建一个变更提案：

```text
你：创建一个用于添加按角色和团队过滤的配置文件的 OpenSpec 变更提案
     （对于有斜杠命令的工具的快捷方式：/openspec:proposal Add profile search filters）

AI： 我将为配置文件过滤器创建一个 OpenSpec 变更提案。
     *使用 proposal.md、tasks.md、spec deltas 搭建 openspec/changes/add-profile-filters/ 结构*
```

#### 2. 验证和审查
检查变更是否正确创建并审查提案：

```bash
$ openspec list                             # 确认变更文件夹存在
$ openspec validate add-profile-filters     # 验证 spec 格式
$ openspec show add-profile-filters         # 审查提案、任务和 spec 增量
```

#### 3. 完善规格
迭代规格直到它们符合你的需求：

```text
你：你能为角色和团队过滤器添加验收标准吗？

AI： 我将使用角色和团队过滤器的场景更新 spec 增量。
     *编辑 openspec/changes/add-profile-filters/specs/profile/spec.md 和 tasks.md*
```

#### 4. 实现变更
一旦规格看起来不错，开始实现：

```text
你：规格看起来不错。让我们实现这个变更。
     （对于有斜杠命令的工具的快捷方式：/openspec:apply add-profile-filters）

AI： 我将通过 add-profile-filters 变更中的任务进行工作。
     *实现来自 openspec/changes/add-profile-filters/tasks.md 的任务*
     *标记任务完成：任务 1.1 ✓、任务 1.2 ✓、任务 2.1 ✓...*
```

#### 5. 归档已完成的变更
实现完成后，归档变更：

```text
AI： 所有任务都完成了。实现已准备好。

你：请归档变更
     （对于有斜杠命令的工具的快捷方式：/openspec:archive add-profile-filters）

AI： 我将归档 add-profile-filters 变更。
     *运行：openspec archive add-profile-filters --yes*
      ✓ 变更归档成功。Specs 已更新。准备好下一个功能！
```

或在终端中自己运行命令：
```bash
$ openspec archive add-profile-filters --yes  # 归档已完成的变更而不提示
```

**注意：** 具有原生斜杠命令的工具（Claude Code、CodeBuddy、Cursor、Codex、Qoder、RooCode）可以使用显示的快捷方式。所有其他工具都可以使用自然语言请求"创建一个 OpenSpec 提案"、"应用 OpenSpec 变更"或"归档变更"。

## 命令参考

```bash
openspec list               # 查看活动变更文件夹
openspec view               # Specs 和变更的交互式仪表盘
openspec show <变更>      # 显示变更详情（提案、任务、规格更新）
openspec validate <变更>  # 检查 spec 格式和结构
openspec archive <变更> [--yes|-y]   # 将已完成的变更移动到 archive/（使用 --yes 非交互式）
```

## 示例：AI 如何创建 OpenSpec 文件

当你要求你的 AI 助手"添加双因素认证"时，它会创建：

```
openspec/
├── specs/
│   └── auth/
│       └── spec.md           # 当前 auth 规格（如果存在）
└── changes/
    └── add-2fa/              # AI 创建这个整个结构
        ├── proposal.md       # 为什么以及什么变更
        ├── tasks.md          # 实现检查清单
        ├── design.md         # 技术决策（可选）
        └── specs/
            └── auth/
                └── spec.md   # 显示增量的 delta
```

### AI 生成的 Spec（创建在 `openspec/specs/auth/spec.md`）：

```markdown
# Auth Specification

## Purpose
Authentication and session management.

## Requirements
### Requirement: User Authentication
The system SHALL issue a JWT on successful login.

#### Scenario: Valid credentials
- WHEN a user submits valid credentials
- THEN a JWT is returned
```

### AI 生成的变更增量（创建在 `openspec/changes/add-2fa/specs/auth/spec.md`）：

```markdown
# Delta for Auth

## ADDED Requirements
### Requirement: Two-Factor Authentication
The system MUST require a second factor during login.

#### Scenario: OTP required
- WHEN a user submits valid credentials
- THEN an OTP challenge is required
```

### AI 生成的任务（创建在 `openspec/changes/add-2fa/tasks.md`）：

```markdown
## 1. Database Setup
- [ ] 1.1 Add OTP secret column to users table
- [ ] 1.2 Create OTP verification logs table

## 2. Backend Implementation  
- [ ] 2.1 Add OTP generation endpoint
- [ ] 2.2 Modify login flow to require OTP
- [ ] 2.3 Add OTP verification endpoint

## 3. Frontend Updates
- [ ] 3.1 Create OTP input component
- [ ] 3.2 Update login flow UI
```

**重要：** 你不需要手动创建这些文件。你的 AI 助手根据你的需求和现有代码库生成它们。

## 理解 OpenSpec 文件

### 增量格式

增量是显示规格如何变化的"补丁"：

- **`## ADDED Requirements`** - 新功能
- **`## MODIFIED Requirements`** - 更改的行为（包含完整的更新文本）
- **`## REMOVED Requirements`** - 弃用的功能

**格式要求：**
- 使用 `### Requirement: <name>` 作为标题
- 每个需求至少需要一个 `#### Scenario:` 块
- 在需求文本中使用 SHALL/MUST

## OpenSpec 如何比较

### vs. spec-kit
OpenSpec 的双文件夹模型（`openspec/specs/` 用于当前真相，`openspec/changes/` 用于提议的更新）保持状态和差异分离。当修改现有功能或触及多个规格时，这会扩展。spec-kit 适用于 greenfield/0→1 但为跨规格更新和演变功能提供更少的结构。

### vs. Kiro.dev
OpenSpec 将一个功能的每个变更分组在一个文件夹中（`openspec/changes/feature-name/`），使相关 specs、任务和设计易于跟踪。Kiro 将更新分散到多个 spec 文件夹中，这可能使功能跟踪更难。

### vs. 无规格
没有规格，AI 编码助手从模糊的提示生成代码，经常错过需求或添加不需要的功能。OpenSpec 通过在任何代码编写之前同意所需的行为来带来可预测性。

## 团队采用

1. **初始化 OpenSpec** – 在你的仓库中运行 `openspec init`。
2. **从新功能开始** – 要求你的 AI 将即将开展的工作捕获为变更提案。
3. **增量增长** – 每个变更归档到记录你的系统的活动规格中。
4. **保持灵活** – 不同的队友可以使用 Claude Code、CodeBuddy、Cursor 或任何 AGENTS.md 兼容工具，同时共享相同的规格。

每当有人切换工具时运行 `openspec update`，以便你的 agent 获取最新指令和斜杠命令绑定。

## 更新 OpenSpec

1. **升级包**
   ```bash
   npm install -g @fission-ai/openspec@latest
   ```
2. **刷新 agent 指令**
   - 在每个项目中运行 `openspec update` 以重新生成 AI 指导并确保最新的斜杠命令处于活跃状态。

## 实验性功能

<details>
<summary><strong>🧪 OPSX：流畅、迭代工作流</strong>（仅限 Claude Code）</summary>

**为什么存在：**
- 标准工作流被锁定 — 你无法调整指令或自定义
- 当 AI 输出不好时，你无法自己改进提示
- 对每个人都一样的工作流，没有方式匹配你的团队工作方式

**有什么不同：**
- **可 hack** — 立即自己编辑模板和 schemas，测试，无需重建
- **细粒度** — 每个 artifact 有自己的指令，单独测试和调整
- **可自定义** — 定义你自己的工作流、artifacts 和依赖关系
- **流畅** — 无阶段门，随时更新任何 artifact

```
你可以随时返回：

  proposal ──→ specs ──→ design ──→ tasks ──→ implement
      ▲           ▲          ▲                    │
      └───────────┴──────────┴────────────────────┘
```

| 命令 | 功能 |
|---------|--------------|
| `/opsx:new` | 开始一个新变更 |
| `/opsx:continue` | 创建下一个 artifact（基于什么准备好） |
| `/opsx:ff` | 快进（一次所有规划 artifacts） |
| `/opsx:apply` | 实现任务，根据需要更新 artifacts |
| `/opsx:archive` | 完成后归档 |

**设置：** `openspec experimental`

[完整文档 →](docs/opsx.md)

</details>

<details>
<summary><strong>遥测</strong> – OpenSpec 收集匿名使用统计（选择退出：<code>OPENSPEC_TELEMETRY=0</code>）</summary>

我们仅收集命令名称和版本以了解使用模式。不收集参数、路径、内容或 PII。在 CI 中自动禁用。

**选择退出：** `export OPENSPEC_TELEMETRY=0` 或 `export DO_NOT_TRACK=1`

</details>

## 贡献

- 安装依赖：`pnpm install`
- 构建：`pnpm run build`
- 测试：`pnpm test`
- 本地开发 CLI：`pnpm run dev` 或 `pnpm run dev:cli`
- 常规提交（一行式）：`type(scope): subject`

<details>
<summary><strong>维护者和顾问</strong></summary>

请参阅 [MAINTAINERS.md](MAINTAINERS.md) 了解核心维护者和顾问的名单，他们帮助指导项目。

</details>

## 许可证

MIT