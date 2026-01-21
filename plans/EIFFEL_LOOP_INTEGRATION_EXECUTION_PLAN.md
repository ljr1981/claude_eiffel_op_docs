# Eiffel-Loop Libraries Integration Execution Plan

## Date: 2026-01-19

## Overview

Execution plan for integrating the four Eiffel-Loop inspired libraries into the top 10 candidate libraries, following the anti-slop workflow system.

### New Libraries to Integrate

| Library | Capabilities |
|---------|--------------|
| **simple_encoding** | UTF-8/UTF-32 conversion, codec registry, encoding detection |
| **simple_factory** | Object pools, singletons, lazy initialization |
| **simple_reflection** | Runtime introspection, field access, object graphs |
| **simple_zstring** | Memory-efficient strings, escaping, splitting, building |

### Target Libraries (Priority Order)

| # | Library | Score | Integrates |
|---|---------|-------|------------|
| 1 | simple_json | 9 | encoding, reflection, zstring |
| 2 | simple_http | 8 | encoding, factory, zstring |
| 3 | simple_xml | 7 | encoding, reflection, zstring |
| 4 | simple_sql | 6 | encoding, factory, reflection |
| 5 | simple_csv | 6 | encoding, reflection, zstring |
| 6 | simple_template | 4 | encoding, reflection, zstring |
| 7 | simple_mock | 4 | factory, reflection |
| 8 | simple_email | 3 | encoding |
| 9 | simple_config | 3 | factory |
| 10 | simple_diff | 3 | reflection |

---

## Workflow Per Library

Each library integration follows this sequence:

```
┌──────────────────────────────────────────────────────────────────┐
│  PHASE 1: DESIGN AUDIT (Workflow 05)                             │
│  Purpose: Plan how to integrate without breaking existing API    │
├──────────────────────────────────────────────────────────────────┤
│  D01-STRUCTURE-ANALYSIS    Map current dependencies              │
│  D02-SMELL-DETECTION       Find integration points               │
│  D03-INHERITANCE-AUDIT     Check if composition needed           │
│  D04-GENERICITY-SCAN       Identify type parameter needs         │
│  D05-REFACTOR-PLAN         Plan integration changes              │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  PHASE 2: MAINTENANCE (Workflow 06)                              │
│  Purpose: Execute integration, strengthen contracts              │
├──────────────────────────────────────────────────────────────────┤
│  M01-AUDIT-CONTRACTS       Audit existing contracts              │
│  M02-AUDIT-STRUCTURE       Check integration points              │
│  M03-AUDIT-VOID-SAFETY     Verify void safety maintained         │
│  M04-M06                   Add new library usage + contracts     │
│  M07-COMPILE-VALIDATE      Verify compilation                    │
│  M08-TEST-NEW-CONTRACTS    Run all tests                         │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  PHASE 3: BUG HUNTING + FIXING (Workflows 08-09)                 │
│  Purpose: Find and fix any integration issues                    │
├──────────────────────────────────────────────────────────────────┤
│  B01-B04                   Hunt for bugs from integration        │
│  F01-F06                   Fix any discovered bugs               │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│  PHASE 4: NAMING + GITHUB (Workflows 10-11)                      │
│  Purpose: Finalize naming, update docs, commit                   │
├──────────────────────────────────────────────────────────────────┤
│  N01-N04                   Review naming conventions             │
│  G01-G03                   Update README, docs, commit           │
└──────────────────────────────────────────────────────────────────┘
```

---

## Library 1: simple_json (Score: 9)

### Integration Goals

| New Library | What to Integrate | Benefit |
|-------------|-------------------|---------|
| **simple_encoding** | Replace UTF_CONVERTER | Native UTF-8, fewer dependencies |
| **simple_reflection** | Add `to_json_auto(obj)` | Automatic serialization |
| **simple_zstring** | Use escape_json, splitter | Consolidated escaping, JSONPath parsing |

### Specific Changes

