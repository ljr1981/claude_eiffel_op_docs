# Verified Eiffel Patterns

This file contains code patterns that have been tested and confirmed to compile and work correctly.

---

## Format

Each pattern includes:
- Description
- Code
- Verification date and EiffelStudio version
- Any notes or caveats

---

## Iteration Patterns

### Safe Iteration with Type Checking
When iterating over collections that may contain different types:

```eiffel
across my_list as ic loop
    if attached {EXPECTED_TYPE} ic as l_item then
        -- Work with l_item
        process (l_item)
    end
end
```
**Verified**: 2025-11-29, EiffelStudio 25.02

### Index-Based Iteration
When you need both item and index:

```eiffel
across my_list as ic loop
    print ("Item " + ic.cursor_index.out + ": " + ic.out + "%N")
end
```
**Verified**: 2025-12-02, EiffelStudio 25.02

### Traditional Loop with Variant/Invariant
For loops requiring termination proof:

```eiffel
from
    i := a_list.lower
until
    i > a_list.upper
loop
    process (a_list [i])
    i := i + 1
variant
    a_list.upper - i + 1
end
```
**Verified**: 2025-12-02, EiffelStudio 25.02

---

## Contract Patterns

### Public Status Query for Preconditions
Avoid VAPE errors by exposing status queries:

```eiffel
feature -- Status
    is_ready: BOOLEAN
            -- Can `execute` be called?
        do
            Result := internal_state /= Void and then not internal_state.is_empty
        end

feature -- Execution
    execute
        require
            ready: is_ready
        do
            -- implementation
        ensure
            -- postconditions
        end

feature {NONE} -- Implementation
    internal_state: detachable STRING
```
**Verified**: 2025-11-29, EiffelStudio 25.02

### Postcondition with Old Expression
Verify state changes:

```eiffel
add_item (a_item: ITEM)
    require
        item_valid: a_item /= Void
    do
        items.extend (a_item)
    ensure
        count_increased: items.count = old items.count + 1
        item_added: items.has (a_item)
    end
```
**Verified**: 2025-12-02, EiffelStudio 25.02

---

## String Handling Patterns

### Safe STRING_32 to STRING_8 Conversion
```eiffel
feature {NONE} -- Implementation
    to_string_8_safe (a_string: READABLE_STRING_GENERAL): STRING_8
            -- Convert to STRING_8, handling potential unicode
        do
            if attached {READABLE_STRING_8} a_string as l_s8 then
                Result := l_s8.to_string_8
            else
                Result := a_string.to_string_8
            end
        end
```
**Verified**: 2025-11-29, EiffelStudio 25.02

### SQL String Escaping (Single Quotes)
```eiffel
escaped_string (a_string: READABLE_STRING_GENERAL): STRING_8
        -- Escape single quotes for SQL
    local
        i: INTEGER
        c: CHARACTER_32
    do
        create Result.make (a_string.count + 10)
        Result.append_character ('%'')
        from i := 1 until i > a_string.count loop
            c := a_string.item (i)
            if c = '%'' then
                Result.append_character ('%'')
                Result.append_character ('%'')
            else
                Result.append_character (c.to_character_8)
            end
            i := i + 1
        variant
            a_string.count - i + 1
        end
        Result.append_character ('%'')
    ensure
        starts_with_quote: Result.item (1) = '%''
        ends_with_quote: Result.item (Result.count) = '%''
    end
```
**Verified**: 2025-12-02, EiffelStudio 25.02

---

## Type Handling Patterns

### Multi-Type Value Conversion
Handle values that could be multiple numeric types:

```eiffel
value_to_string (a_value: detachable ANY): STRING_8
    do
        if a_value = Void then
            Result := "NULL"
        elseif attached {READABLE_STRING_GENERAL} a_value as l_str then
            Result := escaped_string (l_str)
        elseif attached {INTEGER_64} a_value as l_int then
            Result := l_int.out
        elseif attached {INTEGER_32} a_value as l_int32 then
            Result := l_int32.out
        elseif attached {REAL_64} a_value as l_real then
            Result := l_real.out
        elseif attached {REAL_32} a_value as l_real32 then
            Result := l_real32.out
        elseif attached {BOOLEAN} a_value as l_bool then
            Result := if l_bool then "1" else "0" end
        else
            Result := a_value.out
        end
    ensure
        result_not_void: Result /= Void
    end
```
**Verified**: 2025-12-02, EiffelStudio 25.02

---

## Error Handling Patterns

