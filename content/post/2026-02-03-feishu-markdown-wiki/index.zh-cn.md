---
draft: false
title: "飞书：Markdown → Wiki 全流程"
description: "本地 Markdown 上传到飞书 Wiki 的完整工作流，含 Mermaid 图表支持。"
slug: "feishu-markdown-wiki"
date: 2026-02-03T00:00:00+08:00
image: cover.png
categories:
    - Tech
tags:
    - Feishu
    - API
    - Automation
---

> 本地 Markdown 上传到飞书 Wiki 的完整工作流，含 Mermaid 图表支持。

## 概述

飞书**没有直接的 "上传到 Wiki" API**。你必须先经过云文档，走一个 4 步流水线：

| 步骤 | 操作 | 认证 | 备注 |
|------|------|------|------|
| 1 | 导入 Markdown → 云文档 | TAT 或 UAT | 创建的文档归应用所有 |
| 2 | 转移文档所有权给用户 | TAT | Wiki 移动需要用户权限 |
| 3 | 移动云文档 → Wiki 节点 | UAT | **步骤 2 后等 3 秒！** |
| 4 | 修复 Mermaid 块 | UAT | 把 type 2 → type 40 |

---

## 前置条件

- 飞书应用，具有 `wiki`、`docx`、`drive` 权限
- UAT（用户访问令牌）— 通过 OAuth 流程刷新
- TAT（租户访问令牌）— 通过应用凭证获取

```bash
# 配置
APP_ID="cli_xxxxxxxxxxxxx"
APP_SECRET="your_app_secret"
WIKI_SPACE_ID="your_wiki_space_id"
PARENT_NODE="your_parent_node_token"
USER_OPEN_ID="your_user_open_id"

# 获取 TAT
TAT=$(curl -s -X POST 'https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal' \
  -H 'Content-Type: application/json' \
  -d "{\"app_id\":\"$APP_ID\",\"app_secret\":\"$APP_SECRET\"}" \
  | python3 -c "import json,sys; print(json.load(sys.stdin)['tenant_access_token'])")

# 从缓存获取 UAT
UAT=$(python3 -c "import json; print(json.load(open('$HOME/.feishu_token_cache.json'))['access_token'])")
```

---

## 步骤 1：导入 Markdown → 云文档

```bash
curl -X POST "https://open.feishu.cn/open-apis/docx/v1/documents/import" \
  -H "Authorization: Bearer $TAT" \
  -H "Content-Type: application/json" \
  -d '{"markdown": "# 你的文档\n\n内容...", "file_name": "我的文档"}'
```

**已知限制**：
- Mermaid 代码块会被扁平化为纯文本段落
- 最大文件大小：20MB
- 中文文本在某些场景下可能需要 Unicode 转义（`\uXXXX`）

**输出**：返回 `doc_token`，后续步骤使用。

---

## 步骤 2：转移所有权给用户

导入操作创建的文档归应用所有。移动到 Wiki 需要用户所有权。

```bash
curl -X POST "https://open.feishu.cn/open-apis/drive/v1/permissions/${DOC_TOKEN}/members/transfer_owner?type=docx" \
  -H "Authorization: Bearer $TAT" \
  -H "Content-Type: application/json" \
  -d "{\"member_type\":\"openid\",\"member_id\":\"$USER_OPEN_ID\"}"
```

**注意**：
- 使用 TAT（应用转移给用户）
- 如果用户已经是所有者，返回错误码 `1063002` — 可安全忽略
- **⚠️ 关键**：此步骤后等 3 秒再进行下一步！

```bash
sleep 3  # 不要跳过 — 否则 Wiki 移动会失败
```

---

## 步骤 3：移动云文档 → Wiki 节点

```bash
curl -X POST "https://open.feishu.cn/open-apis/wiki/v2/spaces/${WIKI_SPACE_ID}/nodes/move_docs_to_wiki" \
  -H "Authorization: Bearer $UAT" \
  -H "Content-Type: application/json" \
  -d "{\"parent_wiki_token\":\"$PARENT_NODE\",\"obj_type\":\"docx\",\"obj_token\":\"$DOC_TOKEN\"}"
```

**注意**：
- 使用 **UAT** — 用户必须是文档所有者
- 返回 `wiki_token`（不同于 `doc_token`）
- 如果丢失了 `wiki_token`，可以后续查询：

```bash
curl -X GET "https://open.feishu.cn/open-apis/wiki/v2/spaces/get_node?token=${DOC_TOKEN}&obj_type=docx" \
  -H "Authorization: Bearer $UAT"
```

---

## 步骤 4：修复 Mermaid 块

Mermaid 代码块被导入为纯文本段落（`block_type=2`）。需要把它们替换成原生 Mermaid 图表块（`block_type=40`）。

**⚠️ 从下往上处理**，避免删除后索引偏移。

### 4a. 查找被扁平化的 Mermaid 块

```bash
curl -s -X GET "https://open.feishu.cn/open-apis/docx/v1/documents/${DOC_TOKEN}/blocks?page_size=200" \
  -H "Authorization: Bearer $UAT" | python3 -c "
import json, sys
data = json.load(sys.stdin)
for item in data['data']['items']:
    if item.get('block_type') != 2: continue
    for el in item.get('text', {}).get('elements', []):
        content = el.get('text_run', {}).get('content', '')
        if any(kw in content for kw in ['graph TD', 'graph LR', 'flowchart', 'sequenceDiagram']):
            print(f'FOUND: {item[\"block_id\"]} | {content[:100]}')
"
```

