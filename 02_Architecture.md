# SVP 架构设计

## 五层架构总览

```
┌─────────────────────────────────────────────────────────────────┐
│  L5: Blueprint (意图层)                                          │
│  ─────────────────────                                           │
│  • 项目目标和愿景                                                 │
│  • 领域划分和边界                                                 │
│  • 功能性和非功能性约束                                           │
│  • 外部集成点                                                     │
├─────────────────────────────────────────────────────────────────┤
│  L4: Logic Chain (架构层)                                        │
│  ───────────────────────                                         │
│  • 业务流程定义                                                   │
│  • 模块间接口契约                                                 │
│  • 触发器和事件                                                   │
│  • 错误处理策略                                                   │
├─────────────────────────────────────────────────────────────────┤
│  L3: Logic Block (逻辑层)                                        │
│  ──────────────────────                                          │
│  • 函数/方法签名                                                  │
│  • 前置/后置条件（契约）                                          │
│  • 伪代码算法描述                                                 │
│  • 错误类型定义                                                   │
├─────────────────────────────────────────────────────────────────┤
│  L2: Code Block (骨架层)                                         │
│  ─────────────────────                                           │
│  • 目标语言函数骨架                                               │
│  • 类型定义                                                       │
│  • TODO 占位符                                                    │
│  • 文档注释（从契约生成）                                         │
├─────────────────────────────────────────────────────────────────┤
│  L1: Code (实现层)                                               │
│  ───────────────                                                 │
│  • 完整可运行代码                                                 │
│  • 具体算法实现                                                   │
│  • 错误处理逻辑                                                   │
│  • 工具函数集成                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## L5: Blueprint (意图层)

### 目的
回答"为什么要做这个项目"、"最终要解决什么问题"。

### 内容

```yaml
svp_version: "0.1.0"
level: 5

project:
  name: "项目名称"
  description: "一句话描述"
  version: "0.1.0"

# 核心意图
intent:
  problem: "要解决的核心问题"
  solution: "解决方案概述"
  success_criteria:
    - "可衡量的成功标准"

# 约束条件
constraints:
  functional:
    - "功能约束"
  non_functional:
    - "性能、安全、可用性约束"
  business:
    - "业务规则约束"

# 领域划分
domains:
  - name: "领域名称"
    responsibility: "职责描述"
    boundaries:
      in_scope: ["范围内"]
      out_of_scope: ["范围外"]
    dependencies: ["依赖的其他领域"]

# 外部集成
integrations:
  - name: "集成点"
    type: database | api | message_queue
    description: "描述"

# 人类配置的外部上下文
context:
  design_docs:
    - path: "docs/arch.md"
      description: "架构文档"
  environment:
    - name: "DATABASE_URL"
      description: "数据库连接串"
```

### 编译到 L4

AI 编译器（L5→L4）的任务：
1. 根据 domains 生成初始的流程框架
2. 根据 constraints 生成验证步骤
3. 根据 integrations 生成外部调用点

### 人类干预点

- 创建初始蓝图
- 调整领域边界
- 添加/修改约束
- 审阅 AI 生成的 L4

---

## L4: Logic Chain (架构层)

### 目的
描述"系统由哪些流程组成，流程如何流转"。

### 内容

```yaml
svp_version: "0.1.0"
level: 4
compiled_from: "blueprint.svp.yaml"

flows:
  - id: "流程标识"
    name: "流程名称"
    domain: "所属领域"
    trigger:
      type: http | event | schedule | manual
      config:
        # trigger 特定配置
    
    steps:
      - id: "步骤标识"
        name: "步骤名称"
        action: process | decision | call | wait
        config:
          # 根据 action 不同：
          # process: { logic_ref: "引用的 L3 block" }
          # decision: { condition: "条件", branches: [...] }
          # call: { service: "服务", endpoint: "端点" }
          # wait: { event: "事件", timeout: 秒 }
        next: "下一步ID或null"
        error_handler: "错误处理步骤ID"
    
    error_handling:
      - error: "错误类型"
        handler: retry | fallback | alert | fail
        config:
          max_attempts: 3
          backoff: exponential

contracts:
  - domain: "领域"
    provides:
      - name: "提供的接口"
        input: { ... }
        output: { ... }
        errors: ["可能错误"]
    consumes:
      - domain: "依赖领域"
        contract: "依赖的接口"

data_flows:
  - from: "flow.step"
    to: "flow.step"
    data_type: "数据类型"
    persistence:
      type: none | temporary | persistent
