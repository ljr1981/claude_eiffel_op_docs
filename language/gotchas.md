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

## WSF/EWF Library Issues

### WGI_INPUT_STREAM.read_string is a Procedure
- **Docs say**: (expected) Functions return values directly
- **Reality**: `read_string(n)` is a PROCEDURE that populates `last_string` attribute
- **Verified**: 2025-12-02, EiffelStudio 25.02
- **Example**:
```eiffel
-- WRONG: Treating read_string as a function
l_body := wsf_request.input.read_string (content_length)  -- ERROR

-- CORRECT: Call procedure, then access attribute
wsf_request.input.read_string (content_length)
l_body := wsf_request.input.last_string
```

### WSF_VALUE.string_representation Returns READABLE_STRING_32
- **Docs say**: (not documented)
- **Reality**: Use `string_representation` directly, not `as_string.to_string_32`
- **Verified**: 2025-12-02, EiffelStudio 25.02
- **Example**:
```eiffel
-- WRONG: as_string returns WSF_STRING, not a string
Result := l_param.as_string.to_string_32  -- ERROR

-- CORRECT: Use string_representation directly
Result := l_param.string_representation.to_string_32
```

### ECF Environment Variable References
- **Docs say**: (limited documentation)
- **Reality**: Use `$ENV_VAR` syntax in ECF location attributes; avoid hard-coded paths
- **Verified**: 2025-12-02, EiffelStudio 25.02
- **Example**:
```xml
<!-- WRONG: Hard-coded path -->
<library name="framework" location="D:\prod\framework\framework.ecf"/>

<!-- CORRECT: Environment variable reference -->
<library name="framework" location="$FRAMEWORK\framework.ecf"/>
```
- **Setup**: Set environment variables at User level via PowerShell:
```powershell
[System.Environment]::SetEnvironmentVariable('FRAMEWORK', 'D:\prod\framework', 'User')
```

---

## Test Framework

### Use TEST_SET_BASE, Not EQA_TEST_SET
- **Docs say**: Inherit from `EQA_TEST_SET` for test classes
- **Reality**: Use `TEST_SET_BASE` from testing_ext library for consistent assertions and helpers
- **Verified**: 2025-12-02, EiffelStudio 25.02
- **Example**:
```eiffel
-- WRONG: Standard EQA test set
class MY_TEST
inherit
    EQA_TEST_SET

-- CORRECT: Extended test set with helpers
class MY_TEST
inherit
    TEST_SET_BASE
```

### ECF Test Target Configuration
- **Docs say**: (limited documentation on AutoTest configuration)
- **Reality**: Test targets need a simple root class; EQA_EVALUATOR is frozen and cannot be inherited
- **Verified**: 2025-12-05, EiffelStudio 25.02
- **Complete Setup**:

1. **ECF Target Configuration**:
```xml
<target name="my_project_tests" extends="my_project">
    <root class="MY_TEST_APP" feature="make"/>
    <library name="testing" location="$ISE_LIBRARY\library\testing\testing.ecf"/>
    <library name="testing_ext" location="$TESTING_EXT\testing_ext.ecf"/>
    <cluster name="tests" location=".\tests\" recursive="true"/>
</target>
```

2. **Root Class (MY_TEST_APP.e)** - Just a placeholder, AutoTest handles discovery:
```eiffel
class MY_TEST_APP
create
    make
feature -- Initialization
    make
        do
            do_nothing
        end
end
```

3. **Test Set Classes** - Inherit from TEST_SET_BASE:
```eiffel
class MY_FEATURE_TEST_SET
inherit
    TEST_SET_BASE
feature -- Tests
    test_something
        do
            assert ("description", some_condition)
        end
end
```

- **Key Points**:
  - Root class is just for compilation; AutoTest discovers test_* features automatically
  - Do NOT inherit from EQA_EVALUATOR (frozen class error)
  - Do NOT use `all_classes="true"` as root - need explicit class
  - Test features must start with `test_` prefix

