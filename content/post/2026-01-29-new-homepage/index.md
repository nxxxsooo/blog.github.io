---
draft: false
title: "New Homepage with Magic Portfolio"
description: "Why I switched my homepage to Magic Portfolio on Vercel"
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

![New Homepage](cover.jpg "New Homepage")

> Migrating my main site to a Next.js portfolio hosted on Vercel.

## The Switch

I've moved my homepage (`mjshao.fun`) from a static HTML page to [Magic Portfolio](https://github.com/once-ui-system/magic-portfolio), a modern Next.js template built with Once UI.

### Why?

- **Better Design**: Clean, minimal, and responsive out of the box.
- **MDX Support**: Easy to add content and projects.
- **Tech Stack**: Next.js 14 + React 19 + TypeScript.

## The 404 Issue

You might have noticed that `mjshao.fun/siliconlm` is currently broken. 

This is because `mjshao.fun` now points to Vercel (Next.js app), which doesn't know about my old GitHub Pages projects.

### Solution

I've migrated the **SiliconLM** documentation to this blog:

👉 [SiliconLM Project Page](/p/siliconlm/)

I'll be migrating other project links gradually.

## Tech Stack

| Component | Technology |
|-----------|------------|
| Framework | Next.js 14 (App Router) |
| UI Library | Once UI |
| Styling | SCSS + CSS Modules |
| Deployment | Vercel |

Check out the new homepage at [mjshao.fun](https://mjshao.fun).
