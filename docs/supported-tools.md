# 支持的工具

OpenSpec 支持多种 AI 编码助手。运行 `openspec init` 时，OpenSpec 会根据你选择的配置文件和工作流选择以及交付模式来配置选定的工具。

## 工作原理

对于每个选定的工具，OpenSpec 可以安装：

1. **技能**（如果交付包含技能）：`.../skills/openspec-*/SKILL.md`
2. **命令**（如果交付包含命令）：特定于工具的 `opsx-*` 命令文件

默认情况下，OpenSpec 使用 `core` 配置文件，包含：
- `propose`
- `explore`
- `apply`
- `sync`
- `archive`

你可以通过 `openspec config profile` 启用扩展工作流（`new`、`continue`、`ff`、`verify`、`bulk-archive`、`onboard`），然后运行 `openspec update`。

## 工具目录参考

| 工具（ID） | 技能路径模式 | 命令路径模式 |
|-----------|---------------------|----------------------|
| Amazon Q Developer（`amazon-q`） | `.amazonq/skills/openspec-*/SKILL.md` | `.amazonq/prompts/opsx-<id>.md` |
| Antigravity（`antigravity`） | `.agent/skills/openspec-*/SKILL.md` | `.agent/workflows/opsx-<id>.md` |
| Auggie（`auggie`） | `.augment/skills/openspec-*/SKILL.md` | `.augment/commands/opsx-<id>.md` |
| IBM Bob Shell（`bob`） | `.bob/skills/openspec-*/SKILL.md` | `.bob/commands/opsx-<id>.md` |
| Claude Code（`claude`） | `.claude/skills/openspec-*/SKILL.md` | `.claude/commands/opsx/<id>.md` |
| Cline（`cline`） | `.cline/skills/openspec-*/SKILL.md` | `.clinerules/workflows/opsx-<id>.md` |
| CodeBuddy（`codebuddy`） | `.codebuddy/skills/openspec-*/SKILL.md` | `.codebuddy/commands/opsx/<id>.md` |
| Codex（`codex`） | `.codex/skills/openspec-*/SKILL.md` | `$CODEX_HOME/prompts/opsx-<id>.md`\* |
| ForgeCode（`forgecode`） | `.forge/skills/openspec-*/SKILL.md` | 不生成（无命令适配器；使用基于技能的 `/openspec-*` 调用） |
| Continue（`continue`） | `.continue/skills/openspec-*/SKILL.md` | `.continue/prompts/opsx-<id>.prompt` |
| CoStrict（`costrict`） | `.cospec/skills/openspec-*/SKILL.md` | `.cospec/openspec/commands/opsx-<id>.md` |
| Crush（`crush`） | `.crush/skills/openspec-*/SKILL.md` | `.crush/commands/opsx/<id>.md` |
| Cursor（`cursor`） | `.cursor/skills/openspec-*/SKILL.md` | `.cursor/commands/opsx-<id>.md` |
| Factory Droid（`factory`） | `.factory/skills/openspec-*/SKILL.md` | `.factory/commands/opsx-<id>.md` |
| Gemini CLI（`gemini`） | `.gemini/skills/openspec-*/SKILL.md` | `.gemini/commands/opsx/<id>.toml` |
| GitHub Copilot（`github-copilot`） | `.github/skills/openspec-*/SKILL.md` | `.github/prompts/opsx-<id>.prompt.md`** |
| iFlow（`iflow`） | `.iflow/skills/openspec-*/SKILL.md` | `.iflow/commands/opsx-<id>.md` |
| Junie（`junie`） | `.junie/skills/openspec-*/SKILL.md` | `.junie/commands/opsx-<id>.md` |
| Kilo Code（`kilocode`） | `.kilocode/skills/openspec-*/SKILL.md` | `.kilocode/workflows/opsx-<id>.md` |
| Kimi CLI（`kimi`） | `.kimi/skills/openspec-*/SKILL.md` | 不生成（无命令适配器；使用基于技能的 `/skill:openspec-*` 调用） |
| Kiro（`kiro`） | `.kiro/skills/openspec-*/SKILL.md` | `.kiro/prompts/opsx-<id>.prompt.md` |
| Lingma（`lingma`） | `.lingma/skills/openspec-*/SKILL.md` | `.lingma/commands/opsx/<id>.md` |
| OpenCode（`opencode`） | `.opencode/skills/openspec-*/SKILL.md` | `.opencode/commands/opsx-<id>.md` |
| Pi（`pi`） | `.pi/skills/openspec-*/SKILL.md` | `.pi/prompts/opsx-<id>.md` |
| Qoder（`qoder`） | `.qoder/skills/openspec-*/SKILL.md` | `.qoder/commands/opsx/<id>.md` |
| Qwen Code（`qwen`） | `.qwen/skills/openspec-*/SKILL.md` | `.qwen/commands/opsx-<id>.toml` |
| RooCode（`roocode`） | `.roo/skills/openspec-*/SKILL.md` | `.roo/commands/opsx-<id>.md` |
| Trae（`trae`） | `.trae/skills/openspec-*/SKILL.md` | 不生成（无命令适配器；使用基于技能的 `/openspec-*` 调用） |
| Windsurf（`windsurf`） | `.windsurf/skills/openspec-*/SKILL.md` | `.windsurf/workflows/opsx-<id>.md` |

