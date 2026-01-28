---
draft: false
title: "Browser MCP - Complete Setup Guide"
description: "MCP server + Chrome extension for AI-powered browser automation"
slug: "browser-mcp-setup"
date: 2026-01-28T00:00:00+08:00
image: cover.svg
categories:
    - Tech
tags:
    - AI
    - MCP
    - Browser Automation
---

> MCP server + Chrome extension for AI-powered browser automation

## Overview

**Browser MCP** allows AI applications (VS Code, Cursor, Claude, Windsurf) to automate your real browser.

| Feature | Description |
|---------|-------------|
| Fast | Local automation, no network latency |
| Private | Activity stays on device |
| Logged In | Uses existing browser profile & sessions |
| Stealth | Avoids bot detection using real browser fingerprint |

### Available Tools

`Navigate`, `Go Back`, `Go Forward`, `Wait`, `Press Key`, `Snapshot`, `Click`, `Drag & Drop`, `Hover`, `Type Text`, `Get Console Logs`, `Screenshot`

---

## Installation

### Step 1: Prerequisites

```bash
# Ensure Node.js is installed (required)
node --version  # Should output v18+ 

# If not installed:
brew install node  # macOS
```

### Step 2: Install Chrome Extension

```bash
# Open Chrome and install the extension
open "https://chromewebstore.google.com/detail/browser-mcp-automate-your/bjfgambnhccakkhmkepdoekmckoijdlc"
```

Or search "Browser MCP" in Chrome Web Store.

### Step 3: Configure MCP Server

#### For Cursor

```bash
# Open Cursor settings and add to MCP config:
# Settings -> Tools -> New MCP Server -> paste config below
```

#### For Claude Desktop

```bash
# Edit Claude config file
# macOS:
open ~/Library/Application\ Support/Claude/claude_desktop_config.json

# Windows:
notepad %APPDATA%\Claude\claude_desktop_config.json
```

#### For VS Code

```bash
# Add to .vscode/settings.json or user settings
code ~/.config/Code/User/settings.json
```

---

## MCP Server Configuration

### Universal Config (all apps)

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

### VS Code Config (settings.json)

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

## One-Shot Setup Script (macOS)

```bash
#!/bin/bash
# Browser MCP Complete Setup

# 1. Verify Node.js
if ! command -v node &> /dev/null; then
    echo "Installing Node.js..."
    brew install node
fi

# 2. Pre-cache the MCP package
npx @browsermcp/mcp@latest --help 2>/dev/null || true

# 3. Create Claude Desktop config
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

echo "Claude Desktop configured"

# 4. Open Chrome extension page
open "https://chromewebstore.google.com/detail/browser-mcp-automate-your/bjfgambnhccakkhmkepdoekmckoijdlc"

echo "
Setup complete!

Next steps:
1. Install the Browser MCP extension in Chrome (page should be open)
2. Pin the extension to toolbar
3. Restart Claude Desktop
4. Click 'Connect' in the extension when using
"
```

---

## Usage

1. **Open Browser MCP extension** in Chrome toolbar
2. **Click "Connect"** to connect current tab
3. **In your AI app**, ask: `"Go to example.com and click the login button"`

### Example Prompts

```
"Navigate to github.com and search for 'browser mcp'"
"Fill out the contact form on this page"
"Take a screenshot of the current page"
"Click the 'Submit' button"
```

---

## Troubleshooting

### Claude Desktop Double-Start Bug

Claude Desktop currently starts MCP servers twice ([bug #812](https://github.com/modelcontextprotocol/servers/issues/812)). You'll see an error but it still works.

### Extension Not Connecting

1. Make sure you clicked "Connect" in the extension popup
2. Refresh the page and try again
3. Check that the MCP server is configured correctly

---

## Links

| Resource | URL |
|----------|-----|
| Website | https://browsermcp.io |
| Docs | https://docs.browsermcp.io |
| GitHub | https://github.com/BrowserMCP/mcp |
| Chrome Extension | [Chrome Web Store](https://chromewebstore.google.com/detail/bjfgambnhccakkhmkepdoekmckoijdlc) |
