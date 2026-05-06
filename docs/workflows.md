# 工作流

本指南介绍 OpenSpec 的常见工作流模式及各自适用场景。基础配置请参见[快速入门](getting-started.md)，命令参考请参见[命令](commands.md)。

## 理念：动作而非阶段

传统工作流要求你按阶段推进：规划，然后实施，然后完成。但真实工作并非如此泾渭分明。

OPSX 采用了不同的方式：

```text
传统方式（阶段锁定）：

  规划 ─────────► 实施 ─────────► 完成
      │                    │
      │   "无法回头"       │
      └────────────────────┘

OPSX（流动动作）：

  提案 ──► 规格 ──► 设计 ──► 任务 ──► 实施
```

**核心原则：**

- **动作而非阶段** - 命令是你可以执行的事情，而非你被困住的阶段
- **依赖是赋能者** - 它们展示什么是可能的，而非下一步必须做什么

> **自定义：** OPSX 工作流由定义产物序列的 schema 驱动。参见[自定义](customization.md)了解创建自定义 schema 的详情。

## 两种模式

### 默认快速路径（`core` 配置文件）

新安装默认使用 `core`，提供：
- `/opsx:propose`
- `/opsx:explore`
- `/opsx:apply`
- `/opsx:sync`
- `/opsx:archive`

典型流程：

```text
/opsx:propose ──► /opsx:apply ──► /opsx:sync ──► /opsx:archive
```

### 扩展/完整工作流（自定义选择）

如果你想要显式的脚手架和构建命令（`/opsx:new`、`/opsx:continue`、`/opsx:ff`、`/opsx:verify`、`/opsx:bulk-archive`、`/opsx:onboard`），通过以下方式启用：

```bash
openspec config profile
openspec update
```

## 工作流模式（扩展模式）

### 快速功能

当你明确知道要构建什么，只需要执行时：

```text
/opsx:new ──► /opsx:ff ──► /opsx:apply ──► /opsx:verify ──► /opsx:archive
```

**示例对话：**

```text
你: /opsx:new add-logout-button

AI:  已创建 openspec/changes/add-logout-button/
     准备创建：提案

你: /opsx:ff

AI:  正在创建所有规划产物...
     ✓ proposal.md
     ✓ specs/
     ✓ design.md
     ✓ tasks.md
     准备实施！

你: /opsx:apply

AI:  正在执行任务...
     ✓ 已将登出按钮添加到头部
     ✓ 已实现会话清除
     ✓ 已添加确认对话框
     所有任务完成！

你: /opsx:archive

AI:  ✓ 已合并规格
     ✓ 已归档变更
```

**最适用于：**中小型功能、bug 修复、直接了当的变更。

### 探索式

当需求不明确或需要先调查时：

```text
/opsx:explore ──► /opsx:new ──► /opsx:continue ──► ... ──► /opsx:apply
```

**示例对话：**

```text
你: /opsx:explore

AI:  你想探索什么？

你: 我想提高页面加载性能，但不确定瓶颈在哪里。

AI:  让我来调查一下...
     [分析打包大小、识别慢查询、审查组件渲染模式]

     发现了三个主要瓶颈：
     1. 大图未优化
     2. ProductList 中同步数据获取
     3. Context 变化导致的重新渲染

     你想先处理哪一个？

你: 我们来解决数据获取问题吧。

你: /opsx:new optimize-product-list-fetching

AI:  已创建变更。准备创建：提案

你: /opsx:continue

AI:  根据我们的探索创建提案...
     [编写提案，整合调查洞察]

     准备创建：规格、设计
     你想要哪一个？

你: /opsx:continue

AI:  正在创建规格...
```

**最适用于：** 性能优化、调试、架构决策、需求不明确的场景。

### 并行变更

同时处理多个变更：

```text
变更 A: /opsx:new ──► /opsx:ff ──► /opsx:apply（进行中）
                                          │
                                     上下文切换
                                          │
变更 B: /opsx:new ──► /opsx:ff ──────────► /opsx:apply
```

**示例对话：**