### Structured Error with Context
```eiffel
class MY_ERROR

create
    make, make_with_context

feature {NONE} -- Initialization
    make (a_code: INTEGER; a_message: STRING_8)
        do
            code := a_code
            message := a_message
        end

    make_with_context (a_code: INTEGER; a_message: STRING_8; a_context: STRING_8)
        do
            make (a_code, a_message)
            context := a_context
        end

feature -- Access
    code: INTEGER
    message: STRING_8
    context: detachable STRING_8

feature -- Output
    full_description: STRING_8
        do
            create Result.make (100)
            Result.append ("Error ")
            Result.append (code.out)
            Result.append (": ")
            Result.append (message)
            if attached context as l_ctx then
                Result.append (" [")
                Result.append (l_ctx)
                Result.append ("]")
            end
        end

invariant
    message_not_empty: not message.is_empty
end
```
**Verified**: 2025-12-02, EiffelStudio 25.02

---

## Creation Patterns

### Factory with Validation
```eiffel
class WIDGET_FACTORY

feature -- Factory
    new_widget (a_name: STRING_8; a_size: INTEGER): detachable WIDGET
            -- Create widget if parameters valid, Void otherwise
        do
            if is_valid_name (a_name) and is_valid_size (a_size) then
                create Result.make (a_name, a_size)
            end
        ensure
            valid_implies_result: (is_valid_name (a_name) and is_valid_size (a_size)) implies Result /= Void
        end

feature -- Validation
    is_valid_name (a_name: STRING_8): BOOLEAN
        do
            Result := not a_name.is_empty and a_name.count <= 100
        end

    is_valid_size (a_size: INTEGER): BOOLEAN
        do
            Result := a_size > 0 and a_size <= 1000
        end
end
```
**Verified**: 2025-12-02, EiffelStudio 25.02

---

## Class Structure Pattern

### Standard Class Layout
```eiffel
note
    description: "Brief description of class purpose"
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
            -- Public access to data
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
            -- Do the main work
        require
            valid: is_valid
        do
            -- implementation
        end

feature {NONE} -- Implementation

    internal_data: TYPE

invariant
    data_consistent: is_valid implies internal_data /= Void

end
```
**Verified**: 2025-12-02, EiffelStudio 25.02

---

## HTTP Server Patterns (EWF/WSF)

### Basic WSF Server with Execution Handler
```eiffel
class MY_SERVER
inherit
    WSF_DEFAULT_SERVICE [MY_EXECUTION]
        redefine
            initialize
        end
create
    make
feature {NONE} -- Initialization
    make (a_port: INTEGER)
        do
            port := a_port
            initialize
            set_service_option ("port", a_port)
        end

    initialize
        do
            Precursor
        end
feature -- Access
    port: INTEGER
feature -- Server Control
    start
        do
            print ("Starting server on port " + port.out + "...%N")
            launch (service_options)
        end
end

class MY_EXECUTION
inherit
    WSF_EXECUTION
create
    make
feature -- Execution
    execute
        do
            -- Access request with: request.path_info, request.request_method
            -- Send response with: response.set_status_code, response.put_string
            response.set_status_code (200)
            response.put_header_text ("Content-Type: text/plain%R%N%R%N")
            response.put_string ("Hello World")
        end
end
```
**Verified**: 2025-12-02, EiffelStudio 25.02

### Singleton Router Pattern (Once Function)
Share state between server and execution instances using once functions:

```eiffel
-- In both MY_SERVER and MY_EXECUTION:
router: MY_ROUTER
        -- Shared singleton (same instance everywhere)
    once
        create Result
    end
```

**Why**: WSF creates new execution instances per request. Once functions ensure the same router instance is shared across all.
**Verified**: 2025-12-02, EiffelStudio 25.02

### Agent-Based Route Handler
```eiffel
class MY_SERVER
feature -- Route Registration
    on_get (a_pattern: STRING; a_handler: PROCEDURE [MY_REQUEST, MY_RESPONSE])
        local
            l_route: MY_ROUTE
        do
            create l_route.make ("GET", a_pattern, a_handler)
            router.add_route (l_route)
        end
feature {NONE} -- Handlers (example)
    handle_users (a_request: MY_REQUEST; a_response: MY_RESPONSE)
        do
            a_response.send_json ("{%"users%":[]}")
        end
end

-- Usage:
create server.make (8080)
server.on_get ("/api/users", agent handle_users)
server.on_get ("/api/users/{id}", agent handle_user)
server.start
```
**Verified**: 2025-12-02, EiffelStudio 25.02

