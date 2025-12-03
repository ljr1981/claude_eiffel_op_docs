# simple_alpine: Build Plan

## Executive Summary

Create a fluent Eiffel library for generating Alpine.js attributes, mirroring the design of simple_htmx. This library enables type-safe, server-side generation of Alpine.js directives for client-side interactivity without server round-trips.

**Goal**: Generate Alpine.js attributes from Eiffel with the same fluent elegance as simple_htmx, enabling dark mode toggles, dropdowns, modals, and other client-side interactions.

---

## Research Summary

### Alpine.js Overview

Alpine.js is a lightweight (15KB) reactive JavaScript framework that adds interactivity directly in HTML via directives (prefixed with `x-`). It's the perfect complement to HTMX:

| Library | Purpose | When Used |
|---------|---------|-----------|
| HTMX | Server interactions | Fetching data, submitting forms, partial page updates |
| Alpine.js | Client-side state | Toggles, dropdowns, modals, dark mode - no server needed |

### Core Alpine.js Components

#### 1. Directives (18 total)

| Directive | Purpose | Priority |
|-----------|---------|----------|
| `x-data` | Initialize reactive state | **High** |
| `x-show` | Toggle visibility (CSS display) | **High** |
| `x-bind` | Bind attributes reactively | **High** |
| `x-on` | Attach event listeners | **High** |
| `x-text` | Set text content | **High** |
| `x-model` | Two-way form binding | **High** |
| `x-if` | Conditionally add/remove elements | **High** |
| `x-for` | Loop rendering | **High** |
| `x-transition` | CSS transitions | **High** |
| `x-init` | Run code on initialization | Medium |
| `x-html` | Set HTML content | Medium |
| `x-ref` | Named element references | Medium |
| `x-effect` | Reactive side effects | Medium |
| `x-cloak` | Hide until Alpine initializes | Medium |
| `x-ignore` | Skip Alpine processing | Low |
| `x-teleport` | Move elements in DOM | Low |
| `x-modelable` | Custom two-way binding | Low |
| `x-id` | Generate unique IDs | Low |

#### 2. Magic Properties (9 total)

| Magic | Purpose | Priority |
|-------|---------|----------|
| `$el` | Current element | **High** |
| `$refs` | Named element references | **High** |
| `$store` | Global state access | **High** |
| `$dispatch` | Fire custom events | **High** |
| `$watch` | Watch reactive data | Medium |
| `$nextTick` | Run after DOM update | Medium |
| `$root` | Root component element | Low |
| `$data` | Component data object | Low |
| `$id` | Generate unique IDs | Low |

#### 3. Event Modifiers (for x-on)

| Category | Modifiers |
|----------|-----------|
| **Behavior** | `.prevent`, `.stop`, `.self` |
| **Timing** | `.debounce`, `.throttle`, `.once` |
| **Target** | `.window`, `.document`, `.outside` |
| **Options** | `.passive`, `.capture` |
| **Naming** | `.camel`, `.dot` |
| **Keys** | `.enter`, `.escape`, `.shift`, `.ctrl`, `.alt`, `.meta` |

#### 4. Official Plugins

| Plugin | Purpose | Include? |
|--------|---------|----------|
| **Collapse** | Smooth expand/collapse animations | Yes |
| **Persist** | Persist state across page loads | Yes |
| **Focus** | Focus management, trap focus in modals | Yes |
| **Intersect** | Trigger on scroll into view | Maybe |
| **Mask** | Format input as user types | Maybe |
| **Morph** | Morph HTML while preserving state | No (HTMX does this) |
| **Anchor** | Position elements relative to others | Maybe |
| **Sort** | Drag-and-drop sorting | Maybe |

---

## Library Architecture

### Design Principles

1. **Mirror simple_htmx** - Same fluent interface pattern
2. **Type-safe** - Catch typos at compile time
3. **Composable** - Mix with simple_htmx on same elements
4. **Minimal** - Only generate attributes, no behavior

### Class Structure

