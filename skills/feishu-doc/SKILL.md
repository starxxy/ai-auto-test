---
name: feishu-doc
description: |
  Feishu document read/write operations. Activate when user mentions Feishu docs, cloud docs, or docx links.
---

# Feishu Document Tool

Single tool `feishu_doc` with action parameter for all document operations, including table creation for Docx.

## Token Extraction

From URL `https://xxx.feishu.cn/docx/ABC123def` 鈫?`doc_token` = `ABC123def`

## Actions

### Read Document

```json
{ "action": "read", "doc_token": "ABC123def" }
```

Returns: title, plain text content, block statistics. Check `hint` field - if present, structured content (tables, images) exists that requires `list_blocks`.

### Write Document (Replace All)

```json
{ "action": "write", "doc_token": "ABC123def", "content": "# Title\n\nMarkdown content..." }
```

Replaces entire document with markdown content. Supports: headings, lists, code blocks, quotes, links, images (`![](url)` auto-uploaded), bold/italic/strikethrough.

**Limitation:** Markdown tables are NOT supported.

### Append Content

```json
{ "action": "append", "doc_token": "ABC123def", "content": "Additional content" }
```

Appends markdown to end of document.

### Create Document

```json
{ "action": "create", "title": "New Document", "owner_open_id": "ou_xxx" }
```

With folder:

```json
{
  "action": "create",
  "title": "New Document",
  "folder_token": "fldcnXXX",
  "owner_open_id": "ou_xxx"
}
```

**Important:** Always pass `owner_open_id` with the requesting user's `open_id` (from inbound metadata `sender_id`) so the user automatically gets `full_access` permission on the created document. Without this, only the bot app has access.

### List Blocks

```json
{ "action": "list_blocks", "doc_token": "ABC123def" }
```

Returns full block data including tables, images. Use this to read structured content.

### Get Single Block

```json
{ "action": "get_block", "doc_token": "ABC123def", "block_id": "doxcnXXX" }
```

### Update Block Text

```json
{
  "action": "update_block",
  "doc_token": "ABC123def",
  "block_id": "doxcnXXX",
  "content": "New text"
}
```

### Delete Block

```json
{ "action": "delete_block", "doc_token": "ABC123def", "block_id": "doxcnXXX" }
```

### Create Table (Docx Table Block)

```json
{
  "action": "create_table",
  "doc_token": "ABC123def",
  "row_size": 2,
  "column_size": 2,
  "column_width": [200, 200]
}
```

Optional: `parent_block_id` to insert under a specific block.

### Write Table Cells

```json
{
  "action": "write_table_cells",
  "doc_token": "ABC123def",
  "table_block_id": "doxcnTABLE",
  "values": [
    ["A1", "B1"],
    ["A2", "B2"]
  ]
}
```

### Create Table With Values (One-step)

```json
{
  "action": "create_table_with_values",
  "doc_token": "ABC123def",
  "row_size": 2,
  "column_size": 2,
  "column_width": [200, 200],
  "values": [
    ["A1", "B1"],
    ["A2", "B2"]
  ]
}
```

Optional: `parent_block_id` to insert under a specific block.

### Upload Image to Docx (from URL or local file)

```json
{
  "action": "upload_image",
  "doc_token": "ABC123def",
  "url": "https://example.com/image.png"
}
```

Or local path with position control:

```json
{
  "action": "upload_image",
  "doc_token": "ABC123def",
  "file_path": "/tmp/image.png",
  "parent_block_id": "doxcnParent",
  "index": 5
}
```

Optional `index` (0-based) inserts the image at a specific position among sibling blocks. Omit to append at end.

**Note:** Image display size is determined by the uploaded image's pixel dimensions. For small images (e.g. 480x270 GIFs), scale to 800px+ width before uploading to ensure proper display.

### Upload File Attachment to Docx (from URL or local file)

```json
{
  "action": "upload_file",
  "doc_token": "ABC123def",
  "url": "https://example.com/report.pdf"
}
```

Or local path:

```json
{
  "action": "upload_file",
  "doc_token": "ABC123def",
  "file_path": "/tmp/report.pdf",
  "filename": "Q1-report.pdf"
}
```

Rules:

- exactly one of `url` / `file_path`
- optional `filename` override
- optional `parent_block_id`

## Reading Workflow

1. Start with `action: "read"` - get plain text + statistics
2. Check `block_types` in response for Table, Image, Code, etc.
3. If structured content exists, use `action: "list_blocks"` for full data

## Configuration

```yaml
channels:
  feishu:
    tools:
      doc: true # default: true
```

**Note:** `feishu_wiki` depends on this tool - wiki page content is read/written via `feishu_doc`.

## Permissions

Required: `docx:document`, `docx:document:readonly`, `docx:document.block:convert`, `drive:drive`

---

## 鈿狅笍 閲嶈鎿嶄綔瑙勮寖锛堝繀椤婚伒瀹堬級

