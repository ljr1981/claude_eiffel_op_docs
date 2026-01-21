# Eiffel-Loop Integration: Remaining Work Plan

## Date: 2026-01-20

## Overview

This document outlines remaining integration work following the anti-slop workflow system.

---

## Workflow Application

For each integration below, apply workflows in order:
1. **05_design-audit** (D01-D05): Plan integration without breaking existing API
2. **06_maintenance** (M01-M08): Execute integration, strengthen contracts
3. **08_bug-hunting** (B01-B04): Find integration issues
4. **09_bug-fixing** (F01-F06): Fix discovered bugs
5. **11_github-prep** (G01-G03): Update docs, commit

---

## Sprint 1: Complete Partial Integrations

### 1.1 simple_json + simple_encoding

**Current**: Uses simple_zstring for UTF conversion
**Goal**: Add SIMPLE_ENCODING for native UTF-8 handling

**Changes Required**:
```eiffel
-- ECF: Add dependency
<library name="simple_encoding" location="$SIMPLE_EIFFEL/simple_encoding/simple_encoding.ecf"/>

-- Code: Use SIMPLE_ENCODING where needed
encoding: SIMPLE_ENCODING
create encoding.make
utf8_bytes := encoding.utf32_to_utf8 (json_string)
```

**Verification**:
- [ ] Compile: `/d/prod/ec.sh -batch -config simple_json.ecf -target simple_json_tests -c_compile`
- [ ] Test: `./EIFGENs/simple_json_tests/W_code/simple_json.exe`
- [ ] All 260+ existing tests pass

---

### 1.2 simple_http + simple_encoding + simple_factory

**Current**: Uses simple_zstring for URL encoding
**Goal**: Add charset handling and connection pooling

**Changes Required**:

Part A - simple_encoding:
```eiffel
-- ECF: Add dependency
<library name="simple_encoding" location="$SIMPLE_EIFFEL/simple_encoding/simple_encoding.ecf"/>

-- Code: Decode responses by charset
feature decode_response (resp: HTTP_RESPONSE): STRING_32
    local
        charset: STRING
        codec: SIMPLE_CODEC
    do
        charset := resp.content_type_charset
        codec := encoding_registry.codec_for_name (charset)
        Result := codec.decode (resp.body)
    end
```

Part B - simple_factory:
```eiffel
-- ECF: Add dependency
<library name="simple_factory" location="$SIMPLE_EIFFEL/simple_factory/simple_factory.ecf"/>

-- New class: HTTP_CONNECTION_POOL
class HTTP_CONNECTION_POOL inherit SIMPLE_OBJECT_POOL [HTTP_CONNECTION]
```

**Verification**:
- [ ] Compile after each change
- [ ] Run existing tests
- [ ] Add new charset tests
- [ ] Add connection pool tests

---

### 1.3 simple_xml + simple_encoding + simple_reflection

**Current**: Uses simple_zstring for escaping
**Goal**: Add encoding declaration and XML serializer

**Changes Required**:

Part A - simple_encoding:
```eiffel
-- ECF: Add dependency
<library name="simple_encoding" location="$SIMPLE_EIFFEL/simple_encoding/simple_encoding.ecf"/>
```

Part B - simple_reflection:
```eiffel
-- ECF: Add dependency
<library name="simple_reflection" location="$SIMPLE_EIFFEL/simple_reflection/simple_reflection.ecf"/>

-- New class: SIMPLE_XML_SERIALIZER
class SIMPLE_XML_SERIALIZER
feature
    to_xml (a_object: ANY): SIMPLE_XML_ELEMENT
    from_xml (a_xml: SIMPLE_XML_ELEMENT; a_type: TYPE [ANY]): ANY
```

**Verification**:
- [ ] Compile after each change
- [ ] Run existing tests
- [ ] Add XML serialization tests

---

## Sprint 2: Data Layer Integration

### 2.1 simple_sql + All Three Libraries

**Planned Integrations**:
1. **simple_encoding**: UTF-8 string handling
2. **simple_factory**: Connection pooling (SIMPLE_OBJECT_POOL)
3. **simple_reflection**: ORM capabilities

**Key New Classes**:
- DATABASE_CONNECTION_POOL (inherits SIMPLE_OBJECT_POOL)
- SIMPLE_SQL_ORM (uses SIMPLE_REFLECTED_OBJECT)

