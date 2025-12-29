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
EV2's native drawing (EV_DRAWABLE) is basic — lines, rectangles, ellipses, text. No gradients, shadows, blur, or modern effects.

## The Solution: SV_GRAPHICS (Multi-Tier)

SV_GRAPHICS provides a unified API that uses the best available backend:
- **Tier 1 (EV2 only):** Basic shapes, solid colors
- **Tier 2 (Cairo available):** True gradients, shadows, blur, anti-aliased paths (see Innovation 12)
- **Tier 3 (webview):** CSS-based effects in web panels (see Innovation 13)

```eiffel
class SV_GRAPHICS

feature -- Gradients (Cairo-backed when available)
    linear_gradient (a_start, a_end: SV_COLOR; a_angle: REAL): SV_GRADIENT
    radial_gradient (a_center, a_outer: SV_COLOR): SV_GRADIENT

feature -- Shadows (Cairo-backed when available, else simulated)
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
| 9 | **Graphics** | Modern visual effects (see Innovation 12) | CSS3, CoreGraphics |
| 10 | **Extras** | a11y, i18n*, undo, drag-drop, etc. | WCAG, react-intl |
| 11 | **EV_GRID Enhancement** | Data grids with lazy loading, virtual scrolling, 1M+ rows | AG Grid, TanStack Table |
| 12 | **C Library Integration** | Cairo gradients/shadows, stb images, cross-platform | Cairo, stb, webview |
| 13 | **Hybrid Native + Web UI** | Blend native EV2 with web panels (HTMX, Alpine.js) | Electron, Tauri |
| 14 | **GUI Testing Harness** | Automated GUI testing with AI visual validation | Selenium, Playwright, AI Vision |

*i18n via existing simple_i18n library
*Graphics enhanced by C libraries (Cairo, Blend2D) - see Innovation 12

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

## How simple_vision Uses the Ecosystem

simple_vision classes USE simple_* libraries internally — no unnecessary wrapper layers.

### Design Principle: Direct Use, Not Wrapping

```eiffel
-- WRONG: Unnecessary wrapper
class SV_CLIPBOARD  -- Don't do this
    clipboard: SIMPLE_CLIPBOARD
    copy (s: STRING) do clipboard.copy_text (s) end  -- Pointless indirection

-- RIGHT: Direct use inside widgets
class SV_TEXT_FIELD inherit SV_WIDGET
feature {NONE}
    clipboard: SIMPLE_CLIPBOARD  -- Used internally, not exposed
feature
    copy_selection do clipboard.copy_text (selected_text) end
```

### SV_DATA_SOURCE Implementations (Add Value: Reactive Binding)

These ARE new classes because they implement the `SV_DATA_SOURCE [G]` contract, adding reactive binding on top of simple_* libraries:

| Implementation | Uses Internally | Added Value |
|----------------|-----------------|-------------|
| `SV_SQL_DATA_SOURCE [G]` | simple_sql | Reactive queries, auto-refresh on DB change |
| `SV_REST_DATA_SOURCE [G]` | simple_http | Pagination, caching, reactive updates |
| `SV_REALTIME_DATA_SOURCE [G]` | simple_websocket | Live streaming to UI |
| `SV_FILE_DATA_SOURCE [G]` | simple_json/yaml/xml | File-backed reactive store |
| `SV_CSV_DATA_SOURCE [G]` | simple_csv | CSV with reactive binding |

```eiffel
deferred class SV_DATA_SOURCE [G]
    -- Abstract contract — the interface simple_vision widgets bind to

class SV_SQL_DATA_SOURCE [G] inherit SV_DATA_SOURCE [G]
    -- Implementation that uses simple_sql internally
feature {NONE}
    db: SIMPLE_SQL  -- Internal dependency, not exposed
feature
    -- Implements SV_DATA_SOURCE contract
    get_all: LIST [G] do ... end
    on_change: SV_ACTION_SEQUENCE [TUPLE [G]]  -- Reactive!
```

### Widgets That Use simple_* Internally

These are first-class SV_* widgets that happen to use simple_* libraries for their implementation:

| Widget | Uses Internally | Purpose |
|--------|-----------------|---------|
| `SV_DATE_PICKER` | simple_datetime | Date selection with formatting |
| `SV_MARKDOWN_VIEW` | simple_markdown | Rendered markdown display |
| `SV_VIDEO_PLAYER` | simple_ffmpeg | Video playback |
| `SV_AUDIO_PLAYER` | simple_audio | Audio playback |

```eiffel
class SV_DATE_PICKER inherit SV_WIDGET
    -- A date picker widget (not a wrapper!)

feature {NONE} -- Internal dependencies
    date_helper: SIMPLE_DATETIME  -- Used internally for formatting

feature -- Widget API
    value: SV_OBSERVABLE [DATE]
    min_date (a_date: DATE): like Current
    max_date (a_date: DATE): like Current
    format (a_format: STRING): like Current
end

-- Usage: user sees a widget, not simple_datetime
sv.date_picker
    .format ("YYYY-MM-DD")
    .bind (due_date_observable)
```

### SV_* Family Conveniences (Discoverability)

For **family cohesion**, utility classes live in the SV_* namespace. Users look in one place — the SV_* family — and find everything they need. These aren't wrappers; they're **convenient entry points** that expose the underlying simple_* functionality:

| SV_* Class | Exposes | Why in SV_* Family |
|------------|---------|-------------------|
| `SV_CLIPBOARD` | simple_clipboard | Clipboard ops from SV_* namespace |
| `SV_I18N` | simple_i18n | Translations accessible via `sv.i18n` |
| `SV_PDF` | simple_pdf | PDF export via `sv.pdf` |
| `SV_CONFIG` | simple_config | Settings via `sv.config` |
| `SV_LOG` | simple_logger | Logging via `sv.log` |
| `SV_VALIDATE` | simple_validation | Validation via `sv.validate` |

```eiffel
-- Family cohesion: everything accessible via sv.*
class SV_QUICK
feature -- Utilities (family members)
    clipboard: SV_CLIPBOARD once create Result end
    i18n: SV_I18N once create Result end
    pdf: SV_PDF once create Result end
    config: SV_CONFIG once create Result end
    log: SV_LOG once create Result end
    validate: SV_VALIDATE once create Result end
