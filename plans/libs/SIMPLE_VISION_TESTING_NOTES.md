# simple_vision GUI Testing - Design Notes

**Date:** December 28, 2025
**Purpose:** Auxiliary document to capture design decisions and considerations for the GUI testing harness (Innovation 14) without polluting the main implementation or innovation documents.

---

## Widget Identification Strategy

### The Problem
Every GUI widget reachable through the visual interface needs a stable, unique identifier that can be:
1. Set at design time
2. Referenced in JSON use-case scripts
3. Logged in test results for correlation

### Solution: Prefix + Suffix Pattern

```
{type_prefix}_{instance_suffix}
```

**Examples:**
- `button_submit`
- `button_cancel`
- `listitem_1`, `listitem_2`, `listitem_3`
- `field_username`
- `label_error`

**Benefits:**
- Type prefix enables pattern matching (`button_*` matches all buttons)
- Instance suffix differentiates multiple instances of same type
- Stable across sessions (design-time assigned)
- Human-readable in logs and scripts

### Implementation in SV_WIDGET

```eiffel
feature -- Identification

    widget_id: STRING
            -- Stable identifier for testing/automation.
            -- Format: "{type}_{name}" e.g., "button_submit"

    set_widget_id (a_id: STRING): like Current
            -- Set widget identifier (fluent).
        require
            id_not_empty: not a_id.is_empty
        do
            widget_id := a_id
            Result := Current
        end

    id (a_id: STRING): like Current
            -- Fluent alias for set_widget_id.
        do
            Result := set_widget_id (a_id)
        end
```

### Usage in SV_QUICK

```eiffel
-- Widgets get auto-generated IDs or explicit ones
quick.button ("Submit").id ("button_submit")
quick.text ("Error message").id ("label_error")

-- For lists/grids, use index suffixes
across products as p loop
    quick.row.id ("row_" + p.cursor_index.out).children (<<...>>)
end
```

---

## Cross-Tier ID Consistency

### Tier 1 (EV2 Widgets)
Widget IDs stored as Eiffel STRING attributes on SV_WIDGET.

### Tier 2 (Cairo/C Panels)
Widget IDs passed through to C-side if needed for debugging/logging.

### Tier 3 (webview/HTMX/Alpine.js)
Widget IDs become HTML `id` or `data-test-id` attributes:

```html
<button id="button_submit" data-test-id="button_submit">Submit</button>
```

HTMX/Alpine.js can use these for event handling:
```html
<div x-data @click="handleClick($event.target.dataset.testId)">
```

---

## Production Binary Considerations

### The Concern
Testing artifacts (instrumentation hooks, widget IDs, logging) should NOT pollute production binaries unless explicitly requested.

### Solution: Conditional Compilation

**Option 1: Debug Assertions**
Wrap testing features in `debug ("testing")`:

```eiffel
feature {SV_TEST_HARNESS} -- Instrumentation

    test_harness: detachable SV_TEST_HARNESS
            -- Only exists in debug/testing builds.

    attach_harness (a_harness: SV_TEST_HARNESS)
        debug ("testing")
            test_harness := a_harness
        end
```

**Option 2: ECF Target Differentiation**
- `lib_release` target: No testing cluster, dead code removed
- `lib_tests` target: Includes testing, assertions enabled

**Option 3: Compiler Flag (like -keep)**
Similar to `-keep` for contracts, a `-keep-testing` flag could preserve testing infrastructure in production for:
- Field debugging
- Customer site automated testing
- Continuous production validation

### Recommendation
Use **Option 2** (ECF targets) for most cases:
- Default production builds strip testing code via dead code removal
- Explicit test builds include everything
- No runtime overhead in production

For special cases (field testing), provide an `instrumented` target that includes hooks but with minimal overhead when harness not attached.

---

## Test Log ↔ Use-Case Correlation

### Requirement
The test execution log MUST correlate step-by-step with the JSON use-case file:
- Each step from JSON logged when executed
- Expected vs actual results recorded
- Pass/fail for each step
- Timestamps for timing analysis

### Log Format

```
[2025-12-28 21:06:17.604] STEP 1: click -> button_submit
[2025-12-28 21:06:17.650] STEP 2: assert_text -> label_status = "Submitted"
  EXPECTED: "Submitted"
  ACTUAL:   "Submitted"
  RESULT:   PASS
[2025-12-28 21:06:17.651] STEP 3: wait -> 500ms
```

### Structured Log (JSON)
For machine processing, also output structured logs:

```json
{
  "step": 2,
  "action": "assert_text",
  "target": "label_status",
  "expected": "Submitted",
  "actual": "Submitted",
  "passed": true,
  "timestamp": "2025-12-28T21:06:17.650Z",
  "duration_ms": 1
}
```

### simple_logger Integration
The harness should use `simple_logger` for all output:
- Console output for immediate feedback
- File output for persistence
- Structured JSON for CI/CD integration

---

## GUI Event Simulation Requirements

### The Full Range of User Interactions

The GUI test harness must be capable of simulating **all** user-driven events that interact with the application. Here is the complete taxonomy:

### 1. Mouse Events

| Event | Description | Harness Method |
|-------|-------------|----------------|
| **Left Click** | Primary button click | `simulate_click` |
| **Right Click** | Context menu trigger | `simulate_right_click` |
| **Double Click** | Open/activate action | `simulate_double_click` |
| **Mouse Down** | Press without release | `simulate_mouse_down` |
| **Mouse Up** | Release press | `simulate_mouse_up` |
| **Mouse Move** | Hover/cursor position | `simulate_mouse_move (x, y)` |
| **Mouse Drag** | Down + move + up sequence | `simulate_drag (start_x, start_y, end_x, end_y)` |
| **Scroll Wheel** | Vertical scrolling | `simulate_scroll (delta)` |
| **Horizontal Scroll** | Horizontal scrolling | `simulate_scroll_horizontal (delta)` |

