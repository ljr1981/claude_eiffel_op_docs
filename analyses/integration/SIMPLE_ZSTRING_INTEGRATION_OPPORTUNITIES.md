# simple_zstring Integration Opportunities

## Date: 2026-01-19

## Overview

Analysis of simple_* libraries that could benefit from integrating simple_zstring for memory-efficient Unicode string handling, string escaping, splitting, searching, and fluent string building.

## What simple_zstring Provides

- **Memory-efficient strings** - Dual-storage architecture (8-bit primary + 32-bit overflow)
- **UTF-8 codec** - Native encoding/decoding (make_from_utf_8, to_utf_8)
- **String escaping** - XML, JSON, URL, CSV, shell escaping/unescaping
- **String splitting** - By character, string, lines, words
- **String searching** - Index finding, wildcards, prefix/suffix matching
- **String formatting** - Padding, centering, template substitution, joining
- **Fluent builder** - Chainable string construction

## Current Usage

Only simple_zstring itself - **no other libraries currently use it**.

## High Value Candidates

### Serialization Libraries (HIGH PRIORITY)

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_json** | Uses UTF_CONVERTER, manual escaping | Replace UTF_CONVERTER with native UTF-8 codec. Use escape_json for output. Use splitter for JSONPath parsing. |
| **simple_xml** | Manual XML escaping (lines 333-379) | Replace escape_xml implementation. Use builder for element construction. |
| **simple_csv** | Field escaping, parsing, BOM handling | Replace escape_field with escape_csv. Use splitter for parsing. Use builder for output. |
| **simple_template** | HTML escaping, variable substitution | Use escape_xml for HTML. Use substitute for placeholders. Use builder for output. |

**Example Integration (simple_json):**
```eiffel
-- Current: manual UTF conversion
converter: UTF_CONVERTER
result_8 := converter.utf_32_string_to_utf_8_string_8 (json_32)

-- With simple_zstring: native UTF-8
zstr: SIMPLE_ZSTRING
create zstr.make_from_string (json_32)
result_8 := zstr.to_utf_8

-- Escaping JSON strings
escaper: SIMPLE_ZSTRING_ESCAPER
escaped := escaper.escape_json (raw_value)
```

### Markup Libraries (MEDIUM-HIGH PRIORITY)

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_markdown** | Line splitting, HTML escaping | Use splitter for line/token parsing. Use escaper for code blocks. Use builder for HTML output. |
| **simple_toml** | Tokenization, escape sequences | Use splitter for key-value parsing. Use escaper for string literals. |
| **simple_yaml** | Line parsing, string handling | Use splitter for indentation-aware parsing. Use builder for output. |

**Example Integration (simple_markdown):**
```eiffel
-- Split markdown into lines
splitter: SIMPLE_ZSTRING_SPLITTER
lines := splitter.split_lines (markdown_content)

-- Escape code block content for HTML
escaper: SIMPLE_ZSTRING_ESCAPER
html_safe := escaper.escape_xml (code_content)

-- Build HTML output
builder: SIMPLE_ZSTRING_BUILDER
builder.append ("<pre><code>").append (html_safe).append ("</code></pre>")
result := builder.to_string_32
```

### Network Libraries (MEDIUM PRIORITY)

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_http** | URL query building, encoding | Use url_encode for query parameters. Use builder for request construction. |
| **simple_email** | Header encoding, body handling | Use escape functions for header values. Memory efficiency for large bodies. |

**Example Integration (simple_http):**
```eiffel
-- URL encode query parameters
escaper: SIMPLE_ZSTRING_ESCAPER
encoded_value := escaper.url_encode (param_value)

-- Build query string
formatter: SIMPLE_ZSTRING_FORMATTER
query := formatter.join (params, "&")
```

### Logging/Monitoring Libraries (LOWER PRIORITY)

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_logger** | Field formatting, message building | Use builder for log message construction. Performance benefit in hot paths. |
| **simple_i18n** | Locale formatting, substitution | Use substitute for message parameters. Use formatter for locale-aware output. |
| **simple_base64** | MIME line wrapping | Use builder for wrapped output. |