end

-- Usage: one namespace, no confusion
sv.clipboard.copy ("text")
sv.i18n.translate ("hello")
sv.pdf.export (my_grid, "report.pdf")
sv.log.info ("User logged in")
sv.validate.email (email_field.text)
```

### SV_* Convenience Classes (Not Wrappers)

These aren't wrappers — they're the simple_* classes **re-homed** into the SV_* family:

```eiffel
class SV_CLIPBOARD
    -- Not a wrapper! Just inherits and lives in SV_* namespace
inherit
    SIMPLE_CLIPBOARD
        rename
            copy_text as copy,
            paste as paste_text
        end
end

-- Or even simpler — just an alias:
class SV_CLIPBOARD = SIMPLE_CLIPBOARD
```

The point isn't to add indirection — it's **discoverability**:

```eiffel
-- User thinks: "I need clipboard in my SV app"
-- User looks: SV_* classes
-- User finds: SV_CLIPBOARD
-- Done. No hunting through multiple namespaces.
```

## Dependency Graph

```
┌─────────────────────────────────────────────────────────────────┐
│                      simple_vision (SV_*)                       │
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │
│  │ SV_DATA_    │  │ SV_WIDGET   │  │ SV_* Family Conveniences│ │
│  │ SOURCE [G]  │  │ classes     │  │ (discoverability)       │ │
│  │ impls       │  │             │  │                         │ │
│  ├─────────────┤  ├─────────────┤  ├─────────────────────────┤ │
│  │ uses:       │  │ uses:       │  │ re-homes:               │ │
│  │ simple_sql  │  │ simple_     │  │ SV_CLIPBOARD            │ │
│  │ simple_http │  │   datetime  │  │ SV_I18N                 │ │
│  │ simple_ws   │  │ simple_     │  │ SV_PDF                  │ │
│  │ simple_json │  │   markdown  │  │ SV_CONFIG               │ │
│  │ simple_csv  │  │ simple_     │  │ SV_LOG                  │ │
│  └─────────────┘  │   ffmpeg    │  │ SV_VALIDATE             │ │
│                   │ simple_audio│  └─────────────────────────┘ │
│                   └─────────────┘                               │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    simple_* ecosystem                           │
│  simple_sql, simple_http, simple_websocket, simple_json,        │
│  simple_datetime, simple_markdown, simple_ffmpeg, simple_audio, │
│  simple_clipboard, simple_i18n, simple_pdf, simple_config,      │
│  simple_logger, simple_validation, ...                          │
└─────────────────────────────────────────────────────────────────┘
```

---

# INNOVATION 12: C Library Integration (Raising the Bar)

## The Opportunity

EV2 is cross-platform but dated. Modern C libraries can fill gaps:

| EV2 Limitation | C Library Solution |
|----------------|-------------------|
| No gradients | Cairo, Blend2D |
| No shadows | Cairo (simulated), Blend2D |
| Basic anti-aliasing | NanoVG, Cairo, Blend2D |
| No GPU acceleration | NanoVG (OpenGL), Blend2D |
| No rich text/HTML | webview, Ultralight |
| Basic charts | PLplot |
| Limited image formats | stb_image |
| Basic font rendering | stb_truetype, FreeType |

## Candidate Libraries (All Cross-Platform: Win/Linux/Mac)

### Tier 1: High Value, Low Risk

#### 1. stb Libraries (Public Domain)
Single-header, trivial to integrate via inline C:

| Library | Purpose | Complexity |
|---------|---------|------------|
| [stb_image](https://github.com/nothings/stb) | Load PNG, JPG, BMP, GIF, etc. | Very Low |
| [stb_image_write](https://github.com/nothings/stb) | Write PNG, JPG, BMP | Very Low |
| [stb_truetype](https://github.com/nothings/stb) | TrueType font rasterization | Low |

```eiffel
-- Integration pattern (inline C):
class SV_IMAGE_LOADER
feature {NONE} -- C externals
    c_load_image (a_path: POINTER; out_w, out_h, out_channels: POINTER): POINTER
        external "C inline use %"stb_image.h%""
        alias "[
            #define STB_IMAGE_IMPLEMENTATION
            #include "stb_image.h"
            return stbi_load((const char*)$a_path, (int*)$out_w, (int*)$out_h, (int*)$out_channels, 0);
        ]"
        end
end
```

#### 2. webview/webview (MIT License)
Embed HTML/CSS/JS in native apps — tiny footprint:

| Platform | Backend |
|----------|---------|
| Windows | Edge WebView2 |
| Linux | WebKit2 (GTK) |
| macOS | WKWebView |

```eiffel
class SV_WEB_VIEW inherit SV_WIDGET
feature
    load_url (a_url: STRING)
    load_html (a_html: STRING)
    execute_js (a_script: STRING): STRING
    on_message: SV_ACTION_SEQUENCE [TUPLE [message: STRING]]
        -- JS can send messages to Eiffel
end

-- Usage: embed rich content
sv.web_view
    .load_html ("<h1>Dashboard</h1><div id='chart'></div>")
    .execute_js ("renderChart(data)")
```

**Use Cases:**
- Rich text editing (HTML contenteditable)
- Charts (Chart.js, D3.js)
- Markdown preview
- Documentation viewer
- Complex forms

#### 3. Cairo (LGPL/MPL)
Mature 2D vector graphics — already used by GTK on Linux:

| Feature | EV2 | Cairo |
|---------|-----|-------|
| Gradients | No | Yes (linear, radial) |
| Shadows | No | Yes (via blur) |
| Anti-aliasing | Basic | Excellent |
| Path operations | Limited | Full |
| PDF/SVG export | No | Yes |

```eiffel
class SV_CAIRO_CANVAS inherit SV_WIDGET
feature
    -- Drawing
    set_source_linear_gradient (x0, y0, x1, y1: REAL; stops: ARRAY [TUPLE [offset: REAL; color: SV_COLOR]])
    set_source_radial_gradient (...)
    fill_rounded_rect (x, y, w, h, radius: REAL)
    draw_shadow (x, y, w, h, blur, spread: REAL; color: SV_COLOR)

    -- Export
    export_to_pdf (a_path: STRING)
    export_to_svg (a_path: STRING)
