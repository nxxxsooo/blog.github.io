---
draft: false
title: "Browser MCP 完整配置指南"
description: "MCP 服务器 + Chrome 扩展实现 AI 驱动的浏览器自动化"
slug: "browser-mcp-setup"
date: 2026-01-28T00:00:00+08:00
image: cover.png
categories:
    - Tech
tags:
    - AI
    - MCP
    - Browser Automation
---

> MCP 服务器 + Chrome 扩展实现 AI 驱动的浏览器自动化

## 概述

**Browser MCP** 让 AI 应用（VS Code、Cursor、Claude、Windsurf）可以自动化操控你的真实浏览器。

| 特性 | 说明 |
|------|------|
| 快速 | 本地自动化，无网络延迟 |
| 隐私 | 所有操作留在本地 |
| 已登录 | 使用现有浏览器配置和登录状态 |
| 隐蔽 | 使用真实浏览器指纹，规避机器人检测 |

### 可用工具

`Navigate`、`Go Back`、`Go Forward`、`Wait`、`Press Key`、`Snapshot`、`Click`、`Drag & Drop`、`Hover`、`Type Text`、`Get Console Logs`、`Screenshot`

---

## 安装

### 第一步：前置条件

```bash
# 确保已安装 Node.js（必需）
node --version  # 应输出 v18+

# 如未安装：
brew install node  # macOS
```

### 第二步：安装 Chrome 扩展

```bash
# 打开 Chrome 安装扩展
open "https://chromewebstore.google.com/detail/browser-mcp-automate-your/bjfgambnhccakkhmkepdoekmckoijdlc"
```

或在 Chrome 网上应用店搜索 "Browser MCP"。

### 第三步：配置 MCP 服务器

#### Cursor

```bash
# 打开 Cursor 设置，添加 MCP 配置：
# Settings -> Tools -> New MCP Server -> 粘贴以下配置
```

#### Claude Desktop

```bash
# 编辑 Claude 配置文件
# macOS:
open ~/Library/Application\ Support/Claude/claude_desktop_config.json

# Windows:
notepad %APPDATA%\Claude\claude_desktop_config.json
```

#### VS Code

```bash
# 添加到 .vscode/settings.json 或用户设置
code ~/.config/Code/User/settings.json
```

---

## MCP 服务器配置

### 通用配置（所有应用）

```json
{
  "mcpServers": {
    "browsermcp": {
      "command": "npx",
      "args": ["@browsermcp/mcp@latest"]
    }
  }
}
```

### VS Code 配置 (settings.json)

```json
{
  "mcp": {
    "servers": {
      "browsermcp": {
        "command": "npx",
        "args": ["@browsermcp/mcp@latest"]
      }
    }
  }
}
```

---

## 一键安装脚本 (macOS)

```bash
#!/bin/bash
# Browser MCP 完整安装

# 1. 检查 Node.js
if ! command -v node &> /dev/null; then
    echo "正在安装 Node.js..."
    brew install node
fi

# 2. 预缓存 MCP 包
npx @browsermcp/mcp@latest --help 2>/dev/null || true

# 3. 创建 Claude Desktop 配置
CLAUDE_CONFIG="$HOME/Library/Application Support/Claude/claude_desktop_config.json"
mkdir -p "$(dirname "$CLAUDE_CONFIG")"

cat > "$CLAUDE_CONFIG" << 'EOF'
{
  "mcpServers": {
    "browsermcp": {
      "command": "npx",
      "args": ["@browsermcp/mcp@latest"]
    }
  }
}
EOF

echo "Claude Desktop 配置完成"

# 4. 打开 Chrome 扩展页面
open "https://chromewebstore.google.com/detail/browser-mcp-automate-your/bjfgambnhccakkhmkepdoekmckoijdlc"

echo "
安装完成！

后续步骤：
1. 在 Chrome 中安装 Browser MCP 扩展（页面应已打开）
2. 将扩展固定到工具栏
3. 重启 Claude Desktop
4. 使用时在扩展中点击「Connect」
"
```

---

## 使用方法

1. 点击 Chrome 工具栏中的 **Browser MCP 扩展**
2. 点击 **「Connect」** 连接当前标签页
3. 在 AI 应用中输入指令，例如：`"打开 example.com 并点击登录按钮"`

### 示例提示词

```
"打开 github.com 搜索 'browser mcp'"
"填写这个页面上的联系表单"
"截取当前页面的截图"
"点击「提交」按钮"
```

---

## 故障排除

### Claude Desktop 双启动 Bug

Claude Desktop 目前会启动两次 MCP 服务器（[bug #812](https://github.com/modelcontextprotocol/servers/issues/812)）。你会看到一个错误，但功能正常。

### 扩展无法连接

1. 确认已在扩展弹窗中点击了「Connect」
2. 刷新页面后重试
3. 检查 MCP 服务器配置是否正确

---

## 链接

| 资源 | 链接 |
|------|------|
| 官网 | https://browsermcp.io |
| 文档 | https://docs.browsermcp.io |
| GitHub | https://github.com/BrowserMCP/mcp |
| Chrome 扩展 | [Chrome 网上应用店](https://chromewebstore.google.com/detail/bjfgambnhccakkhmkepdoekmckoijdlc) |
