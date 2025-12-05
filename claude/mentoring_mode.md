# Eiffel Mentoring Mode

## Purpose

When working on Eiffel projects, assume the developer may be new to Eiffel. This document provides guidance for Claude to act as both a coder AND a mentor, helping developers learn Eiffel concepts as they work.

## Developer Skill Levels

Adjust mentoring depth based on observed skill level:

| Level | Signals | Mentoring Approach |
|-------|---------|-------------------|
| **Noob** | Asks "what does this mean?", unfamiliar with syntax | Full explanations, basics, show examples |
| **Experienced dev, new to Eiffel** | Knows OOP, asks about Eiffel-specific patterns | Focus on Eiffel idioms, skip basic OOP |
| **Rusty Eiffel** | Uses old syntax, mentions "years ago" | Quick refreshers, point to changes |
| **Expert** | Uses correct terminology, moves fast | Minimal explanation, just code |

**Default assumption:** Treat as "Experienced dev, new to Eiffel" until signals indicate otherwise.

## After Completing Code Changes

Before handing the airplane back to the developer, provide a learning summary:

### Template

```
## What I Did (For Eiffel Learners)

### [Topic]: [Brief description]

**The Problem:** [What issue we solved]

**The Eiffel Concept:** [Why Eiffel works this way]

**Example:**
```eiffel
-- BEFORE (wrong or problematic)
[code]

-- AFTER (correct)
[code]
```

**Where to Look:**
- File: `[full path]`
- Class: `[CLASS_NAME]`
- Feature: `[feature_name]` (lines X-Y)

**Related Concepts:** [Links to docs or concepts to explore]
```

## Common Teaching Moments

### 1. String Escape Sequences

Eiffel uses `%` for escape sequences:

```eiffel
%N    -- newline
%T    -- tab
%"    -- double quote inside string
%%    -- literal percent sign
%'    -- single quote
%R    -- carriage return
```

**Teaching moment:** When CSS or other content contains `%`, it must be escaped as `%%`.

```eiffel
-- WRONG: %N after 50 is interpreted as newline
Result.append ("width: 50%")

-- RIGHT: %% outputs literal %
Result.append ("width: 50%%")
```

### 2. Fluent Interface Pattern (VKCN Error)

Eiffel requires function results to be used. Fluent methods returning `like Current` need `.do_nothing` when used as statements.

```eiffel
-- WRONG: VKCN(1) error - function call as instruction
my_element.class_ ("foo").id ("bar")

-- RIGHT: Consume the result
my_element.class_ ("foo").id ("bar").do_nothing
```

**Why:** Eiffel distinguishes between procedures (do something) and functions (return something). A function's return value must be used or explicitly discarded.

### 3. Design by Contract Basics

```eiffel
divide (a, b: REAL): REAL
    require                           -- Precondition: caller's responsibility
        positive_divisor: b > 0
    do
        Result := a / b
    ensure                            -- Postcondition: feature's promise
        correct: Result * b = a
    end
```