### 2. Keyboard Events

| Event | Description | Harness Method |
|-------|-------------|----------------|
| **Key Press** | Single key down+up | `simulate_key (key_code)` |
| **Key Down** | Key pressed | `simulate_key_down (key_code)` |
| **Key Up** | Key released | `simulate_key_up (key_code)` |
| **Character Input** | Text typing | `simulate_type (text)` |
| **Key Combo** | Ctrl/Alt/Shift + key | `simulate_key_combo (modifiers, key)` |
| **Enter** | Submit/confirm | `simulate_enter` |
| **Tab** | Focus navigation | `simulate_tab` |
| **Escape** | Cancel/close | `simulate_escape` |
| **Arrow Keys** | Navigation | `simulate_arrow (direction)` |
| **F-Keys** | Function keys | `simulate_fkey (number)` |

### 3. Focus Events

| Event | Description | Harness Method |
|-------|-------------|----------------|
| **Focus** | Widget gains focus | `simulate_focus` |
| **Blur** | Widget loses focus | `simulate_blur` |
| **Tab Order** | Sequential focus | `simulate_tab_sequence` |

### 4. Widget-Specific Events

| Widget | Event | Harness Method |
|--------|-------|----------------|
| **Checkbox** | Check/uncheck | `simulate_check`, `simulate_uncheck` |
| **Radio** | Select option | `simulate_select (index)` |
| **Dropdown** | Select item | `simulate_select_item (value)` |
| **List** | Select item(s) | `simulate_list_select (indices)` |
| **Slider** | Set value | `simulate_set_value (value)` |
| **Tree** | Expand/collapse | `simulate_expand`, `simulate_collapse` |
| **Tab Panel** | Switch tab | `simulate_tab_select (index)` |
| **Menu** | Open/select | `simulate_menu_select (path)` |
| **Text Field** | Type text | `simulate_input (text)` |
| **Grid** | Cell click/select | `simulate_cell_click (row, col)` |

### 5. Window Events

| Event | Description | Harness Method |
|-------|-------------|----------------|
| **Close** | Window close request | `simulate_close` |
| **Minimize** | Minimize to taskbar | `simulate_minimize` |
| **Maximize** | Maximize window | `simulate_maximize` |
| **Restore** | Restore from min/max | `simulate_restore` |
| **Resize** | Change window size | `simulate_resize (w, h)` |
| **Move** | Change window position | `simulate_move (x, y)` |

### 6. Drag and Drop Events

| Event | Description | Harness Method |
|-------|-------------|----------------|
| **Drag Start** | Begin drag operation | `simulate_drag_start` |
| **Drag Over** | Dragging over target | `simulate_drag_over (target)` |
| **Drop** | Complete drop | `simulate_drop (target)` |
| **Drag Cancel** | Cancel drag | `simulate_drag_cancel` |

### 7. Clipboard Events

| Event | Description | Harness Method |
|-------|-------------|----------------|
| **Copy** | Ctrl+C | `simulate_copy` |
| **Cut** | Ctrl+X | `simulate_cut` |
| **Paste** | Ctrl+V | `simulate_paste` |
| **Select All** | Ctrl+A | `simulate_select_all` |

### 8. Touch Events (Mobile/Tablet)

| Event | Description | Harness Method |
|-------|-------------|----------------|
| **Tap** | Single touch | `simulate_tap (x, y)` |
| **Long Press** | Press and hold | `simulate_long_press (x, y)` |
| **Swipe** | Directional swipe | `simulate_swipe (direction)` |
| **Pinch** | Zoom in/out | `simulate_pinch (scale)` |
| **Multi-Touch** | Multiple fingers | `simulate_multi_touch (points)` |

### Current Implementation Status

**✅ Implemented:**
- `simulate_click` - Left mouse click
- `simulate_text` assertion - Text content verification

**⬜ To Be Implemented (Priority Order):**
1. `simulate_type` - Text input simulation
2. `simulate_key` - Single key press
3. `simulate_key_combo` - Modifier + key combinations
4. `simulate_double_click` - Double-click actions
5. `simulate_right_click` - Context menus
6. `simulate_drag` - Drag operations
7. `simulate_scroll` - Scroll wheel
8. `simulate_focus` / `simulate_blur` - Focus management
9. Window events (close, resize, etc.)
10. Widget-specific events (expand tree, select dropdown, etc.)

### Implementation Notes

1. **EV2 Event Injection**: Most events can be simulated by calling the widget's action sequences directly (e.g., `button.select_actions.call ([])`)

2. **Low-Level Simulation via simple_* Libraries**: For true OS-level event injection, use the existing simple_* ecosystem:
   - `simple_process` - Process management and inter-process communication
   - `simple_win32` - Direct Win32 API access for SendInput, PostMessage, etc.
   - `simple_memory` - Memory operations if needed for shared state
   - These libraries encapsulate platform-specific code in Eiffel with proper contracts

3. **Timing Considerations**: Some events require delays between steps (e.g., double-click timing window)

4. **State Verification**: After simulating events, always verify widget state changed as expected

---

## Formal Specifications

### Use-Case JSON Schema

