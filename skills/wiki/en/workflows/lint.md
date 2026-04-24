# Lint Workflow

Run a health check on the knowledge base, surface issues, and provide fix recommendations.

> 📌 **Path convention**: `raw/` and `wiki/` in this document refer to the absolute paths of the current knowledge base, determined when SKILL.md routes the command (`KB_RAW`, `KB_WIKI`). `KB_LANG` is the language setting for the current knowledge base.

## Workflow

### 1. Run Deterministic Check Script

**First**, run `python <skill-dir>/scripts/lint.py --wiki-dir <KB_WIKI> --raw-dir <KB_RAW> --json` to get the structural issues detected by the script. The script covers these deterministic checks:

- Broken links (`[[link]]` pointing to pages that don't exist)
- `[[raw/...]]` wikilink misuse (should use plain Markdown links)
- Frontmatter completeness (required fields, valid `type` values)
- Consistency between index.md and actual files
- Bidirectional link integrity (if A→B then B→A)
- Orphan pages
- Entity/concept pages missing the `sources` field

Use the script output as the basis for P0/P1 issues. **Do not re-check items already covered by the script.**

### 2. LLM Supplementary Check

On top of the script results, use LLM capabilities to supplement **semantic issues the script cannot detect**:

#### P1 — Quality (LLM supplement)
- **Language consistency**: Using the `KB_LANG` setting, check whether each page's main content uses the target language. List inconsistent pages and provide a batch translation option (translate page by page after user confirmation, preserving frontmatter and `[[links]]`)
- **Contradiction detection**: Scan all `> ⚠️ Contradiction` markers to summarize known contradictions; also check for unmarked conflicting statements across different pages
- **Missing pages**: Concepts/entities referenced with `[[links]]` 3+ times that don't have their own page
- **Stale information**: Pages with an `updated` date older than 30 days that have newer materials referencing the same topic
- **Empty sections**: Pages with `_to be filled_` or empty sections

#### P2 — Suggestions (LLM supplement)
- **Missing cross-references**: Related pages with no mutual links
- **Inconsistent tags**: The same concept using different tag names
- **Mergeable pages**: Pages with highly overlapping content
- **New page suggestions**: New concept/entity pages suggested based on existing content
- **Data gaps and knowledge expansion** (using WebSearch tool):
  1. Identify obvious information gaps in the knowledge base (e.g. topics referenced by multiple pages but lacking depth)
  2. Use the **WebSearch** tool to search for the latest developments, key papers, and authoritative resources in related areas
  3. Output a suggestion list; each item includes: gap description, recommended sources found (title + link), suggested ingestion priority
  4. Format example:
     ```
     🔍 Knowledge expansion suggestions (based on web search):
     - [[memex]] mentions Ted Nelson's hypertext but has no detailed content
       → Recommended: "Ted Nelson and the Xanadu Project" (https://...)
       → Priority: Medium
     - [[manufacturing-ai]] lacks 2026 latest developments
       → Recommended: "State of AI in Manufacturing 2026" (https://...)
       → Priority: High
     ```

### 3. Generate Report

Output a structured health check report:

```markdown
## Wiki Health Check Report (YYYY-MM-DD)

### 📊 Statistics
- Total pages: N
- Source summaries: N
- Entity pages: N
- Concept pages: N
- Analysis pages: N

### 🔴 P0 — Must Fix
- [ ] Issue description → Fix approach

### 🟡 P1 — Recommended Improvements
- [ ] Issue description → Improvement approach

### 🟢 P2 — Optional Optimizations
- [ ] Suggestion description
```

### 4. Execute Fixes

After presenting the report to the user:
- Recommend fixing P0 issues immediately; request user confirmation
- Confirm P1 issues one by one
- P2 issues are suggestions only

After user confirmation, execute the fixes and update `wiki/log.md`.

## Notes

- Lint does not delete any pages — it only suggests merging or updating
- If there are many issues, present them in batches to avoid information overload
- After each lint, record a summary of the check results in log.md
