# 添加产物再生支持

## 问题

目前，**无法在 OPSX 工作流中再生产物**：

- `/opsx:apply` 只是读取磁盘上的内容
- `/opsx:continue` 只创建下一个产物 - 不会触碰已存在的

如果在 `tasks.md` 存在后编辑 `design.md`，唯一的选择是：
1. 手动删除 tasks.md，然后运行 `/opsx:continue`
2. 手动编辑 tasks.md

文档声称你可以"在飞行中更新产物并继续"，但没有实际支持这一点的机制。

## 提议的解决方案

两部分：

### 第 1 部分：陈旧检测
向 `/opsx:apply` 添加产物陈旧检测：

1. **跟踪修改时间**：生成产物时，记录其依赖的 mtime
2. **检测陈旧**：当 `/opsx:apply` 运行时，检查上游产物（design.md、specs）是否在 tasks.md 生成后被修改
3. **提示用户**：如果陈旧，询问："Design 在 tasks 生成后被修改。是否要使用 `/opsx:continue` 再生 tasks？"

## 用户体验

### 愿景：无缝的飞行中修正

这是我们想要启用的工作流（当前已记录但不支持）：

```
You: /opsx:apply

AI:  Working through tasks...
     ✓ Task 1.1: Created caching layer
     ✓ Task 1.2: Added cache invalidation

     Working on 1.3: Implement TTL...
     I noticed the design assumes Redis, but your project uses
     in-memory caching. Should I update the design?

You: Yes, update it to use the existing cache module.

AI:  Updated design.md to use CacheManager from src/cache/
     Updated tasks.md with revised implementation steps
     Continuing implementation...
     ✓ Task 1.3: Implemented TTL using CacheManager
     ...
```

**无需重启。** 只需更新产物并继续。

### 陈旧警告 UX

当用户手动编辑上游产物时：

```
$ /opsx:apply

⚠️  检测到上游产物的变更：
    - design.md 在 5 分钟前修改（tasks.md 生成之后）

选项：
1. 再生 tasks（推荐）
2. 继续使用当前 tasks
3. 取消

>
```

### 第 2 部分：再生能力

添加再生特定产物的方式：

```bash
# 选项 A：continue 上的标志
/opsx:continue --regenerate tasks

# 选项 B：独立命令
/opsx:regenerate tasks

# 选项 C：当检测到陈旧时的交互式提示
/opsx:apply
# "Design 变更。再生 tasks？[y/N]"
```

## 技术方法

### 选项 A：元数据文件
在变更目录中存储 `.openspec-meta.json`：
```json
{
  "tasks.md": {
    "generated_at": "2025-01-24T10:00:00Z",
    "dependencies": {
      "design.md": "2025-01-24T09:55:00Z",
      "specs/feature/spec.md": "2025-01-24T09:50:00Z"
    }
  }
}
```

### 选项 B：Frontmatter
向生成的产物添加 YAML frontmatter：
```markdown
---
generated_at: 2025-01-24T10:00:00Z
depends_on:
  - design.md@2025-01-24T09:55:00Z
---
# Tasks
...
```

### 选项 C：基于 Git
使用 git 检测上游文件是否在下游最后修改后发生变化。不需要额外元数据但需要 git。

## 非目标

- 自动再生（用户应始终选择）
- 完全阻止 apply（只是警告）
- 跟踪代码文件变更（仅产物依赖）

## 依赖

- 应在 `fix-midflight-update-docs` 之后实现，以便文档先准确
- 如果需要可以与该变更结合

## 成功标准

- 当使用陈旧产物 apply 时警告用户
- 如需再生有明确路径
- 无误报（仅在真正陈旧时警告）
- 文档声称变得实际为真