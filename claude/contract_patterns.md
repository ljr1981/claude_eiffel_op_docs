# Contract Patterns: Systematic Postcondition Templates

**Purpose:** Provide complete postcondition patterns for common operations to avoid the "true but incomplete" problem identified by Meyer.

---

## The Completeness Principle

Every postcondition should answer:
1. **What changed?** (the direct effect)
2. **How did it change?** (the relationship to old state)
3. **What did NOT change?** (frame conditions)

---

## Collection Operations

### Add Item (extend, put, add)

```eiffel
add_item (a_item: G)
    require
      item_valid: a_item /= Void  -- if not detachable
    do
      items.extend (a_item)
    ensure
      -- What changed
      has_item: items.has (a_item)
      -- How it changed
      count_increased: items.count = old items.count + 1
      -- Position (if ordered)
      at_end: items.last ~ a_item
      -- Frame condition (what didn't change)
      others_preserved: across 1 |..| (old items.count) as i all
                          items[i] ~ (old items.twin)[i]
                        end
    end
```

### Remove Item (remove, prune)

```eiffel
remove_item (a_item: G)
    require
      has_item: items.has (a_item)
    do
      items.prune (a_item)
    ensure
      -- What changed
      removed: not items.has (a_item)
      -- How it changed
      count_decreased: items.count = old items.count - 1
      -- Frame condition
      others_still_present: across old items.twin as ic all
                              ic /~ a_item implies items.has (ic)
                            end
    end
```

### Clear Collection

```eiffel
clear
    do
      items.wipe_out
    ensure
      -- What changed
      is_empty: items.is_empty
      -- Explicit count
      count_zero: items.count = 0
    end
```

### Replace Item

```eiffel
replace (a_old, a_new: G)
    require
      has_old: items.has (a_old)
    do
      -- implementation
    ensure
      -- What changed
      has_new: items.has (a_new)
      no_old: not items.has (a_old)
      -- What didn't change
      same_count: items.count = old items.count
    end
```

---

## Search/Query Operations

### Find Item (returns item or Void)

```eiffel
find (a_key: K): detachable V
    do
      -- implementation
    ensure
      -- Found case
      found_exists: Result /= Void implies has_key (a_key)
      found_correct: attached Result as r implies item (a_key) ~ r
      -- Not found case
      not_found_means_absent: Result = Void implies not has_key (a_key)
    end
```

### Find Index (returns index or 0/-1)

```eiffel
index_of (a_item: G): INTEGER
    do
      -- implementation
    ensure
      -- Found case
      found_valid: Result > 0 implies valid_index (Result)
      found_correct: Result > 0 implies items[Result] ~ a_item
      -- Not found case
      not_found: Result <= 0 implies not items.has (a_item)
      -- No state change (query)
      unchanged: items.count = old items.count
    end
```

### Contains/Has (boolean query)

```eiffel
has (a_item: G): BOOLEAN
    do
      -- implementation
    ensure
      -- Definition
      definition: Result = across items as ic some ic ~ a_item end
      -- No state change (query)
      unchanged: items.count = old items.count
    end
```

---

## Attribute Setters

### Simple Setter

```eiffel
set_name (a_name: STRING)
    require
      valid_name: not a_name.is_empty
    do
      name := a_name
    ensure
      -- What changed
      name_set: name ~ a_name
      -- What didn't change (frame conditions)
      age_unchanged: age = old age
      id_unchanged: id = old id
    end
```

### Setter with Side Effects

```eiffel
set_active (a_value: BOOLEAN)
    do
      is_active := a_value
      if a_value then
        activation_time := current_time
      end
    ensure
      -- Primary effect
      active_set: is_active = a_value
      -- Side effect (conditional)
      time_set_if_activated: a_value implies activation_time = current_time
      time_unchanged_if_deactivated: not a_value implies activation_time = old activation_time
    end
```

---

## State Machine Operations

### State Transitions

```eiffel
start
    require
      can_start: state = State_idle
    do
      state := State_running
    ensure
      -- New state
      now_running: state = State_running
      -- Valid transition occurred
      was_idle: old state = State_idle
    end

stop
    require
      can_stop: state = State_running
    do
      state := State_stopped
    ensure
      now_stopped: state = State_stopped
      was_running: old state = State_running
    end
```

### Class Invariant for State Machines

```eiffel
invariant
  valid_state: state = State_idle or state = State_running or state = State_stopped
  -- Derived state consistency
  running_implies_has_start_time: state = State_running implies start_time /= Void
```

---

## Resource Management

### Open/Acquire

```eiffel
open (a_path: STRING)
    require
      not_already_open: not is_open
      valid_path: not a_path.is_empty
    do
      -- implementation
      is_open := True
    ensure
      now_open: is_open
      path_set: path ~ a_path
    end
```

