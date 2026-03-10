# SVP MVP 设计

## 目标

构建一个**最小可行**的 SVP 实现，验证分层编译模型的可行性。

## 成功标准

1. ✅ 能够完整走完 L5 → L4 → L3 → L2 → L1 的流程
2. ✅ 人类可以在 L5-L3 层编辑，AI 编译到 L2-L1
3. ⏳ 修改 L3 能够正确触发重新编译（基础实现，需完善增量编译）
4. ⏳ 与至少一个 AI 编辑器（Cursor/Claude Code）集成（MCP Server 框架完成）

## 范围

### 包含

- ✅ TypeScript 项目支持
- ✅ 命令行工具链（init, status, compile, build, dev, config）
- ⏳ MCP Server 集成（基础实现）
- ⏳ 基础 VS Code 扩展（未开始）
- ✅ 五层 IR 的完整定义
- ✅ AI 编译器（支持 OpenAI API 规范，已验证 DeepSeek）

### 不包含

- 其他语言（Python、Go、Rust）
- 可视化编辑器
- 复杂的状态机/工作流引擎
- 团队协作功能
- 并行编译优化

---

## MVP 架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        用户界面层                                │
├─────────────────────────────────────────────────────────────────┤
│  CLI (svp-cli)    │    VS Code Extension    │    Cursor/Claude │
│  - init ✅        │    - 状态面板 ⏳         │    - MCP Client  │
│  - compile ✅     │    - 层级导航 ⏳         │                  │
│  - status ✅      │    - 编译按钮 ⏳         │                  │
│  - build ✅       │                         │                  │
│  - dev ✅         │                         │                  │
│  - config ✅      │                         │                  │
├─────────────────────────────────────────────────────────────────┤
│                        核心服务层                                │
├─────────────────────────────────────────────────────────────────┤
│                     MCP Server (svp-mcp)                        │
│  - Resources: svp://project/context ✅                          │
│  - Tools: svp_compile, svp_status, svp_list_blocks ✅          │
├─────────────────────────────────────────────────────────────────┤
│                      Core Engine (svp-core)                     │
│  - 编译管道管理 ✅                                               │
│  - AI Compiler Agent ✅                                          │
│  - 多 Provider 支持 ✅ (OpenAI/Anthropic/本地)                  │
│  - 文件监听 ✅                                                   │
│  - 增量编译 ⏳ (基础实现)                                         │
├─────────────────────────────────────────────────────────────────┤
│                        数据层                                    │
├─────────────────────────────────────────────────────────────────┤
│  File System                                                    │
│  - blueprint.svp.yaml (L5) ✅                                   │
│  - .svp/l4/*.yaml (L4) ✅                                       │
│  - .svp/l3/*.yaml (L3) ✅                                       │
│  - .svp/gen/blocks/*.block (L2) ✅                              │
│  - src/blocks/*.ts (L1) ✅                                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 实现进度

### ✅ 已完成

| 组件 | 功能 | 状态 |
|------|------|------|
| svp-spec | IR 规范文档 (L5-L1) | ✅ |
| svp-ir | TypeScript 类型 + Zod 验证 | ✅ |
| svp-core | 编译引擎、配置管理、状态检查 | ✅ |
| svp-core/ai | AI Compiler Agent、多 Provider 支持 | ✅ |
| svp-cli | init, status, compile, build, dev, config | ✅ |
| svp-mcp | MCP Server 基础框架 | ✅ |
| template-ts | TypeScript 项目模板 + .env 配置 | ✅ |

### ⏳ 进行中/待优化

| 功能 | 描述 | 优先级 |
|------|------|--------|
| 并行编译 | L3→L2, L2→L1 多 block 并行 | P1 |
| 增量编译 | 只编译变更的 block | P1 |
| 编译缓存 | 避免重复调用 AI | P2 |
| VS Code 扩展 | GUI 界面 | P2 |
| MCP 完整集成 | 与 Cursor/Claude 深度集成 | P2 |

---

## 组件详细设计

### 1. svp-spec (协议规范)

**状态**: ✅ 已完成 (v0.1.0)

**内容**:
- IR 规范文档 (L5-L1)
- 编译器接口规范
- MCP 集成规范
- AI 配置指南

---

### 2. svp-ir (类型定义)

**状态**: ✅ 已完成

**关键类型**:
```typescript
// L5: Blueprint
interface L5Blueprint {
  svpVersion: string;
  level: 5;
  project: Project;
  intent: Intent;
  domains: Domain[];
}

// L4: Logic Chain
interface L4LogicChain {
  level: 4;
  flows: Flow[];
  contracts: Contract[];
}

// L3: Logic Block
interface L3LogicBlock {
  level: 3;
  types: TypeDefinition[];
  blocks: Block[];
}
```

---

### 3. svp-core (核心引擎)

**状态**: ✅ 已完成

**关键类**:

```typescript
// AI 编译器 Agent
class AICompilerAgent {
  async compileL5ToL4(): Promise<AICompileResult>
  async compileL4ToL3(): Promise<AICompileResult>
  async compileL3ToL2(): Promise<AICompileResult>
  async compileL2ToL1(): Promise<AICompileResult>
}

// 配置管理
interface AICompilerConfig {
  defaultProvider: string;
  providers: Record<string, AIProviderConfig>;
  compilation: {
    l5ToL4: { provider?: string; model?: string };
    l4ToL3: { provider?: string; model?: string };
    l3ToL2: { provider?: string; model?: string };
    l2ToL1: { provider?: string; model?: string };
  };
}
```

**支持的 AI Provider**:
- ✅ OpenAI (官方)
- ✅ 第三方兼容端点 (DeepSeek, OneAPI, NewAPI, etc.)
- ✅ Anthropic Claude
- ✅ 本地模型 (Ollama)

---

### 4. svp-cli (命令行工具)

**状态**: ✅ 已完成

**命令列表**:

```bash
svp init                    # 初始化项目 ✅
svp status                  # 查看项目状态 ✅
svp compile --level 5 --ai  # AI 编译指定层级 ✅
svp build                   # 完整编译管道 ✅
svp dev                     # 开发模式（监听变化）✅
svp config                  # 显示 AI 配置 ✅
```

---

### 5. AI 编译器

**状态**: ✅ 已完成，已验证

**架构**:

```
┌─────────────────────────────────────────────────────┐
│           AI Compiler = Prompt + Agent + API         │
├─────────────────────────────────────────────────────┤
│  1. Prompt Engineering Layer                         │
│     - 标准化Prompt模板（每个层级有特定模板）           │
│     - L5→L4: 意图→架构转换Prompt                      │
│     - L4→L3: 架构→逻辑块Prompt                        │
│     - L3→L2: 逻辑→骨架Prompt                          │
│     - L2→L1: 骨架→实现Prompt                          │
├─────────────────────────────────────────────────────┤
│  2. AI API Adapter (可配置)                          │
│     - 基于OpenAI API规范                              │
│     - 支持OpenAI, Claude, 本地模型                    │
│     - 通过.env配置                                    │
├─────────────────────────────────────────────────────┤
│  3. Agent Layer (内置)                               │
│     - 构建上下文                                      │
│     - 调用API                                         │
│     - 解析YAML/代码                                   │
│     - 验证结果格式                                    │
│     - 错误重试                                        │
└─────────────────────────────────────────────────────┘
```

**已验证的第三方平台**:
- ✅ DeepSeek (api.deepseek.com)
- ⏳ OneAPI (待测试)
- ⏳ NewAPI (待测试)
- ⏳ Azure OpenAI (待测试)

---

## 测试结果

### DeepSeek API 测试 (2026-03-11)

| 测试项 | 结果 | 备注 |
|--------|------|------|
| L5 → L4 | ✅ 通过 | 生成 2 个领域流程 |
| L4 → L3 | ✅ 通过 | 生成 6 个逻辑块 |
| L3 → L2 | ✅ 通过 | 生成 5 个代码骨架 |
| L2 → L1 | ✅ 通过 | 生成 3 个实现文件 |
| 代码质量 | ✅ 良好 | 包含完整JSDoc、类型、错误处理 |

**配置示例** (DeepSeek):
```bash
OPENAI_BASE_URL=https://api.deepseek.com/v1
OPENAI_API_KEY=sk-your-key
SVP_L5_MODEL=deepseek-chat
SVP_L4_MODEL=deepseek-chat
SVP_L3_MODEL=deepseek-chat
SVP_L2_MODEL=deepseek-chat
```

---

## 已知问题与限制

### 1. 性能问题

**问题**: L3→L2 和 L2→L1 编译多个 block 时串行执行，耗时较长

**影响**: 6 个 block 约需 2-3 分钟

**解决方案**: 后续实现并行编译

### 2. Token 成本

**问题**: 每次编译都调用 AI API，无缓存机制

**影响**: 频繁编译成本较高

**解决方案**: 实现增量编译和缓存

### 3. 错误恢复

**问题**: AI 输出格式偶发不符合预期，缺乏自动重试

**影响**: 需要手动重新编译

**解决方案**: 增加输出验证和自动重试机制

---

## 下一步计划

### Phase 2: 优化 (进行中)

1. **并行编译**: 多 block 并行调用 AI
2. **增量编译**: 只编译变更的 block
3. **编译缓存**: 缓存 AI 输出，避免重复调用
4. **VS Code 扩展**: GUI 界面

### Phase 3: 扩展 (计划中)

1. **多语言支持**: Python, Go, Rust
2. **可视化编辑器**: 图形化流程设计
3. **团队协作**: 多人协作编辑
4. **CI/CD 集成**: GitHub Actions, GitLab CI

---

## 文档索引

- [设计哲学](01_Philosophy.md) - 为什么需要 SVP
- [架构设计](02_Architecture.md) - 五层架构详解
- [MVP 路线图](03_MVP.md) - 本文档
- [使用方式](04_Usage.md) - 操作手册
- [AI 配置指南](05_AI_Configuration.md) - AI Provider 配置

---

*最后更新: 2026-03-11*
*MVP 状态: 核心功能已完成，进入优化阶段*
