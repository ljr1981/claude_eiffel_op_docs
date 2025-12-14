# Eiffel Expert Briefing

**Purpose**: This document transforms an AI into an Eiffel development expert for the Simple Eiffel ecosystem, grounded in Design by Contract, void safety, SCOOP concurrency, and the theoretical foundations of Object-Oriented Software Construction (OOSC2).

**Ecosystem**: 59+ `simple_*` libraries built on Design by Contract, void safety, and SCOOP concurrency.

**Authoritative Theory**: Bertrand Meyer, *Object-Oriented Software Construction, 2nd ed.* (OOSC2).

---

## Part 0: Foundational Principles (OOSC2)

### Seamlessness Principle

The same concepts apply at **analysis, design, and implementation** time.

- Classes are *models*, not mere code containers
- Contracts are *specifications*, not defensive checks
- Invariants express *design truths*, not runtime tricks

**Implication for AI agents**: Never treat Eiffel code as "temporary scaffolding." Every class is part of the final software model.

### Abstract Data Type (ADT) Principle

Every class defines an **Abstract Data Type**.

Contracts MUST describe:
- Observable behavior
- Semantic meaning
- State guarantees

Contracts MUST NOT describe:
- Internal data structures
- Algorithmic steps
- Performance characteristics

If a contract references representation details, it is almost certainly wrong.

### Correctness vs Robustness

- **Correctness**: behavior *within* the specification
- **Robustness**: behavior *outside* the specification

Rules:
- Preconditions define the limits of correctness
- Violating a precondition is a **client error**
- Robustness concerns are handled via **exceptions**, not stricter preconditions

**AI Warning**: Do NOT convert robustness problems into tighter preconditions.

---

## Part 1: Eiffel Language Fundamentals

### Design by Contract (DBC)

Every feature should have contracts:

```eiffel
feature_name (arg: TYPE): RESULT
    require
        arg_valid: arg /= Void
        arg_in_range: arg.count > 0
    do
        -- implementation
    ensure
        result_valid: Result /= Void
        state_updated: some_attribute = expected_value
    end
```

**Contract rules:**
- `require`: Preconditions - caller's responsibility
- `ensure`: Postconditions - feature's guarantee
- `invariant`: Class invariants - always true between calls
- Contracts are executable documentation AND runtime checks

**Contract Classification Rule:**

Every precondition must be one of:
1. **Type safety** (attachment, range, validity)
2. **Protocol requirement** (call order, lifecycle)
3. **Environmental assumption** that cannot be enforced locally (file system, OS, external resources)

Preconditions MUST NOT encode:
- Current data structure shape
- Implementation convenience
- Optimizations

When in doubt, weaken the precondition and strengthen the postcondition.

```eiffel
-- WRONG: Implementation-specific precondition
require
    list_has_items: my_list.count > 0  -- Why must it have items?

-- CORRECT: Semantic precondition
require
    list_attached: my_list /= Void  -- Type safety
```

**Critical gotcha**: Preconditions cannot reference private features (VAPE error). Create public status queries:

```eiffel
-- WRONG: Causes VAPE error
feature {NONE}
    internal_data: STRING

feature
    process
        require
            has_data: not internal_data.is_empty  -- ERROR!

-- CORRECT: Public status query
feature -- Status
    has_data: BOOLEAN
        do Result := not internal_data.is_empty end

feature
    process
        require
            has_data: has_data  -- OK
```

### Void Safety

All code must be void-safe. Key patterns:

```eiffel
-- Detachable (can be Void)
my_value: detachable STRING

-- Attached (never Void)
my_required: STRING

-- Check attachment before use
if attached my_value as l_val then
    -- l_val is guaranteed attached here
    process (l_val)
end

-- Object test with type check
if attached {SPECIFIC_TYPE} some_any as l_typed then
    -- l_typed is both attached AND typed
end
```

### Creation Procedures

```eiffel
class MY_CLASS
create
    make, make_with_value, default_create

feature {NONE} -- Initialization
    make
        do
            create internal_list.make (10)
        end

    make_with_value (a_value: INTEGER)
        require
            positive: a_value > 0
        do
            make
            stored_value := a_value
        ensure
            value_set: stored_value = a_value
        end
```

### Inheritance

```eiffel
class CHILD
inherit
    PARENT
        rename
            old_name as new_name
        redefine
            overridden_feature
        undefine
            feature_to_defer  -- effective -> deferred
        select
            chosen_for_polymorphism
        end
```

**Feature joining rules:**
- Deferred + Deferred = Single deferred (auto-join)
- Deferred + Effective = Effective satisfies deferred (auto)
- Effective + Effective = Conflict (must use rename/undefine/select)

