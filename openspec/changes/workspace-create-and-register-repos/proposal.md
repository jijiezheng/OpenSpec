## 为什么

用户通过创建规划之家并链接 OpenSpec 应该知道的 repos 或文件夹来开始 workspace 工作。

在 OpenSpec 可以看到相关的 repos、monorepo 文件夹、包、服务或应用之前，他们不应被迫创建变更。

产品规则是：

```text
Workspace 可见性不是变更承诺。
```

Workspace 是持久的规划之家。变更是该 workspace 内的一个功能、修复、项目或其他计划的工作。

## 什么变更

添加第一个用户面向的 workspace 设置流程：

```text
设置一个 workspace。
链接现有的 repos 或文件夹。
列出已知的 workspaces 及其链接的内容。
检查 OpenSpec 可以解析什么以及如何修复问题。
```

预期的用户表面：

```bash
openspec workspace setup
openspec workspace setup --no-interactive --name platform --link /path/to/api --link web=/path/to/web
openspec workspace list
openspec workspace ls
openspec workspace link /path/to/api
openspec workspace link api-service /path/to/api
openspec workspace relink api /new/path/to/api
openspec workspace doctor
```

`workspace setup` 是用户的创建路径。它应该首先询问 workspace 名称，在标准位置创建 workspace，至少需要一个现有的 repo 或文件夹路径，从文件夹名称推断链接名称，显示 workspace 路径，并在最后运行检查，以便用户知道 OpenSpec 可以看到什么。

`workspace setup --no-interactive` 是自动化路径。它应该需要足够的标志来创建一个有用的 workspace，包括 workspace 名称和至少一个链接。

`workspace list` 从本地 workspace 注册表显示已知的 OpenSpec 管理的 workspaces，包括每个 workspace 路径和链接的 repos 或文件夹。

`workspace link` 为选定的 workspace 记录一个现有的本地 repo 或文件夹路径。它应该支持从文件夹名称推断链接名称的简单形式，以及用于冲突或清晰度的显式名称形式。链接不会在链接的 repo 或文件夹中创建、复制、移动、初始化或编辑文件。

`workspace relink` 让用户修复或更改现有链接的本地路径，而无需重新创建 workspace。在这个 slice 中不应引入 owner 或 handoff 元数据。

`workspace doctor` 解释当前机器可以解析的内容：workspace 根目录、workspace 规划路径、链接的 repos 或文件夹、缺失的路径、过时的本地注册表条目、存在时的 repo 本地 specs 路径，以及建议的修复。它报告问题但不自动修复。

Workspace 命令应该全局工作。当命令需要一个 workspace 而用户未指定时，OpenSpec 应该使用本地注册表显示交互式选择器。在非交互模式下，它应该失败并显示清晰的消息并建议 `--workspace <name>`。

规划依赖：

- 取决于 `workspace-foundation`。

## POC 发现

要保留的行为：

- `workspace setup` 是友好的 onboarding 路径。
- `workspace list` 使托管 workspaces 可被发现。
- 直接的自动化路径仍然有用，但它应该位于 `workspace setup --no-interactive` 下。
- 链接修复很有用，但 owner/handoff 元数据不应在此 slice 中延续。
- `workspace doctor` 是回答"OpenSpec 知道这个 workspace 的什么？"的正确答案。
- 共享 workspace 状态和本地路径分开存储。
- 当非交互式输入不完整时，设置干净地失败。
- 创建的 workspaces 忽略机器本地路径状态。

要改变的行为：

- POC 要求注册的 repos 已经包含 repo 本地的 `openspec/`。这应该成为实现就绪信号，而不是规划先决条件。
- POC 使用仅 repo 的语言。这个 slice 应该在用户面向的文本中使用"repos 或文件夹"。
- 公共命令应该是 `workspace link`，而不是 `workspace add-repo`。
- 修复命令应该是 `workspace relink`，而不是 `workspace update-repo`。
- 公共 `workspace create` 应该在第一次发布中移除。Setup 应该是创建流程。
- POC 的 `setup` 流程存储了首选 agent/open 行为。Agent 启动偏好属于 `workspace-open-agent-context`，不是这个 slice。
- 人类输出应该避免实现术语，如工作集、代码区域、条目、别名或本地覆盖。
- `setup` 应该至少需要一个链接的 repo 或文件夹，以便创建的 workspace 立即有用。

## 非目标

- 在此第一次发布中没有公共 `openspec workspace create` 命令。
- 无 workspace-open agent 启动行为。
- 无首选 agent 提示或保存的 agent 偏好。
- 无 owner 或 handoff 元数据字段。
- 无 workspace 变更创建或目标选择。
- 无 apply、verify、archive、branch 或 worktree 行为。
- 无链接的 repos 或文件夹具有 repo 本地 OpenSpec 状态的要求。
- 在 `workspace doctor` 中无自动修复行为。

## 能力

### 新能力

- `workspace-links`：让用户设置 workspace、链接 repos 或文件夹、列出已知的 workspaces，并在变更创建之前检查 workspace 解析。

### 修改的能力

- `cli-artifact-workflow`：引入在变更创建之前发生的 workspace 设置命令。

## 影响

- `openspec workspace setup`
- `openspec workspace list`
- `openspec workspace ls`
- `openspec workspace link`
- `openspec workspace relink`
- `openspec workspace doctor`
- 来自 `workspace-foundation` 的本地 workspace 注册表使用。
- 解释链接的 repos/folders 作为规划上下文而非实现承诺的文档和生成的指导。