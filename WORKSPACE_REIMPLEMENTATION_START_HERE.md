# 工作区重构从这里开始

这是为从事工作区重构的 agent 提供的 grep 友好入口点。

有用的搜索词：

```text
workspace reimplementation
workspace poc
workspace-poc
workspace reference guide
workspace roadmap
fresh agent
start here
```

## 从这里开始

按顺序阅读这些文件：

1. `WORKSPACE_REIMPLEMENTATION_DIRECTION.md`
2. `openspec/changes/workspace-reimplementation-roadmap/README.md`
3. `openspec/changes/workspace-reimplementation-roadmap/POC_REFERENCE_GUIDE.md`
4. 下一个实现切片的 proposal

POC 参考提交是：

```text
workspace-poc @ 79a45ac043f414e63d13e08b9da83b135cb20a39
```

将 POC 作为研究材料。不要将其合并到实现分支。除非切片 proposal 或设计明确决定，否则不要保留其架构。

## 实现顺序

按顺序实现这些扁平的 OpenSpec 变更：

1. `workspace-foundation`
2. `workspace-create-and-register-repos`
3. `workspace-open-agent-context`
4. `workspace-change-planning`
5. `workspace-apply-repo-slice`
6. `workspace-verify-and-archive`

`workspace-reimplementation-roadmap` 是计划的连续性和参考容器。

## 编辑之前

对于你即将实现的切片，使用 `POC_REFERENCE_GUIDE.md` 检查固定的 POC 提交，然后写下：

```text
<切片> 的 POC 发现：

要保留的用户行为：
- ...

值得翻译的测试或示例：
- ...

要避免的实现捷径：
- ...

悬而未决的设计问题：
- ...
```

在相关的 OpenSpec artifact 中捕获持久性发现，以便未来的会话不依赖于聊天历史。