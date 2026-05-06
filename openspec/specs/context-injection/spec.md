# context-injection 规范

## 目的
定义如何将 `openspec/config.yaml` 中的项目上下文注入到工作流指令中，同时保留源文本和格式。

## 需求

### 需求：将上下文注入所有产物指令

系统应将项目配置中的 context 字段注入所有产物的指令中，用 XML 样式的 `<context>` 标签包装。

#### 场景：配置有 context 字段
- **当** 配置包含 `context: "Tech stack: TypeScript, React"` 时
- **那么** 指令输出包含 `<context>\nTech stack: TypeScript, React\n</context>`

#### 场景：配置没有 context 字段
- **当** 配置省略 context 字段或 context 未定义时
- **那么** 指令输出不包含 `<context>` 标签

#### 场景：Context 是多行字符串
- **当** 配置包含多行上下文时
- **那么** 指令输出在 `<context>` 标签内保留换行符

#### 场景：Context 应用于所有产物
- **当** 为任何产物（proposal、specs、design、tasks）加载指令时
- **那么** context 部分出现在所有指令输出中

### 需求：使用 XML 样式标签格式化上下文

系统应使用 `<context>` 开始标签和 `</context>` 结束标签包装上下文内容，内容在单独的行上。

#### 场景：Context 标签结构
- **当** 上下文被注入到指令中时
- **那么** 格式恰好是 `<context>\n{content}\n</context>\n\n`

#### 场景：Context 出现在模板之前
- **当** 用上下文生成指令时
- **那么** `<context>` 部分出现在 `<template>` 部分之前

### 需求：完全按提供的样子保留上下文内容

系统应注入上下文内容而不进行修改、转义或解释。

#### 场景：Context 包含特殊字符
- **当** 上下文包含如 `<`、`>`、`&`、引号的字符时
- **那么** 字符在配置中完全按书面样子保留

#### 场景：Context 包含 URL
- **当** 上下文包含如"docs at https://example.com"的 URL 时
- **那么** URL 在注入内容中被完全保留

#### 场景：Context 包含 Markdown
- **当** 上下文包含如 `**bold**` 或 `[links](url)` 的 Markdown 格式时
- **那么** Markdown 被保留而不渲染或转义