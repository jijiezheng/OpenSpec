## 为什么

我们需要更快、更可靠的方式来手动验证 CLI 行为变更，如 profile/delivery 同步、迁移行为和工具检测 UX。

今天，手动审查大多是非正式的：每个开发者设置状态不同，运行不同的命令顺序，并 informal 地检查输出。这使得回归很容易被忽视，并减慢 CLI UX 工作的迭代。

80/20 解决方案是添加用于确定性非交互式流程的轻量级烟雾测试工具，以及用于交互式提示行为的简短手动检查清单。

## 什么变更

- 添加用于 OpenSpec CLI 行为的轻量级 QA 烟雾测试工具，具有隔离的每次运行沙箱状态
- 使用 `Makefile` 目标作为主要入口点：
  - `make qa`（默认本地 QA 入口点）
  - `make qa-smoke`（确定性非交互式套件）
  - `make qa-interactive`（打印/打开手动交互检查清单）
- 在脚本中实现烟雾逻辑（由 Make 目标调用），而不是在 Make 本身中
- 确保每个场景在隔离的沙箱中运行，具有临时的 `HOME`、`XDG_CONFIG_HOME`、`XDG_DATA_HOME` 和 `CODEX_HOME`
- 捕获场景工件供检查（命令输出、退出代码和之前/之后的文件系统状态）
- 为高风险行为添加聚焦的场景集：
  - init 核心输出生成
  - 非交互式检测到的工具行为
  - 当 profile 未设置时的迁移
  - 交付清理（`both -> skills`、`both -> commands`）
  - 仅命令更新检测
  - 新工具目录检测消息
  - 无效 profile 覆盖验证
- 为按键/提示 UX 验证添加简短的交互式检查清单（Space 切换、Enter 确认、检测到的预选）
- 将 CI 连接到在 Linux 上运行烟雾套件作为快速回归门

## 能力

### 新能力

- `qa-smoke-harness`：具有单一开发者入口点的确定性、沙箱化 CLI 烟雾验证

### 修改的能力

- `developer-qa-workflow`：用于 CLI 行为和迁移敏感场景的标准化本地/CI QA 流程

## 影响

- `Makefile` - 添加 `qa`、`qa-smoke` 和 `qa-interactive` 目标
- `scripts/qa-smoke.sh`（或等效物）- 实现沙箱设置、场景执行和断言
- `docs/` - 添加/更新面向贡献者的 QA 说明和交互式检查清单使用
- CI 工作流 - 添加烟雾目标执行作为轻量级回归门