The use-case JSON format is the **single source of truth** for test scenarios. Each use-case file MUST conform to this schema:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "SV_TEST_USE_CASE",
  "type": "object",
  "required": ["name", "version", "steps"],
  "properties": {
    "name": {
      "type": "string",
      "description": "Unique identifier for this use-case",
      "pattern": "^[a-z][a-z0-9_]*$"
    },
    "version": {
      "type": "string",
      "description": "Semantic version of use-case format",
      "default": "1.0.0"
    },
    "description": {
      "type": "string",
      "description": "Human-readable description of what this test validates"
    },
    "tags": {
      "type": "array",
      "items": {"type": "string"},
      "description": "Categories for filtering: regression, smoke, integration, etc."
    },
    "setup": {
      "type": "object",
      "properties": {
        "window": {"type": "string", "description": "Target window/demo class name"},
        "preconditions": {"type": "array", "items": {"$ref": "#/definitions/step"}},
        "initial_state": {"type": "object", "description": "Widget states to verify before test"}
      }
    },
    "steps": {
      "type": "array",
      "items": {"$ref": "#/definitions/step"},
      "minItems": 1
    },
    "teardown": {
      "type": "object",
      "properties": {
        "actions": {"type": "array", "items": {"$ref": "#/definitions/step"}},
        "cleanup": {"type": "string", "enum": ["close_window", "reset_state", "none"]}
      }
    }
  },
  "definitions": {
    "step": {
      "type": "object",
      "required": ["step", "action"],
      "properties": {
        "step": {"type": "integer", "minimum": 1},
        "action": {"type": "string", "enum": [
          "click", "double_click", "right_click",
          "type", "clear", "key", "key_combo",
          "focus", "blur", "tab",
          "check", "uncheck", "toggle",
          "select", "select_item", "select_cell",
          "drag", "scroll",
          "wait", "wait_for",
          "assert_text", "assert_enabled", "assert_disabled",
          "assert_visible", "assert_hidden", "assert_checked",
          "assert_value", "assert_count", "assert_selected"
        ]},
        "target": {"type": "string", "description": "Widget ID or path"},
        "value": {"type": ["string", "number", "boolean", "array"]},
        "expected": {"type": ["string", "number", "boolean"]},
        "timeout_ms": {"type": "integer", "default": 5000},
        "comment": {"type": "string"}
      }
    }
  }
}
```

### GUI Test Log Schema

The log output MUST mirror the use-case structure for bidirectional correlation:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "SV_TEST_LOG",
  "type": "object",
  "required": ["use_case", "version", "started_at", "steps", "result"],
  "properties": {
    "use_case": {"type": "string", "description": "Name from use-case file"},
    "use_case_file": {"type": "string", "description": "Path to source JSON"},
    "version": {"type": "string", "description": "Log format version"},
    "started_at": {"type": "string", "format": "date-time"},
    "ended_at": {"type": "string", "format": "date-time"},
    "duration_ms": {"type": "integer"},
    "harness_version": {"type": "string"},
    "environment": {
      "type": "object",
      "properties": {
        "platform": {"type": "string"},
        "eiffel_version": {"type": "string"},
        "ev2_backend": {"type": "string", "enum": ["win32", "gtk3", "cocoa"]}
      }
    },
    "setup_log": {
      "type": "object",
      "properties": {
        "status": {"type": "string", "enum": ["ok", "failed", "skipped"]},
        "timestamp": {"type": "string", "format": "date-time"},
        "message": {"type": "string"}
      }
    },
    "steps": {
      "type": "array",
      "items": {"$ref": "#/definitions/step_log"}
    },
    "teardown_log": {
      "type": "object",
      "properties": {
        "status": {"type": "string", "enum": ["ok", "failed", "skipped"]},
        "timestamp": {"type": "string", "format": "date-time"}
      }
    },
    "result": {
      "type": "string",
      "enum": ["passed", "failed", "error", "skipped"]
    },
    "summary": {
      "type": "object",
      "properties": {
        "total_steps": {"type": "integer"},
        "passed_steps": {"type": "integer"},
        "failed_steps": {"type": "integer"},
        "skipped_steps": {"type": "integer"}
      }
    },
    "failure_info": {
      "type": "object",
      "properties": {
        "first_failure_step": {"type": "integer"},
        "failure_message": {"type": "string"},
        "stack_trace": {"type": "string"}
      }
    }
  },
  "definitions": {
    "step_log": {
      "type": "object",
      "required": ["step", "action", "status", "timestamp"],
      "properties": {
        "step": {"type": "integer", "description": "MUST match use-case step number"},
        "action": {"type": "string", "description": "MUST match use-case action"},
        "target": {"type": "string", "description": "MUST match use-case target"},
        "value": {"type": ["string", "number", "boolean", "array"]},
        "expected": {"type": ["string", "number", "boolean", "null"]},
        "actual": {"type": ["string", "number", "boolean", "null"]},
        "status": {"type": "string", "enum": ["ok", "pass", "fail", "error", "skipped"]},
        "timestamp": {"type": "string", "format": "date-time"},
        "duration_ms": {"type": "integer"},
        "message": {"type": "string", "description": "Additional context on failure"}
      }
    }
  }
}
```

### Correlation Rules

The test harness MUST enforce these rules to ensure use-case ↔ log correspondence:

| Use-Case Field | Log Field | Rule |
|----------------|-----------|------|
| `name` | `use_case` | Exact string match |
| `steps[n].step` | `steps[n].step` | Same integer value |
| `steps[n].action` | `steps[n].action` | Same action name |
| `steps[n].target` | `steps[n].target` | Same widget ID |
| `steps[n].value` | `steps[n].value` | Same value passed |
| `steps[n].expected` | `steps[n].expected` | Same expected value |
| (N/A) | `steps[n].actual` | Runtime-captured value |
| (N/A) | `steps[n].status` | Pass/fail determination |

### Action Composition: High-Level → Low-Level

A **high-level user action** (semantic intent) decomposes into **low-level GUI primitives** (physical interactions):

