# Eiffel Across Loops

## Basic Syntax

```eiffel
across collection as cursor_name loop
    -- body
end
```

## The .item Question - RESOLVED

**The behavior depends on the cursor type returned by the collection.**

### ARRAYED_LIST, LINKED_LIST (and most linear collections)

The cursor **IS** the item. Use the cursor directly:

```eiffel
my_list: ARRAYED_LIST [PERSON]

across my_list as ic loop
    print (ic.name)           -- ic IS the PERSON
    ic.set_age (30)           -- Direct access to PERSON features
end
```

**Why?** These collections return a cursor where the cursor itself conforms to the item type through `new_cursor` returning an `ITERATION_CURSOR [G]` where iteration yields `G` directly.

### HASH_TABLE

The cursor provides `.item` for value and `.key` for key:

```eiffel
my_table: HASH_TABLE [PERSON, STRING]

across my_table as ic loop
    print (ic.key)            -- The STRING key
    print (ic.item.name)      -- ic.item IS the PERSON value
end
```

### INTEGER Intervals

Use `.item` to get the integer value:

```eiffel
across 1 |..| 10 as i loop
    print (i.item)            -- The INTEGER value
end
```

### General Rule

| Collection Type | Cursor Access |
|----------------|---------------|
| `ARRAYED_LIST [G]` | `ic` is `G` directly |
| `LINKED_LIST [G]` | `ic` is `G` directly |
| `HASH_TABLE [G, K]` | `ic.item` for value, `ic.key` for key |
| `INTEGER_INTERVAL` | `ic.item` for the integer |
| `ARRAY [G]` | `ic` is `G` directly |

**Verified**: 2025-12-02, EiffelStudio 25.02

---

## Cursor Features

Common cursor features available:

```eiffel
across my_list as ic loop
    ic.cursor_index    -- Current position (1-based)
    ic.is_first        -- True if at first item
    ic.is_last         -- True if at last item (if available)
end
```

---

## Boolean Variants

### All (universal quantifier)
```eiffel
-- True if ALL elements satisfy condition
across my_list as ic all ic.age >= 18 end
```

### Some (existential quantifier)
```eiffel
-- True if AT LEAST ONE element satisfies condition
across my_list as ic some ic.age >= 65 end
```

---

## Modifying During Iteration

**DO NOT modify the collection during across iteration.**

If you need to modify:
```eiffel
-- WRONG: Modifying during across
across my_list as ic loop
    if ic.should_remove then
        my_list.prune (ic)  -- UNDEFINED BEHAVIOR
    end
end

-- CORRECT: Collect items first, then modify
local
    to_remove: ARRAYED_LIST [ITEM]
do
    create to_remove.make (10)
    across my_list as ic loop
        if ic.should_remove then
            to_remove.extend (ic)
        end
    end
    across to_remove as ic loop
        my_list.prune (ic)
    end
end
```

---

## Traditional Loop Alternative

When you need more control (or must modify):

```eiffel
from
    my_list.start
until
    my_list.after
loop
    print (my_list.item)
    my_list.forth
end
```

With contracts for termination proof:

```eiffel
from
    i := my_list.lower
until
    i > my_list.upper
loop
    process (my_list [i])
    i := i + 1
variant
    my_list.upper - i + 1
invariant
    i >= my_list.lower
end
```

---

## Requirements

- Collection must inherit from `ITERABLE [G]`
- All EiffelBase collections support `across`
- Custom collections need to implement `new_cursor: ITERATION_CURSOR [G]`

---

## Quick Reference

```eiffel
-- List iteration (ic IS the item)
across person_list as ic loop
    print (ic.name)
end

-- Hash table iteration
across person_table as ic loop
    print (ic.key + ": " + ic.item.name)
end

-- Integer range
across 1 |..| count as i loop
    print (i.item)
end

-- Boolean check (all positive)
if across numbers as n all n > 0 end then
    print ("All positive")
end

-- Boolean check (any negative)
if across numbers as n some n < 0 end then
    print ("Has negative")
end
```

**Verified**: 2025-12-02, EiffelStudio 25.02
