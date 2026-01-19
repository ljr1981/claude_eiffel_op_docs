# simple_encoding Integration Opportunities

## Date: 2026-01-19

## Overview

Analysis of simple_* libraries that could benefit from integrating simple_encoding for character encoding conversion, UTF-8/UTF-32 handling, and encoding detection.

## What simple_encoding Provides

- **UTF-8 <-> UTF-32** - Bidirectional conversion with proper error handling
- **Single-byte codecs** - ISO-8859-1 (Latin-1), ISO-8859-15 (Latin-9), Windows-1252
- **Encoding detection** - BOM detection and heuristic analysis
- **Character properties** - Unicode character classification
- **Codec registry** - Extensible codec lookup by name

## Current Usage

Only simple_encoding itself - **no other libraries currently use it**.

## High Value Candidates

### Serialization Libraries

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_json** | 2 src, 16 tests | JSON spec requires UTF-8. Use encoding for proper string handling, escape sequences. |
| **simple_xml** | 5 src, 2 tests | XML has encoding declarations. Detect and convert between encodings. |
| **simple_yaml** | 1 src, 2 tests | YAML is UTF-8. Ensure proper encoding on read/write. |
| **simple_csv** | 2 src, 2 tests | CSV files often have encoding issues. Detect encoding, convert to UTF-8. |

**Example Integration (simple_json):**
```eiffel
-- Ensure JSON output is valid UTF-8
json_string := encoding.to_utf8 (json.to_string_32)

-- Parse JSON with proper encoding
content_32 := encoding.from_utf8 (file_content)
json.parse (content_32)
```

### Network Libraries

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_http** | 8 src, 3 tests | HTTP Content-Type charset handling. Encode request bodies, decode responses. |
| **simple_email** | 5 src, 3 tests | Email has MIME encoding. Handle charset in headers, encode/decode bodies. |
| **simple_smtp** | 2 src, 2 tests | SMTP requires encoding for non-ASCII. Use for subject, body encoding. |
| **simple_websocket** | 4 src, 2 tests | WebSocket text frames are UTF-8. Validate and encode messages. |
| **simple_grpc** | 11 src, 2 tests | gRPC uses UTF-8 strings. Ensure proper encoding. |

**Example Integration (simple_http):**
```eiffel
-- Decode response based on Content-Type charset
charset := response.header ("Content-Type").charset  -- e.g., "iso-8859-1"
codec := encoding.codec_for_name (charset)
body_32 := codec.decode (response.body)
```

### Text Processing Libraries

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_template** | 2 src, 4 tests | Templates may have different encodings. Normalize to UTF-32 internally. |
| **simple_markdown** | 4 src, 2 tests | Markdown files may have various encodings. Detect and convert. |
| **simple_i18n** | 1 src, 3 tests | Internationalization requires proper encoding handling. |

### File/Database Libraries

| Library | Current State | Integration Opportunity |
|---------|--------------|------------------------|
| **simple_file** | 3 src, 3 tests | Read/write files with encoding specification. `read_text (path, "utf-8")`. |
| **simple_sql** | 44 src, 28 tests | Database strings may have different encodings. Ensure UTF-8 for storage. |
| **simple_postgres** | 7 src, 3 tests | PostgreSQL encoding handling. Convert client encoding. |

## Priority Recommendations

### Tier 1 (Highest Impact)

1. **simple_http** - HTTP is encoding-heavy. Content-Type charset, request/response encoding.
2. **simple_json** - JSON must be UTF-8. Critical for proper serialization.
3. **simple_email** - Email encoding is complex. MIME charset handling essential.

### Tier 2 (High Impact)

4. **simple_file** - File encoding is a common pain point. Explicit encoding support.
5. **simple_xml** - XML encoding declaration support.
6. **simple_csv** - CSV encoding detection would solve many real-world issues.

### Tier 3 (Medium Impact)

7. **simple_template** - Template file encoding.
8. **simple_sql** - Database string encoding.
9. **simple_markdown** / **simple_i18n** - Text processing encoding.

## Implementation Notes

### Adding simple_encoding Dependency

```xml
<library name="simple_encoding" location="$SIMPLE_LIBS/simple_encoding/simple_encoding.ecf"/>
```

### Key Classes to Use

- `SIMPLE_ENCODING` - Main UTF-8/UTF-32 conversion
- `SIMPLE_CODEC_REGISTRY` - Look up codecs by name
- `SIMPLE_ENCODING_DETECTOR` - Detect file/data encoding
- `SIMPLE_CODEC` - Base class for single-byte codecs

### Common Patterns

```eiffel
-- Convert file content to STRING_32
encoding: SIMPLE_ENCODING
content_8: STRING_8  -- raw file bytes
content_32: STRING_32

create encoding.make
if encoding.is_valid_utf8 (content_8) then
    content_32 := encoding.utf8_to_utf32 (content_8)
else
    -- Try to detect encoding
    detector: SIMPLE_ENCODING_DETECTOR
    create detector.make
    if attached detector.detect (content_8) as enc_name then
        codec := registry.codec_for_name (enc_name)
        content_32 := codec.decode (content_8)
    end
end
```

## Next Steps

1. Start with simple_http integration - most practical impact
2. Add encoding parameter to simple_file read/write operations
3. Ensure simple_json validates UTF-8 on output
4. Roll out to other text-processing libraries

---

*Analysis generated during ecosystem integration review*
