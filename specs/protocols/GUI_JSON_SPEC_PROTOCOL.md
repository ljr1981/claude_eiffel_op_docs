# GUI JSON Specification Protocol v1.0

A machine-readable format for defining GUI applications. Used for:
- Design-time specification and documentation
- Code generation (JSON → Eiffel)
- Test scenario definition
- UI state machine validation

## Core Principles

1. **Declarative**: Describe WHAT, not HOW
2. **Complete**: Everything needed to generate code
3. **Testable**: Use cases define expected behavior
4. **Layered**: Separates UI, logic, and data concerns

---

## Schema Overview

```json
{
  "app": { },           // Application metadata
  "windows": [ ],       // Window definitions
  "state_machines": [ ],// UI state management
  "data_models": [ ],   // Data structures
  "use_cases": [ ],     // User flow specifications
  "themes": { }         // Optional theme customization
}
```

---

## 1. App Metadata

```json
{
  "app": {
    "name": "speech_studio",
    "title": "Speech Studio",
    "version": "1.0.0",
    "description": "GUI front-end for simple_speech TTS engine",
    "author": "Larry Rix",
    "libraries": ["simple_vision", "simple_speech", "simple_repository"]
  }
}
```

---

## 2. Windows

Each window defines its layout, widgets, and behavior.

```json
{
  "windows": [
    {
      "id": "main_window",
      "type": "window",
      "title": "Speech Studio",
      "size": { "width": 800, "height": 600 },
      "min_size": { "width": 400, "height": 300 },
      "resizable": true,
      "state_machine": "app_states",
      "layout": {
        "type": "vertical",
        "padding": 10,
        "spacing": 8,
        "children": [ ]
      },
      "on_close": "confirm_exit"
    }
  ]
}
```

### Window Types
- `window` - Main application window
- `dialog` - Modal dialog
- `panel` - Embeddable panel (not standalone)

---

## 3. Layouts

Layouts define widget arrangement.

### Vertical Layout
```json
{
  "type": "vertical",
  "padding": 10,
  "spacing": 8,
  "align": "stretch",
  "children": [ ]
}
```

### Horizontal Layout
```json
{
  "type": "horizontal",
  "padding": 10,
  "spacing": 8,
  "align": "center",
  "children": [ ]
}
```

### Grid Layout
```json
{
  "type": "grid",
  "columns": 2,
  "row_spacing": 8,
  "column_spacing": 12,
  "children": [
    { "widget": "...", "row": 0, "col": 0 },
    { "widget": "...", "row": 0, "col": 1, "colspan": 2 }
  ]
}
```

### Tab Layout
```json
{
  "type": "tabs",
  "tabs": [
    { "label": "Basic", "layout": { } },
    { "label": "Advanced", "layout": { } }
  ]
}
```

---

## 4. Widgets

### Common Properties
All widgets support:
```json
{
  "id": "unique_identifier",
  "type": "widget_type",
  "visible": true,
  "enabled": true,
  "tooltip": "Help text",
  "bind": "data_path",
  "validation": { },
  "on_change": "action_name"
}
```

### Widget Types

#### Label
```json
{
  "id": "title_label",
  "type": "label",
  "text": "Enter text to speak:",
  "style": "heading"
}
```
Styles: `normal`, `heading`, `subheading`, `caption`, `error`

#### Text Field
```json
{
  "id": "name_field",
  "type": "text_field",
  "placeholder": "Enter name",
  "max_length": 100,
  "bind": "user.name",
  "validation": {
    "required": true,
    "pattern": "^[A-Za-z ]+$",
    "message": "Name must contain only letters"
  }
}
```

#### Text Area
```json
{
  "id": "speech_text",
  "type": "text_area",
  "placeholder": "Enter text to speak...",
  "rows": 5,
  "bind": "speech.text",
  "validation": {
    "required": true,
    "min_length": 1,
    "max_length": 5000
  }
}
```

#### Password Field
```json
{
  "id": "password",
  "type": "password_field",
  "placeholder": "Enter password",
  "show_toggle": true
}
```

