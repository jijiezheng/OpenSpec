# Workspace 重新实现路线图

此变更是跨多个会话和分支重新实现 workspace 支持的连续性层。

新鲜 agent 的根入口点：`WORKSPACE_REIMPLEMENTATION_START_HERE.md`。

我们正在实现的用户旅程是：

```text
创建 workspace
  -> 添加 repos
  -> 使用 agent 上下文打开 workspace
  -> 规划跨 repo 变更
  -> 实现一个 repo slice
  -> 验证和归档
```

POC 分支仅作为参考材料：

```text
workspace-poc @ 79a45ac043f414e63d13e08b9da83b135cb20a39
```

用它来理解行为、测试和经验教训。不要默认合并它或保留其架构。该分支的完整源方向文档在仓库根目录复制为 `WORKSPACE_REIMPLEMENTATION_DIRECTION.md`。

新鲜 agent 在实现任何 slice 之前应阅读 `POC_REFERENCE_GUIDE.md`。该指南解释如何检查固定的 POC 提交、每个 slice 应阅读哪些文件，以及将哪些发现带回 OpenSpec 工件。

## 变更顺序

按此顺序实现平面兄弟变更：

1. `workspace-foundation`
2. `workspace-create-and-register-repos`
3. `workspace-open-agent-context`
4. `workspace-change-planning`
5. `workspace-apply-repo-slice`
6. `workspace-verify-and-archive`

OpenSpec 当前将活动变更发现为 `openspec/changes/` 下的即时目录，变更名称是 kebab-case 标识符。将这些变更保持为平面兄弟，直到正式的变更堆叠元数据可用。

## 依赖注释

`workspace-foundation` 建立存储、根检测和命名模型。每个后来的 slice 应该在该模型之上构建，而不是重新定义 workspace 元数据。

`workspace-create-and-register-repos` 创建 workspace 并在变更存在之前使链接的 repos 或文件夹可见。链接项可以是完整的 repos、monorepo 模块或仅用于规划的代码区域。这保留了产品规则，即 workspace 可见性不是变更承诺。

`workspace-open-agent-context` 给 agent 提供 workspace 根目录、链接的 repos 或文件夹、活动变更和选定的变更范围。

`workspace-change-planning` 创建 workspace 级规划承诺并识别目标 repo slices。

`workspace-apply-repo-slice` 将 apply 视为对一个选定的 repo slice 的实现，而不是 workspace 规划文件的物化。

`workspace-verify-and-archive` 使跨 repo 进度可见，并将部分 repo 完成与最终 workspace 完成分开。

## 会话交接提示

在未来的实现会话开始时使用此提示：

```text
继续 workspace 重新实现路线图。首先阅读
openspec/changes/workspace-reimplementation-roadmap/README.md 和
openspec/changes/workspace-reimplementation-roadmap/POC_REFERENCE_GUIDE.md，
然后按顺序选择下一个未完成的平面兄弟变更。使用
workspace-poc at 79a45ac043f414e63d13e08b9da83b135cb20a39 作为参考材料。
保留预期行为，但从当前基础重新实现。在编辑之前，总结该 slice 的 POC 发现。
```

## 分支指导

每个兄弟变更可以在自己的分支或 PR 上实现。将影响后来 slice 的决定保留在此 README 或相关提案中，以便未来会话不依赖于聊天历史。