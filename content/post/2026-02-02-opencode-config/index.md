---
draft: false
title: "OpenCode + Oh-My-OpenCode 配置指南"
description: "OpenCode 及其插件 Oh-My-OpenCode 的详细配置手册，包含模型选择、智能体分配及资源策略。"
slug: "opencode-ohmyopencode-config"
date: 2026-02-02T10:00:00+08:00
image: cover.png
categories:
    - Tech
tags:
    - OpenCode
    - AI Agent
    - Config
    - Terminal
---

> 记录于 2026-01-26，更新于 2026-02-02

## 配置文件位置

| 文件 | 路径 | 用途 |
|------|------|------|
| `opencode.json` | `~/.config/opencode/opencode.json` | OpenCode 主配置 |
| `oh-my-opencode.json` | `~/.config/opencode/oh-my-opencode.json` | 插件配置 |
| `mcp.json` | `~/.config/opencode/mcp.json` | MCP 服务器配置 |
| `ui.json` | `~/.config/opencode/ui.json` | UI 外观配置 |
| 项目级配置 | `.opencode/oh-my-opencode.json` | 项目覆盖配置 |
| 模型缓存 | `~/.cache/opencode/models.json` | 远程模型列表缓存 |
| 插件安装 | `~/.local/share/opencode/` | 插件和依赖 |

---

## 配置层级关系

```
┌─────────────────────────────────────────────────────────────────┐
│                      opencode.json                              │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │  1. 定义 PROVIDER (模型访问渠道)                          │ │
│  │     - anthropic, google, claudecode-relay, lmstudio...   │ │
│  │     - baseURL, apiKey, models                            │ │
│  │                                                           │ │
│  │  2. 定义 MODELS (模型能力)                                │ │
│  │     - context/output limit, variants, modalities         │ │
│  │                                                           │ │
│  │  3. 设置主 MODEL (fallback 默认值)                        │ │
│  │     - "model": "provider/model-name"                     │ │
│  │                                                           │ │
│  │  4. 加载 PLUGINS                                          │ │
│  │     - "plugin": ["oh-my-opencode", ...]                  │ │
│  └───────────────────────────────────────────────────────────┘ │
│                         ↓ 提供模型池                           │
└─────────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────────┐
│                   oh-my-opencode.json                           │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │  1. 分配 AGENTS 使用哪个模型                              │ │
│  │     - 从 opencode.json 的模型池中选择                    │ │
│  │                                                           │ │
│  │  2. 开关 FEATURES                                         │ │
│  │     - background_tasks, lsp_tools, hooks...              │ │
│  │                                                           │ │
│  │  3. 配置 UI 显示                                          │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## Oh-My-OpenCode 智能体 (Agents)

### 核心智能体

| 智能体 | 用途 | 推荐模型 |
|--------|------|----------|
| **Sisyphus** | 主智能体，像开发团队负责人 | Opus 4.5 (最强) |
| **Prometheus** | 规划智能体，生成 .sisyphus/plans/*.md | Opus 4.5 |
| **Atlas** | 计划执行编排，协调子任务 | Sonnet 4.5 |
| **Oracle** | 高 IQ 顾问，架构设计和复杂调试 | Opus 4.5 |

### 辅助智能体

| 智能体 | 用途 | 推荐模型 |
|--------|------|----------|
| **Hephaestus** | 自主深度工作者，目标导向执行（工匠） | Opus 4.5 / GPT 5.2 Codex |
| **Librarian** | 搜索官方文档、开源实现 | Sonnet 4.5 |
| **Explore** | 快速上下文感知 grep，代码库探索 | Haiku 4.5 (快速) |
| **Sisyphus-Junior** | 子任务执行，被委派的工作 | Sonnet 4.5 |
| **Multimodal-Looker** | 图片/PDF 分析 | Gemini 3 Pro (多模态) |
| **Frontend-UI-UX** | 前端开发专家 | Gemini 3 Pro High |
| **Document-Writer** | 文档写作 | Sonnet 4.5 |
| **Metis** | 计划前置咨询 | Opus 4.5 |
| **Momus** | 工作计划审查 | Sonnet 4.5 |

> **Hephaestus 注意**: 如果未在 oh-my-opencode.json 配置，会报错 `Agent hephaestus's configured model opencode/gpt-5.2-codex is not valid`。必须显式配置。

