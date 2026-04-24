# Second Brain Schema

This file defines the specifications and conventions for wiki pages. All workflows must follow these rules when executing.

## Language Settings

Each knowledge base has its language set during `/wiki init`, stored in `registries.json` as `KB_LANG`:
- **zh**: Write all wiki content in Chinese (keep proper nouns in their original form, e.g. "Memex", "Vannevar Bush")
- **en**: Write all wiki content in English (keep proper nouns in their original language)

Rules:
- Newly ingested materials produce wiki pages written in the `KB_LANG` language
- Existing pages are not auto-translated; run `/wiki lint` to detect language-inconsistent pages and batch-translate them
- The `title` field in frontmatter follows the current language (but `tags` always use English to ensure consistency)
- Filenames always use lowercase English + hyphens, regardless of the language setting

## Frontmatter Specification

Every wiki page must include a YAML frontmatter:

```yaml
---
title: Page Title
aliases: [Alternative name]              # Used for Obsidian graph display; fill with localized aliases matching KB_LANG
type: source | entity | concept | analysis | overview | conventions
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [tag1, tag2]
sources: [source_filename1, source_filename2]  # Referenced raw materials
---
```

## File Naming

- All lowercase English + hyphens: `reinforcement-learning.md`, `openai.md`
- Source summary pages share the same name as the raw file: `raw/paper-x.pdf` → `wiki/sources/paper-x.md`
- Keep filenames under 50 characters

## Cross-References

- Use Obsidian wikilink syntax: `[[page-name]]` or `[[page-name|Display Text]]`
- Each page has a `## Related` section at the bottom listing all related links
- When creating a new page, check whether it should be added to the Related sections of existing pages

## Content Guidelines

- Write wiki content according to the `KB_LANG` setting (zh = Chinese, en = English). Keep proper nouns in their original form
- Prioritize stating facts and cite information sources
- When new materials contradict existing content, explicitly mark with `> ⚠️ Contradiction: ...` and explain both sides
- Keep pages focused — one page, one topic

## Index Format

`wiki/index.md` organized by category:

```markdown
## Sources
- [Material Title](sources/filename.md) — One-line summary (YYYY-MM-DD)

## Entities
- [Entity Name](entities/filename.md) — One-line description [N source references]

## Concepts
- [Concept Name](concepts/filename.md) — One-line description

## Analyses
- [Analysis Title](analyses/filename.md) — One-line summary (YYYY-MM-DD)
```

## Log Format

`wiki/log.md` in reverse chronological order:

```markdown
## [YYYY-MM-DD] <operation> | <title>
- Specific operation description
- List of affected pages
```

Valid values for `operation`: `ingest`, `query`, `lint`, `update`, `init`, `wipe`

## Contradiction Handling

When new materials contradict existing wiki content:

1. **Do not delete old information** — preserve both perspectives
2. **Mark the contradiction** — use `> ⚠️ Contradiction: ...` callout in the relevant section
3. **Cite sources** — note the source material for each side
4. **Record dates** — note the date of each piece of information to help judge timeliness
5. **Leave judgment to the user** — if the contradiction involves a core argument, flag it during discussion
