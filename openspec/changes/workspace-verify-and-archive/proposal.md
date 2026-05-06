## 为什么

用户需要知道跨 repo workspace 变更是否完成，而不会将所有 repo 进度扁平化为一个模糊的完成状态。

期望的生命周期是：

```text
验证每个 repo slice。
查看哪些 slices 完成或仍待处理。
在适当的时候归档 repo 本地结果。
当跨 repo 目标完成时归档 workspace 变更。
```

验证和归档应该使用户的跨 repo 状态更清晰，而不是强迫他们推理内部工件放置。

## 什么变更

添加 workspace 感知的 verify 和 archive 行为：

- 验证 workspace 级变更结构和目标 repo 状态
- 显示每个 repo slice 的进度
- 支持需要时的 repo 本地归档工作
- 当协调目标完成时支持显式的 workspace 级归档
- 避免将部分 repo 完成视为完整 workspace 完成

规划依赖：

- 取决于 `workspace-apply-repo-slice`。

## 能力

### 新能力

- `workspace-verify-archive`：验证和归档带有每个 repo 进度可见性的 workspace 变更。

### 修改的能力

- `cli-archive`：添加 workspace 感知的归档语义。
- `opsx-verify-skill`：添加 workspace 验证指导。
- `opsx-archive-skill`：添加 workspace 归档指导。

## 影响

- Workspace 状态、verify 和归档行为。
- 每个 repo slice 完成报告。
- Workspace 级 hard-done 标记或等效归档状态。
- 部分完成、最终 workspace 归档以及与独立 repo 本地归档流程兼容性的测试。