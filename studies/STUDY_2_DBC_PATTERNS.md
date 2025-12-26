# Study 2: Design by Contract Patterns in Canonical Eiffel

**Date:** 2025-12-25
**Source:** ISE EiffelStudio 25.02 Standard Library
**Path:** `/c/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/library/base/`
**Files Analyzed:** 484

---

## Executive Summary

DBC in canonical Eiffel is not just "assertions" - it's a **specification language** embedded in code. The patterns below demonstrate how contracts express semantic meaning, not just runtime checks.

---

## Precondition Patterns

### Pattern 1: Attachment/Null Checks

```eiffel
put (v: G; k: K)
    require
        valid_key: k /= Void
        key_attached: attached k
```

**Semantic Meaning:** The feature CANNOT operate on void references. This is a type-system extension, not a runtime guard.

### Pattern 2: Boundary Validation

```eiffel
item alias "[]" (i: INTEGER): G
    require
        valid_index: valid_index (i)
        lower_bound: i >= lower
        upper_bound: i <= upper
```

**Canonical Example:** `base/elks/kernel/array.e:175`

**Semantic Meaning:** Index validity is the CALLER's responsibility. The array promises to deliver only if you promise valid input.

### Pattern 3: State Preconditions

```eiffel
remove
    require
        not_empty: not is_empty
        writable: not is_read_only
```

**Semantic Meaning:** Operations require specific object state. This documents and enforces state machine semantics.

### Pattern 4: Relational Preconditions

```eiffel
make_from_iterable (it: ITERABLE [G])
    require
        it_not_void: it /= Void
        -- Implicit: it is finite
```

### Pattern 5: Index Validity

```eiffel
go_i_th (i: INTEGER)
    require
        valid_cursor_index: valid_cursor_index (i)
```

**Pattern:** Delegate to a query that encapsulates complex validity logic.

---

## Postcondition Patterns

### Pattern 1: State Establishment

```eiffel
make (n: INTEGER)
    require
        non_negative: n >= 0
    do
        ...
    ensure
        capacity_set: capacity = n
        empty: is_empty
        lower_set: lower = 1
```

**Semantic Meaning:** After `make`, these properties HOLD. The constructor establishes invariant conditions.

### Pattern 2: Old Expression Comparisons

```eiffel
extend (v: G)
    do
        ...
    ensure
        one_more: count = old count + 1
        item_inserted: has (v)
        same_lower: lower = old lower
```

**Canonical Example:** `base/elks/structures/list/arrayed_list.e`

**Semantic Meaning:** `old` captures pre-state values. The postcondition relates pre-state to post-state.

### Pattern 3: Quantified Postconditions

```eiffel
wipe_out
    ensure
        is_empty: is_empty
        all_void: across Current as c all c.item = Void end
```

**Semantic Meaning:** Universal quantification over collection elements. Every element satisfies the property.

### Pattern 4: Conditional Postconditions

```eiffel
search (v: G)
    ensure
        found_or_after: found implies item = v
        not_found_implies_exhausted: not found implies exhausted
```

**Pattern:** Use `implies` for conditional guarantees.

### Pattern 5: Result Constraints

```eiffel
twin: like Current
    ensure
        twin_not_void: Result /= Void
        is_equal: Result ~ Current
```

---

## Class Invariant Patterns

### Pattern 1: Representation Invariant

```eiffel
invariant
    non_negative_count: count >= 0
    count_bounded: count <= capacity
    lower_not_greater_than_upper: lower <= upper + 1
    consistent_bounds: upper - lower + 1 = count
```

**Canonical Example:** `base/elks/kernel/array.e`

**Semantic Meaning:** These properties hold between ALL public feature calls. They define what it MEANS to be a valid array.

### Pattern 2: Ordering Invariants (Trichotomy)

```eiffel
class COMPARABLE
invariant
    irreflexive_comparison: not (Current < Current)
    -- Trichotomy: exactly one of <, =, > holds
```

**Canonical Example:** `base/elks/kernel/comparable.e`

**Semantic Meaning:** Mathematical properties that comparison must satisfy.

### Pattern 3: Structural Invariants

```eiffel
class LINKED_LIST [G]
invariant
    empty_means_no_active: is_empty implies (active = Void)
    active_in_bounds: attached active implies index >= 1 and index <= count
```

### Pattern 4: Bidirectional Consistency

```eiffel
invariant
    parent_child_consistency:
        attached parent as p implies p.has (Current)
```

---

## Loop Variant/Invariant Patterns

### Loop Variant (Termination Proof)

```eiffel
from
    i := count
invariant
    i >= 0
    processed: count - i items processed
variant
    i
until
    i = 0
loop
    process (item (i))
    i := i - 1
end
```

**Semantic Meaning:**
- **Variant:** A non-negative integer that DECREASES each iteration
- **Invariant:** Property that holds at each iteration start
- Together they PROVE termination

---

## Semantic Foundations

### Hoare Logic Connection

Eiffel DBC directly implements Hoare triples:

| Hoare | Eiffel |
|-------|--------|
| {P} | `require P` |
| S | `do S` |
| {Q} | `ensure Q` |

```
{P} S {Q} ≡ require P do S ensure Q end
```

### Contract Inheritance (Liskov Substitution)

```eiffel
class CHILD inherit PARENT
    redefine some_feature end

feature
    some_feature
        require else  -- Weaker (disjunction with parent)
            child_precondition
        do
            ...
        ensure then   -- Stronger (conjunction with parent)
            child_postcondition
        end
```

**Semantic Meaning:**
- `require else` = parent_precondition OR child_precondition (weaker)
- `ensure then` = parent_postcondition AND child_postcondition (stronger)
- This IS Liskov Substitution Principle in code!

---

## FAQ Pairs to Generate

1. Q: What's the difference between `require` and `require else`?
2. Q: How do I use `old` in postconditions?
3. Q: What makes a good class invariant?
4. Q: How do I prove a loop terminates in Eiffel?
5. Q: How does contract inheritance work?
6. Q: What is the relationship between DBC and Hoare logic?

---

## References to Canonical Code

| Pattern | File | Line Reference |
|---------|------|----------------|
| Boundary Preconditions | `kernel/array.e` | Lines 175-180 |
| State Postconditions | `structures/list/arrayed_list.e` | extend feature |
| Quantified Postconditions | `structures/list/arrayed_list.e` | wipe_out feature |
| Representation Invariant | `kernel/array.e` | invariant clause |
| Trichotomy Invariant | `kernel/comparable.e` | invariant clause |
| Loop Variant | `structures/list/linked_list.e` | search feature |
