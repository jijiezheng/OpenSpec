# 工作区架构探索

## 上下文

在简化技能安装的过程中，我们发现了关于配置文件、配置和工作区应如何协同工作的更深层问题。本文档记录了我们的决定、开放的问题以及需要研究的内容。

**更新：** 初步探索表明，"工作区"主要不是关于配置分层——而是一个更基本的问题：**当工作跨越多个模块或仓库时，规格和变更应该位于何处？**

---

## 第一部分：Profile 和配置（原始范围）

### 我们已决定的事项

#### Profile UX（简化版）

**之前（原始提案）：**
```
openspec profile set core|extended
openspec profile install <workflow>
openspec profile uninstall <workflow>
openspec profile list
openspec profile show
openspec config set delivery skills|commands|both
openspec config get delivery
openspec config list
```
8 个子命令，两个概念（profile + config）

**之后（简化版）：**
```
openspec config profile          # 交互式选择器（delivery + workflows）
openspec config profile core     # 预设快捷方式
openspec config profile extended # 预设快捷方式
```
1 个命令带预设，一个概念

#### 交互式选择器

```
$ openspec config profile

Delivery: [skills] [commands] [both]
                              ^^^^^^

Workflows:（空格切换，回车保存）
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

一个地方配置 delivery 方法和工作流选择。

#### 为什么是"Profile"（而不是"Workflows"）

Profile 作为抽象允许未来扩展性：
- 方法论捆绑（spec-driven、test-driven）
- 用户创建的 profile
- 可共享的 profile
- 不同方法的不同技能/命令集

### 配置分层研究

我们研究了类似工具如何处理配置分层：

| 工具 | 模型 | 关键模式 |
|------|-------|-------------|
| **VSCode** | User → Workspace → Folder | 对象合并，原始类型覆盖。工作区 = 仓库中提交的 `.vscode/` |
| **ESLint（flat）** | 单个根配置 | *故意杀死级联* - "复杂性呈指数级爆炸" |
| **Turborepo** | Root + package extends | 每个包的 `turbo.json`，使用 `extends: ["//"]` 进行覆盖 |
| **Nx** | Integrated vs Package-based | 两种模式 - 共享 root 或 per-package。难以从集成模式迁移。 |
| **pnpm** | Workspace root 定义范围 | 根目录的 `pnpm-workspace.yaml`。依赖可以共享或 per-package |
| **Claude Code** | 全局 + 项目 | `~/.claude/` 用于全局，每个项目 `.claude/`。没有工作区跟踪。 |
| **Kiro** | 每个 root 分布式 | 每个文件夹有 `.kiro/`。聚合显示，没有继承。 |

**来自 ESLint 的关键洞察：** ESLint 团队明确删除了 flat config 中的级联，因为级联是复杂性的噩梦。他们的新模型：根目录一个配置，使用 glob 模式定位子目录。

**Profile/配置的推荐：** 两层就够了。
- **全局** = 用户的默认值（`~/.config/openspec/`）
- **项目** = 仓库级配置（`.openspec/` 或提交到仓库）

不需要"工作区"层。这与 Claude Code 的模型匹配。

### 配置决策（对此变更）

保持简单：
1. 全局 profile 作为 `openspec init` 的默认值
2. `openspec init` 将当前 profile 应用到项目
3. 没有工作区跟踪（尚无）
4. 没有现有项目的自动同步

这是明确的，不会阻止未来功能。

---

## 第二部分：更深层的问题（规格和变更组织）

### 真正的问题

工作区问题不是关于配置——而是关于**当发生以下情况时规格和变更位于何处**：

1. **Monorepos**：规格或变更可能跨越多个包/应用
2. **Multi-repo**：变更可能完全跨越多个仓库
3. **跨职能工作**：影响多个团队的功能（后端、Web、iOS、Android）

### 当前 OpenSpec 架构

OpenSpec 当前假设：
- 每个仓库一个 `openspec/`，总是在根目录
- CLI 不向上遍历目录——期望你在根目录
- 变更可以触及任何规格（没有范围限定）
- 单个配置适用于所有内容
- 项目内没有"范围"或"边界"的概念

```
openspec/
├── specs/
│   ├── auth/spec.md           # 按领域组织的规格
│   ├── payments/spec.md
│   └── checkout/spec.md
├── changes/
│   └── add-oauth/
│       ├── proposal.md
│       ├── design.md
│       ├── tasks.md
│       └── specs/             # 增量规格（可以触及多个）
│           ├── auth/spec.md
│           └── checkout/spec.md
└── config.yaml
```

**这对于单项目仓库运作良好。** 但以下情况呢：
- 有 50+ 个包的大型 monorepos？
- 多仓库微服务？
- 跨多个团队的功能？

### Checkout/Payment 示例

想象一个支付系统：
- **后端计费团队**：拥有支付处理
- **Web 团队**：拥有 Web 结账 UX
- **iOS 团队**：拥有 iOS 结账 UX
- **Android 团队**：拥有 Android 结账 UX
- **跨领域**：所有客户端必须遵循的支付*契约*

**问题：**
- 共享支付契约规格位于何处？
- 平台特定的结账规格位于何处？
- 如果 iOS 规格"扩展"共享契约，如何表达？
- 当契约变更时，下游规格如何更新？
- 谁拥有什么？

### 核心张力

```
                    范围
                       │
          狭窄       │       广泛
     （团队/模块）     │    （跨领域）
                       │
     ┌─────────────────┼─────────────────┐
     │                 │                 │
     │  "我们团队的     │   "共享        │
     │   结账          │    结账          │
     │   行为"         │    契约"        │
     │                 │                 │
 ────┼─────────────────┼─────────────────┼──── 所有权
     │                 │                 │
     │  容易：          │   困难：        │
     │  一个团队，      │   多个          │
     │  一个规格        │   利益相关者   │
     │                 │                 │
     └─────────────────┴─────────────────┘
