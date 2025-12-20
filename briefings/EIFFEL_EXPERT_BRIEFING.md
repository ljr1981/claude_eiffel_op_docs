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


### Creation Procedure Design Principle

**RULE: Don't let client code dictate supplier design.**

When a client calls `create obj.make` but the supplier has no `make`, ask these questions before "fixing" anything:

**The Design Hierarchy of Questions:**

1. **Does this feature NEED initialization at all?** (Or is it computed/derived?)
2. **If yes, must it be at creation time?** (Or can it use lazy initialization via `once`?)
3. **If eager, for WHICH creation contexts?** (Different makes can initialize different subsets)
4. **Will the object ever need re-initialization?** (If yes, `once` features are wrong)
5. **What arguments does initialization need?** (Factory features vs direct create)

**Anti-Pattern: Empty Makes**

```eiffel
-- WRONG: Creating empty make just to satisfy client
class FOUNDATION_API
create
    make  -- Empty, pointless

feature {NONE}
    make do end  -- Does nothing - why does it exist?
```

If a supplier was designed without `make` (using `default_create`), and the supplier author knew what they were doing, **fix the client**:

```eiffel
-- Client fix: use default_create
create foundation  -- NOT create foundation.make
```

**When `make` IS Necessary:**

- **Required parameters**: Connection strings, file paths, configuration
- **Immediate resource acquisition**: Open files, establish connections
- **Invariant establishment**: State that must be non-default from creation
- **Explicit dependency injection**: Dependencies provided at creation

**Lazy Initialization Pattern (Preferred When Applicable):**

```eiffel
class FOUNDATION_API
-- No explicit make needed - uses default_create
-- Helpers created lazily on first access

feature -- Lazy helpers
    json_helper: JSON_HELPER
        once
            create Result.make
        end

    http_helper: HTTP_HELPER
        once
            create Result.make
        end
```

**Multiple Makes for Different Contexts:**

```eiffel
create
    make,                    -- Minimal: lazy everything
    make_with_connection,    -- Eager: connection; lazy: rest
    make_full                -- Eager: everything initialized

feature {NONE} -- Initialization

    make
        do
            -- Minimal setup, rest is lazy
        end

    make_with_connection (a_conn: CONNECTION)
        require
            conn_valid: a_conn /= Void
        do
            make
            connection := a_conn  -- Eager for this context
        ensure
            connection_set: connection = a_conn
        end
```

**Re-initialization Changes Everything:**

If `make` can be called for RE-initialization (resetting object state):

- `once` features are **wrong** (cannot be reset)
- Once-per-object is **also wrong** (same problem)
- Use factory features + setters instead:

```eiffel
feature -- Factory (can be called repeatedly)
    new_helper (a_config: CONFIG): HELPER
        do
            create Result.make (a_config)
        end

feature -- Initialization AND Re-initialization
    make (a_config: CONFIG)
        do
            helper := new_helper (a_config)  -- Fresh each time
        end

    reset
        do
            make (default_config)  -- Re-initialize to defaults
        end
```

**Contract Implications:**

Different creation procedures have different postconditions:

```eiffel
make
    ensure
        minimal_state: is_initialized
        -- Other features may be Void/lazy

make_full
    ensure
        full_state: is_initialized
        connection_ready: connection /= Void
        cache_ready: cache /= Void
```

This connects to void safety - postconditions declare what's guaranteed attached after each creation procedure.

**Summary:**

> Understand the full lifecycle of the object and its features. Design creation procedures that match actual client contexts, not imagined ones. Lazy is often right. Multiple makes serving different needs is often right. Empty makes serving no purpose are **always wrong**.

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

### UUID Collisions (Critical!)

**Symptom**: "Unknown class" errors for classes that clearly exist, circular dependency warnings when there shouldn't be any.

**Cause**: Multiple ECF files sharing the same UUID. The compiler uses UUIDs to identify libraries - when 4 libraries claim the same UUID, the compiler thinks they're ALL the same library and gets confused about dependencies.