识别方式：段落（`type=2`）中包含箭头模式（`-->`）加上 Mermaid 关键词（`graph`、`flowchart`、`subgraph`）。

### 4b. 获取块索引

```bash
curl -s -X GET "https://open.feishu.cn/open-apis/docx/v1/documents/${DOC_TOKEN}/blocks/${DOC_TOKEN}" \
  -H "Authorization: Bearer $UAT" | python3 -c "
import json, sys
data = json.load(sys.stdin)
children = data['data']['block']['children']
target = '${BLOCK_ID}'
for i, c in enumerate(children):
    if c == target:
        print(f'Index: {i}')
"
```

### 4c. 创建 Mermaid 图表块

Mermaid 块的关键结构：

```json
{
  "block_type": 40,
  "add_ons": {
    "component_type_id": "blk_631fefbbae02400430b8f9f4",
    "record": "{\"data\":\"graph TD\\n    A --> B\",\"theme\":\"default\",\"view\":\"chart\"}"
  }
}
```

- `component_type_id` 固定为：`blk_631fefbbae02400430b8f9f4`
- `record` 是 **JSON 字符串**（请求体中双重编码）
- `record.data`：原始 Mermaid 代码，`\n` 表示换行
- `record.view`：`"chart"` 渲染图表

```bash
RECORD=$(python3 -c "
import json
data = {'data': '''graph LR
    A --> B --> C''', 'theme': 'default', 'view': 'chart'}
print(json.dumps(json.dumps(data, ensure_ascii=False)))
")

curl -s -X POST "https://open.feishu.cn/open-apis/docx/v1/documents/${DOC_TOKEN}/blocks/${DOC_TOKEN}/children" \
  -H "Authorization: Bearer $UAT" \
  -H "Content-Type: application/json" \
  -d "{
    \"children\": [{
      \"block_type\": 40,
      \"add_ons\": {
        \"component_type_id\": \"blk_631fefbbae02400430b8f9f4\",
        \"record\": $RECORD
      }
    }],
    \"index\": $((BLOCK_INDEX + 1))
  }"
```

### 4d. 删除旧的扁平化文本块

```bash
curl -s -X DELETE "https://open.feishu.cn/open-apis/docx/v1/documents/${DOC_TOKEN}/blocks/${DOC_TOKEN}/children/batch_delete" \
  -H "Authorization: Bearer $UAT" \
  -H "Content-Type: application/json" \
  -d "{\"start_index\": ${BLOCK_INDEX}, \"end_index\": $((BLOCK_INDEX + 1))}"
```

### Record 编码辅助

```python
import json

mermaid_code = """graph TD
    A[开始] --> B[处理]
    B --> C{判断}
    C -->|是| D[结束]
    C -->|否| B"""

record = json.dumps({
    "data": mermaid_code,
    "theme": "default",
    "view": "chart"
}, ensure_ascii=False)

# curl 请求体中需要双重编码：
request_record = json.dumps(record)
```

---

## 块类型参考

| 类型 | 名称 | 备注 |
|------|------|------|
| 1 | 页面（根） | 文档根节点 |
| 2 | 文本/段落 | Mermaid 被扁平化到这里 |
| 3-8 | 标题 1-6 | |
| 12 | 无序列表 | |
| 13 | 有序列表 | |
| 14 | 代码块 | 仅语法高亮 — **不渲染 Mermaid** |
| 15 | 引用 | |
| 22 | 高亮块 | |
| 31 | 表格 | 包含 type-32 的表格单元格子节点 |
| 40 | **插件块** | **Mermaid 图表在这里** |

---

## API 端点汇总

| 步骤 | 方法 | 端点 | 认证 |
|------|------|------|------|
| 导入 | POST | `/open-apis/docx/v1/documents/import` | TAT 或 UAT |
| 转移 | POST | `/open-apis/drive/v1/permissions/{token}/members/transfer_owner?type=docx` | TAT |
| 移到 Wiki | POST | `/open-apis/wiki/v2/spaces/{space_id}/nodes/move_docs_to_wiki` | UAT |
| 列出块 | GET | `/open-apis/docx/v1/documents/{doc_id}/blocks` | UAT |
| 获取块 | GET | `/open-apis/docx/v1/documents/{doc_id}/blocks/{block_id}` | UAT |
| 创建块 | POST | `/open-apis/docx/v1/documents/{doc_id}/blocks/{parent_id}/children` | UAT |
| 删除块 | DELETE | `/open-apis/docx/v1/documents/{doc_id}/blocks/{parent_id}/children/batch_delete` | UAT |
| 获取 Wiki 节点 | GET | `/open-apis/wiki/v2/spaces/get_node?token={token}` | UAT |

---

## 踩坑与经验

| 问题 | 解决方案 |
|------|----------|
| 转移所有权后 Wiki 移动失败 | **转移和移动之间等 3 秒** |
| Mermaid 被扁平化为纯文本 | 用块 API 后处理（type 40） |
| 代码块（type 14）不渲染 Mermaid | 必须用 type 40 插件块 |
| PATCH 无法改变 block_type | 新建块 + 删旧块 |
| 删除后块索引偏移 | 从下往上处理 |
| `record` 字段编码 | 双重编码：`json.dumps(json.dumps(data))` |
| 丢失 wiki_token | 通过 `get_node?token={doc_token}&obj_type=docx` 找回 |
| TAT 无法移动文档到 Wiki | 先转移所有权，再用 UAT |
| API 请求体中的中文 | 某些端点需要 `\uXXXX` 转义 |
| Wiki 节点删除 | 不支持 API 删除 — 只能手动 |