```

---

## 第三部分：其他领域如何解决此问题

### 研究中的模式

| 领域 | 共享内容 | 特定内容 | 连接方式 |
|--------|-------------|----------------|------------------|
| **Protobuf** | 根目录的 `common/` | 每个服务的 `domain/service/` | 从 common 导入 |
| **设计系统** | 设计令牌、组件名称、API | 平台实现 | "相同的属性，不同的渲染" |
| **DDD** | 共享内核 | 有界上下文 | 上下文映射定义关系 |
| **RFC** | 跨领域 RFC | 团队范围的 RFC | 不同的审查流程 |
| **OpenAPI** | 基础 schemas | 每个服务的规格 | `$ref` 到共享定义 |

### Protobuf Monorepo 模式

```
proto/
├── common/              # 共享，低变动类型
│   └── money.proto
│   └── address.proto
├── billing/             # 领域特定
│   └── service.proto
└── checkout/
    └── service.proto    # 从 common/ 导入
```

**关键洞察：** "大多数工程组织应该将 proto 文件保存在一个仓库中。心理开销保持不变，而不是随组织规模增长。"

### 设计系统模式（Booking.com、Uber）

> "组件在 iOS 和 Android 之间看起来可能非常不同，因为它们使用原生应用设计标准，但仍然共享**完全相同的代码属性**。这就是为什么属性如此强大——它是每个组件的**唯一真实来源**。"

**关键洞察：** 共享规格定义*契约*（属性、行为）。平台规格定义*实现细节*（在该平台上它的外观/工作方式）。

### DDD 有界上下文

> "一个上下文，一个团队。清晰的所有权避免误解。"

**关键洞察：** 规格应该有清晰的所有权。跨领域关注使用"共享内核"模式——明确共享的代码/规格，需要协调才能变更。

---

## 第四部分：OpenSpec 的三种模型

### 模型 A：扁平根（当前）

```
openspec/
├── specs/
│   ├── checkout-contract/    # 共享契约
│   ├── checkout-web/         # Web 特定
│   ├── checkout-ios/         # iOS 特定
│   ├── checkout-android/     # Android 特定
│   ├── billing/              # 后端
│   └── ... (根级别 50+ 规格)
└── changes/
```

**优点：**
- 简单的心理模型
- 所有规格在一个地方
- 没有嵌套复杂性

**缺点：**
- 在规模上变得笨拙（50+ 目录）
- 没有清晰的所有权信号
- 难以看出哪些规格相关
- 命名约定变得关键（`checkout-*`）

### 模型 B：嵌套规格（Domain → Platform）

```
openspec/
├── specs/
│   ├── checkout/
│   │   ├── spec.md              # 共享契约（"接口"）
│   │   ├── web/spec.md          # Web 实现规格
│   │   ├── ios/spec.md          # iOS 实现规格
│   │   └── android/spec.md      # Android 实现规格
│   └── billing/
│       └── spec.md
└── changes/
```

**优点：**
- 清晰的层次结构（共享在顶部，特定嵌套）
- 相关规格放在一起
- 视觉上更好地扩展
- 所有权可以跟随结构

**缺点：**
- 更复杂的规格引用（`checkout/web` vs `checkout`）
- 需要定义继承/扩展语义
- iOS 规格"扩展"基础规格，还是只是引用它？

**开放问题：** "扩展"是什么意思？
```yaml
# checkout/ios/spec.md
extends: ../spec.md   # 继承所有需求？
requirements:
  - System SHALL support Apple Pay  # 添加到基础？
