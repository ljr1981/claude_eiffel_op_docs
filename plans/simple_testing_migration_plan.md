# simple_testing Migration Plan

## Executive Summary

Migrate `testing_ext` to `simple_testing` with enhanced assertions, then update all 30+ simple_* libraries to use the new library.

---

## Current State Analysis

### testing_ext Contents
- `TEST_SET_BASE` - Deferred class with 25+ assertions:
  - Boolean: `assert_true`, `assert_false`, `refute`
  - Integer: `assert_greater_than`, `assert_less_than`, `assert_in_range`, `assert_positive`, `assert_negative`, `assert_zero`, `assert_non_zero`
  - Real: `assert_reals_equal`, `assert_real_greater_than`, `assert_real_less_than`, `assert_real_in_range`
  - String: `assert_string_contains`, `assert_string_starts_with`, `assert_string_ends_with`, `assert_string_empty`, `assert_string_not_empty`, `assert_strings_equal_diff`

- `TEST_SET_BRIDGE` - Bridge class for accessing {NONE} features in tests

### Libraries Using testing_ext (30+)
- simple_ai_client, simple_alpine, simple_app_api, simple_base64, simple_cache
- simple_clipboard, simple_console, simple_cors, simple_datetime, simple_ec
- simple_env, simple_foundation_api, simple_gui_designer, simple_htmx, simple_ipc
- simple_logger, simple_markdown, simple_mmap, simple_process, simple_randomizer
- simple_rate_limiter, simple_regex, simple_registry, simple_service_api, simple_showcase
- simple_sql, simple_system, simple_template, simple_uuid, simple_validation
- simple_watcher, simple_win32_api

---

## Proposed New Assertions for TEST_SET_BASE

### Object/Reference Assertions
```eiffel
assert_attached (a_tag: STRING; a_object: detachable ANY)
    -- Assert that `a_object' is not Void

assert_void (a_tag: STRING; a_object: detachable ANY)
    -- Assert that `a_object' is Void

assert_same_reference (a_tag: STRING; a_expected, a_actual: ANY)
    -- Assert `a_expected' and `a_actual' are the same object (=)

assert_not_same_reference (a_tag: STRING; a_expected, a_actual: ANY)
    -- Assert `a_expected' and `a_actual' are different objects
```

### Collection Assertions
```eiffel
assert_array_contains (a_tag: STRING; a_array: ARRAY[ANY]; a_item: ANY)
    -- Assert that `a_array' contains `a_item'

assert_array_not_contains (a_tag: STRING; a_array: ARRAY[ANY]; a_item: ANY)
    -- Assert that `a_array' does not contain `a_item'

assert_iterable_empty (a_tag: STRING; a_collection: ITERABLE[ANY])
    -- Assert that `a_collection' is empty

assert_iterable_not_empty (a_tag: STRING; a_collection: ITERABLE[ANY])
    -- Assert that `a_collection' is not empty

assert_count_equals (a_tag: STRING; a_expected: INTEGER; a_collection: FINITE[ANY])
    -- Assert that `a_collection'.count = `a_expected'
```

### String Assertions (Additional)
```eiffel
assert_strings_equal_case_insensitive (a_tag: STRING; a_expected, a_actual: STRING)
    -- Assert strings are equal ignoring case

assert_string_matches (a_tag: STRING; a_string, a_pattern: STRING)
    -- Assert that `a_string' matches regex `a_pattern' (if simple_regex available)

assert_string_length (a_tag: STRING; a_expected_length: INTEGER; a_string: STRING)
    -- Assert that `a_string'.count = `a_expected_length'
```

### Numeric Assertions (Additional)
```eiffel
assert_integers_equal (a_tag: STRING; a_expected, a_actual: INTEGER)
    -- Assert `a_expected' = `a_actual' with detailed message

assert_naturals_equal (a_tag: STRING; a_expected, a_actual: NATURAL_64)
    -- Assert `a_expected' = `a_actual' with detailed message

assert_non_negative (a_tag: STRING; a_value: INTEGER)
    -- Assert that `a_value' >= 0

assert_non_positive (a_tag: STRING; a_value: INTEGER)
    -- Assert that `a_value' <= 0
```

### Comparison Assertions
```eiffel
assert_equal (a_tag: STRING; a_expected, a_actual: ANY)
    -- Assert `a_expected'.is_equal (`a_actual')

