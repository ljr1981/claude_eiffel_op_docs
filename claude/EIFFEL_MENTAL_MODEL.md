# Eiffel Mental Model for AI Assistants

**Purpose:** Internalized understanding of Eiffel for generating correct, idiomatic code.

**Sources:** ECMA-367 (2nd Edition), eiffel.org documentation, practical experience with simple_* libraries.

---

## Core Philosophy

Eiffel is not just a programming language—it's a **full lifecycle software development method**. The three primary quality goals:

1. **Reusability** - Components usable across many applications
2. **Extendibility** - Easily modifiable software
3. **Reliability** - Correct and robust (bug-free)

Additional goals: efficiency, openness, portability, precision.

> "Object-Oriented" is a misnomer; "Class-Oriented Analysis, Design and Programming" is more accurate.

---

## Design by Contract (DbC)

The **centerpiece** of the Eiffel method. Software construction as contracts between clients (callers) and suppliers (routines).

### The Contract

| Party | Obligation | Benefit |
|-------|-----------|---------|
| Client | Satisfy precondition | Get postcondition guaranteed |
| Supplier | Satisfy postcondition | Can assume precondition |

### Assertion Types

```eiffel
feature
    withdraw (sum: INTEGER)
        require                           -- PRECONDITION
            positive: sum > 0
            sufficient: sum <= balance - minimum
        do
            balance := balance - sum
        ensure                            -- POSTCONDITION
            withdrawn: balance = old balance - sum
        end

invariant                                 -- CLASS INVARIANT
    sufficient_balance: balance >= minimum_balance
```

### Key Rules

