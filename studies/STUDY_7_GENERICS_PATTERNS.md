# Study 7: Generics and Constraint Patterns in Canonical Eiffel

**Date:** 2025-12-26
**Source:** ISE EiffelStudio 25.02 Standard Library
**Focus:** Parametric polymorphism and type constraints

---

## Executive Summary

Eiffel generics provide compile-time type safety with constraints. Key patterns: single/multiple type parameters, COMPARABLE/HASHABLE constraints, anchored types (`like item`, `like Current`).

---

## 1. Basic Generic Syntax

### Single Type Parameter

```eiffel
class ARRAYED_LIST [G]
```

### Multiple Type Parameters

```eiffel
class HASH_TABLE [G, K -> detachable HASHABLE]
-- G: value type, K: key type (constrained)
```

### Constrained Generics

```eiffel
class SORTED_LIST [G -> COMPARABLE]
-- G must implement < > = operations
```

---

## 2. Constraint Patterns

### COMPARABLE Constraint

```eiffel
class HEAP_PRIORITY_QUEUE [G -> COMPARABLE]
class SORTED_LIST [G -> COMPARABLE]
class BINARY_SEARCH_TREE_SET [G -> COMPARABLE]
```

**COMPARABLE Interface:**
```eiffel
deferred class COMPARABLE
feature
    is_less alias "<" (other: like Current): BOOLEAN
        deferred
        ensure
            asymmetric: Result implies not (other < Current)
        end

    is_equal (other: like Current): BOOLEAN
        do
            Result := not (Current < other) and not (other < Current)
        ensure
            trichotomy: Result = (not (Current < other) and not (other < Current))
        end
end
```

**File:** `library/base/elks/kernel/comparable.e`

### HASHABLE Constraint

```eiffel
class HASH_TABLE [G, K -> detachable HASHABLE]
```

**HASHABLE Interface:**
```eiffel
deferred class HASHABLE
feature
    hash_code: INTEGER
        deferred
        ensure
            good_hash_value: Result >= 0
        end
end
```

**File:** `library/base/elks/kernel/hashable.e`

### Detachable Constraint

```eiffel
class HASH_TABLE [G, K -> detachable HASHABLE]
-- K can be Void (detachable)
```

---

## 3. Anchored Types

### `like Current`

```eiffel
duplicate (n: INTEGER): like Current
    -- Returns same type as container

twin: like Current
    -- Clone returns exact type
```

### `like item`

```eiffel
i_th alias "[]" (i: INTEGER): like item
first: like item
last: like item
has (v: like item): BOOLEAN
```

**File:** `library/base/elks/structures/list/arrayed_list.e`

---

## 4. Container Hierarchy

```
CONTAINER [G]
    |
    +-- COLLECTION [G]
    |       |
    |       +-- BAG [G]
    |               |
    |               +-- SET [G]
    |               +-- LINEAR [G]
    |                       |
    |                       +-- CHAIN [G]
    |                               |
    |                               +-- ARRAYED_LIST [G]
    |                               +-- LINKED_LIST [G]
    |
    +-- TABLE [G, H]
            |
            +-- HASH_TABLE [G, K -> detachable HASHABLE]
```

---

## 5. Iterator Pattern with Generics

### Cursor Types

```eiffel
class ARRAYED_LIST_ITERATION_CURSOR [G]
inherit
    ITERATION_CURSOR [G]

feature
    target: ARRAYED_LIST [G]
end

class HASH_TABLE_ITERATION_CURSOR [G, K -> detachable HASHABLE]
inherit
    TABLE_ITERATION_CURSOR [G, K]

feature
    item: G
    key: K
end
```

---

## 6. Higher-Order Functions with Generics

```eiffel
do_all (action: PROCEDURE [G])
do_if (action: PROCEDURE [G]; test: FUNCTION [G, BOOLEAN])
there_exists (test: FUNCTION [G, BOOLEAN]): BOOLEAN
for_all (test: FUNCTION [G, BOOLEAN]): BOOLEAN
```

**File:** `library/base/elks/structures/list/arrayed_list.e` (lines 224-285)

---

## 7. Creation Patterns

```eiffel
create
    make,
    make_filled,
    make_from_array,
    make_from_iterable

convert
    make_from_iterable ({ARRAY [G]})

make_filled (n: INTEGER)
    require
        valid_number: n >= 0
        has_default: ({G}).has_default  -- Type query
    do
        make_filled_area (({G}).default, n)
    end
```

---

## 8. Constraint Summary

| Constraint | Purpose | Example |
|-----------|---------|---------|
| `[G]` | Any type | `ARRAYED_LIST [G]` |
| `[G -> COMPARABLE]` | Sorting | `SORTED_LIST [G]` |
| `[G -> HASHABLE]` | Hashing | (Keys only) |
| `[K -> detachable HASHABLE]` | Void keys | `HASH_TABLE [G, K]` |
| `[H -> INTEGER]` | Indexing | `INDEXABLE [G, H]` |

---

## 9. FAQ Pairs to Generate

1. How do I create a generic class in Eiffel?
2. How do I constrain a generic type parameter?
3. What is the difference between `[G]` and `[G -> COMPARABLE]`?
4. What does `like Current` mean?
5. What does `like item` mean?
6. How do I use multiple type parameters?
7. What is `detachable` in a generic constraint?
8. How do I get the default value of a generic type?

---

## References

- `library/base/elks/kernel/comparable.e` - COMPARABLE interface
- `library/base/elks/kernel/hashable.e` - HASHABLE interface
- `library/base/elks/structures/list/arrayed_list.e` - Generic list
- `library/base/elks/structures/table/hash_table.e` - Multiple constraints
- `library/base/elks/structures/access/container.e` - Container hierarchy
