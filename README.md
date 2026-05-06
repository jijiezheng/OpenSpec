# OpenSpec

<p align="center">
  <a href="https://github.com/Fission-AI/OpenSpec">
    <picture>
      <source srcset="assets/openspec_bg.png">
      <img src="assets/openspec_bg.png" alt="OpenSpec logo">
    </picture>
  </a>
</p>

<p align="center">
  <a href="https://github.com/Fission-AI/OpenSpec/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/Fission-AI/OpenSpec/actions/workflows/ci.yml/badge.svg" /></a>
  <a href="https://www.npmjs.com/package/@fission-ai/openspec"><img alt="npm version" src="https://img.shields.io/npm/v/@fission-ai/openspec?style=flat-square" /></a>
  <a href="./LICENSE"><img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square" /></a>
  <a href="https://discord.gg/YctCnvvshC"><img alt="Discord" src="https://img.shields.io/discord/1411657095639601154?style=flat-square&logo=discord&logoColor=white&label=Discord&suffix=%20online" /></a>
</p>

<details>
<summary><strong>最受喜爱的规格框架。</strong></summary>

[![Stars](https://img.shields.io/github/stars/Fission-AI/OpenSpec?style=flat-square&label=Stars)](https://github.com/Fission-AI/OpenSpec/stargazers)
[![Downloads](https://img.shields.io/npm/dm/@fission-ai/openspec?style=flat-square&label=Downloads/mo)](https://www.npmjs.com/package/@fission-ai/openspec)
[![Contributors](https://img.shields.io/github/contributors/Fission-AI/OpenSpec?style=flat-square&label=Contributors)](https://github.com/Fission-AI/OpenSpec/graphs/contributors)

</details>
<p></p>
我们的理念：

```text
→ 流畅而非僵化
→ 迭代而非瀑布
→ 简单而非复杂
→ 为 brownfield 而非仅 greenfield 构建
→ 从个人项目到企业均可扩展
```

> [!TIP]
> **新工作流已上线！** 我们用全新的 artifact 引导工作流重建了 OpenSpec。
>
> 运行 `/opsx:propose "你的想法"` 开始使用。 → [了解更多](docs/opsx.md)

<p align="center">
  在 <a href="https://x.com/0xTab">X 上关注 @0xTab</a> 获取更新 · 加入 <a href="https://discord.gg/YctCnvvshC">OpenSpec Discord</a> 寻求帮助和问题解答。
</p>

<!-- TODO: 添加 /opsx:propose → /opsx:archive 工作流的 GIF 演示 -->

## 看看它的实际效果

```text
你: /opsx:propose add-dark-mode
AI:  已创建 openspec/changes/add-dark-mode/
     ✓ proposal.md — 我们为什么要这样做，发生了什么变化
     ✓ specs/       — 需求和场景
     ✓ design.md    — 技术方案
     ✓ tasks.md     — 实现检查清单
     准备好实现了！

你: /opsx:apply
AI:  正在实现任务...
     ✓ 1.1 添加主题上下文提供器
     ✓ 1.2 创建切换组件
     ✓ 2.1 添加 CSS 变量
     ✓ 2.2 连接 localStorage
     所有任务完成！

你: /opsx:archive
AI:  已归档到 openspec/changes/archive/2025-01-23-add-dark-mode/
     规格已更新。准备好下一个功能。
```

<details>
<summary><strong>OpenSpec 仪表盘</strong></summary>

<p align="center">
  <img src="assets/openspec_dashboard.png" alt="OpenSpec 仪表盘预览" width="90%">
</p>

</details>

## 快速开始

**需要 Node.js 20.19.0 或更高版本。**

全局安装 OpenSpec：

```bash
npm install -g @fission-ai/openspec@latest
```

然后导航到你的项目目录并初始化：

```bash
cd your-project
openspec init
```

现在告诉你的 AI：`/opsx:propose <你想构建的内容>`

如果你想要扩展工作流（`/opsx:new`、`/opsx:continue`、`/opsx:ff`、`/opsx:verify`、`/opsx:bulk-archive`、`/opsx:onboard`），使用 `openspec config profile` 选择它并使用 `openspec update` 应用。

> [!NOTE]
> 不确定你的工具是否受支持？[查看完整列表](docs/supported-tools.md) – 我们支持 25+ 工具并且还在增加。
>
> 还支持 pnpm、yarn、bun 和 nix。[查看安装选项](docs/installation.md)。

## 文档

→ **[入门指南](docs/getting-started.md)**：第一步<br>
→ **[工作流](docs/workflows.md)**：组合和模式<br>
→ **[命令](docs/commands.md)**：斜杠命令和技能<br>
→ **[CLI](docs/cli.md)**：终端参考<br>
→ **[支持的工具](docs/supported-tools.md)**：工具集成和安装路径<br>
→ **[概念](docs/concepts.md)**：一切如何运作<br>
→ **[多语言](docs/multi-language.md)**：多语言支持<br>
→ **[自定义](docs/customization.md)**：让它适合你


## 为什么选择 OpenSpec？

AI 编码助手很强大，但当需求仅存在于聊天历史中时，它们是不可预测的。OpenSpec 添加了一个轻量级的规格层，让你在写任何代码之前就同意要构建什么。

- **在构建之前达成共识** — 人类和 AI 在代码编写之前在规格上对齐
- **保持井井有条** — 每个变更都有自己的文件夹，包含 proposal、specs、design 和 tasks
- **流畅工作** — 随时更新任何 artifact，没有僵化的阶段门
- **使用你的工具** — 通过斜杠命令与 20+ AI 助手配合工作

### 我们如何比较

**vs. [Spec Kit](https://github.com/github/spec-kit)** (GitHub) — 全面但笨重。僵化的阶段门、大量 Markdown、Python 设置。OpenSpec 更轻量，让你自由迭代。

**vs. [Kiro](https://kiro.dev)** (AWS) — 强大，但你被锁定在其 IDE 中，仅限于 Claude 模型。OpenSpec 与你已经使用的工具配合工作。

**vs. 什么都没有** — 没有规格的 AI 编码意味着模糊的提示和不可预测的结果。OpenSpec 带来可预测性而无需繁琐的仪式。

## 更新 OpenSpec

**升级包**

```bash
npm install -g @fission-ai/openspec@latest
```

**刷新 agent 指令**

在每个项目内运行此命令以重新生成 AI 指导并确保最新的斜杠命令处于活跃状态：

```bash
openspec update
```

## 使用说明

**模型选择**：OpenSpec 在高推理模型上效果最佳。我们建议将 Opus 4.5 和 GPT 5.2 用于规划和实现。

**上下文卫生**：OpenSpec 从干净的上下文窗口中受益。在开始实现之前清除你的上下文，并在整个会话中保持良好的上下文卫生。

## 贡献

**小修复** — Bug 修复、拼写纠正和微小改进可以直接作为 PR 提交。

**更大的变更** — 对于新功能、重要重构或架构变更，请先提交 OpenSpec 变更提案，以便我们在开始实现之前对齐意图和目标。

编写提案时，牢记 OpenSpec 理念：我们为跨不同编码 agent、模型和用例的广泛用户群体服务。变更应该对每个人都有效。

**欢迎 AI 生成的代码** — 只要它经过测试和验证。包含 AI 生成代码的 PR 应提及所使用的编码 agent 和模型（例如"Generated with Claude Code using claude-opus-4-5-20251101"）。

### 开发

- 安装依赖：`pnpm install`
- 构建：`pnpm run build`
- 测试：`pnpm test`
- 本地开发 CLI：`pnpm run dev` 或 `pnpm run dev:cli`
- 常规提交（一行式）：`type(scope): subject`

## 其他

<details>
<summary><strong>遥测</strong></summary>

OpenSpec 收集匿名使用统计。

我们仅收集命令名称和版本以了解使用模式。不收集参数、路径、内容或 PII。在 CI 中自动禁用。

**选择退出：** `export OPENSPEC_TELEMETRY=0` 或 `export DO_NOT_TRACK=1`

</details>

<details>
<summary><strong>维护者和顾问</strong></summary>

请参阅 [MAINTAINERS.md](MAINTAINERS.md) 了解核心维护者和顾问的名单，他们帮助指导项目。

</details>



## 许可证

MIT