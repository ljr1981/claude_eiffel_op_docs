# simple_toon Research Notes

## Step 1: Specifications

### Formal TOON Definition
- **TOON (Token-Oriented Object Notation)**: A line-oriented, indentation-based text format that encodes the JSON data model with explicit structure and minimal quoting
- **Version**: 3.0 (2025-11-24) - Working Draft
- **Media Type**: `text/toon` (provisional, pending IANA registration)
- **File Extension**: `.toon`
- **Encoding**: UTF-8 (always)

### Core Design Goals
1. Minimize tokens for LLM input
2. Preserve JSON data model (lossless round-trip)
3. Human-readable
4. Schema-aware (explicit array lengths, field declarations)

### Syntax Rules

**Primitives:**
- Strings: Unquoted unless containing special chars
- Numbers: Decimal form, no exponents (1e6 → 1000000)
- Booleans: `true`, `false`
- Null: `null`

**Objects:**
```
key: value
nested:
  inner_key: inner_value
```

**Primitive Arrays:**
```
tags[3]: admin,ops,dev
```

**Tabular Arrays (uniform objects):**
```
items[2]{sku,name,price}:
  A1,Widget,9.99
  B2,Gadget,14.5
```

**Objects as List Items:**
```
items[2]:
  - id: 1
    name: Alice
  - id: 2
    name: Bob
```

### Escape Rules (Section 7.1)
Only five escapes permitted:
- `\\` (backslash)
- `\"` (double quote)
- `\n` (newline)
- `\r` (carriage return)
- `\t` (tab)

### String Quoting Requirements (Section 7.2)
Strings must be quoted if they:
- Are empty
- Have leading/trailing whitespace
- Equal `true`, `false`, or `null`
- Match numeric patterns or contain leading zeros
- Contain colons, quotes, backslashes, brackets, braces
- Include control characters
- Contain the active delimiter
- Start with hyphen

### Indentation
- Default: 2 spaces per level
- Tabs must not indent (allowed only as HTAB delimiter or in quoted strings)
- No trailing spaces; no trailing newline at EOF

