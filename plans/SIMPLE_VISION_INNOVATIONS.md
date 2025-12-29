# simple_vision Deep Innovations

**Date:** December 28, 2025
**Author:** Larry Rix + Claude
**Status:** Innovation Blueprint - Revolutionary Extensions

---

## Executive Summary

This document extends the base simple_vision implementation plan with **deep innovations** that transform EiffelVision-2 from a 1990s/2000s GUI toolkit into a **2025 state-of-the-art framework**.

We're not just wrapping EV2 — we're **revolutionizing** it.

---

# INNOVATION 1: Reactive Data Binding System

## The Problem
EV2 requires manual UI updates. When data changes, developers must explicitly call `set_text`, `extend`, etc. This is tedious and error-prone.

## The Solution: SV_OBSERVABLE + SV_BINDING

### Core Classes

```eiffel
class SV_OBSERVABLE [G]
    -- A reactive value that notifies subscribers when changed

feature -- Access
    value: G

feature -- Modification
    set_value (a_value: G)
        do
            if value /~ a_value then
                value := a_value
                notify_subscribers
            end
        end

feature -- Subscription (Pub-Sub via Agents)
    subscribe (a_subscriber: PROCEDURE [G])
        -- Add subscriber to be notified on changes

    unsubscribe (a_subscriber: PROCEDURE [G])
        -- Remove subscriber

feature {NONE} -- Implementation
    subscribers: ARRAYED_LIST [PROCEDURE [G]]

    notify_subscribers
        do
            across subscribers as sub loop
                sub.item.call ([value])
            end
        end
end
```

### Data Source Abstraction Layer

```eiffel
deferred class SV_DATA_SOURCE [G]
    -- Abstract data source - could be SQL, REST, file, memory

feature -- Queries
    deferred get (a_id: ANY): detachable G
    deferred get_all: LIST [G]
    deferred query (a_predicate: PREDICATE [G]): LIST [G]

feature -- Commands
    deferred insert (a_item: G): BOOLEAN
    deferred update (a_item: G): BOOLEAN
    deferred delete (a_id: ANY): BOOLEAN

feature -- Reactive
    on_change: SV_ACTION_SEQUENCE [TUPLE [operation: STRING; item: G]]
        -- Fires when data changes
end

-- Concrete implementations
class SV_SQL_DATA_SOURCE [G] inherit SV_DATA_SOURCE [G]
    -- Backed by simple_sql

class SV_MEMORY_DATA_SOURCE [G] inherit SV_DATA_SOURCE [G]
    -- In-memory list

class SV_REST_DATA_SOURCE [G] inherit SV_DATA_SOURCE [G]
    -- REST API backend

class SV_FILE_DATA_SOURCE [G] inherit SV_DATA_SOURCE [G]
    -- JSON/XML file backend
```

### Two-Way Binding

```eiffel
class SV_BINDING [G]
    -- Connects UI widget to data model

feature -- Creation
    make (a_widget: SV_WIDGET; a_observable: SV_OBSERVABLE [G])

    make_two_way (a_widget: SV_WIDGET; a_observable: SV_OBSERVABLE [G])
        -- Changes in UI update model, changes in model update UI

feature -- Configuration
    with_transform (a_to_ui: FUNCTION [G, STRING]; a_from_ui: FUNCTION [STRING, G]): like Current
        -- Transform data between model and UI representations

    with_validation (a_validator: PREDICATE [G]): like Current
        -- Validate before accepting changes
```

### Usage Example

```eiffel
-- Model
user_name: SV_OBSERVABLE [STRING]
user_list: SV_DATA_SOURCE [USER]

-- Binding
sv.text_field
    .bind (user_name)  -- Two-way: typing updates model, model changes update field

sv.list
    .bind_items (user_list)  -- Auto-updates when data source changes
    .on_select (agent handle_selection)

-- When data changes anywhere, UI updates automatically
user_name.set_value ("John")  -- Text field updates
user_list.insert (new_user)   -- List updates
```

---

# INNOVATION 2: State Machine UI Pattern

## The Problem
Complex UIs have many states (loading, error, empty, populated, editing). Managing these imperatively leads to spaghetti code.

## The Solution: SV_STATE_MACHINE

```eiffel
class SV_STATE_MACHINE

feature -- States
    define_state (a_name: STRING; a_view: FUNCTION [SV_WIDGET])
        -- Define a state with its view builder

    define_transition (a_from, a_to: STRING; a_trigger: STRING)
        -- Define allowed transitions

feature -- Control
    go_to (a_state: STRING)
        -- Transition to state (validates transition is allowed)

    trigger (a_event: STRING)
        -- Fire event, auto-transition if defined

feature -- Access
    current_state: STRING
    current_view: SV_WIDGET

feature -- Events
    on_state_change: SV_ACTION_SEQUENCE [TUPLE [from_state, to_state: STRING]]
    on_enter_state: HASH_TABLE [SV_ACTION_SEQUENCE [TUPLE], STRING]
    on_exit_state: HASH_TABLE [SV_ACTION_SEQUENCE [TUPLE], STRING]
end
```

### Usage: Data Loading Pattern

```eiffel
-- Define states
state_machine.define_state ("loading", agent loading_view)
state_machine.define_state ("error", agent error_view)
state_machine.define_state ("empty", agent empty_view)
state_machine.define_state ("populated", agent data_view)

-- Define transitions
state_machine.define_transition ("loading", "populated", "data_loaded")
state_machine.define_transition ("loading", "error", "load_failed")
state_machine.define_transition ("loading", "empty", "no_data")
state_machine.define_transition ("error", "loading", "retry")
state_machine.define_transition ("*", "loading", "refresh")  -- From any state

-- Views
loading_view: SV_WIDGET
    do Result := sv.column_of (<<
        sv.progress.indeterminate,
        sv.text ("Loading...")
    >>) end

error_view: SV_WIDGET
    do Result := sv.column_of (<<
        sv.text ("Error: " + last_error).color (sv.colors.error),
        sv.button ("Retry").on_click (agent state_machine.trigger ("retry"))
    >>) end

-- In async load
load_data
    do
        state_machine.go_to ("loading")
        data_service.fetch_async (agent (items: LIST [ITEM])
            do
                if items.is_empty then
                    state_machine.trigger ("no_data")
                else
                    cached_items := items
                    state_machine.trigger ("data_loaded")
                end
            end,
            agent (error: STRING)
            do
                last_error := error
                state_machine.trigger ("load_failed")
            end
        )
    end
```

---

# INNOVATION 3: Component Composition System

## The Problem
Developers repeatedly create the same widget combinations (label + input, icon + text, form groups).

## The Solution: SV_COMPONENT

### Base Component Class

```eiffel
deferred class SV_COMPONENT
    -- Reusable, composable UI component

feature -- Lifecycle
    deferred build: SV_WIDGET
        -- Build the component's widget tree

    on_mount: SV_ACTION_SEQUENCE [TUPLE]
        -- Called when added to UI

    on_unmount: SV_ACTION_SEQUENCE [TUPLE]
        -- Called when removed from UI

feature -- Props (Configuration)
    set_prop (a_name: STRING; a_value: ANY)
    get_prop (a_name: STRING): detachable ANY

feature -- Slots (Child Content)
    set_slot (a_name: STRING; a_content: SV_WIDGET)
    default_slot: detachable SV_WIDGET

feature -- State
    state: SV_OBSERVABLE [ANY]
    set_state (a_key: STRING; a_value: ANY)
    get_state (a_key: STRING): detachable ANY

feature -- Reactivity
    rebuild
        -- Trigger rebuild when state changes
end
```

### Pre-Built Components (2025 Best Practice)

```eiffel
class SV_LABELED_INPUT inherit SV_COMPONENT
    -- Label + Text Field combination

feature -- Props
    label_text: STRING assign set_label_text
    placeholder: STRING assign set_placeholder
    value: SV_OBSERVABLE [STRING]
    is_required: BOOLEAN
    error_message: detachable STRING

feature -- Build
    build: SV_WIDGET
        do
            Result := sv.column
                .spacing (4)
                .children (<<
                    sv.row.children (<<
                        sv.text (label_text).font_weight (600),
                        if is_required then sv.text ("*").color (sv.colors.error) else sv.spacer.fixed (0) end
                    >>),
                    sv.text_field
                        .placeholder (placeholder)
                        .bind (value)
                        .border_color (if error_message /= Void then sv.colors.error else sv.colors.border end),
                    if attached error_message as err then
                        sv.text (err).font_size (12).color (sv.colors.error)
                    else
                        sv.spacer.fixed (0)
                    end
                >>)
                .build
        end
end

class SV_BUTTON_GROUP inherit SV_COMPONENT
    -- Primary + Secondary button row

class SV_AVATAR inherit SV_COMPONENT
    -- Circular image with fallback initials

class SV_BADGE inherit SV_COMPONENT
    -- Small status indicator

class SV_CHIP inherit SV_COMPONENT
    -- Removable tag/filter

class SV_TOAST inherit SV_COMPONENT
    -- Temporary notification

class SV_MODAL inherit SV_COMPONENT
    -- Dialog with backdrop

class SV_DROPDOWN_MENU inherit SV_COMPONENT
    -- Button with popup menu

class SV_SEARCH_BAR inherit SV_COMPONENT
    -- Input + icon + clear button

class SV_PAGINATION inherit SV_COMPONENT
    -- Page navigation controls

class SV_DATA_TABLE inherit SV_COMPONENT
    -- Sortable, filterable table with pagination

class SV_SIDEBAR inherit SV_COMPONENT
    -- Collapsible navigation panel

class SV_BREADCRUMB inherit SV_COMPONENT
    -- Navigation path

class SV_STEPPER inherit SV_COMPONENT
    -- Multi-step wizard progress

class SV_ACCORDION inherit SV_COMPONENT
    -- Collapsible sections

class SV_SKELETON inherit SV_COMPONENT
    -- Loading placeholder
```

### Component Usage

```eiffel
-- Using pre-built components
create name_input.make
name_input.set_label_text ("Full Name")
name_input.set_placeholder ("Enter your name")
name_input.value := user_name_observable
name_input.is_required := True

-- Or fluent style
sv.component (SV_LABELED_INPUT)
    .prop ("label_text", "Email")
    .prop ("placeholder", "you@example.com")
    .prop ("value", email_observable)
    .prop ("is_required", True)
    .build

-- Custom component
class MY_USER_CARD inherit SV_COMPONENT
feature
    user: USER

    build: SV_WIDGET
        do
            Result := sv.card
                .padding (16)
                .content (
                    sv.row
                        .spacing (12)
                        .children (<<
                            sv.component (SV_AVATAR).prop ("name", user.name).prop ("image", user.avatar_url),
                            sv.column.children (<<
                                sv.text (user.name).bold,
                                sv.text (user.email).color (sv.colors.text_secondary)
                            >>)
                        >>)
                )
                .build
        end
end
```