end

-- Usage
sv.cairo_canvas
    .set_source_linear_gradient (0, 0, 0, 100, <<[0.0, sv.colors.primary], [1.0, sv.colors.primary_dark]>>)
    .fill_rounded_rect (10, 10, 200, 100, 8)
```

### Tier 2: Medium Value, Medium Risk

#### 4. Blend2D (Zlib License)
High-performance 2D graphics engine:

- **Pros:** Modern, fast, active development, C API available
- **Cons:** Larger dependency than stb

```eiffel
class SV_BLEND2D_CANVAS inherit SV_WIDGET
feature
    -- High-performance 2D drawing
    gradient_fill (...)
    blur_effect (...)
    composition_modes (...)  -- Porter-Duff, etc.
end
```

#### 5. PLplot (LGPL)
Scientific plotting library:

```eiffel
class SV_CHART inherit SV_WIDGET
feature
    -- Chart types
    line_chart (data: ARRAY [TUPLE [x, y: REAL]])
    bar_chart (data: ARRAY [TUPLE [label: STRING; value: REAL]])
    pie_chart (data: ARRAY [TUPLE [label: STRING; value: REAL]])
    scatter_plot (...)
    histogram (...)

    -- Configuration
    title (a_title: STRING): like Current
    x_axis_label (a_label: STRING): like Current
    y_axis_label (a_label: STRING): like Current
    legend (a_position: INTEGER): like Current
end

-- Usage
sv.chart
    .line_chart (sales_data)
    .title ("Monthly Sales")
    .x_axis_label ("Month")
    .y_axis_label ("Revenue ($)")
```

### Tier 3: High Value, Higher Risk

#### 6. Ultralight (Commercial/Free Tier)
GPU-accelerated HTML/CSS/JS renderer — more powerful than webview:

- **Pros:** Full browser capabilities, GPU acceleration, smaller than Chromium
- **Cons:** Commercial license for some uses, larger dependency

```eiffel
class SV_ULTRALIGHT_VIEW inherit SV_WIDGET
feature
    -- Full browser capabilities
    load_url (a_url: STRING)
    load_html (a_html: STRING)

    -- Bidirectional JS integration
    bind_function (a_name: STRING; a_callback: PROCEDURE [...])
        -- Expose Eiffel function to JavaScript

    call_js (a_function: STRING; a_args: ARRAY [ANY]): ANY
        -- Call JavaScript function from Eiffel
end
```

## Integration Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      simple_vision (SV_*)                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │ SV_WIDGET   │  │ SV_CANVAS   │  │ SV_WEB_VIEW         │ │
│  │ (EV2-based) │  │ (C-enhanced)│  │ (HTML/CSS/JS)       │ │
│  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘ │
│         │                │                     │            │
│         ▼                ▼                     ▼            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │ EiffelVision│  │ Cairo/      │  │ webview / Ultralight│ │
│  │ 2 (EV_*)    │  │ Blend2D     │  │                     │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ stb_image, stb_truetype, PLplot (utilities)         │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Practical Integration: simple_cairo, simple_webview

Following the simple_* pattern, create focused libraries:

### simple_cairo (D:\prod\simple_cairo)
```eiffel
class SIMPLE_CAIRO
feature
    -- Surface management
    create_image_surface (width, height: INTEGER): CAIRO_SURFACE
    create_svg_surface (path: STRING; width, height: REAL): CAIRO_SURFACE
    create_pdf_surface (path: STRING; width, height: REAL): CAIRO_SURFACE

    -- Drawing context
    create_context (surface: CAIRO_SURFACE): CAIRO_CONTEXT

    -- In CAIRO_CONTEXT:
    set_source_rgb (r, g, b: REAL)
    set_source_rgba (r, g, b, a: REAL)
    set_source_linear_gradient (...)
    set_source_radial_gradient (...)

    move_to (x, y: REAL)
    line_to (x, y: REAL)
    curve_to (...)
    arc (...)
    rectangle (x, y, w, h: REAL)
    rounded_rectangle (x, y, w, h, radius: REAL)

    fill
    stroke
    clip
end
```

### simple_webview (D:\prod\simple_webview)
```eiffel
class SIMPLE_WEBVIEW
feature
    make (a_title: STRING; a_width, a_height: INTEGER)

    navigate (a_url: STRING)
    set_html (a_html: STRING)

    eval (a_js: STRING)
    bind (a_name: STRING; a_callback: PROCEDURE [TUPLE [request: STRING]])

    run  -- Event loop
    terminate
end
```

Then simple_vision uses them:
```eiffel
class SV_CAIRO_PANEL inherit SV_WIDGET
feature {NONE}
    cairo: SIMPLE_CAIRO  -- Uses simple_cairo internally
end

class SV_WEB_PANEL inherit SV_WIDGET
feature {NONE}
    webview: SIMPLE_WEBVIEW  -- Uses simple_webview internally
