---
name: wiki
version: v1.0.0
author: ChavesLiu
description: |
  Knowledge base management tool. Ingest materials, query knowledge, health check, reset/delete.
  TRIGGER when: user mentions "ingest", "add to knowledge base", or asks a question the knowledge base may answer (e.g. "What is XX", "compare XX and YY", "summarize info about XX"), or requests "check the wiki" / "lint", or asks to "wipe" / "delete" / "reset" knowledge base content, or provides feedback/preferences about how the wiki should operate.
  DO NOT TRIGGER when: the question is unrelated to the knowledge base (pure coding tasks, file operations, general conversation), or the user explicitly says they don't need the knowledge base.
user-invocable: true
---

# Second Brain Skill

The unified management interface for the knowledge base. Supports command-style invocation (`/wiki <cmd>`) and natural language triggers.

## Natural Language Routing

When the skill is triggered by natural language (not a `/wiki` command), determine the intent first and map to a sub-command:

| User Intent | Mapped Sub-command | Examples |
|-------------|-------------------|---------|
| Ingest/add materials | `ingest` | "Ingest this article", "Add new files from raw/ to the knowledge base" |
| Ask/query/compare/summarize | `query` | "What is Memex", "Compare RAG and Wiki approaches", "Summarize info about XX" |
| Feedback/preferences/correction | `query` (feedback branch) | "Always cite sources in answers", "You got that wrong last time" |
| Check/audit | `lint` | "Check if there are any issues with the knowledge base" |
| Delete/wipe/reset | `wipe` | "Delete pages related to XX", "Reset the knowledge base" |
| Initialize | `init` | "Create a new knowledge base" |

After determining the sub-command, pass the user's original input as `args` and proceed to the execution steps below.

## Execution Steps

### 1. Run the Router Script

Run `python <skill-dir>/scripts/router.py <subcommand> [args]` to get the routing result (JSON).

`<skill-dir>` is the absolute path of the directory containing this file. For natural language triggers, map the sub-command using the table above before calling.

### 2. Execute Based on Routing Result

Branch on the `status` field in the JSON:

#### `status: "ok"` — Normal execution

```
Routing result fields:
  subcommand   — sub-command name
  args         — sub-command arguments
  workflow     — workflow file path (relative to skill directory)
  schema       — SCHEMA.md path (non-null when needed)
  kb           — knowledge base info (id, name, root, wiki, raw, lang)
  skill_dir    — absolute path to skill directory
  multiple_kbs — whether multiple knowledge bases exist (optional)
  kb_list      — list of knowledge bases (optional, present when multiple_kbs is true)
```

Execution order:
1. If `schema` is non-null → read `<skill_dir>/<schema>` to understand page specifications
2. If `<kb.wiki>/conventions.md` exists → read it as additional constraints for this operation (user preferences for this knowledge base)
3. Read `<skill_dir>/<workflow>` to get the workflow
4. Replace `KB_ROOT`, `KB_WIKI`, `KB_RAW`, `KB_LANG` in the workflow with the paths (`root`, `wiki`, `raw`) and language (`lang`) from `kb`
5. Execute the workflow; pass `args` as sub-command parameters; follow conventions.md
6. If `multiple_kbs` is true, inform the user which knowledge base is currently in use (kb.name), and suggest they can change the default in registries.json

#### `status: "select"` — User needs to choose a knowledge base

Use AskUserQuestion to display `kb_list` for the user to select, then continue with the chosen knowledge base. Update `registries.json`'s `default` field after selection.

#### `status: "no_kb"` — No knowledge base

Output: `⚠️ {message}`

#### `status: "error"` — Error

Output the error message.

#### `subcommand: "help"` — Help

Output directly:

```
Second Brain — LLM-powered knowledge base management tool

Commands:
  /wiki init                  Create and register a new knowledge base
  /wiki ingest [filename]     Ingest new materials from raw/ (omit to ingest all)
  /wiki query <question>      Answer questions based on the knowledge base
  /wiki lint                  Knowledge base health check
  /wiki wipe [subcommand]     Reset/delete (all | <keyword> | trash | restore | empty-trash)
  /wiki test                  Automated testing
  /wiki help                  Show this help

Run /wiki init first if this is your first time
```
