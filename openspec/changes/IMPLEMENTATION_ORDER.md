# 实现顺序和依赖关系

## 必需的 implementation 序列

由于依赖关系，必须按此特定顺序实现以下变更：

### 阶段 1：Foundation
**1. add-zod-validation**（无依赖）
- 创建所有核心 schema（RequirementSchema、ScenarioSchema、SpecSchema、ChangeSchema、DeltaSchema）
- 实现 markdown 解析器工具
- 实现验证基础设施和规则
- 建立所有命令使用的验证模式
- 必须首先完成

### 阶段 2：Change 命令
**2. add-change-commands**（取决于：add-zod-validation）
- 从 zod 验证导入 ChangeSchema 和 DeltaSchema
- 重用 markdown 解析工具
- 实现带有内置验证的 change 命令
- 使用验证基础设施进行 change validate 子命令
- 在 schema 和验证存在之前无法开始

### 阶段 3：Spec 命令
**3. add-spec-commands**（取决于：add-zod-validation、add-change-commands）
- 从 zod 验证导入 RequirementSchema、ScenarioSchema、SpecSchema
- 重用 markdown 解析工具
- 实现带有内置验证的 spec 命令
- 使用验证基础设施进行 spec validate 子命令
- 在 change 命令建立的模式上构建

## 依赖图
```
add-zod-validation
    ↓
add-change-commands
    ↓
add-spec-commands
```

## 关键依赖

### 共享代码依赖
1. **Schemas**：在 add-zod-validation 中创建的所有 schema，由两个命令实现使用
2. **验证**：在 add-zod-validation 中创建的基础设施，集成到两个命令中
3. **解析器**：在 add-zod-validation 中创建的 Markdown 解析工具，由两个命令使用

### 文件依赖
- `src/core/schemas/*.schema.ts`（由 add-zod-validation 创建）→ 由两个命令导入
- `src/core/validation/validator.ts`（由 add-zod-validation 创建）→ 由两个命令使用
- `src/core/parsers/markdown-parser.ts`（由 add-zod-validation 创建）→ 由两个命令使用

## 实现注意事项

### 给开发者
1. 在进入下一阶段之前完全完成每个阶段
2. 每个阶段后运行测试以确保稳定性
3. 在整个过程中遗留的 `list` 命令保持功能

### 给 CI/CD
1. 每个变更可以独立验证
2. 每个阶段后应运行集成测试
3. 第三阶段后需要完整系统测试

### 并行工作机会
在每个阶段内，以下可以并行完成：
- **阶段 1**：Schema 设计、验证规则和解析器实现
- **阶段 2**：Change 命令功能和遗留兼容性工作
- **阶段 3**：Spec 命令功能和最终集成