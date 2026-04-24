# Ingest Workflow

Ingest new materials into the knowledge base. Runs fully automatically by default; only asks the user when human judgment is needed.

> 📌 **Path convention**: `raw/` and `wiki/` in this document refer to the absolute paths of the current knowledge base, determined when SKILL.md routes the command (`KB_RAW`, `KB_WIKI`). `KB_LANG` is the language setting for the current knowledge base.

## Core Principles

1. **Default automation** — Routine ingestion runs the full workflow without interrupting the user
2. **Ask only when needed** — Only prompt the user when encountering contradictions, ambiguities, or boundary judgments
3. **Structured storage** — Pages are strictly placed in the correct directory by type; cross-references form a complete graph

## Workflow

### 1. Discover New Materials

Read `wiki/index.md` to get the list of already-ingested materials. Scan the `raw/` directory (excluding the `assets/` subdirectory) to find files that have not yet been ingested.

If the user specified a filename, process only that file.

### 2. Read Materials

Read each new material:
- Markdown files: read directly
- PDF files: use the Read tool
- Image files: use the Read tool

#### Image Handling

The LLM cannot process Markdown text and embedded images in a single read. For materials containing images, use a two-step approach:

1. **Read text first**: read the Markdown/PDF to understand the main content
2. **Then view images**: scan image paths referenced in the text (`![](path)` or `![[path]]`), view each image with the Read tool, and extract supplementary information (chart data, architecture diagrams, flowcharts, etc.)

Key information from images (data points, structures, flows) should be integrated as text descriptions in wiki pages — don't rely solely on images to convey information. If an image itself is important (e.g. an architecture diagram or experimental results), reference it in the wiki page using standard Markdown image syntax: `![description](../../raw/assets/image-name)`.

### 3. Analyze and Decide

After reading the material, analyze:
- Core takeaways (3–5 items)
- Entity and concept pages to create or update
- Connections to existing knowledge
- Whether any contradictions exist

**Based on the analysis, decide whether to ask the user:**

#### Auto-execute (no prompting) — all of the following must be true:
- No contradictions: new material does not conflict with existing wiki content
- Clear classification: the type (entity vs concept) and directory for each item is unambiguous
- No merge ambiguity: no "might be the same as an existing page, but unsure" situations
- Reasonable scale: ≤ 5 new pages planned

When auto-executing, inform the user with a brief message before creating pages (non-blocking):
```
Ingesting "Material Title" — planning to create N new pages and update M existing pages, running automatically...
```

#### Situations requiring a prompt — any one of the following:

**Scenario A: Contradiction** — new material conflicts with existing content
```json
{
  "questions": [{
    "question": "\"Material Title\" contradicts existing content:\n\n· Existing: <existing page's statement>\n· New material: <new material's statement>\n\nHow should this be handled?",
    "header": "Content Contradiction",
    "multiSelect": false,
    "options": [
      {"label": "Mark contradiction and keep both", "description": "Use a ⚠️ Contradiction tag to preserve both versions (recommended)"},
      {"label": "Use the new material", "description": "Update existing content to match the new material"},
      {"label": "Keep old content", "description": "Ignore the contradicting part of the new material"}
    ]
  }]
}
```

**Scenario B: Merge ambiguity** — uncertain whether a new concept/entity should be merged with an existing page
```json
{
  "questions": [{
    "question": "The new material mentions \"XX\", which may be the same concept as the existing page [[yy]].\n\n· XX: <new material's description>\n· yy: <existing page's description>\n\nShould they be merged?",
    "header": "Page Merge",
    "multiSelect": false,
    "options": [
      {"label": "Merge into existing page", "description": "Integrate the new information into [[yy]]"},
      {"label": "Create separate page", "description": "XX and yy are different concepts — maintain separately"},
      {"label": "Merge and rename", "description": "Merge the content and use a more accurate name"}
    ]
  }]
}
```

**Scenario C: Large-scale ingestion** — planning to create > 5 new pages
```json
{
  "questions": [{
    "question": "\"Material Title\" is content-rich; many pages are planned:\n\nNew: <list>\nUpdates: <list>\n\nSelect ingestion scope:",
    "header": "Ingestion Scope",
    "multiSelect": false,
    "options": [
      {"label": "Ingest all (recommended)", "description": "Create all pages listed above"},
      {"label": "Core pages only", "description": "Create only the 3–5 most important pages"},
      {"label": "Summary page only", "description": "Create only the source page; defer entity and concept splitting"}
    ]
  }]
}
```