```
simple_alpine/
├── simple_alpine.ecf
├── src/
│   ├── core/
│   │   ├── alpine_element.e           # Base class with all directives
│   │   └── alpine_factory.e           # Convenience factory
│   ├── directives/
│   │   ├── alpine_data_builder.e      # x-data JSON builder
│   │   ├── alpine_transition.e        # x-transition helpers
│   │   └── alpine_event_builder.e     # x-on with modifiers
│   └── plugins/
│       ├── alpine_collapse.e          # x-collapse directive
│       ├── alpine_persist.e           # $persist helpers
│       └── alpine_focus.e             # x-trap directive
└── testing/
    ├── test_alpine_directives.e
    ├── test_alpine_events.e
    ├── test_alpine_transitions.e
    └── test_alpine_integration.e      # Combined with simple_htmx
```

### Core Classes

#### ALPINE_ELEMENT (Base Class)

```eiffel
class
    ALPINE_ELEMENT

feature -- Directives: State

    x_data (a_state: STRING): like Current
            -- Initialize reactive state
            -- Example: x_data ("{ open: false }") → x-data="{ open: false }"
        do
            set_attribute ("x-data", a_state)
            Result := Current
        end

    x_data_empty: like Current
            -- Initialize empty state (common for event-only components)
            -- Example: x_data_empty → x-data
        do
            set_attribute ("x-data", "")
            Result := Current
        end

    x_init (a_expression: STRING): like Current
            -- Run expression on initialization
        do
            set_attribute ("x-init", a_expression)
            Result := Current
        end

feature -- Directives: Visibility

    x_show (a_expression: STRING): like Current
            -- Toggle visibility based on expression
            -- Example: x_show ("open") → x-show="open"
        do
            set_attribute ("x-show", a_expression)
            Result := Current
        end

    x_if (a_expression: STRING): like Current
            -- Conditionally add/remove from DOM
            -- Note: Must be on <template> element
        do
            set_attribute ("x-if", a_expression)
            Result := Current
        end

    x_cloak: like Current
            -- Hide element until Alpine initializes
        do
            set_attribute ("x-cloak", "")
            Result := Current
        end

feature -- Directives: Content

    x_text (a_expression: STRING): like Current
            -- Set text content reactively
        do
            set_attribute ("x-text", a_expression)
            Result := Current
        end

    x_html (a_expression: STRING): like Current
            -- Set HTML content reactively
        do
            set_attribute ("x-html", a_expression)
            Result := Current
        end

feature -- Directives: Binding

    x_bind (a_attribute, a_expression: STRING): like Current
            -- Bind any attribute to expression
            -- Example: x_bind ("class", "open ? 'active' : ''")
        do
            set_attribute ("x-bind:" + a_attribute, a_expression)
            Result := Current
        end

    x_bind_class (a_expression: STRING): like Current
            -- Shorthand for x-bind:class
        do
            set_attribute (":class", a_expression)
            Result := Current
        end

    x_bind_style (a_expression: STRING): like Current
            -- Shorthand for x-bind:style
        do
            set_attribute (":style", a_expression)
            Result := Current
        end

    x_bind_disabled (a_expression: STRING): like Current
            -- Shorthand for x-bind:disabled
        do
            set_attribute (":disabled", a_expression)
            Result := Current
        end

feature -- Directives: Events

    x_on (a_event, a_expression: STRING): like Current
            -- Attach event listener
            -- Example: x_on ("click", "open = !open")
        do
            set_attribute ("x-on:" + a_event, a_expression)
            Result := Current
        end

    x_on_click (a_expression: STRING): like Current
            -- Shorthand for x-on:click / @click
        do
            set_attribute ("@click", a_expression)
            Result := Current
        end

    x_on_click_outside (a_expression: STRING): like Current
            -- Click outside element
        do
            set_attribute ("@click.outside", a_expression)
            Result := Current
        end

    x_on_keydown_escape (a_expression: STRING): like Current
            -- Escape key press
        do
            set_attribute ("@keydown.escape", a_expression)
            Result := Current
        end

    x_on_keydown_enter (a_expression: STRING): like Current
            -- Enter key press
        do
            set_attribute ("@keydown.enter", a_expression)
            Result := Current
        end

    x_on_submit_prevent (a_expression: STRING): like Current
            -- Form submit with preventDefault
        do
            set_attribute ("@submit.prevent", a_expression)
            Result := Current
        end

feature -- Directives: Forms

    x_model (a_property: STRING): like Current
            -- Two-way binding for form inputs
        do
            set_attribute ("x-model", a_property)
            Result := Current
        end

    x_model_debounce (a_property: STRING; a_ms: INTEGER): like Current
            -- Two-way binding with debounce
        do
            set_attribute ("x-model.debounce." + a_ms.out + "ms", a_property)
            Result := Current
        end

feature -- Directives: Loops

    x_for (a_expression: STRING): like Current
            -- Loop rendering
            -- Example: x_for ("item in items")
            -- Note: Must be on <template> element
        do
            set_attribute ("x-for", a_expression)
            Result := Current
        end

feature -- Directives: References

    x_ref (a_name: STRING): like Current
            -- Name element for $refs access
        do
            set_attribute ("x-ref", a_name)
            Result := Current
        end

feature -- Directives: Transitions

    x_transition: like Current
            -- Apply default enter/leave transitions
        do
            set_attribute ("x-transition", "")
            Result := Current
        end

    x_transition_opacity: like Current
            -- Fade in/out transition
        do
            set_attribute ("x-transition.opacity", "")
            Result := Current
        end

    x_transition_scale (a_origin: STRING): like Current
            -- Scale transition with origin
            -- Example: x_transition_scale ("top") → x-transition.scale.origin.top
        do
            set_attribute ("x-transition.scale.origin." + a_origin, "")
            Result := Current
        end

    x_transition_duration (a_ms: INTEGER): like Current
            -- Set transition duration
        do
            set_attribute ("x-transition.duration." + a_ms.out + "ms", "")
            Result := Current
        end

feature -- Directives: Effects

    x_effect (a_expression: STRING): like Current
            -- Run side effect when dependencies change
        do
            set_attribute ("x-effect", a_expression)
            Result := Current
        end

feature -- Plugin: Collapse

    x_collapse: like Current
            -- Smooth height transition (requires collapse plugin)
        do
            set_attribute ("x-collapse", "")
            Result := Current
        end

    x_collapse_duration (a_ms: INTEGER): like Current
            -- Collapse with custom duration
        do
            set_attribute ("x-collapse.duration." + a_ms.out + "ms", "")
            Result := Current
        end

feature -- Plugin: Focus/Trap

    x_trap (a_expression: STRING): like Current
            -- Trap focus inside element (requires focus plugin)
            -- Useful for modals
        do
            set_attribute ("x-trap", a_expression)
            Result := Current
        end

    x_trap_inert (a_expression: STRING): like Current
            -- Trap focus and make outside elements inert
        do
            set_attribute ("x-trap.inert", a_expression)
            Result := Current
        end

feature -- Magic Property Helpers (for use in expressions)

    el: STRING = "$el"
            -- Reference to current element

    refs (a_name: STRING): STRING
            -- Reference to named element
        do
            Result := "$refs." + a_name
        end

    store (a_name: STRING): STRING
            -- Reference to global store
        do
            Result := "$store." + a_name
        end

    dispatch (a_event: STRING): STRING
            -- Dispatch custom event
        do
            Result := "$dispatch('" + a_event + "')"
        end

    dispatch_with_data (a_event, a_data: STRING): STRING
            -- Dispatch event with data
        do
            Result := "$dispatch('" + a_event + "', " + a_data + ")"
        end

    next_tick (a_expression: STRING): STRING
            -- Run after DOM update
        do
            Result := "$nextTick(() => { " + a_expression + " })"
        end

    watch (a_property, a_callback: STRING): STRING
            -- Watch property for changes
        do
            Result := "$watch('" + a_property + "', " + a_callback + ")"
        end

end
```