end
```

## Recommended Implementation Order

| Priority | Library | Complexity | Value |
|----------|---------|------------|-------|
| 1 | stb_image/stb_image_write | Very Low (single-header) | Image format support |
| 2 | webview/webview | Low (small C lib) | Rich content, charts, markdown |
| 3 | Cairo | Medium (mature API) | Gradients, shadows, PDF export |
| 4 | stb_truetype | Low (single-header) | Custom font rendering |
| 5 | PLplot | Medium (scientific lib) | Native charts |
| 6 | Blend2D | Medium-High (larger lib) | High-performance graphics |

*At 5K LOC/day velocity, each library integration is achievable in 1-2 sessions.*

## What This Enables

| Feature | Before (EV2 only) | After (C-enhanced) |
|---------|-------------------|-------------------|
| Gradients | Not possible | Full linear/radial |
| Shadows | Not possible | Cairo blur simulation |
| Charts | Manual drawing | PLplot or Chart.js via webview |
| Rich text | Plain text only | HTML via webview |
| PDF export | Not built-in | Cairo PDF surface |
| Image formats | Limited | All major formats via stb |
| Modern UI | 1990s look | 2025 look with webview |

---

# INNOVATION 13: Hybrid Native + Web UI (The Ultimate Blend)

## The Vision: Best of Both Worlds

Combine:
- **Native EV2 widgets** for system integration, performance
- **Web panels (webview)** for modern UI, rich content
- **simple_htmx** for declarative HTML interactions
- **simple_alpine** for reactive JavaScript
- **Eiffel backend** for business logic, data, security

```
┌─────────────────────────────────────────────────────────────┐
│                    simple_vision App                        │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────────────────────┐  │
│  │ Native Toolbar  │  │ Web Panel (SV_WEB_VIEW)         │  │
│  │ (SV_TOOLBAR)    │  │                                 │  │
│  │  [File] [Edit]  │  │  ┌─────────────────────────┐   │  │
│  └─────────────────┘  │  │ HTMX + Alpine.js UI     │   │  │
│                       │  │                         │   │  │
│  ┌─────────────────┐  │  │ <div x-data="...">     │   │  │
│  │ Native Sidebar  │  │  │   <button hx-get=...>  │   │  │
│  │ (SV_TREE)       │  │  │   Modern widgets!      │   │  │
│  │  📁 Projects    │  │  │ </div>                 │   │  │
│  │  📁 Tasks       │  │  └─────────────────────────┘   │  │
│  │  📁 Reports     │  │                                 │  │
│  └─────────────────┘  │  Rendered by webview            │  │
│                       └─────────────────────────────────┘  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Native Status Bar (SV_STATUS_BAR)                   │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## The Stack

| Layer | Technology | Role |
|-------|------------|------|
| **Presentation** | HTMX + Alpine.js | Declarative, reactive HTML |
| **Rendering** | webview (Edge/WebKit) | Cross-platform browser |
| **Bridge** | simple_webview | Eiffel ↔ JS communication |
| **Templates** | simple_htmx, simple_alpine | Generate HTML from Eiffel |
| **Logic** | simple_vision + simple_* | Business logic, data access |
| **Native** | EV2 widgets | System menus, toolbars, dialogs |

## Eiffel-Driven Web UI

### Using simple_htmx to Generate HTML

```eiffel
class MY_DASHBOARD inherit SV_APPLICATION

feature
    web_content: SV_WEB_VIEW

    build_ui
        local
            htmx: SIMPLE_HTMX
            html: STRING
        do
            create htmx.make

            html := htmx.div ([
                htmx.h1 ("Dashboard"),

                -- HTMX-powered search (calls Eiffel backend!)
                htmx.input_text ("search")
                    .hx_get ("/api/search")
                    .hx_trigger ("keyup changed delay:500ms")
                    .hx_target ("#results"),

                htmx.div_id ("results", ""),

                -- Alpine.js reactive component
                htmx.div ([
                    htmx.raw ("<div x-data='{ count: 0 }'>"),
                    htmx.raw ("  <button @click='count++'>Clicked: <span x-text='count'></span></button>"),
                    htmx.raw ("</div>")
                ])
            ])

            web_content.load_html (htmx.full_page ("Dashboard", html))
        end
```

### Bidirectional Communication

```eiffel
class SV_WEB_BRIDGE
    -- Connects Eiffel backend to web frontend

feature -- Eiffel → JavaScript
    send_data (a_name: STRING; a_json: JSON_OBJECT)
        -- Push data to web panel
        do
            web_view.execute_js ("window.eiffelBridge.receive('" + a_name + "', " + a_json.representation + ")")
        end

    update_element (a_id: STRING; a_html: STRING)
        -- Update DOM element
        do
            web_view.execute_js ("document.getElementById('" + a_id + "').innerHTML = `" + a_html + "`")
        end

feature -- JavaScript → Eiffel
    on_js_call: SV_ACTION_SEQUENCE [TUPLE [method: STRING; args: JSON_ARRAY]]
        -- Handle calls from JavaScript

    bind_handler (a_method: STRING; a_handler: PROCEDURE [JSON_ARRAY])
        -- Register Eiffel handler for JS method
        do
            web_view.bind (a_method, agent handle_js_call (a_method, ?))
        end

feature {NONE}
    handle_js_call (a_method: STRING; a_args_json: STRING)
        local
            args: JSON_ARRAY
        do
            create args.make_from_string (a_args_json)
            if attached handlers.item (a_method) as h then
                h.call ([args])
            end
        end
```

### HTMX with Eiffel Backend

```eiffel
-- In JavaScript (web panel):
-- <button hx-get="/api/users" hx-target="#user-list">Load Users</button>

-- Eiffel handles the "API" call:
class MY_APP_BRIDGE inherit SV_WEB_BRIDGE

feature
    setup_handlers
        do
            bind_route ("/api/users", agent handle_get_users)
            bind_route ("/api/search", agent handle_search)
            bind_route ("/api/user/:id", agent handle_get_user)
        end

    handle_get_users (a_request: SV_WEB_REQUEST): STRING
        local
            htmx: SIMPLE_HTMX
            users: LIST [USER]
        do
            users := user_repository.get_all

            create htmx.make
            Result := htmx.table ([
                htmx.thead ([htmx.tr ([htmx.th ("Name"), htmx.th ("Email")])]),
                htmx.tbody (
                    users.collect (agent (u: USER): STRING
                        do
                            Result := htmx.tr ([
                                htmx.td (u.name),
                                htmx.td (u.email)
                            ])
                        end
                    )
                )
            ])
        end

    handle_search (a_request: SV_WEB_REQUEST): STRING
        local
            query: STRING
            results: LIST [ITEM]
        do
            query := a_request.param ("search")
            results := search_service.search (query)
            Result := render_search_results (results)
        end
```

## Pre-Built Hybrid Components

### SV_RICH_EDITOR (Web-based rich text)
```eiffel
class SV_RICH_EDITOR inherit SV_WEB_VIEW
feature
    content: SV_OBSERVABLE [STRING]
        -- HTML content, reactive

    set_content (a_html: STRING)
    get_content: STRING
    get_plain_text: STRING

    -- Toolbar
    bold, italic, underline, strike_through
    heading (a_level: INTEGER)
    bullet_list, numbered_list
    insert_link (a_url, a_text: STRING)
    insert_image (a_path: STRING)

    on_change: SV_ACTION_SEQUENCE [TUPLE [html: STRING]]
end

-- Usage
sv.rich_editor
    .content ("<p>Hello <b>world</b></p>")
    .on_change (agent save_draft)
```