- **Precondition violation** = Bug in CLIENT (caller didn't meet requirements)
- **Postcondition/invariant violation** = Bug in SUPPLIER (routine didn't deliver)
- A routine must either **succeed** (fulfill contract) or **fail** (propagate exception)
- **old expression** - Captures value at routine entry for postcondition comparison

### Monitoring Levels

| Level | What's Checked |
|-------|----------------|
| `no` | Nothing |
| `require` | Preconditions only |
| `ensure` | Pre + postconditions |
| `invariant` | Pre + post + class invariants |
| `all` | Everything including check/loop assertions |

---

## Command-Query Separation (CQS)

**Fundamental methodological principle** (not enforced by compiler):

| Type | Purpose | Returns | Side Effects |
|------|---------|---------|--------------|
| **Query** | Ask a question | Value | None |
| **Command** | Change state | Nothing | Yes |

```eiffel
-- WRONG (violates CQS)
pop: T
    do
        Result := top
        remove_top  -- Side effect in query!
    end

-- CORRECT
top: T              -- Query: returns value
remove             -- Command: modifies state
```

**Uniform Access Principle:** Client cannot tell if `account.balance` is an attribute (stored) or function (computed). Both use same syntax.

```eiffel
-- These are interchangeable from client's perspective:
balance: INTEGER                  -- Attribute (stored)
balance: INTEGER do ... end      -- Function (computed)
```

---

## Classes and Objects

- **Class** = Compile-time concept (template)
- **Object** = Run-time concept (instance)
- **Direct instance** = Object created from exactly this class

### Class Structure

```eiffel
class ACCOUNT

create
    make                         -- Creation procedure(s)

feature {NONE} -- Initialization
    make (initial: INTEGER)
        do
            balance := initial
        end

feature -- Access (exported to all)
    balance: INTEGER
    owner: PERSON

feature -- Basic operations
    deposit (sum: INTEGER)
        require sum > 0
        do add (sum)
        ensure balance = old balance + sum
        end

feature {NONE} -- Implementation (secret)
    add (sum: INTEGER)
        do balance := balance + sum end

invariant
    positive_balance: balance >= 0

end
```

### Feature Categories

| Category | Description |
|----------|-------------|
| Attribute | Data field |
| Constant attribute | `minimum: INTEGER = 1000` |
| Procedure | Command (no return) |
| Function | Query (returns value) |

---

## Types

### Reference vs Expanded

```eiffel
class ACCOUNT            -- Reference type (default)
expanded class POINT     -- Expanded type (copy semantics)
```

| Aspect | Reference | Expanded |
|--------|-----------|----------|
| Assignment | Copies reference | Copies entire object |
| Comparison (=) | Reference equality | Value equality |
| Default | Void | Object with defaults |
| Memory | Pointer to heap | Inline/subobject |

### Basic Types (all expanded)

- `INTEGER`, `INTEGER_8`, `INTEGER_16`, `INTEGER_32`, `INTEGER_64`
- `REAL`, `DOUBLE`
- `CHARACTER`, `CHARACTER_32`
- `BOOLEAN`
- `STRING` (reference, but behaves value-like)

---

## Void Safety

Static protection against null pointer exceptions.

### Attached vs Detachable

```eiffel
x: ACCOUNT              -- Attached (default) - cannot be Void
y: detachable ACCOUNT   -- Detachable - may be Void
```

### Rules

1. Attached types **cannot** hold Void
2. Qualified calls `x.f(...)` only allowed on attached types
3. Detachable → Attached requires check

### Certified Attachment Patterns (CAPs)

```eiffel
-- Object test
if attached y as l_y then
    l_y.deposit (100)  -- l_y is attached within scope
end

-- Conditional attachment
if y /= Void then
    y.deposit (100)    -- y is attached within scope
end
```

---

## Inheritance

### Basic Syntax

```eiffel
class SAVINGS_ACCOUNT

inherit
    ACCOUNT                      -- Conforming (can assign to parent type)
        rename
            withdraw as basic_withdraw
        redefine
            deposit
        export
            {NONE} internal_feature
        end

inherit {NONE}
    HELPER                       -- Non-conforming (pure reuse)

feature
    deposit (sum: INTEGER)       -- Redefined version
        do
            Precursor (sum)      -- Call parent version
            add_interest
        end
```

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Conformance** | B conforms to A if B inherits from A |
| **Redefinition** | Override implementation (explicit `redefine`) |
| **Effecting** | Implement deferred feature |
| **Renaming** | Resolve name conflicts, improve naming |
| **Precursor** | Call parent version |

### Multiple Inheritance

```eiffel
class WINDOW
inherit
    RECTANGLE    -- Graphical features
    TREE [WINDOW]  -- Hierarchical features
```

---

## Polymorphism and Dynamic Binding

```eiffel
local
    p: POLYGON
    r: RECTANGLE
do
    create r.make (...)
    p := r                    -- Polymorphic assignment (valid)
    print (p.perimeter)       -- Dynamic binding: calls RECTANGLE version
end
```

### Subcontracting Rules

Redefined routines must:
- Have **weaker or equal** precondition
- Have **stronger or equal** postcondition

---

## Genericity

### Unconstrained

```eiffel
class LIST [G]
feature
    item: G
    put (v: G) do ... end
end

-- Usage
my_list: LIST [INTEGER]
accounts: LIST [ACCOUNT]
```

### Constrained

```eiffel
class SORTED_LIST [G -> COMPARABLE]
    -- G must be descendant of COMPARABLE
```

### Generic Creation

```eiffel
class FACTORY [G -> WIDGET create make end]
feature
    new_item: G
        do
            create Result.make  -- Allowed because constraint includes create
        end
end
```

---

## Deferred Classes and Features

Abstract classes that cannot be instantiated.

```eiffel
deferred class VEHICLE

feature
    register (year: INTEGER)
        require
            dues_paid (year)
        deferred                 -- No implementation
        ensure
            valid_plate (year)
        end
end

class CAR
inherit
    VEHICLE

feature
    register (year: INTEGER)
        do
            -- Concrete implementation (effecting)
        end
end
```

**Key insight:** Deferred classes can have:
- Mix of deferred and effective features
- Full contracts (pre/post/invariant)
- Gradual refinement toward concrete

---

## Once Routines

Execute only once, then return cached result.

```eiffel
database: DATABASE
    once
        create Result.make ("connection_string")
    end

config: CONFIGURATION
    once ("PROCESS")    -- Once per process
        create Result.load_from_file
    end

thread_local_cache: HASH_TABLE [STRING, INTEGER]
    once ("THREAD")     -- Once per thread
        create Result.make (100)
    end
```

Scopes: `PROCESS`, `THREAD`, `OBJECT`

---

## Agents (First-Class Routines)

```eiffel
-- Create agent for a routine
my_agent: PROCEDURE [TUPLE [INTEGER]]
my_agent := agent account.deposit (?)   -- ? = open operand

-- Call it
my_agent.call ([500])  -- Same as account.deposit (500)

-- Inline agent
do_something (agent (x: INTEGER) do print (x) end)
```

---

## SCOOP (Concurrency)

**Simple Concurrent Object-Oriented Programming**

### Core Concept

Single keyword: `separate`

```eiffel
class WORKER

feature
    process (data: separate DATA_STORE)
        require
            data.is_ready        -- Wait condition (waits until true)
        do
            data.process_items   -- Exclusive access guaranteed
        end
end
```

### Key Rules

1. **Separate objects** live on different processors
2. **Argument passing** grants exclusive access
3. **Preconditions become wait conditions** (wait until true, don't fail)
4. No explicit locks, threads, or synchronization

---

## Exception Handling

```eiffel
attempt_transmission (message: STRING)
    local
        failures: INTEGER
    do
        if failures < 50 then
            transmit (message)
            successful := True
        else
            successful := False
        end
    rescue
        failures := failures + 1
        retry                    -- Restart do clause
    end
```

### Principles

1. Routine must **succeed** (fulfill contract) or **fail** (trigger exception)
2. **rescue** clause: patch up, then `retry` or propagate
3. `retry` restarts the `do` clause
4. No `retry` = routine fails, exception propagates

---

## Eiffel Idioms for AI Code Generation

### Always Include Contracts

```eiffel
feature
    add_item (a_item: ITEM)
        require
            item_not_void: a_item /= Void
            not_full: count < capacity
        do
            items.extend (a_item)
        ensure
            one_more: count = old count + 1
            item_added: has (a_item)
        end
```

### Prefer Queries Over Attributes When Logic Needed

```eiffel
-- Better than storing is_valid separately
is_valid: BOOLEAN
    do
        Result := balance >= minimum_balance and owner /= Void
    end
```

### Use Creation Procedures

```eiffel
class ACCOUNT
create
    make, make_with_minimum

feature {NONE}
    make (a_owner: PERSON)
        require
            owner_exists: a_owner /= Void
        do
            owner := a_owner
            balance := 0
        ensure
            owner_set: owner = a_owner
            zero_balance: balance = 0
        end
```

### Handle Void Safety

```eiffel
if attached optional_value as l_val then
    -- Use l_val safely
else
    -- Handle missing case
end
```

### Document with note Clause

```eiffel
note
    description: "Bank account with overdraft protection"
    author: "Your Name"
    date: "$Date$"
    revision: "$Revision$"
```

---

## Key Validity Rules (Common Compile Errors)

### VAPE - Precondition Assertion Export
Preconditions can only reference features exported to all clients.
```eiffel
-- WRONG: private_attr is {NONE}
require valid: not private_attr.is_empty  -- VAPE error!

-- CORRECT: expose via public query
feature -- Status
    is_valid: BOOLEAN do Result := not private_attr.is_empty end
feature
    my_feature require valid: is_valid do ... end
```

### VJAR - Invalid Assignment Target
Assignment target type must be compatible with source.
```eiffel
-- WRONG: STRING_32 → STRING_8 implicit conversion
l_str8 := some_string_32_value  -- VJAR error!
-- CORRECT: l_str8 := some_string_32_value.to_string_8
```

### VDUS - Undefine Deferred Feature
Cannot `undefine` an already-deferred feature (undefine converts effective → deferred).

### VGCC - Cannot Create Deferred Class
Cannot `create` an instance of a deferred class.
```eiffel
-- WRONG: PATH_NAME is deferred
create l_path.make_from_string (...)  -- VGCC error!
```

### Feature Joining Rules (Multiple Inheritance)
- Deferred + Deferred = Single deferred (need one implementation)
- Deferred + Effective = Effective wins (automatic)
- Effective + Effective = CONFLICT - must use rename/undefine/select

---

## Common Gotchas (from practical experience)

1. **SQLite connections are thread-bound** - Per-request connections in web apps
2. **SQL statements need semicolons** - `is_complete_statement` precondition
3. **Multi-line SQL fails** - Keep SQL on single lines
4. **Semicolons are optional** - Omit them for cleaner code
5. **`old` only in postconditions** - Not in preconditions or body
6. **Void safety is compile-time** - Use `attached` patterns
7. **ARRAYED_LIST.has uses reference equality** - Use `across ... some ic ~ value end` for value equality
8. **Fluent API pattern** - Return `like Current` and `Result := Current` for chaining
9. **Test apps don't inherit TEST_SET_BASE** - Only test *sets* inherit it

---

## Using ec.exe for Code Understanding

When working with unfamiliar code or before making changes, use ec.exe introspection options to understand the codebase:

| Situation | Command | Purpose |
|-----------|---------|---------|
| Before modifying a class | `-clients CLASS` | See who depends on it |
| Understanding dependencies | `-suppliers CLASS` | See what it uses |
| VHRC inheritance errors | `-ancestors CLASS` | Check parent chain for available features |
| Learning a class API | `-short CLASS` | See public interface without implementation |
| Before changing a feature | `-callers CLASS.feature` | Find all call sites |
| Understanding full API | `-flatshort CLASS` | All public features including inherited |

**See:** `ec_introspection_howto.md` for complete use-case guide.

---

*Last updated: 2025-12-07*

**Sources:**
- [ECMA-367 Standard](https://ecma-international.org/publications-and-standards/standards/ecma-367/)
- [Design by Contract](https://www.eiffel.org/doc/solutions/Design_by_Contract_and_Assertions)
- [SCOOP Concurrency](https://www.eiffel.org/doc/solutions/Concurrent_programming_with_SCOOP)
- [Eiffel Language Reference](https://www.eiffel.org/doc/eiffel)
