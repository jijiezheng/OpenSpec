## 为什么

在用户创建 workspace 并注册 repos 后，他们需要使用 agent 打开该 workspace，并让 agent 立即理解工作集。

用户不应需要解释每个 repo 在哪里、哪些别名重要，或者他们当前是在规划还是实现。Workspace 应该提供该上下文。

## 什么变更

添加 workspace-open 体验：

```text
使用我的 agent 打开这个 workspace。
Agent 看到 workspace 根目录、注册 repos、当前变更和相关指令。
```

启动上下文应将稳定指导与动态运行时范围分开：

- 稳定行为尽可能属于 workspace 级 agent 指导
- 动态范围属于启动提示或等效运行时上下文
- 即使没有活动变更，注册 repos 也应可见
- 变更作用域会话应包括选定的变更和目标 repo 上下文

规划依赖：

- 取决于 `workspace-create-and-register-repos`。

## 能力

### 新能力

- `workspace-agent-context`：打开带有足够动态上下文的 workspace 会话，以便 agent 跨注册 repos 进行推理。

### 修改的能力

- `context-injection`：扩展上下文构建以包含 workspace 根目录、repo 注册表、活动 workspace 变更和选定的变更范围。

## 影响

- `openspec workspace open`
- Workspace 提示和 agent 启动上下文。
- 为 workspace 模式生成或提交的 agent 指导。
- 用于在 workspace 外打开、通过名称打开 workspace 和打开变更作用域 workspace 会话的测试。