```text
你: 我正在做 add-dark-mode，但被拉去修一个 bug

你: /opsx:new fix-login-redirect

AI:  已创建 openspec/changes/fix-login-redirect/

你: /opsx:ff

AI:  ✓ 已创建规划产物

你: /opsx:apply

AI:  正在处理 fix-login-redirect...
     [实现修复]
     所有任务完成！

你: /opsx:archive

AI:  ✓ 已归档 fix-login-redirect

你: 继续做暗色模式

你: /opsx:apply add-dark-mode

AI:  恢复 add-dark-mode...
     从任务 2.3 继续：更新 Header...
```

**最适用于：** 并行工作流、紧急中断、团队协作。

当你有多个已完成的变更时，使用 `/opsx:bulk-archive`：

```text
你: /opsx:bulk-archive

AI:  发现 3 个已完成的变更：
     - add-dark-mode（所有任务完成）
     - fix-login-redirect（所有任务完成）
     - update-footer（所有任务完成）

     检查规格冲突...
     ⚠ add-dark-mode 和 update-footer 都涉及 specs/ui/

     检查代码库以解决...
     两个变更都已实施。将按时间顺序应用。

     归档全部 3 个变更？

你: 是

AI:  ✓ 已归档 add-dark-mode
     ✓ 已归档 fix-login-redirect
     ✓ 已归档 update-footer
     规格合并顺序：add-dark-mode → update-footer
```

批量归档会检测多个变更何时触及相同的规格，并通过检查实际实施内容来解决冲突。

### 完成变更

推荐的完成流程：

```text
/opsx:apply ──► /opsx:verify ──► /opsx:archive
                    │                 │
              验证                 提示同步
            实施                 如需要
```

#### Verify：检查你的工作

`/opsx:verify` 从三个维度验证实施是否符合产物：

```text
你: /opsx:verify

AI:  正在验证 add-auth...

     完整性
     ✓ tasks.md 中全部 12 个任务已检查
     ✓ 规格中的所有需求都有对应代码
     ⚠ 场景"无活动后会话超时"未测试

     正确性
     ✓ 实施符合规格意图
     ✓ 场景中的边缘情况已处理
     ✓ 错误状态符合规格定义

     一致性
     ✓ 设计决策反映在代码结构中
     ✓ 命名规范与 design.md 一致
     ⚠ 设计提到"事件驱动"但实施使用轮询

     摘要
     ─────────────────────────────
     严重问题：0
     警告：2
     可以归档：是（含警告）

     建议：
     1. 为会话超时场景添加测试
     2. 考虑重构为事件驱动（如设计，或更新 design.md
```

**verify 检查的内容：**

| 维度 | 验证内容 |
|-----------|------------------|
| 完整性 | 所有任务完成、所有需求已实施、场景已覆盖 |
| 正确性 | 实施符合规格意图、边缘情况已处理 |
| 一致性 | 设计决策反映在代码中、模式一致 |

Verify 不会阻止归档，但它会暴露你可能想先解决的问题。

#### Archive：完成变更

`/opsx:archive` 完成变更并将其移至归档：

```text
你: /opsx:archive

AI:  正在归档 add-auth...

     产物状态：
     ✓ proposal.md 存在
     ✓ specs/ 存在
     ✓ design.md 存在
     ✓ tasks.md 存在（12/12 任务完成）

     增量规格：尚未同步到主分支
     → 你想现在同步吗？

你: 是

AI:  ✓ 已同步规格到 openspec/specs/auth/spec.md
     ✓ 已移动到 openspec/changes/archive/2025-01-24-add-auth/

     变更归档成功。
```

Archive 会在规格未同步时提示。它不会因为未完成的任务而阻止，但会警告你。

## 如何选择

### `/opsx:ff` vs `/opsx:continue`

| 场景 | 使用 |
|-----------|-----|
| 需求明确、准备构建 | `/opsx:ff` |
| 探索中、想逐步检查 | `/opsx:continue` |
| 想在规格之前迭代提案 | `/opsx:continue` |
| 时间压力、需要快速推进 | `/opsx:ff` |
| 复杂变更、需要控制 | `/opsx:continue` |

**经验法则：** 如果你能提前描述完整范围，使用 `/opsx:ff`。如果你在做的过程中逐步了解，使用 `/opsx:continue`。

### 何时更新 vs 重新开始