```

### 模型 C：分布式规格（靠近代码）

```
monorepo/
├── services/
│   └── billing/
│       └── openspec/specs/billing/spec.md
├── clients/
│   ├── web/
│   │   └── openspec/specs/checkout/spec.md
│   ├── ios/
│   │   └── openspec/specs/checkout/spec.md
│   └── android/
│       └── openspec/specs/checkout/spec.md
└── openspec/           # 根级跨领域
    ├── specs/
    │   └── checkout-contract/spec.md   # 共享契约
    └── changes/        # 跨领域变更位于何处？
```

**优点：**
- 规格靠近它们描述的代码
- 团队自然拥有他们的规格
- 也适用于 multi-repo（每个仓库有自己的 `openspec/`）

**缺点：**
- 跨领域规格很尴尬（它们位于何处？）
- 跨越多个 `openspec/` 目录的变更 = ？
- 需要"工作区"概念来聚合
- 要管理多个 `openspec/` 根

### 模型 D：混合（每个项目内 Model B + 跨项目 Model C）

对每个项目使用一个 `openspec/` 根，但允许嵌套规格以实现清晰的所有权和共享契约。
对于 multi-repo 工作，使用工作区清单来协调多个项目，而不复制规范规格。

**Monorepo 形状（单个项目，嵌套规格）：**
```
repo/
└── openspec/
    ├── specs/
    │   ├── contracts/
    │   │   └── checkout/spec.md
    │   ├── billing/
    │   │   └── spec.md
    │   └── checkout/
    │       ├── web/spec.md
    │       ├── ios/spec.md
    │       └── android/spec.md
    └── changes/
        └── add-3ds/
            ├── proposal.md
            ├── design.md
            ├── tasks.md
            └── specs/
                ├── contracts/checkout/spec.md
                ├── billing/spec.md
                ├── checkout/web/spec.md
                ├── checkout/ios/spec.md
                └── checkout/android/spec.md
```

**Multi-repo 形状（多个项目 + 工作区编排）：**
```
~/work/
├── contracts/
│   └── openspec/
│       ├── specs/checkout/spec.md
│       └── changes/add-3ds-contract/
├── billing-service/
│   └── openspec/
│       ├── specs/billing/spec.md
│       └── changes/add-3ds-billing/
├── web-client/
│   └── openspec/
│       ├── specs/checkout/spec.md
│       └── changes/add-3ds-web/
├── ios-client/
│   └── openspec/
│       ├── specs/checkout/spec.md
│       └── changes/add-3ds-ios/
└── payments-workspace/
    └── .openspec-workspace/
        ├── workspace.yaml
        └── initiatives/add-3ds/links.yaml