```eiffel
-- 1. Add ECF dependencies
<library name="simple_encoding" location="$SIMPLE_LIBS/simple_encoding/simple_encoding.ecf"/>
<library name="simple_reflection" location="$SIMPLE_LIBS/simple_reflection/simple_reflection.ecf"/>
<library name="simple_zstring" location="$SIMPLE_LIBS/simple_zstring/simple_zstring.ecf"/>

-- 2. Replace UTF_CONVERTER usage in SIMPLE_JSON
-- BEFORE:
converter: UTF_CONVERTER
result := converter.utf_32_string_to_utf_8_string_8 (json_32)

-- AFTER:
zstr: SIMPLE_ZSTRING
create zstr.make_from_string (json_32)
result := zstr.to_utf_8

-- 3. Add auto-serialization feature
class SIMPLE_JSON_SERIALIZER
feature
    to_json (a_object: ANY): SIMPLE_JSON_OBJECT
        -- Auto-serialize any object using reflection
    from_json (a_json: SIMPLE_JSON_OBJECT; a_type: TYPE [ANY]): ANY
        -- Auto-deserialize to object

-- 4. Use escape_json from zstring
escaper: SIMPLE_ZSTRING_ESCAPER
escaped := escaper.escape_json (raw_string)
```

### Verification Criteria

- [ ] Compiles with new dependencies
- [ ] All existing 16+ tests still pass
- [ ] New auto-serialization tests pass
- [ ] UTF-8 round-trip tests pass
- [ ] Memory usage comparable or better

---

## Library 2: simple_http (Score: 8)

### Integration Goals

| New Library | What to Integrate | Benefit |
|-------------|-------------------|---------|
| **simple_encoding** | Content-Type charset handling | Proper international response decoding |
| **simple_factory** | Connection pooling, singleton client | Performance, keep-alive reuse |
| **simple_zstring** | URL encoding, query building | Clean URL handling |

### Specific Changes

```eiffel
-- 1. Add HTTP connection pool
class HTTP_CONNECTION_POOL inherit SIMPLE_OBJECT_POOL [HTTP_CONNECTION]
feature
    create_item: HTTP_CONNECTION
    reset_item (conn: HTTP_CONNECTION)
        do conn.reset_for_reuse end

-- 2. Add shared HTTP client singleton
class SHARED_HTTP_CLIENT inherit SIMPLE_SHARED_SINGLETON [SIMPLE_HTTP_CLIENT]

-- 3. Decode responses by charset
feature decode_response (resp: HTTP_RESPONSE): STRING_32
    local
        charset: STRING
        codec: SIMPLE_CODEC
    do
        charset := resp.content_type_charset
        codec := encoding_registry.codec_for_name (charset)
        Result := codec.decode (resp.body)
    end

-- 4. URL encode with zstring
escaper: SIMPLE_ZSTRING_ESCAPER
encoded_param := escaper.url_encode (param_value)
```

### Verification Criteria

- [ ] Compiles with new dependencies
- [ ] All existing tests pass
- [ ] Connection pool tests pass
- [ ] Charset decoding tests pass
- [ ] URL encoding tests pass

---

## Library 3: simple_xml (Score: 7)

### Integration Goals

| New Library | What to Integrate | Benefit |
|-------------|-------------------|---------|
| **simple_encoding** | XML encoding declaration | Proper document encoding |
| **simple_reflection** | Auto-serialize objects to XML | Reduce boilerplate |
| **simple_zstring** | Replace manual escape_xml | Consolidated escaping, builder |

### Specific Changes

```eiffel
-- 1. Remove manual escape_xml implementation (lines 333-379)
-- 2. Use SIMPLE_ZSTRING_ESCAPER.escape_xml instead
escaper: SIMPLE_ZSTRING_ESCAPER
escaped := escaper.escape_xml (content)

-- 3. Use SIMPLE_ZSTRING_BUILDER for element construction
builder: SIMPLE_ZSTRING_BUILDER
builder.append ("<").append (tag_name).append (">")
       .append (escaped_content)
       .append ("</").append (tag_name).append (">")

-- 4. Add XML serializer using reflection
class SIMPLE_XML_SERIALIZER
feature
    to_xml (a_object: ANY): SIMPLE_XML_ELEMENT
    from_xml (a_xml: SIMPLE_XML_ELEMENT; a_type: TYPE [ANY]): ANY
```