**Multiple scenarios can be combined**: if there are both contradictions and merge ambiguities, present multiple questions in a single AskUserQuestion (up to 4).

### 4. Create Source Summary Page

Create a summary page under `wiki/sources/`, with a filename matching the original file.

Template:
```markdown
---
title: Material Title
aliases: [Localized alias]
type: source
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [relevant tags]
raw_file: raw/original-filename
---

# Material Title

## Source Info

- **Original file**: [raw/filename](../../raw/filename)
- **Type**: Article / Paper / Report / ...
- **Date**: Publication date (if available)
- **Author**: Author name (if available)

## Core Content

[3–5 paragraphs summarizing key content]

## Key Takeaways

- Takeaway 1
- Takeaway 2
- ...

## Quotes and Data

[Important data points, quotes, statistics]

## Related

- [[related-page]]
```

**Note**: Use standard Markdown links `[text](path)` for raw file links — do not use `[[wikilink]]` to avoid ghost nodes in the Obsidian graph.

### 5. Create/Update Entity and Concept Pages

#### Directory Structure

Strictly store by type to keep the graph and directory structure clean:

| Type | Directory | Criteria | Examples |
|------|-----------|----------|---------|
| Entity | `wiki/entities/` | Concrete "things" with specific names: people, organizations, tools, projects, products, datasets | vannevar-bush, forge-benchmark, openai |
| Concept | `wiki/concepts/` | Abstract ideas, methods, techniques, patterns, theories | memex, knowledge-management, reinforcement-learning |

**Decision principle**: if it can be referred to by a proper noun, it's an entity; if you need to explain "what it is", it's a concept. When in doubt, lean toward concept.

#### Create/Update Rules

**Creating new pages**:
- Only create a page for entities/concepts that are discussed substantially (not just mentioned in passing)
- Items mentioned only 1–2 times without being core content don't need their own page — briefly mention them in a related page

**Updating existing pages**:
- Append new information in the relevant section; do not overwrite existing content
- Update the `sources` list and `updated` date in frontmatter
- Add new related links to the Related section

Template:
```markdown
---
title: Name
aliases: [Localized alias]
type: entity or concept
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [tags]
sources: [referenced source filenames]
---

# Name

[Overview paragraph]

## Details

[Detailed content organized by topic]

## Related

- [[related-page]] — relationship description
```

### 6. Maintain Graph Integrity

This is the most critical step — ensures the knowledge graph is well-connected:

**Bidirectional link rules**:
- If A's Related links to B, then B's Related **must** link back to A
- Every entity/concept page's Related **must** include a back-link to the corresponding source summary page
- After creating new pages, scan all **existing pages** — if an existing page discusses the topic of a new page but doesn't link to it, add the link

**Cross-reference rules**:
- Use `[[wikilink]]` on the first mention of another wiki page's topic in the body text
- The Related section lists all related pages with a brief relationship description (e.g. `— proposed by`, `— source material`)

### 7. Update Index and Log

- Add new entries under the appropriate category in `wiki/index.md`
- Update statistics and recent activity in `wiki/overview.md`
- Append an operation record to the top of `wiki/log.md` (after the `# Wiki Log` heading)

### 8. Run Deterministic Check

After all pages are created/updated, run `python <skill-dir>/scripts/lint.py --wiki-dir <KB_WIKI> --raw-dir <KB_RAW>`. If the script reports P0 issues, fix them immediately before ending the ingest workflow.

Output a completion summary:
```
✅ Ingested "Material Title"
   Created: entities/xx.md, concepts/yy.md
   Updated: concepts/zz.md
   Lint: passed
```

## Notes

- **Language**: Use the `KB_LANG` setting (zh or en); write all new wiki pages in that language. Keep proper nouns in their original form. Regardless of the material's language, wiki pages are always written in the target language.
- **aliases**: Every page's frontmatter must include an `aliases` field. When language=zh, fill with Chinese aliases; when language=en, fill with English aliases. This is what the Obsidian Front Matter Title plugin uses to display titles.
- Raw materials (files under `raw/`) are read-only — never modify them
- If a material contains image references, record the image paths for separate viewing later
- Keep summaries objective and neutral; distinguish facts from opinions
- When new materials contradict existing content, use `> ⚠️ Contradiction` markers