---

# INNOVATION 4: Constraint-Based Layout (Improving EV2 Sizing)

## EV2 Sizing Analysis

From simple_kb, EV2 provides:
- `set_minimum_size`, `set_minimum_width`, `set_minimum_height`
- `disable_item_expand`, `enable_item_expand`, `is_item_expanded`
- `set_padding`, `set_border_width`
- `client_width`, `client_height`

**Problems with EV2:**
1. No maximum size constraints (only minimum)
2. No flexible sizing (flex: 1, flex: 2 ratios)
3. No percentage-based sizing
4. No aspect ratio constraints
5. No responsive breakpoints
6. Verbose expand control

## The Solution: SV_CONSTRAINT System

```eiffel
class SV_CONSTRAINTS
    -- Declarative size constraints

feature -- Fixed Sizes
    width (a_value: INTEGER): like Current
    height (a_value: INTEGER): like Current
    size (a_width, a_height: INTEGER): like Current

feature -- Min/Max
    min_width (a_value: INTEGER): like Current
    max_width (a_value: INTEGER): like Current
    min_height (a_value: INTEGER): like Current
    max_height (a_value: INTEGER): like Current

feature -- Flexible (CSS Flexbox-like)
    expand: like Current
        -- Fill available space (flex: 1)

    expand_ratio (a_ratio: INTEGER): like Current
        -- Relative sizing (flex: a_ratio)

    shrink: like Current
        -- Allow shrinking below preferred size

    no_shrink: like Current
        -- Don't shrink below preferred size

feature -- Percentage-Based
    width_percent (a_percent: REAL): like Current
        -- Percentage of parent width (0.0 to 1.0)

    height_percent (a_percent: REAL): like Current
        -- Percentage of parent height

feature -- Aspect Ratio
    aspect_ratio (a_ratio: REAL): like Current
        -- Maintain width/height ratio (e.g., 16/9)

    square: like Current
        -- 1:1 aspect ratio

feature -- Responsive
    at_width (a_breakpoint: INTEGER; a_constraints: SV_CONSTRAINTS): like Current
        -- Apply different constraints at breakpoint

feature -- Alignment within parent
    align_start: like Current      -- Left/Top
    align_center: like Current     -- Center
    align_end: like Current        -- Right/Bottom
    align_stretch: like Current    -- Fill cross-axis

feature -- Build
    apply_to (a_widget: SV_WIDGET)
end
```

### Responsive Breakpoints

```eiffel
class SV_BREAKPOINTS
feature -- Standard breakpoints (Material Design)
    xs: INTEGER = 0      -- Extra small (phones)
    sm: INTEGER = 600    -- Small (tablets portrait)
    md: INTEGER = 960    -- Medium (tablets landscape)
    lg: INTEGER = 1280   -- Large (desktops)
    xl: INTEGER = 1920   -- Extra large (large desktops)
end
```

### Usage

```eiffel
-- Flexible row with ratio sizing
sv.row.children (<<
    sv.panel.constraints (sv.c.expand_ratio (1)),  -- Takes 1 part
    sv.panel.constraints (sv.c.expand_ratio (2)),  -- Takes 2 parts
    sv.panel.constraints (sv.c.width (200))        -- Fixed 200px
>>)

-- Responsive layout
sv.column
    .constraints (
        sv.c.width_percent (1.0)  -- Full width by default
            .at_width (sv.breakpoints.md, sv.c.width_percent (0.8).align_center)  -- 80% centered on tablet+
            .at_width (sv.breakpoints.lg, sv.c.max_width (1200).align_center)      -- Max 1200px on desktop
    )

-- Aspect ratio (video player)
sv.drawing_area
    .constraints (sv.c.width_percent (1.0).aspect_ratio (16/9))

-- Square avatar
sv.image ("avatar.png")
    .constraints (sv.c.size (48, 48).square)
```

---

# INNOVATION 5: Comprehensive Form System

## The Solution: SV_FORM + SV_FIELD + SV_VALIDATION

### Form Class

```eiffel
class SV_FORM

feature -- Structure
    add_field (a_field: SV_FIELD [ANY])
    add_group (a_name: STRING; a_fields: ARRAY [SV_FIELD [ANY]])
    add_section (a_title: STRING; a_content: SV_WIDGET)

feature -- Values
    values: JSON_OBJECT
        -- All field values as JSON

    set_values (a_json: JSON_OBJECT)
        -- Populate form from JSON

    reset
        -- Clear all fields to defaults

feature -- Validation
    is_valid: BOOLEAN
    errors: HASH_TABLE [STRING, STRING]  -- field_name -> error message

    validate: BOOLEAN
        -- Run all validations, return True if valid

    validate_field (a_name: STRING): BOOLEAN
        -- Validate single field

feature -- Submission
    on_submit: SV_ACTION_SEQUENCE [TUPLE [values: JSON_OBJECT]]
    on_error: SV_ACTION_SEQUENCE [TUPLE [errors: HASH_TABLE [STRING, STRING]]]

    submit
        -- Validate and fire on_submit or on_error

feature -- State
    is_dirty: BOOLEAN
        -- Have values changed since last submit/reset?

    is_submitting: BOOLEAN
        -- Async submission in progress?

feature -- UI
    build: SV_WIDGET
        -- Auto-generate form UI from fields
end
```

### Field Class

```eiffel
class SV_FIELD [G]

feature -- Identity
    name: STRING
    label: STRING

feature -- Value
    value: SV_OBSERVABLE [G]
    default_value: G

feature -- Validation
    add_rule (a_rule: SV_VALIDATION_RULE [G]): like Current

    -- Built-in rules
    required: like Current
    min_length (a_min: INTEGER): like Current
    max_length (a_max: INTEGER): like Current
    pattern (a_regex: STRING): like Current
    email: like Current
    url: like Current
    number: like Current
    min_value (a_min: NUMERIC): like Current
    max_value (a_max: NUMERIC): like Current
    custom (a_validator: PREDICATE [G]; a_message: STRING): like Current

feature -- UI Hints
    placeholder: STRING
    help_text: STRING
    input_type: STRING  -- text, password, email, number, date, etc.
    widget_type: STRING  -- text_field, text_area, dropdown, radio_group, etc.
    options: ARRAY [TUPLE [value: ANY; label: STRING]]  -- For dropdowns/radios

feature -- State
    is_touched: BOOLEAN
    is_valid: BOOLEAN
    error_message: detachable STRING
end
```

### Validation Rules

```eiffel
deferred class SV_VALIDATION_RULE [G]
feature
    deferred validate (a_value: G): BOOLEAN
    deferred message: STRING
end

-- Built-in rules
class SV_REQUIRED_RULE inherit SV_VALIDATION_RULE [ANY]
class SV_MIN_LENGTH_RULE inherit SV_VALIDATION_RULE [STRING]
class SV_MAX_LENGTH_RULE inherit SV_VALIDATION_RULE [STRING]
class SV_PATTERN_RULE inherit SV_VALIDATION_RULE [STRING]
class SV_EMAIL_RULE inherit SV_VALIDATION_RULE [STRING]
class SV_RANGE_RULE inherit SV_VALIDATION_RULE [NUMERIC]

-- Async validation (e.g., check username availability)
class SV_ASYNC_RULE [G] inherit SV_VALIDATION_RULE [G]
feature
    validate_async (a_value: G; a_callback: PROCEDURE [BOOLEAN])
end
```

### Usage

```eiffel
-- Define form
create registration_form.make
registration_form.add_field (
    create {SV_FIELD [STRING]}.make ("username")
        .label ("Username")
        .required
        .min_length (3)
        .max_length (20)
        .pattern ("^[a-zA-Z0-9_]+$")
        .custom (agent is_username_available, "Username already taken")
)
registration_form.add_field (
    create {SV_FIELD [STRING]}.make ("email")
        .label ("Email Address")
        .required
        .email
)
registration_form.add_field (
    create {SV_FIELD [STRING]}.make ("password")
        .label ("Password")
        .input_type ("password")
        .required
        .min_length (8)
        .custom (agent has_special_char, "Must contain special character")
)
registration_form.add_field (
    create {SV_FIELD [STRING]}.make ("country")
        .label ("Country")
        .widget_type ("dropdown")
        .options (country_options)
        .required
)

registration_form.on_submit.extend (agent handle_registration)
registration_form.on_error.extend (agent show_errors)

-- Auto-generate UI
window.content (registration_form.build)

-- Or custom layout
window.content (
    sv.card.title ("Register").content (
        sv.column.spacing (16).children (<<
            registration_form.field ("username").as_widget,
            registration_form.field ("email").as_widget,
            registration_form.field ("password").as_widget,
            registration_form.field ("country").as_widget,
            sv.row.align_right.children (<<
                sv.button ("Cancel").secondary.on_click (agent close),
                sv.button ("Register").primary.on_click (agent registration_form.submit)
            >>)
        >>)
    )
)
```

---

# INNOVATION 6: Navigation/Routing System

## The Solution: SV_NAVIGATOR + SV_ROUTE

```eiffel
class SV_NAVIGATOR

feature -- Routes
    add_route (a_path: STRING; a_builder: FUNCTION [TUPLE, SV_WIDGET])
    add_route_with_params (a_path: STRING; a_builder: FUNCTION [TUPLE [params: HASH_TABLE [STRING, STRING]], SV_WIDGET])

    -- Route patterns
    -- "/users"           -> exact match
    -- "/users/:id"       -> parameter capture
    -- "/users/:id/edit"  -> nested parameters
    -- "/files/*"         -> wildcard

feature -- Navigation
    go_to (a_path: STRING)
    go_to_with_params (a_path: STRING; a_params: HASH_TABLE [STRING, STRING])
    go_back
    go_forward
    replace (a_path: STRING)
        -- Replace current route without adding to history

feature -- Guards
    add_guard (a_path_pattern: STRING; a_guard: FUNCTION [TUPLE, BOOLEAN])
        -- Return False to block navigation (e.g., auth check)

    add_redirect (a_from, a_to: STRING)

feature -- State
    current_path: STRING
    current_params: HASH_TABLE [STRING, STRING]
    can_go_back: BOOLEAN
    can_go_forward: BOOLEAN
    history: LIST [STRING]

feature -- Events
    on_navigate: SV_ACTION_SEQUENCE [TUPLE [from_path, to_path: STRING]]
    on_not_found: SV_ACTION_SEQUENCE [TUPLE [path: STRING]]

feature -- UI
    outlet: SV_WIDGET
        -- Container that displays current route's view
end
```