\* Codex 命令安装在全局 Codex 主目录（`$CODEX_HOME/prompts/`，如果已设置），而非你的项目目录。

\*\* GitHub Copilot 提示文件在 IDE 扩展中识别为自定义斜杠命令（VS Code、JetBrains、Visual Studio）。Copilot CLI 目前不直接消费 `.github/prompts/*.prompt.md`。

## 非交互式设置

对于 CI/CD 或脚本化设置，使用 `--tools`（和可选的 `--profile`）：

```bash
# 配置特定工具
openspec init --tools claude,cursor

# 配置所有支持的工具
openspec init --tools all

# 跳过工具配置
openspec init --tools none

# 覆盖此 init 运行的配置文件
openspec init --profile core
```

**可用的工具 ID（`--tools`）：** `amazon-q`、`antigravity`、`auggie`、`bob`、`claude`、`cline`、`codex`、`forgecode`、`codebuddy`、`continue`、`costrict`、`crush`、`cursor`、`factory`、`gemini`、`github-copilot`、`iflow`、`junie`、`kilocode`、`kimi`、`kiro`、`opencode`、`pi`、`qoder`、`lingma`、`qwen`、`roocode`、`trae`、`windsurf`

## 基于工作流的安装

OpenSpec 根据选择的工作流安装工作流产物：

- **核心配置文件（默认）：** `propose`、`explore`、`apply`、`sync`、`archive`
- **自定义选择：** 所有工作流 ID 的任意子集：
  `propose`、`explore`、`new`、`continue`、`apply`、`ff`、`sync`、`archive`、`bulk-archive`、`verify`、`onboard`

换句话说，技能/命令数量取决于配置文件和交付模式，而非固定。

## 生成的技能名称

当按配置文件/工作流配置选择时，OpenSpec 会生成以下技能：

- `openspec-propose`
- `openspec-explore`
- `openspec-new-change`
- `openspec-continue-change`
- `openspec-apply-change`
- `openspec-ff-change`
- `openspec-sync-specs`
- `openspec-archive-change`
- `openspec-bulk-archive-change`
- `openspec-verify-change`
- `openspec-onboard`

参见[命令](commands.md)了解命令行为，[CLI](cli.md)了解 `init`/`update` 选项。

## 相关文档

- [CLI 参考](cli.md) — 终端命令
- [命令](commands.md) — 斜杠命令和技能
- [快速入门](getting-started.md) — 首次设置