### Percent Signs in Test Assertions
- **Docs say**: (not documented)
- **Reality**: Percent signs in manifest strings must be escaped as `%%`
- **Verified**: 2025-12-05, EiffelStudio 25.02
- **Example**:
```eiffel
-- WRONG: Syntax error "incomplete string: missing final quote"
assert ("has_percent", l_html.has_substring ("41%"))

-- CORRECT: Escape the percent sign
assert ("has_percent", l_html.has_substring ("41%%"))
```

---

## Windows Command Execution

### SIMPLE_PROCESS_HELPER and Environment Variables
- **Docs say**: (internal) Pass command line to `output_of_command`
- **Reality**: The helper splits by spaces, breaking `VAR="value" command` syntax
- **Verified**: 2025-12-02, EiffelStudio 25.02
- **Solution**: Wrap in `cmd /c "set VAR=value && command"` pattern
- **Example**:
```eiffel
-- WRONG: Split by space breaks this
l_cmd := "SIMPLE_JSON=%"D:\prod\simple_json%" ec.exe -batch ..."

-- CORRECT: Use cmd /c with set commands
l_cmd := "cmd /c %"set SIMPLE_JSON=D:\prod\simple_json && ec.exe -batch ...%""
```

### Working Directory for External Processes
- **Docs say**: (not documented)
- **Reality**: Pass directory as second parameter to `output_of_command`
- **Verified**: 2025-12-02, EiffelStudio 25.02
- **Example**:
```eiffel
-- Run ec.exe in project directory so EIFGENs is created there
l_output := process_helper.output_of_command (l_cmd, project_directory)
```

---

## Collection Equality

### ARRAYED_LIST.has Uses Reference Equality
- **Docs say**: (expected) `has` checks if item is in list
- **Reality**: `has` uses `=` (reference equality), not `~` (value equality)
- **Verified**: 2025-12-02, EiffelStudio 25.02
- **Example**:
```eiffel
-- WRONG: Uses reference equality - won't find equal string
if l_origins.has ("https://example.com") then ...

-- CORRECT: Use across with value equality operator
if across l_origins as ic some ic ~ "https://example.com" end then ...

-- Or create a helper function
has_origin (a_list: ARRAYED_LIST [STRING]; a_origin: STRING): BOOLEAN
    do
        Result := across a_list as ic some ic ~ a_origin end
    end
```

### HASH_TABLE Iteration: key_for_iteration Inside across Loop
- **Docs say**: (expected) `across` loop works uniformly for all collections
- **Reality**: `across` loop creates a separate cursor; internal cursor methods like `key_for_iteration` use a DIFFERENT cursor
- **Verified**: 2025-12-03, EiffelStudio 25.02
- **Example**:
```eiffel
-- WRONG: key_for_iteration uses internal cursor, not across cursor
across my_table as ic loop
    print (my_table.key_for_iteration)  -- PRECONDITION VIOLATION: not_off
end

-- CORRECT: Use from/until/loop with proper cursor methods
from my_table.start until my_table.after loop
    print (my_table.key_for_iteration)  -- OK
    print (my_table.item_for_iteration) -- OK
    my_table.forth
end
```

---

## Agent Limitations

### Inline Agents Cannot Access Local Variables
- **Docs say**: (not clear)
- **Reality**: Inline agents cannot capture or modify local variables
- **Verified**: 2025-12-02, EiffelStudio 25.02
- **Example**:
```eiffel
-- WRONG: Inline agent can't access l_executed
local
    l_executed: BOOLEAN
do
    l_pipeline.execute (request, response, agent do l_executed := True end)
    assert (l_executed)

-- CORRECT: Use class attribute and named agent
feature {NONE} -- Test state
    handler_executed: BOOLEAN

    mark_handler_executed
        do
            handler_executed := True
        end

feature -- Test
    test_pipeline
        do
            handler_executed := False
            l_pipeline.execute (request, response, agent mark_handler_executed)
            assert (handler_executed)
        end
```

