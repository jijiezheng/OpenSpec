## 为什么

我们希望将 `spec-driven` 重命名为 `openspec-default`，以更好地反映它是标准/默认工作流。但是，直接重命名会破坏在其 `openspec/config.yaml` 中有 `schema: spec-driven` 的现有项目。添加别名支持允许两个名称可互换地工作，从而实现平滑过渡而无破坏性变更。

## 什么变更

- 在 schema 解析器中添加 schema 别名解析
- `openspec-default` 和 `spec-driven` 都将解析到相同的 schema
- 物理目录保持为 `schemas/spec-driven/`（或者可以重命名为 `schemas/openspec-default/`，并将 `spec-driven` 作为别名）
- 所有 CLI 命令和配置文件都接受任一名称
- 现有用户配置无需更改

## 能力

### 新能力

- `schema-aliases`：支持 schema 名称别名，以便多个名称可以解析到同一 schema 目录

### 修改的能力

<!-- 没有现有的 spec 级别行为在变更 - 这是纯附加的 -->

## 影响

- `src/core/artifact-graph/resolver.ts` - 添加别名解析逻辑
- `schemas/` 目录 - 可能将 `spec-driven` 重命名为 `openspec-default`
- 文档 - 更新为首选 `openspec-default`，同时注意 `spec-driven` 仍然有效
- 默认 schema 常量 - 将 `DEFAULT_SCHEMA` 更新为 `openspec-default`