# epub-read

English-first README. A Simplified Chinese guide is included in the `简体中文` section below.

`epub-read` is a Feishu CLI-based EPUB reading and analysis skill for AstronClaw, ClawHub, and GitHub distribution. Its main goal is to pull `.epub` files from Lark Drive, process them safely through explicit task modes, and publish the resulting notes or extraction outputs back into Lark docs.

It supports overview, targeted reading, continuous chunked reading, structured extraction, complex-content inspection, and batch processing.

This repository is the standalone Feishu CLI contest edition. The original general-purpose repository is kept separately and is not overwritten by this release.

## Start Here

- Download EPUB files from Lark Drive with Feishu CLI
- Parse them into structured outputs you can inspect and reuse
- Read safely by overview, targeted, full-read, extract, or complex-content mode
- Publish notes and extraction results back to Feishu Docs

## First Run

1. Confirm Feishu CLI access:

```bash
lark-cli auth status
```

2. Download an EPUB from Lark Drive:

```bash
lark-cli drive +download --as bot --file-token boxbc_xxx --output ./book.epub
```

3. Parse the book:

```bash
python3 parse_epub.py ./book.epub
```

4. Generate a safe reading plan:

```bash
python3 task_router.py /path/to/book_dir --mode overview
```

5. Publish the result:

```bash
lark-cli docs +create --as bot --title "EPUB Reading Result" --markdown "<result-markdown>"
```

## Why This Skill Is Different

`epub-read` does not load a long ebook into context by default. Instead, it:

- routes work through explicit task modes
- supports chunked long-book reading
- keeps reading state for continue and jump flows
- supports extraction and complex-content inspection
- publishes output back to Feishu Docs

## Highlights

- Feishu CLI intake: download `.epub` files from Lark Drive with `lark-cli drive +download`
- Task-mode routing: six explicit modes for different reading goals
- Safer long-book handling: chunk-first workflows for full-book reading
- Persistent reading state: supports continue, previous, next, and chapter jumps
- Rich output artifacts: metadata, TOC, book-level markdown, chapter files, chunk files, reading index, and complex-content report
- Lark docs publishing: create or update reading outputs with `lark-cli docs +create` and `lark-cli docs +update`
- Backward-compatible outputs: keeps existing parse and chunk outputs usable for older workflows

## Supported Modes

### A. overview

Read metadata, structure, and table of contents without loading the full body text.

### B. targeted_read

Read by chapter, chapter ID, chunk range, or keyword match.

### C. full_read

Read a long book sequentially in chunks while maintaining reading progress.

### D. extract

Extract structured information such as keywords, definitions, quotes, examples, action items, names, locations, organizations, tables, and lists.

### E. complex_content

Inspect images, tables, SVG content, and low-text / high-resource sections.

### F. batch

Process multiple EPUB files from one or more directories.

## Typical Use Cases

- Download an EPUB from Feishu Drive and build an overview
- Get a quick structural overview of an ebook
- Read only selected chapters or chunk ranges
- Safely continue through a long book without loading the full text
- Extract concepts, definitions, quotes, or action items from a parsed EPUB
- Review image-heavy or table-heavy sections
- Batch-scan a library of EPUB files
- Publish reading notes into a Feishu doc for teammates

## Feishu CLI Workflow

1. Download the EPUB from Lark Drive:

```bash
lark-cli drive +download --file-token boxbc_xxx --output ./book.epub
```

2. Parse and analyze it with the local EPUB scripts.

3. Publish the final overview or extraction to Feishu:

```bash
lark-cli docs +create --title "EPUB Reading Result" --markdown "<result-markdown>"
```

## Dependencies

- `lark-cli`
- `python3`
- `beautifulsoup4`
- `lxml`

Before using the Feishu flow, make sure `lark-cli` is already configured with the required Lark Drive and Lark Docs permissions.

If you need personal-resource access instead of bot access, run:

```bash
lark-cli auth login --domain docs --domain drive
```

Install dependencies:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Quick Start

### 1. Download an EPUB from Lark Drive

```bash
lark-cli drive +download --file-token boxbc_xxx --output ./book.epub
```

### 2. Parse the EPUB

```bash
python3 parse_epub.py ./book.epub
```

