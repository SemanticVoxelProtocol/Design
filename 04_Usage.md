# SVP 使用方式

## 快速开始

### 安装

```bash
# 安装 CLI
npm install -g @semanticvoxelprotocol/cli

# 验证安装
svp --version
```

### 创建项目

```bash
# 1. 创建项目目录
mkdir my-project && cd my-project

# 2. 初始化 SVP
svp init

# 交互式配置
# ? Project name: my-project
# ? Project description: A sample project
# ? Target language: TypeScript

# 生成的文件结构:
# .
# ├── blueprint.svp.yaml       # L5: 项目蓝图
# ├── .svp/
# │   ├── l4/                  # L4: 流程定义
# │   ├── l3/                  # L3: 逻辑块
# │   └── gen/                 # L2: 生成的代码骨架
# └── src/                     # L1: 最终代码
```

---

## 工作流程

### 第一阶段：定义意图 (L5)

编辑 `blueprint.svp.yaml`：

```yaml
svp_version: "0.1.0"
level: 5

project:
  name: "User Service"
  description: "用户管理和认证服务"
  version: "1.0.0"

intent:
  problem: "需要统一管理用户注册、登录和资料"
  solution: "提供 REST API 进行用户 CRUD 操作"
  success_criteria:
    - "支持邮箱+密码注册"
    - "支持 JWT 认证"
    - "响应时间 < 100ms"

constraints:
  functional:
    - "邮箱必须唯一"
    - "密码必须加密存储"
  non_functional:
    - "使用 PostgreSQL"
    - "支持水平扩展"

domains:
  - name: "User"
    responsibility: "用户数据管理"
    boundaries:
      in_scope:
        - "用户 CRUD"
        - "密码管理"
      out_of_scope:
        - "OAuth 集成"
        - "权限管理"
    dependencies: []

integrations:
  - name: "PostgreSQL"
    type: database
    description: "用户数据存储"
```

**原则**：
- 关注"为什么"和"做什么"，而非"怎么做"
- 约束要具体、可验证
- 领域边界要清晰

---

### 第二阶段：设计流程 (L4)

```bash
# 编译 L5 → L4
svp compile --level 5 --ai
```

生成的 `.svp/l4/flows.yaml`：

```yaml
flows:
  - id: "create_user"
    name: "创建用户"
    domain: "User"
    trigger:
      type: http
      config:
        method: POST
        path: "/api/users"
    
    steps:
      - id: "validate_input"
        name: "验证输入"
        action: process
        config:
          logic_ref: "validate_create_user_input"
        next: "check_email_unique"
      
      - id: "check_email_unique"
        name: "检查邮箱唯一性"
        action: call
        config:
          service: "Database"
          endpoint: "find_user_by_email"
        next: "decide_email_unique"
      
      - id: "decide_email_unique"
        name: "判断邮箱是否唯一"
        action: decision
        config:
          condition: "user == null"
          branches:
            - when: "true"
              then: "create_user_record"
            - when: "false"
              then: "email_exists_error"
      
      - id: "create_user_record"
        name: "创建用户记录"
        action: process
        config:
          logic_ref: "persist_user"
        next: "return_response"
      
      - id: "return_response"
        name: "返回响应"
        action: process
        config:
          logic_ref: "build_create_user_response"
        next: null
      
      - id: "email_exists_error"
        name: "邮箱已存在错误"
        action: process
        config:
          logic_ref: "return_email_exists_error"
        next: null
    
    error_handling:
      - error: "Database.ConnectionError"
        handler: retry
        config:
          max_attempts: 3
```

**人类审核点**：
- 流程步骤是否完整
- 决策分支是否覆盖所有情况
- 错误处理策略是否合理
- 数据流是否正确

如需修改，直接编辑 `.svp/l4/flows.yaml`。

---

### 第三阶段：定义逻辑 (L3)

```bash
# 编译 L4 → L3
svp compile --level 4 --ai
```

生成的 `.svp/l3/user.yaml`：