assert_not_equal (a_tag: STRING; a_expected, a_actual: ANY)
    -- Assert not `a_expected'.is_equal (`a_actual')
```

---

## simple_testing Structure

```
simple_testing/
├── src/
│   ├── test_set_base.e          -- Enhanced assertions
│   └── test_set_bridge.e        -- Bridge class (unchanged)
├── docs/
│   ├── index.html
│   ├── css/style.css
│   └── images/logo.png
├── simple_testing.ecf
└── README.md
```

### ECF Design
```xml
<system name="simple_testing" library_target="simple_testing">
    <target name="simple_testing">
        <root all_classes="true"/>
        <library name="base" location="$ISE_LIBRARY\library\base\base.ecf"/>
        <library name="testing" location="$ISE_LIBRARY\library\testing\testing.ecf"/>
        <cluster name="src" location=".\src\" recursive="true"/>
    </target>
</system>
```

---

## Migration Strategy

### Phase 1: Create simple_testing
1. Create `D:\prod\simple_testing\` directory structure
2. Copy and enhance `test_set_base.e` with new assertions
3. Copy `test_set_bridge.e` unchanged
4. Create `simple_testing.ecf`
5. Create README.md and docs/

### Phase 2: Batch Update All Libraries
**Approach: Automated text replacement**

For each library's ECF file:
- Replace: `<library name="testing_ext" location="$TESTING_EXT\testing_ext.ecf"/>`
- With: `<library name="simple_testing" location="$SIMPLE_TESTING\simple_testing.ecf"/>`

This is a safe, mechanical replacement since:
- The class names remain identical (TEST_SET_BASE, TEST_SET_BRIDGE)
- The inheritance hierarchy is unchanged
- Only the library location changes

### Phase 3: Spot-Check Compilation
Rather than recompiling all 30+ libraries, spot-check:

1. **simple_foundation_api** - Complex aggregator, tests many assertions
2. **simple_json** - Uses string comparison extensively
3. **simple_hash** - Uses numeric comparisons
4. **simple_process** - Uses various assertion types
5. **simple_win32_api** - Recently created, good baseline

If these 5 compile and pass tests, the migration is successful.

### Phase 4: Cleanup
1. Set SIMPLE_TESTING environment variable
2. Mark testing_ext as deprecated in its README
3. Consider making testing_ext re-export simple_testing for backwards compatibility

---

## Risk Assessment

### Low Risk
- Class names unchanged (TEST_SET_BASE, TEST_SET_BRIDGE)
- All existing assertions preserved
- New assertions are additive only

### Medium Risk
- Environment variable change (TESTING_EXT -> SIMPLE_TESTING)
  - Mitigation: Document clearly, can keep both set temporarily

### Mitigation
- Spot-check compilation of representative libraries
- Keep testing_ext available during transition
- New assertions don't affect existing tests

---

## Execution Checklist

- [ ] Create simple_testing directory structure
- [ ] Create simple_testing.ecf
- [ ] Migrate test_set_base.e with enhancements
- [ ] Migrate test_set_bridge.e
- [ ] Create README.md
- [ ] Create docs/index.html, css/style.css, images/logo.png
- [ ] Set SIMPLE_TESTING environment variable
- [ ] Update all 30+ ECF files (automated)
- [ ] Spot-check: simple_foundation_api
- [ ] Spot-check: simple_json
- [ ] Spot-check: simple_hash
- [ ] Spot-check: simple_process
- [ ] Spot-check: simple_win32_api
- [ ] Commit and push simple_testing
- [ ] Commit and push updated libraries

---

## Estimated Scope

| Task | Complexity |
|------|------------|
| Create simple_testing | LOW |
| Enhance TEST_SET_BASE | MEDIUM |
| Batch ECF updates | LOW (automated) |
| Spot-check compilation | MEDIUM |
| Documentation | LOW |

**Recommendation:** Execute in single session. The batch update is mechanical and low-risk.

---

## Answer to Question 7

> Do we need to recompile/test in all simple_* libs or can we do the work to all of them and spot-check a few?

**Answer:** Spot-checking 5 representative libraries is sufficient because:
1. The class interface is identical
2. The change is purely a library location change
3. No API changes to existing assertions
4. The 5 selected libraries cover different assertion categories

If the 5 spot-checks pass, the remaining libraries will work without recompilation verification.
