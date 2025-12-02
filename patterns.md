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
