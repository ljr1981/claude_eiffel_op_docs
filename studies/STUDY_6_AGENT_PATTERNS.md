# Study 6: Agent (Lambda/Closure) Patterns in Canonical Eiffel

**Date:** 2025-12-26
**Source:** ISE EiffelStudio 25.02 Standard Library
**Focus:** Agents as first-class function objects

---

## Executive Summary

Eiffel agents provide first-class functions supporting delayed execution, callbacks, and functional programming. The hierarchy: ROUTINE → PROCEDURE (no return) / FUNCTION (with return) / PREDICATE (boolean return).

---

## 1. Agent Class Hierarchy

```
ROUTINE [OPEN_ARGS -> TUPLE]
    |
    +-- PROCEDURE [OPEN_ARGS]      -- No return value
    +-- FUNCTION [OPEN_ARGS, RESULT_TYPE]  -- Returns value
            |
            +-- PREDICATE [OPEN_ARGS]  -- Returns BOOLEAN
```

**Files:**
- `library/base/elks/kernel/routine.e` (511 lines)
- `library/base/elks/kernel/procedure.e` (59 lines)
- `library/base/elks/kernel/function.e` (130 lines)
- `library/base/elks/kernel/predicate.e` (26 lines)

---

## 2. Agent Creation Patterns

### Basic Agent to Feature

```eiffel
action: PROCEDURE [STRING]
action := agent my_object.my_procedure
```

### Type-Qualified (Unbound) Agent

```eiffel
action: PROCEDURE [MY_CLASS, STRING]
action := agent {MY_CLASS}.my_procedure

-- Call with dynamic target
action.call ([obj, "argument"])
```

### Inline Agents (Lambdas)

```eiffel
action := agent (x: STRING) do
    io.put_string (x)
end

converter := agent (s: STRING): INTEGER do
    Result := s.count
end
```

### Partial Application with Placeholders

```eiffel
-- Original: send_card (age: INTEGER; name, from: STRING)
birthday_actions.extend (agent send_card (?, ?, "Sam"))
-- Creates PROCEDURE [TUPLE [INTEGER, STRING]] with "Sam" pre-filled
```

---

## 3. Higher-Order Functions

### do_all - Apply to Every Element

```eiffel
my_list.do_all (agent io.put_string)

-- With index
numbers.do_all_with_index (agent (val: INTEGER; idx: INTEGER) do
    io.put_string ("Item at " + idx.out + ": " + val.out + "%N")
end)
```

### do_if - Conditional Application

```eiffel
numbers.do_if (
    agent (x: INTEGER) do io.put_int (x) end,
    agent (x: INTEGER): BOOLEAN do Result := x > 10 end
)
```

### there_exists / for_all - Quantifiers

```eiffel
has_large := numbers.there_exists (agent (x: INTEGER): BOOLEAN do
    Result := x > 1000
end)

all_positive := numbers.for_all (agent (x: INTEGER): BOOLEAN do
    Result := x > 0
end)
```

**File:** `library/base/elks/structures/list/arrayed_list.e` (lines 224-285)

---

## 4. Callback Patterns

### ACTION_SEQUENCE (Observer)

```eiffel
birthday_actions: ACTION_SEQUENCE [TUPLE [INTEGER, STRING]]

-- Register handlers
birthday_actions.extend (agent send_card (?, ?, "Sam"))
birthday_actions.extend (agent buy_gift (?, ?, "Wine", "Sam"))

-- Invoke all
birthday_actions.call ([35, "Julia"])
```

### Observable Pattern

```eiffel
class OBSERVABLE [G]
feature
    subscribe (a_handler: PROCEDURE [G])
        do observers.extend (a_handler) end

    notify (a_data: G)
        do
            across observers as obs loop
                obs.item.call ([a_data])
            end
        end
end
```

---

## 5. Invocation

### PROCEDURE - call

```eiffel
action: PROCEDURE [TUPLE [INTEGER, STRING]]
action.call ([42, "hello"])

-- No args
action.call (Void)
```

### FUNCTION - item

```eiffel
func: FUNCTION [TUPLE [INTEGER, INTEGER], INTEGER]
result := func.item ([5, 3])

-- Or via call (stores in last_result)
func.call ([5, 3])
io.put_int (func.last_result)
```

---

## 6. Advanced Patterns

### Agent Composition

```eiffel
my_list.do_all (agent (s: STRING) do
    printer.call ([converter.item ([s])])
end)
```

### Currying

```eiffel
add: FUNCTION [TUPLE [INTEGER, INTEGER], INTEGER]
add := agent (a, b: INTEGER): INTEGER do Result := a + b end

add_five := agent (x: INTEGER) do
    io.put_int (add.item ([5, x]))
end
```

### Factory with Agents

```eiffel
create_filter (threshold: INTEGER): FUNCTION [INTEGER, BOOLEAN]
    do
        Result := agent (x: INTEGER): BOOLEAN do
            Result := x > threshold
        end
    end
```

---

## 7. FAQ Pairs to Generate

1. What are agents in Eiffel?
2. What is the difference between PROCEDURE and FUNCTION?
3. How do I create an inline agent (lambda)?
4. How do I use partial application with placeholders?
5. How do I apply a function to all list elements?
6. How do I filter elements with an agent?
7. How do I use agents for callbacks?
8. What is the difference between call and item?

---

## References

- `library/base/elks/kernel/routine.e` - Base agent class
- `library/base/elks/kernel/procedure.e` - No-return agents
- `library/base/elks/kernel/function.e` - Return-value agents
- `library/base/elks/structures/list/arrayed_list.e` - Higher-order functions
- `library/base/ise/event/action_sequence.e` - Event handling
