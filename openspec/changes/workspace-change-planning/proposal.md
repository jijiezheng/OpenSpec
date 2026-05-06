## 为什么

一旦 repos 可见且 agent 有 workspace 上下文，用户应该能够规划跨 repo 变更，而无需立即物化 repo 本地工件。

用户目标是：

```text
跨 repos 探索产品目标。
决定范围。
创建一个识别 repo slices 的 workspace 级提案。
```

规划应该是承诺点。Repo 可见性本身应该保持轻量。

## 什么变更

添加 workspace 级变更规划：

- 从协调根创建 workspace 变更
- 一次性捕获产品目标
- 按注册别名识别目标 repos
- 让 agent 在提交实现 slices 之前进行探索
- 将 workspace 保持为规划的真实来源

这个 slice 应该避免重建 POC 的物化优先行为。不应仅仅因为 workspace 变更存在就创建 repo 本地工件。

规划依赖：

- 取决于 `workspace-open-agent-context`。

## 能力

### 新能力

- `workspace-change-planning`：为跨 repo 目标创建和管理 workspace 级提案。

### 修改的能力

- `change-creation`：添加 workspace 感知的变更创建语义和目标 repo 选择。
- `openspec-conventions`：定义 workspace 级规划和 repo 本地实现工作之间的关系。

## 影响

- Workspace 变更创建。
- 目标 repo 元数据和验证。
- 用于提议跨 repo 变更的 agent 指令。
- 测试在变更创建之前 repos 是可见的，以及创建变更不意味着 repo 本地物化。