```

### 编译到 L3

AI 编译器（L4→L3）的任务：
1. 为每个 `process` 类型的 step 生成 L3 Block
2. 为每个 `call` 类型的 step 生成接口契约
3. 根据 `error_handling` 生成错误类型
4. 根据步骤间的数据流识别需要的类型定义

### 人类干预点

- 调整流程步骤顺序
- 修改决策分支逻辑
- 添加或删除步骤
- 定义详细的错误处理策略

---

## L3: Logic Block (逻辑层)

### 目的
描述"每个步骤具体怎么做"——这是人类能理解的最后一层，也是修改的最后一层。

### 内容

```yaml
svp_version: "0.1.0"
level: 3
compiled_from: ".svp/l4/flows.yaml"

types:
  - name: "类型名称"
    definition:
      # 可以是内联定义或引用

blocks:
  - id: "block标识"
    name: "名称"
    signature: "函数签名（伪代码）"
    
    contracts:
      preconditions:
        - "前置条件"
      postconditions:
        - "后置条件"
      invariants:
        - "不变量"
    
    input:
      - name: "参数名"
        type: "类型"
        description: "描述"
        constraints: ["约束条件"]
    
    output:
      name: "返回值名"
      type: "类型"
      description: "描述"
    
    errors:
      - type: "错误类型"
        condition: "触发条件"
        handling: "处理方式"
    
    pseudocode: |
      function name(params):
        # 人类可读的算法描述
        # 可以是类 Python 的伪代码
        # 或结构化的步骤列表
    
    implementation:
      language: typescript | python | go | rust
      max_complexity: 10
      forbidden_patterns: ["禁止的模式"]
      required_patterns: ["必须包含的模式"]
    
    dependencies:
      - block_id: "依赖的 block"
        reason: "原因"

utils:
  - id: "工具函数ID"
    name: "名称"
    signature: "签名"
    description: "描述"
```

### 编译到 L2

AI 编译器（L3→L2）的任务：
1. 将 signature 转换为目标语言的函数签名
2. 从 contracts 生成 JSDoc/TSDoc 注释
3. 将 pseudocode 中的步骤转换为 TODO 占位符
4. 生成类型定义

### 人类干预点

**这是人类修改的最后一层**。

- 完善 pseudocode 的算法细节
- 添加具体的约束条件
- 定义错误类型和处理方式
- 指定实现模式（如"使用策略模式"）

---

## L2: Code Block (骨架层)

### 目的
提供代码骨架和占位符，供审计和预览，**只读**。

### 格式

TypeScript 示例：

```typescript
// <<SVP-BLOCK: block_id>>
// AUTO-GENERATED from L3: .svp/l3/domain.yaml#/blocks/block_id
// DO NOT EDIT - 修改请编辑 L3 层
// GENERATED_AT: 2026-03-11T01:19:38Z
// COMPILER_VERSION: svp-compiler-l3-to-l2@0.1.0

import { Type1, Type2 } from '../types';
import { util1, util2 } from '../utils';

/**
 * 函数描述
 * 
 * @param param1 - 参数描述
 * @returns 返回值描述
 * 
 * @precondition 前置条件
 * @postcondition 后置条件
 */
export function functionName(param1: Type1): Type2 {
  // <<TODO: 步骤1描述>>
  // CONSTRAINT: 必须满足的约束
  // PLACEHOLDER: 实现占位
  // <<TODO>>
  
  // <<TODO: 步骤2描述>>
  // <<REF: 依赖的其他block>>
  // <<TODO>>
  
  // <<PLACEHOLDER: return_value>>
}

// <</SVP-BLOCK>>
```

### 占位符类型

- `<<TODO: description>>` - 需要实现的逻辑，附带约束
- `<<PLACEHOLDER: name>>` - 将由 L2→L1 编译器填充
- `<<REF: block_id>>` - 引用其他 block

### 编译到 L1

AI 编译器（L2→L1）的任务：
1. 读取 L3 的 pseudocode 和 constraints
2. 填充所有 TODO 占位符
3. 确保类型安全和错误处理
4. 生成符合约束的具体实现

### 人类干预点

**只读，不允许修改**。

如果发现 L2 有问题：
1. 回退到 L3 修改 pseudocode 或 constraints
2. 重新编译 L3→L2

---

## L1: Code (实现层)

### 目的
最终的可运行代码。

### 格式

纯目标语言代码，保留 SVP 标记用于追溯。

```typescript
// <<SVP-BLOCK: validate_order_request>>
// AUTO-GENERATED from L2: .svp/gen/blocks/validate_order_request.ts.block
// DO NOT EDIT - 修改请编辑 L3 层
// GENERATED_AT: 2026-03-11T02:00:00Z
// COMPILER_VERSION: svp-compiler-l2-to-l1@0.1.0