```
HIGH-LEVEL ACTION          LOW-LEVEL PRIMITIVES
─────────────────          ────────────────────
"login"             →      focus(field_username)
                           type("testuser")
                           focus(field_password)
                           type("secret123")
                           click(button_login)

"search_for"        →      click(field_search)
                           clear()
                           type("query text")
                           click(button_search)

"select_row"        →      scroll_to_row(row_index)
                           click(cell at row_index, col 0)

"drag_to_folder"    →      mouse_down(item)
                           mouse_move(target_folder)
                           mouse_up()
```

#### Use-Case JSON Supports Both Levels

**Atomic (low-level):** Each step is one GUI primitive
```json
{"step": 1, "action": "focus", "target": "field_username"},
{"step": 2, "action": "type", "target": "field_username", "value": "testuser"},
{"step": 3, "action": "focus", "target": "field_password"},
{"step": 4, "action": "type", "target": "field_password", "value": "secret"},
{"step": 5, "action": "click", "target": "button_login"}
```

**Composite (high-level):** One step expands to multiple primitives
```json
{
  "step": 1,
  "action": "composite",
  "name": "login_as",
  "params": {"username": "testuser", "password": "secret"},
  "expands_to": [
    {"action": "focus", "target": "field_username"},
    {"action": "type", "target": "field_username", "value": "${username}"},
    {"action": "focus", "target": "field_password"},
    {"action": "type", "target": "field_password", "value": "${password}"},
    {"action": "click", "target": "button_login"}
  ]
}
```

#### Log Output Shows Expansion

When a composite action executes, the log shows both levels:
```
[21:10:00.100] STEP 1: composite -> login_as {username: "testuser", password: "****"}
  [21:10:00.102]   1.1: focus -> field_username       OK
  [21:10:00.105]   1.2: type -> field_username        OK
  [21:10:00.150]   1.3: focus -> field_password       OK
  [21:10:00.155]   1.4: type -> field_password        OK
  [21:10:00.200]   1.5: click -> button_login         OK
  RESULT: OK (5 primitives)
```

### Kamikaze Agents for GUI Simulation

For true asynchronous GUI simulation, use **kamikaze agents** (short-lived, single-purpose agents):

```eiffel
class SV_GUI_SIMULATION_AGENT

feature -- Execution

    execute_action (a_action: SV_TEST_ACTION)
            -- Spawn a kamikaze agent to execute GUI action.
            -- Agent lives only for duration of action, then terminates.
        local
            l_thread: WORKER_THREAD
        do
            create l_thread.make (agent do_action (a_action))
            l_thread.launch
            -- Thread terminates after do_action completes
        end

feature {NONE} -- Implementation

    do_action (a_action: SV_TEST_ACTION)
            -- Execute action in isolated thread context.
            -- SCOOP-safe: separate calls to EV2 widgets.
        do
            inspect a_action.action_type
            when {SV_ACTION_TYPE}.click then
                simulate_click (a_action.target)
            when {SV_ACTION_TYPE}.type_text then
                simulate_type (a_action.target, a_action.value)
            -- ... other actions
            end
        ensure
            action_completed: True -- Agent terminates after this
        end
```

**Benefits of Kamikaze Agents:**
1. **Isolation**: Each action runs in its own thread context
2. **Timing**: Natural delays between actions (thread spawn overhead)
3. **SCOOP-safe**: Separate agents for separate GUI regions
4. **Cleanup**: Auto-termination prevents resource leaks
5. **Parallelism**: Multiple independent actions can run concurrently

**Integration with simple_process:**
Use `simple_process` for inter-agent coordination and `simple_oracle` for logging.

### Window Reuse: One Instance → Many Use-Cases

A single window/form/dialog instance can serve **multiple use-cases** without recreation:

```
┌─────────────────────────────────────────────────────────────────────┐
│  DEMO_LOGIN (Window Instance)                                       │
│                                                                     │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐    │
│  │ login_success   │  │ login_empty_    │  │ login_cancel    │    │
│  │ .json           │  │ username.json   │  │ .json           │    │
│  │                 │  │                 │  │                 │    │
│  │ 1:M reuse       │  │ same window     │  │ reset & replay  │    │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘    │
│           │                    │                    │              │
│           ▼                    ▼                    ▼              │
│      ┌────────────────────────────────────────────────────┐       │
│      │  Single DEMO_LOGIN Instance                         │       │
│      │  - reset_state() between use-cases                  │       │
│      │  - no window recreation overhead                    │       │
│      │  - preserves widget references                      │       │
│      └────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────┘
```

#### Test Suite Runner Pattern

```eiffel
class SV_TEST_SUITE_RUNNER

feature -- Execution

    run_suite (a_window: SV_WINDOW; a_use_cases: ARRAY [SV_USE_CASE])
            -- Run multiple use-cases against single window instance.
        require
            window_ready: a_window.is_displayed
            use_cases_valid: a_use_cases.count > 0
        local
            l_result: SV_TEST_LOG
        do
            across a_use_cases as uc loop
                -- Reset window to initial state
                reset_window_state (a_window)

                -- Execute use-case
                l_result := execute_use_case (uc, a_window)

                -- Log result
                log_result (l_result)

                -- Optional: screenshot on failure
                if l_result.has_failures then
                    capture_screenshot (a_window, uc.name)
                end
            end
        ensure
            all_use_cases_executed: results.count = a_use_cases.count
        end

    reset_window_state (a_window: SV_WINDOW)
            -- Reset window to clean state for next use-case.
        do
            -- Clear all text fields
            across a_window.all_text_fields as tf loop
                tf.clear
            end
            -- Uncheck all checkboxes
            across a_window.all_checkboxes as cb loop
                cb.uncheck
            end
            -- Reset focus to first widget
            a_window.set_focus_to_first
            -- Clear any error states
            across a_window.all_labels as lbl loop
                if lbl.is_error_label then
                    lbl.clear
                end
            end
        end
```

#### Use-Case JSON with Reset Directive