---

## Multiple Inheritance

### VDUS(3) - Cannot Undefine Deferred Features
- **Docs say**: Use `undefine` to resolve inheritance conflicts
- **Reality**: `undefine` converts effective → deferred, NOT the reverse. You cannot undefine an already-deferred feature.
- **Verified**: 2025-12-03, EiffelStudio 25.02
- **Example**:
```eiffel
-- WRONG: render_canvas is already deferred in GDS_HTML_RENDERER
class GUI_DESIGNER_SERVER
inherit
    GDS_HTMX_HANDLERS
        undefine
            render_canvas  -- ERROR: VDUS(3) - feature is deferred!
        end
    GDS_HTML_RENDERER

-- CORRECT: Just inherit both - feature joining handles it automatically
class GUI_DESIGNER_SERVER
inherit
    GDS_HTMX_HANDLERS
    GDS_HTML_RENDERER
    -- Eiffel joins the deferred features automatically
```

### Feature Joining Rules
- **Deferred + Deferred**: Single deferred feature (one implementation needed)
- **Deferred + Effective**: Effective satisfies deferred (automatic, no clause needed)
- **Effective + Effective**: Conflict - must use `rename`, `undefine`, or `select`

---

## Contract Completeness (The "True but Incomplete" Problem)

### Contracts Can Be Correct Yet Insufficient
- **Docs say**: Write postconditions to specify what a feature guarantees
- **Reality**: A postcondition can be *true* but *incomplete*, passing verification while missing important guarantees
- **Source**: Meyer, "AI for software engineering: from probable to provable" (CACM 2025)
- **Verified**: 2025-12-03

**Why This Matters for AI-Generated Code:**
AI produces "statistically likely" contracts. These are often true but may miss edge cases or fail to capture full behavior. Human review for completeness is essential.

**Example - Incomplete Contract:**
```eiffel
add_item (a_item: STRING)
    require
      item_not_void: a_item /= Void
    do
      items.extend (a_item)
    ensure
      has_item: items.has (a_item)  -- TRUE but INCOMPLETE!
    end
```

**What's Missing:**
```eiffel
    ensure
      has_item: items.has (a_item)
      count_increased: items.count = old items.count + 1
      at_end: items.last ~ a_item
      others_unchanged: across 1 |..| (old items.count) as i all
                          items[i] ~ (old items.twin)[i]
                        end
```

**Common Incomplete Patterns:**

| Operation | Incomplete | Complete Addition |
|-----------|------------|-------------------|
| Add to collection | `has (item)` | `count = old count + 1` |
| Remove from collection | `not has (item)` | `count = old count - 1` |
| Set attribute | `attr = value` | `old other_attr = other_attr` (unchanged) |
| Clear collection | `is_empty` | `count = 0` |
| Search | `Result /= Void implies has (Result)` | `Result = Void implies not has (target)` |

**Defense Strategy:**
1. After writing postcondition, ask: "What ELSE is guaranteed?"
2. For state changes, explicitly state what did NOT change
3. For counts, always specify old vs new relationship
4. Use Specification Hat workflow (contracts before code)

**Reference**: See `contract_patterns.md` for systematic patterns

---

## HTML Generation

### simple_htmx escape_html Breaks JavaScript in Attributes
- **Docs say**: Attribute values are properly escaped for HTML
- **Reality**: `escape_html` escapes `<`, `>`, `&`, `"` which breaks JavaScript in Alpine.js attributes
- **Verified**: 2025-12-03, EiffelStudio 25.02
- **Example**:
```eiffel
-- WRONG: Arrow functions get escaped
l_div.x_data ("{ init() { items = [] } }")  -- OK
l_div.x_init ("$watch('value', v => console.log(v))")  -- BROKEN: => becomes &gt;

-- Also broken:
l_div.x_show ("count > 5")       -- > becomes &gt;
l_div.x_if ("a && b")            -- && becomes &amp;&amp;

-- CORRECT: Avoid arrow functions, use alternative patterns
l_div.x_effect ("console.log(value)")  -- Use x-effect instead of $watch with arrow
l_div.x_show ("count >= 5")            -- Rewrite comparisons where possible
```
- **Workaround**: Avoid arrow functions (`=>`), logical AND (`&&`), and comparisons with `<` or `>` in Alpine attribute values. Use `x_effect` instead of `$watch` with callbacks.
- **Future fix**: Add `attr_raw` method to simple_htmx for unescaped attribute values.