### SV_CHART (Web-based charts via Chart.js)
```eiffel
class SV_CHART inherit SV_WEB_VIEW
feature
    -- Chart types
    line (a_data: SV_CHART_DATA): like Current
    bar (a_data: SV_CHART_DATA): like Current
    pie (a_data: SV_CHART_DATA): like Current
    doughnut (a_data: SV_CHART_DATA): like Current
    radar (a_data: SV_CHART_DATA): like Current
    polar_area (a_data: SV_CHART_DATA): like Current

    -- Configuration
    title (a_title: STRING): like Current
    legend (a_position: STRING): like Current
    animated: like Current
    responsive: like Current

    -- Update
    update_data (a_data: SV_CHART_DATA)
end

-- Usage
sv.chart
    .bar (sales_by_month)
    .title ("Monthly Sales 2025")
    .legend ("bottom")
    .animated
```

### SV_CODE_EDITOR (Web-based code editor via Monaco/CodeMirror)
```eiffel
class SV_CODE_EDITOR inherit SV_WEB_VIEW
feature
    content: SV_OBSERVABLE [STRING]
    language (a_lang: STRING): like Current  -- "eiffel", "python", "javascript", etc.

    theme (a_theme: STRING): like Current  -- "dark", "light"
    line_numbers: like Current
    minimap: like Current
    word_wrap: like Current

    go_to_line (a_line: INTEGER)
    insert_text (a_text: STRING)

    on_change: SV_ACTION_SEQUENCE [TUPLE [content: STRING]]
    on_save: SV_ACTION_SEQUENCE [TUPLE [content: STRING]]  -- Ctrl+S
end

-- Usage
sv.code_editor
    .language ("eiffel")
    .theme ("dark")
    .line_numbers
    .content (file_contents)
    .on_save (agent save_file)
```

### SV_MARKDOWN_EDITOR (Split-pane markdown)
```eiffel
class SV_MARKDOWN_EDITOR inherit SV_WEB_VIEW
feature
    content: SV_OBSERVABLE [STRING]

    show_preview: like Current
    hide_preview: like Current
    split_vertical: like Current
    split_horizontal: like Current

    export_html: STRING
    export_pdf (a_path: STRING)
end
```

### SV_DATA_TABLE (Web-based interactive table)
```eiffel
class SV_DATA_TABLE inherit SV_WEB_VIEW
feature
    -- Like AG-Grid or TanStack Table
    columns (a_cols: ARRAY [SV_TABLE_COLUMN]): like Current
    data (a_rows: LIST [JSON_OBJECT]): like Current

    sortable: like Current
    filterable: like Current
    paginated (a_page_size: INTEGER): like Current
    selectable: like Current

    on_row_click: SV_ACTION_SEQUENCE [TUPLE [row: JSON_OBJECT]]
    on_selection_change: SV_ACTION_SEQUENCE [TUPLE [rows: LIST [JSON_OBJECT]]]
end
```

## When to Use Native vs Web

| Use Case | Recommendation |
|----------|----------------|
| System menus | Native (SV_MENU) |
| Toolbars | Native (SV_TOOLBAR) |
| File dialogs | Native (SV_FILE_DIALOG) |
| Tree navigation | Native (SV_TREE) |
| Status bar | Native (SV_STATUS_BAR) |
| Rich text editing | Web (SV_RICH_EDITOR) |
| Charts/graphs | Web (SV_CHART) |
| Code editing | Web (SV_CODE_EDITOR) |
| Complex forms | Web (HTMX + Alpine) |
| Dashboards | Web (HTMX + Alpine) |
| Data tables | Web (SV_DATA_TABLE) |
| Markdown preview | Web (SV_MARKDOWN_VIEW) |

## The Power of This Approach

1. **Best of both worlds** — Native performance where needed, web richness where desired
2. **Leverage web ecosystem** — Thousands of JS libraries available
3. **Eiffel stays in control** — Business logic, data, security in Eiffel
4. **Rapid UI development** — HTML/CSS faster to iterate than native widgets
5. **Cross-platform consistency** — Web UI looks the same everywhere
6. **Progressive enhancement** — Start native, add web components as needed

## Integration with Existing simple_* Libraries

