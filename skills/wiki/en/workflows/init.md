# Init Workflow

Create and register a new knowledge base.

## Workflow

### 1. Collect Information

Use AskUserQuestion to ask for the following:

```json
{
  "questions": [
    {
      "question": "Root directory path for the knowledge base? (raw/ and wiki/ subdirectories will be created here)",
      "header": "Path",
      "multiSelect": false,
      "options": [
        {"label": "Current directory", "description": "Use the current working directory as the knowledge base root"},
        {"label": "Custom path", "description": "Specify an absolute path"}
      ]
    },
    {
      "question": "Give the knowledge base a name (shown when switching between multiple bases)",
      "header": "Name",
      "multiSelect": false,
      "options": [
        {"label": "AI Research", "description": ""},
        {"label": "Reading Notes", "description": ""},
        {"label": "Work Knowledge Base", "description": ""}
      ]
    },
    {
      "question": "What language should wiki pages be written in?",
      "header": "Language",
      "multiSelect": false,
      "options": [
        {"label": "zh (Chinese)", "description": "Write wiki content in Chinese; keep proper nouns in English"},
        {"label": "en (English)", "description": "Write all wiki content in English"}
      ]
    }
  ]
}
```

### 2. Generate Knowledge Base ID

Generate a slug from the name as the ID (e.g. "AI Research" → "ai-research"). If the ID already exists in registries.json, prompt the user to modify it.

### 3. Create Directory Structure

Create the following under the target path:

```bash
mkdir -p <path>/raw/assets
mkdir -p <path>/wiki/sources
mkdir -p <path>/wiki/entities
mkdir -p <path>/wiki/concepts
mkdir -p <path>/wiki/analyses
```

### 4. Initialize Core Wiki Files

Create the following files (if they don't exist):

#### `<path>/wiki/index.md`

```markdown
# Wiki Index

> This file is automatically maintained by the LLM and serves as the wiki's content directory. All pages are organized by category, with a one-line summary per page.
> The LLM reads this file first when answering queries to locate relevant pages.

## Overview

- [Overview](overview.md) — Wiki overview and current knowledge graph summary
- [User Conventions](conventions.md) — User preferences for operating this knowledge base

## Sources

## Entities

## Concepts

## Analyses
```

#### `<path>/wiki/log.md`

```markdown
# Wiki Log

> Operation log, recording all wiki operations in reverse chronological order.

## [YYYY-MM-DD] init | Knowledge base initialized

- Created knowledge base directory structure
- Initialized index.md, log.md, overview.md, conventions.md
```

(Replace YYYY-MM-DD with today's date)

#### `<path>/wiki/overview.md`

```markdown
---
title: Overview
aliases: [Overview]
type: overview
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [overview]
---

# Knowledge Base Overview

## Statistics

| Metric | Count |
|--------|-------|
| Raw materials | 0 |
| Source summaries | 0 |
| Entity pages | 0 |
| Concept pages | 0 |
| Analysis pages | 0 |

## Knowledge Graph Summary

(No content yet — auto-updated after ingesting materials)

## Recent Activity

- [YYYY-MM-DD] Knowledge base initialized
```

#### `<path>/wiki/conventions.md`

```markdown
---
title: User Conventions
aliases: [Conventions]
type: conventions
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [meta]
---

# User Conventions

> This page records user preferences and conventions for operating this knowledge base. The LLM should read this page before performing any operation.

## Query

(None yet)

## Ingest

(None yet)

## Lint

(None yet)

## General

(None yet)
```

### 5. Register in registries.json

Read `registries.json` from the skill directory and add a new entry:

```json
{
  "default": "<new-id>",
  "registries": {
    "<new-id>": {
      "name": "User-provided name",
      "path": "/absolute/path/to/kb",
      "language": "en",
      "created": "YYYY-MM-DD"
    }
  }
}
```

Set `default` to the newly created knowledge base ID.

### 6. Handle Existing Directory

If the target path already exists and contains `raw/` and `wiki/` subdirectories:

```json
{
  "questions": [{
    "question": "The target path already contains a knowledge base structure (raw/ and wiki/ found). How should this be handled?",
    "header": "Existing Directory",
    "multiSelect": false,
    "options": [
      {"label": "Register only", "description": "Skip creation and just register the existing knowledge base in the config"},
      {"label": "Overwrite initialization", "description": "Re-create the core wiki files (raw/ is preserved)"},
      {"label": "Cancel", "description": "Do nothing"}
    ]
  }]
}
```

### 7. Output Result

```
✅ Knowledge base created and registered

  Name:     <name>
  Path:     <path>
  Language: en
  ID:       <id>

  Directory structure:
    <path>/raw/          ← Place your materials here
    <path>/wiki/         ← LLM-maintained knowledge base

  Next steps:
    1. Place material files in <path>/raw/
    2. Run /wiki ingest to start ingesting
```