#### Masked Field
```json
{
  "id": "phone",
  "type": "masked_field",
  "mask": "(###) ###-####",
  "placeholder": "(555) 555-5555"
}
```

#### Dropdown
```json
{
  "id": "voice_select",
  "type": "dropdown",
  "options": [
    { "label": "David", "value": "david" },
    { "label": "Zira", "value": "zira" },
    { "label": "Mark", "value": "mark" }
  ],
  "bind": "speech.voice",
  "on_change": "voice_changed"
}
```

#### Dropdown (Dynamic)
```json
{
  "id": "voice_select",
  "type": "dropdown",
  "options_source": "available_voices",
  "label_field": "name",
  "value_field": "id",
  "bind": "speech.voice_id"
}
```

#### Slider
```json
{
  "id": "rate_slider",
  "type": "slider",
  "min": -10,
  "max": 10,
  "step": 1,
  "default": 0,
  "show_value": true,
  "bind": "speech.rate"
}
```

#### Checkbox
```json
{
  "id": "save_audio",
  "type": "checkbox",
  "label": "Save audio to file",
  "bind": "options.save_audio",
  "on_change": "toggle_save_options"
}
```

#### Button
```json
{
  "id": "speak_button",
  "type": "button",
  "text": "Speak",
  "style": "primary",
  "icon": "play",
  "on_click": "do_speak",
  "enabled_when": "can_speak"
}
```
Styles: `primary`, `secondary`, `danger`, `link`

#### Button Group
```json
{
  "id": "playback_controls",
  "type": "button_group",
  "buttons": [
    { "id": "play", "icon": "play", "on_click": "do_play" },
    { "id": "pause", "icon": "pause", "on_click": "do_pause" },
    { "id": "stop", "icon": "stop", "on_click": "do_stop" }
  ]
}
```

#### Progress Bar
```json
{
  "id": "speech_progress",
  "type": "progress",
  "bind": "speech.progress",
  "show_percent": true
}
```

#### Data Grid
```json
{
  "id": "history_grid",
  "type": "grid",
  "data_source": "speech_history",
  "columns": [
    { "field": "text", "label": "Text", "width": 300 },
    { "field": "voice", "label": "Voice", "width": 100 },
    { "field": "created_at", "label": "Date", "width": 150, "format": "datetime" }
  ],
  "selectable": true,
  "on_select": "history_selected",
  "on_double_click": "replay_history"
}
```

#### Separator
```json
{
  "type": "separator",
  "orientation": "horizontal"
}
```

#### Spacer
```json
{
  "type": "spacer",
  "size": 20
}
```

#### Expanding Spacer
```json
{
  "type": "expander"
}
```

---

## 5. State Machines

Define UI states and transitions.

```json
{
  "state_machines": [
    {
      "id": "app_states",
      "initial": "idle",
      "states": [
        {
          "name": "idle",
          "description": "Ready for input",
          "on_enter": ["enable_input", "clear_progress"],
          "widgets": {
            "speak_button": { "enabled": true, "text": "Speak" },
            "stop_button": { "enabled": false }
          }
        },
        {
          "name": "speaking",
          "description": "TTS in progress",
          "on_enter": ["disable_input", "start_progress"],
          "on_exit": ["enable_input"],
          "widgets": {
            "speak_button": { "enabled": false },
            "stop_button": { "enabled": true }
          }
        },
        {
          "name": "paused",
          "description": "Speech paused",
          "widgets": {
            "speak_button": { "enabled": true, "text": "Resume" },
            "stop_button": { "enabled": true }
          }
        },
        {
          "name": "error",
          "description": "Error occurred",
          "on_enter": ["show_error"],
          "widgets": {
            "speak_button": { "enabled": true },
            "error_label": { "visible": true }
          }
        }
      ],
      "transitions": [
        { "event": "speak", "from": "idle", "to": "speaking", "guard": "has_text" },
        { "event": "speak", "from": "paused", "to": "speaking" },
        { "event": "pause", "from": "speaking", "to": "paused" },
        { "event": "stop", "from": ["speaking", "paused"], "to": "idle" },
        { "event": "complete", "from": "speaking", "to": "idle" },
        { "event": "error", "from": "*", "to": "error" },
        { "event": "dismiss_error", "from": "error", "to": "idle" }
      ]
    }
  ]
}
```