### Nested Navigation

```eiffel
class SV_NESTED_NAVIGATOR inherit SV_NAVIGATOR
    -- For tab-based or sidebar navigation within a route
end
```

### Usage

```eiffel
-- Setup routes
navigator.add_route ("/", agent home_view)
navigator.add_route ("/login", agent login_view)
navigator.add_route ("/dashboard", agent dashboard_view)
navigator.add_route ("/users", agent users_list_view)
navigator.add_route_with_params ("/users/:id", agent user_detail_view)
navigator.add_route_with_params ("/users/:id/edit", agent user_edit_view)

-- Auth guard
navigator.add_guard ("/dashboard", agent is_authenticated)
navigator.add_guard ("/users/*", agent is_authenticated)

-- Redirect
navigator.add_redirect ("/home", "/")

-- 404 handler
navigator.on_not_found.extend (agent show_not_found)

-- Window with navigation
sv.window ("My App")
    .content (
        sv.column.children (<<
            sv.component (SV_NAVBAR)
                .prop ("items", nav_items)
                .prop ("on_navigate", agent navigator.go_to),
            navigator.outlet.constraints (sv.c.expand)
        >>)
    )

-- Navigate programmatically
save_button.on_click.extend (agent
    do
        save_user
        navigator.go_to ("/users")
    end
)

-- With params
user_row.on_click.extend (agent (user: USER)
    do
        navigator.go_to ("/users/" + user.id.out)
    end
)
```

---

# INNOVATION 7: AI-Powered UI Generation

## What This Means

Using LLMs (like Claude, GPT, or local Ollama), developers can:
1. **Generate UI from natural language** - "Create a login form with username and password"
2. **Modify existing UI** - "Add a forgot password link below the form"
3. **Generate components from descriptions** - "Make a user card showing avatar, name, and email"
4. **Auto-layout suggestions** - "Arrange these fields in a responsive grid"

## The Solution: SV_AI_BUILDER

```eiffel
class SV_AI_BUILDER

feature -- Configuration
    set_provider (a_provider: SV_AI_PROVIDER)
        -- Claude, GPT, Ollama, etc.

feature -- Generation
    generate_ui (a_prompt: STRING): SV_WIDGET
        -- "Create a login form with email and password fields"
        -- Returns: Ready-to-use widget tree

    generate_component (a_prompt: STRING): SV_COMPONENT
        -- "Create a user profile card component"
        -- Returns: Reusable component class

    generate_form (a_description: STRING): SV_FORM
        -- "Registration form with name, email, password, confirm password, and terms checkbox"
        -- Returns: Complete form with validation

    suggest_layout (a_widgets: ARRAY [SV_WIDGET]; a_context: STRING): SV_WIDGET
        -- Arrange widgets intelligently based on context

feature -- Modification
    modify_ui (a_widget: SV_WIDGET; a_instruction: STRING): SV_WIDGET
        -- "Add a loading spinner to this button"
        -- "Make this form horizontal on desktop"

feature -- Code Generation
    generate_eiffel_code (a_prompt: STRING): STRING
        -- Generate actual Eiffel code for complex UIs

feature -- Interactive
    chat_builder: SV_AI_CHAT_BUILDER
        -- Conversational UI building
end

class SV_AI_CHAT_BUILDER
feature
    send (a_message: STRING): SV_WIDGET
        -- Iteratively build UI through conversation

    context: LIST [TUPLE [role: STRING; content: STRING]]
        -- Conversation history

    current_ui: SV_WIDGET
        -- Current state of generated UI

    undo
    redo
end
```

### Provider Abstraction (Reusing simple_tui's AI work)

```eiffel
deferred class SV_AI_PROVIDER
feature
    deferred generate (a_prompt: STRING; a_context: STRING): STRING
end

class SV_CLAUDE_PROVIDER inherit SV_AI_PROVIDER
    -- Uses ANTHROPIC_API_KEY

class SV_GPT_PROVIDER inherit SV_AI_PROVIDER
    -- Uses OPENAI_API_KEY

class SV_OLLAMA_PROVIDER inherit SV_AI_PROVIDER
    -- Local Ollama at localhost:11434
```

### Usage

```eiffel
-- Quick generation
ai := sv.ai_builder
ai.set_provider (create {SV_CLAUDE_PROVIDER})

login_form := ai.generate_ui ("
    Create a login form with:
    - Email field (required, email validation)
    - Password field (required, min 8 chars)
    - Remember me checkbox
    - Login button (primary)
    - Forgot password link
    - Card container with subtle shadow
")

window.content (login_form)

-- Interactive building
builder := ai.chat_builder
builder.send ("Create a dashboard layout")
-- Returns: Basic dashboard structure

builder.send ("Add a sidebar with navigation")
-- Modifies current UI

builder.send ("The sidebar should collapse on mobile")
-- Adds responsive behavior

final_ui := builder.current_ui
```

---

# INNOVATION 8: Design Token System

## What Design Tokens Are

Design tokens are **named constants** for visual properties:
- Colors (primary, secondary, error, success)
- Typography (font sizes, weights, line heights)
- Spacing (padding, margins, gaps)
- Shadows, borders, radii
- Animations (durations, easings)

Tokens enable:
1. **Consistent theming** across the app
2. **Dark mode** by swapping token values
3. **Brand customization** without code changes
4. **Design system integration**

## The Solution: SV_DESIGN_SYSTEM + SV_TOKENS

```eiffel
class SV_TOKENS
    -- Semantic token definitions

feature -- Colors (Semantic)
    primary: SV_COLOR
    primary_variant: SV_COLOR
    secondary: SV_COLOR
    secondary_variant: SV_COLOR

    background: SV_COLOR
    surface: SV_COLOR
    surface_variant: SV_COLOR

    error: SV_COLOR
    warning: SV_COLOR
    success: SV_COLOR
    info: SV_COLOR

    on_primary: SV_COLOR      -- Text on primary background
    on_secondary: SV_COLOR
    on_background: SV_COLOR
    on_surface: SV_COLOR
    on_error: SV_COLOR

    text_primary: SV_COLOR
    text_secondary: SV_COLOR
    text_disabled: SV_COLOR
    text_hint: SV_COLOR

    border: SV_COLOR
    divider: SV_COLOR

feature -- Typography
    font_family: STRING
    font_family_mono: STRING

    font_size_xs: INTEGER     -- 12
    font_size_sm: INTEGER     -- 14
    font_size_md: INTEGER     -- 16
    font_size_lg: INTEGER     -- 18
    font_size_xl: INTEGER     -- 24
    font_size_2xl: INTEGER    -- 32
    font_size_3xl: INTEGER    -- 48

    font_weight_normal: INTEGER  -- 400
    font_weight_medium: INTEGER  -- 500
    font_weight_bold: INTEGER    -- 700

    line_height_tight: REAL   -- 1.25
    line_height_normal: REAL  -- 1.5
    line_height_relaxed: REAL -- 1.75

feature -- Spacing
    space_xs: INTEGER   -- 4
    space_sm: INTEGER   -- 8
    space_md: INTEGER   -- 16
    space_lg: INTEGER   -- 24
    space_xl: INTEGER   -- 32
    space_2xl: INTEGER  -- 48

feature -- Borders
    border_radius_none: INTEGER   -- 0
    border_radius_sm: INTEGER     -- 4
    border_radius_md: INTEGER     -- 8
    border_radius_lg: INTEGER     -- 16
    border_radius_full: INTEGER   -- 9999 (circular)

    border_width_thin: INTEGER    -- 1
    border_width_medium: INTEGER  -- 2
    border_width_thick: INTEGER   -- 4

feature -- Shadows (EV2 doesn't support, but we can fake with borders/colors)
    shadow_sm: SV_SHADOW
    shadow_md: SV_SHADOW
    shadow_lg: SV_SHADOW
    shadow_xl: SV_SHADOW

feature -- Animation
    duration_fast: INTEGER     -- 150ms
    duration_normal: INTEGER   -- 300ms
    duration_slow: INTEGER     -- 500ms

    easing_default: STRING     -- "ease-in-out"
end
```

### Theme Class

```eiffel
class SV_THEME

feature -- Tokens
    tokens: SV_TOKENS

feature -- Modes
    is_dark_mode: BOOLEAN
    toggle_dark_mode
    set_dark_mode (a_dark: BOOLEAN)

feature -- Theme Loading
    load_from_json (a_path: STRING)
    load_material_light
    load_material_dark
    load_custom (a_tokens: SV_TOKENS)

feature -- Observers
    on_theme_change: SV_ACTION_SEQUENCE [TUPLE [old_tokens, new_tokens: SV_TOKENS]]

feature -- Singleton
    current: SV_THEME
        once
            create Result.make_material_light
        end
end
```

### Pre-Built Themes

```eiffel
class SV_MATERIAL_LIGHT_TOKENS inherit SV_TOKENS
    -- Material Design 3 Light Theme
feature
    primary: SV_COLOR once Result := sv.color.from_hex ("#6750A4") end
    on_primary: SV_COLOR once Result := sv.color.from_hex ("#FFFFFF") end
    background: SV_COLOR once Result := sv.color.from_hex ("#FFFBFE") end
    surface: SV_COLOR once Result := sv.color.from_hex ("#FFFBFE") end
    -- ... etc
end

class SV_MATERIAL_DARK_TOKENS inherit SV_TOKENS
    -- Material Design 3 Dark Theme
feature
    primary: SV_COLOR once Result := sv.color.from_hex ("#D0BCFF") end
    on_primary: SV_COLOR once Result := sv.color.from_hex ("#381E72") end
    background: SV_COLOR once Result := sv.color.from_hex ("#1C1B1F") end
    surface: SV_COLOR once Result := sv.color.from_hex ("#1C1B1F") end
    -- ... etc
end
```

### Usage

