## 为什么

用户抱怨技能/命令太多（目前 10 个），新用户感到不知所措。我们想在保留高级用户能力的同时简化默认体验，同时保持向后兼容性。

目标：**让用户在 1 分钟内达到"啊哈时刻"**。

```text
0:00  $ openspec init
      ✓ 完成。运行 /opsx:propose "你的想法"

0:15  /opsx:propose "添加用户认证"

0:45  Agent 创建 proposal.md、design.md、tasks.md
      "哇，它为我规划了整个事情" ← 啊哈

1:00  /opsx:apply
```

此外，用户对工作流交付方式（技能 vs 命令 vs 两者）有不同的偏好，但这应该是高级用户配置，而不是新用户需要考虑的事情。

## 什么变更

### 1. 智能默认 Init

Init 自动检测工具并请求确认：

```text
$ openspec init

检测到的工具：
  [x] Claude Code
  [x] Cursor
  [ ] Windsurf

按 Enter 确认，或 Space 切换

正在设置 OpenSpec...
✓ 完成

开始你的第一个变更：
  /opsx:propose "添加深色模式"
```

**没有 profile 或 delivery 的提示。** 默认值是：
- Profile: core
- Delivery: both

高级用户可以通过 `openspec config profile` 进行自定义。

### 2. 工具检测行为

Init 扫描现有工具目录（`.claude/`、`.cursor/` 等）：
- **检测到工具（交互式）：** 显示预选复选框，用户确认或调整
- **未检测到工具（交互式）：** 提示完整工具选择
- **非交互式（CI）：** 自动使用检测到的工具，如果没有检测到则失败

### 3. 修复工具选择 UX

当前行为使用户困惑：
- Tab 确认（意外）

新行为：
- **Space** 切换选择
- **Enter** 确认

### 4. 引入 Profiles

Profile 定义要安装哪些工作流：

- **core**（默认）：`propose`、`explore`、`apply`、`archive`（4 个工作流）
- **custom**：用户通过 `openspec config profile` 选择的子集

`propose` 工作流是新的 - 它将 `new` + `ff` 合并为一个创建变更并生成所有产物的命令。

### 5. 改进的 Propose UX

`/opsx:propose` 应该通过解释它正在做什么来自然地引导用户：

```text
我将创建一个带有 3 个产物的变更：
- proposal.md（什么和为什么）
- design.md（如何做）
- tasks.md（实现步骤）

准备好实现时，运行 /opsx:apply
```

这在 진행 中教学 - 大多数用户不需要单独的 onboarding。

### 6. 引入 Delivery Config

Delivery 控制工作流的安装方式：

- **both**（默认）：技能和命令
- **skills**：仅技能
- **commands**：仅命令

存储在现有全局配置（`~/.config/openspec/config.json`）中。init 期间不提示。

### 7. 新 CLI 命令

```shell
# Profile 配置（delivery + 工作流的交互式选择器）
openspec config profile          # 交互式选择器
openspec config profile core     # 预设快捷方式（core 工作流，保留 delivery）
```

交互式选择器允许用户在一个地方配置 delivery 方法和工作流选择：

```
$ openspec config profile

Delivery: [skills] [commands] [both]
                              ^^^^^^

工作流：（space 切换，enter 保存）
[x] propose
[x] explore
[x] apply
[x] archive
[ ] new
[ ] ff
[ ] continue
[ ] verify
[ ] sync
[ ] bulk-archive
[ ] onboard
```

### 8. 向后兼容和迁移

**现有用户保持当前设置。** 当在现有项目上运行 `openspec update` 且全局配置中没有 `profile` 时，执行一次性迁移：

1. 扫描项目中所有工具目录中安装的工作流文件
2. 将 `profile: "custom"`、`delivery: "both"`、`workflows: [<detected>]` 写入全局配置
3. 刷新模板但不添加或删除任何工作流
4. 显示："已迁移：带 N 个现有工作流的 custom profile"

迁移后，后续 `init` 和 `update` 命令尊重迁移的配置。

**关键行为：**
- 现有用户的工作流完全按原样保留（不会自动添加 `propose`）
- `init`（重新 init）和 `update` 在没有 profile 设置的现有项目上触发迁移
- `openspec init` 在**新**项目上（无现有工作流）使用全局配置，默认为 `core`
- 带 custom profile 的 `init` 直接应用配置的工作流（无 profile 确认提示）
- `init` 验证 `--profile` 值（`core` 或 `custom`），无效输入时报错
- 迁移消息提及 `propose` 并建议 `openspec config profile core` 以选择加入
- 迁移后，用户可以通过 `openspec config profile core` 选择加入 `core`
- 工作流模板有条件地仅在"下一步"指导中引用已安装的工作流
- 应用交付变更：切换到 `skills` 删除命令文件，切换到 `commands` 删除技能文件
- 重新运行 `init` 在现有项目上应用交付清理（删除不再匹配 delivery 的文件）
- `update` 将 profile/delivery 漂移视为需要更新，即使模板版本已是最新
- `update` 将仅命令安装视为已配置工具
- 所有工作流通过 custom profile 仍然可用

## 能力

### 新能力

- `profiles`：工作流 profiles（core、custom）、delivery 偏好、全局配置存储、交互式选择器
- `propose-workflow`：创建变更 + 生成所有产物的组合工作流

### 修改的能力

- `cli-init`：具有工具自动检测的智能默认、基于 profile 的技能/命令生成
- `cli-update`：Profile 支持、delivery 变更、现有用户的一次性迁移

## 影响

### 新文件
- `src/core/templates/workflows/propose.ts` - 新的 propose 工作流模板
- `src/core/profiles.ts` - Profile 定义和逻辑
- `src/core/available-tools.ts` - 从目录检测用户有哪些 AI 工具

### 修改的文件
- `src/core/init.ts` - 智能默认、自动检测、工具确认
- `src/core/config.ts` - 添加 profile 和 delivery 类型
- `src/core/global-config.ts` - 向 schema 添加 profile、delivery、workflows 字段
- `src/core/shared/skill-generation.ts` - 按 profile 过滤，尊重 delivery
- `src/core/shared/tool-detection.ts` - 更新 SKILL_NAMES 和 COMMAND_IDS 以包含 propose
- `src/commands/config.ts` - 添加带有交互式选择器的 `profile` 子命令
- `src/core/update.ts` - 添加 profile/delivery 支持、delivery 变更的文件删除
- `src/prompts/searchable-multi-select.ts` - 修复键绑定（space/enter）

### 全局配置 Schema 扩展
```json
// ~/.config/openspec/config.json（扩展现有）
{
  "telemetry": { ... },          // 现有
  "featureFlags": { ... },       // 现有
  "profile": "core",             // 新增：core | custom
  "delivery": "both",            // 新增：both | skills | commands
  "workflows": ["propose", ...]  // 新增：仅当 profile: custom
}
```

## Profiles 参考

| Profile | 工作流 | 描述 |
|---------|--------|------|
| core | propose, explore, apply, archive | 为大多数用户精简的流程（默认） |
| custom | 用户定义 | 通过 `openspec config profile` 精确选择你需要的内容 |