### 3. Generate an overview plan

```bash
python3 task_router.py /path/to/book_dir --mode overview
```

### 4. Chunk a long book

```bash
python3 chunk_book.py /path/to/book_dir --mode balanced
```

### 5. Start or continue full reading

```bash
python3 task_router.py /path/to/book_dir --mode full_read --start-chunk 1
python3 task_router.py /path/to/book_dir --mode full_read --continue
```

### 6. Run structured extraction

```bash
python3 task_router.py /path/to/book_dir --mode extract --extraction-types definitions quotes action_items
```

### 7. Publish the result back to Feishu

```bash
lark-cli docs +create --title "EPUB Reading Result" --markdown "<result-markdown>"
```

Or append to an existing doc:

```bash
lark-cli docs +update --doc "<doc_id_or_url>" --mode append --markdown "<result-markdown>"
```

## Output Artifacts

Parsing produces a directory similar to:

```text
.epub_read_output/<book_id>/
├── metadata.json
├── toc.json
├── book.json
├── book.md
├── manifest.json
├── complex_content.json
├── reading_index.json
├── session_state.json
├── chapters/
└── chunks/
```

Key files:

- `reading_index.json`: chunk ranges, word counts, images, and table information per chapter
- `session_state.json`: current mode, current chapter, current chunk, last action, last query
- `complex_content.json`: images, tables, SVGs, and sections that may need manual review

## Validation

Use the lightweight integration test first:

```bash
python3 test_integration.py --epub /absolute/path/to/test.epub
python3 test_integration.py --epub /absolute/path/to/test.epub --full
```

This contest edition has also been smoke-tested with real Feishu CLI Drive upload/download flows before publication.

## Repository Layout

```text
epub-read/
├── SKILL.md
├── README.md
├── LICENSE.md
├── skill.json
├── parse_epub.py
├── chunk_book.py
├── task_router.py
├── update_session_state.py
├── test_integration.py
├── requirements.txt
├── templates/
├── examples/
└── agents/
```

## ClawHub Notes

This repository is the English-source version intended for GitHub submission and future ClawHub packaging. Keep `SKILL.md`, `README.md`, metadata, and templates aligned with this repository so the public GitHub source stays consistent with the published skill.

## License

This project is released under the `MIT` license for public distribution and reuse.

---

## 简体中文

`epub-read` 是一个基于飞书 CLI 的 EPUB 阅读与分析 skill。它的核心思路是先从飞书云盘下载 `.epub` 文件，再通过明确的任务模式进行解析、分块阅读、结构化抽取或复杂内容检查，最后可以把阅读结果发布回飞书文档。

### 核心亮点

- 通过 `lark-cli drive +download` 从飞书云盘拉取 EPUB
- 用六种任务模式管理不同阅读目标
- 长书默认走分块路线，避免一次性把整本书塞进上下文
- 支持继续阅读、上一块、下一块、跳转章节等会话状态
- 可用 `lark-cli docs +create` / `docs +update` 把结果回写到飞书文档

### 飞书 CLI 工作流

1. 下载飞书云盘中的 EPUB
2. 运行 `parse_epub.py`、`task_router.py`、`chunk_book.py` 等本地脚本
3. 产出概览、定向阅读结果、抽取结果或复杂内容报告
4. 将结果发布到新的或已有的飞书文档

### 前置条件

- 已安装并配置 `lark-cli`
- 已具备飞书 Drive / Docs 所需权限
- 已安装 `python3`
- 已安装 `beautifulsoup4` 与 `lxml`

### 快速开始

下载 EPUB：

```bash
lark-cli drive +download --file-token boxbc_xxx --output ./book.epub
```

解析：

```bash
python3 parse_epub.py ./book.epub
```

生成概览计划：

```bash
python3 task_router.py /path/to/book_dir --mode overview
```

发布结果：

```bash
lark-cli docs +create --title "EPUB Reading Result" --markdown "<result-markdown>"
```

### 适用场景

- 快速查看电子书目录、结构和主题
- 按章节或 chunk 范围做定向阅读
- 对长书进行连续、安全的分块精读
- 抽取概念、定义、金句、例子、行动项
- 将阅读结果沉淀到飞书文档中供团队复用