### Verification Criteria

- [ ] Compiles with new dependencies
- [ ] All existing tests pass
- [ ] Escaping behavior unchanged (regression tests)
- [ ] Auto-serialization tests pass
- [ ] Builder output matches manual output

---

## Library 4: simple_sql (Score: 6)

### Integration Goals

| New Library | What to Integrate | Benefit |
|-------------|-------------------|---------|
| **simple_encoding** | UTF-8 string handling | International data support |
| **simple_factory** | Connection pooling | Major performance gain |
| **simple_reflection** | ORM capabilities | Reduced CRUD boilerplate |

### Specific Changes

```eiffel
-- 1. Add connection pool
class DATABASE_CONNECTION_POOL inherit SIMPLE_OBJECT_POOL [DATABASE_CONNECTION]
feature
    create_item: DATABASE_CONNECTION
        do create Result.make (connection_string) end
    reset_item (conn: DATABASE_CONNECTION)
        do conn.rollback_if_active end

-- 2. Add pool usage
pool.with_item (agent (conn: DATABASE_CONNECTION)
    do
        conn.execute ("SELECT * FROM users")
    end)

-- 3. Add ORM features
class SIMPLE_SQL_ORM
feature
    insert_object (a_table: STRING; a_object: ANY)
        -- Reflect fields to columns
    select_into (a_query: STRING; a_type: TYPE [ANY]): LIST [ANY]
        -- Query results to objects
    update_object (a_table: STRING; a_object: ANY; a_where: STRING)
```

### Verification Criteria

- [ ] Compiles with new dependencies
- [ ] All existing 28 tests pass
- [ ] Connection pool tests pass
- [ ] ORM insert/select/update tests pass

---

## Library 5: simple_csv (Score: 6)

### Integration Goals

| New Library | What to Integrate | Benefit |
|-------------|-------------------|---------|
| **simple_encoding** | BOM detection, encoding conversion | Handle encoding issues |
| **simple_reflection** | Object-to-row mapping | Auto CSV serialization |
| **simple_zstring** | Replace escape_field, use splitter | Consolidated escaping |

### Specific Changes

```eiffel
-- 1. Remove manual escape_field (lines 824-847)
-- 2. Use SIMPLE_ZSTRING_ESCAPER
escaper: SIMPLE_ZSTRING_ESCAPER
escaped := escaper.escape_csv (field_value)

-- 3. Use splitter for parsing
splitter: SIMPLE_ZSTRING_SPLITTER
fields := splitter.split_by_character (line, ',')

-- 4. Use encoding for BOM detection
detector: SIMPLE_ENCODING_DETECTOR
encoding := detector.detect (file_bytes)

-- 5. Add object mapping
class SIMPLE_CSV_MAPPER [G]
feature
    to_row (a_object: G): STRING
    from_row (a_row: STRING): G
```

### Verification Criteria

- [ ] Compiles with new dependencies
- [ ] All existing tests pass
- [ ] BOM detection tests pass
- [ ] Object mapping tests pass

---

## Library 6: simple_template (Score: 4)

### Integration Goals

| New Library | What to Integrate | Benefit |
|-------------|-------------------|---------|
| **simple_encoding** | Template file encoding | Handle international templates |
| **simple_reflection** | Object field access in templates | `{{object.field}}` syntax |
| **simple_zstring** | HTML escaping, substitute, builder | Consolidated escaping |

### Specific Changes

```eiffel
-- 1. Use zstring escaper for HTML
escaper: SIMPLE_ZSTRING_ESCAPER
escaped := escaper.escape_xml (content)  -- XML escape = HTML escape

-- 2. Use zstring substitute for placeholders
formatter: SIMPLE_ZSTRING_FORMATTER
result := formatter.substitute (template, values)

-- 3. Use reflection for object field access
-- Template: "Hello {{customer.name}}"
reflected: SIMPLE_REFLECTED_OBJECT
create reflected.make (customer)
name := reflected.field_value ("name")
```

### Verification Criteria

