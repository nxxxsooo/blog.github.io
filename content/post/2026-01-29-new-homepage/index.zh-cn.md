---
draft: false
title: "新主页：Magic Portfolio"
description: "为什么我把主页换成了 Vercel 上的 Magic Portfolio"
slug: "new-homepage-magic-portfolio"
date: 2026-01-29T10:00:00+08:00
image: cover.jpg
categories:
    - Tech
tags:
    - Next.js
    - Portfolio
    - Vercel
---

![New Homepage](cover.jpg "新主页")

> 把主站迁移到 Vercel 上的 Next.js 作品集。

## 换站

我把主页（`mjshao.fun`）从静态 HTML 页面换成了 [Magic Portfolio](https://github.com/once-ui-system/magic-portfolio)，一个基于 Once UI 的现代 Next.js 模板。

### 为什么？

- **设计更好**：开箱即用的简洁、美观、响应式设计。
- **MDX 支持**：方便添加内容和项目展示。
- **技术栈**：Next.js 14 + React 19 + TypeScript。

## 404 问题

你可能注意到 `mjshao.fun/siliconlm` 现在打不开了。

这是因为 `mjshao.fun` 现在指向了 Vercel（Next.js 应用），它不知道我旧的 GitHub Pages 项目。

### 解决方案

我已经把 **SiliconLM** 文档迁移到了这个博客：

👉 [SiliconLM 项目页](/p/siliconlm/)

其他项目链接会逐步迁移。

## 技术栈

| 组件 | 技术 |
|------|------|
| 框架 | Next.js 14 (App Router) |
| UI 库 | Once UI |
| 样式 | SCSS + CSS Modules |
| 部署 | Vercel |

看看新主页：[mjshao.fun](https://mjshao.fun)。