```json
{
  "name": "login_suite",
  "window": "demo_login",
  "reuse_window": true,
  "reset_between_cases": true,
  "cases": [
    {
      "name": "login_empty_username",
      "steps": [
        {"step": 1, "action": "click", "target": "button_login"},
        {"step": 2, "action": "assert_text", "target": "label_status", "expected": "Please enter username"}
      ]
    },
    {
      "name": "login_empty_password",
      "reset": "partial",
      "reset_fields": ["field_username"],
      "steps": [
        {"step": 1, "action": "type", "target": "field_username", "value": "testuser"},
        {"step": 2, "action": "click", "target": "button_login"},
        {"step": 3, "action": "assert_text", "target": "label_status", "expected": "Please enter password"}
      ]
    },
    {
      "name": "login_success",
      "reset": "full",
      "steps": [
        {"step": 1, "action": "type", "target": "field_username", "value": "testuser"},
        {"step": 2, "action": "type", "target": "field_password", "value": "secret"},
        {"step": 3, "action": "click", "target": "button_login"},
        {"step": 4, "action": "assert_text", "target": "label_status", "expected": "Logging in as testuser..."}
      ]
    }
  ]
}
```

**Trade-offs - Window Reuse vs Recreation:**

| Approach | Pros | Cons |
|----------|------|------|
| **Reuse** | Faster (no recreation), realistic session | State leakage risk, complex reset logic |
| **Recreate** | Clean state guaranteed, simpler code | Slightly slower, new widget refs each time |

**Recommendation:** Start with **window recreation** (destroy + create) between use-cases. It's simpler, guarantees clean state, and avoids subtle bugs from incomplete resets. Only optimize to reuse if test suite performance becomes a bottleneck.

```eiffel
run_use_case_suite (a_demo_class: TYPE [SV_QUICK]; a_use_cases: ARRAY [SV_USE_CASE])
        -- Simple approach: fresh window per use-case.
    local
        l_window: SV_WINDOW
        l_demo: SV_QUICK
    do
        across a_use_cases as uc loop
            -- Create fresh window
            create {like a_demo_class} l_demo.make
            l_window := l_demo.main_window

            -- Execute use-case (fire events in order)
            execute_steps (l_window, uc.steps)

            -- Verify and log results
            log_result (uc, l_window)

            -- Destroy window
            l_window.destroy
        end
    end
```

---

### Verification Algorithm

```eiffel
verify_correlation (use_case: SV_USE_CASE; log: SV_TEST_LOG): BOOLEAN
    -- Ensure log matches use-case structure
    require
        use_case_valid: use_case /= Void
        log_valid: log /= Void
    do
        Result := True
        -- Name must match
        Result := Result and use_case.name.same_string (log.use_case)
        -- Step count must match
        Result := Result and use_case.steps.count = log.steps.count
        -- Each step must correlate
        across use_case.steps as uc loop
            if attached log.steps.i_th (uc.cursor_index) as log_step then
                Result := Result and uc.step = log_step.step
                Result := Result and uc.action.same_string (log_step.action)
                if attached uc.target as t then
                    Result := Result and t.same_string (log_step.target)
                end
            else
                Result := False
            end
        end
    end
```

---

## Test Execution Cycle

### The Complete Action Cycle

Every GUI test follows this cycle, ensuring full traceability from specification to execution to verification:

```
┌─────────────────────────────────────────────────────────────────────┐
│  1. USE-CASE JSON          →  Human-readable test specification    │
│     (testing/use_cases/*.json)                                      │
├─────────────────────────────────────────────────────────────────────┤
│  2. HARNESS LOAD           →  Parse JSON into executable steps     │
│     (SV_TEST_HARNESS.load_use_case)                                │
├─────────────────────────────────────────────────────────────────────┤
│  3. STEP EXECUTION         →  Execute each action in sequence      │
│     (simulate_click, simulate_type, etc.)                          │
├─────────────────────────────────────────────────────────────────────┤
│  4. LOGGING OUTPUT         →  Record each step with timestamps     │
│     (simple_logger → console + file + JSON)                        │
├─────────────────────────────────────────────────────────────────────┤
│  5. CORRELATION            →  Match log entries to use-case steps  │
│     (step_number, action, target, expected/actual)                 │
├─────────────────────────────────────────────────────────────────────┤
│  6. VERIFICATION           →  Assert expected outcomes             │
│     (pass/fail per step, overall test result)                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Use-Case JSON Structure

Each use-case file defines a complete test scenario:

```json
{
  "name": "login_form_validation",
  "description": "Test login form with invalid credentials",
  "setup": {
    "window": "demo_login",
    "initial_state": {}
  },
  "steps": [
    {
      "step": 1,
      "action": "type",
      "target": "field_username",
      "value": "testuser"
    },
    {
      "step": 2,
      "action": "type",
      "target": "field_password",
      "value": "wrongpass"
    },
    {
      "step": 3,
      "action": "click",
      "target": "button_login"
    },
    {
      "step": 4,
      "action": "assert_text",
      "target": "label_status",
      "expected": "Invalid credentials"
    }
  ],
  "teardown": {
    "action": "close_window"
  }
}
```

### Log Output Correspondence

The harness MUST produce logs that exactly mirror the use-case steps:

**Human-Readable Log:**
```
=== USE-CASE: login_form_validation ===
[2025-12-28 21:10:00.100] SETUP: Window demo_login initialized
[2025-12-28 21:10:00.150] STEP 1: type -> field_username = "testuser"
  RESULT: OK
[2025-12-28 21:10:00.200] STEP 2: type -> field_password = "********"
  RESULT: OK
[2025-12-28 21:10:00.250] STEP 3: click -> button_login
  RESULT: OK
