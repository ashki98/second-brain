# Second Brain — Schema & Operating Guide

This file tells Claude Code how to operate this knowledge base. Read it at the start of every session.

---

## What This Is

A personal second brain for Ashik — a software engineer at WellnessLiving with deep Python/Node.js backend experience and growing AI/distributed-systems knowledge. The vault is managed in Obsidian.

Two workflows:
1. **Ingest** — Ashik adds a raw note to `SDE Concepts/`; Claude integrates it into `trails.html`
2. **Query** — Ashik asks a question about his notes; Claude answers it and saves the explanation as a new note + trail node

---

## Directory Structure

```
second_brain3/
├── CLAUDE.md                   ← this file (operating guide)
├── raw/
│   └── vault/
│       ├── SDE Concepts/       ← all notes (Ashik writes; Claude reads & writes)
│       │   ├── Python/
│       │   ├── NodeJS/
│       │   ├── Networking/
│       │   └── ...             (15+ subfolders)
│       ├── Work/               ← work-specific notes (excluded from trails)
│       ├── attachments/        ← images referenced by notes
│       └── trails/
│           └── trails.html     ← THE ONLY FILE THAT MATTERS HERE
└── misc_scripts/               ← one-off migration scripts (do not re-run)
```

**Important:** `trails.html` is the sole source of truth. All other files in `trails/`
(old `.md` trail files, `index.md`, `log.md`) are superseded — ignore them entirely.

---

## trails.html Architecture

The file has two kinds of `<template>` elements and a JS rendering engine.
**No JSON block. No markdown conversion. Content is written as HTML directly.**

### 1. Trail definition templates

```html
<template id="trail-02" data-type="trail" data-order="2"
  data-title="Python Runtime &amp; Concurrency"
  data-objective="Understand how Python actually executes your code..."
  data-prereqs=""
  data-color="#3572A5"
  data-review-time="~50 min"></template>
```

Attributes: `id` (stable), `data-order` (sidebar position), `data-title`, `data-objective`,
`data-prereqs` (comma-separated trail ids, or empty), `data-color` (hex), `data-review-time`.

### 2. Node content templates

```html
<template id="node-02-01" data-type="node"
  data-trail="trail-02"
  data-order="1"
  data-title="OS Scheduling, Processes and Threads"
  data-file="Python/OS Scheduling, Process and Threads.md"
  data-bridge="The OS gives you processes and threads at the hardware level. Now zoom into Python's runtime...">

  <h3>Section Heading</h3>
  <pre>
  ASCII diagram here
  </pre>
  <ul>
    <li><strong>key term</strong>: explanation</li>
  </ul>
  <table>
    <thead><tr><th>Col A</th><th>Col B</th></tr></thead>
    <tbody><tr><td>val1</td><td>val2</td></tr></tbody>
  </table>
</template>
```

Attributes: `id` (node-XX-YY), `data-type="node"`, `data-trail` (parent trail id),
`data-order` (integer), `data-title`, `data-file` (path relative to `SDE Concepts/`),
`data-bridge` (transition sentence to next node; omit or leave empty on last node).

Content is **HTML written directly** — no markdown, no conversion step.

### 3. JS rendering engine

Scans all `<template>` elements on load, builds trail/node data, renders sidebar and trail
view. `getContent(node)` reads `node.el.innerHTML` directly — no parsing, no conversion.

---

## Content Format (HTML)

Write HTML directly in node templates. Keep it readable for an LLM.

### Supported elements

| Element | Use for |
|---|---|
| `<h3>Section Name</h3>` | Major section heading |
| `<h4>Sub-heading</h4>` | Sub-section |
| `<ul><li>...</li></ul>` | Bullet list of key points |
| `<ol><li>...</li></ol>` | Numbered steps |
| `<pre>...</pre>` | ASCII diagrams, code, multi-line examples |
| `<table>` with `<thead>/<tbody>` | Comparison tables, reference data |
| `<strong>term</strong>` | Key term on first use |
| `<code>snippet</code>` | Inline code |
| `<p>...</p>` | Prose paragraphs |

