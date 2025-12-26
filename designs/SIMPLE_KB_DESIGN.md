# simple_kb Design Document

**Eiffel Knowledge Base CLI - Searchable Developer Reference**

**Date:** 2025-12-23
**Status:** Design Complete
**Research:** `/d/prod/reference_docs/research/SIMPLE_KB_RESEARCH.md`

---

## Executive Summary

**simple_kb** is a command-line tool that answers Eiffel programming questions using FTS5 full-text search over a curated knowledge base. Unlike generic LLMs, it provides:

- **Authoritative answers** indexed from actual source code
- **Compiler error decoder** with specific fixes and examples
- **Offline operation** with sub-100ms response times
- **Cross-reference** of 59 simple_* libraries + ISE + Gobo + Rosetta examples

**Core Value Proposition:**
> When you ask "What library handles JSON?" or "What does VEVI mean?", you get accurate, instant answers from indexed source code - not hallucinations.

---

## Problem Statement

### Current Pain Points

| Problem | Impact |
|---------|--------|
| "What library does X?" | Developers don't know what exists in 59 simple_* libs |
| "How do I do X in Eiffel?" | Scattered examples, no central search |
| "What does error X mean?" | Compiler shows VEVI but no actionable fix |
| "What's the Eiffel way?" | Coming from Java/Python, syntax is unfamiliar |
| Generic LLMs hallucinate | ChatGPT invents features that don't exist |

### The Solution

A local FTS5-powered CLI that:
1. Indexes actual source code (no hallucination)
2. Decodes compiler errors with fixes
3. Searches 274 Rosetta Code solutions
4. Works offline, instantly

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        simple_kb CLI                             │
│                                                                  │
│   eiffel-kb search "json"                                        │
│   eiffel-kb error VEVI                                          │
│   eiffel-kb class SIMPLE_HTTP                                   │
│   eiffel-kb example "bubble sort"                               │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│                        KB_SEARCH                                 │
│                                                                  │
│   Query Parser → FTS5 MATCH → BM25 Ranking → Result Format      │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│                       KB_DATABASE                                │
│                                                                  │
│   ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│   │ classes  │ │ features │ │ examples │ │ errors   │          │
│   └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
│   ┌──────────┐ ┌──────────┐ ┌────────────────────────┐          │
│   │ patterns │ │translat. │ │ kb_search (FTS5)      │          │
│   └──────────┘ └──────────┘ └────────────────────────┘          │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│                      KB_INGESTER                                 │
│                                                                  │
│   simple_* src → Parser → Classes/Features                      │
│   rosetta.db   → Import → Examples                              │
│   errors.json  → Load   → Errors                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Database Schema

```sql
-- Classes from source code
CREATE TABLE classes (
    id INTEGER PRIMARY KEY,
    library TEXT NOT NULL,        -- 'simple_json', 'base', 'gobo'
    name TEXT NOT NULL,           -- 'JSON_PARSER'
    description TEXT,             -- From note clause
    file_path TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(library, name)
);

-- Features (methods) from classes
CREATE TABLE features (
    id INTEGER PRIMARY KEY,
    class_id INTEGER REFERENCES classes(id),
    name TEXT NOT NULL,
    signature TEXT,               -- '(text: STRING): JSON_OBJECT'
    description TEXT,
    kind TEXT,                    -- 'query', 'command', 'creation'
    preconditions TEXT,           -- JSON array
    postconditions TEXT,          -- JSON array
    UNIQUE(class_id, name)
);

-- Code examples (Rosetta + manual)
CREATE TABLE examples (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    source TEXT,                  -- 'rosetta', 'manual'
    code TEXT NOT NULL,
    tags TEXT,                    -- JSON array
    tier TEXT
);

-- Compiler error codes
CREATE TABLE errors (
    id INTEGER PRIMARY KEY,
    code TEXT UNIQUE NOT NULL,    -- 'VEVI', 'VD89'
    meaning TEXT NOT NULL,
    explanation TEXT,
    common_causes TEXT,           -- JSON array
    fixes TEXT,                   -- JSON array
    examples TEXT                 -- JSON: bad → good
);

-- Design patterns
CREATE TABLE patterns (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT,
    code TEXT NOT NULL,
    when_to_use TEXT,
    eiffel_idioms TEXT
);

-- Language translations
CREATE TABLE translations (
    id INTEGER PRIMARY KEY,
    source_lang TEXT NOT NULL,
    source_pattern TEXT NOT NULL,
    eiffel_pattern TEXT NOT NULL,
    notes TEXT
);

-- FTS5 unified search index
CREATE VIRTUAL TABLE kb_search USING fts5(
    content_type,                 -- 'class', 'feature', 'example', 'error'
    content_id,
    title,
    body,
    tags,
    tokenize='porter unicode61'
);
```