```eiffel
-- All widgets automatically use tokens
sv.button ("Click")
    -- Automatically uses tokens.primary for background
    -- Uses tokens.on_primary for text
    -- Uses tokens.border_radius_md for corners

-- Override for specific widget
sv.button ("Danger")
    .background (sv.tokens.error)
    .text_color (sv.tokens.on_error)

-- Theme switching
dark_mode_toggle.on_change.extend (agent (is_on: BOOLEAN)
    do
        SV_THEME.current.set_dark_mode (is_on)
        -- All widgets automatically update!
    end
)

-- Custom theme
my_brand := create {SV_TOKENS}
my_brand.primary := sv.color.from_hex ("#FF5722")  -- Orange brand color
my_brand.font_family := "Inter"
SV_THEME.current.load_custom (my_brand)
```

---

# INNOVATION 9: Graphics Enhancement Layer

## The Problem
EV2's drawing is basic and dated. No gradients, shadows, blur, or modern effects.

## The Solution: SV_GRAPHICS

```eiffel
class SV_GRAPHICS

feature -- Gradients
    linear_gradient (a_start, a_end: SV_COLOR; a_angle: REAL): SV_GRADIENT
    radial_gradient (a_center, a_outer: SV_COLOR): SV_GRADIENT

feature -- Shadows (Simulated)
    drop_shadow (a_offset_x, a_offset_y, a_blur: INTEGER; a_color: SV_COLOR): SV_SHADOW
    inner_shadow (a_offset_x, a_offset_y, a_blur: INTEGER; a_color: SV_COLOR): SV_SHADOW

feature -- Rounded Corners
    rounded_rect (a_radius: INTEGER): SV_PATH
    pill_shape: SV_PATH

feature -- Icons (Built-in icon set)
    icon (a_name: STRING): SV_ICON
        -- "check", "close", "menu", "search", "arrow_left", etc.
        -- Uses scalable vector icons

feature -- Modern Shapes
    circle (a_radius: INTEGER): SV_SHAPE
    oval (a_width, a_height: INTEGER): SV_SHAPE
    rounded_rect (a_width, a_height, a_radius: INTEGER): SV_SHAPE

feature -- Transformations
    rotate (a_degrees: REAL): SV_TRANSFORM
    scale (a_factor: REAL): SV_TRANSFORM
    translate (a_x, a_y: INTEGER): SV_TRANSFORM

feature -- Animation (Simple)
    animate_property (a_widget: SV_WIDGET; a_property: STRING; a_from, a_to: ANY; a_duration: INTEGER)
    fade_in (a_widget: SV_WIDGET; a_duration: INTEGER)
    fade_out (a_widget: SV_WIDGET; a_duration: INTEGER)
    slide_in (a_widget: SV_WIDGET; a_direction: STRING; a_duration: INTEGER)
end
```

### Icon System

```eiffel
class SV_ICONS
    -- Built-in icon library (Material-style)

feature -- Navigation
    menu, arrow_left, arrow_right, arrow_up, arrow_down,
    chevron_left, chevron_right, chevron_up, chevron_down,
    home, search, settings, close

feature -- Actions
    add, remove, edit, delete, save, refresh, download, upload,
    copy, paste, cut, undo, redo, check, check_circle

feature -- Status
    info, warning, error, success, help, notification, loading

feature -- Content
    file, folder, image, video, audio, document, link

feature -- Social
    user, users, chat, mail, phone, calendar, star, heart

feature -- Each returns SV_ICON
    menu: SV_ICON do Result := load_icon ("menu") end
    -- ...
end
```

---

# INNOVATION 10: Additional Industry Pain Point Solutions

## 10.1 Accessibility (A11y) Built-In

```eiffel
class SV_ACCESSIBILITY
feature
    set_label (a_widget: SV_WIDGET; a_label: STRING)
        -- Screen reader label

    set_hint (a_widget: SV_WIDGET; a_hint: STRING)
        -- Usage hint for screen readers

    set_role (a_widget: SV_WIDGET; a_role: STRING)
        -- "button", "link", "heading", etc.

    focus_next
    focus_previous

    keyboard_shortcuts: SV_KEYBOARD_SHORTCUTS
end

-- Widgets auto-announce
sv.button ("Save")
    -- Automatically accessible with label "Save"
    -- Keyboard focusable
    -- Announces state changes
```

## 10.2 Internationalization (i18n) — Uses simple_i18n

**Note:** simple_vision leverages the existing **simple_i18n** library (D:\prod\simple_i18n).

```eiffel
-- simple_i18n already provides:
class SIMPLE_I18N
feature
    -- Translation
    translate (a_text: STRING): STRING              -- t() alias
    translate_plural (a_singular, a_plural: STRING; a_count: INTEGER): STRING

    -- Locale management
    current_locale_id: I18N_LOCALE_ID
    set_locale (a_locale_code: STRING)

    -- Formatting (locale-aware)
    format_date (a_date: DATE): STRING
    format_time (a_time: TIME): STRING
    format_currency (a_value: REAL_64): STRING
    format_number (a_value: REAL_64): STRING

    -- .mo file loading
    make_from_directory (a_translations_dir: STRING)
end

-- Integration with simple_vision:
sv.text (i18n.translate ("welcome_message"))
sv.text (i18n.translate_plural ("item", "items", item_count))
sv.label (i18n.format_currency (product.price))
```

simple_vision adds locale change notifications for UI refresh:

```eiffel
SV_I18N_BRIDGE
feature
    on_locale_change: SV_ACTION_SEQUENCE [TUPLE [old_locale, new_locale: STRING]]
        -- Fires when locale changes; widgets can subscribe to refresh
end
```

## 10.3 Undo/Redo Stack

```eiffel
class SV_UNDO_MANAGER
feature
    execute (a_command: SV_COMMAND)
    undo
    redo
    can_undo: BOOLEAN
    can_redo: BOOLEAN
    clear_history

    on_change: SV_ACTION_SEQUENCE [TUPLE]
end

deferred class SV_COMMAND
feature
    deferred execute
    deferred undo
    description: STRING
end
```

## 10.4 Pick-and-Drop / Drag-and-Drop — Leverages EV2 Native

**EV2 has a sophisticated "Pick-and-Drop" system** via `EV_PICK_AND_DROPABLE`:

### EV2 Native Features (Often Unknown)

| Feature | Description |
|---------|-------------|
| `set_pebble` | Set the data object to transport |
| `pebble_function` | Dynamic pebble generation |
| `drop_actions` | Agents called when drop received |
| `accept_cursor` / `deny_cursor` | Visual feedback |
| **Three Modes** | Pick-and-drop, Drag-and-drop, Target menu |

### Three Transport Modes

```eiffel
-- 1. Pick and Drop (default): Right-click pick, right-click drop
widget.set_pick_and_drop_mode

-- 2. Drag and Drop: Left-click hold, release to drop
widget.set_drag_and_drop_mode

-- 3. Target Menu: Right-click shows menu of all valid targets
widget.set_target_menu_mode
```

### Type-Safe Transport
EV2 uses Eiffel's type system — drop only succeeds if target's agent accepts pebble type:

```eiffel
-- Source: any STRING can be picked
button1.set_pebble ("my data")

-- Target: only accepts STRING pebbles
button2.drop_actions.extend (agent handle_string_drop)

handle_string_drop (a_data: STRING)
    do
        print ("Received: " + a_data)
    end
```

### simple_vision Enhancement (SV_DRAG_DROP)

```eiffel
class SV_DRAG_DROP
    -- Simplified wrapper over EV_PICK_AND_DROPABLE

feature -- Setup (Fluent)
    draggable (a_data: ANY): like Current
        -- Make widget a drag source with this data

    drop_target (a_handler: PROCEDURE [ANY]): like Current
        -- Make widget accept drops

    mode (a_mode: INTEGER): like Current
        -- sv.drag.pick_and_drop, sv.drag.drag_and_drop, sv.drag.target_menu

feature -- Cursors
    accept_cursor (a_pixmap: EV_PIXMAP): like Current
    deny_cursor (a_pixmap: EV_PIXMAP): like Current

feature -- Events
    on_drag_start: SV_ACTION_SEQUENCE [TUPLE [data: ANY]]
    on_drag_over: SV_ACTION_SEQUENCE [TUPLE [target: SV_WIDGET; data: ANY]]
    on_drop: SV_ACTION_SEQUENCE [TUPLE [target: SV_WIDGET; data: ANY]]
    on_drag_cancel: SV_ACTION_SEQUENCE [TUPLE]
end

-- Usage
sv.button ("Drag Me")
    .draggable (my_object)
    .mode (sv.drag.drag_and_drop)

sv.panel
    .drop_target (agent handle_drop)
    .accept_cursor (sv.cursors.bullseye)
```

## 10.5 Clipboard — Uses simple_clipboard

**Note:** simple_vision uses the existing **simple_clipboard** library (D:\prod\simple_clipboard).

```eiffel
-- simple_clipboard already provides:
class SIMPLE_CLIPBOARD
feature
    -- Reading
    text: detachable STRING_32       -- Get text from clipboard
    has_text: BOOLEAN                -- Check if text available
    is_empty: BOOLEAN                -- Check if clipboard empty
    format_count: INTEGER            -- Number of formats available

    -- Writing
    set_text (a_text: STRING)        -- Put text on clipboard
    copy_text (a_text: STRING)       -- Alias for set_text
    clear                            -- Clear clipboard

    -- Aliases
    paste: detachable STRING_32      -- Alias for text
end

-- Integration with simple_vision widgets:
sv.text_field
    .context_menu (<<
        sv.menu_item ("Cut").hotkey ("Ctrl+X").on_click (agent do
            clipboard.set_text (text_field.selected_text)
            text_field.delete_selection
        end),
        sv.menu_item ("Copy").hotkey ("Ctrl+C").on_click (agent do
            clipboard.set_text (text_field.selected_text)
        end),
        sv.menu_item ("Paste").hotkey ("Ctrl+V").on_click (agent do
            if attached clipboard.text as t then
                text_field.insert_text (t)
            end
        end)
    >>)
```

## 10.6 Print/PDF Export — Uses simple_pdf

**Note:** simple_vision uses the existing **simple_pdf** library (D:\prod\simple_pdf).

