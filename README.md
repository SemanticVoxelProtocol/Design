# SVP 设计文档

Semantic Voxel Protocol (SVP) 的设计文档集合。

## 文档索引

### 1. [设计哲学 (01_Philosophy.md)](01_Philosophy.md)

**内容**：
- 问题本质分析
- 核心洞察（为什么需要分层编译）
- SVP 的解决方案
- 与现有方案的关系
- 价值主张
- 设计取舍

**阅读建议**：先读此文档理解 SVP 的核心理念。

---

### 2. [架构设计 (02_Architecture.md)](02_Architecture.md)

**内容**：
- 五层架构详解（L5-L1）
- 每层的目的、内容和格式
- 编译管道流程
- 层间关系和数据流
- 版本控制策略
- 扩展性设计

**阅读建议**：理解每层的作用和如何协作。

---

### 3. [MVP 设计 (03_MVP.md)](03_MVP.md)

**内容**：
- MVP 目标和范围
- 组件详细设计
- 技术栈选择
- 项目里程碑
- 风险与缓解
- 成功指标

**阅读建议**：了解实现规划和开发路线图。

---

### 4. [使用方式 (04_Usage.md)](04_Usage.md)

**内容**：
- 快速开始指南
- 详细工作流程（L5→L1）
- 日常开发流程
- 与 Cursor/Claude Code 集成
- 最佳实践
- 故障排除

**阅读建议**：作为实际使用的操作手册。

---

### 5. [AI 配置指南 (05_AI_Configuration.md)](05_AI_Configuration.md)

**内容**：
- 支持的 AI Provider（OpenAI/DeepSeek/Claude/Ollama）
- 第三方聚合平台配置（OneAPI/NewAPI）
- 分层模型配置策略
- 故障排除
- 最佳实践

**阅读建议**：配置 AI 编译器时参考。

---

### 6. [实现总结 (06_Implementation_Summary.md)](06_Implementation_Summary.md)

**内容**：
- MVP 实现状态总览
- 架构实现对比
- 功能测试报告（含 DeepSeek 测试结果）
- 性能测试数据
- 问题与限制

**阅读建议**：了解当前实现状态和测试结果。

---

## 快速导航

| 你想了解 | 阅读文档 |
|---------|---------|
| 为什么需要 SVP？ | [01_Philosophy.md](01_Philosophy.md) |
| SVP 如何工作？ | [02_Architecture.md](02_Architecture.md) |
| 如何实现 SVP？ | [03_MVP.md](03_MVP.md) |
| 如何使用 SVP？ | [04_Usage.md](04_Usage.md) |
| 如何配置 AI？ | [05_AI_Configuration.md](05_AI_Configuration.md) |
| 实现状态如何？ | [06_Implementation_Summary.md](06_Implementation_Summary.md) |

---

## 核心概念速览

```
L5: Blueprint    (意图层)     人类编写 —— 为什么、做什么
       ↓ 编译 (AI)
L4: Logic Chain  (架构层)     人类审核 —— 流程、模块、接口
       ↓ 编译 (AI)
L3: Logic Block  (逻辑层)     人类审核 —— 伪代码、契约、约束
       ↓ 编译 (AI)
L2: Code Block   (骨架层)     只读审计 —— 函数签名、类型、占位符
       ↓ 编译 (AI)
L1: Code         (实现层)     派生产物 —— 完整可运行代码
```

**核心原则**：
1. 人类只编辑 L5-L3，AI 编译到 L2-L1
2. 下层是派生产物，上层是源数据
3. 重新编译优于直接修改
4. 单向编译，禁止反向修改

---

## 实现状态

### MVP 核心功能 ✅ 已完成

- ✅ **五层架构**: L5→L4→L3→L2→L1 完整编译流程
- ✅ **AI 编译器**: 基于 OpenAI API 规范，支持多 Provider
- ✅ **CLI 工具**: init, status, compile, build, dev, config
- ✅ **分层模型**: 每层可配置不同 AI 模型
- ✅ **多 Provider**: 已验证 DeepSeek，支持 OpenAI/Claude/Ollama

### 测试验证 ✅

- **测试日期**: 2026-03-11
- **验证 API**: DeepSeek (api.deepseek.com)
- **代码质量**: ⭐⭐⭐⭐⭐ 生产可用
- **完整编译**: ~25K tokens, 成本约 ¥0.03

### 待优化项 ⏳

- 并行编译（提升 L3→L2, L2→L1 速度）
- 增量编译 + 缓存
- VS Code 扩展 GUI

详见 [实现总结](06_Implementation_Summary.md)

---

## 相关资源

- [项目缘由](../SVP项目缘由.md) - 原始项目动机和思考过程
- [GitHub Organization](https://github.com/SemanticVoxelProtocol) - 代码仓库
- [svp-spec](../svp-spec/) - 协议规范详细定义
