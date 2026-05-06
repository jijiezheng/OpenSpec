## 为什么

OpenSpec 目前假设命令交付直接映射到命令适配器。这种假设并非对所有工具都成立。

Trae 是一个具体的例子：它通过技能条目调用 OpenSpec 工作流（例如 `/openspec-new-change`），而不是通过适配器生成的命令文件。在这个模型中，技能是命令表面。

今天，这造成了行为差距：

- `delivery=commands` 可以删除技能
- 没有适配器的工具跳过命令生成
- 结果：像 Trae 这样的选定工具可能最终没有任何可调用的工作流工件

这不仅仅是提示 UX 问题，因为非交互式和 CI 流程绕过交互式指导。我们需要在核心生成逻辑中需要一个能力感知模型。

## 什么变更

### 1. 添加明确的命令表面能力元数据

在工具元数据中添加可选字段以描述工具如何暴露命令：

- `adapter`：命令文件通过命令适配器生成
- `skills-invocable`：技能可直接调用为命令
- `none`：无 OpenSpec 命令表面

字段应该是可选的。默认行为从适配器注册存在推断：有注册适配器的工具解析为 `adapter`；没有适配器注册且没有明确注解的工具解析为 `none`。
能力值使用 kebab-case 字符串标记以与序列化元数据约定保持一致。

初始明确覆盖：

- Trae -> `skills-invocable`

### 2. 使交付行为能力感知

更新 `init` 和 `update` 以从以下计算每个工具的有效工件操作：

- 全局交付（`both | skills | commands`）
- 工具命令表面能力

行为矩阵：

- `both`：
  - 为所有有 `skillsDir` 的工具生成技能（包括 `skills-invocable`）
  - 仅对 `adapter` 工具生成命令文件
  - `none`：无工件操作；可能发出兼容性警告
- `skills`：
  - 为所有有 `skillsDir` 的工具生成技能（包括 `skills-invocable`）
  - 删除适配器生成的命令文件
  - `none`：无工件操作；可能发出兼容性警告
- `commands`：
  - `adapter`：生成命令，删除技能
  - `skills-invocable`：生成（或保留如果最新）技能作为命令表面；不要删除它们
  - `none`：快速失败并显示清晰错误

### 3. 添加 preflight 验证和更清晰的输出

在写入/删除工件之前，验证选定/配置的 tool 对抗交付模式：

- 交互式流程：在确认前显示清晰的兼容性说明
- 非交互式流程：用确定性错误失败，列出不兼容的工具和支持的替代方案

更新摘要以显示每个工具的有效交付结果（例如，当 commands 模式仍为 skills-invocable 工具安装技能时）。

### 4. 更新文档和测试

- 记录能力模型和 Trae 在交付模式下的行为
- 确保 CLI 文档和 supported-tools 文档反映有效行为
- 添加测试覆盖率：
  - `init --tools trae` 与 `delivery=commands`
  - 在 `delivery=commands` 下配置 Trae 时的 `update`
  - 混合选择（`claude + trae`）跨所有交付模式
  - 在 `delivery=commands` 下对无命令表面的工具的明确错误路径

### 5. 与安装范围行为协调

当与 `add-global-install-scope` 组合时，init/update 规划必须组合：

- 安装范围（`global | project`）
- 交付模式（`both | skills | commands`）
- 命令表面能力（`adapter | skills-invocable | none`）

实现测试应覆盖混合工具矩阵以确保当两个变更都 active 时的确定性行为。

## 能力

### 新能力

- `tool-command-surface`：将工具分类为 `adapter`、`skills-invocable` 或 `none` 的能力模型，以驱动交付行为

### 修改的能力

- `cli-init`：交付处理变得工具能力感知，具有 preflight 兼容性验证
- `cli-update`：交付同步变得工具能力感知，具有一致的兼容性验证和消息传递
- `supported-tools-docs`：记录非适配器工具的命令表面语义

## 影响

- `src/core/config.ts` - 添加可选命令表面元数据和 Trae 覆盖
- `src/core/command-generation/registry.ts`（或共享 helper）- 从适配器存在推断能力
- `src/core/init.ts` - 能力感知生成/删除规划 + 兼容性验证 + 摘要消息
- `src/core/update.ts` - 能力感知同步/删除规划 + 兼容性验证 + 摘要消息
- `src/core/shared/tool-detection.ts` - 包含能力感知检测，以便 `skills-invocable` 工具在 `delivery=commands` 下保持可检测，并且 `none` 工具从命令表面工件检测中排除
- `docs/supported-tools.md` 和 `docs/cli.md` - 记录交付行为和兼容性说明
- `test/core/init.test.ts` 和 `test/core/update.test.ts` - 为 skills-invocable 行为和混合工具交付场景添加覆盖率

## 排序说明

- 此变更旨在通过为 init/update 引入附加的、能力特定的需求来安全地与 `simplify-skill-installation` 堆叠
- 如果 `simplify-skill-installation` 先合并，此变更应 rebase 并将能力感知规则保留为 `delivery=commands` 行为在 `skills-invocable` 工具上的真实来源
- 如果此变更先合并，`simplify-skill-installation` 分支应 rebase 以避免重新引入全局"commands-only 意味着所有工具无技能"的假设
- 如果 `add-global-install-scope` 先合并，此变更应 rebase 以在该变更的范围解析路径决策之上组合能力感知行为
- 如果此变更先合并，`add-global-install-scope` 应 rebase 以保留第 5 节组合规则（`install scope` + `delivery mode` + `command surface capability`），而不覆盖能力感知的命令表面结果