**VDUS(3) error**: Cannot `undefine` an already-deferred feature.

**Contract Substitution Rule (OOSC2):**

For a redefined feature:
- Preconditions may only be **weakened** (or kept same)
- Postconditions may only be **strengthened** (or kept same)

Violating this rule breaks **substitutability** - a child instance cannot safely replace a parent instance, even if code compiles.

```eiffel
-- Parent
process (x: INTEGER)
    require x > 0
    ensure Result > 0

-- WRONG child redefinition
process (x: INTEGER)
    require x > 10  -- STRONGER precondition = violation

-- CORRECT child redefinition
process (x: INTEGER)
    require x >= 0  -- WEAKER precondition = OK
    ensure Result > 100  -- STRONGER postcondition = OK
```

**Taxomania Warning (OOSC2):**

Do NOT introduce inheritance merely to:
- Share code
- Reuse implementation
- "Organize" classes

Prefer **composition** unless a true *is-a* relationship exists. Inheritance is **classification**, not code reuse.

### Generics

```eiffel
-- Unconstrained
class CONTAINER [G]

-- Constrained
class ACCOUNT_LIST [G -> ACCOUNT]

-- Multiple constraints
class AUDITABLE_STORE [G -> {ACCOUNT, AUDITABLE}]
```

### Agents (First-Class Functions)

```eiffel
-- Agent from method
my_agent := agent my_method
my_agent := agent {TARGET_CLASS}.method

-- Inline agent
my_agent := agent (x: INTEGER): BOOLEAN do Result := x > 0 end

-- Agent with open/closed arguments
agent my_method (?, closed_value)  -- ? = open argument
```

### Once Functions (Singletons/Caching)

```eiffel
shared_config: MY_CONFIG
    once
        create Result.make
    end

-- Variants
once ("PROCESS")  -- once per process (default)
once ("THREAD")   -- once per thread
once ("OBJECT")   -- once per object instance
```

### SCOOP (Concurrency)

SCOOP uses the `separate` keyword for safe concurrency:

```eiffel
my_worker: separate WORKER

-- Cannot call separate attribute directly - wrap in routine
do_work
    do
        launch_worker (my_worker)  -- Pass as argument
    end

launch_worker (a_worker: separate WORKER)
    do
        a_worker.perform  -- OK: locked via argument
    end
```

**Key rules:**
- Separate commands are asynchronous (fire and forget)
- Separate queries are synchronous (wait for result)
- Preconditions on separate args become wait conditions

**Inline separate block:**
```eiffel
separate my_worker as l_worker do
    l_worker.perform
end
```

**SCOOP Liveness Warning:**

SCOOP correctness does not imply liveness. Code can be 100% rule-compliant and still deadlock or starve.

Agents MUST NOT:
- Introduce nested separate calls without review
- Add blocking queries on separate objects in loops
- Introduce new wait conditions unless explicitly required by protocol

Blocking queries on separate objects ARE permitted when:
- The routine is explicitly documented as a synchronization point
- The blocking is semantically required, not incidental (e.g., join points, barriers)

```eiffel
-- DANGEROUS: Nested separate calls
process_all (a_workers: separate ARRAY [separate WORKER])
    do
        across a_workers as w loop
            w.result  -- Blocking query inside loop = potential starvation
        end
    end
```

---

## Part 2: Simple Eiffel Ecosystem Standards

### ECF Configuration

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<system name="simple_xxx" uuid="UNIQUE-UUID-HERE" library_target="simple_xxx">
    <target name="simple_xxx">
        <root all_classes="true"/>
        <option warning="warning">
            <assertions precondition="true" postcondition="true"
                       check="true" invariant="true" loop="true"/>
        </option>
        <capability>
            <concurrency support="scoop" use="thread"/>
            <void_safety support="all"/>
        </capability>
        <library name="base" location="$ISE_LIBRARY\library\base\base.ecf"/>
        <library name="simple_json" location="$SIMPLE_JSON\simple_json.ecf"/>
        <cluster name="src" location=".\src\" recursive="true"/>
    </target>
    <target name="simple_xxx_tests" extends="simple_xxx">
        <root class="XXX_TEST_APP" feature="make"/>
        <library name="testing" location="$ISE_LIBRARY\library\testing\testing.ecf"/>
        <library name="simple_testing" location="$SIMPLE_TESTING\simple_testing.ecf"/>
        <cluster name="test_classes" location=".\testing\" recursive="true"/>
    </target>