```

`workspace.yaml` 列出项目/根。`links.yaml` 将一个跨领域计划映射到每个项目的变更。

规范规格留在拥有仓库；工作区数据只是协调元数据。

**优点：**
- 清晰的所有权边界（一个项目拥有其规格和变更）
- 共享契约可以有一个专门的拥有仓库（作为真实来源没有重复）
- 用一个心理模型适用于 monorepo 和 multi-repo
- 避免继承复杂性（关系可以开始作为显式引用）
- 从当前模型增量迁移路径

**缺点：**
- 需要新的工作区 UX 用于跨仓库协调
- 跨仓库功能工作创建要管理的多个变更 ID
- 需要所有权的契约和计划链接的约定
- 一些用户可能期望一个全局的" mega change"而不是链接的 per-project 变更
- 工具必须支持主规格和变更增量中的嵌套规格路径

---

## 第五部分：Multi-Repo 考虑

对于 multi-repo 设置，几乎被迫使用模型 C（或模型 D 的协调部分）：

```
~/work/
├── billing-service/
│   └── openspec/specs/billing/
├── web-client/
│   └── openspec/specs/checkout/
├── ios-client/
│   └── openspec/specs/checkout/
└── contracts/                    # 共享规格的专用仓库？
    └── openspec/specs/
        └── checkout-contract/
```

### Multi-Repo 的问题

1. **共享规格位于何处？**
   - 专用的"contracts"仓库？
   - 每个仓库重复（漂移风险）？
   - 一个仓库是"真实来源"，其他引用它？

2. **跨仓库变更位于何处？**
   - 在其中一个仓库中？（感觉不对——有偏的所有权）
   - 在单独的"workspace"仓库中？
   - 在 `~/.config/openspec/workspaces/my-platform/changes/`？

3. **变更如何传播？**
   - 对 `checkout-contract` 的变更影响所有客户端仓库
   - 我们需要明确的依赖跟踪吗？
   - 还是这是"out of band"（团队手动协调）？

### 对于 Multi-Repo，"工作区"可能意味着什么

如果我们添加工作区支持，它可以：

> **工作区是可以一起操作的 OpenSpec 根的集合。**

```yaml
# ~/.config/openspec/workspaces.yaml（或类似）
workspaces:
  my-platform:
    roots:
      - ~/work/billing-service
      - ~/work/web-client
      - ~/work/ios-client
      - ~/work/contracts
    shared_context: |
      All services use TypeScript.
      API contracts follow OpenAPI 3.1.
```

这将启用：
1. **跨仓库变更**：创建跨多个根跟踪增量的变更
2. **聚合规格视图**：查看跨工作区的所有规格
3. **共享上下文**：适用于所有根的上下文/规则

---

## 第六部分：关键设计问题

### 1. 规格应该是分层的（有继承）？

**选项 A：无继承，只是组织**
- 嵌套目录纯粹是组织性的
- 每个规格是独立的
- 关系是隐式的（命名）或手动文档化的

**选项 B：显式继承**
```yaml
# checkout/ios/spec.md
extends: ../spec.md
requirements:
  - System SHALL support Apple Pay  # 添加到基础
```
- 子规格继承父需求
- 可以添加、覆盖或扩展
- 更强大但更复杂

**选项 C：引用但无继承**
```yaml
# checkout/ios/spec.md
references:
  - ../spec.md  # "另见"但无继承
requirements:
  - System SHALL implement checkout per checkout-contract
  - System SHALL support Apple Pay
