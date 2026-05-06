## 为什么

并行变更经常触及相同的能力和 `cli-init`/`cli-update` 行为，但目前没有机器可读的方式来表达排序、依赖关系或预期的合并顺序。

这造成了三个 recurring 问题：

- 团队无法判断哪个变更应首先落地
- 大型变更难以拆分为安全的可合并 slices
- 并行工作可能无意中重新引入已被另一个变更移除的假设

我们需要轻量级的规划元数据和 CLI 指导，以便贡献者可以安全地将计划堆叠在彼此之上。

## 什么变更

### 1. 为变更添加轻量级堆栈元数据

扩展变更元数据以支持排序和分解上下文，例如：

- `dependsOn`：必须首先落地的变更
- `provides`：此变更暴露的能力标记
- `requires`：此变更所需的能力标记
- `touches`：可能受影响的能力/spec 区域（仅作为 advisory；警告信号，不是硬依赖）
- `parent`：用于拆分工作的可选父变更

元数据是可选的，对现有变更向后兼容。

排序语义：

- `dependsOn` 是执行/归档排序的真实来源
- `provides`/`requires` 是验证和规划可见性的能力契约
- `provides`/`requires` 不创建隐式依赖边；作者仍需通过 `dependsOn` 声明所需的排序

### 2. 添加堆栈感知验证

增强变更验证以 early 检测规划问题：

- 缺失的依赖
- 依赖循环
- 归档排序违规（例如，试图在所有 `dependsOn` 前驱归档之前归档变更）
- 不匹配的能力标记（例如，带有提供者历史中无提供者的 `requires` 标记发出非阻塞警告）
- 当活动变更触及相同能力时的重叠警告

验证仅对确定性阻止者（例如循环或缺失必需依赖）失败，并将重叠检查作为可操作的警告保留。

### 3. 添加排序可见性命令

添加轻量级 CLI 支持以检查和执行计划顺序：

- `openspec change graph` 显示依赖 DAG/顺序
- `openspec change graph` 首先验证循环；当存在循环时，以与堆栈感知验证相同的确定性循环错误失败
- `openspec change next` 建议准备好实现/归档的无阻塞变更

### 4. 为大型变更添加拆分脚手架

添加 helper 工作流以将大型提案分解为可堆叠的 slices：

- `openspec change split <change-id>` 用 `parent` + `dependsOn` 构建子变更脚手架
- 为每个子 slice 生成最小的 proposal/tasks 存根
- 将源变更转换为父规划容器（无重复子实现任务）
- 重新运行已拆分源变更的 split 返回确定性可操作错误，除非传递 `--overwrite`（别名 `--force`）
- `--overwrite` / `--force` 完全重新生成拆分的托管子脚手架存根和元数据链接，替换先前的脚手架内容

### 5. 记录堆栈优先工作流

更新文档以描述：

- 如何建模依赖关系和父/子 slices
- 何时拆分大型变更
- 如何在并行开发期间使用 graph/next 验证信号
- `openspec/changes/IMPLEMENTATION_ORDER.md` 的迁移指导：
  - 机器可读的变更元数据成为规范依赖来源
  - `IMPLEMENTATION_ORDER.md` 在过渡期间保持可选的叙述性上下文

## 能力

### 新能力

- `change-stacking-workflow`：变更规划的依赖感知排序和拆分脚手架

### 修改的能力

- `cli-change`：添加 graph/next/split 规划命令和堆栈感知验证消息
- `change-creation`：创建或拆分变更时支持 parent/dependency 元数据
- `openspec-conventions`：定义变更提案的可选堆栈元数据约定

## 影响

- `src/core/project-config.ts` 和变更元数据加载的相关解析/验证工具
- `src/core/config-schema.ts`（或专用变更 schema）用于堆栈元数据验证
- `src/commands/change.ts` 和/或 `src/core/list.ts` 用于 graph/next/split 命令行为
- `src/core/validation/*` 用于依赖循环和重叠检查
- `docs/cli.md`、`docs/concepts.md` 和贡献者指导用于堆栈感知工作流
- 元数据解析、图排序、下一项建议和拆分脚手架的测试