# Test Workflow

Automated testing of the three core workflows (ingest, query, lint) and architecture integrity verification.

> 📌 **Path convention**: `raw/` and `wiki/` in this document refer to the absolute paths of the current knowledge base, determined when SKILL.md routes the command (`KB_RAW`, `KB_WIKI`). A knowledge base must be selected (via knowledge base selection or `/wiki init`) before running tests.

## Test Procedures

### Test 1: Architecture Integrity Check

Verify that the directory structure and core files exist:

```
Checks:
- [ ] raw/ directory exists
- [ ] raw/assets/ directory exists
- [ ] wiki/ directory exists
- [ ] wiki/sources/ directory exists
- [ ] wiki/entities/ directory exists
- [ ] wiki/concepts/ directory exists
- [ ] wiki/analyses/ directory exists
- [ ] wiki/index.md exists and contains the correct category headings
- [ ] wiki/log.md exists and contains an initial entry
- [ ] wiki/overview.md exists and contains frontmatter
- [ ] <skill-dir>/SCHEMA.md exists and contains schema definitions
- [ ] <skill-dir>/SKILL.md exists
- [ ] <skill-dir>/workflows/ingest.md exists
- [ ] <skill-dir>/workflows/query.md exists
- [ ] <skill-dir>/workflows/lint.md exists
- [ ] <skill-dir>/workflows/wipe.md exists
- [ ] <skill-dir>/scripts/lint.py exists
```

Use the Glob and Read tools to verify each item and report results.

### Test 2: Ingest Workflow Test

1. Check whether a test material `raw/test-sample.md` exists under `raw/`
2. If it doesn't exist, create one:
   ```markdown
   # Vannevar Bush and the Memex Vision

   In 1945, Vannevar Bush proposed the concept of the Memex in his article "As We May Think"...
   (Contains entities: Vannevar Bush, Memex; concepts: associative memory, knowledge management)
   ```
3. Execute the ingest workflow (run the logic of `/wiki ingest test-sample.md`)
4. Verify:
   - [ ] `wiki/sources/test-sample.md` has been created
   - [ ] The file contains correct frontmatter
   - [ ] `wiki/index.md` has been updated with the new entry
   - [ ] `wiki/log.md` has been updated with an ingest record
   - [ ] `wiki/overview.md`'s `source_count` has been updated

### Test 3: Query Workflow Test

1. Execute a query: "What is the Memex?"
2. Verify:
   - [ ] The answer references relevant wiki pages
   - [ ] The answer contains `[[` style page references
   - [ ] The answer is based on knowledge base content, not the LLM's own knowledge

### Test 4: Lint Workflow Test

1. Run a lint check
2. Verify:
   - [ ] A structured report was generated
   - [ ] The report contains statistics
   - [ ] The report is categorized by priority
   - [ ] Any issues introduced by the test are detected (e.g. missing cross-references)

### Test 5: Cleanup

After testing, ask the user whether to keep the test data:
- Keep: leave the test materials in the wiki as examples
- Clean up: delete all files produced by the test and restore index.md and log.md

## Results Summary

```
========================================
  Wiki Automated Test Report
========================================
  Architecture Integrity:  ✅/❌ (N/M passed)
  Ingest Workflow:         ✅/❌ (N/M passed)
  Query Workflow:          ✅/❌ (N/M passed)
  Lint Workflow:           ✅/❌ (N/M passed)
========================================
  Total: N/M passed
========================================
```