**Diagnosis**:
```bash
# Find duplicate UUIDs across all ECFs
grep -r 'uuid="' simple_*/*.ecf | sed 's/.*uuid="\([^"]*\)".*/\1/' | sort | uniq -d

# Find which files have the duplicate
grep -l 'uuid="THE-DUPLICATE-UUID"' simple_*/*.ecf
```

**Fix**: Generate new UUIDs for duplicates using `uuidgen`.

**Prevention**: **NEVER copy ECF files or UUIDs between projects.** Always run `uuidgen` for new projects.

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

The ecosystem uses layered API facades to provide unified access to functionality while hiding internal library complexity.

### Facade Hierarchy

| Layer | API Facade | Consumes | Provides |
|-------|-----------|----------|----------|
| **Foundation** | `simple_foundation_api` | 25+ core libs | JSON, file, env, hash, config, datetime, regex, xml, yaml |
| **Services** | `simple_service_api` | foundation_api + 20 service libs | JWT, SMTP, SQL, web, websocket, cache, PDF, scheduler |
| **Platform** | `simple_platform_api` | win32_api + OS libs | Registry, clipboard, mmap, IPC, system info, file watcher |
| **Application** | `simple_app_api` | service_api + platform_api | Full application capabilities |

**Key Rule**: Each layer ONLY consumes the layer directly below. Clients use ONE facade, never reach through to enclosed libraries.

### ECF Structure

A facade ECF includes ALL underlying libraries directly:

```xml
<system name="simple_foundation_api" uuid="..." library_target="simple_foundation_api">
    <target name="simple_foundation_api">
        <root all_classes="true"/>
        <capability>
            <concurrency support="scoop" use="thread"/>
            <void_safety support="all"/>
        </capability>

        <!-- Core dependencies -->
        <library name="base" location="$ISE_LIBRARY\library\base\base.ecf"/>

        <!-- ALL underlying simple_* libraries included directly -->
        <library name="simple_json" location="$SIMPLE_EIFFEL\simple_json\simple_json.ecf"/>
        <library name="simple_file" location="$SIMPLE_EIFFEL\simple_file\simple_file.ecf"/>
        <library name="simple_hash" location="$SIMPLE_EIFFEL\simple_hash\simple_hash.ecf"/>
        <!-- ... 20+ more libraries ... -->

        <cluster name="src" location=".\src\" recursive="true"/>
    </target>
</system>
```

**Critical**: Client ECFs only need `<library name="simple_foundation_api" .../>` - they get all underlying libs transitively.

### Main Facade Class Implementation

The facade class (e.g., `FOUNDATION_API`) provides **three types of features** per domain:

#### 1. Convenience Methods (One-Liners)

Direct operations that clients call most often:

```eiffel
feature -- Hashing convenience

    sha256 (a_input: STRING): STRING
            -- SHA-256 hash of `a_input`
        require
            input_valid: a_input /= Void
        do
            Result := hasher.sha256 (a_input)
        ensure
            result_attached: Result /= Void
            result_64_chars: Result.count = 64
        end
```

#### 2. Factory Methods (Return Configured Objects)

When clients need objects for extended interaction:

```eiffel
feature -- Factory

    new_logger (a_name: STRING): SIMPLE_LOGGER
            -- New logger with `a_name`
        require
            name_valid: a_name /= Void and then not a_name.is_empty
        do
            create Result.make (a_name)
        ensure
            result_attached: Result /= Void
        end
```

#### 3. Direct Access (Expose Underlying Helpers)

When clients need full control over an underlying system:

```eiffel
feature -- Direct access

    json: JSON_HELPER
            -- Direct access to JSON helper for complex operations
        do
            Result := json_internal
        end
```

### Lazy Initialization Pattern

**Critical Design**: Facades use `once ("OBJECT")` for lazy initialization - no explicit creation procedure needed:

