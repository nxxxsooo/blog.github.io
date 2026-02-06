---
draft: false
title: "Feishu: Markdown → Wiki Pipeline"
description: "End-to-end workflow for uploading local markdown to Feishu Wiki with mermaid diagram support."
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

> End-to-end workflow for uploading local markdown to Feishu Wiki with mermaid diagram support.

## Overview

Feishu has **no direct "upload to wiki" API**. You must go through cloud docs first, following a 4-step pipeline:

| Step | Action | Auth | Notes |
|------|--------|------|-------|
| 1 | Import Markdown → Cloud Doc | TAT or UAT | Creates doc owned by app |
| 2 | Transfer ownership to user | TAT | Required for wiki move |
| 3 | Move Cloud Doc → Wiki Node | UAT | **Sleep 3s after step 2!** |
| 4 | Fix Mermaid blocks | UAT | Replace type 2 → type 40 |

---

## Prerequisites

- Feishu App with `wiki`, `docx`, `drive` scopes
- UAT (User Access Token) — refreshed via OAuth flow
- TAT (Tenant Access Token) — obtained via app credentials

```bash
# Config
APP_ID="cli_xxxxxxxxxxxxx"
APP_SECRET="your_app_secret"
WIKI_SPACE_ID="your_wiki_space_id"
PARENT_NODE="your_parent_node_token"
USER_OPEN_ID="your_user_open_id"

# Get TAT
TAT=$(curl -s -X POST 'https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal' \
  -H 'Content-Type: application/json' \
  -d "{\"app_id\":\"$APP_ID\",\"app_secret\":\"$APP_SECRET\"}" \
  | python3 -c "import json,sys; print(json.load(sys.stdin)['tenant_access_token'])")

# Get UAT from cached token
UAT=$(python3 -c "import json; print(json.load(open('$HOME/.feishu_token_cache.json'))['access_token'])")
```

---

## Step 1: Import Markdown → Cloud Doc

```bash
curl -X POST "https://open.feishu.cn/open-apis/docx/v1/documents/import" \
  -H "Authorization: Bearer $TAT" \
  -H "Content-Type: application/json" \
  -d '{"markdown": "# Your Document\n\nContent here...", "file_name": "My Document"}'
```

**Known limitations**:
- Mermaid code blocks are flattened to plain text paragraphs
- Max file size: 20MB
- Chinese text may need Unicode escaping (`\uXXXX`) in some contexts

**Output**: Returns a `doc_token` for use in subsequent steps.

---

## Step 2: Transfer Ownership to User

Import creates a doc owned by the app. Wiki move requires user ownership.

```bash
curl -X POST "https://open.feishu.cn/open-apis/drive/v1/permissions/${DOC_TOKEN}/members/transfer_owner?type=docx" \
  -H "Authorization: Bearer $TAT" \
  -H "Content-Type: application/json" \
  -d "{\"member_type\":\"openid\",\"member_id\":\"$USER_OPEN_ID\"}"
```

**Notes**:
- Uses TAT (app transfers to user)
- If user already owns it, returns error code `1063002` — safe to ignore
- **⚠️ CRITICAL**: Wait 3 seconds after this before the next step!

```bash
sleep 3  # DO NOT SKIP — wiki move fails without this delay
```

---

## Step 3: Move Cloud Doc → Wiki Node

```bash
curl -X POST "https://open.feishu.cn/open-apis/wiki/v2/spaces/${WIKI_SPACE_ID}/nodes/move_docs_to_wiki" \
  -H "Authorization: Bearer $UAT" \
  -H "Content-Type: application/json" \
  -d "{\"parent_wiki_token\":\"$PARENT_NODE\",\"obj_type\":\"docx\",\"obj_token\":\"$DOC_TOKEN\"}"
```

**Notes**:
- Uses **UAT** — user must own the doc
- Returns `wiki_token` (different from `doc_token`)
- If you lose the `wiki_token`, retrieve it later:

```bash
curl -X GET "https://open.feishu.cn/open-apis/wiki/v2/spaces/get_node?token=${DOC_TOKEN}&obj_type=docx" \
  -H "Authorization: Bearer $UAT"
```

---

## Step 4: Fix Mermaid Blocks

Mermaid code blocks are imported as plain text paragraphs (`block_type=2`). You need to replace them with native mermaid diagram blocks (`block_type=40`).

