# 工作区重构方向

日期：2026-04-30

新鲜 agent 入口点：首先阅读 `WORKSPACE_REIMPLEMENTATION_START_HERE.md`，然后返回本文档获取完整的产品方向。

本文档记录了我们从工作区 POC 中学到的内容，用于从零开始重新实现 OpenSpec 工作区支持的方向。

重新实现应该围绕真实用户通过 OpenSpec 的路径排序：

```text
创建工作区
  -> 添加仓库
  -> 打开工作区
  -> 跨仓库探索
  -> 创建提案
  -> 应用一个仓库切片
  -> 验证
  -> 归档
```

目标不是重建每个 POC 机制。目标是让用户面对的能力一次一个，按用户自然创建、实现、验证和归档变更的相同顺序。

## 北极星

用户应该这样想：

```text
我有一个多仓库产品目标。
我创建一个 OpenSpec 工作区。
我用我的 agent 打开它。
agent 可以看到已注册的仓库。
我们探索直到范围清晰。
然后我们创建一个提案。
然后我们一次实现一个仓库切片。
```

而不应该这样想：

```text
我需要创建一个变更以便仓库变得可见。
我需要将仓库本地的 artifact 具体化。
我需要理解工作区覆盖层。
我需要将目标元数据与提案文件分开管理。
```

核心产品规则是：

```text
仓库可见性不是变更承诺。
```

已注册的仓库是工作区的工作集。创建变更是计划承诺。应用变更是实现工作流。

## 构建顺序

### 1. 工作区创建

首先让工作区创建变得平凡而坚实。

用户目标：

```text
创建一个存放跨仓库规划的地方。
```

预期表面：

```bash
openspec workspace create my-workspace
openspec workspace add-repo openspec /path/to/openspec
openspec workspace add-repo landing /path/to/openspec-landing
```

预期结果：

```text
workspace/
  AGENTS.md
  changes/
  .openspec-workspace/
```

产品决策：

- 使用 `.openspec-workspace/`，而不是 `.openspec/`，用于工作区元数据。
- 将 `changes/` 保持在工作区根目录可见。
- 将已注册的仓库视为工作区工作集。
- 让 `doctor` 显示人类可读的仓库名称和解析后的路径。

推迟：

- 分支。
- 工作树。
- 应用。
- 归档。
- 复杂的目标生命周期。

完成标准：用户可以创建工作区、注册仓库，并运行 `doctor` 查看 OpenSpec 确切知道的内容。

### 2. 工作区打开

接下来让工作区以用户期望的方式打开。

用户目标：

```text
用我的编码 agent 打开这个多仓库工作集。
```

预期表面：

```bash
openspec workspace open
openspec workspace open --agent codex
openspec workspace open --agent github-copilot
```

产品行为：

- `workspace open` 打开协调工作区以及已注册的仓库。
- 仓库可见性是默认的。
- 变更选择是可选的焦点，不是仓库访问的机制。
- `--agent` 默认为一次会话覆盖。保存首选 agent 需要明确的偏好设置操作。

对于 GitHub Copilot，生成或打开一个 `.code-workspace` 文件，包含：

```text
workspace root
registered repo A
registered repo B
```

对于 Claude 和 Codex，通过 agent 支持的机制附加已注册的仓库目录。

推迟：

- `workspace open --change`。
- 会话内升级流程。
- 每变更附件限制。

完成标准：打开工作区让 agent 可以看到协调根目录和所有已注册的仓库。

### 3. Agent 引导和探索

然后让探索工作。

用户目标：

```text
告诉 agent 一个粗略的产品目标，让它在创建提案之前检查仓库。
```

预期的用户提示：

```text
探索我们应该如何让 OpenSpec 文档在落地页上可用。
跨已注册的仓库探索，但先不实现。
```

Agent 行为：

- 理解它处于工作区模式。
- 检查已注册的仓库。
- 解释可能受影响的仓库。
- 仅在需要时才要求澄清。
- 在探索期间避免实现编辑。

构建：

- 工作区级 `AGENTS.md` 指导。
- 工作区会话中的正常 OpenSpec 技能和命令。
- 在正常 `/explore` 之上分层的工作区特定指导，而不是替换它。

推迟：

- 提案 artifact 生成。
- 目标确认命令。
- 应用上下文提供器。

完成标准：用户可以打开工作区并运行有用的跨仓库探索，而不创建虚拟变更。

### 4. 提案创建

只有在探索工作之后，才构建提案创建。

用户目标：

```text
既然我们了解了范围，就捕获计划。
```

预期用户提示：

```text
为此变更创建一个提案。
针对实际受影响的仓库。
```

首选 artifact 形状：

```text
changes/integrate-docs/
  proposal.md
  design.md
  tasks.md
  specs/
    openspec/
      docs-conventions/spec.md
    landing/
      docs-routing/spec.md
```

关键工作流规则：

```text
/explore 可能让目标未知.
/propose 可能发现目标
/propose 必须在说准备好应用之前确认目标
```

