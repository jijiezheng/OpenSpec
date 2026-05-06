## 为什么

Workspace 支持需要作为用户面向的工作流重新实现，而不是作为概念验证的直接端口。

用户应该能够说他们有多 repo 产品目标，创建一个 workspace，添加相关的 repos，使用 agent 打开该 workspace，规划变更，一次实现一个 repo slice，验证它，并归档它。POC 分支捕获了有用的行为和发现，但其实现应该保留为参考材料而不是基础架构。

这个路线图还需要跨多个会话和分支存活。当前的 OpenSpec 变更发现将活动变更视为 `openspec/changes/` 下的平面即时目录，变更名称是 kebab-case 标识符而不是嵌套路径。因此，这个变更是带有兄弟提案变更的平面规划容器，而不是嵌套子变更。

参考材料：

- `workspace-poc` 在 `79a45ac043f414e63d13e08b9da83b135cb20a39`
- 该分支上的 `WORKSPACE_REIMPLEMENTATION_DIRECTION.md`
- 该分支上的 `WORKSPACE_POC_FOLLOWUP_NOTES.md`

## 什么变更

添加一个轻量级路线图，用于将 workspace 支持重新实现为一堆平面兄弟 OpenSpec 变更：

- `workspace-foundation`
- `workspace-create-and-register-repos`
- `workspace-open-agent-context`
- `workspace-change-planning`
- `workspace-apply-repo-slice`
- `workspace-verify-and-archive`

每个兄弟变更拥有 lived 用户旅程中的一个步骤。依赖关系记录在提案正文中。一旦变更堆叠元数据落地，这个路线图可以迁移到显式的 `parent` 和 `dependsOn` 元数据。

预期顺序是：

```text
workspace-foundation
  -> workspace-create-and-register-repos
  -> workspace-open-agent-context
  -> workspace-change-planning
  -> workspace-apply-repo-slice
  -> workspace-verify-and-archive
```

## 能力

### 新能力

- `workspace-reimplementation-roadmap`：跨多个平面 OpenSpec 变更协调 workspace 重新实现计划。

### 修改的能力

- `openspec-conventions`：澄清这个 workspace 工作使用平面兄弟变更，直到支持嵌套或堆叠变更元数据。

## 影响

- 仅此 PR 中的规划。
- 未来变更将影响 workspace 元数据、workspace CLI 流程、agent 上下文构建、workspace 变更规划、repo-slice 应用、验证和归档行为。
- 此路线图提案不引入运行时行为变更。