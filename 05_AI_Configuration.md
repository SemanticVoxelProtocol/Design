# AI 编译器配置指南

## 概述

SVP AI 编译器支持多种 AI Provider，包括：
- **OpenAI 官方 API**
- **Anthropic Claude**
- **第三方聚合平台**（OneAPI、NewAPI 等）
- **本地模型**（Ollama、LM Studio）

## 核心概念

### 分层模型配置

SVP 为每层编译提供独立的模型配置：

```
L5 → L4: 意图→架构（最关键，用最强模型）
L4 → L3: 架构→逻辑块（重要，用强模型）
L3 → L2: 逻辑→骨架（代码能力，用代码模型）
L2 → L1: 骨架→实现（可并行，用快速模型）
```

### 配置优先级

```
特定层级配置 > 全局默认配置
SVP_L5_MODEL > OPENAI_MODEL > 内置默认值
```

## 第三方聚合平台配置

### OneAPI / NewAPI 通用配置

```bash
# 1. 设置第三方端点（OpenAI兼容格式）
OPENAI_BASE_URL=https://your-api-provider.com/v1
OPENAI_API_KEY=sk-your-key

# 2. 每层使用不同模型
SVP_L5_MODEL=gpt-4              # 或平台自定义名称
SVP_L4_MODEL=gpt-4
SVP_L3_MODEL=gpt-3.5-turbo
SVP_L2_MODEL=gpt-3.5-turbo
```

### 常见第三方平台

#### 1. OpenRouter（多模型聚合）

```bash
OPENAI_BASE_URL=https://openrouter.ai/api/v1
OPENAI_API_KEY=sk-or-v1-...

# 使用不同模型
SVP_L5_MODEL=anthropic/claude-3-opus
SVP_L4_MODEL=anthropic/claude-3-sonnet
SVP_L3_MODEL=openai/gpt-4
SVP_L2_MODEL=google/gemini-pro
```

#### 2. 自定义 OneAPI 实例

```bash
OPENAI_BASE_URL=https://oneapi.your-domain.com/v1
OPENAI_API_KEY=sk-oneapi-key

# 模型名称根据你的OneAPI配置
SVP_L5_MODEL=gpt-4-1106-preview
SVP_L4_MODEL=gpt-4
SVP_L3_MODEL=gpt-3.5-turbo-1106
SVP_L2_MODEL=gpt-3.5-turbo
```

#### 3. Azure OpenAI

```bash
OPENAI_BASE_URL=https://your-resource.openai.azure.com/openai/deployments
OPENAI_API_KEY=your-azure-key

# Azure使用部署名称
SVP_L5_MODEL=gpt-4-deployment
SVP_L4_MODEL=gpt-4-deployment
SVP_L3_MODEL=gpt-35-deployment
SVP_L2_MODEL=gpt-35-deployment
```

## 混合策略配置

### 场景 1：成本优化

```bash
# L5-L4: 官方 GPT-4（确保架构质量）
# L3-L2: 第三方 GPT-3.5（降低成本）

# 官方GPT-4配置
OPENAI_API_KEY=sk-openai-key
SVP_L5_PROVIDER=openai
SVP_L5_MODEL=gpt-4-turbo-preview
SVP_L4_PROVIDER=openai
SVP_L4_MODEL=gpt-4

# 第三方GPT-3.5配置
SVP_L3_BASE_URL=https://third-party.com/v1
SVP_L3_API_KEY=sk-third-party-key
SVP_L3_PROVIDER=openai
SVP_L3_MODEL=gpt-3.5-turbo

SVP_L2_PROVIDER=openai
SVP_L2_MODEL=gpt-3.5-turbo
```

### 场景 2：多模型组合

```bash
# 每层用不同模型，发挥各自优势

# L5: Claude Opus（最强推理）
SVP_L5_PROVIDER=anthropic
SVP_L5_MODEL=claude-3-opus-20240229
ANTHROPIC_API_KEY=sk-ant-...

# L4: GPT-4（架构设计强）
SVP_L4_PROVIDER=openai
SVP_L4_MODEL=gpt-4
OPENAI_API_KEY=sk-openai-...

# L3-L2: 本地模型（免费快速）
SVP_L3_PROVIDER=local
SVP_L3_MODEL=codellama
SVP_L2_PROVIDER=local
SVP_L2_MODEL=codellama
LOCAL_AI_URL=http://localhost:11434/v1
```

## 配置验证

使用命令验证配置：

```bash
svp config
```