[2025-12-28 21:10:00.350] STEP 4: assert_text -> label_status
  EXPECTED: "Invalid credentials"
  ACTUAL:   "Invalid credentials"
  RESULT: PASS
[2025-12-28 21:10:00.400] TEARDOWN: Window closed
=== RESULT: PASSED (4/4 steps) ===
```

**Machine-Readable JSON Log:**
```json
{
  "use_case": "login_form_validation",
  "timestamp": "2025-12-28T21:10:00.100Z",
  "steps": [
    {"step": 1, "action": "type", "target": "field_username", "result": "ok", "duration_ms": 50},
    {"step": 2, "action": "type", "target": "field_password", "result": "ok", "duration_ms": 50},
    {"step": 3, "action": "click", "target": "button_login", "result": "ok", "duration_ms": 100},
    {"step": 4, "action": "assert_text", "target": "label_status", "expected": "Invalid credentials", "actual": "Invalid credentials", "passed": true, "duration_ms": 50}
  ],
  "overall": "passed",
  "passed_steps": 4,
  "total_steps": 4
}
```

### Correlation Requirements

For proper use-case ↔ log correlation:

1. **Step Numbers Match**: Log step numbers MUST match JSON step numbers exactly
2. **Action Names Match**: Log action names MUST match JSON action names
3. **Target IDs Match**: Widget IDs in logs MUST match JSON target values
4. **Expected/Actual Recorded**: Assertions MUST record both expected and actual values
5. **Timestamps Per Step**: Each step gets its own timestamp for timing analysis
6. **Duration Tracking**: Record milliseconds per step for performance regression detection

---

## Demo Test Coverage Requirements

### Required Use-Case Files

Each demo application MUST have corresponding use-case JSON files covering all testable interactions:

#### Demo: Login Form (demo_login)
- `login_success.json` - Valid credentials flow
- `login_empty_username.json` - Empty username validation
- `login_empty_password.json` - Empty password validation
- `login_cancel.json` - Cancel button clears fields
- `login_remember_toggle.json` - Checkbox toggle behavior

#### Demo: Complex Layout (demo_layout)
- `layout_refresh_button.json` - Refresh button updates status
- `layout_clear_button.json` - Clear button clears status
- `layout_checkbox_toggles.json` - Settings checkboxes work
- `layout_splitter_drag.json` - Splitter repositioning
- `layout_tab_navigation.json` - Tab switching

#### Demo: Data Browser (demo_data)
- `data_add_row.json` - Add button creates new row
- `data_edit_selection.json` - Edit requires selection
- `data_delete_selection.json` - Delete requires selection
- `data_search_input.json` - Search field behavior
- `data_row_selection.json` - Row selection updates status

### Random Test Data with simple_randomizer

Use-case JSON files often need **semantically appropriate random data**. The `simple_randomizer` library provides domain-aware random generation:

```eiffel
class SV_TEST_DATA_GENERATOR

inherit
    SIMPLE_RANDOMIZER

feature -- Test Data Generation

    random_username: STRING
            -- Generate realistic username.
        do
            Result := random_name.as_lower + random_integer_range (1, 999).out
            -- e.g., "johnsmith42", "alice_jones123"
        end

    random_email: STRING
            -- Generate realistic email address.
        do
            Result := random_username + "@" + random_choice (<<"example.com", "test.org", "demo.net">>)
        end

    random_password: STRING
            -- Generate secure-looking password.
        do
            Result := random_alphanumeric (8) + random_special_char + random_integer_range (0, 9).out
            -- e.g., "xK7mPq2n$4"
        end

    random_status: STRING
            -- Generate random status value.
        do
            Result := random_choice (<<"Active", "Inactive", "Pending", "Suspended">>)
        end