### Close/Release

```eiffel
close
    require
      is_open: is_open
    do
      -- implementation
      is_open := False
    ensure
      now_closed: not is_open
      -- Resource released (if trackable)
      handle_released: handle = Void or handle = default_handle
    end
```

---

## Numeric Operations

### Increment/Decrement

```eiffel
increment
    require
      not_at_max: count < max_count
    do
      count := count + 1
    ensure
      increased: count = old count + 1
      still_valid: count <= max_count
    end

decrement
    require
      not_at_min: count > min_count
    do
      count := count - 1
    ensure
      decreased: count = old count - 1
      still_valid: count >= min_count
    end
```

### Bounded Operations

```eiffel
add_value (a_value: INTEGER)
    require
      non_negative: a_value >= 0
      room_available: count + a_value <= capacity
    do
      count := count + a_value
    ensure
      added: count = old count + a_value
      within_bounds: count <= capacity
    end
```

---

## String Operations

### Append

```eiffel
append (a_suffix: STRING)
    require
      suffix_not_void: a_suffix /= Void
    do
      content.append (a_suffix)
    ensure
      -- Length changed correctly
      length_increased: content.count = old content.count + a_suffix.count
      -- Original content preserved
      prefix_preserved: content.substring (1, old content.count) ~ old content.twin
      -- Suffix added
      suffix_present: content.substring (old content.count + 1, content.count) ~ a_suffix
    end
```

### Prepend

```eiffel
prepend (a_prefix: STRING)
    require
      prefix_not_void: a_prefix /= Void
    do
      content.prepend (a_prefix)
    ensure
      length_increased: content.count = old content.count + a_prefix.count
      prefix_present: content.substring (1, a_prefix.count) ~ a_prefix
      original_follows: content.substring (a_prefix.count + 1, content.count) ~ old content.twin
    end
```

---

## Factory/Creation Patterns

### Factory Method

```eiffel
new_widget (a_name: STRING): WIDGET
    require
      valid_name: not a_name.is_empty
    do
      create Result.make (a_name)
    ensure
      -- Result exists
      result_created: Result /= Void
      -- Result properly initialized
      result_named: Result.name ~ a_name
      result_valid: Result.is_valid
      -- No side effects on factory
      count_unchanged: widget_count = old widget_count  -- unless factory tracks
    end
```

### Creation with Registration

```eiffel
create_and_register (a_name: STRING): WIDGET
    require
      valid_name: not a_name.is_empty
      not_duplicate: not registry.has_key (a_name)
    do
      create Result.make (a_name)
      registry.put (Result, a_name)
    ensure
      result_created: Result /= Void
      result_named: Result.name ~ a_name
      registered: registry.has_key (a_name)
      registry_grown: registry.count = old registry.count + 1
    end
```

---

## Loop Invariants (for AutoProof)

### Linear Search

```eiffel
from
  i := 1
  Result := 0
invariant
  -- Progress bound
  valid_index: i >= 1 and i <= items.count + 1
  -- Partial result
  not_yet_found: Result = 0 implies
    across 1 |..| (i - 1) as j all items[j] /~ target end
  -- Found result valid
  found_valid: Result > 0 implies items[Result] ~ target
until
  i > items.count or Result > 0
loop
  if items[i] ~ target then
    Result := i
  end
  i := i + 1
variant
  items.count - i + 1
end
```

### Accumulation Loop

```eiffel
from
  i := items.lower
  Result := 0
invariant
  valid_index: i >= items.lower and i <= items.upper + 1
  -- Partial sum
  partial_sum: Result = sum_of (items.subarray (items.lower, i - 1))
until
  i > items.upper
loop
  Result := Result + items[i]
  i := i + 1
variant
  items.upper - i + 1
end
```

---

## Frame Condition Helpers

When specifying "what didn't change," use these patterns:

### For Objects with Few Attributes

```eiffel
ensure
  -- Explicit frame
  a_unchanged: a = old a
  b_unchanged: b = old b
  -- Only c changed
  c_set: c = new_value
```

### For Collections

```eiffel
ensure
  -- Items not matching condition unchanged
  others_unchanged: across items as ic all
                      not condition (ic) implies ic ~ (old items.twin)[items.index_of (ic, 1)]
                    end
```

### For Complex Objects

Consider adding a `same_except_X` helper:

```eiffel
same_except_name (a_old: like Current): BOOLEAN
    do
      Result := age = a_old.age and id = a_old.id and status = a_old.status
    end

-- Then in postcondition:
ensure
  name_changed: name ~ a_new_name
  rest_unchanged: same_except_name (old twin)
```

---

## Update Log

| Date | Change |
|------|--------|
| 2025-12-03 | Initial creation based on Meyer's completeness concerns |