### Layout helpers (CSS classes)

```html
<!-- Side-by-side comparison -->
<div class="compare">
  <div><h4>Before</h4><pre>old code</pre></div>
  <div><h4>After</h4><pre>new code</pre></div>
</div>

<!-- Callout boxes -->
<div class="info">Blue info callout — key insight or context</div>
<div class="warn">Amber warning — common mistake or gotcha</div>
<div class="good">Green success — correct pattern or best practice</div>
```

### HTML escaping in templates

Use HTML entities for special characters in template content:
- `<` → `&lt;`  |  `>` → `&gt;`  |  `&` → `&amp;`
- Or avoid them: use `≤`, `≥`, `→` (Unicode) instead of `<`, `>`, `->`

---

## Ingest Workflow

**Trigger:** Ashik says `"ingest SDE Concepts/path/to/note.md"`

### Step 1 — Read the note fully

Read every concept, comparison, analogy, and diagram. Identify:
- What the note teaches
- What a reader needs to know before it (prerequisites)
- What naturally comes after

### Step 2 — Decide placement

Read `SDE Concepts/index.md` to understand what notes already exist and which trail(s)
the topic belongs to. Then scan `trails.html` node templates to find the right position.

Ask: which trail does this note belong to? Which position in the node sequence is most
natural? A note may fit in **multiple trails**. It may warrant a **new trail** if it
starts a conceptual cluster with no good home.

### Step 3 — Update the trail template (if new trail)

If creating a new trail, add a `<template data-type="trail">` element in the trail
definitions section, following the existing pattern.

### Step 4 — Add/update the node template

**For a new last node** (most common):
1. Find the current last node template for that trail
2. Set `data-bridge` on it (it previously had no bridge or an empty one)
3. Add the new `<template data-type="node">` before `</body>` with `data-bridge` empty

**For a mid-trail insert** (less common):
1. Update `data-order` on affected nodes (shift down by 1)
2. Update `data-bridge` on the preceding node to point to the new node's concept
3. Set `data-bridge` on the new node to point to the next node
4. Add the new template

### Step 5 — Write the node content (HTML)

In the new template's innerHTML, cover **every concept from the source note**:
- Every named mechanism, comparison, analogy, and rule
- Restructure and reword freely — but omit nothing
- Lead with the most important visual
- At least one `<pre>` diagram OR `<div class="compare">` OR `<table>` per node
- End with practical implications or "when to use"

### Step 6 — Report to Ashik

```
## What changed

**Trail NN — Trail Name**
Added [Note Title] at position N, between [Prev Node] and [Next Node].

↑ [Prev Node] covers [one phrase].
→ [New Note] adds [one phrase].
↓ [Next Node] then [one phrase].
```

If multiple trails were touched, repeat the block for each.

---

## Query Workflow

**Trigger:** Ashik asks a question about a concept, requests clarification, or wants
something from his notes explained in more depth.

Examples:
- "Explain how consistent hashing works from my notes"
- "I don't understand how the GIL interacts with asyncio — explain it"
- "What does my note on CQRS say about snapshots, and why do they exist?"

### Step 1 — Find the relevant notes using the index

Read `SDE Concepts/index.md` first. It maps topics → file paths so you don't have to
scan the whole folder. Then read the identified source notes for full detail.

### Step 2 — Answer the question

Explain the concept clearly:
- Ground the explanation in Ashik's own notes (cite the source concept)
- Add any **bridging knowledge** needed to fully understand it (concepts not in his notes
  but required as context)
- Use examples, diagrams, or analogies where they help
- Keep it tailored to his level (senior SDE, Python/Node.js background)

### Step 3 — Decide: append to existing note OR create new note

**Append to existing note when:**
- The question is a clarification or deeper dive on a topic already in a note
- The answer adds a new section, analogy, or worked example to an existing concept
- Example: "Why does the GIL release during I/O?" → append a new section to
  `Python/GIL, Threads and Processes.md`