import { OrderRequest, ValidationResult, ValidationError } from '../types';
import { isValidUUID, isValidEmail } from '../utils';

/**
 * 验证订单请求
 */
export function validateOrderRequest(request: OrderRequest): ValidationResult {
  const errors: ValidationError[] = [];
  
  // 验证用户ID
  if (!isValidUUID(request.user_id)) {
    errors.push({
      field: 'user_id',
      code: 'INVALID_USER_ID',
      message: 'User ID must be a valid UUID'
    });
  }
  
  // ... 完整实现 ...
  
  return { valid: errors.length === 0, errors };
}

// <</SVP-BLOCK>>
```

### 特性

- 完全可运行的代码
- 包含所有导入和依赖
- 完整的错误处理
- 通过类型检查

### 人类干预点

**禁止直接修改**。

如果需要修改：
1. 定位到对应的 L3 block
2. 在 L3 修改 pseudocode
3. 重新编译 L3→L2→L1

---

## 编译管道

```
人类修改 L5
    ↓
触发编译
    ↓
AI 编译器 (L5→L4):
  - 分析意图
  - 生成流程框架
  - 创建初始 contracts
    ↓
生成 L4 (人类审核)
    ↓
人类修改 L4
    ↓
触发编译
    ↓
AI 编译器 (L4→L3):
  - 为每个 process step 生成 block
  - 识别类型需求
  - 生成错误类型
    ↓
生成 L3 (人类修改的最后一层)
    ↓
人类完善 L3 (pseudocode, constraints)
    ↓
触发编译
    ↓
AI 编译器 (L3→L2):
  - 生成函数签名
  - 添加契约注释
  - 创建 TODO 占位符
    ↓
生成 L2 (只读审计)
    ↓
触发编译
    ↓
AI 编译器 (L2→L1):
  - 读取 pseudocode
  - 填充 TODO
  - 确保约束满足
    ↓
生成 L1 (最终代码)
```

### 增量编译

当 L3 的某个 block 修改时：

```
修改 L3 block X
    ↓
重新编译 block X → L2
    ↓
重新编译 block X → L1
    ↓
其他 blocks 不受影响
```

---

## 版本控制策略

### 提交到 Git

```
blueprint.svp.yaml      → 提交（L5 源数据）
.svp/l4/*.yaml          → 提交（L4 源数据）
.svp/l3/*.yaml          → 提交（L3 源数据，人类修改的最后一层）
.svp/gen/blocks/*.block → 可选提交（L2 派生产物，用于审计）
src/blocks/*.ts         → 可选提交（L1 派生产物，可 CI 生成）
```

### 建议的 .gitignore

```gitignore
# 依赖
node_modules/

# 编译缓存
.svp/cache/

# 日志
*.log

# L1 可以在 CI 生成（可选）
# src/blocks/*.ts

# L2 可以在 CI 生成（可选）
# .svp/gen/blocks/*.block
```

---

## 错误处理策略

### 编译错误

当某层编译失败时：

1. **记录错误**：编译器输出详细的错误信息
2. **阻止下游**：失败层阻止下游编译
3. **人类干预**：人类在失败层修复问题
4. **重新编译**：修复后重新触发编译

### 运行时错误

L1 代码运行时的错误：

1. **追溯来源**：通过 SVP 标记定位到 L3 block
2. **修复 L3**：在 L3 修改错误处理逻辑
3. **重新编译**：生成修复后的 L1
4. **禁止热修复**：不允许直接修改 L1 "临时解决问题"

---

## 扩展性设计

### 多语言支持

L5-L3 是语言无关的，L2-L1 针对特定语言：

```
L5-L3 (语言无关)
    ↓
L2-L1 编译器选择:
  - svp-compiler-l3-to-l2-ts  → TypeScript
  - svp-compiler-l3-to-l2-py  → Python
  - svp-compiler-l3-to-l2-go  → Go
  - svp-compiler-l3-to-l2-rs  → Rust
```

### 领域扩展

不同领域可以有自己的 L5 DSL：

- **Web 服务**：OpenAPI 风格的 L5
- **数据处理**：DAG 风格的 L5
- **嵌入式**：资源约束明确的 L5

### 编译器插件

AI 编译器可以替换或增强：

- 使用不同的 LLM（GPT-4、Claude、本地模型）
- 针对特定领域的优化编译器
- 人类编写的规则驱动编译器