```

#### Use-Case JSON with Data Placeholders

```json
{
  "name": "login_random_credentials",
  "data_generators": {
    "username": "random_username",
    "password": "random_password"
  },
  "steps": [
    {"step": 1, "action": "type", "target": "field_username", "value": "${username}"},
    {"step": 2, "action": "type", "target": "field_password", "value": "${password}"},
    {"step": 3, "action": "click", "target": "button_login"}
  ]
}
```

The harness resolves `${username}` at runtime via `simple_randomizer`, ensuring:
- Each test run uses unique data
- Data is semantically appropriate (usernames look like usernames)
- Reproducible with seed (for debugging)

---

## Screen Pathway Coverage

### The Problem

A single screen/form has **multiple pathways** a user might take:
- Happy path (fill all fields correctly, submit)
- Validation paths (each field can be empty/invalid)
- Tab navigation paths (keyboard-only users)
- State-dependent paths (buttons enabled/disabled based on state)
- Random exploration paths (users clicking around)

### Pathway Graph Model

Model each screen as a **state machine graph**:

```
┌─────────────────────────────────────────────────────────────────────┐
│  LOGIN FORM - State Machine                                         │
│                                                                     │
│  States:                                                            │
│    S0: Initial (all empty)                                          │
│    S1: Username entered                                             │
│    S2: Password entered                                             │
│    S3: Both entered                                                 │
│    S4: Remember checked                                             │
│    S5: Submitted (success/failure)                                  │
│                                                                     │
│  Transitions (edges):                                               │
│    S0 --[type username]--> S1                                       │
│    S0 --[type password]--> S2                                       │
│    S1 --[type password]--> S3                                       │
│    S2 --[type username]--> S3                                       │
│    S0,S1,S2,S3 --[check remember]--> S4                            │
│    S3 --[click login]--> S5                                         │
│    S0,S1,S2 --[click login]--> VALIDATION_ERROR                    │
│    ANY --[click cancel]--> S0                                       │
│                                                                     │
│  Widget Enable/Disable by State:                                    │
│    button_login: always enabled (shows validation on invalid)       │
│    button_cancel: always enabled                                    │
│    field_*: always enabled                                          │
└─────────────────────────────────────────────────────────────────────┘
```

### Use-Case JSON for Pathway Coverage

#### 1. Explicit Pathway Enumeration

```json
{
  "name": "login_pathways",
  "type": "pathway_coverage",
  "screen": "DEMO_LOGIN",
  "pathways": [
    {
      "name": "happy_path",
      "path": ["S0", "S1", "S3", "S5"],
      "steps": [
        {"action": "type", "target": "field_username", "value": "user"},
        {"action": "type", "target": "field_password", "value": "pass"},
        {"action": "click", "target": "button_login"},
        {"action": "assert_text", "target": "label_status", "expected": "Logging in"}
      ]
    },
    {
      "name": "password_first",
      "path": ["S0", "S2", "S3", "S5"],
      "steps": [
        {"action": "type", "target": "field_password", "value": "pass"},
        {"action": "type", "target": "field_username", "value": "user"},
        {"action": "click", "target": "button_login"}
      ]
    },
    {
      "name": "validation_no_username",
      "path": ["S0", "VALIDATION_ERROR"],
      "steps": [
        {"action": "click", "target": "button_login"},
        {"action": "assert_text", "target": "label_status", "expected": "Please enter username"}
      ]
    },
    {
      "name": "cancel_resets",
      "path": ["S0", "S3", "S0"],
      "steps": [
        {"action": "type", "target": "field_username", "value": "user"},
        {"action": "type", "target": "field_password", "value": "pass"},
        {"action": "click", "target": "button_cancel"},
        {"action": "assert_value", "target": "field_username", "expected": ""}
      ]
    }
  ]
}
```

#### 2. Tab Navigation Pathways

```json
{
  "name": "login_tab_navigation",
  "type": "tab_pathway",
  "screen": "DEMO_LOGIN",
  "description": "Test keyboard-only navigation through form",
  "tab_order": [
    "field_username",
    "field_password",
    "checkbox_remember",
    "button_cancel",
    "button_login"
  ],
  "steps": [
    {"step": 1, "action": "focus", "target": "field_username"},
    {"step": 2, "action": "assert_focused", "target": "field_username"},
    {"step": 3, "action": "type", "value": "testuser"},
    {"step": 4, "action": "tab"},
    {"step": 5, "action": "assert_focused", "target": "field_password"},
    {"step": 6, "action": "type", "value": "secret"},
    {"step": 7, "action": "tab"},
    {"step": 8, "action": "assert_focused", "target": "checkbox_remember"},
    {"step": 9, "action": "key", "value": "space", "comment": "Toggle checkbox"},
    {"step": 10, "action": "assert_checked", "target": "checkbox_remember", "expected": true},
    {"step": 11, "action": "tab"},
    {"step": 12, "action": "tab"},
    {"step": 13, "action": "assert_focused", "target": "button_login"},
    {"step": 14, "action": "key", "value": "enter"},
    {"step": 15, "action": "assert_text", "target": "label_status", "expected": "Logging in"}
  ]
}
```

#### 3. State-Machine Driven Enable/Disable Testing

```json
{
  "name": "data_browser_state_machine",
  "type": "state_machine",
  "screen": "DEMO_DATA",
  "states": {
    "no_selection": {
      "widget_states": {
        "button_add": {"enabled": true},
        "button_edit": {"enabled": true, "click_result": "Select a row to edit"},
        "button_delete": {"enabled": true, "click_result": "Select a row to delete"},
        "button_search": {"enabled": true}
      }
    },
    "row_selected": {
      "precondition": [
        {"action": "select_cell", "target": "grid_data", "value": [1, 1]}
      ],
      "widget_states": {
        "button_add": {"enabled": true},
        "button_edit": {"enabled": true, "click_result": "Editing row 1"},
        "button_delete": {"enabled": true, "click_result": "Delete not implemented"},
        "button_search": {"enabled": true}
      }
    }
  },
  "test_all_states": true,
  "test_all_transitions": true
}
```

#### 4. Random Exploration Testing (Monkey Testing)

```json
{
  "name": "login_random_exploration",
  "type": "random_exploration",
  "screen": "DEMO_LOGIN",
  "seed": 12345,
  "max_actions": 50,
  "action_weights": {
    "type_random_text": 0.3,
    "click_enabled_button": 0.25,
    "tab": 0.15,
    "clear_field": 0.1,
    "toggle_checkbox": 0.1,
    "wait": 0.1
  },
  "invariants": [
    {"type": "no_crash", "description": "App must not crash"},
    {"type": "no_hang", "timeout_ms": 5000},
    {"type": "widget_visible", "target": "button_login", "always": true}
  ],
  "stop_conditions": [
    {"type": "state_reached", "state": "S5"},
    {"type": "action_count", "count": 50}
  ]
}
```

### Pathway Coverage Matrix

Track which pathways are covered:

```
┌──────────────────────────────────────────────────────────────┐
│  LOGIN FORM - Coverage Matrix                                 │
├──────────────────────────────────────────────────────────────┤
│  Pathway                      │ Test File              │ ✓  │
├───────────────────────────────┼────────────────────────┼────┤
│  S0 → S1 → S3 → S5           │ login_success.json     │ ✅ │
│  S0 → S2 → S3 → S5           │ login_password_first   │ ⬜ │
│  S0 → S5 (empty)             │ login_empty_username   │ ✅ │
│  S1 → S5 (no password)       │ login_empty_password   │ ✅ │
│  ANY → S0 (cancel)           │ login_cancel.json      │ ✅ │
│  Tab order complete          │ login_tab_nav.json     │ ⬜ │
│  Random exploration (50 acts)│ login_random.json      │ ⬜ │
├───────────────────────────────┼────────────────────────┼────┤
│  Coverage: 4/7 (57%)         │                        │    │
└──────────────────────────────────────────────────────────────┘
```

### Implementation: Pathway Generator

```eiffel
class SV_PATHWAY_GENERATOR