</system>
```

**Critical rules:**
- Always use SCOOP capability (`concurrency="scoop"`)
- Use `$ENV_VAR` for library paths, never hardcoded
- UUID must be truly unique (generate fresh, never copy)
- Test cluster name is `test_classes` (not `testing` - conflicts with ISE library)
- Test folder is `testing/` (standardized across ecosystem)

### Testing Standard

**CRITICAL: Test target requires BOTH libraries:**

```xml
<target name="lib_tests" extends="lib">
    <root class="TEST_APP" feature="make"/>
    <library name="simple_testing" location="$SIMPLE_TESTING\simple_testing.ecf"/>
    <library name="testing" location="$ISE_LIBRARY\library\testing\testing.ecf"/>
    <cluster name="test_classes" location=".\testing\" recursive="true"/>
</target>
```

1. **`simple_testing`** → Provides `TEST_SET_BASE` with enhanced assertions
2. **ISE `testing`** → Provides EQA framework so Autotest detects and runs tests

**BOTH libraries are required. No shortcuts.**

**Standard file structure:**
```
testing/
    test_app.e      -- TEST_APP (root class, console runner)
    lib_tests.e     -- LIB_TESTS (inherits TEST_SET_BASE)
```

**LIB_TESTS class** (inherits TEST_SET_BASE):

```eiffel
class
    LIB_TESTS

inherit
    TEST_SET_BASE

feature -- Tests

    test_something
        local
            l_obj: MY_CLASS
        do
            create l_obj.make
            assert_strings_equal ("name matches", "expected", l_obj.name)
            assert_integers_equal ("count", 5, l_obj.count)
            assert_true ("is valid", l_obj.is_valid)
        end
```

**TEST_APP class** (console runner, has-a LIB_TESTS):

```eiffel
class
    TEST_APP

create
    make

feature {NONE} -- Initialization

    make
            -- Run all tests with console output
        do
            print ("=== MY_LIB TESTS ===%N")
            create tests
            run_test (agent tests.test_something, "test_something")
            print ("=== RESULTS: " + passed.out + " passed, " + failed.out + " failed ===%N")
        end

feature {NONE} -- Implementation

    tests: LIB_TESTS
    passed, failed: INTEGER

    run_test (a_test: PROCEDURE; a_name: STRING)
            -- Run single test with exception handling
        local
            l_failed: BOOLEAN
        do
            if not l_failed then
                a_test.call (Void)
                print ("  " + a_name + ": PASSED%N")
                passed := passed + 1
            end
        rescue
            l_failed := True
            print ("  " + a_name + ": FAILED%N")
            failed := failed + 1
            retry
        end

end
```

**Dual-use pattern:**
- **Console**: Run TEST_APP directly (Claude CLI or EiffelStudio run)
- **Autotest**: EiffelStudio discovers LIB_TESTS via EQA framework

**Libraries with multiple test classes:**

Some libraries (simple_json, simple_sql, simple_web, etc.) have multiple specialized test classes. For these:

1. **TEST_APP** - knows about and runs ALL test classes
2. **LIB_TESTS** - empty placeholder with explanatory note

```eiffel
note
    description: "[
        Placeholder test class for pattern consistency.

        This library has multiple specialized test classes:
        - TEST_SIMPLE_JSON: Core JSON parsing tests
        - TEST_JSON_PATH_QUERIES: JSONPath query tests
        - (etc.)

        All test classes inherit from TEST_SET_BASE and are
        discoverable by Autotest. TEST_APP runs all tests
        with console output.
    ]"

class
    LIB_TESTS

inherit
    TEST_SET_BASE

end
```

This maintains pattern consistency while accommodating libraries that need multiple test sets.

### Library Dependencies

**ALWAYS use simple_* over ISE stdlib when available:**
- `simple_process` NOT `$ISE_LIBRARY/process`
- `simple_json` for JSON (not gobo or other)
- `simple_file` for file operations
- `simple_sql` for SQLite

**Only ISE libraries allowed directly:**
- `base` - fundamental data structures
- `time` - date/time operations
- `testing` - EQA framework base

### Class Structure Standard

```eiffel
note
    description: "Brief description"
    author: "Author name"
    date: "$Date$"
    revision: "$Revision$"

class
    MY_CLASS

inherit
    PARENT_CLASS
        redefine
            some_feature
        end

create
    make

feature {NONE} -- Initialization

    make (a_param: TYPE)
            -- Create with `a_param`.
        require
            param_valid: a_param /= Void
        do
            internal_data := a_param
        ensure
            data_set: internal_data = a_param
        end

feature -- Access

    data: TYPE
            -- Public accessor
        do
            Result := internal_data
        end

feature -- Status report

    is_valid: BOOLEAN
            -- Is current state valid?
        do
            Result := internal_data /= Void
        end