### State Properties
- `name` - State identifier
- `description` - Human-readable description
- `on_enter` - Actions when entering state
- `on_exit` - Actions when leaving state
- `widgets` - Widget property overrides in this state

### Transition Properties
- `event` - Trigger event name
- `from` - Source state(s), `"*"` for any
- `to` - Target state
- `guard` - Condition that must be true
- `action` - Action to execute during transition

---

## 6. Actions

Actions are named operations triggered by events.

```json
{
  "actions": {
    "do_speak": {
      "type": "sequence",
      "steps": [
        { "validate": "speech_form" },
        { "call": "speech_engine.speak", "args": ["${speech.text}", "${speech.voice}"] },
        { "trigger": "speak" }
      ]
    },
    "do_stop": {
      "type": "call",
      "target": "speech_engine.stop"
    },
    "voice_changed": {
      "type": "call",
      "target": "preview_voice"
    },
    "confirm_exit": {
      "type": "confirm",
      "message": "Are you sure you want to exit?",
      "on_yes": "exit_app"
    }
  }
}
```

### Action Types
- `call` - Call a method
- `trigger` - Trigger state machine event
- `sequence` - Execute multiple actions
- `confirm` - Show confirmation dialog
- `validate` - Validate a form
- `set` - Set a data value
- `navigate` - Show a different window/panel

---

## 7. Data Models

Define data structures used by the UI.

```json
{
  "data_models": [
    {
      "id": "speech_request",
      "fields": [
        { "name": "text", "type": "string", "required": true },
        { "name": "voice_id", "type": "string", "required": true },
        { "name": "rate", "type": "integer", "default": 0, "min": -10, "max": 10 },
        { "name": "volume", "type": "integer", "default": 100, "min": 0, "max": 100 },
        { "name": "save_to_file", "type": "boolean", "default": false },
        { "name": "output_path", "type": "string" }
      ]
    },
    {
      "id": "voice",
      "fields": [
        { "name": "id", "type": "string" },
        { "name": "name", "type": "string" },
        { "name": "language", "type": "string" },
        { "name": "gender", "type": "string", "enum": ["male", "female", "neutral"] }
      ]
    }
  ]
}
```

---

## 8. Data Sources

Connect to data providers.

```json
{
  "data_sources": [
    {
      "id": "available_voices",
      "type": "method",
      "call": "speech_engine.get_voices",
      "cache": true
    },
    {
      "id": "speech_history",
      "type": "repository",
      "entity": "speech_record",
      "order_by": "created_at DESC",
      "limit": 100
    }
  ]
}
```

---

## 9. Use Cases

Define user flows for testing and documentation.

```json
{
  "use_cases": [
    {
      "id": "UC001",
      "name": "Basic Speech",
      "description": "User enters text and hears it spoken",
      "preconditions": ["app_in_idle_state", "voices_available"],
      "steps": [
        {
          "action": "type",
          "target": "speech_text",
          "value": "Hello, world!"
        },
        {
          "action": "select",
          "target": "voice_select",
          "value": "david"
        },
        {
          "action": "click",
          "target": "speak_button"
        },
        {
          "action": "wait_for_state",
          "state_machine": "app_states",
          "state": "speaking"
        },
        {
          "action": "wait_for_state",
          "state_machine": "app_states",
          "state": "idle"
        }
      ],
      "postconditions": ["speech_completed", "no_errors"],
      "expected_results": [
        "Audio played successfully",
        "Progress bar reached 100%",
        "App returned to idle state"
      ]
    },
    {
      "id": "UC002",
      "name": "Stop Speech",
      "description": "User stops speech in progress",
      "preconditions": ["app_in_speaking_state"],
      "steps": [
        {
          "action": "click",
          "target": "stop_button"
        },
        {
          "action": "assert_state",
          "state_machine": "app_states",
          "state": "idle"
        }
      ],
      "postconditions": ["speech_stopped", "app_in_idle_state"]
    },
    {
      "id": "UC003",
      "name": "Empty Text Validation",
      "description": "System prevents speaking empty text",
      "steps": [
        {
          "action": "clear",
          "target": "speech_text"
        },
        {
          "action": "click",
          "target": "speak_button"
        },
        {
          "action": "assert_visible",
          "target": "validation_error"
        },
        {
          "action": "assert_state",
          "state_machine": "app_states",
          "state": "idle"
        }
      ]
    }
  ]
}
```