### URL Pattern Matching with Path Parameters
```eiffel
feature -- Matching
    path_matches (a_path: STRING_32): BOOLEAN
            -- Does `a_path' match this route's pattern?
            -- Pattern "/users/{id}" matches "/users/123"
        local
            l_path_segs, l_pattern_segs: LIST [STRING_32]
            i: INTEGER
        do
            l_path_segs := a_path.split ('/')
            l_pattern_segs := pattern.to_string_32.split ('/')
            -- Remove empty segments from leading/trailing slashes
            -- ... (cleanup code)
            if l_path_segs.count = l_pattern_segs.count then
                Result := True
                from i := 1 until i > l_pattern_segs.count or not Result loop
                    if is_parameter (l_pattern_segs.i_th (i)) then
                        Result := not l_path_segs.i_th (i).is_empty
                    else
                        Result := l_path_segs.i_th (i).same_string (l_pattern_segs.i_th (i))
                    end
                    i := i + 1
                end
            end
        end

    is_parameter (a_segment: STRING_32): BOOLEAN
        do
            Result := a_segment.count >= 2 and then
                      a_segment.item (1) = '{' and then
                      a_segment.item (a_segment.count) = '}'
        end
```
**Verified**: 2025-12-02, EiffelStudio 25.02

### Reading Request Body from WSF
```eiffel
body: STRING_8
        -- Request body as string
    local
        l_length: INTEGER
    do
        l_length := wsf_request.content_length_value.to_integer_32.max (0)
        if l_length > 0 and then not wsf_request.input.end_of_input then
            wsf_request.input.read_string (l_length)
            Result := wsf_request.input.last_string
        else
            create Result.make_empty
        end
    end
```
**Note**: `read_string` is a PROCEDURE, not a function. It populates `last_string`.
**Verified**: 2025-12-02, EiffelStudio 25.02

### Middleware Pipeline Pattern
```eiffel
deferred class SIMPLE_WEB_MIDDLEWARE

feature -- Access
    name: STRING
        deferred end

feature -- Processing
    process (a_request: SIMPLE_WEB_SERVER_REQUEST;
             a_response: SIMPLE_WEB_SERVER_RESPONSE;
             a_next: PROCEDURE)
            -- Process request, call a_next to continue chain
        deferred
        end
end

class SIMPLE_WEB_MIDDLEWARE_PIPELINE

feature -- Access
    middlewares: ARRAYED_LIST [SIMPLE_WEB_MIDDLEWARE]

feature -- Element Change
    use (a_middleware: SIMPLE_WEB_MIDDLEWARE)
        do
            middlewares.extend (a_middleware)
        end

feature -- Execution
    execute (a_request: SIMPLE_WEB_SERVER_REQUEST;
             a_response: SIMPLE_WEB_SERVER_RESPONSE;
             a_handler: PROCEDURE)
        do
            execute_from_index (1, a_request, a_response, a_handler)
        end

feature {NONE} -- Implementation
    execute_from_index (a_index: INTEGER;
                        a_request: SIMPLE_WEB_SERVER_REQUEST;
                        a_response: SIMPLE_WEB_SERVER_RESPONSE;
                        a_handler: PROCEDURE)
        do
            if a_index > middlewares.count then
                a_handler.call (Void)
            else
                middlewares.i_th (a_index).process (
                    a_request,
                    a_response,
                    agent execute_from_index (a_index + 1, a_request, a_response, a_handler)
                )
            end
        end
