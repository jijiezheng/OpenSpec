# 快速入门

本指南解释在你安装并初始化 OpenSpec 后它如何工作。安装说明见[主 README](../README.md#快速开始)。

## 工作原理

OpenSpec 帮助你和你的 AI 编码助手在任何代码编写之前就构建什么达成一致。

**默认快速路径（core 配置文件）：**

```text
/opsx:propose ──► /opsx:apply ──► /opsx:sync ──► /opsx:archive
```

**扩展路径（自定义工作流选择）：**

```text
/opsx:new ──► /opsx:ff 或 /opsx:continue ──► /opsx:apply ──► /opsx:verify ──► /opsx:archive
```

默认全局配置文件是 `core`，包含 `propose`、`explore`、`apply`、`sync` 和 `archive`。你可以用 `openspec config profile` 启用扩展工作流命令，然后用 `openspec update`。

## OpenSpec 创建什么

运行 `openspec init` 后，你的项目有这个结构：

```
openspec/
├── specs/              # 事实来源（你系统的行为）
│   └── <domain>/
│       └── spec.md
├── changes/            # 提议的更新（每个变更一个文件夹）
│   └── <change-name>/
│       ├── proposal.md
│       ├── design.md
│       ├── tasks.md
│       └── specs/      # 增量规格（正在变更的内容）
│           └── <domain>/
│               └── spec.md
└── config.yaml         # 项目配置（可选）
```

**两个关键目录：**

- **`specs/`** - 事实来源。这些规格描述你系统当前如何运作。按领域组织（例如 `specs/auth/`、`specs/payments/`）。

- **`changes/`** - 提议的修改。每个变更有自己的文件夹，包含所有相关产物。当变更完成时，其规格合并到主 `specs/` 目录。

## 理解产物

每个变更文件夹包含引导工作的产物：

| 产物 | 用途 |
|----------|---------|
| `proposal.md` | "为什么"和"是什么" — 捕获意图、范围和方法 |
| `specs/` | 增量规格，显示新增/修改/删除的需求 |
| `design.md` | "怎么做" — 技术方法和架构决策 |
| `tasks.md` | 带有复选框的实施检查清单 |

**产物相互构建：**

```
proposal ──► specs ──► design ──► tasks ──► implement
    ▲           ▲          ▲                    │
    └───────────┴──────────┴────────────────────┘
             在学习中更新
```

你随时可以返回并根据在实施中学到的东西精炼早期的产物。

## 增量规格如何工作

增量规格是 OpenSpec 工作的关键概念。它们显示相对于你当前规格的变更。

### 格式

增量规格使用节来指示变更类型：

```markdown
# Auth 增量

## 新增需求

### 需求：双因素认证
系统必须在登录期间要求第二个因素。

#### 场景：OTP 必填
- 给定已启用 2FA 的用户
- 当用户提交有效凭证
- 然后显示 OTP 挑战

## 修改需求

### 需求：会话超时
系统必须在 30 分钟不活动后使会话过期。
（之前：60 分钟）

#### 场景：空闲超时
- 给定一个已认证会话
- 当 30 分钟没有活动
- 然后会话无效

## 删除需求

### 需求：记住我
（已弃用，支持 2FA）
```

### 归档时会发生什么

当你归档变更时：

1. **新增**需求被附加到主规格
2. **修改**需求替换现有版本
3. **删除**需求从主规格中删除

变更文件夹移动到 `openspec/changes/archive/` 用于审计历史。

## 示例：你的第一个变更

让我们走过向应用添加暗色模式。

### 1. 开始变更（默认）

```text
你: /opsx:propose add-dark-mode

AI:  已创建 openspec/changes/add-dark-mode/
     ✓ proposal.md — 我们为什么这样做，发生了什么变化
     ✓ specs/       — 需求和场景
     ✓ design.md    — 技术方法
     ✓ tasks.md     — 实施检查清单
     准备实施！
```

如果你已启用扩展工作流配置文件，你也可以分两步做：`/opsx:new` 然后 `/opsx:ff`（或增量 `/opsx:continue`）。

### 2. 创建了什么

**proposal.md** - 捕获意图：

```markdown
# 提案：添加暗色模式

## 意图
用户请求暗色模式选项以减少夜间使用的眼睛疲劳。

## 范围
范围内：
- 在设置中添加主题切换
- 支持系统偏好检测
- 在 localStorage 中持久化偏好

## 方法
使用 CSS 自定义属性进行主题化，React context 进行状态管理。
```

**specs/ui/spec.md** - 显示新需求的增量：

```markdown
# UI 增量

## 新增需求

### 需求：主题选择
系统应允许用户在浅色和深色主题之间选择。

#### 场景：手动切换
- 给定任何页面上的用户
- 当用户点击主题切换
- 然后主题立即切换
- 并且偏好跨会话持久化

#### 场景：系统偏好
- 给定没有保存偏好的用户
- 当应用加载
- 然后使用系统的首选配色方案
```

**tasks.md** - 实施检查清单：

```markdown
# 任务

## 1. 主题基础设施
- [ ] 1.1 创建带浅色/深色状态的 ThemeContext
- [ ] 1.2 为颜色添加 CSS 自定义属性
- [ ] 1.3 实现 localStorage 持久化

## 2. UI 组件
- [ ] 2.1 创建 ThemeToggle 组件
- [ ] 2.2 在设置页面添加切换
- [ ] 2.3 更新 Header 以包含快速切换

## 3. 样式
- [ ] 3.1 定义暗色主题调色板
- [ ] 3.2 更新组件使用 CSS 变量
```

### 3. 实施

```
你: /opsx:apply

AI:  正在执行任务...
     ✓ 1.1 创建了带浅色/深色状态的 ThemeContext
     ✓ 1.2 添加了 CSS 自定义属性到 globals.css
     ✓ 1.3 实现了 localStorage 持久化
     ✓ 2.1 创建了 ThemeToggle 组件
     ...
     所有任务完成！
```

在实施期间，如果你发现设计需要调整，只需更新产物然后继续。

### 4. 归档

```
你: /opsx:archive

AI:  正在归档 add-dark-mode...
     ✓ 已合并规格到 openspec/specs/ui/spec.md
     ✓ 已移动到 openspec/changes/archive/2025-01-24-add-dark-mode/
     完成！准备下一个功能。
```

你的增量规格现在是主规格的一部分，记录你系统如何运作。

## 验证和审查

使用 CLI 检查你的变更：

```bash
# 列出活动变更
openspec list

# 查看变更详情
openspec show add-dark-mode

# 验证规格格式
openspec validate add-dark-mode

# 交互式仪表板
openspec view
```

## 下一步

- [工作流](workflows.md) - 常见模式和每个命令的使用时机
- [命令](commands.md) - 所有斜杠命令的完整参考
- [概念](concepts.md) - 深入理解规格、变更和 schema
- [自定义](customization.md) - 让 OpenSpec 按你的方式工作