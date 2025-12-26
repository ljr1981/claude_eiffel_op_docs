# Study 3: Iteration Patterns in Canonical Eiffel

**Date:** 2025-12-25
**Sources:**
- ISE EiffelStudio 25.02 Standard Library
- Gobo Library (EiffelStudio contrib)

---

## Executive Summary

Eiffel iteration has evolved from traditional `from/until/loop` to the modern `across` construct. The cursor pattern provides a clean separation between containers and traversal, enabling both imperative and functional-style iteration.

---

## Evolution of Iteration

### Traditional Style (Pre-2008)

```eiffel
from
    list.start
until
    list.after
loop
    process (list.item)
    list.forth
end
```

**Problems:**
- Cursor state managed internally by container
- Can't have multiple simultaneous iterations
- Mutable iterator state

### Modern Style (across loops)

```eiffel
across list as cursor loop
    process (cursor.item)
end
```

**Benefits:**
- External iterator (cursor object)
- Multiple simultaneous iterations
- Immutable feel (cursor manages its own state)

---

## Cursor Hierarchy (ISE)

```
ITERATION_CURSOR [G]
    |
    +-- TABLE_ITERATION_CURSOR [G, K]
    |       |
    |       +-- HASH_TABLE_ITERATION_CURSOR
    |       +-- ARRAY_ITERATION_CURSOR
    |
    +-- INDEXABLE_ITERATION_CURSOR [G]
            |
            +-- ARRAYED_LIST_ITERATION_CURSOR
            +-- LINKED_LIST_ITERATION_CURSOR
```

### Key Features

```eiffel
deferred class ITERATION_CURSOR [G]

feature -- Access
    item: G
        -- Current element
        deferred
        end

feature -- Status report
    after: BOOLEAN
        -- Is cursor past last element?
        deferred
        end

feature -- Cursor movement
    forth
        -- Move to next position
        require
            not_after: not after
        deferred
        ensure
            -- Advanced by one
        end
```

**Canonical Example:** `base/elks/kernel/iteration_cursor.e`

---

## Cursor Hierarchy (Gobo)

Gobo uses a different architecture:

```
DS_LINEAR_CURSOR [G]
    |
    +-- DS_ARRAYED_LIST_CURSOR
    +-- DS_LINKED_LIST_CURSOR
    +-- DS_BILINKED_LIST_CURSOR
    +-- DS_HASH_TABLE_CURSOR
```

### Key Difference

```eiffel
class DS_LINEAR_CURSOR [G]

feature -- Status
    is_first: BOOLEAN
    is_last: BOOLEAN
    off: BOOLEAN  -- Beyond bounds (before or after)

feature -- Traversal
    start
    finish
    forth
    back      -- Gobo supports bidirectional!
```

**Gobo Path:** `contrib/library/gobo/library/structure/src/container/`

---

## Across Loop Variants

### Basic Iteration

```eiffel
across list as c loop
    io.put_string (c.item.out)
end
```

### With Index (INDEXABLE containers)

```eiffel
across array as c loop
    io.put_string (c.cursor_index.out + ": " + c.item.out)
end
```

### Universal Quantifier (all)

```eiffel
if across list as c all c.item > 0 end then
    io.put_string ("All positive")
end
```

**Semantic Meaning:** ∀x ∈ list: x > 0

### Existential Quantifier (some)

```eiffel
if across list as c some c.item = target end then
    io.put_string ("Found target")
end
```

**Semantic Meaning:** ∃x ∈ list: x = target

---

## Table Iteration (Key-Value)

```eiffel
across hash_table as c loop
    io.put_string ("Key: " + c.key.out)
    io.put_string ("Value: " + c.item.out)
end
```

**Cursor Type:** `TABLE_ITERATION_CURSOR [G, K]` exposes both `key` and `item`.

---

## Pattern: Iterator with Modification

```eiffel
-- WRONG: Modifying during iteration
across list as c loop
    if c.item < 0 then
        list.remove  -- BAD! Invalidates cursor
    end
end

-- CORRECT: Collect then modify
create to_remove.make (10)
across list as c loop
    if c.item < 0 then
        to_remove.extend (c.item)
    end
end
across to_remove as c loop
    list.prune (c.item)
end
```

---

## Pattern: Nested Iteration

```eiffel
across outer_list as outer loop
    across inner_list as inner loop
        if outer.item ~ inner.item then
            -- Found match
        end
    end
end
```

**Why This Works:** Each `across` creates its own cursor object. No shared state.

---

## Pattern: Early Exit

```eiffel
-- Using 'some' for early exit
found := across list as c some
    c.item = target
end

-- Traditional with explicit exit
across list as c until found loop
    if c.item = target then
        found := True
    end
end
```

---

## Pattern: Accumulation

```eiffel
-- Sum all elements
total := 0
across list as c loop
    total := total + c.item
end

-- In one expression (functional style)
total := (across list as c from 0 all agent (acc, x: INTEGER): INTEGER do Result := acc + x end end)
-- Note: This syntax is conceptual; Eiffel uses explicit loops
```

---

## ITERABLE Interface

```eiffel
deferred class ITERABLE [G]

feature -- Access
    new_cursor: ITERATION_CURSOR [G]
        -- Fresh cursor for iteration
        deferred
        ensure
            result_attached: Result /= Void
        end
```

**Any class inheriting ITERABLE can be used with across.**

### Making Your Class Iterable

```eiffel
class MY_CONTAINER [G]
inherit
    ITERABLE [G]

feature
    new_cursor: MY_CURSOR [G]
        do
            create Result.make (Current)
        end
```

---

## FAQ Pairs to Generate

1. Q: How do I iterate over a list in Eiffel?
2. Q: What's the difference between `across` and `from/until/loop`?
3. Q: How do I check if all elements satisfy a condition?
4. Q: How do I check if any element satisfies a condition?
5. Q: Can I iterate over a hash table's keys and values?
6. Q: How do I make my own class work with `across`?
7. Q: Why can't I modify a collection during across iteration?
8. Q: How do cursors work in Eiffel?

---

## Comparison: ISE vs Gobo

| Aspect | ISE | Gobo |
|--------|-----|------|
| Base class | ITERATION_CURSOR | DS_LINEAR_CURSOR |
| Bidirectional | No (separate type) | Yes (back feature) |
| Position tracking | after only | is_first, is_last, off |
| Integration | Built into language | Library pattern |

---

## References to Canonical Code

| Pattern | File | Path |
|---------|------|------|
| ITERATION_CURSOR | `iteration_cursor.e` | `base/elks/kernel/` |
| TABLE_ITERATION_CURSOR | `table_iteration_cursor.e` | `base/elks/support/` |
| ITERABLE | `iterable.e` | `base/elks/kernel/` |
| DS_LINEAR_CURSOR | `ds_linear_cursor.e` | `gobo/library/structure/src/container/` |
| across examples | `arrayed_list.e` | `base/elks/structures/list/` |