```eiffel
class MY_HYBRID_APP

feature -- Dependencies
    htmx: SIMPLE_HTMX           -- Generate HTMX HTML
    alpine: SIMPLE_ALPINE       -- Alpine.js components
    json: SIMPLE_JSON           -- Data serialization
    sql: SIMPLE_SQL             -- Database
    http: SIMPLE_HTTP           -- API calls
    markdown: SIMPLE_MARKDOWN   -- Markdown processing

feature -- UI Components
    native_sidebar: SV_TREE                 -- Native tree
    web_content: SV_WEB_VIEW                -- Web panel
    native_toolbar: SV_TOOLBAR              -- Native toolbar
    native_status_bar: SV_STATUS_BAR        -- Native status

    -- Full hybrid app!
end
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

# CROSS-PLATFORM CONSTRAINTS: Reality Check (Updated)

## The Three-Tier Rendering Architecture

simple_vision provides **three rendering strategies**, each with different capabilities and trade-offs:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    THREE-TIER RENDERING                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  TIER 1: Pure EV2 Widgets                                          │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ • Cross-platform native widgets (buttons, trees, grids)       │ │
│  │ • System menus, toolbars, dialogs                             │ │
│  │ • EV_DRAWABLE for basic drawing                               │ │
│  │ • Mature, stable, lightweight                                 │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  TIER 2: C-Enhanced Rendering (Cairo, Blend2D, stb)               │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ • Gradients (linear, radial) via Cairo                        │ │
│  │ • Drop shadows, blur effects via Cairo                        │ │
│  │ • Anti-aliased vector graphics                                │ │
│  │ • PDF/SVG export                                              │ │
│  │ • Additional image formats via stb                            │ │
│  │ Cross-platform: Win/Linux/Mac via C libraries                 │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  TIER 3: Web Panels (webview + HTMX + Alpine.js)                  │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ • Modern HTML/CSS/JS UI                                       │ │
│  │ • Rich text editing, charts (Chart.js), code editors (Monaco) │ │
│  │ • GPU-accelerated rendering                                   │ │
│  │ • Responsive, animated, 2025-quality interfaces               │ │
│  │ Cross-platform: Edge WebView2 (Win), WebKit (Linux/Mac)       │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## Platform Support by Tier

| Platform | Tier 1 (EV2) | Tier 2 (Cairo/C) | Tier 3 (webview) |
|----------|--------------|------------------|------------------|
| Windows | Win32/GDI+ ✓ | Cairo (prebuilt) ✓ | Edge WebView2 ✓ |
| Linux | GTK+ 3 ✓ | Cairo (native to GTK) ✓ | WebKit2 ✓ |
| macOS | Cocoa ✓ | Cairo (Homebrew) ✓ | WKWebView ✓ |

**All three tiers are cross-platform.** Developers choose the tier based on needs.

## What Each Tier Can Do

### Tier 1: Pure EV2 Widgets (Native, Lightweight)

| Feature | Status | Notes |
|---------|--------|-------|
| Buttons, labels, text fields | ✓ Full | Native widgets |
| Trees, lists, grids | ✓ Full | EV_TREE, EV_LIST, EV_GRID |
| Menus, toolbars, status bars | ✓ Full | Native chrome |
| Dialogs (file, color, font) | ✓ Full | System dialogs |
| Basic drawing | ✓ Full | EV_DRAWABLE |
| Colors, fonts | ✓ Full | EV_COLOR, EV_FONT |
| Gradients | ✗ None | Use Tier 2 |
| Shadows | ✗ None | Use Tier 2 |
| Blur/glass effects | ✗ None | Use Tier 2/3 |

### Tier 2: C-Enhanced (Cairo, Blend2D, stb)

| Feature | Status | Library |
|---------|--------|---------|
| Linear gradients | ✓ Full | Cairo |
| Radial gradients | ✓ Full | Cairo |
| Drop shadows | ✓ Full | Cairo |
| Blur effects | ✓ Full | Cairo, Blend2D |
| Anti-aliased paths | ✓ Full | Cairo, Blend2D |
| PDF export | ✓ Full | Cairo |
| SVG export | ✓ Full | Cairo |
| PNG/JPG/BMP loading | ✓ Full | stb_image |
| Image writing | ✓ Full | stb_image_write |
| TrueType fonts | ✓ Full | stb_truetype |
| Scientific charts | ✓ Full | PLplot |

### Tier 3: Web Panels (webview + HTMX + Alpine.js)

| Feature | Status | Technology |
|---------|--------|------------|
| Rich text editing | ✓ Full | TinyMCE, Quill via webview |
| Interactive charts | ✓ Full | Chart.js, D3.js via webview |
| Code editors | ✓ Full | Monaco Editor via webview |
| Complex forms | ✓ Full | HTMX + Alpine.js |
| Dashboards | ✓ Full | HTMX + Alpine.js |
| Markdown preview | ✓ Full | marked.js via webview |
| Data tables | ✓ Full | AG-Grid via webview |
| GPU acceleration | ✓ Full | Browser compositor |
| Animations | ✓ Full | CSS transitions/animations |

## Innovation Compatibility Matrix (Updated)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    INNOVATION COMPATIBILITY                         │
├───────────────────────┬─────────────────────────────────────────────┤
│ Innovation            │ Win │ Linux │ macOS │ Tier  │ Notes         │
├───────────────────────┼─────┼───────┼───────┼───────┼───────────────┤
│ 1. Reactive Binding   │  ✓  │   ✓   │   ✓   │  All  │ Pure Eiffel   │
│ 2. State Machine      │  ✓  │   ✓   │   ✓   │  All  │ Pure Eiffel   │
│ 3. Components         │  ✓  │   ✓   │   ✓   │  All  │ Composes any  │
│ 4. Constraints        │  ✓  │   ✓   │   ✓   │ 1,2   │ Layout calc   │
│ 5. Forms              │  ✓  │   ✓   │   ✓   │  All  │ Pure Eiffel   │
│ 6. Navigation         │  ✓  │   ✓   │   ✓   │  All  │ Widget mgmt   │
│ 7. AI Builder         │  ✓  │   ✓   │   ✓   │  All  │ Code gen      │
│ 8. Design Tokens      │  ✓  │   ✓   │   ✓   │  All  │ Color/spacing │
│ 9. Graphics Base      │  ✓  │   ✓   │   ✓   │  1    │ EV_DRAWABLE   │
│ 10. Extras (a11y...)  │  ✓  │   ✓   │   ✓   │  All  │ Various       │
│ 11. EV_GRID Enhanced  │  ✓  │   ✓   │   ✓   │  1    │ Native grid   │
│ 12. C Library (Cairo) │  ✓  │   ✓   │   ✓   │  2    │ Grad/shadow   │
│ 13. Hybrid Web UI     │  ✓  │   ✓   │   ✓   │  3    │ HTMX/Alpine   │
├───────────────────────┼─────┼───────┼───────┼───────┼───────────────┤
│ Gradients             │  ✓  │   ✓   │   ✓   │  2,3  │ Cairo/CSS     │
│ Drop Shadows          │  ✓  │   ✓   │   ✓   │  2,3  │ Cairo/CSS     │
│ Blur/Glass            │  ✓  │   ✓   │   ✓   │  2,3  │ Cairo/CSS     │
│ Rich Text Editing     │  ✓  │   ✓   │   ✓   │  3    │ TinyMCE       │
│ Charts                │  ✓  │   ✓   │   ✓   │  2,3  │ PLplot/Chart.js│
│ Code Editor           │  ✓  │   ✓   │   ✓   │  3    │ Monaco        │
│ GPU Acceleration      │  ✓  │   ✓   │   ✓   │  3    │ WebView       │
├───────────────────────┴─────┴───────┴───────┴───────┴───────────────┤
│ ✓ = Full Cross-Platform Support                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## Choosing the Right Tier

| Use Case | Recommended Tier | Why |
|----------|------------------|-----|
| System menus, toolbars | Tier 1 (EV2) | Native look & feel |
| Tree navigation | Tier 1 (EV2) | Native performance |
| Data grids (1M+ rows) | Tier 1 (EV2) | Virtual scrolling |
| File/color/font dialogs | Tier 1 (EV2) | System integration |
| Buttons with gradients | Tier 2 (Cairo) | Visual polish |
| Cards with shadows | Tier 2 (Cairo) | Modern appearance |
| Custom charts | Tier 2 (PLplot) | Scientific plotting |
| Rich text editing | Tier 3 (webview) | Full WYSIWYG |
| Interactive dashboards | Tier 3 (webview) | Modern web stack |
| Code editing | Tier 3 (Monaco) | Syntax highlighting |
| Complex animated forms | Tier 3 (webview) | CSS animations |

## Design Decisions

### 1. Three Tiers Coexist
An application can mix all three tiers:

```eiffel
sv.window ("My App")
    .content (
        sv.split
            .left (sv.tree)                    -- Tier 1: Native tree
            .right (
                sv.column.children (<<
                    sv.cairo_panel.gradient (...),  -- Tier 2: Cairo
                    sv.web_view.load_htmx (...)     -- Tier 3: Web
                >>)
            )
    )
