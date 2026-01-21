# EIFGENs Metadata Integration Design

**Date:** December 10, 2025
**Status:** Design Document
**Author:** Claude (with Larry)

## Overview

After exploring the compiled output of ISE EiffelStudio, we discovered that the compiler generates rich metadata in C source files that can be parsed to provide accurate semantic information for IDE features. This document outlines a design for integrating this metadata into simple_lsp and simple_oracle.

## Discovery Summary

When `ec.exe` compiles an Eiffel system, it generates the following metadata files in `EIFGENs/<target>/W_code/E1/`:

| File | Contents | Extractable Data |
|------|----------|------------------|
| `eparents.c` | Inheritance hierarchy | Class names, parent chains, type indices |
| `enames.c` | Feature name tables | Attribute/feature names per class |
| `eskelet.c` | Class skeletons | Attribute types (REF, BOOL, INT32, etc.) |
| `evisib.c` | Visibility tables | Complete class name list, hash lookup |
| `ecall.c` | Dispatch tables | Routine dispatch information |
| `project.epr` | Binary database | Full project state (5+ MB, binary format) |

### Key Insight

The C files are **generated**, **human-readable**, and contain **structured data** that can be parsed with regular expressions or simple parsers. We don't need to reverse-engineer the binary `project.epr` - the C files already expose what we need.

## Architecture

### Two-Mode Operation for simple_lsp

```
+------------------+     +------------------+
|   Parser Mode    |     | Compiler Mode    |
| (Current)        |     | (New)            |
+------------------+     +------------------+
| - Works on       |     | - Requires       |
|   broken code    |     |   successful     |
| - Fast startup   |     |   compilation    |
| - Limited        |     | - Full semantic  |
|   semantics      |     |   information    |
| - No type        |     | - Accurate       |
|   resolution     |     |   inheritance    |
+------------------+     +------------------+
         |                       |
         +----------+------------+
                    |
            +-------v-------+
            | Hybrid Symbol |
            |   Database    |
            +---------------+
```

### Mode Selection Logic

```
IF EIFGENs/<target>/W_code/E1/ exists
AND E1/*.c files newer than source .e files
THEN
    Use compiler-assisted mode
    Parse C metadata for accurate info
ELSE
    Use parser-only mode
    Parse .e files directly
END
```

## simple_lsp Integration

### New Classes

```
EIFGENS_METADATA_PARSER
    -- Parses compiled metadata from EIFGENs folder

    parse_eparents (a_path: PATH)
        -- Extract class hierarchy from eparents.c

    parse_enames (a_path: PATH)
        -- Extract feature names from enames.c

    parse_eskelet (a_path: PATH)
        -- Extract attribute types from eskelet.c

    parse_evisib (a_path: PATH)
        -- Extract class name table from evisib.c

    is_valid_eifgens (a_project_path: PATH): BOOLEAN
        -- Check if EIFGENs has valid compiled output

    eifgens_newer_than_sources (a_project_path: PATH): BOOLEAN
        -- Check if compilation is up to date

COMPILED_CLASS_INFO
    -- Rich class information from compiler

    name: STRING
    type_index: INTEGER
    parent_indices: ARRAYED_LIST [INTEGER]
    attributes: ARRAYED_LIST [COMPILED_ATTRIBUTE_INFO]

COMPILED_ATTRIBUTE_INFO
    -- Attribute with resolved type

    name: STRING
    type_kind: INTEGER  -- SK_REF, SK_BOOL, SK_INT32, etc.
    type_index: INTEGER -- For reference types
```

### Parsing Strategy

#### eparents.c Parser

```eiffel
-- Pattern: /* CLASS_NAME */
-- Pattern: static EIF_TYPE_INDEX ptfN[] = {parent_id,0xFFFF};
-- Pattern: static struct eif_par_types parN = {N, ptfN, ...};

parse_class_entry (a_line: STRING)
    -- "/* SIMPLE_JSON */" -> class name
    -- "static EIF_TYPE_INDEX ptf563[] = {0,0xFFFF};" -> parents
```

#### enames.c Parser

```eiffel
-- Pattern: char *namesN [] = { "feature1", "feature2", ... };

parse_feature_names (a_class_index: INTEGER): ARRAYED_LIST [STRING]
    -- Extract feature names for class N
```