---

## CLI Interface

```
Usage: eiffel-kb <command> [options]

SEARCH COMMANDS:
  search <query>          Full-text search across all content
  class <name>            Show class details + features
  feature <class.feat>    Show feature signature + contracts

ERROR HELP:
  error <code>            Explain compiler error code (VEVI, VD89, etc.)

EXAMPLES:
  example <topic>         Find code examples
  pattern <name>          Show design pattern
  translate <lang:code>   Show Eiffel equivalent

ADMIN:
  ingest <path>           Index source files
  import-rosetta          Import from simple_rosetta
  stats                   Show database statistics

OPTIONS:
  --limit N       Max results (default: 10)
  --json          Output as JSON
  --verbose       Include full code

EXAMPLES:
  eiffel-kb search "json parsing"
  eiffel-kb error VEVI
  eiffel-kb class SIMPLE_HTTP
  eiffel-kb example "file read"
  eiffel-kb pattern singleton
```

---

## Key Classes

| Class | Purpose | Dependencies |
|-------|---------|--------------|
| `SIMPLE_KB` | Main facade with type references | All |
| `KB_DATABASE` | SQLite + FTS5 management | simple_sql |
| `KB_SEARCH` | Query processing | KB_DATABASE |
| `KB_INGESTER` | Source code indexing | simple_eiffel_parser |
| `KB_CLI` | Command-line interface | simple_cli |
| `KB_RESULT` | Search result model | - |
| `KB_FORMATTER` | Output formatting | simple_json |
| `KB_QUICK` | One-liner facade | All |

---

## Sample Interactions

### Error Code Lookup
```
$ eiffel-kb error VEVI

ERROR: VEVI - Variable Not Properly Set
═══════════════════════════════════════

MEANING:
  An attached variable is not assigned a value on all execution paths.

COMMON CAUSES:
  1. Result not assigned in else branch
  2. Missing initialization before loop
  3. Conditional assignment without fallback

EXAMPLE (BAD):
  get_name: STRING
    do
      if has_name then
        Result := stored_name
      end
      -- Missing: what if not has_name?
    end

FIX OPTIONS:

  Option 1: Add else branch
    else
      Result := ""
    end

  Option 2: Make return type detachable
    get_name: detachable STRING

REFERENCE: ECMA-367 Section 8.19.17
```

### Library Search
```
$ eiffel-kb search "http client"

SEARCH RESULTS (3 matches)
═══════════════════════════

1. [CLASS] SIMPLE_HTTP (simple_http)
   High-level HTTP client with get/post/put/delete
   Features: get, post, get_json, post_json, ...

2. [CLASS] SIMPLE_HTTP_QUICK (simple_http)
   One-liner facade for common HTTP operations
   Example: http.get_json ("https://api.example.com")

3. [EXAMPLE] HTTP Get Request (rosetta)
   Task: Send HTTP GET request
   Tier: TIER2
```

### Class Details
```
$ eiffel-kb class JSON_PARSER

CLASS: JSON_PARSER
══════════════════
Library: simple_json
Description: Parse JSON text into structured objects

CREATION:
  make

FEATURES (Query):
  parse (text: STRING): JSON_VALUE
      -- Parse JSON text, return root value
      require: text_not_empty: not text.is_empty
      ensure: result_not_void: Result /= Void

  parse_object (text: STRING): JSON_OBJECT
      -- Parse JSON expecting object {...}

  parse_array (text: STRING): JSON_ARRAY
      -- Parse JSON expecting array [...]

FEATURES (Status):
  has_error: BOOLEAN
  last_error: detachable STRING

SEE ALSO: JSON_OBJECT, JSON_ARRAY, JSON_VALUE
```

