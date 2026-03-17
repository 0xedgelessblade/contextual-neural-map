---
name: contextual-neural-map
description: |
  通用 knowledge graph 視覺化工具。掃描任意資料夾，偵測檔案及其關聯，
  產出一份 Obsidian Graph View 風格的互動式 force-directed graph HTML 檔案。
  鉛筆手繪美學搭配紙張質感背景，由 D3.js 驅動。
  支援拖曳、縮放、hover、搜尋、圖層篩選。
  觸發語句："draw graph"、"knowledge graph"、"neural map"、"context map"、
  "relationship map"、"visualize connections"、"graph view"、"node map"、
  "draw backlinks"、"generate graph"、"map this folder"。
  也適用於使用者說「畫關係圖」、「知識圖譜」、「畫 graph」、「視覺化連結」、
  「像 Obsidian 的 graph」、「更新 graph」、「show me how they relate」。
  即使使用者只是說「graph」或「map it」，只要上下文跟視覺化知識/內容關聯有關，都應觸發。
  任何跟「將散落的知識節點視覺化為 force-directed graph」相關的請求都應觸發此技能。
---

# Contextual Neural Map

Produces an Obsidian Graph View–style interactive knowledge graph from any folder. Paper-white background, pencil-sketch hand-drawn feel, D3.js force-directed layout.

## Design Aesthetic

The core aesthetic is "pencil doodles on a notebook page":

- **Background**: cream paper `#f5f2ed`, with faint grid lines and subtle grain noise
- **Nodes**: SVG `feTurbulence` + `feDisplacementMap` filters for hand-drawn wobbly edges
- **Edges**: same sketch filter applied; dashed vs solid lines distinguish relationship types
- **Palette**: all grey/brown/cream tones (`#2c2825` ~ `#d5cfc6`), no saturated colors
- **Fonts**: Caveat (handwriting) for titles and tags, Inter for body text
- **Panels**: frosted glass `backdrop-filter: blur(16px)` + dashed dividers

## Node Types

| Type | Visual | Size Rule |
|------|--------|-----------|
| Document (document) | Solid dark circle `#4a4540`, soft shadow | Scaled by importance metric (e.g. word count, links-in), 4–20 px |
| Note (note) | Hollow dashed circle | Fixed 5.5 px |
| Draft (draft) | Semi-transparent filled circle | Fixed 5.5 px |
| Tag / Topic (tag) | Large semi-transparent circle, Caveat handwriting label | Fixed 10 px |

## Edge Types

| Type | Visual | Meaning |
|------|--------|---------|
| tag-link | Thin solid line `#cdc6bb`, 0.5 px | Node → Tag membership |
| content-link | Dashed line `#a09888`, 1.2 px | Direct reference or shared keyword between nodes |
| sequence-link | Thick dashed line `#8a8279`, 1.8 px | Sequential or hierarchical relationship |

## Workflow

### Step 1 — Identify the target folder

Ask the user which folder to visualize. Accept any path. Then recursively scan the folder to inventory all files.

### Step 2 — Analyze and extract data

Read the files and build three data structures:

**Tags / Topics**: Detect recurring themes across files. Strategies:
- Extract headings (H1–H3) from Markdown files
- Extract frontmatter tags/categories if present (YAML front matter)
- Identify frequently recurring keywords or phrases across multiple files
- Use folder names as high-level categories
- If the folder contains a manifest, index, or README, use its structure as a topic guide

**Nodes**: Each file or meaningful content unit becomes a node.
- For each node, capture: `id`, `label` (filename or title), `type` (document / note / draft), `tags` (which topic clusters it belongs to), `desc` (brief summary or first line), and an optional `weight` (word count, size, or other importance metric)
- Classify type by convention: files with substantial content → `document`; short notes or stubs → `note`; files with "draft" in name or WIP markers → `draft`

**Edges / Links**:
- Node → Tag membership = `tag-link`
- Explicit cross-references (wiki-links `[[...]]`, markdown links, or shared unique terms) = `content-link`
- Files in the same subfolder or numbered sequence = `sequence-link`

### Step 3 — Inject data into the HTML template

Read `assets/template.html` and replace the `// __DATA_INJECT__` … `// __DATA_END__` block with the extracted data.

Data format:

```javascript
const tags = [
  { id: 't_topicname', label: 'Topic Name' },
  // ...
];

const documents = [
  { id: 'd_filename', label: 'Document Title', date: 'YYYY-MM-DD', weight: 1200, type: 'report', tags: ['t_topicA', 't_topicB'], desc: 'Brief summary' },
  // ...
];

const notes = [
  { id: 'n_001', label: 'Note Title', status: 'note', tags: ['t_topicA'], desc: 'Description', related: ['n_002', 'd_filename'] },
  // ...
];

const sequenceGroups = [
  ['d_chapter1', 'd_chapter2', 'd_chapter3'],  // ordered series
  // ...
];
```

### Step 4 — Save and deliver

Save the completed HTML to the workspace root as `contextual-neural-map.html` (or a user-specified name), then provide a `computer://` link.

## Interactive Features

The template includes these interactions out of the box — no extra development needed:

- **Hover**: highlights connected nodes and edges, dims the rest, shows tooltip card
- **Click**: locks focus mode on a node; click again to release
- **Drag**: reposition any node
- **Scroll wheel**: zoom the canvas
- **Search**: top-right input box, live-filters matching nodes (also reveals connected tags)
- **Legend filter**: click legend items to toggle type visibility
- **Stats bar**: bottom bar showing node count, edge count, cluster count, and total weight

## Customization Guide

Common tweaks users may request:

| Need | Where to change |
|------|----------------|
| Background color | `body { background: #xxx }` |
| Node colors | `nodeEls` fill and stroke values |
| Fonts | `@import url(...)` and `font-family` |
| Node size formula | `Math.sqrt(d.weight / N)` — adjust N |
| New edge type | Add a new `type` to links data, add matching stroke/dasharray style |
| Force layout params | `simulation.force(...)` distance / strength / charge values |

## Applicable to Any Folder

This skill works with any folder that contains text-based files:

- **Project docs** (each doc = node, shared topics = edges)
- **Note-taking vaults** (each note = node, wiki-links = edges)
- **Research papers** (each paper = node, citations/keywords = edges)
- **Course materials** (each lesson = node, prerequisites = edges)
- **Codebases** (each module = node, imports/dependencies = edges)
- **Writing projects** (each chapter/draft = node, themes = edges)

As long as you can extract nodes and links into the JSON structure above, the template renders it beautifully.