---

## 模型选择机制

### Fallback Chain (内置默认)

Oh-My-OpenCode 为每个智能体定义了优先级链：

```javascript
sisyphus: {
  fallbackChain: [
    { providers: ["anthropic"], model: "claude-opus-4-5" },     // 第1优先
    { providers: ["openai"], model: "gpt-5.2-codex" },
    { providers: ["google"], model: "gemini-3-pro" }
  ]
}
```

**解析顺序：**
1. 检查 `oh-my-opencode.json` 中是否显式配置
2. 如果没有，检查用户可用的 provider
3. 选择 fallback chain 中第一个可用的模型

### Prometheus 特殊处理

Prometheus 不使用 fallback chain，直接使用主 model：

```javascript
resolvedModel = prometheusOverride?.model ?? defaultModel  // 无 fallback chain
```

**因此必须显式配置 Prometheus，否则会用 opencode.json 的主 model。**

---

## 推荐配置策略

### 资源分配原则

```
无限制资源 (Relay)           有限资源 (Antigravity)
├── 核心工作流                └── 仅多模态场景
├── Opus: 规划/顾问              └── Gemini 3 Pro High
├── Sonnet: 执行/搜索
└── Haiku: 快速探索
```

### 示例配置（使用 claudecode-relay + Antigravity）

**oh-my-opencode.json:**
```json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json",
  "agents": {
    "sisyphus": {
      "model": "claudecode-relay/claude-opus-4-5-20251101",
      "variant": "high"
    },
    "prometheus": {
      "model": "claudecode-relay/claude-opus-4-5-20251101",
      "variant": "max"
    },
    "hephaestus": {
      "model": "claudecode-relay/claude-opus-4-5-20251101",
      "variant": "high"
    },
    "oracle": {
      "model": "claudecode-relay/claude-opus-4-5-20251101",
      "variant": "high"
    },
    "atlas": {
      "model": "google/antigravity-gemini-3-pro",
      "variant": "high"
    },
    "librarian": {
      "model": "claudecode-relay/claude-sonnet-4-5-20250929",
      "variant": "low"
    },
    "explore": {
      "model": "claudecode-relay/claude-haiku-4-5-20251001",
      "variant": "low"
    },
    "multimodal-looker": {
      "model": "google/antigravity-gemini-3-pro",
      "variant": "high"
    }
  },
  "experimental": {
    "aggressive_truncation": true
  }
}
```

**资源分配策略：**
- **核心决策** (sisyphus, prometheus, oracle, hephaestus): Opus 4.5 — 需要最强推理能力
- **快速执行** (atlas): Gemini 3 Pro — 快速响应，成本低
- **搜索** (librarian): Sonnet 4.5 — 平衡能力与成本
- **探索** (explore): Haiku 4.5 — 纯速度，用于 grep 式搜索
- **多模态** (multimodal-looker): Gemini 3 Pro — 原生视觉支持

---

## Variant 使用

### Gemini 模型 (thinkingLevel)

```json
"antigravity-gemini-3-pro": {
  "variants": {
    "low": { "thinkingLevel": "low" },
    "high": { "thinkingLevel": "high" }
  }
}
```

### Claude 模型 (thinking budgetTokens)

```json
"claude-opus-4-5-20251101": {
  "name": "Opus 4.5",
  "limit": { "context": 200000, "output": 64000 },
  "options": {
    "thinking": { "type": "enabled", "budgetTokens": 16000 }
  },
  "variants": {
    "low": { "options": { "thinking": { "type": "enabled", "budgetTokens": 8192 } } },
    "high": { "options": { "thinking": { "type": "enabled", "budgetTokens": 16000 } } },
    "max": { "options": { "thinking": { "type": "enabled", "budgetTokens": 32768 } } }
  }
}
```

### 在 oh-my-opencode.json 使用