## Escaping Consolidation Opportunity

**Four libraries have their own escape implementations:**

| Library | Current Escaper | simple_zstring Replacement |
|---------|----------------|---------------------------|
| simple_xml | `escape_xml` (lines 333-348) | `SIMPLE_ZSTRING_ESCAPER.escape_xml` |
| simple_json | Manual escaping | `SIMPLE_ZSTRING_ESCAPER.escape_json` |
| simple_csv | `escape_field` (lines 824-847) | `SIMPLE_ZSTRING_ESCAPER.escape_csv` |
| simple_http | URL encoding | `SIMPLE_ZSTRING_ESCAPER.url_encode` |

Consolidating to simple_zstring eliminates code duplication and ensures consistent escaping behavior.

## Priority Recommendations

### Tier 1 (Highest Impact)

1. **simple_json** - Foundational library. Remove UTF_CONVERTER dependency. Many libraries depend on it.
2. **simple_xml** - Core library. Heavy escaping needs. Replace manual escape implementation.
3. **simple_csv** - String-intensive. Parsing and generation both benefit.

### Tier 2 (High Impact)

4. **simple_template** - Text-heavy templating. Escaping + substitution + builder.
5. **simple_markdown** - Large document processing. Splitting + escaping + builder.
6. **simple_http** - URL encoding. Query string building.

### Tier 3 (Medium Impact)

7. **simple_toml** - Config file parsing. Splitting + escaping.
8. **simple_yaml** - Config file parsing. Splitting + builder.
9. **simple_encoding** - Coordination opportunity (zstring uses encoding internally).

### Tier 4 (Lower Impact)

10. **simple_logger** - Performance optimization in logging hot paths.
11. **simple_i18n** - Formatting and substitution.
12. **simple_base64** - Builder for MIME output.

## Implementation Notes

### Adding simple_zstring Dependency

```xml
<library name="simple_zstring" location="$SIMPLE_LIBS/simple_zstring/simple_zstring.ecf"/>
```

### Key Classes to Use

- `SIMPLE_ZSTRING` - Main dual-storage string class
- `SIMPLE_ZSTRING_SPLITTER` - Split by character, string, lines, words
- `SIMPLE_ZSTRING_SEARCHER` - Find patterns, wildcards, prefix/suffix
- `SIMPLE_ZSTRING_ESCAPER` - XML, JSON, URL, CSV, shell escaping
- `SIMPLE_ZSTRING_FORMATTER` - Padding, centering, substitution, joining
- `SIMPLE_ZSTRING_BUILDER` - Fluent chainable string construction

### Common Patterns

```eiffel
-- Memory-efficient string storage
z: SIMPLE_ZSTRING
create z.make_from_string ("Western European text with euro sign")
-- Uses ~70% less memory than STRING_32 for Latin text

-- UTF-8 round-trip
create z.make_from_utf_8 (file_bytes)
processed := do_something (z.to_string_32)
output_bytes := z.to_utf_8

-- Escape for multiple formats
escaper: SIMPLE_ZSTRING_ESCAPER
xml_safe := escaper.escape_xml (user_input)
json_safe := escaper.escape_json (user_input)
url_safe := escaper.url_encode (user_input)

-- Split and process
splitter: SIMPLE_ZSTRING_SPLITTER
lines := splitter.split_lines (content)
words := splitter.split_words (line)

-- Build output efficiently
builder: SIMPLE_ZSTRING_BUILDER
builder.append ("<root>")
       .append_new_line
       .append ("  <item>")
       .append (escaped_content)
       .append ("</item>")
       .append_new_line
       .append ("</root>")
result := builder.to_string_32
```

## Next Steps

1. Start with simple_json integration - foundational, most dependents benefit
2. Replace manual escape implementations in simple_xml and simple_csv
3. Add builder pattern to simple_template and simple_markdown
4. Roll out to remaining text-processing libraries

---

*Analysis generated during ecosystem integration review*
*Inspired by ZSTRING from [Eiffel-Loop](https://github.com/finnianr/Eiffel-Loop)*
