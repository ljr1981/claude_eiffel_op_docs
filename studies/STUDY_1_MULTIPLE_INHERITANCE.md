# Study 1: Multiple Inheritance Patterns in Canonical Eiffel

**Date:** 2025-12-25
**Source:** ISE EiffelStudio 25.02 Standard Library
**Path:** `/c/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/library/base/`
**Files Analyzed:** 484

---

## Executive Summary

Multiple inheritance in Eiffel is a **compositional tool**, not a liability. The canonical ISE library demonstrates sophisticated MI patterns that would require complex workarounds in single-inheritance languages.

---

## Pattern 1: Mixin + Implementation Composition

**Prevalence:** Very common (50+ classes)

The dominant pattern combines an abstract interface with a concrete implementation:

```eiffel
class ARRAYED\_STACK \[G]
inherit
    ARRAYED\_LIST \[G]      -- Implementation (how to store)
        export {NONE} all
        redefine ...
        end
    STACK \[G]             -- Interface (what operations)
        undefine ...
        end
```

**Canonical Example:** `base/elks/structures/dispenser/arrayed\_stack.e`

**Semantic Meaning:**

* ARRAYED\_LIST provides the storage mechanism
* STACK defines the abstract dispenser behavior
* `export {NONE}` hides implementation details from clients

**FAQ Pairs to Generate:**

* Q: How do I combine an interface with an implementation in Eiffel?
* Q: When should I use `export {NONE}` in inheritance?

---

## Pattern 2: Bounded Generic Constraints

**Prevalence:** Common (30+ classes)

```eiffel
class SORTED\_SET \[G -> COMPARABLE]
inherit
    LINEAR\_SUBSET \[G]
    COMPARABLE\_STRUCT \[G]
```

**Canonical Example:** `base/elks/structures/set/binary\_search\_tree\_set.e`

**Semantic Meaning:**

* The constraint `G -> COMPARABLE` is a precondition on the generic
* Multiple inheritance provides both container behavior AND comparison operations

---

## Pattern 3: Tree Structure Composition

**Prevalence:** Specialized (tree classes)

```eiffel
class LINKED\_TREE \[G]
inherit
    DYNAMIC\_TREE \[G]          -- Tree interface
        redefine child\_start, child\_forth, ...
        end
    CELL \[G]                  -- Single value storage
        rename item as node\_item, put as node\_put
        undefine is\_equal, copy
        end
    LINKED\_LIST \[LIKE Current] -- Child storage
        rename ... 15+ renames
        export {NONE} all
        end
```

**Canonical Example:** `base/elks/structures/tree/linked\_tree.e`

**Semantic Meaning:**

* A tree IS a node (CELL) that HAS children (LINKED\_LIST) and BEHAVES as a tree (DYNAMIC\_TREE)
* 15+ renames resolve vocabulary conflicts (list's "item" vs tree's "item")
* Demonstrates Eiffel's rename/redefine/undefine power

---

## Pattern 4: Hash Table Aggregation (5 Parents)

**Prevalence:** Core data structures

```eiffel
class HASH\_TABLE \[G, K -> detachable HASHABLE]
inherit
    TABLE \[G, K]
    READABLE\_INDEXABLE \[G]
    TABLE\_ITERABLE \[G, K]
    MISMATCH\_CORRECTOR
    DEBUG\_OUTPUT
```

**Canonical Example:** `base/elks/structures/table/hash\_table.e`

**Semantic Meaning:**

* TABLE: core key-value semantics
* READABLE\_INDEXABLE: array-like access
* TABLE\_ITERABLE: across-loop support
* MISMATCH\_CORRECTOR: serialization compatibility
* DEBUG\_OUTPUT: debugging representation

---

## Pattern 5: COMPARABLE + HASHABLE Mixins

**Prevalence:** Very common

```eiffel
class MY\_KEY
inherit
    COMPARABLE
        redefine is\_less
        end
    HASHABLE
        redefine hash\_code
        end
```

**Semantic Meaning:**

* COMPARABLE: enables sorting and comparison operations
* HASHABLE: enables use as hash table key
* Together: full value semantics for custom types

---

## Pattern 6: Non-Conforming Inheritance

**Prevalence:** Kernel classes

```eiffel
class ROUTINE \[BASE\_TYPE, OPEN\_ARGS -> detachable TUPLE]
inherit
    HASHABLE
    DEBUG\_OUTPUT
inherit {NONE}              -- Non-conforming!
    REFLECTOR
        export {NONE} all
        end
```

**Canonical Example:** `base/elks/kernel/routine.e`

**Semantic Meaning:**

* `inherit {NONE}` = implementation-only inheritance
* ROUTINE uses REFLECTOR internally but is NOT a REFLECTOR
* Prevents invalid polymorphic assignments

**FAQ Pairs to Generate:**

* Q: What is non-conforming inheritance in Eiffel?
* Q: When should I use `inherit {NONE}`?

---

## Feature Adaptation Summary

| Mechanism | Purpose | Example |
|-----------|---------|---------|
| `rename old as new` | Resolve name conflicts | `item as node\_item` |
| `redefine` | Change implementation | `redefine is\_less` |
| `undefine` | Remove for diamond resolution | `undefine is\_equal` |
| `select` | Choose version for dynamic binding | `select forth` |
| `export {NONE}` | Hide inherited features | Hide ARRAYED\_LIST internals |
| `export {ANY}` | Re-export hidden features | Make feature public again |

---

## Key Semantic Insight

Multiple inheritance in Eiffel is **principled composition**:

1. **One parent provides implementation** (storage, algorithms)
2. **Other parents provide interfaces** (contracts, types)
3. **Feature adaptation** resolves conflicts systematically
4. **Non-conforming inheritance** separates IS-A from USES

This is fundamentally different from "diamond problem" fears in other languages.

---

## References to Canonical Code

| Pattern | File | Line Reference |
|---------|------|----------------|
| Mixin+Impl | `structures/dispenser/arrayed\_stack.e` | Lines 1-50 |
| Tree Composition | `structures/tree/linked\_tree.e` | Lines 1-80 |
| Hash Aggregation | `structures/table/hash\_table.e` | Lines 1-40 |
| Non-Conforming | `kernel/routine.e` | Lines 10-30 |
| Bounded Generic | `structures/set/binary\_search\_tree\_set.e` | Lines 1-20 |

