# Study 5: SCOOP Concurrency Patterns in Canonical Eiffel

**Date:** 2025-12-26
**Source:** ISE EiffelStudio 25.02 Standard Library
**Focus:** SCOOP (Simple Concurrent Object-Oriented Programming)

---

## Executive Summary

SCOOP leverages Design by Contract for declarative concurrency. The `separate` keyword combined with preconditions creates automatic synchronization without explicit locks.

---

## 1. Separate Keyword Usage

### Declaration on Arguments

```eiffel
is_valid_peer_address (addr: attached separate like address): BOOLEAN
    require
        address_exists: addr /= Void
    do
        Result := True
    end
```

**File:** `library/net/abstract/socket/socket.e`

### Creating Separate Objects

```eiffel
create counter_1.make (1, 1000_000_000)
create counter_2.make (2, 500_000_000)
launch_two (counter_1, counter_2)

launch_two (a_counter_1: separate COUNTER; a_counter_2: separate COUNTER)
    do
        a_counter_1.run (50)
        a_counter_2.run (50)
    end
```

**File:** `examples/scoop/counter/application.e`

### Wait Conditions in Preconditions

```eiffel
launch_one_with_precondition (a_counter: separate COUNTER)
    require
        a_counter.value >= 200  -- WAITS until true
    do
        a_counter.run (100)
    end
```

Preconditions on separate objects become **wait conditions** - caller blocks until satisfied.

---

## 2. SCOOP Patterns

### Producer-Consumer (Dining Savages)

```eiffel
class POT
feature {COOK}
    fill
        require
            is_empty  -- Cook waits until pot empty
        do
            servings := max_servings
        ensure
            is_full
        end

feature {SAVAGE}
    get_meal
        require
            not is_empty  -- Savage waits until pot has food
        do
            servings := servings - 1
        end
end
```

**File:** `examples/scoop/dining_savages/pot.e`

### Lock Passing (Dining Philosophers)

```eiffel
eat (left, right: separate FORK)
    -- Two separate args = two independent locks
    do
        left.pick (Current)
        right.pick (Current)
        delay (200)
        left.put (Current)
        right.put (Current)
    end
```

**File:** `examples/scoop/dining_philosophers/philosopher.e`

### Import Pattern (Cross-Processor Data)

```eiffel
make (a_message: detachable separate STRING; a_port: INTEGER)
    local
        l_message: separate STRING
    do
        if a_message /= Void then
            l_message := a_message
        else
            l_message := default_message
        end
        create message.make_from_separate (l_message)  -- Import to local
    end
```

**File:** `library/net/test/client_stub.e`

### Inline Separate Syntax (Modern)

```eiffel
if attached a_args [l_index] as separate_argument then
    separate separate_argument as argument do
        Result.append (argument.out)
    end
end
```

**File:** `library/argument_parser/support/string_formatter.e`

---

## 3. Key Classes

### ISE_SCOOP_RUNTIME

```eiffel
class ISE_SCOOP_RUNTIME
feature
    region_id (a_object: separate ANY): NATURAL_16
        -- Get processor ID of separate object

    pin_to_thread
        -- Pin current processor to dedicated thread

    pin_processor_to_thread (a_object: separate ANY)
        -- Pin separate processor to thread
end
```

**File:** `library/base/elks/kernel/ise_scoop_runtime.e`

---

## 4. Common Idioms

### Safe Separate Access

```eiffel
-- Pattern 1: Precondition guard
require
    a_counter.value >= 200
do
    -- Guaranteed: a_counter.value IS >= 200

-- Pattern 2: Import and convert
local
    l_message: STRING
do
    create l_message.make_from_separate (a_separate_message)
    -- l_message is now non-separate, safe to use
```

### Multiple Separate Objects

```eiffel
eat (left, right: separate FORK)
    -- Each argument = independent processor
    -- SCOOP ensures mutual exclusion automatically
    -- No explicit lock ordering needed
```

### Exception Handling Across Processors

```eiffel
do_accept (other: separate like Current; a_address: like address)
    local
        retried: BOOLEAN
    do
        if not retried then
            -- work
        end
    rescue
        if not assertion_violation then
            retried := True
            retry
        end
    end
```

---

## 5. SCOOP vs Traditional Threading

| Aspect | SCOOP | Traditional Threads |
|--------|-------|---------------------|
| Lock mechanism | Preconditions | Explicit mutex/locks |
| Synchronization | Declarative | Imperative API calls |
| Deadlock | Compiler prevents | Programmer responsible |
| Race conditions | Automatic exclusion | Manual critical sections |

---

## 6. FAQ Pairs to Generate

1. What is SCOOP in Eiffel?
2. How do I declare a separate object?
3. How do preconditions work with separate objects?
4. How do I pass data between SCOOP processors?
5. What is the inline separate syntax?
6. How do I avoid deadlock in SCOOP?
7. How do I handle exceptions across processors?
8. What is lock passing in SCOOP?

---

## References

- `examples/scoop/counter/` - Simple counter with preconditions
- `examples/scoop/dining_philosophers/` - Lock passing pattern
- `examples/scoop/dining_savages/` - Producer-consumer pattern
- `library/base/elks/kernel/ise_scoop_runtime.e` - Runtime API
- `unstable/library/scoop_patterns/` - Reusable patterns library
