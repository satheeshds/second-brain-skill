# Wipe Workflow

Clear or delete knowledge base content. All deletions require user confirmation; files are moved to the recycle bin and can be recovered.

> 📌 **Path convention**: `raw/` and `wiki/` in this document refer to the absolute paths of the current knowledge base, determined when SKILL.md routes the command (`KB_RAW`, `KB_WIKI`).

## Core Principles

1. **Manual confirmation** — All delete operations must list the files and wait for user confirmation before executing
2. **Recycle bin mechanism** — Deleting = moving to `wiki/.trash/`, not permanent deletion; recovery is supported
3. **Never touch raw materials** — The `raw/` directory is never modified
4. **Clean up associations** — When deleting a page, also clean up its entries in index.md, overview.md, and Related links in other pages

## Recycle Bin

Path: `wiki/.trash/`

- Files are stored preserving the original directory structure, e.g. `wiki/concepts/swiglu.md` → `wiki/.trash/concepts/swiglu.md`
- On name conflicts, the old file gets a timestamp suffix: `swiglu.2026-04-13.md`
- Restoration moves the file back to its original path and re-updates index.md, overview.md, and Related links in associated pages
- Users can run `/wiki wipe trash` to view the recycle bin, `/wiki wipe restore` to restore files, and `/wiki wipe empty-trash` to permanently empty it

## Usage

- `/wiki wipe` — Interactive operation selection
- `/wiki wipe all` — Full reset
- `/wiki wipe <keyword>` — Delete pages matching the keyword
- `/wiki wipe trash` — View the recycle bin
- `/wiki wipe restore` — Restore pages from the recycle bin
- `/wiki wipe empty-trash` — Permanently empty the recycle bin

## Workflow

### 1. Determine Operation Mode

#### No arguments — Interactive selection

```json
{
  "questions": [{
    "question": "Select a knowledge base operation:",
    "header": "Operation Type",
    "multiSelect": false,
    "options": [
      {"label": "Full reset", "description": "Move all wiki pages to the recycle bin and return to initial state (raw/ unaffected)"},
      {"label": "Delete specific material", "description": "Delete a material and its associated pages (moved to recycle bin)"},
      {"label": "Delete specific pages", "description": "Delete one or more entity/concept/analysis pages (moved to recycle bin)"},
      {"label": "View/restore recycle bin", "description": "View recycle bin contents and restore deleted pages"}
    ]
  }]
}
```

### 2. Full Reset Flow

#### 2a. Inventory current content

Read `wiki/index.md` and count the files to be moved to the recycle bin:
- `wiki/sources/*.md`
- `wiki/entities/*.md`
- `wiki/concepts/*.md`
- `wiki/analyses/*.md`

#### 2b. Confirmation (required)

```json
{
  "questions": [{
    "question": "Confirm full reset? The following N files will be moved to the recycle bin (wiki/.trash/):\n\nSource summaries: X\nEntity pages: X\nConcept pages: X\nAnalysis pages: X\n\n⚠️ Recoverable via /wiki wipe restore\n⚠️ raw/ directory is not affected",
    "header": "Confirm Reset",
    "multiSelect": false,
    "options": [
      {"label": "Confirm reset", "description": "Move all wiki pages to the recycle bin and reset index/overview"},
      {"label": "Cancel", "description": "Do nothing"}
    ]
  }]
}
```

#### 2c. Execute reset

After user confirmation:

