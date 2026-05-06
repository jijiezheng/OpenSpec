## 为什么

OpenSpec 安装路径当前不一致：

- 大多数技能和命令写入项目本地目录。
- Codex 命令已经是全局的（`$CODEX_HOME/prompts` 或 `~/.codex/prompts`）。
- 用户无法在工具间选择一致的安装范围策略。

这为偏好用户级设置并默认期望工具工件被全局管理的用户创造了摩擦。

## 什么变更

### 1. 添加具有遗留安全默认值的安装范围偏好

引入具有两种模式的全局安装范围设置：

- `global`（新创建配置的默认）
- `project`

设置存储在全局配置中，可以按命令运行覆盖。
对于 `installScope` 缺失的 schema 演化的遗留配置，有效默认值保持为 `project`，直到用户选择加入全局范围。

### 2. 为技能和命令添加范围感知路径解析

重构路径解析，以便 `init` 和 `update` 从以下计算安装目标：

- 选定的范围偏好（`global` 或 `project`）
- 工具能力元数据（每个工具/表面支持哪些范围）
- 运行时上下文（项目根、主目录、环境覆盖）

### 3. 为范围支持添加工具级能力元数据

扩展工具元数据以明确声明每个表面的范围支持：

- 技能范围支持
- 命令范围支持

当首选范围不支持工具/表面时，系统使用确定性回退规则并在输出中报告有效范围。

### 4. 使命令生成上下文感知

扩展命令适配器路径解析，以便适配器接收安装上下文（范围 + 环境上下文），而不是仅命令 ID。这消除了特殊处理并允许跨工具的一致范围行为。

### 5. 更新 init/update UX 和行为

- `openspec init`：
  - 接受范围覆盖标志
  - 使用配置的范围或迁移感知默认值（新配置默认为 global；遗留配置保留 project 直到迁移）
  - 应用范围感知生成和清理规划
- `openspec update`：
  - 应用当前范围偏好
  - 按工具/表面同步有效范围内的工件
  - 跟踪每个工具/表面最后成功的有效范围以进行确定性范围漂移检测
  - 清楚报告有效范围决策

### 6. 扩展配置 UX 和文档

- 在 `openspec config profile` 交互式流程中添加安装范围控制。
- 扩展 `openspec config list` 输出，添加安装范围来源（`explicit`、`new-default`、`legacy-default`）。
- 添加明确的迁移指导和提示路径，以便遗留用户选择加入 `global` 范围。
- 更新支持的工具和 CLI 文档以解释范围行为和回退规则。

### 7. 与命令表面能力交付规则协调

`cli-init` 和 `cli-update` 规划应组合：

- 安装范围（`global | project`）
- 交付模式（`both | skills | commands`）
- 命令表面能力（`adapter | skills-invocable | none`）

本提案专注于范围解析，但实现和测试覆盖率应包括混合工具案例，以避免与 `add-tool-command-surface-capabilities` 组合时的回归。

## 能力

### 新能力

- `installation-scope`：工具工件安装的范围偏好模型和有效范围解析。

### 修改的能力

- `global-config`：使用 schema 演化默认值持久化安装范围偏好。
- `cli-config`：配置和检查安装范围偏好。
- `ai-tool-paths`：添加工具级范围支持元数据和路径策略。
- `command-generation`：通过安装上下文的范围感知适配器路径解析。
- `cli-init`：范围感知初始化规划和输出。
- `cli-update`：范围感知更新同步、漂移检测和输出。
- `migration`：具有范围感知工作流查找的范围感知迁移扫描。

## 影响

- `src/core/global-config.ts` - 新的安装范围字段和默认值
- `src/core/config-schema.ts` - 安装范围配置键的验证支持
- `src/commands/config.ts` - 用于安装范围的交互式 profile/config UX 添加
- `src/core/config.ts` - 工具范围能力元数据
- `src/core/available-tools.ts` 和 `src/core/shared/tool-detection.ts` - 范围感知配置检测
- `src/core/command-generation/types.ts` 和适配器实现 - 上下文感知文件路径解析
- `src/core/init.ts` - 范围感知生成/删除规划
- `src/core/update.ts` - 范围感知同步/删除/漂移规划
- `src/core/migration.ts` - 范围感知工作流扫描支持
- `docs/supported-tools.md` 和 `docs/cli.md` - 安装范围行为文档
- `test/core/init.test.ts`、`test/core/update.test.ts`、适配器测试、配置测试 - 范围覆盖率