目标应该尽可能由提案 artifact 本身表示。如果有 `specs/landing/...`，那么 `landing` 在范围内。避免单独的必需 `targets: [...]` 元数据列表作为活动真相来源。

推迟：

- 仓库本地具体化。
- 工作树选择。
- 多仓库实现。
- 归档。

完成标准：用户可以探索，然后创建具有仓库范围 specs 和 tasks 的工作区提案。

### 5. 状态

在实现之前，让状态变得出色。

用户目标：

```text
我们在哪里，涉及哪些仓库，这准备好实现了吗？
```

预期表面：

```bash
openspec status
openspec status --change integrate-docs
```

人类输出应回答：

```text
变更：integrate-docs
范围：openspec, landing
提案：存在
设计：存在
任务：存在
准备好应用：是否
```

状态还应捕获结构性错误：

- `specs/` 下未知的仓库文件夹。
- 缺失的任务。
- 没有确认的受影响仓库。
- 已注册的仓库路径缺失。

完成标准：agent 和用户可以在应用之前信任状态。

### 6. 应用一个仓库切片

现在才构建 `/apply`。

用户目标：

```text
实现一个仓库的计划切片。
```

预期用户提示：

```text
/apply integrate-docs for landing
```

产品合约：

```text
/apply 意味着实现。
```

它不意味着：

```text
复制规划文件
将仓库本地的 OpenSpec 状态具体化
首次创建提案文件
```

Agent 行为：

1. 向 OpenSpec 请求应用上下文。
2. 阅读提案、设计、任务和相关 specs。
3. 确认目标仓库 checkout。
4. 仅编辑该仓库。
5. 更新工作区任务。
6. 运行相关检查。

这可能需要在内部使用规范化上下文命令，但那是支持机制：

```json
{
  "mode": "workspace",
  "change": "integrate-docs",
  "target": "landing",
  "implementationRoot": "/repos/openspec-landing",
  "contextFiles": [
    "changes/integrate-docs/proposal.md",
    "changes/integrate-docs/design.md",
    "changes/integrate-docs/tasks.md",
    "changes/integrate-docs/specs/landing/docs-routing/spec.md"
  ],
  "allowedEditRoots": [
    "/repos/openspec-landing"
  ],
  "tasksFile": "changes/integrate-docs/tasks.md"
}
```

推迟：

- 一次应用多个仓库。
- 自动分支创建。
- 工作树管理。
- 仓库本地 OpenSpec 镜像。

完成标准：可以从中央工作区计划实现一个仓库切片。

### 7. 验证

然后构建验证。

用户目标：

```text
检查已实现的仓库切片是否满足计划。
```

预期提示：

```text
/verify integrate-docs for landing
```

行为：

- 读取与 `/apply` 相同的规范化上下文。
- 检查实现 checkout。
- 检查该仓库的任务和 specs。
- 运行仓库验证。
- 报告差距清晰。

默认行为应验证一个仓库切片。全工作区验证可以稍后出现。

完成标准：用户可以针对中央工作区计划验证一个已实现的仓库切片。

### 8. 归档

在第一个完整循环中最后是归档。

用户目标：

```text
变更完成了。将其从活动规划中移出。
```

预期提示：

```text
/archive integrate-docs
```

行为：

- 要求所有目标仓库切片完成或明确接受。
- 归档工作区变更。
- 不要求仓库本地的规划副本，除非 OpenSpec 稍后决定仓库本地归档重要。

完成标准：用户可以完成完整生命周期：

```text
创建工作区
  -> 打开
  -> 探索
  -> 提案
  -> 应用仓库 A
  -> 应用仓库 B
  -> 验证
  -> 归档
```

## 实现纪律

仅构建下一个用户可见的步骤。

序列应保持基于这些问题的接地：

```text
1. 我可以创建工作区吗？
2. 我可以看到我的仓库吗？
3. 我的 agent 可以探索它们吗？
4. 我们可以捕获提案吗？
5. 状态可以告诉我们是否准备好吗？
6. agent 可以实现一个仓库切片吗？
7. 我们可以验证它吗？
8. 我们可以归档它吗？
```

除非下一个用户可见能力需要，否则不要从内部抽象开始。

不要从以下开始：

- 目标元数据机制。
- 具体化。
- 适配器抽象。
- 分支编排。
- 工作树编排。
- 多仓库应用。

这些以后可能很重要，但它们不应该定义第一个重新实现路径。

## 产品形态

工作区应该感觉像 OpenSpec 正常工作流跨多个仓库的延伸，而不是第二个具有自己生命周期的产品。

持久性产品模型是：

```text
workspace = 中央规划真相来源
registered repos = 可见工作集
proposal = 范围规划承诺
repo target = 计划中一个受影响的仓库
branch/worktree = 实现 checkout
/apply = 实现一个选定的仓库切片
```

保持用户旅程简单：

```text
打开工作区。
让 agent 探索。
当范围清晰时创建提案。
一次实现一个仓库切片。
验证。
归档。
```