```
- 显式引用用于文档
- 没有自动继承
- 更简单的语义

### 2. "共享内核"位于何处？

**选项 A：根级别（模型 B）**
- `openspec/specs/checkout/spec.md` 是共享内核
- 平台规格嵌套在其下

**选项 B：专用区域**
- `openspec/specs/_shared/checkout-contract/spec.md`
- 或 `openspec/specs/_contracts/checkout/spec.md`
- 明确的"共享"命名空间

**选项 C：单独仓库（模型 C 用于 multi-repo）**
- 专用的 `contracts` 或 `specs` 仓库
- 其他仓库引用它

### 3. "工作区"vs"项目"是什么？

如果我们引入工作区：

| 概念 | 定义 |
|---------|-------------|
| **项目** | 单个 OpenSpec 根（一个 `openspec/` 目录） |
| **工作区** | 可以一起操作的项目集合 |

工作区将启用：
- 跨项目聚合规格视图
- 跨项目变更
- 跨项目共享上下文

**问题：** 我们需要显式的工作区跟踪，还是只是 ad-hoc multi-root（如 Claude Code 的 `/add-dir`）？

### 4. OpenSpec 需要理解依赖吗？

如果 `checkout-web` 依赖 `checkout-contract`：
- OpenSpec 应该知道这个关系吗？
- 对 `checkout-contract` 的变更应该警告下游规格吗？
- 还是依赖跟踪"超出范围"？

**权衡：**
- 有依赖跟踪：更强大，自动传播警告
- 没有：更简单，团队自己管理依赖

### 5. 跨领域工作如何变更？

**对于 monorepos（模型 B）：**
- 一个变更，`specs/` 中的多个增量规格
- 今天已经工作

**对于 multi-repo（模型 C）：**
- 选项 A：一个"工作区变更"引用多个仓库变更
- 选项 B：每个仓库中相互引用的单独变更
- 选项 C：变更总是在一个仓库中，引用其他仓库中的规格

---

## 第七部分："令人惊叹"会是什么样子？

基于研究，团队喜欢：

1. **一个查看的地方**（Protobuf："心理开销保持不变"）
2. **清晰的所有权**（DDD："一个上下文，一个团队"）
3. **共享契约与本地扩展**（设计系统："相同的属性，不同的渲染"）
4. **自动一致性**（设计系统："设计令牌作为基础"）
5. **低认知负荷**（不应该过多考虑组织）

### 可能的北极星

**雄心勃勃：**
> OpenSpec 自动理解您的仓库结构，检测跨领域规格，并帮助您创建流向正确位置的变更。

**更简单：**
> 您可以随意组织规格。OpenSpec 就是工作。

**实用：**
> 嵌套规格用于组织。显式依赖用于跨领域。无魔法。

---

## 第八部分：可能的推进路径

### 对于此变更（simplify-skill-installation）

现在不解决规格组织问题。保持范围：
1. Profile UX 简化
2. `openspec init` 改进
3. 还没有工作区跟踪

### 未来：规格组织变更

一个单独的变更来探索和实现：

1. **决定模型 A、B、C 或 D（混合）**
2. **决定继承语义**（或无）
3. **更新规格解析**以处理嵌套
4. **更新变更增量**以处理嵌套规格

### 未来：Multi-Repo / 工作区变更

如果需要，一个单独的变更：

1. **定义工作区概念**
2. **实现工作区跟踪**（或 ad-hoc multi-root）
3. **跨仓库变更**
4. **跨仓库共享上下文**

---

## 第九部分：规格哲学（行为优先、轻量、代理对齐）

### OpenSpec 中的规格是什么？

对于 OpenSpec，规格应被视为**边界处可验证的行为契约**：
- 用户、集成商或操作员可以观察和依赖的内容
- 可以用测试、检查或明确审查验证的内容
- 即使内部实现变更也应保持稳定的内容

### 规格中应有什么和不应有什么

**包括：**
- 可观察的行为和结果
- 接口/数据契约（输入、输出、错误条件）
- 对外重要的非功能性约束（隐私、安全、可靠性）
- 下游消费者依赖的兼容性保证

**避免：**
- 内部实现细节（类名、库选择、控制流）
- 可以变更而不影响行为的工具机制
- 逐步执行计划（属于 tasks/design）

### 保持严谨与业务比例（避免官僚主义）

使用渐进式严谨：

1. **Lite 规格（大多数变更的默认设置）**
   - 简短的行为要点、清晰的范围和验收检查
2. **完整规格（仅用于高风险或跨边界工作）**
   - API 破坏、迁移、安全/隐私或跨团队/仓库变更的更深层契约细节

这保持了日常使用的轻量，同时在失败昂贵的地方保留了清晰度。

### 人类探索 -> 代理编写的规格

OpenSpec 通常是代理从人类探索编写的。为了使其可靠：

- 人类从探索中提供意图、约束和示例
- 代理将它们转换为简洁的、行为优先的需求和场景
- 代理将实现细节保存在 design/tasks 中，而不是规格中
- 验证检查强制执行结构和可测试性

简言之：人类塑造意图；代理产生一致的、可验证的契约。

### 这个哲学应该位于何处

为了避免在探索笔记中丢失这一点，将其编入：
1. `docs/concepts.md` 用于面向人类的框架
2. `openspec/specs/openspec-conventions/spec.md` 用于规范性规格约定
3. `openspec/specs/docs-agent-instructions/spec.md` 用于代理指令编写规则

---

## 第十部分：设计决策（2026 年 4 月）

在针对真实 multi-repo 用例评估上述模型后（参见 [#725](https://github.com/Fission-AI/OpenSpec/issues/725)），我们收敛到以下设计方向。

### 核心洞察

工作区本身不是持久的东西。对于大型团队，持久的设计对象是**计划/计划**，而仓库本地的规格和变更是执行工件，由每个仓库拥有。参与功能的仓库集合通常是功能范围的，随时间变化，因此必须在工作开始前配置静态工作区清单会创建与团队实际工作方式不匹配的仪式。

### 决策：模型 D 与懒工作区

选择第四部分中的模型 D（混合），但使工作区清单**可选且懒，不是先决条件**。

- **每个仓库保留自己的 canonical `openspec/`** — 基本存储模型没有变更。
- **跨根工作可以通过协调工作区中的计划来协调** — 这是当工作不再是干净的范围时共享规划所在的地方。
- **"工作区"是链接仓库和链接变更的派生或显式协调视图** — 不是用户必须提前注册的东西。
- **仅在有人明确想要可重用的跨仓库捆绑时才保存工作区清单** — 这是 opt-in 便利，不是要求。

### 决策：计划优先规划与链接仓库本地变更

对于更大的多团队工作，仓库中心规划是错误的主要抽象。团队和仓库是同一工作的多对多方面。OpenSpec 应将**计划/计划**作为第一类规划对象，然后链接仓库本地变更。

这很重要，因为今天的变更捆绑了：

- `proposal.md`
- `design.md`
- `tasks.md`
- 增量规格
- `.openspec.yaml`

那个捆绑形状适用于仓库本地工作，但当一块工作跨越多个仓库或团队时变得尴尬。在那些情况下，单个仓库本地变更试图同时充当：

- 共享规划对象
- 仓库特定的执行工件

那些应该分开。

首选模型是：

```text
协调工作区/
  .openspec-workspace/
    workspace.yaml
    initiatives/
      add-3ds/
        initiative.yaml
        proposal.md
        design.md
        links.yaml

