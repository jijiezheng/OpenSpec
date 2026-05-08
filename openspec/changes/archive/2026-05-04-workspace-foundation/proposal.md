## 为什么

用户需要 workspace 感觉像是跨多个 repos 或文件夹规划的明显之家。

他们应该能够这样想：

```text
我有经常一起规划的 repos 或文件夹。
我创建一个 OpenSpec workspace。
那个 workspace 是变更所在的地方。
我的代码保持在原处。
OpenSpec 将 workspace 链接到那些本地路径。
```

Workspace 不是功能。它是跨 repo 规划的持久协调之家。单个功能、修复和项目是 workspace 内的变更。

用户不应被迫选择存储位置、 early 创建变更或在 OpenSpec 可以 orient 自身之前了解内部 workspace 状态。

POC 证明了 workspace 状态是有用的。这个重新实现应该把它变成一个用户和 agent 可以在没有特殊 case 词汇的情况下解释的简单产品模型。

## 什么变更

此变更定义了 OpenSpec workspaces 的用户面向基础。

OpenSpec workspace 有一个可识别的规划之家：

```text
workspace-root/
  changes/
  .openspec-workspace/
```

`changes/` 是 workspace 级规划所在的地方。`.openspec-workspace/` 将目录标识为 OpenSpec workspace 并存储 workspace 状态。

OpenSpec 管理的 workspaces 位于一个标准位置：

```text
<global-data-dir>/workspaces/
```

用户不需要选择那个位置。OpenSpec 仍在设置后显示 workspace 路径，以便用户知道规划文件所在的位置。这个基础 slice 不提供用于更改托管 workspace 存储的特定于 workspace 的环境变量或配置覆盖。

OpenSpec 还保留了一个已知 workspaces 的轻量级本地注册表。注册表为全局命令、选择器和列表提供支持，但每个 workspace 文件夹仍然是真实来源。

Workspace 状态按用户期望分割：

- 可以跨机器移动的共享 workspace 信息
- 本地 checkout 路径保持在本地
- 链接的 repos 和文件夹通过稳定的链接名称引用，而不是绝对路径

链接路径可以是完整的 repo、monorepo 内的文件夹或 workspace 应该规划的其他现有文件夹。链接路径不需要在包含在 workspace 规划之前具有 repo 本地的 `openspec/` 状态。Repo 本地的 OpenSpec 状态可能在以后用于实现、验证或归档工作，但不是规划可见性的先决条件。

支持原生 Windows/PowerShell 和 WSL2。每种运行时使用自己的路径约定。OpenSpec 在这个基础 slice 中不翻译 Windows 和 WSL 之间的路径。

## 结果

在此变更之后，后续 workspace 功能可以依赖一个清晰的产品契约：

- OpenSpec 可以判断用户是否在 workspace 内。
- OpenSpec 知道默认在哪里创建托管 workspaces。
- OpenSpec 可以保留已知 workspaces 的本地注册表。
- Workspace 有一个可见的规划区域：`changes/`。
- Workspace 状态与 repo 本地 `openspec/` 状态可区分。
- 共享 workspace 状态不会将一个用户的本地路径强加给另一个用户。
- Workspace 规划可以通过稳定的链接名称引用现有 repos 或文件夹。
- 链接的 repos 或文件夹不需要 repo 本地 OpenSpec 状态用于 workspace 规划。
- 多 repo 和大型 monorepo 工作可以使用相同的 workspace 规划模型。
- Repo 拥有的 specs 和实现仍归其 repos 或源区域所有。
- Windows、PowerShell 和 WSL2 路径行为可预测。

此变更不提供完整的 workspace 工作流。它为 `workspace-create-and-register-repos` 提供了添加第一个用户面向命令所需的基础。

## POC 发现

要保留的行为：

- Workspace 是跨 repo 规划的持久协调之家。
- Workspace 在其根目录有一个可见的 `changes/` 目录。
- 链接的 repos 和文件夹提供 workspace 可以规划上下文的上下文。
- 稳定的链接名称比本地 checkout 路径更重要。
- 本地机器路径不应成为共享 workspace 状态。
- 规范 specs 和实现仍属于其拥有 repos。
- 链接名称在使用中很重要。

要携带的教训：

- POC 的隐藏 `.openspec/` workspace 元数据形状使 workspace 状态太容易与 repo 本地 OpenSpec 状态混淆。
- 用户不需要在 workspace 根目录内运行 repo 本地的 `openspec init`。
- POC 要求注册的 repos 已有 `openspec/` 对于规划来说太严格了。Repos 和文件夹应该在采用 repo 本地 OpenSpec 状态之前可链接。
- Repo 或文件夹可见性不应取决于创建变更。
- Workspace 设置不应暗示 repo 本地实现、branch、worktree、apply、verify 或归档行为。
- `add-repo` 对于用户面向的模型来说太窄了。链接现有 repo 或文件夹更清晰。

## 决策

- Workspace 身份目录：`.openspec-workspace/`。
- Workspace 身份文件：`.openspec-workspace/workspace.yaml`。
- Workspace 名称：当前 OS 的有效文件夹名称，排除空名称、`.`/`..` 和路径分隔符。
- Workspace 名称使用：存储在 `workspace.yaml` 中，用作默认托管 workspace 文件夹名称，并用作本地注册表名称。
- 规划表面：顶层 `changes/`。
- 本地机器状态：`.openspec-workspace/local.yaml`。
- 本地机器状态排除：OpenSpec 创建的 workspaces 默认从便携式协作状态中排除 `.openspec-workspace/local.yaml`。
- 本地 workspace 注册表：`<global-data-dir>/workspaces/registry.yaml`。
- 默认 workspace 基础：`<global-data-dir>/workspaces/`。
- 平台行为：原生 Windows 和 WSL2 各自使用运行 OpenSpec 的运行时的路径约定。
- 链接路径可以是完整的 repos、monorepo 文件夹或其他现有文件夹。
- 链接名称：非空稳定名称，在 workspace 内唯一，排除 `.`/`..` 和路径分隔符。
- Repo 本地 `openspec/` 状态不是 workspace 规划可见性的必需项。
- 链接仅记录关系；它不在链接的 repo 或文件夹中创建、复制、移动、初始化或编辑文件。

规划依赖：

- 无。这是第一个实现 slice。

## 非目标

- 尚无完整的 `openspec workspace setup`、`openspec workspace link` 或 `openspec workspace relink` 流程。
- 在第一个用户面向的 workspace 流程中尚无公共 `openspec workspace create` 命令。
- 用于更改标准 workspace 位置的用户面向命令、环境变量或配置设置尚无。
- 没有询问用户 OpenSpec 默认应该在哪里存储 workspaces 的问题。
- 无自动 Windows 到 WSL 或 WSL 到 Windows 路径转换。
- 无 workspace-open agent 启动行为。
- 无 workspace 级提案创建。
- 无 repo-slice apply、verify、archive、branch 或 worktree 行为。
- 不将 workspace 规划文件复制到链接的 repos 或文件夹中，作为创建、检测或链接 workspace 的副作用。

## 能力

### 新能力

- `workspace-foundation`：定义 OpenSpec workspaces 的产品基础。

### 修改的能力

- `openspec-conventions`：描述协调 workspaces 与 repo 本地 OpenSpec 项目的区别。

## 影响

- Workspace 识别和路径行为。
- Workspace 状态解析。
- 本地 workspace 注册表解析。
- Workspace 心理模型的文档和 agent 指导。
- 后续 workspace slices 应在此契约之上构建，而不是重新定义 workspace 存储、身份、注册表或路径行为。