end
```
**Note**: Uses recursive agents to chain middleware. Each middleware receives `a_next` to continue the chain.
**Verified**: 2025-12-02, EiffelStudio 25.02

---

## Windows Process Patterns

### Environment Variables with cmd /c
```eiffel
build_command (a_project: CI_PROJECT): STRING_32
        -- Build command with environment variables for Windows
    do
        create Result.make (500)
        -- Wrap in cmd /c with SET commands
        Result.append ("cmd /c %"")

        -- Add SET commands chained with &&
        across a_project.environment_variables as ic loop
            Result.append ("set ")
            Result.append (ic.key)
            Result.append ("=")
            Result.append (ic.item)
            Result.append (" && ")
        end

        -- Add the actual command
        Result.append ("%"")
        Result.append (ec_path)
        Result.append ("%" -batch -config ...")

        -- Close cmd /c quote
        Result.append ("%"")
    end
```
**Note**: Windows requires `set VAR=value &&` syntax, not Unix-style `VAR=value command`.
**Verified**: 2025-12-02, EiffelStudio 25.02

### Value Equality Check for Collections
```eiffel
has_value (a_list: ARRAYED_LIST [STRING]; a_value: STRING): BOOLEAN
        -- Does list contain value (using value equality)?
    do
        Result := across a_list as ic some ic ~ a_value end
    end
```
**Note**: ARRAYED_LIST.has uses reference equality. Use `~` operator in across loop for value comparison.
**Verified**: 2025-12-02, EiffelStudio 25.02

---

## JSON Serialization Patterns (simple_json)

### Entity Serialization with SIMPLE_JSON_SERIALIZABLE
```eiffel
class MY_ENTITY
inherit
    SIMPLE_JSON_SERIALIZABLE

feature -- JSON Serialization
    to_json: SIMPLE_JSON_OBJECT
        do
            create Result.make
            Result.put_integer (id, "id")
            Result.put_string (name, "name")
            if attached email as e then
                Result.put_string (e, "email")
            end
        end

feature -- JSON Deserialization
    apply_json (a_json: SIMPLE_JSON_OBJECT)
        do
            if attached a_json.string_item ("name") as n then
                name := n.to_string_8
            end
            if attached a_json.optional_string ("email") as e then
                email := e.to_string_8
            end
        end

feature -- Validation
    json_has_required_fields (a_json: SIMPLE_JSON_OBJECT): BOOLEAN
        do
            Result := a_json.has_all_keys (<<"id", "name">>)
        end
end
```
**Verified**: 2025-12-02, EiffelStudio 25.02

### Friction-Free JSON Field Handling
```eiffel
-- Multiple required field check (instead of has_key && has_key)
if json.has_all_keys (<<"title", "priority">>) then
    -- All required fields present
end

-- Direct INTEGER_32 extraction (instead of .to_integer_32)
l_priority := json.integer_32_item ("priority")

-- Optional string (returns detachable)
if attached json.optional_string ("description") as d then
    l_description := d.to_string_8
end

-- Optional with default value
l_page := json.optional_integer ("page", 1)          -- Default 1
l_active := json.optional_boolean ("active", True)    -- Default True

-- Get list of missing required fields
l_missing := json.missing_keys (<<"name", "email", "age">>)
across l_missing as m loop
    print ("Missing: " + m + "%N")
end
```
**Verified**: 2025-12-02, EiffelStudio 25.02

---

## Multiple Inheritance Patterns

### God Class Extraction via Mixin Classes
When a class has grown too large with multiple distinct feature groups, extract each group into a deferred "mixin" class that inherits from a shared state base:

```
                    SHARED_STATE (deferred)
                           |
    +----------+----------+----------+----------+
    |          |          |          |          |
HANDLER_A   HANDLER_B   HANDLER_C   RENDERER  ... (all deferred)
    |          |          |          |          |
    +----------+----------+----------+----------+
                           |
                     MAIN_CLASS (effective)
```

**Key Insight**: This pattern replaces pub-sub, dependency injection, or mediator patterns needed in single-inheritance languages. All handlers share state via inherited deferred features - they're literally the same `Current` at runtime.

**Verified**: 2025-12-03, EiffelStudio 25.02

### Shared State Base Class Pattern
```eiffel
deferred class
    MY_SHARED_STATE

feature -- Shared State (deferred - implemented by final class)

    data_store: HASH_TABLE [MY_ITEM, STRING_32]
            -- Shared data storage.
        deferred
        end

    server: MY_SERVER
            -- Server instance.
        deferred
        end

feature -- Shared Helpers (effective - used by all handlers)

    item_by_id (a_id: detachable STRING_32): detachable MY_ITEM
            -- Find item by ID with null-safety.
        do
            if attached a_id as l_id then
                Result := data_store.item (l_id)
            end
        end

    send_not_found (a_response: MY_RESPONSE; a_type: STRING; a_id: detachable STRING_32)
            -- Standard 404 response.
        do
            a_response.set_status (404)
            a_response.send_json ("{%"error%":%"" + a_type + " not found%"}")
        end

    s8 (a_str: STRING_32): STRING_8
            -- Quick STRING_32 to STRING_8 conversion.
        do
            Result := a_str.to_string_8
        end

end
```
**Note**: Deferred features define what state the mixins need. Effective features provide shared utilities.
**Verified**: 2025-12-03, EiffelStudio 25.02

### Handler Mixin Pattern
```eiffel
deferred class
    MY_ITEM_HANDLERS

inherit
    MY_SHARED_STATE

feature -- Route Setup

    setup_item_routes
            -- Register item routes with server.
        do
            server.on_get ("/api/items", agent handle_list_items)
            server.on_get ("/api/items/{id}", agent handle_get_item)
            server.on_post ("/api/items", agent handle_create_item)
        end

feature -- Handlers

    handle_list_items (a_request: MY_REQUEST; a_response: MY_RESPONSE)
        do
            -- Uses inherited `data_store` directly
            a_response.send_json (items_to_json (data_store))
        end

    handle_get_item (a_request: MY_REQUEST; a_response: MY_RESPONSE)
        do
            -- Uses inherited `item_by_id` helper
            if attached item_by_id (a_request.path_parameter ("id")) as l_item then
                a_response.send_json (l_item.to_json)
            else
                -- Uses inherited `send_not_found` helper
                send_not_found (a_response, "Item", a_request.path_parameter ("id"))
            end
        end

end
```
**Note**: Handler classes are deferred because they inherit deferred features from SHARED_STATE.
**Verified**: 2025-12-03, EiffelStudio 25.02

### Final Composition Class
```eiffel
class
    MY_MAIN_SERVER

inherit
    MY_ITEM_HANDLERS
    MY_USER_HANDLERS
    MY_AUTH_HANDLERS
    MY_RENDERER

create
    make

feature {NONE} -- Initialization

    make (a_port: INTEGER)
        do
            create server_impl.make (a_port)
            create data_store_impl.make (100)
            setup_item_routes
            setup_user_routes
            setup_auth_routes
        end

feature -- Access (Effectuate deferred features)

    server: MY_SERVER
        do
            Result := server_impl
        end

    data_store: HASH_TABLE [MY_ITEM, STRING_32]
        do
            Result := data_store_impl
        end

feature {NONE} -- Implementation

    server_impl: MY_SERVER
    data_store_impl: HASH_TABLE [MY_ITEM, STRING_32]

end
```
**Note**: No `undefine` clauses needed - Eiffel automatically joins deferred features.
**Verified**: 2025-12-03, EiffelStudio 25.02

### Diamond Problem Resolution
When multiple inheritance paths share a common ancestor:

```eiffel
class CHILD
inherit
    PARENT_A
        rename
            feature_x as feature_x_from_a    -- Keep both with different names
        undefine
            feature_y                         -- Convert effective to deferred
        redefine
            feature_z                         -- Replace with new implementation
        select
            feature_w                         -- Resolve polymorphic ambiguity
        end
    PARENT_B
        -- ... similar adaptation clauses
        end
```

**When to use each:**
- `rename`: Need both versions with different names
- `undefine`: Parent provides default but you want other parent's version
- `redefine`: Replace with completely new implementation
- `select`: Multiple effective versions exist, pick one for dynamic dispatch

**Feature Joining Rules:**
- Deferred + Deferred = Single deferred (one implementation needed)
- Deferred + Effective = Effective satisfies deferred (automatic)
- Effective + Effective = Conflict (must use rename/undefine/select)

**Common Error - VDUS(3):** "Undefine subclause lists deferred feature"
- You cannot `undefine` an already-deferred feature
- `undefine` converts effective → deferred, not the reverse
- Fix: Remove the `undefine` clause; feature joining handles it

**Verified**: 2025-12-03, EiffelStudio 25.02

---

## Fluent Builder Patterns (simple_htmx)

### Content Accumulation vs Overwrite
When building fluent interfaces that add content (like HTML builders), decide whether methods should **accumulate** or **replace**:

```eiffel
-- WRONG: Each call overwrites previous content (bug!)
raw_html (a_html: READABLE_STRING_GENERAL): like Current
    do
        content_text := a_html.to_string_32  -- Overwrites!
        Result := Current
    end

-- Usage - only "Third" appears
l_div.raw_html ("<span>First</span>").do_nothing
l_div.raw_html ("<span>Second</span>").do_nothing
l_div.raw_html ("<span>Third</span>").do_nothing

-- CORRECT: Accumulate content
raw_html (a_html: READABLE_STRING_GENERAL): like Current
    do
        content_text.append (a_html.to_string_32)  -- Appends!
        Result := Current
    end

-- Now all three spans appear
```

**Naming Convention:**
- `set_content`: Replaces/overwrites
- `add_content`, `append_content`, `raw_html`: Accumulates

**Verified**: 2025-12-03, EiffelStudio 25.02

### When MI Replaces Other Patterns
In single-inheritance languages, cross-component communication requires:
- **Pub-sub/Events**: Components publish events, others subscribe
- **Dependency Injection**: Pass dependencies through constructors
- **Service Locator**: Global registry to find services
- **Mediator**: Central coordinator knows all components

Eiffel MI eliminates this ceremony: if `HTMX_HANDLERS` needs `RENDERER.render_canvas`, they simply share the same `Current`. No wiring, no indirection, no ceremony.

**Verified**: 2025-12-03, EiffelStudio 25.02
