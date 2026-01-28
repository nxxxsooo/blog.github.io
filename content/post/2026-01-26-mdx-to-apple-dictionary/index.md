---
draft: false
title: "MDX 转 Apple Dictionary 完整指南"
description: "将 MDict (.mdx/.mdd) 词典转换为 macOS Apple Dictionary 格式"
slug: "mdx-to-apple-dictionary"
date: 2026-01-26T16:13:22+08:00
image: cover.png
categories:
    - Tech
tags:
    - macOS
    - Dictionary
    - Python
---

将 MDict (.mdx/.mdd) 词典转换为 macOS Apple Dictionary 格式。

## 推荐工具

我写了一个自动化脚本 [mdx2apple](https://github.com/nxxxsooo/mdx2apple)，可以一键完成整个转换流程：

```bash
git clone https://github.com/nxxxsooo/mdx2apple.git
cd mdx2apple
pip install pyglossary lxml beautifulsoup4 html5lib

# 一键转换并安装
python mdx2apple.py /path/to/dictionary.mdx --install
```

如果你想了解手动转换的原理，继续往下看。

---

## 手动转换指南

### 工具准备

### 1. 安装 pyglossary
```bash
pip3 install --user pyglossary lxml beautifulsoup4 html5lib
```

### 2. 安装 Dictionary Development Kit (DDK)
从 GitHub 镜像安装（无需 Apple Developer 账号）：
```bash
git clone https://github.com/ikey4u/macddk.git ~/Library/Developer/Dictionary-Development-Kit
```

## 转换步骤

### Step 1: MDX 转 AppleDict 源文件
```bash
pyglossary input.mdx output.apple --write-format=AppleDict
```

生成的文件：
- `output.apple/` 目录
- `*.xml` - 词典内容
- `*.css` - 样式
- `*.plist` - 元数据
- `Makefile`
- `OtherResources/` - 音频、图片等资源

### Step 2: 关键修复

#### 问题1：CSS 缺少 display:block
原始 CSS 中 `d|entry {}` 为空，导致内容不显示。

修复：
```css
d|entry {
    display: block;
}
```

#### 问题2：HTML 结构过于复杂
MDict HTML 包含大量自定义属性和复杂嵌套，Apple Dictionary 无法正确渲染。

解决方案：用 Python 清理 XML，保留核心内容：

```python
from bs4 import BeautifulSoup
import re

def clean_entry(entry):
    # 保留 d:entry 标签和 d:index 索引
    # 提取：headword, pos, pronunciation, definitions, examples
    # 移除：自定义属性(hclass, htag, idm_id等)、javascript、复杂嵌套
    # 转义特殊字符：& < >
    pass
```

#### 问题3：plist 过于复杂
pyglossary 生成的 plist 可能有问题。使用简化版：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "...">
<plist version="1.0">
<dict>
    <key>CFBundleDevelopmentRegion</key>
    <string>English</string>
    <key>CFBundleIdentifier</key>
    <string>com.example.dict</string>
    <key>CFBundleDisplayName</key>
    <string>My Dictionary</string>
    <key>CFBundleName</key>
    <string>MyDict</string>
    <key>CFBundleShortVersionString</key>
    <string>1.0</string>
    <key>DCSDictionaryCopyright</key>
    <string>Copyright</string>
    <key>DCSDictionaryManufacturerName</key>
    <string>Publisher</string>
</dict>
</plist>
```

### Step 3: 编译

修改 Makefile 中的 DDK 路径：
```makefile
DICT_BUILD_TOOL_DIR = "$(HOME)/Library/Developer/Dictionary-Development-Kit"
DICT_BUILD_OPTS = -v 10.11
```

资源文件太多会导致编译失败，需要先移走：
```bash
mv OtherResources /tmp/
make
mv /tmp/OtherResources ./
```

### Step 4: 安装

```bash
# 复制词典
cp -R objects/MyDict.dictionary ~/Library/Dictionaries/

# 添加资源（音频、图片）
rsync -a OtherResources/ ~/Library/Dictionaries/MyDict.dictionary/Contents/Resources/

# 清除缓存
rm -rf ~/Library/Caches/com.apple.Dictionary
killall Dictionary
killall DictionaryWorkerService
```

## 限制

1. **无音频播放**：Apple Dictionary 不支持 JavaScript 音频，MDict 的发音功能无法使用
2. **复杂排版丢失**：原始词典的复杂 CSS 样式大部分无法保留
3. **大词典可能有问题**：6万+ 词条需要简化 HTML 结构才能正常显示

## 调试技巧

1. **先测试小词典**：创建 100 条目的测试词典验证流程
2. **检查 Body.data**：如果为空或很小，说明内容未正确编译
3. **查看 Console.app**：搜索 "Dictionary" 相关日志
4. **简化 plist**：复杂的 plist 设置可能导致问题

## 文件位置
- 词典安装位置：`~/Library/Dictionaries/`
- 缓存位置：`~/Library/Caches/com.apple.Dictionary/`
- DDK 位置：`~/Library/Developer/Dictionary-Development-Kit/`

## 参考
- [pyglossary](https://github.com/ilius/pyglossary)
- [DDK 镜像](https://github.com/ikey4u/macddk)
- Apple Dictionary 格式文档: Apple Developer Documentation
