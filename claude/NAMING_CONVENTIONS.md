# Eiffel Naming Conventions

> Consolidated from [reference_docs](file:///D:/prod/reference_docs/) and official [Eiffel.org documentation](https://www.eiffel.org/doc/eiffel/Style_Guidelines)

## Quick Reference

| Element | Convention | Example |
|---------|------------|---------|
| Class | ALL_CAPS with underscores | `LINKED_LIST`, `PRICING_MODEL` |
| Feature | all_lowercase with underscores | `set_owner`, `yearly_premium` |
| Constant | Initial_cap with underscores | `Pi`, `Welcome_message` |
| Generic parameter | Single uppercase letter | `G`, `K`, `V` |
| Local variable | lowercase, abbreviated OK | `i`, `c`, `succ` |
| Argument | `a_` prefix + lowercase | `a_name`, `a_count` |
| Loop cursor | `ic` or `ic_` prefix (informal) | `ic`, `ic_item`, `ic_attr` |
| Attachment Local | `al` or `al_` prefix (informal) | `al`, `al_item` |

---

## 1. Class Names

**Rule**: ALL_CAPS with underscores separating words.

```eiffel
POINT
LINKED_LIST
PRICING_MODEL
HASH_TABLE
```

**Content guidelines**:
- Use **nouns** for concrete classes
- Use **adjectives** for deferred classes describing structural properties (e.g., `COMPARABLE`, `HASHABLE`)
- Use full words, not abbreviations (unless widely accepted like `HTTP`, `JSON`, `SQL`)

**Critical**: Eiffel class names are **globally unique** within a system. There are no namespaces.

**Sources**: [Style Guidelines](https://www.eiffel.org/doc/eiffel/Style_Guidelines), [ET: Lexical Conventions](https://www.eiffel.org/doc/eiffel/ET-_Lexical_Conventions_and_Style_Rules)

---

## 2. Feature Names

**Rule**: all_lowercase with underscores separating words.

```eiffel
set_owner
yearly_premium
last_month_balance
```

### 2.1 Commands (Procedures)

Use **verbs** in infinitive/imperative form:

| Pattern | Example | Use Case |
|---------|---------|----------|
| `make` | `make`, `make_from_string` | Creation procedures |
| `set_X` | `set_color`, `set_position` | Attribute setters |
| Action verb | `move`, `deposit`, `remove` | State-changing operations |
| `extend` | `extend (item)` | Add element to collection |
| `wipe_out` | `wipe_out` | Clear all elements |

### 2.2 Queries (Functions/Attributes)

**Non-boolean queries** use **nouns**:
```eiffel
number        -- not part_number (avoid redundancy with class name)
count
item
capacity
last_month_balance
```

**Boolean queries** use **adjectives** or `is_` / `has_` prefix:
```eiffel
full
empty
is_empty
is_valid
has_key
```

### 2.3 Standard Feature Names (EiffelBase conventions)

| Category | Feature | Purpose |
|----------|---------|---------|
| Access | `item` | Access an element |
| Access | `count` | Number of elements |
| Access | `capacity` | Maximum elements |
| Status | `is_empty` | No elements? |
| Status | `is_full` | At capacity? |
| Status | `has (v)` | Contains value? |
| Status | `is_extendible` | Can add elements? |
| Status | `is_prunable` | Can remove elements? |
| Command | `extend (v)` | Add element |
| Command | `put (v)` | Add/replace element |
| Command | `replace (v)` | Replace current |
| Command | `force (v)` | Add, resizing if needed |
| Command | `remove` | Remove current |
| Command | `prune (v)` | Remove specific value |
| Command | `wipe_out` | Remove all |

**Sources**: [Style Guidelines](https://www.eiffel.org/doc/eiffel/Style_Guidelines), [Eiffel Class and Feature Names](https://www.eiffel.org/doc/solutions/Eiffel_Class_and_Feature_Names)

---

## 3. Constants and Once Functions

**Rule**: Initial_cap with remaining lowercase.

```eiffel
Pi: REAL_64 = 3.14159265358979

Welcome_message: STRING = "Welcome!"

Error_window: EV_WINDOW
    once
        create Result.make
    end
```

**Sources**: [Style Guidelines](https://www.eiffel.org/doc/eiffel/Style_Guidelines)

---

## 4. Arguments and Local Variables

### 4.1 Arguments

**Modern convention**: Prefix with `a_` (for "argument"):

```eiffel
set_name (a_name: STRING)
    do
        name := a_name
    end

transfer (a_source: ACCOUNT; a_amount: DECIMAL)
```

### 4.2 Local Variables

Local scope allows **abbreviated names**:

```eiffel
local
    i: INTEGER           -- loop counter (conventional, OK)
    j, k: INTEGER        -- additional loop indices (OK)
    c: CHARACTER         -- single char (OK)
    n: INTEGER           -- count (OK)
    s: STRING            -- temporary string
    l_result: like item  -- prefixed for clarity
```

**Short loop counter names are conventional** and do NOT need `l_` prefix:
- `i`, `j`, `k` for integer indices in from-loops
- `c`, `n`, `x`, `y` for simple counters or characters
- These are universally understood in programming

**Prefix conventions** (informal but common):
- `l_` for locals when clarity needed (not required for simple counters)
- `ic_` for across-loop cursors (avoids VOIT(2) conflicts)

### 4.3 VOIT(2) Conflict Avoidance

Loop variable names **cannot conflict** with feature names:

```eiffel
-- WRONG: 'attr' may conflict with a feature named 'attr'
across attributes as attr loop
    process (attr.item)
end

-- RIGHT: Use ic_ prefix
across attributes as ic_attr loop
    process (ic_attr.item)
end
```

**Sources**: [ET: Lexical Conventions](https://www.eiffel.org/doc/eiffel/ET-_Lexical_Conventions_and_Style_Rules), [gotchas.md](file:///D:/prod/reference_docs/language/gotchas.md)

---

## 5. Generic Parameters

**Rule**: Single uppercase letters.

```eiffel
class HASH_TABLE [G, K -> HASHABLE]

class LINKED_LIST [G]

class FUNCTION [ARGS -> TUPLE, RESULT_TYPE]
```

Common conventions:
- `G` - General element type
- `K` - Key type
- `V` - Value type
- `T` - Type parameter

**Sources**: [Style Guidelines](https://www.eiffel.org/doc/eiffel/Style_Guidelines)

---

## 6. Contract Naming Patterns

### 6.1 Precondition Tags

| Pattern | Example | Meaning |
|---------|---------|---------|
| `X_valid` | `index_valid: valid_index (a_index)` | Parameter validation |
| `X_not_empty` | `name_not_empty: not a_name.is_empty` | Non-empty requirement |
| `X_attached` | `item_attached: a_item /= Void` | Non-void requirement |
| `X_positive` | `count_positive: a_count > 0` | Numeric constraint |
| `has_X` | `has_connection: connection /= Void` | State requirement |
| `not_X` | `not_closed: not is_closed` | Negative state |

### 6.2 Postcondition Tags

| Pattern | Example | Meaning |
|---------|---------|---------|
| `X_set` | `name_set: name = a_name` | Attribute assigned |
| `X_increased` | `count_increased: count = old count + 1` | Delta change |
| `result_X` | `result_not_void: Result /= Void` | Result guarantee |
| `X_unchanged` | `count_unchanged: count = old count` | No side effect |

### 6.3 Invariant Tags

| Pattern | Example | Meaning |
|---------|---------|---------|
| `X_valid` | `count_valid: count >= 0` | Always-valid state |
| `X_consistent` | `bounds_consistent: lower <= upper` | Related values |

**Sources**: [patterns.md](file:///D:/prod/reference_docs/language/patterns.md)

---

## 7. Creation Procedure Naming

**Convention**: Names begin with `make`:

| Pattern | Example | Use Case |
|---------|---------|----------|
| `make` | `make` | Default creation |
| `make_empty` | `make_empty` | Empty initialization |
| `make_from_X` | `make_from_string (s)` | Conversion creation |
| `make_with_X` | `make_with_capacity (n)` | Parameterized creation |
| `default_create` | (inherited) | No-argument creation |

```eiffel
class ACCOUNT

create
    make,
    make_with_balance

feature {NONE} -- Initialization

    make (a_number: STRING)
        do
            number := a_number
            balance := 0
        end

    make_with_balance (a_number: STRING; a_balance: DECIMAL)
        do
            number := a_number
            balance := a_balance
        end
```

**Sources**: [Constructors and Creation Procedures](https://www.eiffel.org/doc/solutions/Constructors_and_Creation_Procedures)

---

## 8. Conversion Feature Naming

| Pattern | Example | Meaning |
|---------|---------|---------|
| `to_X` | `to_string`, `to_integer` | Convert to type X |
| `as_X` | `as_lower`, `as_upper` | View as variant |
| `X_value` | `integer_value`, `real_value` | Extract typed value |

```eiffel
to_string: STRING
    -- String representation

as_lower: like Current
    -- Lowercase copy

integer_value: INTEGER
    -- Value as integer
```

---

## 9. Fluent Builder Naming

For builder pattern classes:

| Prefix | Meaning | Returns |
|--------|---------|---------|
| `set_X` | Replace value | `like Current` |
| `with_X` | Add/configure | `like Current` |
| `add_X` | Accumulate | `like Current` |

```eiffel
class HTTP_REQUEST_BUILDER

feature -- Building

    set_method (a_method: STRING): like Current
        do
            method := a_method
            Result := Current
        end

    with_header (a_name, a_value: STRING): like Current
        do
            headers.extend ([a_name, a_value])
            Result := Current
        end

    add_parameter (a_key, a_value: STRING): like Current
        do
            parameters.extend ([a_key, a_value])
            Result := Current
        end
```

**Sources**: [patterns.md](file:///D:/prod/reference_docs/language/patterns.md)

---

## 10. Feature Clause Naming

Standard feature clause labels (for documentation grouping):

| Clause | Contents |
|--------|----------|
| `Initialization` | Creation procedures |
| `Access` | Public queries |
| `Measurement` | Size/count queries |
| `Status report` | Boolean status queries |
| `Element change` | Commands modifying elements |
| `Removal` | Commands removing elements |
| `Transformation` | Commands transforming state |
| `Conversion` | Type conversion features |
| `Duplication` | Copying features |
| `Basic operations` | Core functionality |
| `Implementation` | Private helpers |

```eiffel
feature {NONE} -- Initialization

feature -- Access

feature -- Measurement

feature -- Status report

feature -- Element change

feature {NONE} -- Implementation
```

**Sources**: [Feature clauses](https://www.eiffel.org/doc/eiffelstudio/Feature_clauses)

---

## 11. Semantic Frame Naming (Multi-Name Features)

Eiffel allows **multiple names** for a single routine, enabling domain-specific vocabulary:

```eiffel
name,
account_holder,    -- banking domain
username,          -- authentication domain
gamertag           -- gaming domain
    : STRING_32
    do
        Result := internal_name
    end
```

**Critical limitation**: This works for **routines only**, NOT attributes:

```eiffel
-- WRONG: Creates TWO separate attributes!
x, y: INTEGER

-- RIGHT: Separate declarations
x: INTEGER
y: INTEGER
```

**When to use**:
- High fan-in supplier classes used across multiple domains
- Bridge classes connecting different subsystems
- API facades presenting domain-appropriate vocabulary

**Sources**: [SEMANTIC_FRAME_NAMING.md](file:///D:/prod/reference_docs/standards/SEMANTIC_FRAME_NAMING.md)

---

## 12. TUPLE Label Restrictions

TUPLE labels **cannot shadow** TUPLE's own features:

```eiffel
-- WRONG: 'count' shadows TUPLE.count
l_data: TUPLE [word: STRING; count: INTEGER]

-- RIGHT: Use different name
l_data: TUPLE [word: STRING; cnt: INTEGER]
l_data: TUPLE [word: STRING; word_count: INTEGER]
```

Reserved TUPLE feature names to avoid:
- `count`
- `is_equal`
- `default`
- `generating_type`
- `generator`

**Sources**: [EIFFEL_EXPERT_BRIEFING.md](file:///D:/prod/reference_docs/briefings/EIFFEL_EXPERT_BRIEFING.md)

---

## 13. Composite Name Rules

### 13.1 Use Underscores, Not CamelCase

```eiffel
-- WRONG (Smalltalk/Java style)
yearlyPremium
setOwner
linkedList

-- RIGHT (Eiffel style)
yearly_premium
set_owner
linked_list
```

### 13.2 Avoid Redundancy with Class Name

```eiffel
class PART

feature -- Access

    -- WRONG: Redundant 'part' prefix
    part_number: STRING
    part_name: STRING

    -- RIGHT: Class context implies 'part'
    number: STRING
    name: STRING
```

**Sources**: [Style Guidelines](https://www.eiffel.org/doc/eiffel/Style_Guidelines)

---

## 14. Header Comments

Every feature should have a **header comment** immediately after its signature:

```eiffel
balance: DECIMAL
    -- Current account balance in local currency

deposit (a_amount: DECIMAL)
    -- Add `a_amount` to current balance.
    require
        positive: a_amount > 0
    do
        balance := balance + a_amount
    ensure
        increased: balance = old balance + a_amount
    end
```

EiffelStudio tools use header comments for:
- Auto-generated documentation
- Feature completion tooltips
- Class interface views

**Sources**: [ET: Hello World](https://www.eiffel.org/doc/eiffel/ET-_Hello_World)

---

## 15. Summary: Do's and Don'ts

### Do

- Use ALL_CAPS for classes
- Use all_lowercase with underscores for features
- Use Initial_cap for constants
- Prefix arguments with `a_`
- Prefix across-loop cursors with `ic_` (avoids VOIT(2))
- Use short names (`i`, `j`, `k`, `c`, `n`) for from-loop counters
- Use standard EiffelBase names (`item`, `count`, `extend`, etc.)
- Write header comments for all features
- Use verbs for commands, nouns for queries
- Use `is_` or `has_` prefix for boolean queries

### Don't

- Use CamelCase (`yearlyPremium`)
- Use abbreviations unless universally known
- Include class name in feature names (`part_number` in class `PART`)
- Use `count` as a TUPLE label
- Name loop variables same as features
- Create multi-name attributes (only works for routines)
- Omit underscores in multi-word identifiers

---

## References

### Official Eiffel.org
- [Style Guidelines](https://www.eiffel.org/doc/eiffel/Style_Guidelines)
- [ET: Lexical Conventions and Style Rules](https://www.eiffel.org/doc/eiffel/ET-_Lexical_Conventions_and_Style_Rules)
- [Eiffel Class and Feature Names](https://www.eiffel.org/doc/solutions/Eiffel_Class_and_Feature_Names)
- [Constructors and Creation Procedures](https://www.eiffel.org/doc/solutions/Constructors_and_Creation_Procedures)
- [Feature clauses](https://www.eiffel.org/doc/eiffelstudio/Feature_clauses)

### Simple Eiffel Ecosystem
- [SEMANTIC_FRAME_NAMING.md](file:///D:/prod/reference_docs/standards/SEMANTIC_FRAME_NAMING.md)
- [EIFFEL_EXPERT_BRIEFING.md](file:///D:/prod/reference_docs/briefings/EIFFEL_EXPERT_BRIEFING.md)
- [patterns.md](file:///D:/prod/reference_docs/language/patterns.md)
- [gotchas.md](file:///D:/prod/reference_docs/language/gotchas.md)
- [EIFFEL_MENTAL_MODEL.md](file:///D:/prod/reference_docs/claude/EIFFEL_MENTAL_MODEL.md)