```

### 2. Graceful Degradation
If a C library isn't available, fall back to Tier 1:

```eiffel
sv.card
    .shadow (sv.shadows.md)  -- Uses Cairo if available, else skip
```

### 3. Pure Eiffel Logic Everywhere
All business logic stays in Eiffel regardless of rendering tier:
- Data binding logic (pure Eiffel)
- State machine logic (pure Eiffel)
- Validation logic (pure Eiffel)
- Web panel communication (Eiffel ↔ JS bridge)

### 4. Dependency Management
Each tier has optional dependencies:

| Tier | Required | Optional |
|------|----------|----------|
| 1 | EiffelVision-2 | (none) |
| 2 | simple_cairo | simple_blend2d, simple_stb |
| 3 | simple_webview | simple_htmx, simple_alpine |

### 5. Documentation Notes

```eiffel
feature -- Shadows (Tier 2 Enhanced)
    shadow (a_shadow: SV_SHADOW): like Current
            -- Apply drop shadow effect.
            -- Uses Cairo if available (true drop shadow).
            -- Falls back to overlay simulation if Cairo unavailable.
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

# INNOVATION 14: GUI Testing Automation Harness

## The Problem

Testing GUI applications is traditionally:
1. **Manual clicking** - Slow, unreliable, doesn't scale
2. **Record/playback tools** - Brittle, break on layout changes
3. **Accessibility-based automation** - Complex setup, platform-specific

Most Eiffel GUI logic goes untested because the barrier is too high.

## The Solution: SV_TEST_HARNESS

A testing framework that treats simple_vision widgets like debuggable code, drawing inspiration from EiffelStudio's W_code debugger hooks.

### Architecture Parallel

| EiffelStudio Debugger | simple_vision Test Harness |
|----------------------|---------------------------|
| W_code (instrumented) | Widgets with logging hooks |
| F_code (production) | Widgets with hooks disabled |
| Breakpoints | Event injection points |
| Object inspection | Widget state queries |
| Step execution | Script-driven playback |
| Debug output window | simple_logger output |

### Three-Layer Testing

```
┌─────────────────────────────────────────────────────────────────┐
│  Autotest (Unit/Integration)                                    │  ← Already have this
├─────────────────────────────────────────────────────────────────┤
│  simple_vision_test (GUI Automation Harness)                    │  ← NEW
│  ┌─────────────────────────────────────────────────────────────┐│
│  │ Script Player (JSON/TOML use-cases)                         ││
│  │ Event Injector (mouse moves, clicks, keystrokes → EV)       ││
│  │ simple_logger integration (what happened log)               ││
│  └─────────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────────┤
│  simple_vision (GUI Library)                                    │
│  - Widgets emit loggable events                                 │
│  - All state is queryable                                       │
│  - Optional instrumentation mode                                │
└─────────────────────────────────────────────────────────────────┘
```

## The Harness API

### Widget Instrumentation (Built into SV_WIDGET)

```eiffel
feature {SV_TEST_HARNESS} -- Instrumentation (always present, cheap when disabled)

    test_harness: detachable SV_TEST_HARNESS
            -- Attached harness (like debugger attachment).

    attach_harness (a_harness: SV_TEST_HARNESS)
            -- Attach test harness to this widget.
        do
            test_harness := a_harness
        end

    notify_event (a_event: SV_EVENT)
            -- Notify harness of widget event.
        do
            if attached test_harness as h then
                h.record_event (Current, a_event)
            end
        end
```

### Test Harness Core

```eiffel
class SV_TEST_HARNESS

feature -- Configuration
    load_script (a_path: PATH)
            -- Load JSON/TOML use-case script.

    set_logger (a_logger: SIMPLE_LOGGER)
            -- Configure logging destination.

feature -- Script Playback
    play
            -- Execute all steps in loaded script.

    step
            -- Execute next step only.

    pause
    resume
    stop

feature -- Event Injection
    simulate_click (a_widget: SV_WIDGET)
    simulate_double_click (a_widget: SV_WIDGET)
    simulate_key_press (a_key: INTEGER; a_modifiers: INTEGER)
    simulate_key_sequence (a_text: STRING)
    simulate_mouse_move (a_x, a_y: INTEGER)
    simulate_drag_drop (a_from, a_to: SV_WIDGET)

feature -- State Queries
    assert_text (a_widget: SV_TEXT; a_expected: STRING)
    assert_enabled (a_widget: SV_WIDGET)
    assert_disabled (a_widget: SV_WIDGET)
    assert_visible (a_widget: SV_WIDGET)
    assert_hidden (a_widget: SV_WIDGET)
    assert_focus (a_widget: SV_WIDGET)
    assert_child_count (a_container: SV_CONTAINER; a_count: INTEGER)

feature -- Event Recording
    record_event (a_widget: SV_WIDGET; a_event: SV_EVENT)
            -- Log event for verification.

    events: LIST [TUPLE [widget: SV_WIDGET; event: SV_EVENT; timestamp: DATE_TIME]]
            -- Recorded events for assertion.

feature -- Recording Mode
    start_recording
            -- Record user interactions for playback.

    stop_recording
            -- Stop recording.

    save_recording (a_path: PATH)
            -- Save recorded interactions as script.
end
```