**⚠️ Process from bottom to top** to avoid index drift after deletion.

### 4a. Find flattened mermaid blocks

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

Detection heuristic: paragraphs (`type=2`) containing arrow patterns (`-->`) plus mermaid keywords (`graph`, `flowchart`, `subgraph`).

### 4b. Get block index

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

### 4c. Create mermaid diagram block

The key structure for a Mermaid block:

```json
{
  "block_type": 40,
  "add_ons": {
    "component_type_id": "blk_631fefbbae02400430b8f9f4",
    "record": "{\"data\":\"graph TD\\n    A --> B\",\"theme\":\"default\",\"view\":\"chart\"}"
  }
}
```

- `component_type_id` is fixed: `blk_631fefbbae02400430b8f9f4`
- `record` is a **JSON string** (double-encoded in request body)
- `record.data`: raw mermaid code, `\n` for newlines
- `record.view`: `"chart"` renders the diagram

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

### 4d. Delete old flattened text block

```bash
curl -s -X DELETE "https://open.feishu.cn/open-apis/docx/v1/documents/${DOC_TOKEN}/blocks/${DOC_TOKEN}/children/batch_delete" \
  -H "Authorization: Bearer $UAT" \
  -H "Content-Type: application/json" \
  -d "{\"start_index\": ${BLOCK_INDEX}, \"end_index\": $((BLOCK_INDEX + 1))}"
```

### Record encoding helper

```python
import json

mermaid_code = """graph TD
    A[Start] --> B[Process]
    B --> C{Decision}
    C -->|Yes| D[End]
    C -->|No| B"""

record = json.dumps({
    "data": mermaid_code,
    "theme": "default",
    "view": "chart"
}, ensure_ascii=False)

# For curl request body, double-encode:
request_record = json.dumps(record)
```

---

## Block Type Reference

| Type | Name | Notes |
|------|------|-------|
| 1 | Page (root) | Document root |
| 2 | Text/Paragraph | Where mermaid gets flattened to |
| 3-8 | Heading 1-6 | |
| 12 | Bullet List | |
| 13 | Ordered List | |
| 14 | Code Block | Syntax highlighting only — **does NOT render mermaid** |
| 15 | Quote | |
| 22 | Callout | |
| 31 | Table | Has type-32 table_cell children |
| 40 | **Add-ons** | **Mermaid diagrams live here** |

---

## API Endpoints Summary

| Step | Method | Endpoint | Auth |
|------|--------|----------|------|
| Import | POST | `/open-apis/docx/v1/documents/import` | TAT or UAT |
| Transfer | POST | `/open-apis/drive/v1/permissions/{token}/members/transfer_owner?type=docx` | TAT |
| Move to Wiki | POST | `/open-apis/wiki/v2/spaces/{space_id}/nodes/move_docs_to_wiki` | UAT |
| List blocks | GET | `/open-apis/docx/v1/documents/{doc_id}/blocks` | UAT |
| Get block | GET | `/open-apis/docx/v1/documents/{doc_id}/blocks/{block_id}` | UAT |
| Create block | POST | `/open-apis/docx/v1/documents/{doc_id}/blocks/{parent_id}/children` | UAT |
| Delete blocks | DELETE | `/open-apis/docx/v1/documents/{doc_id}/blocks/{parent_id}/children/batch_delete` | UAT |
| Get wiki node | GET | `/open-apis/wiki/v2/spaces/get_node?token={token}` | UAT |

---

## Gotchas & Lessons Learned

| Issue | Solution |
|-------|----------|
| Wiki move fails after ownership transfer | **Sleep 3 seconds** between transfer and move |
| Mermaid flattened to plain text | Post-process with block API (type 40) |
| Code block (type 14) doesn't render mermaid | Must use type 40 add_ons |
| PATCH cannot change block_type | Create new block + delete old block |
| Block indices shift after delete | Process bottom to top |
| `record` field encoding | Double-encode: `json.dumps(json.dumps(data))` |
| Lost wiki_token | Retrieve via `get_node?token={doc_token}&obj_type=docx` |
| TAT can't move docs to wiki | Transfer ownership first, then use UAT |
| Chinese in API payloads | Some endpoints need `\uXXXX` escaping |
| Wiki node deletion | Not supported via API — manual only |