repo-A/
  openspec/
    changes/
      add-3ds-api/
        .openspec.yaml
        tasks.md
        specs/

repo-B/
  openspec/
    changes/
      add-3ds-web/
        .openspec.yaml
        tasks.md
        specs/
```

计划拥有共享规划层：

- 提案/意图
- 共享设计和权衡
- 参与的团队
- 受影响的仓库
- 里程碑、风险和依赖
- 链接到仓库本地变更

每个仓库本地变更拥有该仓库的执行层：

- 仓库特定任务
- 增量规格
- 本地实现状态
- 应与该仓库工作一起归档的可选本地笔记

跨仓库链接仍然重要，但它应该挂在计划和仓库本地变更上：

```yaml
# billing-service/openspec/changes/add-3ds/.openspec.yaml
schema: spec-driven
created: 2026-04-12
initiative: add-3ds
links:
  - project: github.com/fission/web-client
    change: add-3ds-checkout
  - project: github.com/fission/ios-client
    change: add-3ds-checkout
```

每个仓库仍然拥有自己的变更和自己的增量。跨仓库努力表示为一个计划加上 N 个链接的单个仓库变更。这比单个 mega-change 更好，因为：
- 共享规划有一个真实来源
- 每个仓库的变更经历自己的归档周期
- 不需要在增量规格中解析跨仓库文件路径
- 团队可以以不同的速度移动（Web 在 iOS 之前发布）

对于小的单仓库工作，仓库本地变更可能仍然"足够好"作为计划和执行捆绑。计划优先拆分一旦工作变得跨团队、跨模块、跨仓库或以其他方式协调繁重时才重要。

### 决策：稳定的项目标识符，不是路径

跨仓库链接必须使用**稳定的项目标识符**，而不是文件系统路径。

- **规范形式：** 规范化的 `host/org/repo` 元组（例如 `github.com/fission/web-client`）。
- **创作简写：** CLI 接受 `org/repo`（例如 `fission/web-client`）并从当前仓库的 remote 推断主机。
- **相对路径永远不是持久的标识符。** 它们可能仅作为缓存的本地解析结果存在。

### 决策：离线优先解析

CLI 使用离线优先链将项目标识符解析为本地路径：

1. **显式路径** 为当前运行传递（例如 CLI 标志、ad-hoc multi-root）。
2. **本地 OpenSpec 仓库注册表** — `~/.config/openspec/` 或 `~/.local/share/openspec/` 中的持久映射（参见 `src/core/global-config.ts`）。
3. **父目录扫描** — 扫描已知父目录中其 remote 匹配目标标识符的 git checkouts。
4. **未解析** — 如果未找到本地路径，则将目标保留为未解析并继续使用部分工作区。CLI 不能失败。

注册表是渐进填充的：当 CLI 通过扫描或用户提示发现克隆时，它为未来解析持久化映射。注册表还存储"已知扫描根"（例如 `~/work/`），以便扫描随着时间的推移而改进，无需前置配置。

### 决策：仅信息性引用（v1）

规格级跨仓库引用是**仅文档指针**：

```yaml
# web-client/openspec/specs/checkout/spec.md frontmatter
references:
  - project: github.com/fission/contracts-service
    spec: checkout-contract