#### ALPINE_FACTORY

```eiffel
class
    ALPINE_FACTORY

feature -- Common Patterns

    toggle (a_property: STRING): STRING
            -- Generate toggle expression
            -- Example: toggle ("open") → "open = !open"
        do
            Result := a_property + " = !" + a_property
        end

    set_true (a_property: STRING): STRING
            -- Set property to true
        do
            Result := a_property + " = true"
        end

    set_false (a_property: STRING): STRING
            -- Set property to false
        do
            Result := a_property + " = false"
        end

    ternary (a_condition, a_true_value, a_false_value: STRING): STRING
            -- Generate ternary expression
            -- Example: ternary ("open", "'block'", "'none'")
        do
            Result := a_condition + " ? " + a_true_value + " : " + a_false_value
        end

    class_if (a_class, a_condition: STRING): STRING
            -- Conditional class binding
            -- Example: class_if ("active", "isActive") → "{ 'active': isActive }"
        do
            Result := "{ '" + a_class + "': " + a_condition + " }"
        end

feature -- Data Builders

    data_bool (a_name: STRING; a_value: BOOLEAN): STRING
            -- Boolean property
        do
            Result := a_name + ": " + a_value.out.as_lower
        end

    data_string (a_name, a_value: STRING): STRING
            -- String property
        do
            Result := a_name + ": '" + a_value + "'"
        end

    data_int (a_name: STRING; a_value: INTEGER): STRING
            -- Integer property
        do
            Result := a_name + ": " + a_value.out
        end

    data_object (a_properties: ARRAY [STRING]): STRING
            -- Combine properties into x-data object
            -- Example: data_object (<<data_bool ("open", false), data_int ("count", 0)>>)
            --          → "{ open: false, count: 0 }"
        local
            i: INTEGER
        do
            create Result.make_from_string ("{ ")
            from i := a_properties.lower until i > a_properties.upper loop
                if i > a_properties.lower then
                    Result.append (", ")
                end
                Result.append (a_properties [i])
                i := i + 1
            end
            Result.append (" }")
        end

feature -- Pre-built Component Patterns

    dark_mode_toggle_data: STRING
            -- Standard dark mode x-data
        do
            Result := "{ %N%
                %dark: localStorage.getItem('darkMode') === 'true' || %N%
                %      (!localStorage.getItem('darkMode') && %N%
                %       window.matchMedia('(prefers-color-scheme: dark)').matches)%N%
                %}"
        end

    dark_mode_toggle_init: STRING
            -- Standard dark mode x-init
        do
            Result := "$watch('dark', val => { %N%
                %localStorage.setItem('darkMode', val); %N%
                %document.documentElement.classList.toggle('dark', val) %N%
                %})"
        end

    dropdown_data: STRING
            -- Standard dropdown x-data
        do
            Result := "{ open: false }"
        end

    modal_data: STRING
            -- Standard modal x-data
        do
            Result := "{ open: false }"
        end

    accordion_data: STRING
            -- Standard accordion x-data (single open)
        do
            Result := "{ activeIndex: null }"
        end

    tabs_data (a_default_tab: STRING): STRING
            -- Standard tabs x-data
        do
            Result := "{ activeTab: '" + a_default_tab + "' }"
        end

end
```