### Use-Case Script Format (JSON)

```json
{
  "name": "Login Flow",
  "steps": [
    { "action": "type", "target": "username_field", "value": "admin" },
    { "action": "type", "target": "password_field", "value": "secret123" },
    { "action": "click", "target": "login_button" },
    { "action": "wait", "milliseconds": 500 },
    { "action": "assert_visible", "target": "dashboard_panel" },
    { "action": "assert_text", "target": "welcome_label", "value": "Welcome, Admin!" }
  ]
}
```

### Test Class Usage

```eiffel
class LOGIN_TESTS

inherit
    SIMPLE_TEST_SET

feature -- Tests

    test_successful_login
        local
            harness: SV_TEST_HARNESS
            app: MY_APPLICATION
        do
            create harness.make
            create app.make_for_testing (harness)

            harness.load_script (test_data_path / "login_success.json")
            harness.play

            assert_true (harness.all_assertions_passed)
        end

    test_invalid_password
        local
            harness: SV_TEST_HARNESS
            app: MY_APPLICATION
        do
            create harness.make
            create app.make_for_testing (harness)

            harness.simulate_key_sequence ("admin")
            harness.simulate_click (app.login_form.password_field)
            harness.simulate_key_sequence ("wrongpass")
            harness.simulate_click (app.login_form.login_button)

            assert_true (app.login_form.error_label.is_visible)
            assert_text (app.login_form.error_label, "Invalid credentials")
        end
```

## AI-Powered Visual Validation (Advanced)

### The Problem with Layout Testing

Automated testing can verify:
- Widget state (enabled, visible, text content)
- Event firing
- Data binding

But **cannot easily verify**:
- Visual layout correctness
- Color/font rendering
- Spacing/alignment
- Overall "does this look right?"

### The AI Solution

```eiffel
class SV_AI_VISUAL_VALIDATOR

feature -- Configuration
    set_ai_client (a_client: SIMPLE_AI_CLIENT)
            -- Configure Claude/GPT/Ollama connection.

feature -- Capture
    capture_window (a_window: SV_WINDOW): SV_SCREENSHOT
            -- Capture window to clipboard/file.

feature -- Validation
    validate_against_description (a_screenshot: SV_SCREENSHOT; a_description: STRING): BOOLEAN
            -- Ask AI: "Does this screenshot match this description?"
            -- Returns True if AI confirms match.

    validate_against_state (a_screenshot: SV_SCREENSHOT; a_state: STRING; a_state_descriptions: TABLE [STRING, STRING]): BOOLEAN
            -- "Given state 'logged_in', does screenshot match the description for that state?"

    describe_differences (a_current, a_expected: SV_SCREENSHOT): STRING
            -- AI describes what's different between two screenshots.

    suggest_fixes (a_screenshot: SV_SCREENSHOT; a_expected_description: STRING): STRING
            -- AI suggests what code changes would fix visual issues.
end
```

### State Machine Integration

```eiffel
class MY_APP_VISUAL_TESTS

feature -- State Descriptions
    state_descriptions: TABLE [STRING, STRING]
        once
            create Result.make (10)
            Result.put ("A login form centered in the window with username field, ~
                         password field, and blue Login button. Clean white background.",
                        "initial")
            Result.put ("The same login form but with a red error banner above the fields ~
                         saying 'Invalid credentials'. The Login button should still be enabled.",
                        "error")
            Result.put ("A dashboard view with a left sidebar navigation, ~
                         header showing 'Welcome, [username]', and a card grid layout.",
                        "logged_in")
        end

feature -- Tests
    test_visual_states
        local
            validator: SV_AI_VISUAL_VALIDATOR
            harness: SV_TEST_HARNESS
            screenshot: SV_SCREENSHOT
        do
            create validator.make
            create harness.make

            -- Initial state
            screenshot := validator.capture_window (app.main_window)
            assert_true (validator.validate_against_state (screenshot, "initial", state_descriptions))

            -- Error state
            harness.simulate_click (app.login_form.login_button)
            harness.wait (500)
            screenshot := validator.capture_window (app.main_window)
            assert_true (validator.validate_against_state (screenshot, "error", state_descriptions))

            -- Success state
            harness.simulate_key_sequence ("admin")
            harness.simulate_click (app.login_form.password_field)
            harness.simulate_key_sequence ("correctpass")
            harness.simulate_click (app.login_form.login_button)
            harness.wait (1000)
            screenshot := validator.capture_window (app.main_window)
            assert_true (validator.validate_against_state (screenshot, "logged_in", state_descriptions))
        end
```

## Implementation Notes

### Phase 1 (Basic Harness)
1. Widget instrumentation hooks (low overhead when disabled)
2. Event injection (simulate_click, simulate_key)
3. State assertions (assert_text, assert_visible)
4. simple_logger integration

### Phase 2 (Script-Driven)
1. JSON/TOML script parser
2. Script playback engine
3. Recording mode (record user interactions)
4. Step debugging

### Phase 3 (AI Visual)
1. Screenshot capture (clipboard integration)
2. simple_ai_client integration
3. State machine visual descriptions
4. AI-powered "does this look right?" validation

### Dependencies
- simple_testing (test framework)
- simple_logger (event logging)
- simple_ai_client (visual validation, optional)
- simple_json (script parsing)

## Benefits

1. **80% of GUI logic testable without human**
   - Event handling
   - Data binding
   - State transitions
   - Widget state

2. **20% visual validation with AI assist**
   - Layout correctness
   - Visual regression
   - "Does it look right?" checks

3. **Integration with existing Autotest**
   - GUI tests run alongside unit tests
   - Same test infrastructure
   - CI/CD compatible

4. **Recording → Playback workflow**
   - Humans/AI record use-cases once
   - Scripts replay automatically
   - Build test suite incrementally

---

*simple_vision — Not just wrapping EV2. Revolutionizing Eiffel GUI development.*
