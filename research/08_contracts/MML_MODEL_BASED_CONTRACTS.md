# Model-Based Contracts and MML for Simple Eiffel

**Date**: January 21, 2026
**Context**: Response to Bertrand Meyer's suggestion that Simple Eiffel adopt model-based contracts using MML
**Status**: Initial Research

---

## Background

On January 21, 2026, Bertrand Meyer responded to the Simple Eiffel announcement on the Eiffel Users mailing list. While acknowledging the naming concern (which Ian Joyner also raised), he offered a substantive technical suggestion:

> "If really it is meant to be a better library, it should use model-based contracts, namely MML."

This document captures research into what model-based contracts are, how MML works, and what adoption would mean for Simple Eiffel.

---

## What Are Model-Based Contracts?

### The Problem with Traditional Contracts

Traditional Design by Contract postconditions often specify *partial* behavior:

```eiffel
-- Traditional partial postcondition
push (x: G)
    ensure
        top_is_x: item = x
        count_increased: count = old count + 1
```

This says the top is `x` and count increased, but doesn't specify what happened to the rest of the stack. Did the other elements stay in place? The contract doesn't say.

### The Model-Based Solution

Model-based contracts associate each class with a **mathematical model** - a pure mathematical object that represents the abstract state. Then postconditions describe **complete transformations** on that model:

```eiffel
-- Model-based complete postcondition
push (x: G)
    ensure
        sequence_effect: sequence |=| (<< x >>) + old sequence
```

This says: "The new sequence IS the element `x` prepended to the old sequence." Nothing is left unspecified.

### Key Insight

The mathematical model makes the abstraction explicit:
- A **STACK** *is* a mathematical sequence with LIFO operations
- A **SET** *is* a mathematical set with membership operations
- A **TABLE** *is* a mathematical map (partial function)

---

## The Mathematical Model Library (MML)

### What It Is

MML is a library of **immutable, side-effect-free classes** that represent mathematical concepts. Because they're immutable, they can safely appear in contracts without causing side effects.

### Core Mathematical Types

| MML Class | Mathematical Concept | Models For |
|-----------|---------------------|------------|
| `MML_SEQUENCE [G]` | Ordered sequence | Lists, stacks, queues, arrays, iterators |
| `MML_SET [G]` | Unordered set | Sets, collections, unique containers |
| `MML_BAG [G]` | Multiset | Collections with duplicates |
| `MML_MAP [K, V]` | Partial function | Tables, dictionaries, hash maps |
| `MML_RELATION [G, H]` | Binary relation | Graphs, associations |

### MML_SEQUENCE Operations

The sequence type is the most commonly used. Operations include:

| Operation | Description | Example |
|-----------|-------------|---------|
| `sequence [i]` | Element at index i | `sequence [1]` |
| `sequence.count` | Number of elements | `sequence.count = 5` |
| `sequence.front (i)` | First i elements | `sequence.front (3)` |
| `sequence.tail (i)` | Elements from index i onward | `sequence.tail (2)` |
| `sequence.extended (v)` | New sequence with v appended | `sequence.extended (x)` |
| `sequence.prepended (v)` | New sequence with v at front | `sequence.prepended (x)` |
| `sequence.replaced_at (i, v)` | New sequence with v at position i | `sequence.replaced_at (2, x)` |
| `sequence.removed_at (i)` | New sequence without element at i | `sequence.removed_at (3)` |
| `s1 + s2` | Concatenation | `front + tail` |
| `sequence.interval (i, j)` | Subsequence from i to j | `sequence.interval (2, 5)` |

### MML_SET Operations

| Operation | Description |
|-----------|-------------|
| `set.has (x)` | Membership test |
| `set.extended (x)` | New set with x added |
| `set.removed (x)` | New set with x removed |
| `set & other` | Intersection |
| `set \| other` | Union |
| `set - other` | Difference |

### MML_MAP Operations

| Operation | Description |
|-----------|-------------|
| `map [k]` | Value for key k |
| `map.domain` | Set of keys |
| `map.range` | Set of values |
| `map.extended (k, v)` | New map with k->v added |
| `map.removed (k)` | New map with key k removed |