输出示例：
```
  ✓ 配置有效

  Provider 配置:
    默认: openai (OpenAI)
    端点: https://api.openai.com/v1
    模型: gpt-4-turbo-preview
    API Key: 已配置

  分层编译模型配置:
    L5 → L4: gpt-4-turbo-preview @ openai
      意图→架构（最重要）
    L4 → L3: gpt-4 @ openai
      架构→逻辑
    L3 → L2: gpt-3.5-turbo @ openai
      逻辑→骨架
    L2 → L1: gpt-3.5-turbo @ openai
      骨架→实现
```

## 故障排除

### 1. API Key 无效

```
错误: API key not configured for provider "openai"
```

**解决**: 
- 检查 `.env` 文件中 `OPENAI_API_KEY` 是否设置
- 确认 API Key 格式正确（通常以 `sk-` 开头）

### 2. 模型名称错误

```
错误: The model `gpt-4-xxx` does not exist
```

**解决**:
- 确认第三方平台支持的模型名称
- 有些平台使用自定义名称，如 `gpt-4-1106-preview`

### 3. 端点无法访问

```
错误: fetch failed
```

**解决**:
- 检查网络连接
- 确认 `OPENAI_BASE_URL` 格式正确（需包含 `/v1`）
- 检查是否需要代理

### 4. 配额不足

```
错误: Rate limit exceeded
```

**解决**:
- 增加 API 配额
- 使用不同层级的不同 provider 分散负载

## 最佳实践

### 1. 分层模型选择

| 层级 | 推荐模型 | 原因 |
|------|---------|------|
| L5→L4 | GPT-4 / Claude-3-Opus | 理解意图、架构设计需要最强推理 |
| L4→L3 | GPT-4 / Claude-3-Sonnet | 流程设计需要精确 |
| L3→L2 | GPT-3.5 / GPT-4 | 伪代码转骨架，代码能力强即可 |
| L2→L1 | GPT-3.5 / 本地模型 | 填充实现，可并行且成本低 |

### 2. 成本控制策略

```bash
# 关键层级用好模型
SVP_L5_MODEL=gpt-4          # 架构正确性最重要
SVP_L4_MODEL=gpt-4

# 次要层级用便宜模型
SVP_L3_MODEL=gpt-3.5-turbo  # 便宜10倍
SVP_L2_MODEL=gpt-3.5-turbo
```

### 3. 本地模型辅助

```bash
# L5-L4: 云端（确保质量）
# L3-L1: 本地（零成本）

SVP_L5_PROVIDER=openai
SVP_L4_PROVIDER=openai

SVP_L3_PROVIDER=local
SVP_L2_PROVIDER=local
LOCAL_AI_URL=http://localhost:11434/v1
LOCAL_MODEL=codellama:34b
```

## 环境变量参考

| 变量 | 说明 | 示例 |
|------|------|------|
| `OPENAI_API_KEY` | OpenAI/兼容端点 API Key | `sk-...` |
| `OPENAI_BASE_URL` | 自定义端点 URL | `https://api.x.com/v1` |
| `OPENAI_MODEL` | 默认模型 | `gpt-4-turbo-preview` |
| `ANTHROPIC_API_KEY` | Claude API Key | `sk-ant-...` |
| `SVP_AI_PROVIDER` | 全局默认 provider | `openai`, `anthropic`, `local` |
| `SVP_L5_MODEL` | L5→L4 模型 | `gpt-4` |
| `SVP_L4_MODEL` | L4→L3 模型 | `claude-3-sonnet` |
| `SVP_L3_MODEL` | L3→L2 模型 | `gpt-3.5-turbo` |
| `SVP_L2_MODEL` | L2→L1 模型 | `codellama` |

## 高级配置

### 自定义请求参数

```bash
# 温度（创造性 vs 确定性）
# L5需要准确: 0.1-0.2
# L2可以灵活: 0.2-0.3
SVP_TEMPERATURE=0.2

# 最大token（影响生成长度）
SVP_MAX_TOKENS=4000

# 超时时间（防止卡死）
SVP_TIMEOUT=60000
```

### 多 Provider 配置

```bash
# Provider A: OpenAI
OPENAI_API_KEY=sk-openai
OPENAI_BASE_URL=https://api.openai.com/v1

# Provider B: 第三方
THIRD_PARTY_KEY=sk-third
THIRD_PARTY_URL=https://third.com/v1

# L5: 官方 GPT-4
SVP_L5_PROVIDER=openai
SVP_L5_MODEL=gpt-4

# L4: 第三方 GPT-4（可能更便宜）
SVP_L4_PROVIDER=third_party
SVP_L4_MODEL=gpt-4
```
