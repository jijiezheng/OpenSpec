# legacy-cleanup 规范

## 目的
定义在初始化和更新工作流期间检测和清理旧版 OpenSpec 工件的行为。

## 需求

### 需求：旧版工件检测

系统应检测来自先前 init 版本的旧版 OpenSpec 工件。

#### 场景：检测旧版配置文件
- **当** 在现有项目上运行 `openspec init` 时
- **那么** 系统应检查带有 OpenSpec 标记的配置文件：
  - `CLAUDE.md`
  - `.cursorrules`
  - `.windsurfrules`
  - `.clinerules`
  - `.kilocode_rules`
  - `.github/copilot-instructions.md`
  - `.amazonq/instructions.md`
  - `CODEBUDDY.md`
  - `IFLOW.md`
  - 以及来自旧版 ToolRegistry 的所有其他工具配置文件

#### 场景：检测旧版斜杠命令目录
- **当** 在现有项目上运行 `openspec init` 时
- **那么** 系统应检查旧版斜杠命令目录：
  - `.claude/commands/openspec/`
  - `.cursor/commands/openspec/`（注意：旧格式在命令根目录使用 `openspec-*.md`）
  - `.windsurf/workflows/openspec-*.md`
  - 以及来自旧版 SlashCommandRegistry 的所有工具的等效目录

#### 场景：检测旧版 OpenSpec 结构文件
- **当** 在现有项目上运行 `openspec init` 时
- **那么** 系统应检查：
  - `openspec/AGENTS.md`
  - `openspec/project.md`（仅用于迁移消息，不删除）
  - 带有 OpenSpec 标记的根 `AGENTS.md`

### 需求：旧版清理确认

系统应在移除旧版工件之前提示确认。

#### 场景：当检测到旧版时提示清理
- **当** 检测到旧版工件时
- **那么** 系统应显示找到的内容
- **并且** 提示："检测到旧版文件。升级并清理？[Y/n]"
- **并且** 如果用户按 Enter 则默认为 Yes

#### 场景：用户确认清理
- **当** 用户响应 Y 或按 Enter 时
- **那么** 系统应移除旧版工件
- **并且** 继续进行基于技能的配置

#### 场景：用户拒绝清理
- **当** 用户响应 N 时
- **那么** 系统应中止初始化
- **并且** 显示建议手动清理或使用 `--force` 标志的消息

#### 场景：非交互模式
- **当** 使用 `--no-interactive` 运行或在 CI 环境中时
- **并且** 检测到旧版工件时
- **那么** 系统应以退出代码 1 中止
- **并且** 显示检测到的旧版工件
- **并且** 建议以交互方式运行或使用 `--force` 标志

### 需求：配置文件内容的有针对性移除

系统在从配置文件移除 OpenSpec 标记时应保留用户内容。

#### 场景：仅包含 OpenSpec 内容的配置文件
- **当** 配置文件仅包含 OpenSpec 标记块（外部空白可接受）时
- **那么** 系统应移除 OpenSpec 标记块
- **并且** 保留文件（即使为空或仅空白）
- **并且** 不删除文件（配置文件属于用户的项目根目录）

#### 场景：混合内容的配置文件
- **当** 配置文件包含 OpenSpec 标记之外的内容时
- **那么** 系统应仅移除 `<!-- OPENSPEC:START -->` 到 `<!-- OPENSPEC:END -->` 块
- **并且** 保留标记之前和之后的所有内容
- **并且** 清理任何 resulting 的双空行

#### 场景：带混合内容的根 AGENTS.md
- **当** 根 `AGENTS.md` 包含 OpenSpec 标记和其他内容时
- **那么** 系统应仅移除 OpenSpec 标记块
- **并且** 保留文件的其余部分

### 需求：旧版目录移除

系统应完全移除旧版斜杠命令目录。

#### 场景：移除旧版斜杠命令目录
- **当** 旧版斜杠命令目录存在时（例如 `.claude/commands/openspec/`）
- **那么** 系统应删除整个目录及其内容
- **并且** 不删除父目录（例如 `.claude/commands/` 保留）

#### 场景：移除旧版 AGENTS.md
- **当** `openspec/AGENTS.md` 存在时
- **那么** 系统应删除该文件
- **并且** 不删除 `openspec/` 目录本身

### 需求：project.md 迁移提示

系统应保留 project.md 并显示迁移提示，而不是删除它。

#### 场景：在升级期间 project.md 存在
- **当** 在旧版清理期间 `openspec/project.md` 存在时
- **那么** 系统不应删除该文件
- **并且** 系统应在输出中显示迁移提示：
  ```
  需要手动迁移：
    → openspec/project.md 仍然存在
      将有用内容移至 config.yaml 的 "context:" 字段，然后删除
  ```

#### 场景：project.md 迁移原理
- **鉴于** project.md 可能包含用户编写的项目文档
- **并且** config.yaml 的 context 字段提供相同目的（自动注入到工件）
- **当** 显示迁移提示时
- **那么** 用户可以手动迁移或使用 `/opsx:explore` 获取 AI 辅助

### 需求：清理报告

系统应报告清理了哪些内容。

#### 场景：显示清理摘要
- **当** 旧版清理完成时
- **那么** 系统应显示摘要部分：
  ```
  清理旧版文件：
    ✓ 从 CLAUDE.md 移除 OpenSpec 标记
    ✓ 移除 .claude/commands/openspec/（由 /opsx:* 替换）
    ✓ 移除 openspec/AGENTS.md（不再需要）
  ```
- **并且** 如果 `openspec/project.md` 存在
- **那么** 系统应显示单独的迁移部分：
  ```
  需要手动迁移：
    → openspec/project.md 仍然存在
      将有用内容移至 config.yaml 的 "context:" 字段，然后删除
  ```

#### 场景：未检测到旧版
- **当** 未找到旧版工件时
- **那么** 系统不应显示清理部分
- **并且** 直接继续进行技能配置