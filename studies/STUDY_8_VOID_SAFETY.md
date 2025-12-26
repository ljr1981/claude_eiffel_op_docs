# Study 8: Void Safety Patterns in Canonical Eiffel

**Date:** 2025-12-26
**Source:** ISE EiffelStudio 25.02 Standard Library
**Focus:** Certified Attachment Patterns (CAP)

---

## Executive Summary

Void safety eliminates null pointer errors at compile time. Key mechanisms: `attached`/`detachable` type modifiers, object tests (`attached x as lx`), stable attributes, and result guarantees.

---

## 1. Attachment Status

### Type Modifiers

```eiffel
feature
    always_valid: attached SOME_TYPE  -- Never void
    maybe_void: detachable SOME_TYPE  -- Can be void
```

### Default Status

- Reference types default to `detachable`
- Expanded types always attached (by definition)

---

## 2. Object Test Pattern (CAP)

### Basic Syntax

```eiffel
if attached o then
    s := o.out  -- o known attached here
end
```

### With Local Variable

```eiffel
if attached {MISMATCH_CORRECTOR} Current as l_corrector then
    l_corrector.correct_mismatch
end
```

**File:** `library/base/elks/kernel/any.e` (line 272)

### Type-Based Object Tests

```eiffel
if attached {READABLE_STRING_32} s as s32 then
    io.put_string_32 (s32)
elseif attached {READABLE_STRING_8} s as s8 then
    io.put_string (s8)
else
    io.put_string_32 (s.as_string_32)
end
```

**File:** `library/base/elks/kernel/any.e` (print feature)

### Chained Object Tests

```eiffel
if attached exception_manager.last_exception as e and then
   attached e.original.description as d then
    Result := {UTF_CONVERTER}.escaped_utf_32_string_to_utf_8_string_8 (d)
end
```

**File:** `library/base/elks/kernel/exceptions.e` (line 68)

### Check Block Pattern

```eiffel
item: G
    do
        check attached active as a then
            Result := a.item
        end
    end
```

**File:** `library/base/elks/structures/list/linked_list.e` (lines 53-56)

Use `check attached` when you can prove attachment but compiler can't.

---

## 3. Stable Attributes

```eiffel
internal_detachable_name_pointer: detachable MANAGED_POINTER
    note option: stable attribute end
```

Once attached through object test, stable attributes remain attached for subsequent accesses.

**File:** `library/base/elks/kernel/directory.e` (line 669)

---

## 4. Result Attachment

### Guaranteed Non-Void

```eiffel
frozen standard_twin: like Current
    external "built_in"
    ensure
        standard_twin_not_void: Result /= Void
        equal: standard_equal (Result, Current)
    end
```

### Conditional (Detachable Result)

```eiffel
clone (other: detachable ANY): like other
    do
        if other /= Void then
            Result := other.twin
        end
    ensure
        equal: Result ~ other
    end
```

**File:** `library/base/elks/kernel/any.e` (lines 187-197)

---

## 5. Common Idioms

### Safe Attribute Access

```eiffel
if attached internal_file_name as l_name then
    close_file (l_name)
end
```

### Optional Values

```eiffel
separate_character_option_value (o: CHARACTER): detachable STRING
    -- Void if not found
```

### Default Value Access

```eiffel
({G}).default     -- Default value for generic type G
({G}).has_default -- Query if type has default
```

**File:** `library/base/elks/kernel/array.e`

### Lazy Initialization

```eiffel
last_result: detachable RESULT_TYPE

clear_last_result
    local
        l_result: detachable RESULT_TYPE
    do
        last_result := l_result  -- Assigns void
    end
```

### Void Comparison

```eiffel
frozen equal (a: detachable ANY; b: like a): BOOLEAN
    do
        if a = Void then
            Result := b = Void
        else
            Result := b /= Void and then a.is_equal (b)
        end
    end
```

**File:** `library/base/elks/kernel/any.e` (lines 87-101)

---

## 6. Error Patterns (What to Avoid)

### VEVI - Call on Possibly Void

```eiffel
-- WRONG
x: detachable SOME_TYPE
x.some_feature  -- Compiler error VEVI

-- FIX
if attached x then
    x.some_feature
end
```

### Detachable to Attached Assignment

```eiffel
-- WRONG
y: attached SOME_TYPE
x: detachable SOME_TYPE
y := x  -- Compiler error

-- FIX
if attached x as x_attached then
    y := x_attached
end
```

---

## 7. VOID_TARGET Exception

```eiffel
class VOID_TARGET
inherit LANGUAGE_EXCEPTION
feature
    tag: IMMUTABLE_STRING_32
        once
            create Result.make_from_string_8 ("Feature call on void target.")
        end
end
```

**File:** `library/base/elks/kernel/exceptions/void_target.e`

Runtime exception if void safety is bypassed (shouldn't happen with proper compilation).

---

## 8. Key Insights

1. **Void is Explicit** - Requires `detachable` declaration
2. **Attachment Certified** - Every access preceded by verification
3. **Lazy Evaluation Essential** - `and then` prevents void errors
4. **Local Variables Preferred** - Object tests create safe locals
5. **Detachable = Design Choice** - Void means "absent value"

---

## 9. FAQ Pairs to Generate

1. What is void safety in Eiffel?
2. What is the difference between attached and detachable?
3. How do I safely access a possibly-void reference?
4. What is the object test pattern?
5. How do I use `attached x as lx`?
6. What is a stable attribute?
7. How do I fix VEVI compiler errors?
8. What does `and then` do for void safety?

---

## References

- `library/base/elks/kernel/any.e` - Core void-safe patterns
- `library/base/elks/kernel/exceptions.e` - Chained object tests
- `library/base/elks/structures/list/linked_list.e` - Check attached
- `library/base/elks/structures/table/hash_table.e` - Detachable results
- `library/base/elks/kernel/exceptions/void_target.e` - Exception class