feature -- Basic operations

    process
            -- Main operation
        require
            valid: is_valid
        do
            -- implementation
        ensure
            processed: was_processed
        end

feature {NONE} -- Implementation

    internal_data: TYPE

invariant
    data_consistent: is_valid implies internal_data /= Void

end
```

### Inline C Pattern (Eric Bezault Style)

For Win32 API and external C calls, use inline C - NO separate .c files:

```eiffel
feature {NONE} -- External

    c_get_tick_count: NATURAL_32
            -- Windows GetTickCount
        external
            "C inline use <windows.h>"
        alias
            "return (EIF_NATURAL_32)GetTickCount();"
        end

    c_create_file (a_path: POINTER; a_access, a_share, a_disp: NATURAL_32): POINTER
            -- Windows CreateFile
        external
            "C inline use <windows.h>"
        alias
            "[
                return CreateFileW(
                    (LPCWSTR)$a_path,
                    (DWORD)$a_access,
                    (DWORD)$a_share,
                    NULL,
                    (DWORD)$a_disp,
                    FILE_ATTRIBUTE_NORMAL,
                    NULL
                );
            ]"
        end
```

---

## Part 3: Critical Gotchas

### Collection Iteration

```eiffel
-- ARRAYED_LIST: ic IS the item
across my_list as ic loop
    process (ic)  -- ic is the item directly
end

-- HASH_TABLE: ic is value, @ic.key is key
across my_table as ic loop
    print (@ic.key)  -- key
    print (ic)       -- value
end

-- HASH_TABLE with from/until (when you need key_for_iteration)
from my_table.start until my_table.after loop
    print (my_table.key_for_iteration)
    print (my_table.item_for_iteration)
    my_table.forth
end
```

### Collection Equality

```eiffel
-- WRONG: has uses reference equality
if my_list.has ("value") then ...

-- CORRECT: Use ~ for value equality
if across my_list as ic some ic ~ "value" end then ...
```

### String Types

```eiffel
-- STRING_32 to STRING_8 conversion
l_str8 := l_str32.to_string_8  -- NOT as_string_8 (obsolete)

-- Percent sign escaping in manifest strings
l_sql := "WHERE name LIKE 'John%%'"  -- %% becomes single %
```

### Inline If-Then-Else Returns ANY

```eiffel
-- WRONG: Returns ANY type
some_call (if condition then "a" else "b" end)  -- Type error

-- CORRECT: Use local variable
if condition then l_value := "a" else l_value := "b" end
some_call (l_value)
```

### EIFGENs Directory

**NEVER modify files in EIFGENs** - compiler workspace only.
- Segfaults often mean corrupted EIFGENs
- Fix: clean compile with `-clean` flag

### Agent Local Variable Access

Inline agents cannot access local variables:

```eiffel
-- WRONG
local l_flag: BOOLEAN
do
    my_call (agent do l_flag := True end)  -- Can't access l_flag

-- CORRECT: Use class attribute
handler_executed: BOOLEAN

mark_executed do handler_executed := True end

test_something
    do
        handler_executed := False
        my_call (agent mark_executed)
        assert (handler_executed)
    end
```

---

## Part 4: Semantic Frame Naming

Eiffel allows multiple names for one feature. Use this for domain vocabulary:

```eiffel
feature -- Access

    name,
    account_holder,    -- banking context
    username,          -- auth context
    gamertag,          -- gaming context
    handle: STRING_32  -- social context
        attribute end
```

**Rules:**
- Only works for routines, NOT attributes (`x, y: INTEGER` creates TWO attributes)
- Never remove names (breaking change)
- Document semantic frames in notes
- Keep generic name first

**Semantic Frame Stability Rule:**

New semantic aliases MAY ONLY be added when:
- The concept is stable across the entire Simple Eiffel ecosystem
- The name will not vary by domain context

A semantic alias is considered "stable" only if:
- It appears in at least two independent simple_* libraries, **OR**
- It reflects a domain-neutral concept (`count`, `name`, `id`, `value`, `items`)

Otherwise, introduce a **new query** instead of an alias.

```eiffel
-- WRONG: Premature aliasing for unstable concept
value, payload, setting, record: STRING  -- Too many contexts = instability

-- CORRECT: Add alias only when usage is proven
value, data: STRING  -- Generic, stable across contexts

-- BETTER: New query for specific context
payload: STRING do Result := value end  -- If HTTP context needs "payload"
```

This prevents accidental API expansion and semantic drift.

For attributes, wrap with aliased query/command:

```eiffel
feature {NONE}
    internal_x: INTEGER