### 1. 鍒涘缓鏂囨。鍚庡繀椤诲鏍稿唴瀹?
**闂**锛歚create` 鍔ㄤ綔鍙垱寤烘爣棰橈紝涓嶄細鍐欏叆鍐呭銆?
**瑙勮寖**锛?```yaml
# 姝ラ 1锛氬垱寤烘枃妗ｏ紙鍙垱寤烘爣棰橈級
{ "action": "create", "title": "鏂囨。鏍囬", "owner_open_id": "ou_xxx" }

# 姝ラ 2锛氱珛鍗崇敤 write 鍐欏叆瀹屾暣鍐呭
{ "action": "write", "doc_token": "杩斿洖鐨?doc_token", "content": "# 瀹屾暣鍐呭..." }

# 姝ラ 3锛氬鏍搁獙璇侊紙鍙€変絾鎺ㄨ崘锛?{ "action": "read", "doc_token": "杩斿洖鐨?doc_token" }
# 妫€鏌ヨ繑鍥炵殑 content 鏄惁涓虹┖锛屽鏋滀负绌鸿鏄?write 澶辫触
```

**绂佹琛屼负**锛?- 鉂?鍙皟鐢?`create` 涓嶈皟鐢?`write`
- 鉂?璋冪敤 `create` 鍚庡亣璁惧唴瀹瑰凡鍐欏叆
- 鉂?涓嶈皟鐢?`read` 澶嶆牳灏辩洿鎺ュ憡璇夌敤鎴锋枃妗ｅ凡鍒涘缓

**姝ｇ‘娴佺▼**锛?1. `create` 鈫?鑾峰彇 `doc_token`
2. `write` 鈫?鍐欏叆瀹屾暣鍐呭
3. `read` 鈫?楠岃瘉鍐呭闈炵┖锛堝彲閫夛紝浣嗘帹鑽愮敤浜庨噸瑕佹枃妗ｏ級
4. 鍚戠敤鎴疯繑鍥炴枃妗ｉ摼鎺?
### 2. Skill 杩愯鏃舵満

**鑷姩婵€娲绘潯浠?*锛堟弧瓒充换涓€鍗宠繍琛岋級锛?- 鐢ㄦ埛鎻愬埌 "椋炰功鏂囨。"銆?椋炰功 doc"銆?docx"銆?浜戞枃妗?
- 鐢ㄦ埛鎻愪緵椋炰功鏂囨。閾炬帴锛坄feishu.cn/docx/`锛?- 鐢ㄦ埛瑕佹眰鍒涘缓/缂栬緫/璇诲彇椋炰功鏂囨。
- 鐢ㄦ埛鎻愬埌 "鐭ヨ瘑搴?銆?wiki"锛堜細鑱斿姩 `feishu-wiki` skill锛?
**琚姩婵€娲?*锛?- 鐢ㄦ埛鏄庣‘鎸囦护濡?"鍒涘缓椋炰功鏂囨。"銆?鏇存柊杩欎釜鏂囨。"

### 3. 閿欒澶勭悊

濡傛灉 `write` 澶辫触锛?1. 閲嶈瘯涓€娆★紙缃戠粶闂鍙兘锛?2. 濡傛灉浠嶅け璐ワ紝鍚戠敤鎴锋姤鍛婇敊璇?3. 涓嶈鍋囪鎴愬姛

濡傛灉 `read` 澶嶆牳鍙戠幇鍐呭涓虹┖锛?1. 绔嬪嵆閲嶆柊 `write`
2. 鍐嶆 `read` 楠岃瘉
3. 濡傛灉浠嶄负绌猴紝鍚戠敤鎴锋姤鍛婇棶棰?
---

## 鏈€浣冲疄璺电ず渚?
### 鍒涘缓瀹屾暣鏂囨。锛堟爣鍑嗘祦绋嬶級

```yaml
# 1. 鍒涘缓
feishu_doc:
  action: create
  title: GitHub 鏁欑▼
  owner_open_id: ou_xxx  # 浠?inbound metadata 鑾峰彇

# 2. 鍐欏叆鍐呭锛堢珛鍗虫墽琛岋紝涓嶈闂撮殧锛?feishu_doc:
  action: write
  doc_token: XOjRdDuIAoFz6nxpyNKcGyNHnxg  # 涓婁竴姝ヨ繑鍥?  content: |
    # 鏍囬
    
    瀹屾暣鍐呭...

# 3. 澶嶆牳锛堥噸瑕佹枃妗ｆ帹鑽愶級
feishu_doc:
  action: read
  doc_token: XOjRdDuIAoFz6nxpyNKcGyNHnxg

# 4. 杩斿洖缁撴灉缁欑敤鎴?鏂囨。宸插垱寤猴細https://feishu.cn/docx/XOjRdDuIAoFz6nxpyNKcGyNHnxg
```

### 鏇存柊鐜版湁鏂囨。

```yaml
# 1. 璇诲彇鐜版湁鍐呭
feishu_doc:
  action: read
  doc_token: ABC123def

# 2. 鍐欏叆鏇存柊鍚庣殑鍐呭
feishu_doc:
  action: write
  doc_token: ABC123def
  content: |
    # 鏇存柊鍚庣殑瀹屾暣鍐呭
```