```json
"sisyphus": {
  "model": "claudecode-relay/claude-opus-4-5-20251101",
  "variant": "high"
},
"multimodal-looker": {
  "model": "google/antigravity-gemini-3-pro",
  "variant": "high"
}
```

---

## 常用操作

### 清除模型缓存

```bash
rm ~/.cache/opencode/models.json
# 重启 OpenCode 会重新拉取
```

### 查看插件版本

```bash
cat ~/.local/share/opencode/node_modules/oh-my-opencode/package.json | grep version
```

### 验证配置

```bash
# 检查 JSON 语法
cat ~/.config/opencode/opencode.json | jq .
cat ~/.config/opencode/oh-my-opencode.json | jq .
```

---

## 魔法词

| 关键词 | 作用 |
|--------|------|
| `ultrawork` / `ulw` | 启用所有增强功能，持续工作直到完成 |
| `/start-work` | 从 Prometheus 计划开始执行 |
| `/ralph-loop` | 自我引用开发循环 |

---

## 核心功能开关

```json
{
  "features": {
    "background_tasks": true,      // 并行后台任务
    "todo_enforcer": true,         // 强制完成 TODO
    "session_management": true,    // 会话管理
    "lsp_tools": true,             // LSP 重构工具
    "ast_grep": true,              // AST 代码搜索
    "hooks": true,                 // 钩子系统
    "skills": true,                // 技能系统
    "mcps": true                   // MCP 集成
  }
}
```

---

## Shell 配置 (AI Agent 兼容)

### 问题

AI Agent 执行命令时是 **non-interactive shell**，不会加载 `~/.zshrc`，导致 PATH 不完整。

### 解决方案

创建 `~/.zshenv`（所有 zsh 进程都会加载）：

```bash
# ~/.zshenv
export PATH="/opt/homebrew/bin:$PATH"
```

### Shell 配置文件加载顺序

| 文件 | 加载时机 | 用途 |
|------|----------|------|
| `~/.zshenv` | **所有** zsh 进程 | PATH 等基础环境变量 |
| `~/.zprofile` | login shell | 登录时执行一次 |
| `~/.zshrc` | interactive shell | alias、prompt、补全等 |

**AI Agent 只能访问 `~/.zshenv` 中的配置！**

---

## 已知问题

### attach client 每次刷新 models.dev

**问题**: 使用 `opencode attach` 连接到已有 server 时，client 仍然每次请求 models.dev，导致 ~1 秒延迟。

**状态**: 已提交 issue https://github.com/anomalyco/opencode/issues/10781

**临时方案**: 无，等待官方修复。

### oh-my-opencode 插件加载时间

**问题**: oh-my-opencode 增加约 0.8 秒启动时间（初始化 agents、hooks、tools 等）。

**这是功能丰富的代价**，无法避免。

---

## MCP 启用/禁用

MCP 配置在 `opencode.json` 的 `mcp` 字段，通过 `enabled` 控制：

```json
"mcp": {
  "filesystem": {
    "type": "local",
    "command": ["/opt/homebrew/bin/mcp-server-filesystem", "/Users/mingjian/Documents/sync"],
    "enabled": true,  // 改为 false 禁用
    "environment": {
      "PATH": "/opt/homebrew/bin:/usr/bin:/bin:/usr/sbin:/sbin"
    }
  }
}
```

**注意**: `mcp.json` 是另一个文件（可能用于其他 client），`opencode.json` 中的 `mcp` 字段才是 OpenCode 读取的配置。

---

## 注意事项

1. **模型引用必须匹配**: oh-my-opencode.json 引用的模型必须在 opencode.json 中定义
2. **显式配置所有智能体**: 避免依赖 fallback chain 带来的不确定性（尤其是 Hephaestus）
3. **主 model 是必须的**: oh-my-opencode 要求 opencode.json 必须定义主 model
4. **缓存可能过期**: 如果看到不存在的模型，清除 `~/.cache/opencode/models.json`
5. **JSON 语法**: 
   - 不支持尾随逗号（`},` 后面不能没有下一项）
   - 每个 `"key": "value"` 后必须有逗号（除非是最后一项）
6. **Hephaestus 必须配置**: 否则会报 `model opencode/gpt-5.2-codex is not valid` 错误