#### evisib.c Parser

```eiffel
-- Pattern: static char * type_key [] = { ... "CLASS_NAME", ... };

parse_all_class_names: ARRAYED_LIST [STRING]
    -- Extract complete class name list
```

### Database Schema Extension

```sql
-- New table for compiled metadata
CREATE TABLE compiled_classes (
    id INTEGER PRIMARY KEY,
    type_index INTEGER UNIQUE,
    name TEXT NOT NULL,
    parent_indices TEXT,  -- JSON array
    eifgens_path TEXT,
    compiled_at TIMESTAMP
);

CREATE TABLE compiled_attributes (
    id INTEGER PRIMARY KEY,
    class_id INTEGER REFERENCES compiled_classes(id),
    name TEXT NOT NULL,
    type_kind INTEGER,  -- SK_REF=0, SK_BOOL=1, etc.
    type_index INTEGER  -- For reference types
);

-- Index for fast lookups
CREATE INDEX idx_compiled_name ON compiled_classes(name);
CREATE INDEX idx_compiled_type ON compiled_classes(type_index);
```

### LSP Feature Enhancements

| Feature | Parser-Only | With EIFGENs |
|---------|-------------|--------------|
| Go to Definition | Class/feature | + Inherited features |
| Hover | Signature | + Resolved types, full inheritance |
| Completion | Local symbols | + All inherited features |
| Type Hierarchy | Direct parents | + Complete ancestor chain |
| Find References | Text search | + Polymorphic dispatch sites |

## simple_oracle Integration

The Oracle can also benefit from compiled metadata:

### New Oracle Commands

```bash
# Scan a compiled project
oracle-cli.exe scan-compiled /d/prod/simple_json

# Query compiled class info
oracle-cli.exe class-info SIMPLE_JSON

# Show inheritance chain
oracle-cli.exe ancestors SIMPLE_JSON_VALUE
```

### Oracle Knowledge Enhancement

```sql
-- Store compiled metadata in Oracle's knowledge base
INSERT INTO knowledge (type, key, value)
VALUES ('compiled_class', 'SIMPLE_JSON', '{
    "type_index": 563,
    "parents": ["ANY"],
    "features": ["parse", "to_string", ...],
    "attributes": [...]
}');
```

### Cross-Project Analysis

Oracle could aggregate metadata from multiple compiled projects:
- Which classes are most commonly inherited?
- Which features are overridden most often?
- What's the typical inheritance depth?

## Implementation Phases

### Phase 1: Basic C File Parsing (simple_lsp)
- [ ] EIFGENS_METADATA_PARSER class
- [ ] eparents.c parser (class hierarchy)
- [ ] evisib.c parser (class names)
- [ ] Detection of valid EIFGENs folder
- [ ] Timestamp comparison for freshness

### Phase 2: Full Metadata Extraction (simple_lsp)
- [ ] enames.c parser (feature names)
- [ ] eskelet.c parser (attribute types)
- [ ] ecall.c parser (dispatch tables)
- [ ] Database schema extension
- [ ] Hybrid symbol resolution

### Phase 3: Enhanced LSP Features
- [ ] Complete inheritance chain in hover
- [ ] Inherited feature completion
- [ ] Accurate type display for generics
- [ ] Polymorphic reference resolution

### Phase 4: Oracle Integration
- [ ] `scan-compiled` command
- [ ] `class-info` command
- [ ] `ancestors` command
- [ ] Cross-project metadata aggregation

## Technical Considerations

### Parsing Challenges

1. **Generated code variations**: Different EiffelStudio versions may format slightly differently
2. **Generic types**: Type indices for generics involve complex encoding (`0xFF01`, etc.)
3. **Expanded types**: Need special handling for expanded vs reference types
4. **Large projects**: 1000+ classes means large C files to parse

### Performance

- Parse on first access, cache in SQLite
- Incremental updates when single files recompile
- Background parsing during idle time

### Limitations

1. **Requires compilation**: Won't work until `ec.exe -c_compile` succeeds
2. **Windows paths**: C files contain absolute Windows paths
3. **No source positions**: C metadata doesn't preserve line numbers in original .e files
4. **Melted vs finalized**: W_code and F_code may differ