```eiffel
class
    FOUNDATION_API

-- No creation procedure listed - uses default_create
-- Client just: create foundation

feature {NONE} -- Internal helpers (lazy initialized)

    hasher: SIMPLE_HASH
            -- Hash helper, created on first access
        once ("OBJECT")
            create Result.make
        end

    json_internal: JSON_HELPER
            -- JSON helper, created on first access
        once ("OBJECT")
            create Result.make
        end
```

**Why `once ("OBJECT")`**:
- `once ("PROCESS")` - single instance per process (wrong: all facades share)
- `once ("THREAD")` - single instance per thread (wrong: loses object isolation)
- `once ("OBJECT")` - single instance per object (correct: each facade has own helpers)

### Multiple Aliases for Discoverability

Provide multiple names for the same feature to improve API discoverability:

```eiffel
feature -- Hashing

    sha256,
    hash_sha256,
    sha256_hash (a_input: STRING): STRING
            -- SHA-256 hash of `a_input`
            -- Multiple aliases for discoverability
        do
            Result := hasher.sha256 (a_input)
        end
```

### Backward Compatibility Alias Class

For renaming main class without breaking clients:

```eiffel
class FOUNDATION inherit FOUNDATION_API end
```

### Client Usage Pattern

Clients inherit or create the facade:

```eiffel
class MY_APPLICATION
inherit
    FOUNDATION_API  -- Inherit for direct access

feature -- Operations
    process_data (a_json_text: STRING)
        do
            -- Convenience method
            print (sha256 (a_json_text))

            -- Direct access when needed
            if json.is_valid (a_json_text) then
                -- complex operations
            end
        end
```

Or create as attribute:

```eiffel
class MY_SERVICE
feature {NONE} -- Initialization
    make
        do
            create foundation  -- Uses default_create
        end

feature {NONE} -- Implementation
    foundation: FOUNDATION_API
```

### Anti-Pattern: Reaching Through

**WRONG**: Client reaching through facade to enclosed library:

```xml
<!-- Client ECF -->
<library name="simple_foundation_api" location="..."/>
<library name="simple_hash" location="..."/>  <!-- WRONG: reaching through -->
```

```eiffel
-- Client code using enclosed library directly
local l_hasher: SIMPLE_HASH  -- WRONG
do create l_hasher.make
```

**CORRECT**: Use facade features only:

```eiffel
inherit FOUNDATION_API
do Result := sha256 (input)  -- Use facade feature
```

### Benefits

1. **Single ECF dependency** - clients include one library, get everything
2. **Stable API** - internal changes don't affect clients
3. **Discoverability** - all features in one class with aliases
4. **Lazy loading** - helpers created only when used
5. **Testability** - mock facade for testing, not 20 individual libs
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

## Part 10: DBC Logging Pattern

For applications that need troubleshooting support, implement DBC-aware logging that traces the contract evaluation flow.

### Purpose

Logging isn't just about failures - it's about traceability. A DBC log shows:
- What features were called (call stack)
- What preconditions were evaluated
- What postconditions were satisfied
- What check assertions passed
- Complete picture of execution flow

This allows users to send logs with bug reports, showing exactly what happened even when nothing "failed".

### Implementation Pattern

