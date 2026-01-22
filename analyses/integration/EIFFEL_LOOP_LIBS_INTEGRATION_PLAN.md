# Eiffel-Loop Libraries Integration Plan

## Date: 2026-01-19

## Overview

Consolidated integration plan for the four new libraries inspired by [Eiffel-Loop](https://github.com/finnianr/Eiffel-Loop):

| Library | Provides |
|---------|----------|
| **simple_encoding** | UTF-8/UTF-32 conversion, codec registry, encoding detection |
| **simple_factory** | Object pools, singletons, lazy initialization, factories |
| **simple_reflection** | Runtime type introspection, field access, object graph walking |
| **simple_zstring** | Memory-efficient strings, escaping, splitting, searching, building |

This plan prioritizes which simple_* libraries should integrate these new capabilities first, based on combined impact across all four.

---

## Priority Scoring

Each library was scored based on how many of the four new libraries could benefit it:
- Tier 1 candidate = 3 points
- Tier 2 candidate = 2 points
- Tier 3 candidate = 1 point

| Target Library | encoding | factory | reflection | zstring | Score | Priority |
|---------------|----------|---------|------------|---------|-------|----------|
| **simple_json** | T1 (3) | - | T1 (3) | T1 (3) | **9** | **#1** |
| **simple_http** | T1 (3) | T1 (3) | - | T2 (2) | **8** | **#2** |
| **simple_xml** | T2 (2) | - | T2 (2) | T1 (3) | **7** | **#3** |
| **simple_sql** | T3 (1) | T1 (3) | T2 (2) | - | **6** | **#4** |
| **simple_csv** | T2 (2) | - | T3 (1) | T1 (3) | **6** | **#5** |
| **simple_template** | T3 (1) | - | T3 (1) | T2 (2) | **4** | **#6** |
| **simple_mock** | - | T3 (1) | T1 (3) | - | **4** | **#7** |
| **simple_email** | T1 (3) | - | - | - | **3** | **#8** |
| **simple_config** | - | T1 (3) | - | - | **3** | **#9** |
| **simple_diff** | - | - | T1 (3) | - | **3** | **#10** |
| **simple_logger** | - | T2 (2) | T3 (1) | - | **3** | **#11** |
| **simple_markdown** | T3 (1) | - | - | T2 (2) | **3** | **#12** |
| **simple_cache** | - | T2 (2) | - | - | **2** | **#13** |
| **simple_mq** | - | T2 (2) | - | - | **2** | **#14** |
| **simple_validation** | - | - | T2 (2) | - | **2** | **#15** |
| **simple_file** | T2 (2) | - | - | - | **2** | **#16** |
| **simple_yaml** | T3 (1) | - | T3 (1) | T3 (1) | **3** | **#17** |
| **simple_toml** | - | - | - | T3 (1) | **1** | **#18** |
| **simple_postgres** | T3 (1) | T3 (1) | - | - | **2** | **#19** |

---

## Implementation Phases

### Phase 1: Core Foundation (Score 7-9)

**Target: simple_json, simple_http, simple_xml**

These three libraries are foundational - many other libraries depend on them. Improvements here cascade through the ecosystem.

#### 1.1 simple_json (Score: 9) - HIGHEST PRIORITY

| New Library | Integration |
|-------------|-------------|
| **simple_encoding** | Replace UTF_CONVERTER with native UTF-8 codec |
| **simple_reflection** | Add `to_json_auto(obj)` for automatic serialization |
| **simple_zstring** | Use escape_json, splitter for JSONPath, builder for output |

**Benefits:**
- Automatic object serialization (no manual field mapping)
- Memory-efficient JSON handling for large documents
- Native UTF-8 without external converter

#### 1.2 simple_http (Score: 8)

| New Library | Integration |
|-------------|-------------|
| **simple_encoding** | Content-Type charset handling, decode responses |
| **simple_factory** | Connection pooling for keep-alive, singleton for shared client |
| **simple_zstring** | URL encoding, query string building |

**Benefits:**
- HTTP connection reuse (performance)
- Proper charset handling for international content
- Clean URL encoding

#### 1.3 simple_xml (Score: 7)

| New Library | Integration |
|-------------|-------------|
| **simple_encoding** | XML encoding declaration support |
| **simple_reflection** | Auto-serialize objects to XML elements |
| **simple_zstring** | Replace manual escape_xml, use builder for element construction |

**Benefits:**
- Automatic object-to-XML mapping
- Consolidated escaping (remove duplicate code)
- Memory efficiency for large XML documents

---

### Phase 2: Data & Persistence (Score 6)

**Target: simple_sql, simple_csv**

#### 2.1 simple_sql (Score: 6)

| New Library | Integration |
|-------------|-------------|
| **simple_factory** | Connection pooling (critical for performance) |
| **simple_reflection** | ORM capabilities: insert_object, select_into_object |
| **simple_encoding** | Database string encoding (UTF-8 storage) |

**Benefits:**
- Connection reuse (major performance gain)
- Reduced boilerplate for CRUD operations
- Proper encoding for international data

#### 2.2 simple_csv (Score: 6)

| New Library | Integration |
|-------------|-------------|
| **simple_zstring** | Replace escape_field, use splitter for parsing, builder for output |
| **simple_encoding** | BOM detection, encoding conversion |
| **simple_reflection** | Map objects to/from CSV rows automatically |

**Benefits:**
- Consolidated escaping
- Handle encoding issues common in CSV files
- Object-to-CSV mapping

---

### Phase 3: Text Processing (Score 3-4)

**Target: simple_template, simple_mock, simple_markdown**

#### 3.1 simple_template (Score: 4)

| New Library | Integration |
|-------------|-------------|
| **simple_zstring** | HTML escaping, substitute for placeholders, builder for output |
| **simple_reflection** | Access object fields in templates: `{{customer.name}}` |
| **simple_encoding** | Template file encoding handling |

#### 3.2 simple_mock (Score: 4)

| New Library | Integration |
|-------------|-------------|
| **simple_reflection** | Dynamic mock generation based on class structure |
| **simple_factory** | Factory pattern for mock creation |

#### 3.3 simple_markdown (Score: 3)

| New Library | Integration |
|-------------|-------------|
| **simple_zstring** | Line splitting, HTML escaping, builder for output |
| **simple_encoding** | Markdown file encoding detection |

---

### Phase 4: Infrastructure (Score 3)

**Target: simple_email, simple_config, simple_diff, simple_logger**

#### 4.1 simple_email (Score: 3)

| New Library | Integration |
|-------------|-------------|
| **simple_encoding** | MIME charset handling, header encoding |

#### 4.2 simple_config (Score: 3)

| New Library | Integration |
|-------------|-------------|
| **simple_factory** | Singleton pattern for global configuration |

#### 4.3 simple_diff (Score: 3)

| New Library | Integration |
|-------------|-------------|
| **simple_reflection** | Deep object comparison by field |

#### 4.4 simple_logger (Score: 3)

| New Library | Integration |
|-------------|-------------|
| **simple_factory** | Singleton for global logger |
| **simple_reflection** | Debug object dumping: `log.debug_object(customer)` |

---

### Phase 5: Supporting Libraries (Score 1-2)

**Target: simple_cache, simple_mq, simple_validation, simple_file, simple_yaml, simple_toml, simple_postgres**

Lower priority - integrate as needed or when touching these libraries for other reasons.

| Library | Primary Integration |
|---------|---------------------|
| simple_cache | factory (singleton) |
| simple_mq | factory (connection pool) |
| simple_validation | reflection (generic field validation) |
| simple_file | encoding (read/write with encoding) |
| simple_yaml | zstring (splitting), reflection (auto-serialize) |
| simple_toml | zstring (splitting, escaping) |
| simple_postgres | factory (connection pool), encoding |

---

## Dependency Graph

```
                    simple_encoding
                          │
    ┌─────────────────────┼─────────────────────┐
    │                     │                     │
    ▼                     ▼                     ▼
simple_zstring      simple_json           simple_http
    │                     │                     │
    │              ┌──────┴──────┐              │
    │              ▼             ▼              │
    └──────► simple_xml    simple_csv ◄────────┘
                  │              │
                  ▼              ▼
             simple_template  simple_sql ◄── simple_factory
                                              simple_reflection
```

**Note:** simple_zstring already depends on simple_encoding.

---

## Quick Reference: What Each New Library Brings

### simple_encoding
```eiffel
-- Convert file content to STRING_32
encoding: SIMPLE_ENCODING
content_32 := encoding.utf8_to_utf32 (file_bytes)
```

### simple_factory
```eiffel
-- Connection pooling
pool: DATABASE_POOL
pool.with_item (agent (conn) do conn.execute (...) end)

-- Singleton
config := (create {APP_CONFIG}).instance
```

### simple_reflection
```eiffel
-- Auto-serialize any object
json := serializer.to_json (customer)  -- no manual field mapping

-- Deep compare
differences := differ.compare (old_obj, new_obj)
```

### simple_zstring
```eiffel
-- Escape for any format
escaper: SIMPLE_ZSTRING_ESCAPER
xml_safe := escaper.escape_xml (input)
json_safe := escaper.escape_json (input)
url_safe := escaper.url_encode (input)

-- Build strings fluently
builder.append ("<root>").append (content).append ("</root>")
```

---

## Success Metrics

After integration:

1. **Code reduction** - Measure lines of code removed (duplicate escapers, manual serialization)
2. **Performance** - Benchmark connection pooling impact on simple_sql
3. **Memory** - Measure memory usage for large JSON/XML with simple_zstring
4. **Developer experience** - Count eliminated boilerplate in common patterns

---

## Next Steps

1. **Start with simple_json** - highest impact, foundational
2. **Document patterns** - Create integration examples for each combination
3. **Update ECF files** - Add new dependencies as libraries are integrated
4. **Run benchmarks** - Validate performance claims before/after

---

*Consolidated analysis from individual integration opportunity documents*
*Libraries inspired by [Eiffel-Loop](https://github.com/finnianr/Eiffel-Loop)*