1. **Create recycle bin directories** (if they don't exist): `mkdir -p wiki/.trash/{sources,entities,concepts,analyses}`

2. **Move all content pages to the recycle bin**:
   ```bash
   mv wiki/sources/*.md wiki/.trash/sources/
   mv wiki/entities/*.md wiki/.trash/entities/
   mv wiki/concepts/*.md wiki/.trash/concepts/
   mv wiki/analyses/*.md wiki/.trash/analyses/
   ```
   On name conflicts, add a timestamp suffix to the old file before moving.

3. **Reset index.md** to an empty index template:
   ```markdown
   # Wiki Index

   > This file is automatically maintained by the LLM and serves as the wiki's content directory. All pages are organized by category, with a one-line summary per page.
   > The LLM reads this file first when answering queries to locate relevant pages.

   ## Overview

   - [Overview](overview.md) — Wiki overview and current knowledge graph summary

   ## Sources

   ## Entities

   ## Concepts

   ## Analyses
   ```

4. **Reset overview.md**: preserve frontmatter, zero out statistics, clear the knowledge graph summary and recent activity.

5. **Append a record to the top of log.md**:
   ```markdown
   ## [YYYY-MM-DD] wipe | Full reset

   - Moved to recycle bin: X source summaries, X entities, X concepts, X analyses
   - Reset index.md and overview.md
   - Recoverable via /wiki wipe restore
   ```

6. **Output result**:
   ```
   ✅ Knowledge base reset
      Moved to recycle bin: N pages
      raw/ preserved: M materials available for re-ingest with /wiki ingest
      💡 /wiki wipe restore to recover
   ```

### 3. Selective Delete Flow

#### 3a. Determine the deletion target

**Delete by material**:
1. Read `wiki/index.md`, list materials for the user to select (or match by keyword)
2. Find the source summary page `wiki/sources/xxx.md`
3. Analyze the impact:
   - "Will be deleted": pages only produced by this material (the `sources` field contains only this material)
   - "Will be updated": pages still referenced by other materials (kept, but the association is removed)

**Delete by page**:
1. User specifies or selects pages to delete from a list
2. Analyze which other pages' Related sections reference the target pages

#### 3b. Confirmation (required)

```json
{
  "questions": [{
    "question": "Deleting 'Target' will have the following effects:\n\n🗑️ Move to recycle bin:\n- wiki/sources/xxx.md\n- wiki/entities/yyy.md (only referenced by this material)\n\n✏️ Will be updated (remove association):\n- wiki/concepts/zzz.md (still referenced by other materials; kept)\n\nConfirm?",
    "header": "Confirm Deletion",
    "multiSelect": false,
    "options": [
      {"label": "Confirm deletion", "description": "Execute the deletions and updates listed above"},
      {"label": "Delete summary page only", "description": "Delete only the source page; keep all entity/concept pages"},
      {"label": "Cancel", "description": "Do nothing"}
    ]
  }]
}
```

#### 3c. Execute deletion

After user confirmation:

1. **Move target pages to the recycle bin** (preserving the original directory structure)
2. **Update affected pages**:
   - Remove the deleted material reference from the `sources` frontmatter field
   - Remove links pointing to deleted pages from Related sections
   - Update the `updated` date
3. **Update index.md** — remove entries for deleted pages
4. **Update overview.md** — update statistics and the knowledge graph summary
5. **Append a record to the top of log.md**
6. **Run `python <skill-dir>/scripts/lint.py --wiki-dir <KB_WIKI> --raw-dir <KB_RAW>`** to check for any new broken links or orphans
7. **Output result**:
   ```
   ✅ Deletion complete
      Moved to recycle bin: N pages
      Updated: M pages (associations removed)
      Lint: passed
      💡 /wiki wipe restore to recover
   ```

### 4. Recycle Bin Operations

#### `/wiki wipe trash` — View recycle bin

Scan the `wiki/.trash/` directory and list all files by category along with their original paths before deletion. If the recycle bin is empty, display "Recycle bin is empty."

#### `/wiki wipe restore` — Restore pages

1. List all files in the recycle bin for the user to select (multi-select supported)
2. **Confirm restoration**: show the list of files to be restored; execute after confirmation
3. Execute restoration:
   - Move files from `wiki/.trash/` back to their original paths
   - Re-add entries to index.md
   - Re-add links to the Related sections of associated pages
   - Update overview.md statistics
   - Append a restore record to log.md
4. Run `python <skill-dir>/scripts/lint.py --wiki-dir <KB_WIKI> --raw-dir <KB_RAW>` to verify consistency after restoration

#### `/wiki wipe empty-trash` — Permanently empty the recycle bin

1. List all files in the recycle bin
2. **Confirmation**: ⚠️ This operation is irreversible (except via git recovery); require user confirmation
3. After confirmation: `rm -rf wiki/.trash/*`

## Notes

- **Never delete raw/** — raw materials are not wiki content; they are read-only
- **log.md is append-only** — the history of resets is itself valuable
- It is recommended to add `wiki/.trash/` to `.gitignore` to prevent recycle bin contents from entering version control
- If the user wants to clear `raw/`, they must do it manually; the skill will suggest but will not execute it
