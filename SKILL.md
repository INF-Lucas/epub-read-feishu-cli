---
name: "epub-read"
slug: "epub-read"
version: "2.0.0"
description: "Feishu CLI-based EPUB reading and analysis skill that downloads books from Lark Drive, parses them safely, and can publish reading outputs to Lark docs."
changelog: "Feishu CLI release for GitHub submission with Lark Drive intake and Lark Docs publishing."
metadata: {"clawdbot":{"emoji":"📚","os":["linux","darwin","win32"],"requires":{"bins":["lark-cli","python3"]}}}
---

<objective>
Provide a strict, auditable EPUB workflow that safely handles long books through explicit task routing instead of loading full-book text by default.
提供一套严格、可审计的 EPUB 工作流，通过明确的任务模式安全处理长书，而不是默认一次性加载整本正文。
</objective>

<use_when>
- The user mentions an `.epub` file or ebook / 用户提到 `.epub` 文件或电子书
- The ebook is stored in Lark Drive / 电子书位于飞书云盘
- The user wants a quick structural overview / 用户需要快速结构概览
- The user wants chapter-specific or chunk-specific reading / 用户希望按章节或 chunk 定向阅读
- The user wants full-book sequential reading with chunking / 用户希望做整本书的连续分块阅读
- The user wants structured extraction / 用户需要结构化抽取
- The user wants to inspect images, tables, or other complex content / 用户想检查图片、表格或其他复杂内容
- The user wants to batch-process multiple EPUB files / 用户希望批量处理多个 EPUB
- The user wants the resulting notes published into Lark docs / 用户希望将结果发布到飞书文档
</use_when>

<process>

**Feishu CLI intake and publish flow / 飞书 CLI 输入与发布流程**

Prefer the Feishu route whenever the book lives in Feishu:

如果书籍位于飞书内，优先走飞书 CLI 路线。

Download an EPUB from Lark Drive:

从飞书云盘下载 EPUB：

```bash
lark-cli drive +download --file-token boxbc_xxx --output ./book.epub
```

Publish the final overview or extraction to a Lark doc:

将最终概览或抽取结果发布到飞书文档：

```bash
lark-cli docs +create --title "EPUB Reading Result" --markdown "<result-markdown>"
```

Append to an existing doc:

追加到已有文档：

```bash
lark-cli docs +update --doc "<doc_id_or_url>" --mode append --markdown "<result-markdown>"
```

**STEP 0 - Choose exactly one task mode before doing anything else / 步骤 0 - 先确定任务模式**

| Mode | Purpose | Use when |
|------|---------|----------|
| `overview` | Fast structural overview | Metadata, TOC, themes, structure only |
| `targeted_read` | Focused reading | Specific chapters, chunk ranges, or keyword hits |
| `full_read` | Sequential reading | Long-book chunked reading with saved progress |
| `extract` | Structured extraction | Keywords, definitions, quotes, examples, action items, entities, tables, lists |
| `complex_content` | Complex-layout inspection | Images, tables, SVG, low-text sections |
| `batch` | Multi-book planning | Multiple EPUB files or folders |

Default to `overview` or `targeted_read` when the user intent is ambiguous. Never load a long book's full body text by default.

如果用户意图不够明确，默认优先选择 `overview` 或 `targeted_read`，不要默认加载整本长书正文。

**STEP 1 - Parse if needed / 步骤 1 - 必要时先解析**

1. Check whether the output directory already exists and contains `manifest.json`.
2. If not, run `parse_epub.py`.
3. After parsing, report:
   - title
   - author
   - chapter count
   - chunk count if available
   - image count
   - table count
   - output directory

**STEP 2 - Build an execution plan / 步骤 2 - 构建执行计划**

Use `task_router.py` to decide whether parsing, chunking, or state updates are required:

```bash
python3 task_router.py <book_dir> --mode <mode> [params...]
```

The plan should tell you:
- whether parsing is required
- whether chunking is required
- which files are recommended to read
- whether session state must be updated

**STEP 3 - Mode-specific behavior / 步骤 3 - 按模式执行**

### overview

- Read only metadata, TOC, reading index, and other structural outputs
- Do not load the whole book body by default
- Return:
  - title
  - author
  - chapter count
  - TOC structure
  - theme overview
  - suggested next actions

### targeted_read

Support:

- `--chapter`
- `--chapter-id`
- `--chunk-start`
- `--chunk-end`
- `--keyword`

Return:

- the requested section
- short context
- concise summary

### full_read

- Prefer chunk-based reading for long books
- Support continue, previous, next, and jump flows
- Always update `session_state.json`
- Never pretend progress exists if the session state is missing

### extract

Support extracting:

- keywords
- definitions
- quotes
- examples
- action_items
- names
- locations
- organizations
- tables
- lists

Return a hit list with chapter references and short context.

### complex_content

Inspect:

- images
- SVG
- tables
- image-heavy sections
- low-text / high-resource sections

Return a structured report. OCR is not required by default.

### batch

Support:

- multiple EPUB file paths
- directory scanning
- batch planning
- batch extraction requests

Return success / failure counts and a concise overview.

**STEP 4 - Long-book safety rules / 步骤 4 - 长书安全规则**

- Never push the full body of a long book into context at once
- Prefer `chunks/` over chapter markdown for full sequential reading
- When chunking is required, run `chunk_book.py` first
- Use `reading_index.json` to map chapters to chunk ranges

**STEP 5 - State management rules / 步骤 5 - 状态管理规则**

When running `full_read` or any progress-sensitive flow:

1. Read `session_state.json` first
2. Update it after every progress-changing action
3. Respect existing saved progress unless the user explicitly asks to restart

**STEP 6 - Output style / 步骤 6 - 输出风格**

Be explicit about:

- what files were used
- what mode was selected
- why a long book was chunked instead of loaded fully
- what the user can do next

When possible, point the user toward the safest next step:

- continue reading
- jump to a chapter
- inspect a chunk range
- extract a structure
- review complex content

</process>

<validation>

Before considering the task complete, check:

- parsing outputs exist
- chunk files exist when required
- reading index and session state are coherent
- extraction targets match the requested type
- complex-content reports are generated from real parsed outputs

</validation>