```

- CLI **不会**因为引用的跨仓库规格缺失或未解析而使验证失败。
- CLI **确实**在规划、查看或应用变更时向人类和代理公开引用。
- 更强的保证（例如陈旧警告、跨仓库验证）是通过 `lint`、`doctor` 或功能标志添加的 opt-in 层——不是基线行为。

这避免了在用例证明之前意外将 OpenSpec 承诺给完整的依赖图系统。

### 决策：共享契约的显式拥有仓库

当规格不能映射到单个实现仓库时（例如共享 API 契约）：

- **一个仓库必须是显式拥有者。** 这可以是专用的"contracts"仓库，或者是谁是自然真实来源的仓库。
- **其他仓库通过信息性引用引用拥有者的规格**（见上文）。
- **没有默认的"纯规格仓库"模式。** 将规格所有权与代码所有权分离得太激进会使代理执行尴尬并分散责任。

### Monorepo vs. Multi-Repo 总结

| 关注点 | Monorepo | Multi-Repo |
|---------|----------|------------|
| **规格组织** | 一个 `openspec/` 内的嵌套规格（模型 B） | 每个仓库有自己的 `openspec/` |
| **跨领域规格** | 嵌套在 `contracts/` 或 `shared/` 目录下 | 专用拥有仓库，其他引用它 |
| **规划对象** | 简单工作可选计划，大型跨团队工作有用 | 计划是主要协调对象 |
| **变更** | 一个或多个仓库本地变更可以实现一个计划 | 链接的 per-repo 变更实现一个计划 |
| **关系** | 引用（v1 无继承） | 项目标识符链接，仅信息性 |
| **工作区** | 通常不需要，但可以为复杂工作托管计划 | 协调工作区托管计划；可选清单用于重用 |

### 实现路径

1. **定义计划工件** — 在协调工作区中添加共享规划的計画格式。
2. **扩展变更元数据** — 让仓库本地变更指向计划和链接的兄弟变更。
3. **扩展规格元数据** — 为跨仓库规格指针添加 `references` 字段。
4. **构建项目解析** — 实现离线优先解析链和本地注册表。
5. **构建计划和链接视图** — 解析和显示计划图以及链接的仓库本地变更的命令。
6. **支持 ad-hoc multi-root** — "为此运行添加这些目录"或"从该计划的链接派生根"。
7. **可选工作区清单** — 仅在团队展示重用模式时才添加保存的工作区。

嵌套规格（单个仓库内的模型 B）是清晰 monorepo 支持的先决条件，应该首先处理，如 #662 中所述。

---

## 总结

| 问题 | 状态 | 备注 |
|----------|--------|-------|
| Profile UX | 已决定 | `openspec config profile` 带预设 |
| 配置分层 | 已决定 | 两层：全局 + 项目（没有工作区层） |
| 规格组织 | **方向已定** | 每个仓库嵌套规格，共享契约的显式拥有仓库，用于跨仓库上下文的引用 |
| 规格哲学 | 方向已定 | 行为优先契约，渐进严谨，代理对齐创作 |
| 规格继承 | **已决定** | 仅引用，v1 无继承 |
| 计划/规划模型 | **方向已定** | 大型工作的计划优先规划，仓库本地变更作为执行工件 |
| Multi-repo 支持 | **方向已定** | 链接的 per-repo 变更在共享计划下；工作区是协调，不是 canonical 执行存储 |
| 依赖跟踪 | **已决定** | v1 超出范围；引用仅信息性 |
| 跨仓库解析 | **已决定** | 带本地注册表的离线优先解析链 |
| 共享契约 | **已决定** | 需要显式拥有仓库；没有默认纯规格仓库模式 |

### 关键洞察

"工作区"问题实际上是两个独立的问题：
1. **Config/profile 范围** → 已解决全局 + 项目（不需要工作区）
2. **规划 vs. 执行组织** → 方向已定：计划协调，仓库本地变更实现，工作区保持协调层

这些应该是带独立探索的独立变更。

---

## 参考

- [VSCode 设置优先级](https://code.visualstudio.com/docs/configure/settings)
- [ESLint Flat Config 在 Monorepos 中的讨论](https://github.com/eslint/eslint/discussions/16960)
- [Turborepo 包配置](https://turborepo.dev/docs/reference/package-configurations)
- [pnpm Workspaces](https://pnpm.io/workspaces)
- [Claude Code 设置](https://code.claude.com/docs/en/settings)
- [Kiro 多根工作区](https://kiro.dev/docs/editor/multi-root-workspaces/)
- [DDD 有界上下文](https://martinfowler.com/bliki/BoundedContext.html)
- [Protobuf Monorepo 模式](https://www.lesswrong.com/posts/xts8dC3NeTHwqYgCG/keep-your-protos-in-one-repo)
- [Booking.com 多平台设计系统](https://booking.design/how-we-built-our-multi-platform-design-system-at-booking-com-d7b895399d40)
- [InnerSource RFC 模式](https://patterns.innersourcecommons.org/p/transparent-cross-team-decision-making-using-rfcs)