# Query Workflow

Answer user questions based on the knowledge base. Search for relevant wiki pages, synthesize information, and provide cited answers. Core philosophy: **every query should make the knowledge base better**.

> 📌 **Path convention**: `raw/` and `wiki/` in this document refer to the absolute paths of the current knowledge base, determined when SKILL.md routes the command (`KB_RAW`, `KB_WIKI`). `KB_LANG` is the language setting for the current knowledge base.

## Workflow

### 1. Understand the Question

Analyze the user's input and determine its type:

- **Knowledge query** — a normal knowledge question → continue to steps 2–5
- **Operational preference/feedback** — a preference about how operations should work (e.g. "always cite sources in answers") → jump directly to step 4d
- **Correction** — a correction to a previous answer (e.g. "you were wrong, it's actually XX") → jump to step 4a to fix the corresponding wiki page
- **Preference + correction** — both at once (e.g. "that answer was too shallow; XX should also cover YY") → 4a to add content + 4d to record the preference

**Design principle**: the wiki itself is memory. Content errors or gaps are resolved by updating wiki pages (so the next query naturally gives the right answer) — no need to store "lessons learned" separately. `conventions.md` stores only preference rules, not correction history.

If it's a normal knowledge query, identify keywords, entities, and concepts involved.

### 2. Retrieve Relevant Pages

Search in order:
1. Read `wiki/index.md` and use the question's keywords to locate potentially relevant pages
2. **Check `wiki/analyses/` for any existing relevant analysis** (avoid duplicating work; build on prior analysis)
3. Read the located wiki pages
4. If more context is needed, use Grep to search for keywords under `wiki/`
5. If raw data is involved, consult the original materials under `raw/`

### 3. Synthesize the Answer

Based on retrieved information, generate a structured answer:
- Answer the question directly
- Cite specific wiki pages as sources: `(see [[page-name]])`
- **If relevant prior analysis exists, build on it** rather than starting from scratch
- If information is insufficient, clearly state what is missing from the knowledge base
- If there is conflicting information, list what different sources say

### 4. Knowledge Write-back (Self-evolution Core)

After each query, evaluate whether the answer produced new knowledge and handle it according to the rules below:

#### 4a. Auto write-back (no user confirmation needed)

Update existing pages directly in these cases, without asking:
- **Supplementary info**: the answer synthesized information that an entity/concept page hasn't recorded yet → append to that page and update the `updated` date
- **New cross-references**: the answer process uncovered a connection between pages that wasn't previously linked → add links in both pages' Related sections
- **Fix small errors**: a factual error is found in a page during answering → fix it directly

#### 4b. Suggested write-back (requires user confirmation)

Suggest the following to the user and execute upon confirmation:
- **Analysis worth its own page**: comparative analysis, synthesis, newly discovered connections → save as a new page under `wiki/analyses/`
- **New entities/concepts**: the answer involves an important entity or concept not yet in the wiki → suggest creating a new page
- **Contradiction discovered**: conflicting information found between existing pages → suggest flagging it

Suggestion format:
```
📝 This query produced the following knowledge updates:

Auto-updated:
  - Updated [[memex]] page with a comparison to modern RAG systems
  - Added cross-references between [[vannevar-bush]] and [[knowledge-management]]

Suggested actions:
  - 💡 Save this comparative analysis as wiki/analyses/memex-vs-rag.md?
  - 💡 Create a new concept page wiki/concepts/rag.md for "RAG"?
```

#### 4d. User feedback write-back (auto-execute)

Triggered when the user's input is an operational preference or feedback (not a knowledge query).

**Classification rules:**

| Category | Criteria | Examples |
|----------|----------|---------|
| Query | Relates to answer style, output format, citation style | "Always cite sources", "Use tables for comparison questions" |
| Ingest | Relates to ingestion strategy, page splitting, naming preferences | "Don't create standalone pages for brief mentions", "Use English for entity page titles" |
| Lint | Relates to check preferences, fix strategy | "Don't report P2 issues", "Auto-fix broken links without asking" |
| General | Cross-operation preferences | "Always reply in English", "Just give a one-line summary after operations" |

**Execution steps:**
1. Read `wiki/conventions.md` (create from the initial template if it doesn't exist)
2. Determine the category and append the feedback as a `- ` list item under the corresponding section
3. **Deduplicate**: if a semantically identical item already exists, update it instead of adding a duplicate
4. **Refine**: if a category has > 10 items, merge semantically similar items into more concise rules
5. Update the `updated` date
6. Inform the user which category the feedback was recorded under

> ⚠️ Do not store operational preferences in Claude Code memory — store them in the knowledge base's own `conventions.md`.

#### 4c. Write-back execution

After user confirmation:
- Create new pages (with complete frontmatter and Related sections)
- Update the Related sections of relevant existing pages (bidirectional links)
- Update `wiki/index.md`
- Update `wiki/overview.md` (if statistics change)
- Record in `wiki/log.md` with format: `## [YYYY-MM-DD] query | brief question description`

### 5. Answer Format

Choose the best format based on the question type:
- **Fact lookup**: short direct answer + citations
- **Comparative analysis**: Markdown table
- **Synthesis**: structured long-form
- **Timeline**: chronologically ordered list
- **Overview**: hierarchical mind-map-style list
- **Slides**: Marp-format slide deck (for presentations and sharing)
- **Charts**: matplotlib visualizations (for data comparisons, trend analysis)
- **Canvas**: Obsidian Canvas format (for relationship diagrams, flowcharts)

## Notes

- **Language**: Use the `KB_LANG` setting; answers and new pages should consistently use the target language
- Prioritize existing synthesized information in the wiki rather than re-deriving from raw materials on every query
- **Reuse prior analysis**: if `analyses/` already contains relevant analysis, build on it rather than repeating
- Clearly distinguish between "information recorded in the wiki" and "LLM's own knowledge"
- If the question is entirely outside the knowledge base's scope, inform the user and suggest ingesting relevant materials
- Be conservative with auto write-backs: only add information with high certainty; do not add speculative content