---

## Integration with simple_htmx

### Combined Usage Pattern

```eiffel
-- Element with both HTMX and Alpine attributes
Result := html.div
    -- Alpine: client-side state
    .x_data ("{ open: false }")
    .x_show ("open")
    .x_transition
    -- HTMX: server interaction
    .hx_get ("/api/content")
    .hx_trigger ("click")
    .hx_target ("#result")
    .to_html_8
```

### Inheritance Strategy

**Option A**: Separate classes, use both
```eiffel
class MY_HANDLER
inherit
    HTMX_FACTORY
    ALPINE_FACTORY
```

**Option B**: ALPINE_ELEMENT inherits HTMX_ELEMENT
```eiffel
class ALPINE_ELEMENT
inherit
    HTMX_ELEMENT
        -- All HTMX methods available plus Alpine methods
```

**Recommendation**: Option B - single element with all capabilities. User doesn't need to think about which factory to use.

---

## Test Cases

### Unit Tests

```eiffel
class TEST_ALPINE_DIRECTIVES

feature -- x-data tests

    test_x_data_simple
        local
            l_div: ALPINE_DIV
        do
            l_div := alpine.div.x_data ("{ open: false }")
            assert_equal ("<div x-data=%"{ open: false }%"></div>", l_div.to_html_8)
        end

    test_x_data_empty
        local
            l_div: ALPINE_DIV
        do
            l_div := alpine.div.x_data_empty
            assert_equal ("<div x-data></div>", l_div.to_html_8)
        end

feature -- x-show tests

    test_x_show
        local
            l_div: ALPINE_DIV
        do
            l_div := alpine.div.x_show ("isVisible")
            assert_equal ("<div x-show=%"isVisible%"></div>", l_div.to_html_8)
        end

feature -- x-on tests

    test_x_on_click
        local
            l_btn: ALPINE_BUTTON
        do
            l_btn := alpine.button.x_on_click ("open = !open").text ("Toggle")
            assert_equal ("<button @click=%"open = !open%">Toggle</button>", l_btn.to_html_8)
        end

    test_x_on_click_outside
        local
            l_div: ALPINE_DIV
        do
            l_div := alpine.div.x_on_click_outside ("open = false")
            assert_equal ("<div @click.outside=%"open = false%"></div>", l_div.to_html_8)
        end

    test_x_on_keydown_escape
        local
            l_div: ALPINE_DIV
        do
            l_div := alpine.div.x_on_keydown_escape ("open = false")
            assert_equal ("<div @keydown.escape=%"open = false%"></div>", l_div.to_html_8)
        end

feature -- x-bind tests

    test_x_bind_class
        local
            l_div: ALPINE_DIV
        do
            l_div := alpine.div.x_bind_class ("{ 'active': isActive }")
            assert_equal ("<div :class=%"{ 'active': isActive }%"></div>", l_div.to_html_8)
        end

feature -- x-model tests

    test_x_model
        local
            l_input: ALPINE_INPUT
        do
            l_input := alpine.input_text ("search").x_model ("query")
            assert_equal ("<input type=%"text%" name=%"search%" x-model=%"query%"/>", l_input.to_html_8)
        end

feature -- x-transition tests

    test_x_transition_default
        local
            l_div: ALPINE_DIV
        do
            l_div := alpine.div.x_show ("open").x_transition
            assert_equal ("<div x-show=%"open%" x-transition></div>", l_div.to_html_8)
        end

    test_x_transition_opacity
        local
            l_div: ALPINE_DIV
        do
            l_div := alpine.div.x_show ("open").x_transition_opacity
            assert_equal ("<div x-show=%"open%" x-transition.opacity></div>", l_div.to_html_8)
        end

feature -- x-for tests

    test_x_for
        local
            l_template: ALPINE_TEMPLATE
        do
            l_template := alpine.template.x_for ("item in items")
            assert_equal ("<template x-for=%"item in items%"></template>", l_template.to_html_8)
        end

feature -- Plugin tests

    test_x_collapse
        local
            l_div: ALPINE_DIV
        do
            l_div := alpine.div.x_show ("open").x_collapse
            assert_equal ("<div x-show=%"open%" x-collapse></div>", l_div.to_html_8)
        end

    test_x_trap
        local
            l_div: ALPINE_DIV
        do
            l_div := alpine.div.x_trap ("open")
            assert_equal ("<div x-trap=%"open%"></div>", l_div.to_html_8)
        end

end
```

