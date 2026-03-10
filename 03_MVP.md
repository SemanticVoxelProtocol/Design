# SVP MVP 设计

## 目标

构建一个**最小可行**的 SVP 实现，验证分层编译模型的可行性。

## 成功标准

1. 能够完整走完 L5 → L4 → L3 → L2 → L1 的流程
2. 人类可以在 L5-L3 层编辑，AI 编译到 L2-L1
3. 修改 L3 能够正确触发重新编译
4. 与至少一个 AI 编辑器（Cursor/Claude Code）集成

## 范围

### 包含

- TypeScript 项目支持
- 命令行工具链
- MCP Server 集成
- 基础 VS Code 扩展
- 五层 IR 的完整定义

### 不包含

- 其他语言（Python、Go、Rust）
- 可视化编辑器
- 自动 L5→L4 编译（MVP 阶段需要人类辅助）
- 复杂的状态机/工作流引擎
- 团队协作功能

---

## MVP 架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        用户界面层                                │
├─────────────────────────────────────────────────────────────────┤
│  CLI (svp-cli)    │    VS Code Extension    │    Cursor/Claude │
│  - init           │    - 状态面板           │    - MCP Client  │
│  - compile        │    - 层级导航           │                  │
│  - status         │    - 编译按钮           │                  │
├─────────────────────────────────────────────────────────────────┤
│                        核心服务层                                │
├─────────────────────────────────────────────────────────────────┤
│                     MCP Server (svp-mcp)                        │
│  - Resources: svp://project/context                             │
│  - Tools: svp_compile, svp_list_blocks, svp_get_block          │
├─────────────────────────────────────────────────────────────────┤
│                      Core Engine (svp-core)                     │
│  - 编译管道管理                                                  │
│  - 增量编译                                                      │
│  - 文件监听                                                      │
│  - 缓存管理                                                      │
├─────────────────────────────────────────────────────────────────┤
│                        数据层                                    │
├─────────────────────────────────────────────────────────────────┤
│  File System                                                    │
│  - blueprint.svp.yaml (L5)                                      │
│  - .svp/l4/*.yaml (L4)                                          │
│  - .svp/l3/*.yaml (L3)                                          │
│  - .svp/gen/blocks/*.block (L2)                                 │
│  - src/blocks/*.ts (L1)                                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 组件详细设计

### 1. svp-spec (协议规范)

**状态**: 已完成 (v0.1.0)

**内容**:
- IR 规范文档 (L5-L1)
- 编译器接口规范
- MCP 集成规范

**文件结构**:
```
svp-spec/
├── README.md
├── ir/
│   ├── l5-blueprint.md
│   ├── l4-logic-chain.md
│   ├── l3-logic-block.md
│   ├── l2-code-block.md
│   └── l1-code.md
├── compiler/
│   └── interface.md
└── mcp/
    └── integration.md
```

---

### 2. svp-ir (类型定义)

**状态**: 基础完成，需要完善

**职责**: TypeScript 类型定义和 Zod 验证 Schema

**关键类型**:
```typescript
// L5
interface L5Blueprint {
  svpVersion: string;
  level: 5;
  project: Project;
  intent: Intent;
  constraints: Constraints;
  domains: Domain[];
  integrations: Integration[];
  context: Context;
}

// L4
interface L4LogicChain {
  svpVersion: string;
  level: 4;
  compiledFrom: string;
  flows: Flow[];
  contracts: Contract[];
  dataFlows: DataFlow[];
}

// L3
interface L3LogicBlock {
  svpVersion: string;
  level: 3;
  compiledFrom: string;
  types: TypeDefinition[];
  blocks: Block[];
  utils: Util[];
}
```

**MVP 交付**:
- [x] 基础类型定义
- [ ] 完整的 Zod 验证 Schema
- [ ] 序列化/反序列化工具

---

### 3. svp-core (核心引擎)

**状态**: 骨架完成，需要实现

**职责**:
- 编译管道管理
- 增量编译检测
- 文件监听
- 缓存管理

**关键类**:
```typescript
class SVPEngine {
  constructor(config: SVPConfig);
  
  // 编译指定层级
  compile(options: CompileOptions): Promise<CompileResult>;
  
  // 加载 L5
  loadL5(): Promise<L5Blueprint>;
  
  // 保存输出
  saveOutput(path: string, content: string): Promise<void>;
}

class CompilePipeline {
  constructor(engine: SVPEngine);
  
  // 运行完整管道
  run(options: PipelineOptions): Promise<PipelineResult>;
  
  // 增量编译
  runDelta(changedBlocks: string[]): Promise<PipelineResult>;
}

class SVPWatcher {
  constructor(options: WatchOptions);
  start(): void;
  stop(): Promise<void>;
}
```

**MVP 交付**:
- [x] 类和方法定义
- [ ] 完整实现
- [ ] 单元测试

---

### 4. svp-cli (命令行工具)

**状态**: 骨架完成，需要实现

**命令**:

```bash
# 初始化项目
svp init
  --language <lang>    # 目标语言 (默认: typescript)

# 编译指定层级
svp compile
  --level <5|4|3|2>    # 要编译的层级
  --target <block>     # 特定 block (可选)
  --dry-run            # 试运行
  --ai                 # 使用 AI 编译器 (MVP 必需)

# 完整编译管道
svp build
  --watch              # 监听变化

# 开发模式
svp dev
  # 监听 L5-L3 变化，自动编译

# 查看状态
svp status
  # 显示各层级状态

# 启动 MCP Server
svp serve
  --port <port>        # 端口 (默认: 8080)
```

**MVP 交付**:
- [x] 命令定义
- [ ] init 实现
- [ ] compile 实现（集成 AI）
- [ ] status 实现
- [ ] build 实现
- [ ] dev 实现

---

### 5. svp-mcp (MCP Server)

**状态**: 骨架完成，需要实现

**职责**: 提供 MCP 接口给 AI 编辑器

**Resources**:
```
svp://{project}/context          → 项目上下文快照
svp://{project}/ir/5             → L5 Blueprint
svp://{project}/ir/4             → L4 Logic Chain
svp://{project}/ir/3             → L3 Logic Blocks
svp://{project}/block/{id}       → 特定 block
```

**Tools**:
```typescript
// 编译
svp_compile({
  level: 5 | 4 | 3 | 2,
  target?: string
}): CompileResult

// 列出 blocks
svp_list_blocks({
  level?: 3 | 2 | 1,
  domain?: string
}): BlockList

// 获取 block
svp_get_block({
  blockId: string,
  level?: number
}): Block

// 追溯
svp_trace({
  blockId: string
}): TraceChain
```

**MVP 交付**:
- [x] Server 框架
- [ ] Resources 实现
- [ ] Tools 实现
- [ ] 与 Cursor/Claude 集成测试

---

### 6. AI 编译器 (L5→L4, L4→L3, L3→L2, L2→L1)

**状态**: 骨架完成，核心依赖 AI

**MVP 策略**:

由于 AI 编译器是核心但复杂的组件，MVP 采用**混合策略**：

1. **L5→L4**: 半自动
   - AI 生成建议
   - 人类审核和修改
   - 不完全依赖自动化

2. **L4→L3**: AI 驱动
   - 通过 MCP 调用 AI
   - 为每个 process step 生成 block
   - 人类在 L3 修改

3. **L3→L2**: AI 驱动
   - 相对机械化的转换
   - 生成函数签名和占位符

4. **L2→L1**: AI 驱动
   - 核心实现层
   - 根据 pseudocode 生成代码

**实现方式**:
```typescript
// 伪代码示例
async function compileL4ToL3(l4: L4LogicChain): Promise<L3LogicBlock[]> {
  // 1. 构建 prompt
  const prompt = buildCompilerPrompt(l4);
  
  // 2. 调用 AI (通过 MCP 或 direct API)
  const response = await ai.complete({
    system: "You are a SVP compiler. Convert L4 Logic Chain to L3 Logic Blocks.",
    prompt,
    format: "yaml"
  });
  
  // 3. 解析和验证
  const l3 = YAML.parse(response);
  const validation = validateL3(l3);
  
  // 4. 返回结果
  return l3;
}
```

**MVP 交付**:
- [ ] Prompt 模板
- [ ] AI 调用封装
- [ ] 输出解析和验证
- [ ] 错误处理

---

### 7. svp-vscode (VS Code 扩展)

**状态**: 未开始

**MVP 功能**:

1. **状态面板**
   - 显示当前项目的 L5-L1 状态
   - 指示哪些层需要编译

2. **层级导航**
   - 快速跳转到 L5/L4/L3 文件
   - 显示 block 之间的关系

3. **编译按钮**
   - 一键编译当前层级
   - 显示编译进度和结果

4. **代码高亮**
   - 区分 L1 代码和手写代码
   - 显示 block 边界

**MVP 交付**:
- [ ] 状态面板
- [ ] 基础命令

---

## MVP 工作流程

### 场景 1: 创建新项目

```bash
# 1. 创建目录
mkdir my-project && cd my-project

# 2. 初始化 SVP
svp init --language typescript
# 生成:
#   - blueprint.svp.yaml (L5 模板)
#   - .svp/ 目录结构

# 3. 编辑 L5
vim blueprint.svp.yaml
# 填写项目意图、领域、约束

# 4. 编译到 L4 (通过 Cursor/Claude)
# 在 Cursor 中:
# "请帮我把 L5 编译到 L4"
# AI 调用 svp_compile(level: 5)
# 生成 .svp/l4/flows.yaml

# 5. 审核 L4
vim .svp/l4/flows.yaml
# 检查流程是否合理，调整步骤

# 6. 编译到 L3
svp compile --level 4 --ai
# 生成 .svp/l3/*.yaml

# 7. 完善 L3
vim .svp/l3/domain.yaml
# 完善 pseudocode 和 constraints

# 8. 编译到 L2-L1
svp build
# 生成 .svp/gen/blocks/*.block
# 生成 src/blocks/*.ts

# 9. 运行代码
npm run build
npm start
```

### 场景 2: 修改功能

```bash
# 1. 发现需要修改 validate_order_request

# 2. 定位到 L3
vim .svp/l3/order.yaml
# 修改 pseudocode

# 3. 重新编译
svp compile --level 3 --ai
# 重新生成 L2 和 L1

# 4. 验证
npm run build
npm test
```

---

## 技术栈

| 组件 | 技术 | 说明 |
|------|------|------|
| 核心 | TypeScript | 全栈 TypeScript |
| 配置 | YAML | IR 使用 YAML 格式 |
| 验证 | Zod | 运行时类型验证 |
| CLI | Commander.js | 命令行框架 |
| MCP | @modelcontextprotocol/sdk | MCP 协议实现 |
| 监听 | chokidar | 文件变化监听 |
| 哈希 | xxhashjs | 增量编译检测 |

---

## 项目结构

```
SemanticVoxelProtocol/
├── svp-spec/              # 协议规范 [独立 git]
├── svp-ir/                # 类型定义 [独立 git]
├── svp-core/              # 核心引擎 [独立 git]
├── svp-cli/               # CLI [独立 git]
├── svp-mcp/               # MCP Server [独立 git]
├── svp-vscode/            # VS Code 扩展 [独立 git]
├── template-ts/           # TypeScript 模板 [独立 git]
└── Design/                # 设计文档
    ├── 01_Philosophy.md   # 本文件
    ├── 02_Architecture.md # 架构设计
    ├── 03_MVP.md          # MVP 设计
    └── 04_Usage.md        # 使用方式
```

---

## 里程碑

### Milestone 1: 基础设施 (Week 1-2)

- [ ] 完成 svp-spec 文档
- [ ] 完成 svp-ir 类型定义
- [ ] 完成 svp-core 基础实现
- [ ] 完成 svp-cli init/status 命令

### Milestone 2: MCP 集成 (Week 2-3)

- [ ] 完成 svp-mcp 基础实现
- [ ] 与 Cursor 集成测试
- [ ] 与 Claude Code 集成测试
- [ ] 定义 AI 编译器 prompt 模板

### Milestone 3: 编译管道 (Week 3-4)

- [ ] 完成 L4→L3 AI 编译器
- [ ] 完成 L3→L2 AI 编译器
- [ ] 完成 L2→L1 AI 编译器
- [ ] 完成完整编译管道测试

### Milestone 4: 示例项目 (Week 4-5)

- [ ] 使用 template-ts 创建示例项目
- [ ] 完整走通 L5→L1 流程
- [ ] 验证修改-重新编译流程
- [ ] 文档和教程

### Milestone 5: VS Code 扩展 (Week 5-6)

- [ ] 基础状态面板
- [ ] 编译按钮
- [ ] 发布到市场

---

## 风险与缓解

| 风险 | 影响 | 缓解措施 |
|------|------|----------|
| AI 编译质量不稳定 | 高 | 保留人类审核层(L4-L3)，提供修改能力 |
| 学习曲线陡峭 | 中 | 提供模板和示例，渐进式采用 |
| 现有项目迁移困难 | 中 | MVP 专注新项目，迁移工具后续开发 |
| 性能问题（大项目） | 低 | 增量编译，只处理变更 blocks |
| 与 AI 编辑器耦合 | 低 | 基于标准 MCP 协议，支持多编辑器 |

---

## 成功指标

### 技术指标

- [ ] 完整编译管道端到端运行时间 < 30 秒
- [ ] 增量编译时间 < 5 秒
- [ ] L3→L1 编译成功率 > 90%

### 用户体验指标

- [ ] 新用户可以在 30 分钟内创建第一个 SVP 项目
- [ ] 修改-编译-验证循环 < 2 分钟
- [ ] 用户愿意在真实项目中使用

### 生态指标

- [ ] 至少一个外部贡献者
- [ ] 至少 3 个示例项目
- [ ] 社区反馈积极
