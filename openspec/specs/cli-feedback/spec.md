# cli-feedback 规范

## 目的
定义 `openspec feedback` 行为，用于通过 `gh` 安全地创建 GitHub Issue，当自动化不可用时提供手动回退。

## 需求

### 需求：Feedback 命令

系统应提供 `openspec feedback` 命令，使用 `gh` CLI 在 openspec 仓库中创建 GitHub Issue。系统应使用 `execFileSync` 和参数数组来防止 shell 注入漏洞。

#### 场景：简单的反馈提交

- **当** 用户执行 `openspec feedback "Great tool!"` 时
- **那么** 系统执行 `gh issue create`，标题为 "Feedback: Great tool!"
- **并且** Issue 在 openspec 仓库中创建
- **并且** Issue 具有 `feedback` 标签
- **并且** 系统显示创建的 Issue URL

#### 场景：安全的命令执行

- **当** 通过 `gh` CLI 提交反馈时
- **那么** 系统使用带有单独参数数组的 `execFileSync`
- **并且** 用户输入不通过 shell 传递
- **并且** shell 元字符（引号、反引号、$() 等）作为字面文本处理

#### 场景：带正文的反馈

- **当** 用户执行 `openspec feedback "Title here" --body "Detailed description..."` 时
- **那么** 系统创建具有指定标题的 GitHub Issue
- **并且** Issue 正文包含详细描述
- **并且** Issue 正文包含元数据（OpenSpec 版本、平台、时间戳）

### 需求：GitHub CLI 依赖

系统应在 `gh` CLI 可用时使用它进行自动反馈提交，并在 `gh` 未安装或未认证时提供手动提交回退。系统应使用平台适当的命令来检测 `gh` CLI 的可用性。

#### 场景：缺少 gh CLI 并回退

- **当** 用户运行 `openspec feedback "message"` 时
- **并且** `gh` CLI 未安装（PATH 中找不到）
- **那么** 系统显示警告："GitHub CLI not found. Manual submission required."
- **并且** 输出带有分隔符的结构化反馈内容：
  - "--- FORMATTED FEEDBACK ---"
  - 标题行
  - 标签行
  - 带元数据的正文内容
  - "--- END FEEDBACK ---"
- **并且** 显示用于手动提交的预填充 GitHub Issue URL
- **并且** 以零代码退出（成功的回退）

#### 场景：Unix 上的跨平台 gh CLI 检测

- **当** 系统在 macOS 或 Linux 上运行（平台是 'darwin' 或 'linux'）时
- **并且** 检查 `gh` CLI 是否安装
- **那么** 系统执行 `which gh` 命令

#### 场景：Windows 上的跨平台 gh CLI 检测

- **当** 系统在 Windows 上运行（平台是 'win32'）时
- **并且** 检查 `gh` CLI 是否安装
- **那么** 系统执行 `where gh` 命令

#### 场景：未认证的 gh CLI 并回退

- **当** 用户运行 `openspec feedback "message"` 时
- **并且** `gh` CLI 已安装但未认证
- **那么** 系统显示警告："GitHub authentication required. Manual submission required."
- **并且** 输出结构化反馈内容（与缺失 gh CLI 场景相同的格式）
- **并且** 显示用于手动提交的预填充 GitHub Issue URL
- **并且** 显示认证说明："To auto-submit in the future: gh auth login"
- **并且** 以零代码退出（成功的回退）

#### 场景：已认证的 gh CLI

- **当** 用户运行 `openspec feedback "message"` 时
- **并且** `gh auth status` 返回成功（已认证）
- **那么** 系统继续进行反馈提交

### 需求：Issue 元数据

系统应在 GitHub Issue 正文中包含相关元数据。

#### 场景：标准元数据

- **当** 为反馈创建 GitHub Issue 时
- **那么** Issue 正文包括：
  - OpenSpec CLI 版本
  - 平台（darwin、linux、win32）
  - 提交时间戳
  - 分隔线："---\nSubmitted via OpenSpec CLI"

#### 场景：Windows 平台元数据

- **当** 在 Windows 上为反馈创建 GitHub Issue 时
- **那么** Issue 正文包含 "Platform: win32"
- **并且** 所有平台检测使用 Node.js `os.platform()` API

#### 场景：没有敏感元数据

- **当** 为反馈创建 GitHub Issue 时
- **那么** Issue 正文不包含：
  - 用户系统中的文件路径
  - 项目名称或目录名称
  - 环境变量
  - IP 地址

### 需求：反馈始终有效

系统应允许无论遥测设置如何都能提交反馈。

#### 场景：禁用遥测时的反馈

- **当** 用户已通过 `OPENSPEC_TELEMETRY=0` 禁用遥测时
- **并且** 用户运行 `openspec feedback "message"` 时
- **那么** 反馈仍通过 `gh` CLI 提交
- **并且** 不发送遥测事件

#### 场景：CI 环境中的反馈

- **当** 环境中设置了 `CI=true` 时
- **并且** 用户运行 `openspec feedback "message"` 时
- **那么** 反馈提交正常进行（如果 `gh` 可用且已认证）

### 需求：错误处理

系统应优雅地处理反馈提交错误。

#### 场景：gh CLI 执行失败

- **当** `gh issue create` 命令失败时
- **那么** 系统显示 `gh` CLI 的错误输出
- **并且** 以与 `gh` 相同的退出代码退出

#### 场景：网络故障

- **当** `gh` CLI 报告网络连接问题时
- **那么** 系统显示来自 `gh` 的错误消息
- **并且** 建议检查网络连接
- **并且** 以非零代码退出

### 需求：代理的反馈技能

系统应提供 `/feedback` 技能，引导代理完成收集和提交用户反馈的过程。

#### 场景：代理发起的反馈

- **当** 用户在代理对话中调用 `/feedback` 时
- **那么** 代理从对话中收集上下文
- **并且** 起草包含富内容的反馈 Issue
- **并且** 对敏感信息进行匿名化
- **并且** 将草稿展示给用户等待批准
- **并且** 在用户确认后通过 `openspec feedback` 命令提交

#### 场景：上下文富化

- **当** 代理起草反馈时
- **那么** 代理包含相关上下文，例如：
  - 正在执行什么任务
  - 什么工作得好或不好
  - 具体的摩擦点或表扬

#### 场景：匿名化

- **当** 代理起草反馈时
- **那么** 代理移除或替换：
  - 用 `<path>` 或通用描述替换文件路径
  - 用 `<redacted>` 替换 API 密钥、令牌、密钥
  - 用 `<company>` 替换公司/组织名称
  - 用 `<user>` 替换人名
  - 除非公开/相关，否则用 `<url>` 替换特定 URL

#### 场景：需要用户确认

- **当** 代理起草了反馈时
- **那么** 代理必须向用户显示完整草稿
- **并且** 在提交前请求明确批准
- **并且** 允许用户请求修改
- **并且** 仅在用户确认后提交

### 需求：Shell 补全

系统应为 feedback 命令提供 shell 补全。

#### 场景：命令补全

- **当** 用户输入 `openspec fee<TAB>` 时
- **那么** shell 完成到 `openspec feedback`

#### 场景：标志补全

- **当** 用户输入 `openspec feedback "msg" --<TAB>` 时
- **那么** shell 建议可用标志（`--body`）