### Integration Tests (with simple_htmx)

```eiffel
class TEST_ALPINE_HTMX_INTEGRATION

feature -- Combined usage tests

    test_alpine_and_htmx_on_same_element
        local
            l_div: ALPINE_DIV
        do
            l_div := alpine.div
                .x_data ("{ loading: false }")
                .x_bind_class ("{ 'opacity-50': loading }")
                .hx_get ("/api/data")
                .hx_trigger ("click")
                .hx_indicator ("#spinner")
            assert_has_substring (l_div.to_html_8, "x-data")
            assert_has_substring (l_div.to_html_8, "hx-get")
        end

    test_htmx_trigger_with_alpine_state
        local
            l_btn: ALPINE_BUTTON
        do
            -- HTMX fetches, Alpine shows loading state
            l_btn := alpine.button
                .x_on_click ("loading = true")
                .hx_get ("/api/submit")
                .hx_target ("#result")
                .attr ("hx-on::after-request", "loading = false")  -- HTMX event
            assert_has_substring (l_btn.to_html_8, "@click")
            assert_has_substring (l_btn.to_html_8, "hx-get")
        end

end
```

### Component Pattern Tests

```eiffel
class TEST_ALPINE_COMPONENTS

feature -- Dark mode toggle

    test_dark_mode_toggle
        local
            l_factory: ALPINE_FACTORY
            l_btn: ALPINE_BUTTON
        do
            create l_factory
            l_btn := alpine.button
                .x_data (l_factory.dark_mode_toggle_data)
                .x_init (l_factory.dark_mode_toggle_init)
                .x_on_click (l_factory.toggle ("dark"))
                .x_text (l_factory.ternary ("dark", "'Light Mode'", "'Dark Mode'"))
            -- Verify key attributes present
            assert_has_substring (l_btn.to_html_8, "localStorage")
            assert_has_substring (l_btn.to_html_8, "$watch")
        end

feature -- Dropdown

    test_dropdown_pattern
        local
            l_container, l_menu: ALPINE_DIV
            l_btn: ALPINE_BUTTON
        do
            l_container := alpine.div.x_data ("{ open: false }")
            l_btn := alpine.button
                .x_on_click ("open = !open")
                .text ("Menu")
            l_menu := alpine.div
                .x_show ("open")
                .x_on_click_outside ("open = false")
                .x_on_keydown_escape ("open = false")
                .x_transition
            -- Just verify they compile and generate expected attributes
            assert_has_substring (l_container.to_html_8, "x-data")
            assert_has_substring (l_menu.to_html_8, "@click.outside")
        end

feature -- Modal

    test_modal_pattern
        local
            l_modal: ALPINE_DIV
        do
            l_modal := alpine.div
                .x_show ("open")
                .x_transition_opacity
                .x_trap ("open")
                .x_on_keydown_escape ("open = false")
            assert_has_substring (l_modal.to_html_8, "x-trap")
            assert_has_substring (l_modal.to_html_8, "x-transition.opacity")
        end

feature -- Accordion

    test_accordion_item_pattern
        local
            l_header: ALPINE_BUTTON
            l_content: ALPINE_DIV
        do
            l_header := alpine.button
                .x_on_click ("activeIndex = activeIndex === 0 ? null : 0")
            l_content := alpine.div
                .x_show ("activeIndex === 0")
                .x_collapse
            assert_has_substring (l_content.to_html_8, "x-collapse")
        end

feature -- Tabs

    test_tabs_pattern
        local
            l_factory: ALPINE_FACTORY
            l_tab_btn: ALPINE_BUTTON
            l_tab_content: ALPINE_DIV
        do
            create l_factory
            l_tab_btn := alpine.button
                .x_on_click ("activeTab = 'settings'")
                .x_bind_class (l_factory.class_if ("active", "activeTab === 'settings'"))
            l_tab_content := alpine.div
                .x_show ("activeTab === 'settings'")
            assert_has_substring (l_tab_btn.to_html_8, "activeTab = 'settings'")
        end

end
```

