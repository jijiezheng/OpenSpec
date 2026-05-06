# global-config 规范

## 目的

本规范定义 OpenSpec 如何解析、读取和写入用户级全局配置。它管辖 `src/core/global-config.ts` 模块，该模块提供存储用户偏好、功能标志和跨项目持久设置的基础。本规范通过遵循 XDG 基本目录规范和特定平台回退来确保跨平台兼容性，并通过 schema 演变规则保证向前/向后兼容性。

## 需求

### 需求：全局配置存储
系统应将全局配置存储在 `~/.config/openspec/config.json` 中，包括带有 `anonymousId` 和 `noticeSeen` 字段的遥测状态。

#### 场景：初始配置创建
- **当** 不存在全局配置文件且即将发送第一个遥测事件时
- **那么** 系统创建 `~/.config/openspec/config.json`，包含遥测配置

#### 场景：遥测配置结构
- **当** 读取或写入遥测配置时
- **那么** 配置包含带有 `anonymousId`（字符串 UUID）和 `noticeSeen`（布尔）字段的 `telemetry` 对象

#### 场景：配置文件格式
- **当** 存储配置时
- **那么** 系统写入可被用户读取和修改的有效 JSON

#### 场景：现有配置保留
- **当** 向现有配置文件添加遥测字段时
- **那么** 系统保留所有现有配置字段

### 需求：全局配置目录路径

系统应遵循 XDG 基本目录规范和特定平台回退来解析全局配置目录路径。

#### 场景：Unix/macOS 且 XDG_CONFIG_HOME 已设置
- **当** `$XDG_CONFIG_HOME` 环境变量设置为 `/custom/config` 时
- **那么** `getGlobalConfigDir()` 返回 `/custom/config/openspec`

#### 场景：Unix/macOS 且 XDG_CONFIG_HOME 未设置
- **当** `$XDG_CONFIG_HOME` 环境变量未设置时
- **并且** 平台是 Unix 或 macOS 时
- **那么** `getGlobalConfigDir()` 返回 `~/.config/openspec`（扩展为绝对路径）

#### 场景：Windows 平台
- **当** 平台是 Windows 时
- **并且** `%APPDATA%` 设置为 `C:\Users\User\AppData\Roaming` 时
- **那么** `getGlobalConfigDir()` 返回 `C:\Users\User\AppData\Roaming\openspec`

### 需求：全局配置加载

系统应从配置目录加载全局配置，在配置文件不存在或无法解析时使用合理的默认值。

#### 场景：配置文件存在且有效
- **当** 全局配置目录中存在 `config.json` 时
- **并且** 文件包含匹配配置 schema 的有效 JSON 时
- **那么** `getGlobalConfig()` 返回解析的配置

#### 场景：配置文件不存在
- **当** 全局配置目录中不存在 `config.json` 时
- **那么** `getGlobalConfig()` 返回默认配置
- **并且** 不创建目录或文件

#### 场景：配置文件是无效 JSON
- **当** `config.json` 存在但包含无效 JSON 时
- **那么** `getGlobalConfig()` 返回默认配置
- **并且** 记录警告到 stderr

### 需求：全局配置保存

系统应将全局配置保存到配置目录，在目录不存在时创建它。

#### 场景：保存配置到新目录
- **当** 调用 `saveGlobalConfig(config)` 时
- **并且** 全局配置目录不存在时
- **那么** 创建目录
- **并且** 用提供的配置写入 `config.json`

#### 场景：保存配置到现有目录
- **当** 调用 `saveGlobalConfig(config)` 时
- **并且** 全局配置目录已存在时
- **那么** 写入 `config.json`（如果存在则覆盖）

### 需求：默认配置

系统应提供在配置文件不存在时使用的默认配置。

#### 场景：默认配置结构
- **当** 不存在配置文件时
- **那么** 默认配置包含空的 `featureFlags` 对象

### 需求：配置 Schema 演变

系统应将加载的配置与默认值合并，以确保即使加载较旧的配置文件也能使用新的配置字段。

#### 场景：配置文件缺少新字段
- **当** `config.json` 存在且为 `{ "featureFlags": {} }` 时
- **并且** 当前 schema 包含新字段 `defaultAiTool` 时
- **那么** `getGlobalConfig()` 返回 `{ featureFlags: {}, defaultAiTool: <default> }`
- **并且** 对于同时存在于两者中的字段，加载值优先于默认值

#### 场景：配置文件有额外的未知字段
- **当** `config.json` 包含当前 schema 中不存在的字段时
- **那么** 未知字段保留在返回的配置中
- **并且** 不引发错误或警告