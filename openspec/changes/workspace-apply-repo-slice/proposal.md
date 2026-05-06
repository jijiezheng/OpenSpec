## 为什么

在 workspace 提案存在后，用户需要一个实际的方式来一次实现一个 repo slice。

在正确的 workspace 模型中，apply 意味着实现：

```text
获取选定的 workspace 变更。
获取选定的 repo slice。
打开或使用正确的 checkout。
在保留 workspace 计划的同时实现该 slice。
```

它不应该意味着将规划文件复制或物化到每个 repo 作为用户面向的工作流。

## 什么变更

添加 workspace 变更的 repo-slice apply 工作流：

- 选择一个 workspace 变更
- 选择一个目标 repo 别名
- 解析该别名的本地 checkout
- 为 agent 提供 workspace 计划和 repo 特定的实现上下文
- 在不让 workspace 失去计划所有权的情况下跟踪进度

工作流应支持跨独立分支或会话的实现，同时保持 workspace 提案作为连续性层。

规划依赖：

- 取决于 `workspace-change-planning`。

## 能力

### 新能力

- `workspace-repo-slice-apply`：将 workspace 变更的一个 repo slice 作为实现工作流应用。

### 修改的能力

- `cli-artifact-workflow`：将 workspace apply 定义为实现而非物化。
- `context-injection`：从 workspace 变更提供 repo 特定的实现上下文。

## 影响

- Workspace apply 命令行为。
- Repo-slice 实现的 agent 交接文本。
- 本地 checkout 解析和 branch/worktree 假设。
- 测试 apply 操作在一个目标 repo slice 上，不需要将 workspace 规划工件复制作为主要用户契约。