一个常见问题：什么时候可以更新现有变更，什么时候应该开始新的？

**在以下情况下更新现有变更：**

- 意图相同、执行优化
- 范围缩小（先做 MVP，其余后续）
- 学习驱动的修正（代码库不是你想的那样）
- 基于实施发现的设计调整

**在以下情况下开始新变更：**

- 意图根本改变
- 范围爆炸到完全不同的工作
- 原始变更可以独立"完成"
- 补丁会造成更多混淆而非澄清

```text
                      ┌─────────────────────────────────────┐
                      │     这是相同的工作吗？             │
                      └──────────────┬──────────────────────┘
                                     │
                  ┌──────────────────┼──────────────────┐
                  │                  │                  │
                  ▼                  ▼                  ▼
           意图相同？           超过 50% 重叠？      原始变更
           问题相同？           范围相同？           能否独立
                  │                  │                 "完成"？
                  │                  │                  │
        ┌────────┴────────┐  ┌──────┴──────┐   ┌───────┴───────┐
        │                 │  │             │   │               │
       是               否 是            否  否              是
        │                 │  │             │   │               │
        ▼                 ▼  ▼             ▼   ▼               ▼
     更新              新建 更新          新建 更新           新建
```

**示例："添加暗色模式"**

- "还需要支持自定义主题" → 新变更（范围爆炸）
- "系统偏好检测比预期难" → 更新（意图相同）
- "先发布切换按钮，偏好设置后续" → 先更新再归档，然后新变更

## 最佳实践

### 保持变更聚焦

每个逻辑工作单元一个变更。如果你要做"添加功能 X 和重构 Y"，考虑分成两个独立的变更。

**重要性：**
- 更易于审查和理解
- 更干净的归档历史
- 可以独立发布
- 需要回滚时更简单

### 对不明确的需求使用 `/opsx:explore`

在提交变更之前，探索问题空间：

```text
你: /opsx:explore

AI:  你想探索什么？

你: 我们应该如何看待 API 速率限制？

AI:  让我调查你当前的设置和选项...
     [研究代码库、考虑模式]

     我看到三种方法：
     1. 中间件方式（简单、按端点）
     2. Redis 支持（可扩展、共享状态）
     3. 每个用户的令牌桶（公平、复杂）

     你当前的技术栈建议 #1 或 #2。你的规模是多少？
```

探索能在你创建产物之前理清思路。

### 归档前验证

使用 `/opsx:verify` 检查实施是否符合产物：

```text
你: /opsx:verify

AI:  正在验证 add-dark-mode...

     ✓ tasks.md 中所有任务已检查
     ✓ 规格中的需求有对应代码
     ✓ 设计决策反映在实施中

     准备归档！
```

在关闭变更之前捕捉不匹配。

### 清晰地命名变更

好名字使 `openspec list` 更有用：

```text
好：                          避免：
add-dark-mode                 feature-1
fix-login-redirect            update
optimize-product-query         changes
implement-2fa                  wip
```

## 命令快速参考

完整的命令详情和选项，请参见[命令](commands.md)。

| 命令 | 用途 | 使用时机 |
|---------|---------|-------------|
| `/opsx:propose` | 创建变更 + 规划产物 | 快速默认路径（`core` 配置文件） |
| `/opsx:explore` | 思考想法 | 需求不明确、调查 |
| `/opsx:new` | 启动变更脚手架 | 扩展模式、显式产物控制 |
| `/opsx:continue` | 创建下一个产物 | 扩展模式、逐步创建产物 |
| `/opsx:ff` | 创建所有规划产物 | 扩展模式、范围明确 |
| `/opsx:apply` | 实施任务 | 准备写代码 |
| `/opsx:verify` | 验证实施 | 扩展模式、归档前 |
| `/opsx:sync` | 合并增量规格 | 扩展模式、可选 |
| `/opsx:archive` | 完成变更 | 所有工作完成 |
| `/opsx:bulk-archive` | 归档多个变更 | 扩展模式、并行工作 |

## 下一步

- [命令](commands.md) - 完整的命令参考及选项
- [概念](concepts.md) - 深入了解规格、产物和 schema
- [自定义](customization.md) - 创建自定义工作流