```yaml
blocks:
  - id: "validate_create_user_input"
    name: "验证创建用户输入"
    signature: "validateCreateUserInput(input: CreateUserDTO): ValidationResult"
    
    contracts:
      preconditions:
        - "input 不为 null"
      postconditions:
        - "email 格式有效时返回无错误"
        - "password 长度 >= 8"
    
    input:
      - name: "input"
        type: "CreateUserDTO"
        description: "用户输入数据"
        constraints:
          - "email 必须符合邮箱格式"
          - "password 长度 >= 8"
    
    output:
      name: "result"
      type: "ValidationResult"
      description: "验证结果"
    
    errors:
      - type: "InvalidEmail"
        condition: "email 格式无效"
        handling: "返回 400 错误"
      - type: "WeakPassword"
        condition: "password 长度 < 8"
        handling: "返回 400 错误"
    
    pseudocode: |
      function validateCreateUserInput(input):
        errors = []
        
        if !isValidEmail(input.email):
          errors.append({ field: "email", code: "INVALID_EMAIL" })
        
        if input.password.length < 8:
          errors.append({ field: "password", code: "WEAK_PASSWORD" })
        
        return {
          valid: errors.length == 0,
          errors: errors
        }
    
    implementation:
      language: "typescript"
      max_complexity: 5
      required_patterns:
        - "使用 zod 进行验证"
```

**人类修改的重点层**：

1. **完善伪代码**：
   ```yaml
   pseudocode: |
     function validateCreateUserInput(input):
       errors = []
       
       # 验证邮箱格式
       if !isValidEmail(input.email):
         errors.append({
           field: "email",
           code: "INVALID_EMAIL",
           message: "Email must be a valid email address"
         })
       
       # 验证密码强度
       if input.password.length < 8:
         errors.append({
           field: "password",
           code: "TOO_SHORT",
           message: "Password must be at least 8 characters"
         })
       else if !hasMixedCase(input.password):
         errors.append({
           field: "password",
           code: "NO_MIXED_CASE",
           message: "Password must contain both uppercase and lowercase letters"
         })
       
       return {
         valid: errors.length == 0,
         errors: errors
       }
   ```

2. **明确约束**：
   ```yaml
   constraints:
     - "使用 zod 的 email() 验证邮箱"
     - "密码必须包含大小写字母和数字"
     - "返回的错误信息要用户友好"
   ```

3. **定义类型**：
   ```yaml
   types:
     - name: "CreateUserDTO"
       definition:
         email: "string"
         password: "string"
         name: "string"
   ```

---

### 第四阶段：生成代码 (L2-L1)

```bash
# 编译 L3 → L2 → L1
svp build
```

生成的代码：

**L2** (`.svp/gen/blocks/validate_create_user_input.ts.block`):
```typescript
// <<SVP-BLOCK: validate_create_user_input>>
// AUTO-GENERATED from L3: .svp/l3/user.yaml#/blocks/validate_create_user_input
// DO NOT EDIT - 修改请编辑 L3 层

import { CreateUserDTO, ValidationResult } from '../types';
import { isValidEmail, hasMixedCase } from '../utils';

/**
 * 验证创建用户输入
 * 
 * @param input - 用户输入数据
 * @returns 验证结果
 * 
 * @precondition input 不为 null
 * @postcondition email 格式有效时返回无错误
 * @postcondition password 长度 >= 8
 */
export function validateCreateUserInput(
  input: CreateUserDTO
): ValidationResult {
  // <<TODO: 验证邮箱格式>>
  // CONSTRAINT: 使用 zod 的 email() 验证
  // PLACEHOLDER: email_validation
  // <<TODO>>
  
  // <<TODO: 验证密码强度>>
  // CONSTRAINT: 密码必须包含大小写字母和数字
  // PLACEHOLDER: password_validation
  // <<TODO>>
  
  // <<PLACEHOLDER: return_result>>
}

// <</SVP-BLOCK>>
```

**L1** (`src/blocks/validate_create_user_input.ts`):
```typescript
// <<SVP-BLOCK: validate_create_user_input>>
// AUTO-GENERATED from L2: .svp/gen/blocks/validate_create_user_input.ts.block
// DO NOT EDIT - 修改请编辑 L3 层

import { CreateUserDTO, ValidationResult, ValidationError } from '../types';
import { isValidEmail, hasMixedCase } from '../utils';
import { z } from 'zod';

const createUserSchema = z.object({
  email: z.string().email('Email must be a valid email address'),
  password: z.string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[a-z]/, 'Password must contain lowercase letter')
    .regex(/[A-Z]/, 'Password must contain uppercase letter')
    .regex(/[0-9]/, 'Password must contain number'),
  name: z.string()
});

export function validateCreateUserInput(
  input: CreateUserDTO
): ValidationResult {
  const result = createUserSchema.safeParse(input);
  
  if (result.success) {
    return { valid: true, errors: [] };
  }
  
  const errors: ValidationError[] = result.error.issues.map(issue => ({
    field: issue.path.join('.'),
    code: issue.code.toUpperCase(),
    message: issue.message
  }));
  
  return { valid: false, errors };
}

// <</SVP-BLOCK>>
```

---

## 日常开发流程

### 新增功能

