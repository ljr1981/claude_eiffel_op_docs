# Study 4: Observer/Event Patterns (ACTION_SEQUENCE) in Canonical Eiffel

**Date:** 2025-12-25
**Sources:**
- ISE EiffelStudio 25.02 Standard Library
- Patterns Library
- Vision2 GUI Library

---

## Executive Summary

Eiffel implements the Observer pattern through two complementary mechanisms:

1. **Legacy Observer (EiffelBase):** `ACTION_SEQUENCE[G -> TUPLE]` - Direct agent-based implementation
2. **Modern Observer (Patterns Library):** `EVENT_TYPE[EVENT_DATA -> TUPLE]` - Formalized, componentized pattern

Both use **agents** as the core subscription mechanism, eliminating explicit observer interfaces.

---

## Key Classes

| Class | Location | Role |
|-------|----------|------|
| `ACTION_SEQUENCE [G]` | `base/ise/event/action_sequence.e` | Low-level event container |
| `EVENT_TYPE [G]` | `patterns/event/event_type.e` | Modern formalized events |
| `INTERACTIVE_LIST [G]` | `base/ise/event/interactive_list.e` | Base with add/remove callbacks |
| `ACTIVE_LIST [G]` | `base/ise/event/active_list.e` | List with action sequences |

---

## ACTION_SEQUENCE Architecture

```eiffel
class ACTION_SEQUENCE [EVENT_DATA -> detachable TUPLE create default_create end]
inherit
    INTERACTIVE_LIST [PROCEDURE [EVENT_DATA]]
```

### Core Feature: call

```eiffel
call (event_data: detachable EVENT_DATA)
    -- Call each procedure in order unless `is_blocked'.
    -- If `is_paused' delay execution until `resume'.
    -- Stop at current point in list on `abort'.
```

### State Management

```
Normal --[pause]--> Paused --[resume]--> Normal
Normal --[block]--> Blocked --[resume]--> Normal
```

---

## Usage Patterns

### Pattern 1: Simple Event

```eiffel
feature
    button_click_actions: ACTION_SEQUENCE [TUPLE]

-- Subscribe
button_click_actions.extend (agent on_button_clicked)

-- Notify
button_click_actions.call ([])

-- Unsubscribe
button_click_actions.prune (agent on_button_clicked)
```

### Pattern 2: Typed Event Data

```eiffel
value_changed_actions: ACTION_SEQUENCE [TUPLE [old_value, new_value: INTEGER]]

-- Subscribe with partial application
value_changed_actions.extend (agent on_value_changed (?, ?))
value_changed_actions.extend (agent log_change (?, ?, "system"))

-- Notify
value_changed_actions.call ([old_val, new_val])
```

### Pattern 3: Kamikaze (One-Time)

```eiffel
-- Subscribe for single notification
init_complete_actions.extend_kamikaze (agent on_first_completion)
```

### Pattern 4: State Control

```eiffel
-- Pause (buffer events)
event_sequence.pause
perform_critical_operation
event_sequence.resume  -- Executes buffered events

-- Block (discard events)
event_sequence.block
cleanup
event_sequence.resume
```

---

## EVENT_TYPE (Modern Pattern)

```eiffel
frozen class EVENT_TYPE [EVENT_DATA -> TUPLE]
inherit
    EVENT_TYPE_I [EVENT_DATA]
    EVENT_TYPE_PUBLISHER_I [EVENT_DATA]

feature -- Subscription
    subscribe (a_action: PROCEDURE [EVENT_DATA])
    unsubscribe (a_action: PROCEDURE [EVENT_DATA])
    subscribe_for_single_notification (a_action: PROCEDURE [EVENT_DATA])

feature -- Publication
    publish (a_args: EVENT_DATA)
    publish_if (a_args: EVENT_DATA; a_predicate: PREDICATE [EVENT_DATA])

feature -- Suspension
    suspend_subscriptions
    restore_subscriptions
```

### Usage

```eiffel
-- Declaration
temperature_changed: EVENT_TYPE [TUPLE [value: INTEGER]]

-- Subscribe
temperature_changed.subscribe (agent window.on_temperature_changed (?))

-- Publish
temperature_changed.publish ([72])

-- Conditional publish
temperature_changed.publish_if ([new_temp],
    agent (t: TUPLE [v: INTEGER]): BOOLEAN do Result := t.v > 80 end)
```

---

## Eiffel-Specific Advantages

### 1. Agents Over Interfaces

**Java requires:**
```java
interface ButtonListener { void onButtonClicked(); }
class Handler implements ButtonListener { ... }
button.addListener(new Handler());
```

**Eiffel:**
```eiffel
button.select_actions.extend (agent on_button_clicked)
-- No intermediate class needed
```

### 2. Type-Safe Tuples

```eiffel
mouse_events: ACTION_SEQUENCE [TUPLE [x, y: INTEGER; button: CHARACTER]]
-- Compiler enforces correct structure
```

### 3. Partial Application

```eiffel
-- Bind specific arguments
data_events.extend (agent on_data (?, handler_id))
data_events.extend (agent on_data (?, other_handler_id))
```

### 4. Snapshot-Based Iteration

```eiffel
-- Safe during modification
call (event_data)
    -- Internally creates snapshot
    -- Modifications during iteration don't break loop
```

---

## Vision2 Integration

```eiffel
-- Button click
button.select_actions.extend (agent on_button_clicked)

-- Window close
window.close_request_actions.extend (agent request_close_window)

-- Grid selection
grid.row_select_actions.extend (agent on_row_selected (?))
```

---

## When to Use Which

| Scenario | Use |
|----------|-----|
| Simple events | ACTION_SEQUENCE |
| GUI events | Vision2 EV_ACTION_SEQUENCE |
| Formal observer hierarchy | EVENT_TYPE |
| Need pause/block | ACTION_SEQUENCE |
| Need suspension semantics | EVENT_TYPE |

---

## Arnout Classification

**Observer Pattern Status:** Fully Componentized

Karine Arnout's dissertation identifies Observer as one of the ~65% of GoF patterns that can be fully componentized in Eiffel. The Event Library (ACTION_SEQUENCE, EVENT_TYPE) is the canonical implementation.

**Key Innovation:** Agents eliminate the need for explicit Observer interfaces, reducing boilerplate while maintaining type safety.

---

## FAQ Pairs to Generate

1. Q: How do I implement the Observer pattern in Eiffel?
2. Q: What is ACTION_SEQUENCE and how do I use it?
3. Q: How do I subscribe to button clicks in Vision2?
4. Q: What's the difference between ACTION_SEQUENCE and EVENT_TYPE?
5. Q: How do I pause event notifications temporarily?
6. Q: How do I create a one-time event subscription?
7. Q: How do I pass data with events in Eiffel?
8. Q: How do I unsubscribe from an event?

---

## Canonical Code References

| Pattern | File | Path |
|---------|------|------|
| ACTION_SEQUENCE | `action_sequence.e` | `base/ise/event/` |
| EVENT_TYPE | `event_type.e` | `patterns/event/` |
| INTERACTIVE_LIST | `interactive_list.e` | `base/ise/event/` |
| Vision2 events | `ev_action_sequence.e` | `vision2/interface/events/` |
| Sensor example | `sensor.e` | `patterns/event/examples/src/` |