```eiffel
-- simple_pdf provides comprehensive PDF generation:
class SIMPLE_PDF
feature
    -- Document creation
    create_document (a_path: STRING)
    add_page
    save

    -- Text
    set_font (a_name: STRING; a_size: REAL)
    draw_text (a_text: STRING; a_x, a_y: REAL)
    draw_text_wrapped (a_text: STRING; a_x, a_y, a_width: REAL)

    -- Graphics
    draw_line (x1, y1, x2, y2: REAL)
    draw_rectangle (x, y, w, h: REAL)
    draw_image (a_path: STRING; x, y, w, h: REAL)

    -- Tables
    draw_table (a_data: ARRAY2 [STRING]; x, y: REAL)
end

-- Integration with simple_vision:
class SV_PRINT
feature
    export_widget_to_pdf (a_widget: SV_WIDGET; a_path: STRING)
        -- Render widget tree to PDF

    export_data_grid_to_pdf (a_grid: SV_DATA_GRID; a_path: STRING)
        -- Export grid data to formatted PDF table

    print_preview (a_widget: SV_WIDGET): SV_DIALOG
        -- Show print preview dialog

    print_widget (a_widget: SV_WIDGET)
        -- Send to system printer
end
```

## 10.7 Context Menus Made Easy

```eiffel
sv.text_field
    .context_menu (<<
        sv.menu_item ("Cut").icon (sv.icons.cut).on_click (agent cut),
        sv.menu_item ("Copy").icon (sv.icons.copy).on_click (agent copy),
        sv.menu_item ("Paste").icon (sv.icons.paste).on_click (agent paste),
        sv.menu_separator,
        sv.menu_item ("Select All").on_click (agent select_all)
    >>)
```

## 10.8 Tooltips with Rich Content

```eiffel
sv.button ("Info")
    .tooltip ("Simple text tooltip")

sv.button ("Help")
    .rich_tooltip (
        sv.column.children (<<
            sv.text ("Keyboard Shortcuts").bold,
            sv.text ("Ctrl+S - Save"),
            sv.text ("Ctrl+Z - Undo")
        >>)
    )
```

## 10.9 Virtual Lists (For Large Data)

```eiffel
class SV_VIRTUAL_LIST [G]
    -- Only renders visible items (performance for 10K+ items)
feature
    set_items (a_items: LIST [G])
    set_item_height (a_height: INTEGER)
    set_item_builder (a_builder: FUNCTION [G, SV_WIDGET])

    visible_range: TUPLE [start_index, end_index: INTEGER]
    scroll_to_index (a_index: INTEGER)
end
```

## 10.10 Hot Keys / Keyboard Shortcuts

```eiffel
sv.window ("Editor")
    .hotkey ("Ctrl+S", agent save_file)
    .hotkey ("Ctrl+Z", agent undo)
    .hotkey ("Ctrl+Shift+Z", agent redo)
    .hotkey ("F1", agent show_help)
    .hotkey ("Escape", agent close_dialog)
```

---

# Summary: The simple_vision Innovation Stack

| # | Innovation | What It Does | Industry Inspiration |
|---|------------|--------------|---------------------|
| 1 | **Reactive Binding** | Auto-update UI when data changes | React, Vue, SwiftUI |
| 2 | **State Machine** | Declarative UI states | XState, Statecharts |
| 3 | **Components** | Reusable, composable pieces | React Components |
| 4 | **Constraints** | Modern layout system | CSS Flexbox, Auto Layout |
| 5 | **Forms** | Complete form handling | React Hook Form, Formik |
| 6 | **Navigation** | Page routing | React Router, Vue Router |
| 7 | **AI Builder** | Generate UI from prompts | Galileo AI, Vercel v0 |
| 8 | **Design Tokens** | Semantic theming | Material Design 3 |
| 9 | **Graphics** | Modern visual effects | CSS3, CoreGraphics |
| 10 | **Extras** | a11y, i18n*, undo, drag-drop, etc. | WCAG, react-intl |
| 11 | **EV_GRID Enhancement** | Data grids with lazy loading, virtual scrolling, 1M+ rows | AG Grid, TanStack Table |

*i18n via existing simple_i18n library

---

# ECOSYSTEM INTEGRATION: Leveraging simple_* Libraries

simple_vision doesn't reinvent the wheel — it integrates with the existing simple_* ecosystem.

## Direct Dependencies

| Library | Purpose in simple_vision |
|---------|-------------------------|
| **simple_clipboard** | Cut/copy/paste operations in text widgets |
| **simple_i18n** | Internationalization, locale-aware formatting |
| **simple_pdf** | PDF export of widgets and data grids |
| **simple_validation** | Form field validation rules |
| **simple_json** | Theme loading, state serialization |
| **simple_config** | Application settings persistence |
| **simple_logger** | Debug logging, error tracking |

## Data Source Backends (SV_DATA_SOURCE implementations)

| Library | Data Source Class |
|---------|------------------|
| **simple_sql** | `SV_SQL_DATA_SOURCE` — SQLite/database backend |
| **simple_http** | `SV_REST_DATA_SOURCE` — REST API backend |
| **simple_websocket** | `SV_REALTIME_DATA_SOURCE` — Live updates |
| **simple_json** | `SV_JSON_FILE_DATA_SOURCE` — JSON file storage |
| **simple_xml** | `SV_XML_DATA_SOURCE` — XML file storage |
| **simple_yaml** | `SV_YAML_DATA_SOURCE` — YAML config files |
| **simple_csv** | `SV_CSV_DATA_SOURCE` — CSV import/export |

## AI Integration (SV_AI_BUILDER providers)

| Library | Provider Class |
|---------|---------------|
| **simple_ai_client** | `SV_CLAUDE_PROVIDER`, `SV_GPT_PROVIDER`, `SV_OLLAMA_PROVIDER` |

## Utility Libraries

| Library | Usage |
|---------|-------|
| **simple_uuid** | Unique IDs for components, widgets |
| **simple_datetime** | Date pickers, time formatting |
| **simple_regex** | Input validation patterns |
| **simple_template** | Dynamic text generation |
| **simple_markdown** | Markdown rendering in rich text |
| **simple_watcher** | File change notifications |
| **simple_encryption** | Secure credential storage |
| **simple_cache** | Response caching for data sources |

## Media Libraries

| Library | Usage |
|---------|-------|
| **simple_ffmpeg** | Video/audio widget support |
| **simple_audio** | Sound playback, notifications |

## Example: Full Stack Integration

```eiffel
class MY_APP

feature -- Dependencies (all from simple_* ecosystem)
    db: SIMPLE_SQL              -- Database access
    http: SIMPLE_HTTP           -- REST API client
    config: SIMPLE_CONFIG       -- App settings
    logger: SIMPLE_LOGGER       -- Logging
    i18n: SIMPLE_I18N           -- Translations
    clipboard: SIMPLE_CLIPBOARD -- Clipboard ops
    pdf: SIMPLE_PDF             -- PDF export
    validator: SIMPLE_VALIDATION -- Form validation
    ai: SIMPLE_AI_CLIENT        -- AI features

feature -- Data Sources
    users_source: SV_SQL_DATA_SOURCE [USER]
        do
            create Result.make (db, "SELECT * FROM users")
        end

    api_source: SV_REST_DATA_SOURCE [PRODUCT]
        do
            create Result.make (http, "https://api.example.com/products")
        end

feature -- UI
    main_window: SV_WINDOW
        do
            Result := sv.window (i18n.translate ("app_title"))
                .content (
                    sv.data_grid [USER]
                        .bind (users_source)  -- Reactive binding to SQL
                        .columns (user_columns)
                        .toolbar (<<
                            sv.button (i18n.translate ("export_pdf"))
                                .on_click (agent export_to_pdf),
                            sv.button (i18n.translate ("add_user"))
                                .on_click (agent add_user)
                        >>)
                )
                .build
        end

    export_to_pdf
        do
            pdf.export_data_grid (main_grid, "users.pdf")
        end
end
```

## SV_* Wrapper Classes for Ecosystem Integration

Each simple_* library gets an SV_* wrapper that provides GUI-friendly integration:

### Data Source Wrappers

| simple_* Library | SV_* Class | Purpose |
|------------------|------------|---------|
| simple_sql | `SV_SQL_DATA_SOURCE [G]` | Reactive SQL queries, auto-refresh on change |
| simple_http | `SV_REST_DATA_SOURCE [G]` | REST API binding with pagination, caching |
| simple_websocket | `SV_REALTIME_DATA_SOURCE [G]` | Live data streaming to grids/lists |
| simple_json | `SV_JSON_DATA_SOURCE [G]` | JSON file as reactive data store |
| simple_xml | `SV_XML_DATA_SOURCE [G]` | XML file binding |
| simple_yaml | `SV_YAML_DATA_SOURCE [G]` | YAML config as data source |
| simple_csv | `SV_CSV_DATA_SOURCE [G]` | CSV import/export with grid binding |

### UI Utility Wrappers

| simple_* Library | SV_* Class | Purpose |
|------------------|------------|---------|
| simple_i18n | `SV_I18N_BRIDGE` | Locale change events, widget refresh |
| simple_clipboard | `SV_CLIPBOARD` | Widget-aware cut/copy/paste actions |
| simple_pdf | `SV_PDF_EXPORTER` | Export widgets, grids, forms to PDF |
| simple_validation | `SV_VALIDATOR` | Rule-based field validation with UI feedback |
| simple_config | `SV_SETTINGS` | Persistent app settings with change notification |
| simple_logger | `SV_DEBUG_CONSOLE` | Visual log viewer widget |
| simple_uuid | `SV_ID_GENERATOR` | Unique IDs for widget tracking |
| simple_datetime | `SV_DATE_PICKER`, `SV_TIME_PICKER`, `SV_CALENDAR` | Date/time input widgets |
| simple_regex | `SV_PATTERN_VALIDATOR` | Regex-based input validation |
| simple_template | `SV_TEMPLATE_TEXT` | Dynamic text labels with variable substitution |
| simple_markdown | `SV_MARKDOWN_VIEW` | Rendered markdown display widget |
| simple_watcher | `SV_FILE_WATCHER` | Auto-reload UI when files change |
| simple_encryption | `SV_SECURE_FIELD` | Encrypted credential storage |
| simple_cache | `SV_CACHED_DATA_SOURCE [G]` | Caching layer for any data source |

### AI/Media Wrappers

| simple_* Library | SV_* Class | Purpose |
|------------------|------------|---------|
| simple_ai_client | `SV_AI_BUILDER`, `SV_CLAUDE_PROVIDER`, `SV_GPT_PROVIDER`, `SV_OLLAMA_PROVIDER` | AI-powered UI generation |
| simple_ffmpeg | `SV_VIDEO_PLAYER`, `SV_VIDEO_THUMBNAIL` | Video playback widget, thumbnails |
| simple_audio | `SV_AUDIO_PLAYER`, `SV_SOUND_EFFECT` | Audio playback, notification sounds |