feature -- Generation

    generate_all_pathways (a_state_machine: SV_STATE_MACHINE): ARRAY [SV_PATHWAY]
            -- Generate all possible paths through state machine.
        local
            l_paths: ARRAYED_LIST [SV_PATHWAY]
            l_visited: HASH_TABLE [BOOLEAN, STRING]
        do
            create l_paths.make (20)
            create l_visited.make (10)

            -- DFS from initial state to all terminal states
            explore_paths (a_state_machine.initial_state,
                          create {SV_PATHWAY}.make,
                          l_visited,
                          l_paths,
                          a_state_machine)

            Result := l_paths.to_array
        end

    generate_tab_pathway (a_window: SV_WINDOW): SV_PATHWAY
            -- Generate tab order pathway for window.
        do
            create Result.make
            across a_window.tab_order_widgets as w loop
                Result.add_step (create {SV_STEP}.make_focus (w.widget_id))
                Result.add_step (create {SV_STEP}.make_assert_focused (w.widget_id))
                Result.add_step (create {SV_STEP}.make_tab)
            end
        end

    generate_random_pathway (a_window: SV_WINDOW; a_seed: INTEGER; a_max_steps: INTEGER): SV_PATHWAY
            -- Generate random exploration pathway.
        local
            l_random: SIMPLE_RANDOMIZER
        do
            create l_random.make_with_seed (a_seed)
            create Result.make

            from until Result.step_count >= a_max_steps loop
                Result.add_step (random_valid_action (a_window, l_random))
            end
        end
```

### Future: Integration with Data Binding & State Machines

This pathway coverage infrastructure becomes **critical** when we implement:

| Innovation | Testing Impact |
|------------|----------------|
| **SV_OBSERVABLE** (reactive binding) | Auto-generate tests for all bound properties |
| **SV_BINDING** (two-way sync) | Verify model ↔ UI synchronization |
| **SV_STATE_MACHINE** (formal states) | Auto-generate pathways from state graph |
| **SV_FORM** (validation) | Test all validation rule combinations |

When `SV_STATE_MACHINE` is applied to a window:
```eiffel
class DEMO_LOGIN_WITH_STATE
inherit
    SV_QUICK
    SV_STATE_MACHINE [LOGIN_STATE]

feature -- State Machine

    define_states
        do
            add_state ({LOGIN_STATE}.initial)
            add_state ({LOGIN_STATE}.username_entered)
            add_state ({LOGIN_STATE}.password_entered)
            add_state ({LOGIN_STATE}.ready_to_submit)
            add_state ({LOGIN_STATE}.submitted)

            add_transition ({LOGIN_STATE}.initial, {LOGIN_STATE}.username_entered,
                           agent username_field.text_entered)
            -- etc.
        end

    apply_state (a_state: LOGIN_STATE)
        do
            inspect a_state
            when {LOGIN_STATE}.ready_to_submit then
                login_button.enable
            else
                login_button.disable
            end
        end
```

The test harness can then **automatically** generate:
1. All possible state transitions as test cases
2. Enable/disable assertions for each state
3. Invalid transition attempts (should be blocked)
4. Edge case: rapid state changes

---

### Integration with SV_TEST_HARNESS

```eiffel
run_pathway_coverage (a_screen: SV_QUICK; a_pathways_file: STRING)
        -- Run all pathways from JSON file against screen.
    local
        l_pathways: ARRAY [SV_PATHWAY]
        l_coverage: SV_COVERAGE_TRACKER
    do
        l_pathways := parse_pathways (a_pathways_file)
        create l_coverage.make (l_pathways.count)

        across l_pathways as p loop
            -- Create fresh window
            l_window := create_window (a_screen)

            -- Execute pathway
            if execute_pathway (p, l_window) then
                l_coverage.mark_passed (p.name)
            else
                l_coverage.mark_failed (p.name, last_error)
            end

            -- Destroy window
            l_window.destroy
        end

        -- Report coverage
        print_coverage_report (l_coverage)
    end
```

---

### Test File Organization

```
simple_vision/
└── testing/
    └── use_cases/
        ├── hello_world_demo.json       (existing)
        ├── demo_login/
        │   ├── login_success.json
        │   ├── login_empty_username.json
        │   ├── login_empty_password.json
        │   └── login_cancel.json
        ├── demo_layout/
        │   ├── layout_refresh_button.json
        │   └── layout_tab_navigation.json
        └── demo_data/
            ├── data_add_row.json
            ├── data_row_selection.json
            └── data_search_input.json
```

---

## Future Enhancements

### 1. Widget Tree Serialization
Export entire widget tree as JSON for comparison:
```json
{
  "window": "main_window",
  "children": [
    { "id": "button_submit", "type": "SV_BUTTON", "text": "Submit", "enabled": true },
    { "id": "label_error", "type": "SV_TEXT", "text": "", "visible": false }
  ]
}
```

### 2. Visual Diff
Compare screenshots programmatically (pixel diff or AI-based).

### 3. Record Mode
Watch user interactions and generate JSON use-case files automatically.

### 4. Headless Mode
Run tests without visible windows (if EV2 supports it).

---

## Related Documents
- `SIMPLE_VISION_INNOVATIONS.md` - Innovation 14: GUI Testing Harness
- `SIMPLE_VISION_IMPLEMENTATION_PLAN.md` - Phase estimates
- `testing/use_cases/hello_world_demo.json` - Example use-case file

---

*This document captures design discussions and decisions. Update as the testing infrastructure evolves.*