```bash
# 1. 查看当前状态
svp status

# 2. 在 L3 添加新 block
vim .svp/l3/feature.yaml
# 添加新的 block 定义

# 3. 编译
svp compile --level 3 --ai

# 4. 验证
npm run build
npm test
```

### 修改现有功能

```bash
# 1. 定位到需要修改的 L3 block
svp list_blocks --level 3

# 2. 查看 block 详情
svp get_block --block-id validate_create_user_input

# 3. 修改 L3
vim .svp/l3/user.yaml
# 修改对应 block 的 pseudocode

# 4. 重新编译
svp compile --level 3 --ai

# 5. 查看变更
git diff src/blocks/

# 6. 验证
npm run build
npm test
```

### 使用开发模式

```bash
# 启动开发模式（监听 L5-L3 变化）
svp dev

# 输出:
# [12:34:56] change: .svp/l3/user.yaml
#   → Recompiling L3...
#   ✓ L3 compiled
#   → Recompiling L2...
#   ✓ L2 compiled
#   → Recompiling L1...
#   ✓ L1 compiled
```

---

## 与 Cursor/Claude Code 集成

### Cursor 配置

```json
// .cursor/mcp.json
{
  "mcpServers": {
    "svp": {
      "command": "npx",
      "args": ["@semanticvoxelprotocol/mcp-server", "--project", "."]
    }
  }
}
```

### 使用示例

**用户**: "帮我实现用户注册功能"

**Cursor (通过 MCP)**:
```
1. 调用 svp_get_context 获取项目状态
2. 发现 L5 已有 User 领域
3. 调用 svp_compile(level: 4) 生成 L4 流程
4. 展示生成的流程给人类审核
```

**用户**: "流程看起来不错"

**Cursor**:
```
5. 调用 svp_compile(level: 3) 生成 L3 blocks
6. 展示生成的 blocks 给人类审核
7. 人类可以在 L3 修改 pseudocode
8. 调用 svp_compile(level: 3) 重新编译
9. 调用 svp_compile(level: 2) 生成代码骨架
10. 调用 svp_compile(level: 1) 生成最终代码
```

---

## 最佳实践

### DO (推荐)

- ✅ 在 L5 清晰定义问题域和约束
- ✅ 在 L4 仔细审核流程逻辑
- ✅ 在 L3 详细编写伪代码
- ✅ 使用明确的约束条件指导 AI
- ✅ 小步快跑，频繁编译验证
- ✅ 使用版本控制管理 L5-L3
- ✅ 在 L3 添加充分的错误处理定义

### DON'T (避免)

- ❌ 直接修改 L1 代码（会被下次编译覆盖）
- ❌ 在 L5 写实现细节
- ❌ 跳过 L4/L3 的审核直接编译到 L1
- ❌ 让 pseudocode 过于抽象
- ❌ 忽略错误处理定义
- ❌ 一次修改太多 blocks

---

## 故障排除

### 编译失败

```bash
# 查看详细错误
svp compile --level 3 --ai --verbose

# 检查 L3 语法
svp validate --level 3
```

### L1 代码不符合预期

1. **检查 L3 pseudocode 是否清晰**
2. **检查 constraints 是否明确**
3. **重新编译**: `svp compile --level 3 --ai`

### 循环依赖

```bash
# 检查 block 依赖关系
svp trace --block-id my_block
```

---

## 示例项目

查看 `template-ts` 获取完整示例：

```bash
cd template-ts

# 查看 L5
cat blueprint.svp.yaml

# 查看 L4
cat .svp/l4/flows.yaml

# 查看 L3
cat .svp/l3/*.yaml

# 编译
svp build

# 查看生成的 L1
cat src/blocks/*.ts
```

---

## 进阶主题

### 自定义编译器

```typescript
// 使用自己的 AI 编译器
const engine = new SVPEngine({
  projectPath: '.',
  targetLanguage: 'typescript',
  compilers: {
    l4ToL3: new MyCustomL4ToL3Compiler()
  }
});
```

### 增量编译配置

```yaml
# .svp/config.yaml
compile:
  incremental: true
  parallel: true
  cacheDir: ".svp/cache"
  
  l3ToL2:
    timeout: 30000
    maxTokens: 4000
    
  l2ToL1:
    timeout: 60000
    maxTokens: 8000
```

### 团队工作流

```bash
# 开发者 A: 修改 L3
git checkout -b feature/user-auth
vim .svp/l3/user.yaml
svp compile --level 3 --ai
git add .svp/l3/ src/
git commit -m "Add user auth logic"
git push

# CI: 验证编译
svp build --strict
npm test

# 代码审查：审查 L3 和生成的 L1
# 合并后：L1 在 CI 重新生成
```