Sources:
- [Official TOON Specification](https://github.com/toon-format/spec)
- [TOON Format Website](https://toonformat.dev/)

---

## Step 2: Tech-Stack Library Analysis

### TypeScript - @toon-format/toon (Reference Implementation)
**Strengths:**
- Official reference implementation
- Full spec compliance
- Encoder, decoder, and CLI tools
- npm package for easy integration

**API Pattern:**
```typescript
import { encode, decode } from '@toon-format/toon';
const toon = encode(jsonObject);
const json = decode(toonString);
```

### Python - python-toon
**Strengths:**
- Spec-compliant (v2.0)
- Simple API: `encode()` and `decode()`
- Configurable options (indent, delimiter, strict mode)

**API Pattern:**
```python
from toon import encode, decode
toon_str = encode(data, options={'indent': 2})
data = decode(toon_str, options={'strict': True})
```

**Options:**
- `indent`: Spaces per level (default: 2)
- `delimiter`: Array separator - comma, tab, pipe
- `strict`: Enable validation for syntax errors

### Rust - toon-format (Official)
**Strengths:**
- Fully compliant with TOON Spec v1.4
- Safe, fast Rust implementation
- Serde integration for easy serialization
- Zero-copy scanner and parser

**Features:**
- Smart tabular arrays
- Streaming serialization
- SIMD acceleration on supported platforms

### Rust - toon-rs
**Strengths:**
- v3.0 spec compliant
- 345/345 conformance tests passing
- Full serde integration

### Go - toon-go (Official)
**Strengths:**
- `Marshal()` and `Unmarshal()` functions
- Numbers as float64, objects as map[string]any
- Spec-compliant escaping

### Key API Patterns Across Libraries
1. **Simple encode/decode functions** - not complex object hierarchies
2. **Configuration via options object** - indent, delimiter, strict mode
3. **Strict mode toggle** - validation vs lenient parsing
4. **JSON interoperability** - works with native JSON types
5. **Error handling** - detailed error messages with positions

Sources:
- [toon-format/toon (TypeScript)](https://github.com/toon-format/toon)
- [python-toon](https://github.com/xaviviro/python-toon)
- [toon-format (Rust)](https://crates.io/crates/toon-format)
- [toon-go](https://pkg.go.dev/github.com/toon-format/toon-go)

---

## Step 3: Eiffel Ecosystem

### Existing Serialization Libraries

**simple_json**
- Full JSON parsing and generation
- SIMPLE_JSON_VALUE wrapper hierarchy
- JSONPath queries
- JSON Patch (RFC 6902)
- Semantic frame naming (multiple aliases per feature)
- Comprehensive error tracking with line/column positions

**simple_yaml**
- YAML parsing and generation
- Converts to/from JSON internally
- Supports YAML 1.2

**simple_csv**
- CSV parsing with configurable delimiters
- Header row support
- Streaming for large files

**simple_toml**
- TOML parsing and generation
- Type-safe value access

**simple_xml**
- XML parsing and generation
- XPath-like queries

### Gap Analysis
- **No TOON library exists** in the Eiffel ecosystem
- **No LLM-optimized serialization format** available
- simple_json provides the closest pattern to follow
- Opportunity to be the first Eiffel TOON implementation

### Integration Points
- Convert simple_json objects to TOON for LLM input
- Parse TOON responses back to simple_json
- Leverage existing SIMPLE_JSON_VALUE for internal representation

Sources:
- [simple_json source](file:///d:/prod/simple_json/src/core/simple_json.e)
- [Simple Eiffel ecosystem](https://github.com/simple-eiffel)

---

## Step 4: Developer Pain Points

### JSON Problems for LLMs

**Token Inefficiency:**
- Every `{}`, `[]`, `""`, `,` counts as tokens
- Keys repeated for every object in arrays
- 100 products = 100x "product_name" key

**Structural Overhead:**
- Deeply nested braces confuse models
- Missing comma/bracket causes parse failures
- LLMs "hallucinate" syntax errors

**Cost Impact:**
- Large JSON payloads burn through token budgets
- API costs scale with verbose formatting
- Context window filled with redundant syntax

### What Developers Want
1. **30-60% token reduction** without losing structure
2. **LLM-friendly format** that models parse reliably
3. **Lossless round-trip** to JSON
4. **Simple API** - encode/decode functions
5. **Strict validation** option for debugging
6. **Clear error messages** with position info

### When TOON Excels
- Uniform arrays of objects (tabular data)
- Lists with repeated field structures
- API responses with consistent schemas

### When to Avoid TOON
- Deeply nested, non-uniform structures
- Purely tabular data (CSV is smaller)
- Semi-uniform data (marginal savings)

Sources:
- [TOON vs JSON comparison](https://dev.to/abhilaksharora/toon-token-oriented-object-notation-the-smarter-lighter-json-for-llms-2f05)
- [LLM token optimization strategies](https://medium.com/@ghaaribkhurshid/the-yaml-manifesto-a-deep-dive-into-slashing-llm-costs-and-why-your-json-prompts-are-burning-money-423d7e7cb7ea)

---

## Step 5: Innovation Opportunities

### simple_toon Differentiators

1. **Contract-Based Validation**
   - Preconditions catch invalid input early
   - Postconditions guarantee correct encoding/decoding
   - Invariants maintain encoder state consistency
   - **Unique to Eiffel** - no other TOON library has DBC

2. **Seamless JSON Integration**
   - Accept SIMPLE_JSON_VALUE directly
   - Return SIMPLE_JSON_VALUE from decode
   - No intermediate conversion step

3. **Semantic Frame Naming**
   - `encode`, `to_toon`, `serialize`, `compress_for_llm`
   - `decode`, `from_toon`, `deserialize`, `parse_toon`
   - Match user's mental model

4. **Token Statistics**
   - Report estimated token count before/after
   - Show compression ratio achieved
   - Help developers measure savings

5. **Tabular Detection**
   - Auto-detect uniform arrays
   - Suggest tabular format when beneficial
   - `is_tabular_eligible` query

6. **Strict Mode with Detailed Errors**
   - Line/column position for all errors
   - Specific error types (delimiter mismatch, count mismatch)
   - Match simple_json error pattern

7. **SCOOP-Ready Design**
   - Thread-safe encoding/decoding
   - No shared mutable state

---

## Step 6: Design Strategy

### Core Design Principles
- **Simple**: One facade class for most use cases
- **Safe**: Contracts catch errors at runtime
- **Compatible**: Works with existing simple_json
- **Efficient**: Minimize string copying

### API Surface

#### Main Class: SIMPLE_TOON

```eiffel
class SIMPLE_TOON

create
    make

feature -- Encoding (JSON to TOON)

    encode,
    to_toon,
    serialize,
    compress_for_llm (a_json: SIMPLE_JSON_VALUE): STRING_32
        -- Convert JSON value to TOON format
        require
            json_not_void: a_json /= Void
        ensure
            result_not_empty: not Result.is_empty

    encode_string,
    json_to_toon (a_json_text: STRING_32): STRING_32
        -- Parse JSON string and convert to TOON
        require
            not_empty: not a_json_text.is_empty
        ensure
            result_not_empty: Result /= Void implies not Result.is_empty

feature -- Decoding (TOON to JSON)

    decode,
    from_toon,
    deserialize,
    parse_toon (a_toon_text: STRING_32): detachable SIMPLE_JSON_VALUE
        -- Parse TOON text and return JSON value
        require
            not_empty: not a_toon_text.is_empty
        ensure
            errors_on_failure: Result = Void implies has_errors

    decode_to_string,
    toon_to_json (a_toon_text: STRING_32): detachable STRING_32
        -- Parse TOON and return JSON string
        require
            not_empty: not a_toon_text.is_empty

feature -- Configuration

    set_indent (a_spaces: INTEGER)
        -- Set indentation spaces (default: 2)
        require
            positive: a_spaces > 0

    set_delimiter (a_delimiter: CHARACTER_32)
        -- Set array delimiter: ',' (comma), '%T' (tab), '|' (pipe)
        require
            valid_delimiter: a_delimiter = ',' or a_delimiter = '%T' or a_delimiter = '|'

    set_strict_mode (a_enabled: BOOLEAN)
        -- Enable/disable strict validation

    indent: INTEGER
    delimiter: CHARACTER_32
    is_strict: BOOLEAN

feature -- Analysis

    is_valid_toon (a_text: STRING_32): BOOLEAN
        -- Check if text is valid TOON

    token_estimate (a_json: SIMPLE_JSON_VALUE): INTEGER
        -- Estimate token count for JSON

    compression_ratio (a_json: SIMPLE_JSON_VALUE): REAL_64
        -- Ratio of TOON tokens to JSON tokens (< 1.0 = savings)

    is_tabular_eligible (a_json: SIMPLE_JSON_VALUE): BOOLEAN
        -- Would this JSON benefit from tabular TOON encoding?

feature -- Error Handling

    has_errors: BOOLEAN
    last_errors: ARRAYED_LIST [SIMPLE_TOON_ERROR]
    error_count: INTEGER
    first_error: detachable SIMPLE_TOON_ERROR
    errors_as_string: STRING_32
    clear_errors

feature -- File Operations

    encode_file (a_json_path, a_toon_path: STRING_32): BOOLEAN
        -- Read JSON file, write TOON file

    decode_file (a_toon_path, a_json_path: STRING_32): BOOLEAN
        -- Read TOON file, write JSON file

end
```

### Supporting Classes

**SIMPLE_TOON_ERROR**
- `message: STRING_32`
- `line: INTEGER`
- `column: INTEGER`
- `error_type: INTEGER` (syntax, delimiter, count_mismatch, etc.)

**SIMPLE_TOON_ENCODER** (internal)
- Handles JSON → TOON conversion
- Detects tabular eligibility
- Manages indentation

**SIMPLE_TOON_DECODER** (internal)
- Parses TOON syntax
- Validates structure
- Builds SIMPLE_JSON_VALUE

### Contract Strategy

**Preconditions:**
```eiffel
encode (a_json: SIMPLE_JSON_VALUE): STRING_32
    require
        json_not_void: a_json /= Void
```

**Postconditions:**
```eiffel
encode (a_json: SIMPLE_JSON_VALUE): STRING_32
    ensure
        result_not_empty: not Result.is_empty
        round_trip: attached decode (Result) as l_decoded implies
            l_decoded.is_equal (a_json)
```

**Invariants:**
```eiffel
invariant
    valid_indent: indent > 0
    valid_delimiter: delimiter = ',' or delimiter = '%T' or delimiter = '|'
    error_consistency: has_errors = not last_errors.is_empty
```

### ECF Structure

```xml
<target name="simple_toon">
    <root all_classes="true"/>
    <option void_safety="all">
        <assertions precondition="true" postcondition="true"
                    check="true" invariant="true"/>
    </option>
    <library name="base" location="$ISE_LIBRARY/library/base/base.ecf"/>
    <library name="simple_json" location="$SIMPLE_JSON/simple_json.ecf"/>
    <cluster name="src" location=".\src\" recursive="true"/>
</target>
```

### Test Strategy
- Encode/decode round-trip for all JSON types
- Tabular array formatting
- Delimiter variations (comma, tab, pipe)
- Strict mode validation
- Error position accuracy
- Edge cases: empty values, special characters, nested structures
- Conformance with official test fixtures

---

## Step 7: Implementation Assessment

### Implementation Plan

**Phase 1: Core (MVP)**
1. SIMPLE_TOON facade class
2. Basic encode (JSON → TOON)
3. Basic decode (TOON → JSON)
4. Primitive values (strings, numbers, booleans, null)
5. Simple objects (key: value)
6. Primitive arrays with [N] notation
7. Error handling framework

**Phase 2: Tabular Arrays**
1. Detect uniform object arrays
2. Generate tabular format with {fields}
3. Parse tabular rows
4. Delimiter configuration

**Phase 3: Advanced Features**
1. Objects as list items (- notation)
2. Nested structures
3. File operations
4. Token estimation
5. Compression ratio calculation

**Phase 4: Polish**
1. Full spec compliance testing
2. Performance optimization
3. Documentation
4. SERVICE_API integration

### Dependencies
- simple_json (for SIMPLE_JSON_VALUE)
- ISE base library

### Estimated Scope
- ~5 Eiffel classes
- ~50 features total
- ~30 test cases minimum

### Risks
- Spec is still "Working Draft" - may change
- LLMs not trained on TOON - adoption uncertain
- Tabular detection heuristics may need tuning

---

## Checklist

- [x] Formal specifications reviewed (Step 1)
- [x] Top 5 libraries in other languages studied (Step 2)
- [x] Eiffel ecosystem researched (Step 3)
- [x] Developer pain points documented (Step 4)
- [x] Innovation opportunities identified (Step 5)
- [x] Design strategy synthesized (Step 6)
- [x] Implementation assessment completed (Step 7)
- [ ] Library created
- [ ] Core encoding implemented
- [ ] Core decoding implemented
- [ ] Tests passing
- [ ] Documentation complete
