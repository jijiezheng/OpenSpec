# ai-tool-paths 规范

## 目的
定义用于在特定工具目录中生成 OpenSpec skills 和命令的 AI 工具路径元数据。

## 需求

### 需求：AIToolOption skillsDir 字段

`AIToolOption` 接口应包含可选的 `skillsDir` 字段用于技能生成路径配置。

#### 场景：接口包含 skillsDir 字段
- **当** 在 `AI_TOOLS` 中定义支持技能生成的工具条目时
- **那么** 它应包含指定项目本地基础目录的 `skillsDir` 字段（例如 `.claude`）

#### 场景：技能路径遵循 Agent Skills 规范
- **当** 为带有 `skillsDir: '.claude'` 的工具生成技能时
- **那么** 技能应写入 `<projectRoot>/<skillsDir>/skills/`
- **并且** 根据 Agent Skills 规范追加 `/skills` 后缀

### 需求：支持工具的路径配置

`AI_TOOLS` 数组应为支持 Agent Skills 规范的工具包含 `skillsDir`。

#### 场景：Claude Code 路径定义
- **当** 查找 `claude` 工具时
- **那么** `skillsDir` 应为 `.claude`

#### 场景：Cursor 路径定义
- **当** 查找 `cursor` 工具时
- **那么** `skillsDir` 应为 `.cursor`

#### 场景：Windsurf 路径定义
- **当** 查找 `windsurf` 工具时
- **那么** `skillsDir` 应为 `.windsurf`

#### 场景：Kimi CLI 路径定义
- **当** 查找 `kimi` 工具时
- **那么** `skillsDir` 应为 `.kimi`

#### 场景：没有 skillsDir 的工具
- **当** 工具没有定义 `skillsDir` 时
- **那么** 技能生成应报错，指示工具不支持

### 需求：跨平台路径处理

系统应在不同操作系统间正确处理路径。

#### 场景：Windows 上的路径构建
- **当** 在 Windows 上构建技能路径时
- **那么** 系统应使用 `path.join()` 进行所有路径构建
- **并且** 不应硬编码正斜杠

#### 场景：Unix 上的路径构建
- **当** 在 macOS 或 Linux 上构建技能路径时
- **那么** 系统应使用 `path.join()` 保持一致性