---

## Named Tuple Field Access in Across Loops

### Cannot Access Named Tuple Fields via across Cursor
- **Docs say**: (expected) Tuples with named fields can be accessed via dot notation
- **Reality**: Named tuple field access (`tuple.field_name`) doesn't work when iterating with `across` loops
- **Verified**: 2025-12-04, EiffelStudio 25.02
- **Example**:
```eiffel
-- WRONG: VUAR(1) error - wrong number of actual arguments
pages: ARRAY [TUPLE [title: STRING; url: STRING; description: STRING]]
...
across pages as page loop
    -- Cannot access: page.item.title, page.item.url, page.item.description
    l_menu.add_item (page.item.title, page.item.url)  -- ERROR!
end

-- CORRECT: Use parallel arrays instead
page_titles: ARRAY [STRING]
    once Result := <<"Home", "About", "Contact">> end

page_urls: ARRAY [STRING]
    once Result := <<"/", "/about", "/contact">> end

feature -- Generation
    from i := 1 until i > page_count loop
        l_menu.add_item (page_titles[i], page_urls[i])
        i := i + 1
    end
```
- **Alternative**: Use explicit local variables with positional tuple access `page.item.item_1` if needed

---

## Custom Scroll Containers (Alpine.js / JavaScript)

### window.scrollY Doesn't Track Custom Scroll Containers
- **Docs say**: (Alpine.js/JavaScript general knowledge) `window.scrollY` tracks page scroll position
- **Reality**: When using CSS `overflow-y: scroll` on a container (e.g., snap scrolling), `window.scrollY` remains 0. Must track container scroll directly.
- **Verified**: 2025-12-04
- **Example**:
```javascript
// WRONG: Always returns 0 when custom container handles scrolling
x-init="window.addEventListener('scroll', () => { visible = window.scrollY > 100 })"

// CORRECT: Detect and listen to the scroll container
x-init="
  const container = document.querySelector('.snap-container');
  if (container) {
    container.addEventListener('scroll', () => { visible = container.scrollTop > 100 });
  } else {
    window.addEventListener('scroll', () => { visible = window.scrollY > 100 });
  }"
```
- **Common Containers**: `.snap-container`, modal overlays, custom scroll areas

---

## Citation and URL Verification

### AI-Generated URLs May Be Hallucinated
- **Reality**: AI models can generate plausible-looking URLs that don't exist. Always verify external links.
- **Verified**: 2025-12-04
- **Examples of Hallucinated URLs** (all returned 404):
  - `uplevelteam.com/blog/copilot-code-quality-study` → Should be `ai-for-developer-productivity`
  - `veracode.com/state-of-software-security-report` → Should be `blog/ai-generated-code-security-risks/`
  - `answer.ai/posts/2024-09-10-devin-benchmark.html` → Should be `2025-01-08-devin.html`
  - `apiiro.com/blog/ai-generated-code-security-risks/` → Should be `4x-velocity-10x-vulnerabilities...`
- **Defense**:
  1. Test all external links before committing
  2. Use WebFetch or curl to verify URLs return 200
  3. When citing research, verify the actual URL from the source website

---

## Pending Investigation

### Across Loop Item Access
- **Status**: RESOLVED - see `across_loops.md`
- **Summary**: Whether you use `ic` or `ic.item` depends on the iteration cursor type. For ARRAYED_LIST, the cursor IS the item. For HASH_TABLE, use `ic.item` for value.