---

## Implementation Order

### Phase 1: Core (Day 1 Morning - 3 hours)

| # | Task | Tests |
|---|------|-------|
| 1 | Create simple_alpine.ecf | - |
| 2 | ALPINE_ELEMENT base class | 5 |
| 3 | Basic directives: x-data, x-show, x-if | 6 |
| 4 | Text directives: x-text, x-html | 4 |
| 5 | Reference: x-ref, x-cloak | 3 |

### Phase 2: Events & Binding (Day 1 Afternoon - 3 hours)

| # | Task | Tests |
|---|------|-------|
| 6 | x-on with common events | 8 |
| 7 | Event modifiers (.prevent, .stop, .outside, etc.) | 6 |
| 8 | Key modifiers (.escape, .enter, etc.) | 5 |
| 9 | x-bind and shortcuts (:class, :style, :disabled) | 6 |
| 10 | x-model and modifiers | 4 |

### Phase 3: Control Flow & Transitions (Day 1 Evening - 2 hours)

| # | Task | Tests |
|---|------|-------|
| 11 | x-for directive | 3 |
| 12 | x-transition and variants | 5 |
| 13 | x-effect | 2 |
| 14 | ALPINE_TEMPLATE class (for x-if, x-for) | 3 |

### Phase 4: Plugins & Factory (Day 2 Morning - 2 hours)