---

## How Model-Based Contracts Work

### Step 1: Declare the Model

In the class note, declare what mathematical object represents this class:

```eiffel
class STACK [G]

note
    model: sequence

feature -- Model
    sequence: MML_SEQUENCE [G]
        -- Mathematical model of this stack
```

### Step 2: Write Model Queries

Model queries expose the abstract state. They can be:
- **Dedicated model queries**: Created specifically for specification
- **Regular queries reused**: Existing queries that fit the model

```eiffel
feature -- Model Queries
    sequence: MML_SEQUENCE [G]
        -- Abstract sequence representing stack contents

    -- 'count' is a regular query that also serves as model query
    count: INTEGER
        -- Number of elements
```

### Step 3: Complete Postconditions

Write postconditions that fully specify behavior in terms of the model:

```eiffel
push (v: G)
    -- Push `v` onto the stack
    ensure
        sequence_effect: sequence |=| (<< v >>) + old sequence
        count_effect: count = old count + 1

pop
    -- Remove top element
    require
        not_empty: count > 0
    ensure
        sequence_effect: sequence |=| old sequence.tail (2)
        count_effect: count = old count - 1

item: G
    -- Top element
    require
        not_empty: count > 0
    ensure
        definition: Result = sequence [1]
```

### Step 4: Invariants Connect Model to Implementation

The class invariant connects the abstract model to the concrete implementation:

```eiffel
invariant
    sequence_count: sequence.count = count
    sequence_matches_array: across 1 |..| count as i all
        sequence [i] = storage [count - i + 1]
    end
```

---

## EiffelBase2: MML in Practice

### About EiffelBase2

EiffelBase2 is a fully verified container library developed at ETH Zurich by Nadia Polikarpova and others. It serves as the canonical example of model-based contracts in Eiffel.

**Repository**: https://github.com/nadia-polikarpova/eiffelbase2

### Design Principles

1. **All classes have models**: Every container class declares its mathematical model
2. **Complete specifications**: Postconditions fully specify behavior
3. **Immutable interfaces**: Read-only access patterns
4. **Verification-ready**: Designed for use with AutoProof

### Naming Convention

All classes use the `V_` prefix (for "Verified"):
- `V_ARRAY`, `V_LIST`, `V_LINKED_LIST`
- `V_SET`, `V_HASH_SET`
- `V_TABLE`, `V_HASH_TABLE`
- `V_STACK`, `V_QUEUE`

### Real Example: V_LINKED_LIST

```eiffel
class V_LINKED_LIST [G]

note
    model: sequence
    manual_inv: true

feature -- Model
    sequence: MML_SEQUENCE [G]
        -- Abstract sequence of elements

feature -- Modification
    extend_back (v: G)
        -- Add `v` at the end
        ensure
            sequence_effect: sequence |=| old sequence.extended (v)

    put_front (v: G)
        -- Add `v` at the front
        ensure
            sequence_effect: sequence |=| old sequence.prepended (v)

invariant
    cells_domain: sequence.count = cells.count
    sequence_implementation: across 1 |..| cells.count as i all
        sequence [i] = cells [i].item
    end
```

---

## Benefits of Model-Based Contracts

### 1. Completeness

Traditional contracts often leave behavior partially specified. Model-based contracts force you to say exactly what happens to the entire abstract state.

### 2. Improved Bug Detection

Research by Polikarpova, Furia, and Meyer showed that the same automated testing tools find **2x more real bugs** when applied to code with model-based contracts versus traditional contracts.

> "The verdict on the effect of strong specifications as captured by automated testing is clear: the same automatic testing tools applied to the versions with strong contracts yield twice as many real faults."

### 3. Clearer Abstractions

The mathematical model makes the abstraction explicit. When you say a list "is" a sequence, you're forced to think clearly about what operations mean mathematically.

### 4. Modest Effort