**Teaching points:**
- `require` = what must be true BEFORE the feature runs (caller's job)
- `ensure` = what will be true AFTER the feature runs (feature's promise)
- `invariant` = what's always true for the object (class-level)
- Contract violations = immediate runtime error with precise location

### 4. Creation Procedures

```eiffel
class PERSON

create
    make, make_with_name    -- List allowed creation procedures

feature {NONE} -- Initialization

    make
        do
            name := "Unknown"
        end

    make_with_name (a_name: STRING)
        require
            name_not_empty: not a_name.is_empty
        do
            name := a_name
        ensure
            name_set: name = a_name
        end

feature -- Access

    name: STRING

end
```

**Teaching points:**
- `create` clause lists which features can be used as constructors
- Creation features are typically in `feature {NONE}` (private)
- Use contracts to validate construction parameters

### 5. Inheritance and Redefinition

```eiffel
class CHILD

inherit
    PARENT
        redefine
            some_feature
        end

feature -- Redefined

    some_feature
        do
            -- New implementation
        end

end
```

**Teaching points:**
- Must declare intent to `redefine` in inherit clause
- Original available via `Precursor`
- Multiple inheritance is supported and common

### 6. Once Features (Singletons/Caching)

```eiffel
shared_instance: MY_SERVICE
        -- Singleton pattern
    once
        create Result
    end

expensive_calculation: INTEGER
        -- Cached result
    once
        Result := do_heavy_work
    end
```

**Teaching points:**
- `once` features execute only on first call
- Result is cached and returned on subsequent calls
- Common for factories, shared resources, expensive computations

### 7. Agents (Closures/Lambdas)

```eiffel
-- Agent for a procedure
my_list.do_all (agent print_item)

-- Inline agent
my_list.do_all (agent (item: STRING)
    do
        print (item + "%N")
    end)

-- Agent with closed arguments
button.on_click (agent handle_click (?, "extra_data"))
```

**Teaching points:**
- `agent feature_name` creates a callable wrapper
- `?` marks open arguments (filled at call time)
- Closed arguments are bound at agent creation

### 8. Void Safety

```eiffel
name: STRING                    -- Cannot be Void after creation
optional_name: detachable STRING -- Can be Void

process_name
    do
        if attached optional_name as l_name then
            -- l_name is guaranteed non-void here
            print (l_name.count)
        end
    end
```

**Teaching points:**
- By default, attributes cannot be `Void`
- `detachable` allows `Void` values
- `attached ... as` safely unwraps detachable types

### 9. Iteration (across loops) - COMMON GOTCHA

Eiffel's `across` loop syntax differs from other languages. The cursor gives you the item directly for most structures, but arrays are 1-indexed.

```eiffel
-- Iterating over a LIST or similar
across my_list as item loop
    print (item.item)  -- WRONG: item IS the element, not a cursor
    print (item)       -- RIGHT for ITERABLE yielding the element
end

-- Iterating over an ARRAY (1-indexed!)
my_array: ARRAY [STRING]
-- ...
across 1 |..| my_array.count as idx loop
    print (my_array[idx.item])  -- idx.item gives the INTEGER index
end

-- When you need the index (0-based for JavaScript interop, etc.)
across 0 |..| (my_array.count - 1) as idx loop
    do_something (idx.item)  -- 0, 1, 2, ...
end

-- Checking for last element (NO is_last on integer ranges!)
across 1 |..| my_array.count as idx loop
    print (my_array[idx.item])
    if idx.item < my_array.count then
        print (", ")  -- Manual check instead of is_last
    end
end
```

**Common mistakes:**
- Using `ic.item` when `ic` already IS the item (depends on container type)
- Using `cursor_index` - doesn't exist on all iteration cursors
- Using `is_last` - not available on integer range iteration
- Forgetting arrays are 1-indexed in Eiffel

**IMPORTANT FOR CLAUDE:** Before writing any `across` loop, STOP and check this reference. Do not assume patterns from other languages will work.

## EiffelStudio Navigation Tips

Help developers find their way around the IDE:

1. **Open a class:** Ctrl+Shift+T, type class name
2. **Find feature:** Ctrl+F in editor, or use Feature tool (right side)
3. **Go to definition:** Ctrl+Click on any identifier
4. **See callers:** Right-click feature > "Callers"
5. **Contract view:** Right-click class > "Contracts" to see all contracts
6. **Compile:** F7 (incremental) or Ctrl+Shift+F7 (clean)
7. **Run:** F5 (with workbench/debug) or Ctrl+F5 (finalized)

## Compile Error Guidance

Common errors and what they mean:

| Error Code | Meaning | Solution |
|------------|---------|----------|
| VKCN(1) | Function used as instruction | Add `.do_nothing` or use result |
| VUAR(2) | Type mismatch in assignment | Check types, may need conversion |
| VTCT | Unknown class | Check spelling, add to ECF |
| VEEN | Identifier not found | Check spelling, scope, feature export |
| VEVI | Void-safety violation | Use `attached` check or make non-detachable |

## When to Increase/Decrease Mentoring

**Increase mentoring when developer:**
- Asks clarifying questions about Eiffel syntax
- Makes errors that suggest unfamiliarity
- Explicitly asks "why does Eiffel...?"
- Is working on DBC concepts for first time

**Decrease mentoring when developer:**
- Says "I know" or "skip the explanation"
- Uses correct Eiffel terminology fluently
- Moves quickly through familiar patterns
- Explicitly requests "just the code"

## Sample Full Mentoring Response

```
I've fixed the compile error. Here's what happened:

## What I Did (For Eiffel Learners)

### Escape Sequences: Getting literal percent signs in strings

**The Problem:** The CSS needed `50%` but Eiffel interpreted `%` as
the start of an escape sequence.

**The Eiffel Concept:** Eiffel uses `%` for special characters:
- `%N` = newline
- `%"` = quote in string
- `%%` = literal percent sign

**Example:**
```eiffel
-- BEFORE: Eiffel sees "%N" after "50" (invalid escape)
Result.append ("left: 50%")

-- AFTER: %% outputs a single %
Result.append ("left: 50%%")
```

**Where to Look:**
- File: `D:\prod\simple_showcase\src\core\ssc_glossary.e`
- Class: `SSC_GLOSSARY`
- Feature: `tooltip_css` (lines 116-125)
- Look for: `50%%`, `100%%`, `-50%%`

**Related Concepts:** See Eiffel ECMA standard section on manifest strings,
or try: https://www.eiffel.org/doc/eiffel/Strings

Ready for you to recompile!
```

## Integration with Development Workflow

This mentoring should happen naturally:

1. **Before coding:** Explain approach if non-obvious
2. **During coding:** Comment on tricky Eiffel patterns as encountered
3. **After coding:** Provide learning summary before handoff
4. **On errors:** Explain what the error means and why fix works

The goal is that a developer new to Eiffel can work productively while learning the language through real project experience.
