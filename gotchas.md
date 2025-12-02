# Eiffel Gotchas: Documentation vs. Reality

This file tracks cases where official Eiffel documentation differs from actual compiler behavior.

**Note**: SQLite/database-specific gotchas are in `sqlite_gotchas.md`

---

## EIFGENs Directory - Hands Off!

**NEVER modify files in EIFGENs** - this is EiffelStudio's workspace exclusively.

- **Read-only for everyone** (human and AI alike)
- Any modifications risk corrupting the build
- Segmentation faults on startup often indicate corrupted EIFGENs
- **Solution for segfaults**: Clean compile (`-clean` flag)

```batch
:: Clean compile to fix corrupted EIFGENs
ec.exe -batch -config project.ecf -target my_target -c_compile -freeze -clean
```

**Verified**: 2025-12-02, EiffelStudio 25.02

---

## Format

Each entry follows this structure:

```
### [Topic]
- **Docs say**: What the documentation claims
- **Reality**: What actually works
- **Verified**: Date and EiffelStudio version
- **Example**: Working code
```

---

## Design Principles

### NEVER Use rescue/retry for Internal Code
- **Rule**: ONLY use `rescue`/`retry` when talking to external systems beyond the current Eiffel project universe (e.g. COM, external C libraries, network calls)
- **Reason**: Eiffel's Design by Contract means preconditions should be satisfied BEFORE calling a feature. If a precondition fails, the caller has a bug - don't mask it with exception handling.
- **Action**: Fix the root cause (satisfy the precondition) instead of catching exceptions
- **Verified**: 2025-11-29

---

## Type System Issues

### VAPE Error - Preconditions Referencing Private Features
- **Docs say**: Preconditions should express contract requirements
- **Reality**: Preconditions CANNOT reference features that aren't exported to clients (VAPE = Violation of Assertion PExport)
- **Verified**: 2025-11-29, EiffelStudio 24.x
- **Example**:
```eiffel
-- WRONG: Causes VAPE error
feature {NONE} -- Implementation
    table_name: STRING_8

feature -- Execution
    execute
        require
            has_table: not table_name.is_empty  -- ERROR: table_name is private!

-- CORRECT: Add public status query
feature -- Status (for preconditions)
    has_table: BOOLEAN
        do
            Result := not table_name.is_empty
        end

feature -- Execution
    execute
        require
            has_table: has_table  -- OK: has_table is public
```

### STRING_32 vs STRING_8 Conversions
- **Docs say**: String types are largely interchangeable
- **Reality**: Assignments require explicit conversion; many features return STRING_32
- **Verified**: 2025-11-29, EiffelStudio 24.x
- **Example**:
```eiffel
-- WRONG: Type mismatch (VJAR error)
l_name: STRING_8
l_name := some_feature_returning_string_32

-- CORRECT: Explicit conversion
l_name_32: STRING_32
l_name: STRING_8
l_name_32 := some_feature_returning_string_32
l_name := l_name_32.to_string_8
```

### Inline If-Then-Else Returns ANY
- **Docs say**: Inline conditionals return typed values
- **Reality**: Inline `if-then-else` expressions return type ANY, causing type mismatches
- **Verified**: 2025-11-29, EiffelStudio 24.x
- **Example**:
```eiffel
-- WRONG: Returns ANY, causes VUAR(2) error when passed to typed parameter
some_feature.make (if condition then "a" else "b" end)

-- CORRECT: Use explicit variable
local
    l_value: STRING_8
do
    if condition then
        l_value := "a"
    else
        l_value := "b"
    end
    some_feature.make (l_value)
```

### Obsolete as_string_8
- **Docs say**: Use `as_string_8` for conversion
- **Reality**: `as_string_8` is obsolete; use `to_string_8` instead
- **Verified**: 2025-11-29, EiffelStudio 24.x
- **Example**:
```eiffel
-- WRONG: Obsolete warning
l_upper := a_name.as_string_8.as_upper

-- CORRECT:
l_upper := a_name.to_string_8.as_upper
```

### Numeric Literal Types (REAL_32 vs REAL_64)
- **Docs say**: (assumed) Floating point literals are REAL_64
- **Reality**: Literals like `3.14` may be stored as REAL_32 when passed through ANY. When checking types with `attached {REAL_64}`, REAL_32 values won't match.
- **Verified**: 2025-11-29, EiffelStudio 24.x
- **Example**:
```eiffel
-- WRONG: Only handles REAL_64
if attached {REAL_64} a_value as l_real then
    Result := l_real.out
else
    Result := "NULL"  -- REAL_32 falls through here!
end

-- CORRECT: Handle both REAL types
if attached {REAL_64} a_value as l_real then
    Result := l_real.out
elseif attached {REAL_32} a_value as l_real32 then
    Result := l_real32.out
else
    Result := "NULL"
end
```

---

## String Handling

### Percent Signs in Manifest Strings Need Escaping
- **Docs say**: (assumed) Strings are passed as-is
- **Reality**: In Eiffel manifest strings, `%` is an escape character. For literal `%`, use `%%`
- **Verified**: 2025-12-01, EiffelStudio 25.02
- **Example**:
```eiffel
-- WRONG: Single percent is escape character
l_sql := "SELECT * FROM users WHERE name LIKE 'John%'"  -- May cause string parsing issues

-- CORRECT: Escape percent signs
l_sql := "SELECT * FROM users WHERE name LIKE 'John%%'"  -- %% becomes single % in output
```

---

## Documentation Features

### EIS (Eiffel Information System) Annotations
- **Docs say**: Limited documentation available on Eiffel.org
- **Reality**: EIS provides bidirectional linking between Eiffel source and external documentation
- **Verified**: 2025-12-01, EiffelStudio 25.02
- **Key Points**:
  - Outgoing links (source → docs): Add `note EIS:` annotations to class header
  - Incoming links (docs → source): Use `eiffel:?class=...&feature=...` URI scheme
  - Protocol must be "URI" for HTML files, "pdf" for PDFs
  - Path is relative from the .e file location
- **Example**:
```eiffel
-- In Eiffel source (outgoing):
note
    description: "My class"
    EIS: "name=API Reference", "src=../docs/api/myclass.html", "protocol=URI", "tag=documentation"
    EIS: "name=Tutorial", "src=../docs/tutorial.html", "protocol=URI", "tag=tutorial"

-- In HTML documentation (incoming):
<a href="eiffel:?class=MY_CLASS&feature=my_feature">View in EiffelStudio</a>
<a href="eiffel:?class=MY_CLASS">View class</a>
```
- **Usage**: Press F1 in EiffelStudio on a class with EIS annotations to open linked documentation

---

## Pending Investigation

### Across Loop Item Access
- **Status**: RESOLVED - see `across_loops.md`
- **Summary**: Whether you use `ic` or `ic.item` depends on the iteration cursor type. For ARRAYED_LIST, the cursor IS the item. For HASH_TABLE, use `ic.item` for value.