| # | Task | Tests |
|---|------|-------|
| 15 | x-collapse (plugin) | 3 |
| 16 | x-trap (focus plugin) | 3 |
| 17 | ALPINE_FACTORY helper methods | 6 |
| 18 | Pre-built component patterns | 5 |

### Phase 5: Integration & Polish (Day 2 Afternoon - 2 hours)

| # | Task | Tests |
|---|------|-------|
| 19 | Integrate with HTMX_ELEMENT (inheritance) | 4 |
| 20 | Magic property helpers | 4 |
| 21 | Integration tests (Alpine + HTMX) | 5 |
| 22 | Documentation (README, ROADMAP) | - |

---

## Estimated Output

| Metric | Estimate |
|--------|----------|
| **Classes** | ~10 |
| **Lines of Code** | ~1,500-2,000 |
| **Test Cases** | 80+ |
| **Development Time** | 10-12 hours |

---

## Files to Create

```
D:\prod\simple_alpine\
├── simple_alpine.ecf
├── README.md
├── ROADMAP.md
├── src/
│   ├── core/
│   │   ├── alpine_element.e
│   │   └── alpine_factory.e
│   ├── elements/
│   │   ├── alpine_div.e
│   │   ├── alpine_button.e
│   │   ├── alpine_input.e
│   │   ├── alpine_template.e
│   │   └── ... (inherit from simple_htmx elements)
│   └── plugins/
│       ├── alpine_collapse.e
│       └── alpine_focus.e
└── testing/
    ├── application.e
    ├── test_alpine_directives.e
    ├── test_alpine_events.e
    ├── test_alpine_transitions.e
    ├── test_alpine_plugins.e
    ├── test_alpine_factory.e
    └── test_alpine_htmx_integration.e
```

---

## Success Criteria

The library is complete when:

1. **All 18 directives** have type-safe methods
2. **Event modifiers** work correctly (.prevent, .outside, .escape, etc.)
3. **Transitions** generate correct x-transition variants
4. **Plugins** (collapse, focus) are supported
5. **Factory helpers** simplify common patterns
6. **Integration** with simple_htmx works seamlessly
7. **80+ tests** pass
8. **Documentation** is complete

---

## References

- [Alpine.js Directives Documentation](https://alpinejs.dev/directives/)
- [Alpine.js Magic Properties](https://alpinejs.dev/magics/refs)
- [Alpine.js Plugins](https://alpinejs.dev/plugins)
- [Alpine.js Events & Modifiers](https://alpinejs.dev/directives/on)
- [HTMX + Alpine.js Integration](https://www.infoworld.com/article/3856520/htmx-and-alpine-js-how-to-combine-two-great-lean-front-ends.html)
- [Alpine Toolbox Examples](https://www.alpinetoolbox.com/examples/)
- [Code with Hugo - Alpine Tips](https://codewithhugo.com/alpine-tips/)
- [Dark Mode with Alpine + Tailwind](https://caffeinecreations.ca/blog/dark-mode-with-alpinejs-and-tailwind/)

---

**Document Created**: December 3, 2025
**Target Start**: After simple_showcase deployment
**Estimated Duration**: 10-12 hours (1.5 days)