- [ ] Compiles with new dependencies
- [ ] All existing 4 tests pass
- [ ] Object field access tests pass
- [ ] HTML escaping tests pass

---

## Library 7: simple_mock (Score: 4)

### Integration Goals

| New Library | What to Integrate | Benefit |
|-------------|-------------------|---------|
| **simple_factory** | Mock factory patterns | Type-safe mock creation |
| **simple_reflection** | Dynamic mock generation | Auto-mock from class structure |

### Specific Changes

```eiffel
-- 1. Add mock factory
class MOCK_FACTORY [G] inherit SIMPLE_FACTORY [G]
feature
    create_mock: G
        -- Create mock implementing G's interface

-- 2. Use reflection for auto-mocking
class SIMPLE_AUTO_MOCK
feature
    mock_for (a_type: TYPE [ANY]): ANY
        -- Create mock that tracks all calls
    verify_called (a_mock: ANY; a_feature: STRING): BOOLEAN
    verify_called_with (a_mock: ANY; a_feature: STRING; a_args: TUPLE): BOOLEAN
```

### Verification Criteria

- [ ] Compiles with new dependencies
- [ ] All existing tests pass
- [ ] Auto-mock creation tests pass
- [ ] Call verification tests pass

---

## Library 8: simple_email (Score: 3)

### Integration Goals

| New Library | What to Integrate | Benefit |
|-------------|-------------------|---------|
| **simple_encoding** | MIME charset handling | Proper header/body encoding |

### Specific Changes

```eiffel
-- 1. Use encoding for MIME headers
feature encode_header (a_value: STRING_32): STRING_8
    -- RFC 2047 encoded-word
    local
        encoding: SIMPLE_ENCODING
    do
        create encoding.make
        if encoding.is_ascii (a_value) then
            Result := a_value.to_string_8
        else
            Result := "=?UTF-8?B?" + base64_encode (encoding.utf32_to_utf8 (a_value)) + "?="
        end
    end

-- 2. Use encoding for body charset
feature set_body_charset (a_charset: STRING)
    -- Set Content-Type charset parameter
```

### Verification Criteria

- [ ] Compiles with new dependencies
- [ ] All existing tests pass
- [ ] MIME encoding tests pass
- [ ] International subject/body tests pass

---

## Library 9: simple_config (Score: 3)

### Integration Goals

| New Library | What to Integrate | Benefit |
|-------------|-------------------|---------|
| **simple_factory** | Singleton pattern | Global configuration access |

### Specific Changes

```eiffel
-- 1. Make config a singleton
class APP_CONFIG inherit SIMPLE_SINGLETON [APP_CONFIG]
feature
    database_url: STRING
        do Result := get ("database.url") end

    api_key: STRING
        do Result := get ("api.key") end

feature {NONE}
    cached_data: SIMPLE_CACHED_VALUE [CONFIG_DATA]
        once
            create Result.make (agent load_config)
        end

    load_config: CONFIG_DATA
        do
            create Result.make_from_file ("config.yaml")
        end
end

-- Usage:
config := (create {APP_CONFIG}).instance
url := config.database_url
```

### Verification Criteria

- [ ] Compiles with new dependencies
- [ ] All existing tests pass
- [ ] Singleton pattern tests pass
- [ ] Cached value tests pass

---

## Library 10: simple_diff (Score: 3)

### Integration Goals

| New Library | What to Integrate | Benefit |
|-------------|-------------------|---------|
| **simple_reflection** | Deep object comparison | Compare any objects by field |

### Specific Changes

```eiffel
-- 1. Add object differ
class SIMPLE_OBJECT_DIFFER
feature
    compare (a_old, a_new: ANY): LIST [FIELD_DIFFERENCE]
        -- Compare all fields of two objects
        local
            reflected_old, reflected_new: SIMPLE_REFLECTED_OBJECT
        do
            create reflected_old.make (a_old)
            create reflected_new.make (a_new)
            create Result.make (10)
            across reflected_old.field_names as name loop
                if not reflected_old.field_value (name).is_equal (
                       reflected_new.field_value (name)) then
                    Result.extend (create {FIELD_DIFFERENCE}.make (
                        name,
                        reflected_old.field_value (name),
                        reflected_new.field_value (name)))
                end
            end
        end

class FIELD_DIFFERENCE
feature
    field_name: STRING
    old_value: ANY
    new_value: ANY
```