The research found that creating model-based specifications requires about **5 person-weeks of effort** for a substantial library, yielding about **4 defects detected per person-day** of specification work.

### 5. Better Documentation

The model serves as precise documentation. A developer reading the model knows exactly what abstraction the class represents.

---

## Challenges and Considerations

### 1. MML Dependency

Adopting model-based contracts requires including MML or writing your own model classes. EiffelBase2's MML is available but may need adaptation.

### 2. Learning Curve

Contributors need to understand:
- Mathematical concepts (sequences, sets, maps)
- How to express operations as transformations
- The immutability requirement for model classes

### 3. Verbosity

Model-based postconditions are often longer than traditional ones. However, they're more precise.

### 4. Tool Support

Full benefits require verification tools like AutoProof. Without them, model-based contracts still improve testing and documentation but don't provide formal guarantees.

### 5. Compatibility

MML classes must be immutable and side-effect-free. This is essential for contracts but requires discipline.

---

## Applicability to Simple Eiffel

### Libraries That Would Benefit Most

| Library | Potential Model |
|---------|-----------------|
| `simple_container` | `MML_SEQUENCE`, `MML_SET`, `MML_MAP` |
| `simple_json` | `MML_MAP` for objects, `MML_SEQUENCE` for arrays |
| `simple_xml` | Tree structure model |
| `simple_graph` | `MML_RELATION` for edges |
| `simple_http` | Request/response as structured models |
| `simple_sql` | Result sets as sequences of maps |

### Libraries Where It's Less Applicable

- **I/O libraries** (simple_file, simple_process): Effects are external
- **FFI wrappers** (simple_c, simple_win32): Wrapping C behavior
- **UI libraries**: State is often in external systems

### Incremental Adoption Path

1. **Phase 1**: Create `simple_mml` library with core model classes
2. **Phase 2**: Apply to `simple_container` as proof of concept
3. **Phase 3**: Extend to other data structure libraries
4. **Phase 4**: Document patterns for ecosystem-wide adoption

---

## Initial Assessment

### What We Expect to Find

Examining Simple Eiffel libraries for MML applicability, we anticipate:

1. **Good fit for data structures**: `simple_container`, `simple_json`, `simple_xml` could benefit significantly from model-based contracts

2. **Partial fit for I/O**: Libraries like `simple_http` and `simple_sql` could model their data representations (responses, result sets) but not I/O effects

3. **Challenge with inline C**: Many `simple_*` libraries use inline C for Win32 APIs. The C layer isn't verifiable, so model-based contracts would apply to the Eiffel facade only

4. **Ecosystem-wide benefit**: Even partial adoption would improve specification quality and could catch more bugs through AutoTest

### Questions to Investigate

1. Can MML work with SCOOP? (Simple Eiffel requires SCOOP compatibility)
2. What subset of MML is sufficient for Simple Eiffel's needs?
3. How do model-based contracts interact with inline C externals?
4. What tooling support exists for model-based contracts today?

---

## References

### Primary Sources

- **MML Project Page**: https://se.inf.ethz.ch/people/schoeller/mml.html
- **EiffelBase2 Repository**: https://github.com/nadia-polikarpova/eiffelbase2
- **EiffelBase2 Documentation**: https://dev.eiffel.com/EiffelBase2

### Papers

- Schoeller, Widmer & Meyer: "Making Specifications Complete Through Models" (2006)
- Polikarpova, Furia & Meyer: "Specifying Reusable Components" (VSTTE 2010)

### Blog Posts

- Bertrand Meyer: "Strong specifications are well worth the effort" (2013)
  https://bertrandmeyer.com/2013/05/22/reading-notes-strong-specifications-are-well-worth-the-effort/

---

## Next Steps

1. **Clone EiffelBase2** and examine MML implementation
2. **Identify minimal MML subset** needed for Simple Eiffel
3. **Prototype on simple_container**: Apply model-based contracts to one library
4. **Evaluate SCOOP compatibility**: Test MML with concurrent code
5. **Document findings** for ecosystem adoption decision

---

*This document will be updated as research progresses.*