## Alternatives Considered

### Direct project.epr Parsing
- Rejected: Binary format, undocumented, may change between versions

### ISE Library Integration
- Rejected: Would require ISE runtime, adds dependency

### ec.exe -short Output
- Partially useful: Good for individual class info but no batch mode

## VS Code Integration: Compile and Refresh Loop

A key question: How does VS Code trigger compilation and how does simple_lsp know when to refresh metadata?

### The Full Flow

```
User triggers compile (Ctrl+Shift+B or command)
         │
         ▼
    ┌─────────────────┐
    │   ec.exe runs   │──────► Output in VS Code panel
    │   -c_compile    │
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │  EIFGENs/E1/*.c │
    │  files updated  │
    └────────┬────────┘
             │
    ┌────────┴────────┐
    │                 │
    ▼                 ▼
File watcher     LSP notification
(fallback)       (primary)
    │                 │
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │ simple_lsp      │
    │ parses metadata │
    │ updates symbols │
    └─────────────────┘
```

### Implementation Options

**Option 1: VS Code Tasks (tasks.json)**

```json
{
    "version": "2.0.0",
    "tasks": [{
        "label": "Eiffel: Compile",
        "type": "shell",
        "command": "ec.exe",
        "args": ["-batch", "-config", "${workspaceFolder}/project.ecf",
                 "-target", "main", "-c_compile"],
        "group": { "kind": "build", "isDefault": true },
        "problemMatcher": {
            "owner": "eiffel",
            "pattern": {
                "regexp": "^(.+):(\\d+):(\\d+): (.+)$",
                "file": 1, "line": 2, "column": 3, "message": 4
            }
        }
    }]
}
```

**Option 2: Extension Command with LSP Notification**

```typescript
// extension.ts
vscode.commands.registerCommand('eiffel.compile', async () => {
    const output = vscode.window.createOutputChannel('Eiffel');
    output.show();

    const process = spawn('ec.exe', ['-batch', '-config', ecf, '-c_compile']);
    process.stdout.on('data', (data) => output.append(data.toString()));

    process.on('close', (code) => {
        if (code === 0) {
            // Tell LSP to refresh metadata
            client.sendNotification('eiffel/compilationComplete', {
                ecf: ecf, target: target, success: true
            });
        }
    });
});
```

**Option 3: File Watcher (Fallback)**

simple_lsp watches EIFGENs for changes using simple_watcher:

```eiffel
watcher.on_modified (agent (a_path: PATH)
    do
        if a_path.name.ends_with (".c") then
            schedule_metadata_refresh  -- debounced
        end
    end)
```

### Recommended: Hybrid Approach

1. **Primary**: Extension command sends LSP notification after compile
2. **Fallback**: File watcher detects EIFGENs changes (for CLI compiles)
3. **Manual**: Tasks.json for keyboard shortcut (Ctrl+Shift+B)

### LSP Custom Notifications

simple_lsp will handle these notifications:

| Notification | Purpose |
|-------------|---------|
| `eiffel/compilationComplete` | Trigger immediate metadata refresh |
| `eiffel/refreshMetadata` | Manual refresh request |
| `eiffel/setTarget` | Change active compilation target |

### Problem Matcher for Eiffel Errors

VS Code can parse ec.exe output and show errors inline:

```json
"problemMatcher": {
    "owner": "eiffel",
    "fileLocation": "absolute",
    "pattern": [{
        "regexp": "^Error: (.+) line (\\d+) column (\\d+): (.+)$",
        "file": 1, "line": 2, "column": 3, "message": 4
    }]
}
```

## Conclusion

Parsing the generated C files provides a pragmatic path to rich semantic information without requiring ISE library dependencies or reverse-engineering binary formats. The hybrid approach (parser + compiler metadata) gives us the best of both worlds:

- **Parser mode**: Works during active editing, handles broken code
- **Compiler mode**: Provides accurate semantic information after successful build

This design enables features like complete inheritance chains, resolved generic types, and accurate polymorphic dispatch information that would be impossible with parsing alone.

---

*Design document for Simple Eiffel ecosystem*
*https://github.com/simple-eiffel*