```eiffel
feature -- Execution with logging

    execute_with_logging (a_input: STRING): STRING
            -- Execute operation with full DBC trace
        require
            input_valid: log_precondition ("input_valid", not a_input.is_empty)
        local
            l_result: STRING
        do
            log_entry ("execute_with_logging", "input=" + truncate(a_input, 50))

            -- Core logic
            l_result := process (a_input)

            check
                intermediate_valid: log_check ("result_not_void", l_result /= Void)
            end

            Result := l_result
            log_exit ("execute_with_logging", "success")
        ensure
            result_valid: log_postcondition ("result_valid", Result.count > 0)
        end

feature {NONE} -- Logging helpers

    log_precondition (a_name: STRING; a_condition: BOOLEAN): BOOLEAN
            -- Log precondition evaluation, return condition unchanged
        do
            log_event ("PRE", a_name + "=" + a_condition.out)
            Result := a_condition
        end

    log_postcondition (a_name: STRING; a_condition: BOOLEAN): BOOLEAN
            -- Log postcondition evaluation, return condition unchanged
        do
            log_event ("POST", a_name + "=" + a_condition.out)
            Result := a_condition
        end

    log_check (a_name: STRING; a_condition: BOOLEAN): BOOLEAN
            -- Log check assertion, return condition unchanged
        do
            log_event ("CHECK", a_name + "=" + a_condition.out)
            Result := a_condition
        end

    log_entry (a_feature: STRING; a_details: STRING)
            -- Log feature entry
        do
            log_event ("ENTER", a_feature + " | " + a_details)
        end

    log_exit (a_feature: STRING; a_status: STRING)
            -- Log feature exit
        do
            log_event ("EXIT", a_feature + " | " + a_status)
        end
```

### Log Output Example

```
[14:32:01] ENTER | execute_with_logging | input=print ("Hello World")
[14:32:01] PRE | input_valid=True
[14:32:02] CHECK | result_not_void=True
[14:32:02] POST | result_valid=True
[14:32:02] EXIT | execute_with_logging | success
```

### Benefits

1. **Full traceability** - See exactly what happened in any session
2. **DBC verification** - Confirm contracts were evaluated correctly
3. **Bug reports** - Users can attach logs showing the problem
4. **Performance profiling** - Timestamps show where time is spent
5. **Regression detection** - Compare logs across versions

### Key Principles

- Logging wraps conditions, never changes them (pure functions)
- Log at boundaries: feature entry/exit, contract evaluation
- Include version in log header for issue tracking
- Use rescue clauses to ensure logging never crashes the app
- Truncate long strings to keep logs readable

### Simplified Caller/Supplier Pattern (No Agents)

For simpler cases where modifying contracts isn't practical, use explicit caller/supplier logging:

```eiffel
feature {NONE} -- DBC trace logging

    log_dbc_call (a_caller, a_supplier, a_preconditions: STRING)
            -- Log caller-side: about to call supplier with preconditions met
        do
            log_event ("→CALL", a_caller + " → " + a_supplier + " | pre: " + a_preconditions)
        end

    log_dbc_return (a_supplier, a_postconditions: STRING)
            -- Log supplier-side: returning with postconditions met
        do
            log_event ("←RET", a_supplier + " | post: " + a_postconditions)
        end

    b (a_bool: BOOLEAN): STRING
            -- Boolean to compact string: T or F
        do
            if a_bool then Result := "T" else Result := "F" end
        end
```

Usage in caller:

```eiffel
execute_code (a_code: STRING)
    local
        l_cell_count_before: INTEGER
    do
        -- DBC trace: caller logs preconditions BEFORE call
        l_cell_count_before := notebook.cell_count
        log_dbc_call ("CLI.execute_code", "NOTEBOOK.run",
            "code_not_empty=" + b(not a_code.is_empty) +
            ", cell_count=" + l_cell_count_before.out)

        l_result := notebook.run (a_code)

        -- DBC trace: log postconditions AFTER return
        log_dbc_return ("NOTEBOOK.run",
            "result_attached=" + b(l_result /= Void) +
            ", cell_added=" + b(notebook.cell_count > l_cell_count_before))
    end
```

Log output:

```
[10:15:01] →CALL | CLI.execute_code → NOTEBOOK.run | pre: code_not_empty=T, cell_count=0
[10:15:02] ←RET | NOTEBOOK.run | post: result_attached=T, cell_added=T
```

**When to use each pattern:**
- **Agent-in-contract pattern**: When you want logging to be part of the contract definition itself
- **Caller/supplier pattern**: When you want explicit tracing without modifying existing contracts

**Future work**: Apply this pattern to all simple_* CLI apps and test harnesses for ecosystem-wide traceability.

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