### Example: SV_DATE_PICKER using simple_datetime

```eiffel
class SV_DATE_PICKER
    -- Wraps simple_datetime for visual date selection

inherit
    SV_WIDGET

feature -- Access
    value: SV_OBSERVABLE [DATE]
        -- Selected date (reactive)

    simple_date: SIMPLE_DATE
        -- Underlying simple_datetime object

feature -- Configuration
    min_date (a_date: DATE): like Current
    max_date (a_date: DATE): like Current
    format (a_format: STRING): like Current  -- e.g., "MM/DD/YYYY"
    locale (a_locale: STRING): like Current  -- Uses simple_i18n

feature -- Events
    on_change: SV_ACTION_SEQUENCE [TUPLE [old_date, new_date: DATE]]
end

-- Usage
sv.date_picker
    .format ("YYYY-MM-DD")
    .min_date (today)
    .bind (due_date_observable)
```

### Example: SV_MARKDOWN_VIEW using simple_markdown

```eiffel
class SV_MARKDOWN_VIEW
    -- Renders markdown content using simple_markdown

inherit
    SV_WIDGET

feature -- Content
    set_markdown (a_md: STRING)
        -- Parse and render markdown

    set_markdown_file (a_path: STRING)
        -- Load and render from file

feature -- Configuration
    enable_syntax_highlighting
    disable_syntax_highlighting
    set_code_theme (a_theme: STRING)  -- "dark", "light", "github"
    enable_link_clicking
    on_link_click: SV_ACTION_SEQUENCE [TUPLE [url: STRING]]
end

-- Usage
sv.markdown_view
    .set_markdown ("# Hello World%NThis is **bold** text.")
    .enable_syntax_highlighting
    .on_link_click (agent open_url)
```

### Example: SV_VIDEO_PLAYER using simple_ffmpeg

```eiffel
class SV_VIDEO_PLAYER
    -- Video playback widget using simple_ffmpeg

inherit
    SV_WIDGET

feature -- Playback
    load (a_path: STRING)
    play
    pause
    stop
    seek (a_seconds: REAL_64)

feature -- Status
    is_playing: BOOLEAN
    duration: REAL_64
    current_position: REAL_64

feature -- Configuration
    show_controls: like Current
    hide_controls: like Current
    autoplay: like Current
    loop: like Current
    muted: like Current
    volume (a_percent: REAL): like Current  -- 0.0 to 1.0

feature -- Events
    on_play: SV_ACTION_SEQUENCE [TUPLE]
    on_pause: SV_ACTION_SEQUENCE [TUPLE]
    on_end: SV_ACTION_SEQUENCE [TUPLE]
    on_progress: SV_ACTION_SEQUENCE [TUPLE [position: REAL_64]]
end

-- Usage
sv.video_player
    .load ("intro.mp4")
    .show_controls
    .volume (0.8)
    .on_end (agent play_next_video)
```

## Dependency Graph

```
                    ┌─────────────────┐
                    │  simple_vision  │
                    │    (SV_*)       │
                    └────────┬────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌───────────────┐    ┌───────────────┐    ┌───────────────┐
│ Data Sources  │    │   UI Utils    │    │   AI/Media    │
│  SV_*_SOURCE  │    │    SV_*       │    │    SV_*       │
├───────────────┤    ├───────────────┤    ├───────────────┤
│ simple_sql    │    │ simple_i18n   │    │ simple_ai     │
│ simple_http   │    │ simple_clip   │    │ simple_ffmpeg │
│ simple_ws     │    │ simple_pdf    │    │ simple_audio  │
│ simple_json   │    │ simple_valid  │    └───────────────┘
│ simple_xml    │    │ simple_config │
│ simple_yaml   │    │ simple_logger │
│ simple_csv    │    │ simple_uuid   │
└───────────────┘    │ simple_regex  │
                     │ simple_datetime│
                     │ simple_template│
                     │ simple_markdown│
                     │ simple_watcher │
                     │ simple_encrypt │
                     │ simple_cache   │
                     └───────────────┘
```

---

## Research Sources