**Verification**:
- [ ] Compile incrementally
- [ ] All 46 existing tests pass
- [ ] New connection pool tests
- [ ] New ORM tests

---

### 2.2 simple_csv + Three Libraries

**Planned Integrations**:
1. **simple_encoding**: BOM detection, encoding conversion
2. **simple_reflection**: Object-to-row mapping
3. **simple_zstring**: Replace escape_field, use splitter

**Key Changes**:
- Remove manual escape_field
- Add SIMPLE_ENCODING_DETECTOR for BOM
- Add SIMPLE_CSV_MAPPER using reflection

---

## Sprint 3: Supporting Libraries

### 3.1 simple_template + Three Libraries

**Planned Integrations**:
1. **simple_encoding**: Template file encoding
2. **simple_reflection**: Object field access in templates
3. **simple_zstring**: HTML escaping, substitute, builder

### 3.2 simple_mock + Two Libraries

**Planned Integrations**:
1. **simple_factory**: Mock factory patterns
2. **simple_reflection**: Dynamic mock generation

### 3.3 simple_email + simple_encoding

**Planned Integration**: MIME charset handling

### 3.4 simple_config + simple_factory

**Planned Integration**: Singleton pattern for global config

### 3.5 simple_diff + simple_reflection

**Planned Integration**: Deep object comparison

---

## Anti-Slop Checklist (Per Integration)

Before marking any integration complete:

```
[ ] Read 00_ANTI-SLOP-RULES.md
[ ] Executed design audit (D01-D05) - documented in {lib}/design-audit/
[ ] Executed maintenance (M01-M08) - code changes made
[ ] Compiled after EVERY code change - ec.sh output pasted
[ ] Ran tests after EVERY change - test output pasted
[ ] All original tests still pass - count matches
[ ] New integration tests pass - new tests written
[ ] Updated README with new dependencies
[ ] Committed and pushed
```

---

## Effort Estimates

| Integration | Complexity | New Classes | Est. Tests |
|-------------|------------|-------------|------------|
| simple_json + encoding | Low | 0 | 5 |
| simple_http + encoding | Medium | 0 | 10 |
| simple_http + factory | High | 2 | 15 |
| simple_xml + encoding | Low | 0 | 5 |
| simple_xml + reflection | High | 1 | 20 |
| simple_sql + all three | Very High | 3 | 30 |
| simple_csv + all three | Medium | 2 | 20 |
| simple_template + all three | Medium | 0 | 15 |
| simple_mock + two | High | 2 | 20 |
| simple_email + encoding | Low | 0 | 10 |
| simple_config + factory | Low | 0 | 5 |
| simple_diff + reflection | Medium | 1 | 15 |

---

## Recommended Order of Execution

1. **simple_json + encoding** (quick win, already uses zstring)
2. **simple_http + encoding** (builds on pattern)
3. **simple_xml + encoding** (same pattern)
4. **simple_email + encoding** (simple integration)
5. **simple_config + factory** (simple singleton)
6. **simple_xml + reflection** (XML serializer)
7. **simple_http + factory** (connection pooling)
8. **simple_sql** (ORM is complex, do last in data layer)
9. **simple_csv** (after SQL patterns established)
10. **simple_template** (uses reflection patterns from above)
11. **simple_mock** (complex auto-mocking)
12. **simple_diff** (object comparison last)

---

## Success Criteria

Integration is complete when:

1. All 10 target libraries have planned integrations
2. All original tests pass (no regressions)
3. New integration tests pass
4. Documentation updated
5. All changes committed and pushed
6. READMEs reflect new dependencies

---

## File References

- Status Report: `D:\prod\reference_docs\plans\EIFFEL_LOOP_INTEGRATION_STATUS.md`
- Original Plan: `D:\prod\reference_docs\plans\EIFFEL_LOOP_INTEGRATION_EXECUTION_PLAN.md`
- Anti-Slop Rules: `D:\prod\reference_docs\research\AI-slop\prompts\00_ANTI-SLOP-RULES.md`
- Workflow Orchestrator: `D:\prod\reference_docs\research\AI-slop\prompts\WORKFLOW-ORCHESTRATOR.md`