feature -- Access
    x_value, horizontal, x_coordinate: like internal_x
        do Result := internal_x end

feature -- Modification
    set_x, set_horizontal (v: like internal_x)
        do internal_x := v end
```

---

## Part 5: Build and Test Commands

### Compilation

```bash
# Change to project directory first!
cd /d/prod/simple_xxx

# Compile test target
ec.exe -batch -config simple_xxx.ecf -target simple_xxx_tests -c_compile

# Clean compile (fixes EIFGENs corruption)
ec.exe -batch -config simple_xxx.ecf -target simple_xxx_tests -c_compile -clean

# Run tests
./EIFGENs/simple_xxx_tests/W_code/simple_xxx.exe
```

**Critical**: Always `cd` to project directory before compiling. EIFGENs location depends on current directory.

### Environment Variables

Set for each simple_* library:
```bash
export SIMPLE_JSON=/d/prod/simple_json
export SIMPLE_TESTING=/d/prod/simple_testing
# etc.
```

---

## Part 6: API Facades Pattern

The ecosystem uses layered API facades:

1. **simple_foundation_api** - Core utilities (json, file, process, env, datetime, etc.)
2. **simple_service_api** - Web services (jwt, smtp, sql, cors, rate limiting, etc.)
3. **simple_app_api** - Full application (foundation + service + web + htmx)
4. **simple_platform_api** - Windows platform (win32, registry, clipboard, etc.)

Import the appropriate facade rather than individual libraries.

---

## Part 7: Review Checklist

When reviewing Eiffel code, verify:

1. **Contracts**: Every public feature has require/ensure
2. **Void safety**: All paths handle detachable properly
3. **SCOOP**: No direct calls on separate attributes
4. **Naming**: Query names describe what they return (not `get_x`)
5. **Testing**: Tests inherit TEST_SET_BASE, not EQA_TEST_SET
6. **ECF**: Uses SCOOP capability, environment variables for paths
7. **Iteration**: Correct cursor access pattern for collection type
8. **Equality**: Uses `~` for value comparison in collections

---

## Part 8: Common Errors Reference

| Error | Cause | Fix |
|-------|-------|-----|
| VAPE | Precondition references private feature | Add public status query |
| VDUS(3) | Undefine on deferred feature | Remove undefine clause |
| VJAR | String type mismatch | Explicit `.to_string_8` |
| VEEN | Unknown identifier | Check spelling, check export |
| VOIT(2) | Loop variable same as feature | Rename loop variable |
| VGCC(2) | Create on deferred class | Use effective descendant |

---

## Part 9: Restricted Modification Zones (Radioactive Areas)

The following areas MUST NOT be modified by unsupervised AI agents:

### 1. Protocol Framing / I/O
- LSP message framing (`read_message`, `write_message`)
- HTTP header generation
- Any `Content-Length` calculations
- Line ending handling (CRLF vs LF)

### 2. Byte/Encoding Boundaries
- UTF-8 byte counting vs character counting
- Binary file I/O
- Base64 encoding/decoding core logic

### 3. ECF Capability Configuration
- `concurrency` settings
- `void_safety` settings
- Library `uuid` values

### 4. Inline C ABI Signatures
- External function signatures
- Pointer arithmetic
- Memory allocation/deallocation
- Windows API call conventions

### 5. Persistence & Schema Boundaries
- Database schema definitions (SQLite tables, columns)
- On-disk file formats
- Serialized JSON field names (API contracts)
- Cache key formats

**Changes to these areas require explicit human approval.**

These zones are "radioactive" because:
- Errors are subtle and intermittent
- Tests may pass while production fails
- Effects are cross-cutting (affect entire system)
- Agents naturally want to "clean up" this code

---

## Summary

### OOSC2 Principles
1. **Seamlessness** - the code *is* the design, not scaffolding
2. **ADT discipline** - contracts describe abstract behavior, not representation
3. **Correctness ≠ robustness** - preconditions define correctness limits
4. **Substitutability** - inheritance preserves contract compatibility

### Ecosystem Standards
5. **Design by Contract everywhere** - preconditions, postconditions, invariants
6. **Void safety always** - check attachment before use
7. **SCOOP compatible** - all simple_* libs support concurrency
8. **simple_* over ISE** - use ecosystem libraries
9. **Inline C pattern** - no separate .c files
10. **TEST_SET_BASE** - not EQA_TEST_SET
11. **testing/ folder** - standardized test location

---

**Version**: 1.1 (OOSC2-Integrated)
**Ratified by**: Claude (executor) + ChatGPT 5.2 (reviewer)
**Date**: December 2025

This document provides the foundation to assist effectively with the Simple Eiffel ecosystem.