- [EiffelVision Pick and Drop](https://www.eiffel.org/doc/solutions/EiffelVision_Pick_and_Drop) — Pebbles and holes metaphor
- [EV_GRID Flat contracts](https://www.eiffel.org/files/doc/static/17.05/libraries/vision2/ev_grid_flatshort.html) — Complete EV_GRID API
- [Stack Overflow 2024 Survey - Developer Pain Points](https://stackoverflow.blog/2024/08/06/2024-developer-survey/)
- [Hacker News - Cross-platform GUI 2024](https://news.ycombinator.com/item?id=38817920)
- [Material Design 3 Tokens](https://m3.material.io/foundations/design-tokens)
- [React Hook Form](https://www.wearedevelopers.com/en/magazine/399/react-form-libraries)
- [ReactiveUI Data Binding](https://www.reactiveui.net/docs/handbook/data-binding/)
- [Galileo AI - UI Generation](https://www.usegalileo.ai/)
- [Vercel AI SDK - Generative UI](https://ai-sdk.dev/docs/ai-sdk-ui/generative-user-interfaces)
- [Design Token System](https://www.contentful.com/blog/design-token-system/)

---

# INNOVATION 11: EV_GRID Deep Enhancement (SV_DATA_GRID)

## The Hidden Power of EV_GRID

EV_GRID is one of EV2's most sophisticated widgets — a combined tree/list/table with **60+ related classes**. Most developers only scratch the surface. simple_vision exposes its full power through a simplified API.

## EV_GRID's Native Capabilities (Often Unknown)

### 1. Lazy Loading / Dynamic Content
EV_GRID has **built-in lazy loading** via `dynamic_content_function`:

```eiffel
-- EV2 Native (Verbose)
my_grid.set_dynamic_content_function (agent create_item_on_demand)
my_grid.enable_partial_dynamic_content
my_grid.set_row_count_to (1_000_000)  -- Supports millions of virtual rows!

create_item_on_demand (a_column, a_row: INTEGER): EV_GRID_ITEM
    do
        -- Only called when row becomes visible
        create {EV_GRID_LABEL_ITEM} Result.make_with_text (fetch_data (a_row))
    end
```

**This means:** You can have a grid with 1 million rows and only the visible ~20 are actually created!

### 2. Virtual Positioning
```eiffel
-- Virtual coordinates (total scrollable area)
virtual_x_position, virtual_y_position
maximum_virtual_x_position, maximum_virtual_y_position
virtual_width, virtual_height

-- Viewport (visible window into virtual space)
viewable_width, viewable_height
viewable_x_offset, viewable_y_offset

-- Navigation
row_at_virtual_position (y_coord)
column_at_virtual_position (x_coord)
first_visible_row, last_visible_row
visible_row_indexes, visible_column_indexes
```

### 3. Rich Item Types

| EV2 Class | Purpose |
|-----------|---------|
| EV_GRID_LABEL_ITEM | Text with optional pixmap |
| EV_GRID_EDITABLE_ITEM | In-place text editing |
| EV_GRID_CHECKBOX_LABEL_ITEM | Checkbox + text |
| EV_GRID_CHOICE_ITEM | Dropdown selection |
| EV_GRID_COMBO_ITEM | Editable dropdown |
| EV_GRID_DRAWABLE_ITEM | Custom drawing |
| EV_GRID_LABEL_ELLIPSIS_ITEM | Truncated text with "..." |
| EV_GRID_PIXMAPS_ON_RIGHT_LABEL_ITEM | Icons on right side |

### 4. Tree Mode
```eiffel
my_grid.enable_tree
-- Now rows can have parent/child relationships
my_grid.insert_new_row_parented (parent_row)
-- With expand/collapse, connectors, indentation
```

### 5. Performance Optimization
```eiffel
-- Batch updates (critical for large changes)
my_grid.lock_update
-- ... make many changes ...
my_grid.unlock_update

-- Fixed row height (faster rendering)
my_grid.enable_row_height_fixed
my_grid.set_row_height (24)

-- Scrolling modes
my_grid.enable_vertical_scrolling_per_item  -- Snap to rows
```

### 6. Overlay Drawing (Custom Rendering)
```eiffel
-- Draw BEFORE item content
item.set_pre_draw_overlay_function (agent draw_background)

-- Draw AFTER item content
item.set_post_draw_overlay_function (agent draw_border)
```

### 7. Locked Rows/Columns
```eiffel
-- Header rows/columns that don't scroll
my_grid.locked_rows
my_grid.locked_columns
```

## The Problem: EV_GRID is Powerful but Complex

- 60+ classes to understand
- Verbose API for common operations
- Easy to miss performance optimizations
- Dynamic content setup is non-obvious

## The Solution: SV_DATA_GRID

### Simplified Lazy Loading

```eiffel
class SV_DATA_GRID [G]
    -- G is the row data type

feature -- Data Source
    set_items (a_items: LIST [G])
        -- Eagerly load all items (for small datasets)

    set_lazy_items (a_count: INTEGER; a_fetcher: FUNCTION [INTEGER, G])
        -- Lazy load: only fetch visible rows
        -- a_fetcher called with row index, returns item data

    set_data_source (a_source: SV_DATA_SOURCE [G])
        -- Full reactive: auto-updates when source changes

feature -- Columns
    add_column (a_title: STRING; a_accessor: FUNCTION [G, ANY]): SV_GRID_COLUMN_BUILDER
        -- Fluent column definition

feature -- Paging
    set_page_size (a_size: INTEGER)
    set_total_count (a_count: INTEGER)
    on_page_change: SV_ACTION_SEQUENCE [TUPLE [page: INTEGER]]
```

### Fluent Column Definitions

```eiffel
sv.data_grid [USER]
    .add_column ("Name", agent {USER}.name)
        .width (200)
        .sortable
        .searchable
    .add_column ("Email", agent {USER}.email)
        .width (250)
        .sortable
    .add_column ("Role", agent {USER}.role)
        .width (100)
        .editable_dropdown (role_options)
    .add_column ("Active", agent {USER}.is_active)
        .width (80)
        .checkbox
    .add_column ("Actions", agent action_cell)
        .width (120)
        .custom_renderer (agent render_action_buttons)
```

### Virtual Scrolling (Millions of Rows)

```eiffel
sv.data_grid [LOG_ENTRY]
    .virtual_mode (1_000_000)  -- 1 million rows
    .fetch_page (agent (a_start, a_count: INTEGER): LIST [LOG_ENTRY]
        do
            Result := database.query_logs (a_start, a_count)
        end)
    .row_height (24)  -- Fixed for performance
```

### Tree Mode Made Easy

```eiffel
sv.tree_grid [FOLDER]
    .children_accessor (agent {FOLDER}.subfolders)
    .is_expandable (agent {FOLDER}.has_subfolders)
    .on_expand (agent load_children)
```

### Built-in Features

```eiffel
sv.data_grid [PRODUCT]
    .columns (product_columns)

    -- Sorting
    .sortable
    .default_sort ("name", ascending)
    .on_sort (agent handle_sort)

    -- Filtering
    .filterable
    .search_columns (<<"name", "sku", "description">>)
    .on_filter (agent handle_filter)

    -- Selection
    .selection_mode (sv.selection.multiple_rows)
    .on_select (agent handle_selection)
    .on_double_click (agent edit_product)

    -- Editing
    .editable
    .on_cell_edit (agent save_change)

    -- Context menu
    .row_context_menu (<<
        sv.menu_item ("Edit").on_click (agent edit_selected),
        sv.menu_item ("Delete").on_click (agent delete_selected),
        sv.menu_separator,
        sv.menu_item ("Export...").on_click (agent export_selected)
    >>)

    -- Performance
    .lock_during_updates
    .fixed_row_height (24)
```

### Server-Side Processing

For truly large datasets, push sorting/filtering to the server:

```eiffel
sv.data_grid [ORDER]
    .server_mode
    .on_request (agent (a_request: SV_GRID_REQUEST): SV_GRID_RESPONSE
        do
            -- a_request contains: page, page_size, sort_column, sort_dir, filters
            Result := api.fetch_orders (a_request.as_query_params)
        end)
```

### Cell Renderers

Custom rendering for special cells:

```eiffel
class SV_CELL_RENDERERS
feature
    -- Built-in renderers
    text: SV_TEXT_RENDERER
    checkbox: SV_CHECKBOX_RENDERER
    dropdown: SV_DROPDOWN_RENDERER
    progress_bar: SV_PROGRESS_RENDERER
    sparkline: SV_SPARKLINE_RENDERER
    avatar: SV_AVATAR_RENDERER
    badge: SV_BADGE_RENDERER
    rating: SV_RATING_RENDERER
    action_buttons: SV_ACTION_BUTTONS_RENDERER
end

-- Usage
.add_column ("Progress", agent {TASK}.percent_complete)
    .renderer (sv.renderers.progress_bar)

.add_column ("Status", agent {ORDER}.status)
    .renderer (sv.renderers.badge
        .color_map (<<
            ["pending", sv.colors.warning],
            ["shipped", sv.colors.info],
            ["delivered", sv.colors.success]
        >>))
```

### Inline Editing Types

```eiffel
class SV_CELL_EDITORS
feature
    text: SV_TEXT_EDITOR
    number: SV_NUMBER_EDITOR
    date: SV_DATE_EDITOR
    dropdown: SV_DROPDOWN_EDITOR
    checkbox: SV_CHECKBOX_EDITOR
    color_picker: SV_COLOR_EDITOR
    autocomplete: SV_AUTOCOMPLETE_EDITOR
end

.add_column ("Category", agent {PRODUCT}.category)
    .editor (sv.editors.autocomplete
        .suggestions (agent category_suggestions))
```

### Export Capabilities

```eiffel
sv.data_grid
    .exportable
    .export_formats (<<sv.export.csv, sv.export.excel, sv.export.json>>)
    .export_button  -- Adds export button to toolbar

-- Programmatic
my_grid.export_to_csv ("products.csv")
my_grid.export_to_json ("products.json")
```

## Complete Example: Product Management Grid

```eiffel
feature -- UI
    create_product_grid: SV_DATA_GRID [PRODUCT]
        do
            Result := sv.data_grid [PRODUCT]
                -- Data source (reactive)
                .bind (product_repository)

                -- Columns
                .add_column ("", agent {PRODUCT}.image_url)
                    .width (50)
                    .renderer (sv.renderers.avatar.size (32))
                .add_column ("Name", agent {PRODUCT}.name)
                    .width (200)
                    .sortable
                    .searchable
                    .editable
                .add_column ("SKU", agent {PRODUCT}.sku)
                    .width (100)
                    .sortable
                .add_column ("Price", agent {PRODUCT}.price)
                    .width (100)
                    .sortable
                    .format ("$%.2f")
                    .editable
                    .editor (sv.editors.number.min (0))
                .add_column ("Stock", agent {PRODUCT}.stock_count)
                    .width (80)
                    .sortable
                    .renderer (agent stock_renderer)
                .add_column ("Status", agent {PRODUCT}.status)
                    .width (100)
                    .renderer (sv.renderers.badge.color_map (status_colors))
                    .editable
                    .editor (sv.editors.dropdown.options (status_options))
                .add_column ("Actions", Void)
                    .width (120)
                    .renderer (sv.renderers.action_buttons (<<
                        sv.action ("Edit").icon (sv.icons.edit).on_click (agent edit_product),
                        sv.action ("Delete").icon (sv.icons.delete).on_click (agent delete_product)
                    >>))

                -- Features
                .sortable
                .filterable
                .selection_mode (sv.selection.multiple_rows)
                .row_height (48)
                .header_height (40)
                .alternating_row_colors
                .row_hover_highlight

                -- Toolbar
                .toolbar (<<
                    sv.button ("Add Product").primary.on_click (agent add_product),
                    sv.spacer,
                    sv.search_field.placeholder ("Search products..."),
                    sv.button ("Export").on_click (agent export_products)
                >>)

                -- Pagination
                .pagination (25)
                .pagination_position (bottom)

                -- Events
                .on_double_click (agent edit_product)
                .on_selection_change (agent update_toolbar_state)

                .build
        end

    stock_renderer (a_stock: INTEGER): SV_WIDGET
        do
            if a_stock = 0 then
                Result := sv.badge ("Out of Stock").color (sv.colors.error)
            elseif a_stock < 10 then
                Result := sv.badge ("Low: " + a_stock.out).color (sv.colors.warning)
            else
                Result := sv.text (a_stock.out)
            end
        end
```

## EV_GRID vs SV_DATA_GRID Comparison

| Task | EV_GRID (Lines) | SV_DATA_GRID (Lines) |
|------|----------------|---------------------|
| Basic table with 5 columns | ~50 | ~15 |
| Lazy loading setup | ~30 | ~5 |
| Sortable columns | ~40 | ~5 |
| Inline editing | ~60 | ~10 |
| Full-featured product grid | ~300 | ~50 |

## Implementation Notes

SV_DATA_GRID wraps EV_GRID and uses:
- `dynamic_content_function` for lazy loading
- `lock_update/unlock_update` for batch operations
- `enable_row_height_fixed` for performance
- Item type selection based on column config
- Automatic EV_GRID_LABEL_ITEM, EV_GRID_EDITABLE_ITEM, EV_GRID_CHECKABLE_LABEL_ITEM creation

---

# CROSS-PLATFORM CONSTRAINTS: Reality Check

## The EV2 Foundation

simple_vision sits atop EiffelVision-2, which provides cross-platform support via the Bridge Pattern:

| Platform | Backend | Status |
|----------|---------|--------|
| Windows | Win32/GDI+ | Mature, Full Support |
| Linux | GTK+ 3 | Mature, Full Support |
| macOS | Cocoa | Supported |

**We are constrained by what EV2 can do at the platform level.**

## What We CAN Do (Within EV2 Constraints)

### 1. High-Level Abstractions (Safe)
These innovations work entirely within EV2's widget model:

| Innovation | Cross-Platform | Notes |
|------------|---------------|-------|
| Reactive Binding | YES | Pure Eiffel logic, no platform code |
| State Machine | YES | Pure Eiffel logic |
| Component System | YES | Composes EV2 widgets |
| Form System | YES | Validation is pure Eiffel |
| Navigation | YES | Widget swapping is EV2-native |
| AI Builder | YES | Generates EV2-compatible code |
| Design Tokens | YES | Color/font values work everywhere |

### 2. Layout Constraints (Mostly Safe)
The constraint system enhances EV2's existing layout:

| Feature | Cross-Platform | Approach |
|---------|---------------|----------|
| Expand/Shrink | YES | Maps to EV_BOX.enable_item_expand |
| Percentage sizing | YES | Calculated at layout time |
| Min/Max sizes | PARTIAL | EV2 has min only; max via container |
| Responsive breakpoints | YES | Window resize events + recalculation |
| Flex ratios | YES | Custom container implementation |
| Aspect ratio | YES | Calculated, not native |

### 3. Graphics (Platform-Dependent)

| Feature | Cross-Platform | Notes |
|---------|---------------|-------|
| Basic drawing (lines, rects, ellipses) | YES | EV_DRAWABLE |
| Filled shapes | YES | EV_DRAWABLE |
| Text rendering | YES | EV_DRAWABLE.draw_text |
| Colors | YES | EV_COLOR |
| Fonts | MOSTLY | Font availability varies |
| Images | YES | EV_PIXMAP, EV_PIXBUF |
| **Gradients** | NO | Not in EV2; would need platform-specific |
| **Drop shadows** | PARTIAL | Simulated with layered drawing |
| **Blur effects** | NO | Not available |
| **Rounded corners (native)** | NO | Simulated with clipping regions |

### 4. Accessibility

| Feature | Cross-Platform | Notes |
|---------|---------------|-------|
| Keyboard focus | YES | EV2 native |
| Tab navigation | YES | EV2 native |
| Screen reader labels | PARTIAL | Platform AT-SPI (Linux), MSAA (Win) |
| High contrast | YES | Via Design Tokens |

## What We CANNOT Do (Platform Limitations)

1. **True native shadows** - EV2 has no shadow API
2. **Blur/frosted glass** - Requires compositor access
3. **Native rounded corners** - Must simulate with clipping
4. **Animations > 30fps** - EV2 timer resolution limits
5. **Custom window chrome** - Platform window manager controls this
6. **Transparency/translucency** - Limited support in EV2
7. **GPU acceleration** - EV2 uses CPU rendering

## Innovation Compatibility Matrix

```
┌─────────────────────────────────────────────────────────────────────┐
│                    INNOVATION COMPATIBILITY                         │
├───────────────────────┬─────────────────────────────────────────────┤
│ Innovation            │ Win │ Linux │ macOS │ Notes                 │
├───────────────────────┼─────┼───────┼───────┼───────────────────────┤
│ Reactive Binding      │  ✓  │   ✓   │   ✓   │ Pure Eiffel           │
│ State Machine         │  ✓  │   ✓   │   ✓   │ Pure Eiffel           │
│ Components            │  ✓  │   ✓   │   ✓   │ Composes EV2 widgets  │
│ Constraints           │  ✓  │   ✓   │   ✓   │ Layout calculation    │
│ Forms                 │  ✓  │   ✓   │   ✓   │ Pure Eiffel           │
│ Navigation            │  ✓  │   ✓   │   ✓   │ Widget management     │
│ AI Builder            │  ✓  │   ✓   │   ✓   │ Generates Eiffel code │
│ Design Tokens         │  ✓  │   ✓   │   ✓   │ Color/spacing values  │
│ Dark Mode             │  ✓  │   ✓   │   ✓   │ Swap token values     │
│ Icons (vector-like)   │  ✓  │   ✓   │   ✓   │ Pre-rendered pixmaps  │
│ Gradients             │  ~  │   ~   │   ~   │ Simulated line-by-line│
│ Shadows               │  ~  │   ~   │   ~   │ Simulated overlay     │
│ Animations            │  ~  │   ~   │   ~   │ Timer-based, limited  │
│ Rounded Corners       │  ~  │   ~   │   ~   │ Clipping regions      │
│ Blur/Glass            │  ✗  │   ✗   │   ✗   │ Not possible          │
├───────────────────────┴─────┴───────┴───────┴───────────────────────┤
│ ✓ = Full Support   ~ = Simulated/Limited   ✗ = Not Possible        │
└─────────────────────────────────────────────────────────────────────┘
```

## Design Decisions for Cross-Platform

### 1. Graceful Degradation
When a feature isn't fully supported, degrade gracefully:

```eiffel
-- Shadow: use simulated on all platforms
sv.card
    .shadow (sv.tokens.shadow_md)  -- Will use overlay simulation
```

### 2. Prefer Pure Eiffel
Keep as much logic as possible in pure Eiffel (not platform-dependent):
- All data binding logic
- All state machine logic
- All validation logic
- All layout calculations

### 3. Abstraction Over Platform Code
If platform-specific features are ever needed, use the EV2 bridge pattern:

```eiffel
class SV_NATIVE_FEATURE
    -- Abstract interface
feature
    do_platform_thing
        do
            implementation.do_platform_thing
        end

feature {NONE}
    implementation: SV_NATIVE_FEATURE_I
        -- Created based on platform at runtime
end
```

### 4. Document Platform Differences
All SV_* classes document platform behavior:

```eiffel
feature -- Shadows
    shadow (a_shadow: SV_SHADOW): like Current
            -- Apply drop shadow effect.
            -- Note: Simulated on all platforms using overlay technique.
            -- May have performance impact on complex UIs.
```

---

# AI-FRIENDLY DESIGN: Built for Claude Code

## Design Philosophy

simple_vision is designed to be **supremely friendly to AI coding assistants**, specifically Claude Code. Every API decision considers: "Can Claude generate this correctly on the first try?"

## Principles for AI-Friendly Design

### 1. Predictable, Consistent Naming

**Rule: Same pattern everywhere**

```eiffel
-- ALL widget builders follow the same pattern:
sv.button (label)
sv.text (content)
sv.text_field
sv.checkbox (label)
sv.dropdown

-- ALL property setters use the same naming:
.padding (value)
.margin (value)
.spacing (value)
.width (value)
.height (value)

-- ALL event handlers use on_* prefix:
.on_click (agent)
.on_change (agent)
.on_submit (agent)
.on_focus (agent)

-- ALL boolean properties are is_* or has_*:
.is_enabled
.is_visible
.is_readonly
.has_focus
```

**Why this helps Claude:** Pattern recognition. Once Claude sees one widget, it can correctly guess all others.

### 2. Fluent Chains That Read Like English

```eiffel
-- Claude can generate this from natural language easily:
sv.button ("Submit")
    .primary
    .icon (sv.icons.check)
    .on_click (agent submit_form)
    .disabled_when (form.is_invalid)
```

**The prompt:** "Create a primary submit button with a check icon that calls submit_form and is disabled when the form is invalid"

**Why this helps Claude:** Direct mapping from natural language to code.

### 3. No Magic, No Hidden State

```eiffel
-- BAD (hidden state, hard for AI to track):
form.field("email").text  -- Requires knowing field was added earlier

-- GOOD (explicit, self-documenting):
email_field: SV_TEXT_FIELD
email_field.text
```

**Why this helps Claude:** Claude can see all relevant state in the current scope.

### 4. Semantic Method Names Over Flags

```eiffel
-- BAD (boolean flag, Claude must guess meaning):
sv.button ("OK").style (True, False, True)

-- GOOD (semantic, self-documenting):
sv.button ("OK").primary.rounded.full_width
```

**Why this helps Claude:** No guessing about boolean parameter meanings.

### 5. Factory Methods Over Complex Constructors

```eiffel
-- BAD (many constructor args, easy to mix up):
create my_button.make ("Label", True, False, 12, "#FF0000")

-- GOOD (factory with named configuration):
my_button := sv.button ("Label").primary.font_size (12).color (sv.colors.error)
```

**Why this helps Claude:** Named parameters are unambiguous.

### 6. Inline Documentation Via Types

```eiffel
-- The type tells Claude what's expected:
set_sensitivity (a_value: REAL_64)
    require
        valid_range: a_value >= 0.0 and a_value <= 1.0
```

**Why this helps Claude:** Preconditions document valid inputs.

### 7. Composable Components

```eiffel
-- Claude can combine pieces predictably:
sv.column.children (<<
    sv.component (SV_HEADER),
    sv.component (SV_SIDEBAR),
    sv.component (SV_CONTENT),
    sv.component (SV_FOOTER)
>>)
```

**Why this helps Claude:** Modular composition is easier to reason about than inheritance.

## AI Generation Prompt Patterns

### Pattern 1: Widget Creation
```
User: "Create a [widget type] with [properties]"
Claude generates:
sv.[widget_type]
    .[property_1] (value_1)
    .[property_2] (value_2)
```

### Pattern 2: Event Handling
```
User: "When [event] happens, [action]"
Claude generates:
widget.on_[event].extend (agent [action])
-- OR fluent:
widget.on_[event] (agent [action])
```

### Pattern 3: Layout
```
User: "Arrange [widgets] in a [direction] with [spacing]"
Claude generates:
sv.[column/row]
    .spacing ([spacing])
    .children (<<
        [widget_1],
        [widget_2]
    >>)
```

### Pattern 4: Reactive Binding
```
User: "Bind [widget] to [data]"
Claude generates:
[widget].bind ([observable_data])
```

### Pattern 5: Conditional UI
```
User: "Show [widget] only when [condition]"
Claude generates:
[widget].visible_when ([condition_observable])
-- OR:
if [condition] then [widget] else sv.empty end
```

## Common AI Generation Tasks

### Task: "Create a login form"
```eiffel
sv.card
    .title ("Login")
    .padding (24)
    .content (
        sv.column
            .spacing (16)
            .children (<<
                sv.text_field
                    .label ("Email")
                    .placeholder ("you@example.com")
                    .bind (email_observable),
                sv.text_field
                    .label ("Password")
                    .input_type ("password")
                    .bind (password_observable),
                sv.checkbox ("Remember me")
                    .bind (remember_observable),
                sv.button ("Login")
                    .primary
                    .full_width
                    .on_click (agent handle_login)
            >>)
    )
```

### Task: "Create a data table with sorting"
```eiffel
sv.data_table
    .columns (<<
        sv.column ("Name").sortable,
        sv.column ("Email").sortable,
        sv.column ("Role"),
        sv.column ("Actions").width (100)
    >>)
    .bind_items (users_data_source)
    .row_actions (<<
        sv.action ("Edit").icon (sv.icons.edit).on_click (agent edit_user),
        sv.action ("Delete").icon (sv.icons.delete).on_click (agent delete_user)
    >>)
    .pagination (10)
```

### Task: "Create a responsive sidebar layout"
```eiffel
sv.row
    .children (<<
        sv.component (SV_SIDEBAR)
            .items (nav_items)
            .on_select (agent navigator.go_to)
            .constraints (
                sv.c.width (250)
                    .at_width (sv.breakpoints.sm, sv.c.width (0))  -- Hidden on mobile
            ),
        sv.column
            .constraints (sv.c.expand)
            .children (<<
                navigator.outlet
            >>)
    >>)
```

## Error Prevention for AI

### 1. Compile-Time Catches
```eiffel
-- Type system prevents wrong types:
sv.button ("OK").padding ("large")  -- COMPILE ERROR: expects INTEGER

-- Use semantic types:
sv.button ("OK").padding (sv.space.lg)  -- CORRECT
```

### 2. Precondition Failures (Runtime but Clear)
```eiffel
set_sensitivity (a_value: REAL_64)
    require
        valid_range: a_value >= 0.0 and a_value <= 1.0
    -- Clear error message if Claude generates out-of-range value
```

### 3. Helpful Error Messages
When Claude generates incorrect code, error messages should help it self-correct:
```
Precondition violation: valid_range in SV_TRANSITION_DETECTOR.set_sensitivity
Expected: 0.0 <= value <= 1.0
Actual: 5.0
```

## Claude Code Integration Notes

### 1. Simple API First
When Claude needs to generate UI quickly, use SV_QUICK:
```eiffel
sv.quick_window ("My App")
    .with_sidebar (nav_items)
    .with_header ("Dashboard")
    .content (my_content)
    .show
```

### 2. AI Builder for Complex UI
When user describes complex UI, Claude can use AI Builder:
```eiffel
ui := sv.ai_builder.generate_ui (user_description)
window.content (ui)
```

### 3. Predictable File Structure
All simple_vision projects follow the same structure:
```
my_app/
├── src/
│   ├── app.e           -- Main application
│   ├── views/          -- UI views
│   ├── components/     -- Custom components
│   └── models/         -- Data models
├── resources/
│   └── theme.json      -- Custom theme (optional)
└── my_app.ecf
```

---

*simple_vision — Not just wrapping EV2. Revolutionizing Eiffel GUI development.*