### Use Case Step Actions
- `type` - Type text into field
- `clear` - Clear a field
- `click` - Click a button/widget
- `select` - Select dropdown option
- `check` / `uncheck` - Toggle checkbox
- `wait` - Wait milliseconds
- `wait_for_state` - Wait for state machine state
- `assert_state` - Verify state machine state
- `assert_visible` / `assert_hidden` - Check visibility
- `assert_enabled` / `assert_disabled` - Check enabled state
- `assert_text` - Verify text content
- `assert_value` - Verify field value

---

## 10. Guards (Conditions)

Named conditions for state transitions and widget states.

```json
{
  "guards": {
    "has_text": {
      "type": "not_empty",
      "field": "speech.text"
    },
    "can_speak": {
      "type": "and",
      "conditions": [
        { "type": "not_empty", "field": "speech.text" },
        { "type": "state", "machine": "app_states", "in": ["idle", "paused"] }
      ]
    },
    "is_saving": {
      "type": "equals",
      "field": "options.save_audio",
      "value": true
    }
  }
}
```

---

## 11. Themes (Optional)

Override default theme settings.

```json
{
  "themes": {
    "colors": {
      "primary": "#0066CC",
      "secondary": "#6C757D",
      "success": "#28A745",
      "danger": "#DC3545",
      "background": "#FFFFFF",
      "surface": "#F8F9FA",
      "text_primary": "#212529"
    },
    "fonts": {
      "family": "Segoe UI",
      "size_base": 14
    },
    "spacing": {
      "padding": 12,
      "margin": 8
    }
  }
}
```

---

## Complete Example

See: `SPEECH_STUDIO_SPEC.json` for a complete application specification.

---

## Code Generation

The JSON spec generates:

1. **Window classes** - One class per window
2. **Form classes** - Data binding and validation
3. **State machine instances** - Baked into code (not loaded at runtime)
4. **Test classes** - From use cases
5. **Repository interfaces** - From data sources

### Generated Code Pattern

```eiffel
class SPEECH_STUDIO_MAIN_WINDOW
inherit
    SV_WINDOW

create
    make

feature {NONE} -- Initialization
    make
        do
            -- Generated from JSON spec
            create_widgets
            setup_state_machine
            setup_bindings
            setup_actions
        end

feature {NONE} -- Widgets (generated)
    speech_text: SV_TEXT_AREA
    voice_select: SV_DROPDOWN
    speak_button: SV_BUTTON
    -- ... more widgets

feature {NONE} -- State Machine (baked in, not loaded)
    app_states: SV_STATE_MACHINE

    setup_state_machine
        do
            create app_states.make ("app_states")
            app_states
                .state ("idle").enter_action (agent on_enter_idle)
                .state ("speaking").enter_action (agent on_enter_speaking)
                .state ("paused")
                .state ("error").enter_action (agent on_enter_error)
            app_states
                .on ("speak").from_state ("idle").to ("speaking")
                    .only_if (agent has_text).apply
                -- ... more transitions
            app_states.set_initial ("idle").start
        end
end
```

---

## Validation

JSON specs are validated for:
- Required fields present
- Widget IDs unique
- State machine states exist for transitions
- Bound data paths valid
- Use case targets exist

Use `simple_vision` validator:
```eiffel
validator := create {SV_SPEC_VALIDATOR}.make
result := validator.validate_file ("app_spec.json")
if result.has_errors then
    across result.errors as e loop
        print (e.item)
    end
end
```