**Create a new note when:**
- The question covers a genuinely new topic not in any existing note
- The explanation is long enough to stand alone
- The bridging knowledge introduces a concept that's broadly reusable
- Example: "Explain how epoll works under the hood" → new note
  `Redis/epoll and IO Multiplexing Internals.md`

**Note format** (same style as Ashik's existing notes — concise, bullet-heavy):
```markdown
# [Topic Title]

[1–2 sentence summary]

## [Section]

- bullet point
- bullet point with **key term** in bold
```

Write as if Ashik wrote it — his voice, his phrasing patterns. Not Claude's voice.

### Step 4 — Ingest into trails.html

Whether appending or creating new:
- If **appended**: update the existing node's template in `trails.html` to include the
  new content (the node content should always reflect the full current state of the note)
- If **new note**: follow the Ingest Workflow (Steps 2–6) to add it as a new trail node

Also update `SDE Concepts/index.md` to include the new note or new topics.

### Step 5 — Report

```
## Answer

[The explanation]

---
## What was saved

[Appended to: SDE Concepts/path/note.md]  OR  [Created: SDE Concepts/path/note.md]
Trail node updated: Trail NN — position N
```

---

## Content Standards (applies to both workflows)

### Visual representations (mandatory)

Ashik learns better with visuals. Every node must have at least one:
- `<pre>` ASCII architecture diagram or flow diagram
- `<div class="compare">` side-by-side before/after
- `<table>` comparison or reference table

Use Unicode box-drawing for diagrams: `─ │ ┌ ┐ └ ┘ ├ ┤ ┬ ┴ → ← ↓ ↑ ▼ ▲ ►`

### Completeness

Every concept, mechanism, comparison, analogy, and table from the source note must appear
in the trail node. Nothing dropped. Restructuring allowed; omission is not.

### Bridge notes

Bridge notes explain **why** one concept precedes the next — not just what the next
concept is. They should name a specific mechanism from the preceding node and connect it
to the entry point of the following node.

---

## Current Trail Inventory (12 trails, 67 nodes)

| ID | Trail | Nodes | Prereqs |
|---|---|---|---|
| trail-01 | Data Structures & Hashing | 5 | none |
| trail-02 | Python Runtime & Concurrency | 10 | none |
| trail-03 | Node.js Runtime | 3 | trail-02 |
| trail-04 | Networking | 4 | none |
| trail-05 | Backend Architecture | 8 | trail-04 |
| trail-06 | Databases & Storage | 9 | trail-05 |
| trail-07 | Scaling & Distributed Systems | 7 | trail-06, trail-04 |
| trail-08 | System Design | 4 | trail-07 |
| trail-09 | DevOps & Infrastructure | 4 | trail-05 |
| trail-10 | Frontend | 4 | trail-04 |
| trail-11 | Security | 3 | trail-04, trail-05 |
| trail-12 | AI & Modern Stack | 6 | trail-05 |

Node IDs: `node-{trailNum}-{position}` (e.g. `node-02-03`).
When inserting mid-trail, keep existing IDs stable — add new node at end of sequence
and adjust `data-order` values. Do not renumber IDs.

---

## Rules

1. **`trails.html` is the only file to edit** for trail changes. Ignore all other `trails/` files.
2. **`SDE Concepts/` is where notes live.** For the Query workflow, Claude writes new notes here.
3. **Never modify Ashik's existing notes** without being asked.
4. **Content is HTML, not markdown.** No `md()`, no `mdFull()`, no conversion. Write `<h3>`, `<ul>`, `<pre>`, `<table>` directly.
5. **Visuals are mandatory** — at least one diagram or table per node.
6. **No concept left behind** — every point in the source note must appear in the trail node.
7. **Bridge notes are connective**, not descriptive — explain the conceptual link, not just the topic.
8. **When in doubt about placement**, prefer an existing trail over creating a new one.
9. **Work notes** (`Work/` folder) and personal notes are excluded from trails.