---

## Implementation Plan

### Phase 1: Foundation (5 days)

| Task | Days | Deliverable |
|------|------|-------------|
| 1A. Schema + KB_DATABASE | 2 | SQLite with FTS5 |
| 1B. Error database (curate 30+ codes) | 2 | errors.json + loader |
| 1C. CLI with `error` command | 1 | Working error lookup |

**Milestone:** `eiffel-kb error VEVI` works

### Phase 2: Source Indexing (6 days)

| Task | Days | Deliverable |
|------|------|-------------|
| 2A. KB_INGESTER (parser integration) | 3 | Index from .e files |
| 2B. `search`, `class`, `feature` | 2 | Full search capability |
| 2C. Rosetta import | 1 | 274 examples indexed |

**Milestone:** Search across all simple_* libraries

### Phase 3: Enrichment (6 days)

| Task | Days | Deliverable |
|------|------|-------------|
| 3A. Pattern library (30 patterns) | 2 | patterns.json + display |
| 3B. Translation mappings | 2 | Python/Java → Eiffel |
| 3C. Tests + documentation | 2 | README, tests, docs |

**Milestone:** Production-ready v1.0

---

## Dependencies

| Library | Use |
|---------|-----|
| simple_sql | FTS5, database operations |
| simple_eiffel_parser | Parse .e files for indexing |
| simple_cli | Command-line framework |
| simple_json | JSON config files, output |
| simple_file | Read source files |
| simple_rosetta | Import examples |

---

## Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Classes indexed | 500+ | Count from simple_* |
| Features indexed | 3000+ | Extracted signatures |
| Error codes | 50+ | Curated database |
| Examples | 274+ | Rosetta import |
| Query time | <100ms | Benchmark |
| Offline | 100% | No network dependency |

---

## Future Enhancements

| Feature | Priority | Notes |
|---------|----------|-------|
| Signature search | Medium | "Find (ARRAY, INTEGER): STRING" |
| Web UI | Medium | simple_web + simple_htmx |
| IDE integration | Medium | LSP hover info |
| Vector search | Low | Semantic similarity |
| Auto-update | Low | Watch source changes |

---

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Parser limitations | Graceful degradation, log unparseable |
| Large codebase | Incremental indexing, progress display |
| Stale content | Version tracking, re-index command |
| Missing error codes | Community contributions, gradual addition |

---

## Conclusion

simple_kb fills a critical gap in the Eiffel ecosystem: instant, accurate answers to programming questions. By indexing actual source code and curating error explanations, it provides value that generic LLMs cannot match.

**Start with error codes** - highest value, immediate impact.
**Then source indexing** - make the 59 simple_* libraries discoverable.
**Finally patterns and translations** - help developers coming from other languages.

The 17-day implementation produces a tool that will be used daily by every Eiffel developer.

---

## Appendix: Error Code Template

```json
{
  "code": "VEVI",
  "meaning": "Variable not properly set",
  "ecma_section": "8.19.17",
  "explanation": "An attached local variable or Result must be assigned...",
  "common_causes": [
    "Result not assigned in all branches",
    "Missing initialization before loop",
    "Conditional assignment without else"
  ],
  "fixes": [
    {
      "title": "Add missing branch",
      "description": "Ensure all code paths assign the variable",
      "example": "if cond then Result := x else Result := y end"
    },
    {
      "title": "Make detachable",
      "description": "Allow Void as valid return",
      "example": "my_feature: detachable STRING"
    }
  ],
  "bad_code": "if cond then Result := x end",
  "good_code": "if cond then Result := x else Result := default end"
}
```

---

*Generated by 7-Step Research Protocol*
*Research: SIMPLE_KB_RESEARCH.md*