### Verification Criteria

- [ ] Compiles with new dependencies
- [ ] All existing tests pass
- [ ] Object comparison tests pass
- [ ] Nested object comparison tests pass

---

## Execution Schedule

### Sprint 1: Foundation (Libraries 1-3)

| Week | Library | Focus |
|------|---------|-------|
| 1 | simple_json | Design audit + Integration |
| 2 | simple_json | Testing + Bug fixing |
| 3 | simple_http | Design audit + Integration |
| 4 | simple_http | Testing + Bug fixing |
| 5 | simple_xml | Design audit + Integration |
| 6 | simple_xml | Testing + Bug fixing |

### Sprint 2: Data Layer (Libraries 4-5)

| Week | Library | Focus |
|------|---------|-------|
| 7 | simple_sql | Connection pooling + ORM |
| 8 | simple_sql | Testing + Bug fixing |
| 9 | simple_csv | Escaping + Mapping |
| 10 | simple_csv | Testing + Bug fixing |

### Sprint 3: Supporting (Libraries 6-10)

| Week | Library | Focus |
|------|---------|-------|
| 11 | simple_template | Integration |
| 12 | simple_mock | Integration |
| 13 | simple_email, simple_config | Integration |
| 14 | simple_diff | Integration |
| 15 | All | Final testing + Documentation |

---

## Anti-Slop Checkpoints

### Per-Library Gate

Before marking any library integration complete:

```
[ ] Read 00_ANTI-SLOP-RULES.md
[ ] Executed design audit workflow (D01-D05)
[ ] Executed maintenance workflow (M01-M08)
[ ] Compiled after EVERY code change
[ ] Ran tests after EVERY change
[ ] Pasted ACTUAL compiler output
[ ] Pasted ACTUAL test output
[ ] All original tests still pass
[ ] New integration tests pass
[ ] Updated README with new dependencies
[ ] Committed and pushed
```

### Evidence Required

| Claim | Evidence |
|-------|----------|
| "Integration compiles" | Paste ec.sh output showing "System Recompiled" |
| "Tests pass" | Paste test output showing pass counts |
| "No regressions" | Show original test count matches |
| "New features work" | Show new tests passing |

---

## Success Metrics

After all integrations complete:

1. **Code reduction**: Measure LOC removed (duplicate escapers, converters)
2. **Performance**: Benchmark connection pooling, memory usage
3. **API consistency**: Same patterns across all libraries
4. **Test coverage**: Each new feature has tests
5. **Documentation**: All READMEs updated with new dependencies

---

## Quick Reference: Workflow Commands

```bash
# Start any library integration
READ: D:\prod\reference_docs\research\AI-slop\prompts\00_ANTI-SLOP-RULES.md

# Design Audit
READ: D:\prod\reference_docs\research\AI-slop\prompts\05_design-audit\D01-STRUCTURE-ANALYSIS.md
# ... through D05

# Maintenance
READ: D:\prod\reference_docs\research\AI-slop\prompts\06_maintenance\M01-AUDIT-CONTRACTS.md
# ... through M08

# Compile
/d/prod/ec.sh -batch -config {lib}.ecf -target {lib}_tests -c_compile

# Test
./EIFGENs/{lib}_tests/W_code/{lib}.exe

# Bug Hunting/Fixing
READ: D:\prod\reference_docs\research\AI-slop\prompts\08_bug-hunting\B01-*.md
READ: D:\prod\reference_docs\research\AI-slop\prompts\09_bug-fixing\F01-*.md

# Naming Review
READ: D:\prod\reference_docs\research\AI-slop\prompts\10_naming-review\N01-*.md

# GitHub Prep
READ: D:\prod\reference_docs\research\AI-slop\prompts\11_github-prep\G01-*.md
```

---

*Plan generated from EIFFEL_LOOP_LIBS_INTEGRATION_PLAN.md analysis*
*Following anti-slop workflow system*
