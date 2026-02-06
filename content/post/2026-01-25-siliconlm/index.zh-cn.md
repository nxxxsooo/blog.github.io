---
draft: false
title: "SiliconLM - Apple Silicon 本地大模型控制台"
description: "Apple Silicon Mac 本地大模型管理工具，支持模型管理、服务控制、Embedding 和下载"
slug: "siliconlm"
date: 2026-01-25T02:05:13+08:00
image: cover.png
categories:
    - Tech
tags:
    - Apple Silicon
    - LLM
    - MLX
    - Python
links:
    - title: GitHub 仓库
      description: 源代码
      website: https://github.com/nxxxsooo/siliconlm
      image: https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png
---

![SiliconLM Dashboard](cover.png "SiliconLM Dashboard")

> Apple Silicon Mac 本地大模型管理工具。管理模型、服务、Embedding 和下载。

[项目主页](https://mjshao.fun/siliconlm/) | [GitHub 仓库](https://github.com/nxxxsooo/siliconlm)

## 功能特点

- **硬件信息** - 芯片、GPU 核心、Neural Engine、内存、磁盘一目了然
- **MLX Embedding 服务** - 兼容 OpenAI 的 `/v1/embeddings` API，端口 8766
- **多后端支持** - MLX、mlx-lm（解码器模型）、sentence-transformers
- **服务管理** - 启停 LMStudio、MLX Embeddings、OpenCode
- **智能代理** - `/v1/embeddings` 路由到 MLX，`/v1/chat` 路由到 LMStudio
- **模型下载** - HuggingFace 搜索 + aria2 加速大文件下载
- **设置面板** - 配置模型目录、默认 Embedding 模型

## 架构

```
CherryStudio / 客户端
        │
        ▼
http://localhost:8765/v1/*  (SiliconLM 代理)
        │
   ┌────┴────┐
   ▼         ▼
/v1/embeddings   /v1/chat/*
   │              │
   ▼              ▼
:8766 (MLX)    :1234 (LMStudio)
   │
   ├─► MLX (bert, roberta)
   ├─► mlx-lm (Qwen3, gte-Qwen2)
   └─► sentence-transformers (bge-m3)
```

## 支持的 Embedding 模型

| 模型 | 后端 | 维度 | 速度 |
|------|------|------|------|
| mixedbread-ai/mxbai-embed-large-v1 | MLX | 1024 | 快 |
| BAAI/bge-m3 | sentence-transformers | 1024 | 中等 |
| mlx-community/Qwen3-Embedding-0.6B-4bit | mlx-lm | 1024 | 快 |
| mlx-community/Qwen3-Embedding-8B-4bit | mlx-lm | 4096 | 中等 |
| mlx-community/gte-Qwen2-7B-instruct-4bit | mlx-lm | 3584 | 中等 |

## 技术栈

| 组件 | 技术 |
|------|------|
| 后端 | FastAPI + uvicorn |
| 前端 | TailwindCSS + Vanilla JS |
| Embedding | MLX + mlx-lm + sentence-transformers |
| 下载 | huggingface_hub + aria2 |
| 代理 | httpx async |

## 许可证

MIT
