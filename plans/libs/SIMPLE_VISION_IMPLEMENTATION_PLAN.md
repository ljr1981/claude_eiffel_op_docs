# simple_vision Implementation Plan

**Date:** December 28, 2025
**Author:** Larry Rix + Claude
**Status:** Research Complete, Ready for Implementation

---

## Executive Summary

**simple_vision** is a simplified, modernized wrapper library over EiffelVision-2 (EV2) that:

1. **Simplifies** the complex EV2 API into intuitive one-liner patterns
2. **Modernizes** terminology from 1990s European/Academic to 2025 US/American GUI conventions
3. **Provides** dual naming: classic EV2 names for veterans + modern names for new developers
4. **Delivers** an `SV_QUICK` fluent API for rapid GUI construction
5. **Extends** with a 3-tier rendering architecture:
   - **Tier 1:** Pure EV2 widgets (native, cross-platform)
   - **Tier 2:** C-enhanced graphics (Cairo gradients, shadows, blur)
   - **Tier 3:** Web panels (webview + HTMX + Alpine.js for modern UI)

See `SIMPLE_VISION_INNOVATIONS.md` for the complete innovation stack (14 innovations).

---

## Effort Estimate (Ball-Park)

Based on proven 5K LOC/day velocity:

### Phase Breakdown

| Phase | Est. LOC | Sessions |
|-------|----------|----------|
| 1. Foundation | ~2,500 | 1 |
| 2. Core Widgets | ~3,500 | 1 |
| 3. Advanced Layouts | ~3,500 | 1 |
| 4. Data Widgets (SV_DATA_GRID is meaty) | ~5,000 | 1 |
| 5. Dialogs & Polish | ~3,000 | 1 |
| 6. Styling & Theming | ~2,500 | 1 |
| 7. C Library Integration (Cairo, stb) | ~4,000 | 1 |
| 8. Web Panel Integration (webview) | ~5,000 | 1 |
| 9. Docs & Testing | ~3,000 | 1 |
| **Subtotal (Phases)** | **~32,000** | **9** |

### Innovations Not Explicitly in Phases

| Innovation | Est. LOC | Sessions |
|------------|----------|----------|
| Reactive Binding (SV_OBSERVABLE, SV_BINDING) | ~2,000 | ½ |
| State Machine (SV_STATE_MACHINE) | ~1,500 | ½ |
| Form System (SV_FORM, SV_FIELD, validation) | ~3,000 | 1 |
| Navigation (SV_NAVIGATOR) | ~2,000 | ½ |
| Constraint System (SV_CONSTRAINTS) | ~2,000 | ½ |
| GUI Testing Harness (SV_TEST_HARNESS) | ~3,000 | 1 |
| **Subtotal (Innovations)** | **~13,500** | **4** |

### Summary

| Scope | Sessions |
|-------|----------|
| **MVP (Tier 1 only, Phases 1-6)** | 6 |
| **Full Tier 1 + Innovations** | 9 |
| **Complete (all 3 tiers)** | 12 |
| **With buffer for integration/debugging** | 15 |

**Conservative estimate: 13-16 sessions** for the complete vision with all 14 innovations, 3 tiers, and polished documentation.

That's roughly 40,000-45,000 LOC — a substantial library, but achievable given our velocity.

---

## Part 1: EiffelVision-2 Architecture Analysis

### 1.1 Core Design Patterns

EV2 uses sophisticated architectural patterns that add complexity but enable cross-platform support:

#### Bridge Pattern (Core Architecture)
```
EV_BUTTON (public interface)
    |
    +-- implementation: EV_BUTTON_I (deferred interface)
                            |
                            +-- EV_BUTTON_IMP (Windows)
                            +-- EV_BUTTON_IMP (GTK2/GTK3)
                            +-- EV_BUTTON_IMP (Cocoa)
```

Every class has three layers:
- **Interface** (EV_*) - What developers use
- **Implementation Interface** (EV_*_I) - Platform-agnostic contract
- **Implementation** (EV_*_IMP) - Platform-specific code

#### Mixin/Trait Pattern
Widgets inherit capabilities through multiple inheritance:
```eiffel
EV_BUTTON inherits
    EV_PRIMITIVE          -- Base widget
    EV_TEXTABLE           -- Text content
    EV_PIXMAPABLE         -- Icon/image support
    EV_FONTABLE           -- Font styling
    EV_TEXT_ALIGNABLE     -- Alignment control
    EV_BUTTON_ACTION_SEQUENCES  -- Click events
```

#### Action Sequence Pattern (Events)
```eiffel
button.select_actions.extend (agent on_click)
window.close_request_actions.extend (agent on_close)
text_field.change_actions.extend (agent on_text_changed)
```

### 1.2 Class Counts from simple_kb

| Category | Count |
|----------|-------|
| Total EV_* classes | 382 |
| Concrete widgets | ~80 |
| Deferred (abstract) | ~150 |
| Implementation (*_I, *_IMP) | ~152 |
| Action sequences | ~40 |

### 1.3 Key Widget Categories

**Containers:**
- EV_WINDOW, EV_TITLED_WINDOW, EV_DIALOG
- EV_HORIZONTAL_BOX, EV_VERTICAL_BOX
- EV_TABLE, EV_FIXED, EV_NOTEBOOK
- EV_FRAME, EV_SCROLLABLE_AREA, EV_SPLIT_AREA

**Primitives:**
- EV_BUTTON, EV_TOGGLE_BUTTON, EV_CHECK_BUTTON, EV_RADIO_BUTTON
- EV_LABEL, EV_TEXT_FIELD, EV_TEXT, EV_COMBO_BOX
- EV_LIST, EV_TREE, EV_GRID
- EV_PROGRESS_BAR, EV_RANGE, EV_SPIN_BUTTON

**Dialogs:**
- EV_FILE_OPEN_DIALOG, EV_FILE_SAVE_DIALOG
- EV_COLOR_DIALOG, EV_FONT_DIALOG
- EV_MESSAGE_DIALOG, EV_CONFIRMATION_DIALOG

---

## Part 2: Modern GUI Terminology Mapping

### 2.1 Research Sources

Modern naming conventions from:
- [React Component Naming](https://www.sufle.io/blog/naming-conventions-in-react)
- [SwiftUI Components](https://fuckingswiftui.com/)
- [Flutter Widgets](https://docs.flutter.dev/ui/layout)
- [Smashing Magazine Best Practices](https://www.smashingmagazine.com/2024/05/naming-best-practices/)

### 2.2 Terminology Translation Table

| EV2 Term (1990s/Academic) | Modern Term (2025/US) | Notes |
|---------------------------|----------------------|-------|
| **Containers** | | |
| EV_HORIZONTAL_BOX | Row, HStack | SwiftUI: HStack, Flutter: Row |
| EV_VERTICAL_BOX | Column, VStack | SwiftUI: VStack, Flutter: Column |
| EV_TABLE | Grid, GridView | CSS Grid, Flutter GridView |
| EV_FIXED | AbsoluteLayout, Stack | Flutter: Stack with Positioned |
| EV_CELL | Container, Box | Generic wrapper |
| EV_FRAME | Card, Panel, GroupBox | Material: Card |
| EV_NOTEBOOK | TabView, TabBar | SwiftUI: TabView |
| EV_SPLIT_AREA | SplitView, Resizable | Modern split panes |
| EV_SCROLLABLE_AREA | ScrollView | Universal term |
| **Windows** | | |
| EV_WINDOW | Window | Same |
| EV_TITLED_WINDOW | MainWindow, AppWindow | Implies title bar |
| EV_DIALOG | Dialog, Modal | Same |
| EV_POPUP_WINDOW | Popup, Overlay | Modern overlay |
| **Buttons** | | |
| EV_BUTTON | Button | Same |
| EV_TOGGLE_BUTTON | ToggleButton, Switch | iOS: Switch |
| EV_CHECK_BUTTON | Checkbox | Universal |
| EV_RADIO_BUTTON | RadioButton | Same |
| **Text Input** | | |
| EV_TEXT_FIELD | TextField, TextInput | Same |
| EV_PASSWORD_FIELD | PasswordField, SecureField | SwiftUI: SecureField |
| EV_TEXT | TextArea, TextEditor | Multi-line |
| EV_COMBO_BOX | Dropdown, Select, Picker | HTML: select, SwiftUI: Picker |
| EV_SPIN_BUTTON | Stepper, NumberInput | SwiftUI: Stepper |
| **Display** | | |
| EV_LABEL | Text, Label | SwiftUI: Text |
| EV_PIXMAP | Image, Icon | Same |
| EV_PROGRESS_BAR | ProgressBar, ProgressView | Same |
| EV_RANGE | Slider | Universal |
| **Lists & Trees** | | |
| EV_LIST | ListView, List | Same |
| EV_TREE | TreeView | Same |
| EV_GRID | DataGrid, Table | Complex data display |
| **Actions** | | |
| select_actions | onClick, onTap | Event handlers |
| change_actions | onChange | Text change |
| close_request_actions | onClose, onDismiss | Window close |
| **Properties** | | |
| set_text | text = , setText | Property setter |
| is_sensitive | isEnabled, enabled | Enable/disable |
| set_minimum_size | minSize, minWidth/minHeight | Size constraints |

### 2.3 Modern Patterns to Adopt

**Fluent/Builder Pattern (React/SwiftUI style):**
```eiffel
-- Modern: Chainable configuration
window := sv.window ("My App")
    .size (800, 600)
    .centered
    .content (
        sv.column
            .spacing (10)
            .padding (20)
            .children (<<
                sv.text ("Hello World").font_size (24),
                sv.button ("Click Me").on_click (agent handle_click),
                sv.text_field.placeholder ("Enter name...").bind (name_model)
            >>)
    )
    .show
```

**Declarative Style:**
```eiffel
-- Describe what you want, not how to build it
create app.make ("My App", agent build_ui)

build_ui: SV_VIEW
    do
        Result := sv.column (<<
            sv.text ("Welcome"),
            sv.button ("Start").primary,
            sv.spacer
        >>)
    end
```

---

## Part 3: simple_vision Library Architecture

### 3.1 Naming Convention

**Prefix:** `SV_` (Simple Vision)

```
SV_* classes wrap EV_* classes
  |
  +-- SV_APPLICATION wraps EV_APPLICATION
  +-- SV_WINDOW wraps EV_TITLED_WINDOW
  +-- SV_BUTTON wraps EV_BUTTON
  +-- SV_ROW wraps EV_HORIZONTAL_BOX (modern alias)
  +-- SV_COLUMN wraps EV_VERTICAL_BOX (modern alias)
```

### 3.2 Core Classes

#### Application Layer
```
SV_APPLICATION
  |-- Wraps EV_APPLICATION
  |-- Simplified: make(title, agent), run
  |-- Auto window management
```

#### Window Layer
```
SV_WINDOW
  |-- Wraps EV_TITLED_WINDOW
  |-- Fluent: .title(), .size(), .centered(), .content()
  |-- Events: .on_close(), .on_resize()

SV_DIALOG
  |-- Wraps EV_DIALOG
  |-- .modal(), .show_and_wait()
```

#### Layout Layer (Dual Naming)
```
SV_ROW / SV_HBOX (aliases)
  |-- Wraps EV_HORIZONTAL_BOX
  |-- .spacing(), .padding(), .children()
  |-- .align_left(), .align_center(), .align_right()

SV_COLUMN / SV_VBOX (aliases)
  |-- Wraps EV_VERTICAL_BOX
  |-- Same API as SV_ROW

SV_GRID
  |-- Wraps EV_TABLE
  |-- .columns(), .rows(), .gap()
  |-- .add_at(widget, row, col)

SV_STACK
  |-- Wraps EV_FIXED
  |-- Absolute positioning
  |-- .add_positioned(widget, x, y)

SV_CARD / SV_PANEL
  |-- Wraps EV_FRAME
  |-- .title(), .elevated(), .bordered()

SV_TABS
  |-- Wraps EV_NOTEBOOK
  |-- .add_tab(title, content)

SV_SCROLL
  |-- Wraps EV_SCROLLABLE_AREA
  |-- .content(), .horizontal(), .vertical()

SV_SPLIT
  |-- Wraps EV_SPLIT_AREA
  |-- .horizontal(), .vertical(), .ratio()
```

#### Widget Layer
```
SV_BUTTON
  |-- Wraps EV_BUTTON
  |-- .text(), .icon(), .on_click()
  |-- .primary(), .secondary(), .danger() (styles)
  |-- .disabled()

SV_CHECKBOX
  |-- Wraps EV_CHECK_BUTTON
  |-- .checked(), .on_change()

SV_RADIO / SV_RADIO_GROUP
  |-- Wraps EV_RADIO_BUTTON
  |-- .options(), .selected(), .on_change()

SV_TEXT / SV_LABEL
  |-- Wraps EV_LABEL
  |-- .text(), .font_size(), .bold(), .color()

SV_TEXT_FIELD / SV_INPUT
  |-- Wraps EV_TEXT_FIELD
  |-- .placeholder(), .value(), .on_change()
  |-- .bind(model) for two-way binding

SV_PASSWORD_FIELD / SV_SECURE_INPUT
  |-- Wraps EV_PASSWORD_FIELD
  |-- Same API as SV_TEXT_FIELD

SV_TEXT_AREA / SV_TEXT_EDITOR
  |-- Wraps EV_TEXT
  |-- Multi-line support

SV_DROPDOWN / SV_SELECT / SV_PICKER
  |-- Wraps EV_COMBO_BOX
  |-- .options(), .selected(), .on_change()

SV_SLIDER
  |-- Wraps EV_RANGE
  |-- .min(), .max(), .value(), .on_change()

SV_PROGRESS
  |-- Wraps EV_PROGRESS_BAR
  |-- .value(), .indeterminate()

SV_STEPPER / SV_NUMBER_INPUT
  |-- Wraps EV_SPIN_BUTTON
  |-- .min(), .max(), .step(), .value()

SV_LIST
  |-- Wraps EV_LIST
  |-- .items(), .selected(), .on_select()
  |-- .multi_select()

SV_TREE
  |-- Wraps EV_TREE
  |-- .nodes(), .on_expand(), .on_select()

SV_DATA_GRID / SV_TABLE
  |-- Wraps EV_GRID
  |-- .columns(), .rows(), .on_cell_click()

SV_IMAGE
  |-- Wraps EV_PIXMAP
  |-- .from_file(), .from_url(), .size()

SV_SEPARATOR / SV_DIVIDER
  |-- Wraps EV_SEPARATOR
  |-- .horizontal(), .vertical()

SV_SPACER
  |-- Flexible space widget
  |-- .fixed(size), .flexible()
```

#### Dialog Layer
```
SV_FILE_PICKER
  |-- Wraps EV_FILE_OPEN_DIALOG / EV_FILE_SAVE_DIALOG
  |-- .open(), .save(), .filter(), .result()

SV_COLOR_PICKER
  |-- Wraps EV_COLOR_DIALOG
  |-- .show(), .result()

SV_FONT_PICKER
  |-- Wraps EV_FONT_DIALOG
  |-- .show(), .result()

SV_ALERT / SV_MESSAGE
  |-- Wraps EV_MESSAGE_DIALOG
  |-- .info(), .warning(), .error()
  |-- .title(), .message(), .show()

SV_CONFIRM
  |-- Wraps EV_CONFIRMATION_DIALOG
  |-- .title(), .message(), .on_yes(), .on_no()
```

### 3.3 Class Hierarchy

```
SV_ANY (base)
  |
  +-- SV_WIDGET (deferred)
  |     |
  |     +-- SV_CONTAINER (deferred)
  |     |     +-- SV_WINDOW
  |     |     +-- SV_DIALOG
  |     |     +-- SV_ROW / SV_HBOX
  |     |     +-- SV_COLUMN / SV_VBOX
  |     |     +-- SV_GRID
  |     |     +-- SV_STACK
  |     |     +-- SV_CARD / SV_PANEL
  |     |     +-- SV_TABS
  |     |     +-- SV_SCROLL
  |     |     +-- SV_SPLIT
  |     |
  |     +-- SV_CONTROL (deferred)
  |           +-- SV_BUTTON
  |           +-- SV_CHECKBOX
  |           +-- SV_RADIO
  |           +-- SV_TEXT / SV_LABEL
  |           +-- SV_TEXT_FIELD / SV_INPUT
  |           +-- SV_TEXT_AREA
  |           +-- SV_DROPDOWN / SV_SELECT
  |           +-- SV_SLIDER
  |           +-- SV_PROGRESS
  |           +-- SV_STEPPER
  |           +-- SV_LIST
  |           +-- SV_TREE
  |           +-- SV_DATA_GRID
  |           +-- SV_IMAGE
  |           +-- SV_SEPARATOR
  |           +-- SV_SPACER
  |
  +-- SV_QUICK (factory/builder)
  +-- SV_STYLE (styling)
  +-- SV_THEME (theming)
```

---

## Part 4: SV_QUICK Fluent API Design

### 4.1 The SV_QUICK Factory

`SV_QUICK` is the centerpiece - a factory class that provides fluent construction of all widgets.

```eiffel
class SV_QUICK

feature -- Factory (short names for rapid coding)

    window (a_title: STRING): SV_WINDOW_BUILDER
    dialog (a_title: STRING): SV_DIALOG_BUILDER

    -- Layout (modern names)
    row: SV_ROW_BUILDER
    column: SV_COLUMN_BUILDER
    grid: SV_GRID_BUILDER
    stack: SV_STACK_BUILDER
    card: SV_CARD_BUILDER
    tabs: SV_TABS_BUILDER
    scroll: SV_SCROLL_BUILDER
    split: SV_SPLIT_BUILDER

    -- Layout (classic aliases)
    hbox: SV_ROW_BUILDER do Result := row end
    vbox: SV_COLUMN_BUILDER do Result := column end

    -- Widgets
    button (a_text: STRING): SV_BUTTON_BUILDER
    checkbox (a_text: STRING): SV_CHECKBOX_BUILDER
    radio (a_text: STRING): SV_RADIO_BUILDER
    text (a_content: STRING): SV_TEXT_BUILDER
    label (a_content: STRING): SV_TEXT_BUILDER do Result := text (a_content) end
    input: SV_INPUT_BUILDER
    text_field: SV_INPUT_BUILDER do Result := input end
    password: SV_PASSWORD_BUILDER
    text_area: SV_TEXT_AREA_BUILDER
    dropdown (a_options: ARRAY [STRING]): SV_DROPDOWN_BUILDER
    select (a_options: ARRAY [STRING]): SV_DROPDOWN_BUILDER do Result := dropdown (a_options) end
    picker (a_options: ARRAY [STRING]): SV_DROPDOWN_BUILDER do Result := dropdown (a_options) end
    slider: SV_SLIDER_BUILDER
    progress: SV_PROGRESS_BUILDER
    stepper: SV_STEPPER_BUILDER
    list: SV_LIST_BUILDER
    tree: SV_TREE_BUILDER
    data_grid: SV_DATA_GRID_BUILDER
    image (a_path: STRING): SV_IMAGE_BUILDER
    separator: SV_SEPARATOR_BUILDER
    divider: SV_SEPARATOR_BUILDER do Result := separator end
    spacer: SV_SPACER_BUILDER

    -- Dialogs
    alert (a_message: STRING): SV_ALERT_BUILDER
    confirm (a_message: STRING): SV_CONFIRM_BUILDER
    file_open: SV_FILE_PICKER_BUILDER
    file_save: SV_FILE_PICKER_BUILDER
    color_picker: SV_COLOR_PICKER_BUILDER
    font_picker: SV_FONT_PICKER_BUILDER

feature -- Convenience

    row_of (a_widgets: ARRAY [SV_WIDGET]): SV_ROW
    column_of (a_widgets: ARRAY [SV_WIDGET]): SV_COLUMN

end
```

### 4.2 Builder Pattern Classes

Each widget has a builder for fluent configuration:

```eiffel
class SV_BUTTON_BUILDER

feature -- Configuration

    text (a_text: STRING): like Current
    icon (a_icon: SV_IMAGE): like Current
    on_click (a_action: PROCEDURE): like Current

    -- Styles
    primary: like Current
    secondary: like Current
    danger: like Current
    outlined: like Current
    text_only: like Current

    -- States
    disabled: like Current
    enabled: like Current

    -- Size
    small: like Current
    medium: like Current
    large: like Current

    -- Layout
    width (a_width: INTEGER): like Current
    height (a_height: INTEGER): like Current
    min_width (a_width: INTEGER): like Current

feature -- Build

    build: SV_BUTTON

feature -- Conversion

    as_widget: SV_WIDGET
        do Result := build end

end
```

### 4.3 Usage Examples

#### Example 1: Simple Window
```eiffel
local
    sv: SV_QUICK
    app: SV_APPLICATION
do
    create sv
    create app.make ("My App")

    sv.window ("Hello World")
        .size (400, 300)
        .centered
        .content (
            sv.column
                .padding (20)
                .spacing (10)
                .children (<<
                    sv.text ("Welcome to simple_vision!").font_size (24).bold,
                    sv.button ("Click Me").primary.on_click (agent handle_click)
                >>)
        )
        .show

    app.run
end
```

#### Example 2: Form Layout
```eiffel
sv.window ("User Registration")
    .size (500, 400)
    .content (
        sv.card
            .title ("Register")
            .padding (20)
            .content (
                sv.column
                    .spacing (15)
                    .children (<<
                        sv.row.children (<<
                            sv.text ("Name:").width (100),
                            sv.input.placeholder ("Enter name").expand
                        >>),
                        sv.row.children (<<
                            sv.text ("Email:").width (100),
                            sv.input.placeholder ("Enter email").expand
                        >>),
                        sv.row.children (<<
                            sv.text ("Password:").width (100),
                            sv.password.placeholder ("Enter password").expand
                        >>),
                        sv.spacer,
                        sv.row.align_right.children (<<
                            sv.button ("Cancel").secondary,
                            sv.button ("Register").primary
                        >>)
                    >>)
            )
    )
    .show
```

#### Example 3: Data Display
```eiffel
sv.window ("Customer List")
    .size (800, 600)
    .content (
        sv.column
            .children (<<
                sv.row.children (<<
                    sv.input.placeholder ("Search...").expand,
                    sv.button ("Search").primary,
                    sv.button ("Add").icon (add_icon)
                >>),
                sv.data_grid
                    .columns (<<"ID", "Name", "Email", "Status">>)
                    .rows (customer_data)
                    .on_row_click (agent handle_row_click)
                    .expand
            >>)
    )
    .show
```

#### Example 4: Tabs
```eiffel
sv.window ("Settings")
    .size (600, 500)
    .content (
        sv.tabs
            .add_tab ("General", general_settings_view)
            .add_tab ("Display", display_settings_view)
            .add_tab ("Advanced", advanced_settings_view)
    )
    .show
```

---

## Part 5: Implementation Plan

**Note:** This plan follows our proven approach (5K+ LOC/day velocity). Each phase is achievable in 1-2 focused sessions.

### Phase 1: Foundation (Tier 1 Core) ✅ COMPLETE
- ✅ Create simple_vision library skeleton
- ✅ Implement SV_ANY base class
- ✅ Implement SV_QUICK factory (basic version)
- ✅ Implement SV_APPLICATION wrapper
- ✅ Implement SV_WINDOW and SV_WINDOW_BUILDER
- ✅ Implement SV_ROW/SV_COLUMN (SV_HBOX/SV_VBOX)
- ✅ Basic demo: Hello World window

### Phase 2: Core Widgets (Tier 1) ✅ COMPLETE
- ✅ SV_BUTTON with full builder
- ✅ SV_TEXT/SV_LABEL
- ✅ SV_TEXT_FIELD/SV_INPUT
- ✅ SV_PASSWORD_FIELD
- ✅ SV_CHECKBOX
- ✅ SV_RADIO/SV_RADIO_GROUP
- ✅ SV_DROPDOWN/SV_SELECT
- ✅ Demo: Login form (with harness tests + use-case JSONs)

### Phase 3: Advanced Layouts (Tier 1) ✅ COMPLETE
- ✅ SV_GRID
- ✅ SV_STACK
- ✅ SV_CARD/SV_PANEL
- ✅ SV_TABS (SV_TAB_PANEL)
- ✅ SV_SCROLL
- ✅ SV_SPLIT (SV_SPLITTER)
- ✅ SV_SPACER
- ✅ SV_SEPARATOR
- ✅ Demo: Complex layout (with harness tests + use-case JSONs)

### Phase 4: Data Widgets (Tier 1) ✅ COMPLETE
- ✅ SV_LIST
- ✅ SV_TREE
- ✅ SV_DATA_GRID (leveraging EV_GRID)
- ✅ SV_PROGRESS (SV_PROGRESS_BAR)
- ✅ SV_SLIDER
- ✅ SV_STEPPER (SV_SPIN_BOX)
- ✅ SV_MENU_BAR, SV_MENU, SV_MENU_ITEM (bonus)
- ✅ SV_TOOLBAR, SV_TOOLBAR_BUTTON (bonus)
- ✅ SV_STATUSBAR (bonus)
- ✅ Demo: Data browser (with harness tests + use-case JSONs)

### Phase 5: Dialogs & Polish (Tier 1) ✅ COMPLETE
- ✅ SV_DIALOG
- ✅ SV_ALERT/SV_MESSAGE (SV_MESSAGE_BOX)
- ✅ SV_CONFIRM (in SV_MESSAGE_BOX)
- ✅ SV_FILE_PICKER (SV_FILE_DIALOG)
- ✅ SV_COLOR_PICKER
- ✅ SV_FONT_PICKER
- ✅ SV_IMAGE
- ✅ Demo: Complete application (dialogs covered by widget tests)

### Phase 6: Styling & Theming (Tier 1)
- ✅ SV_COLOR class (hex parsing, lighten/darken, conversion)
- ✅ SV_TOKENS class (semantic design tokens: colors, typography, spacing, borders)
- ✅ SV_THEME class (singleton, theme switching, preference persistence)
- ✅ Theme modes: Light, Dark, System (follows OS preference)
- ✅ Color schemes: Material Purple (default), Blue, Green, Orange, Red, Teal
- ✅ UI scaling (browser-style zoom 50%-300%, persisted to config)
- ✅ Font scaling (separate from UI scale, for accessibility)
- ✅ Preference file persistence (~/.simple_vision_prefs.json)
- ✅ Add theme methods to SV_QUICK factory
- ✅ Update SV_WIDGET base with apply_theme and subscribe_to_theme
- ✅ Wire theme to SV_TEXT, SV_BUTTON, SV_WINDOW, SV_CARD, SV_ROW, SV_COLUMN
- ✅ Wire theme to SV_TEXT_FIELD, SV_CHECKBOX, SV_DROPDOWN, SV_RADIO_GROUP
- ✅ Wire theme to SV_TAB_PANEL, SV_SPLITTER, SV_DATA_GRID
- ✅ Demo: Themed application (theme switcher, scale controls)
- ⬜ Tests for styling/theming
- ⬜ Use-case JSON for theme demo

### Phase 6.5: Theme Polish & Grid Enhancements ✅ COMPLETE
**Scale Application (Critical)**
- ✅ Font scale: Apply theme.font_scale to all text widgets on creation and theme change
- ✅ UI scale: Apply theme.ui_scale to widget sizes, spacing, padding
- ✅ Replace `divider` with themed colored box (SV_DIVIDER uses EV_CELL, not EV_SEPARATOR)

**Grid Enhancements**
- ✅ SV_DATA_GRID column headers with clickable sorting (Asc/Desc/Natural cycle, [ASC]/[DESC] indicators)
- ✅ SV_DATA_GRID row striping/zebra striping (theme-controlled surface_variant for alternate rows)

**Input Enhancements**
- ✅ Placeholder/explainer text for SV_TEXT_FIELD (focus in/out handlers, text_hint color)
- ✅ SV_DECIMAL_FIELD with simple_decimal integration (precise currency/numeric input)

**Demo: demo_grid.exe** - Shows sorting, striping, theme toggle, dynamic row addition

**Known Windows Native Control Limitations (Accepted & Documented)**
These are fundamental Windows behaviors that would require Tier 2/3 to resolve:
- Dropdown popup list: White background (Windows-drawn, ignores custom colors)
- EV_SEPARATOR: Gray system-styled line (workaround: use SV_DIVIDER instead)
- Some button/checkbox styles: May not fully respect custom colors on all Windows versions

### Phase 6.75: Field Masking & GUI Testing (COMPLETE)
**Field Masking with simple_regex** ✅ COMPLETE
- ✅ SV_MASKED_FIELD: Text field with regex-based input validation
- ✅ Common masks: phone, SSN, date, email, ZIP, credit card, IP, time (18+ pre-built masks)
- ✅ Real-time validation feedback (background color change on invalid input)
- ✅ Factory methods in SV_QUICK: masked_field, phone_field, ssn_field, email_field, etc.
- ✅ Demo: demo_masked.exe (7th demo in bin/)
- ✅ **TRUE Input Masking** (character constraint + auto-insert literals):
  - Input mask templates: `#` = digit, `A` = letter, `*` = alphanumeric, others = literal
  - Key press interception filters invalid characters at input time
  - Auto-inserts literal characters (parens, dashes, slashes, spaces)
  - Masks: `(###) ###-####` (phone), `###-##-####` (SSN), `##/##/####` (date), etc.
  - Validation-only fields (email, IP) retain flexible format with regex validation
- ⬜ Apply masking capability to SV_TEXT_FIELD, SV_DECIMAL_FIELD (optional - SV_MASKED_FIELD covers use cases)

**GUI Test Harness** (Deferred)
- ⬜ Kamikaze agents for GUI simulation (short-lived agents that simulate user input)
- ⬜ Screen pathway coverage (state machine model for exhaustive UI path testing)

**Form Validation System** ✅ COMPLETE
- ✅ SV_FORM: Form container with field management and validation
- ✅ SV_FIELD: Individual field with fluent validation rules
- ✅ Validation rules: SV_REQUIRED_RULE, SV_MIN_LENGTH_RULE, SV_MAX_LENGTH_RULE, SV_PATTERN_RULE, SV_EMAIL_RULE, SV_RANGE_RULE
- ✅ Auto-build form UI from field definitions
- ✅ Demo: demo_form.exe (8th demo in bin/)

**Advanced Features** (Deferred to Phase 6.9)
- ⬜ SV_OBSERVABLE / SV_BINDING: Reactive data binding for widgets
- ⬜ SV_STATE_MACHINE: UI state management
- ⬜ SV_CONSTRAINTS: Constraint-based layout
- ⬜ SV_NAVIGATOR: Page routing/navigation stack
- ⬜ SV_AI_BUILDER: Generate UI from natural language prompts

**Extras from Innovations** (Deferred)
- ⬜ Accessibility (a11y): Screen reader support, keyboard navigation
- ⬜ Undo/Redo Stack: Widget state history
- ⬜ Tooltips: Rich content tooltips
- ⬜ Virtual Lists: Lazy loading for large datasets
- ⬜ Hot Keys: Global keyboard shortcuts

### Phase 7: C Library Integration (Tier 2)
- [ ] Create simple_cairo library (Cairo wrapper)
- [ ] Create simple_stb library (stb_image wrapper)
- [ ] SV_CAIRO_CANVAS widget
- [ ] Gradient support in SV_GRAPHICS
- [ ] Shadow support in SV_GRAPHICS
- [ ] Demo: Modern styled cards with gradients/shadows

### Phase 8: Web Panel Integration (Tier 3)
- [ ] Create simple_webview library (webview wrapper)
- [ ] SV_WEB_VIEW widget with Eiffel ↔ JS bridge
- [ ] Integration with simple_htmx for HTML generation
- [ ] Integration with simple_alpine for reactive JS
- [ ] SV_RICH_EDITOR (web-based)
- [ ] SV_CHART (Chart.js via webview)
- [ ] SV_CODE_EDITOR (Monaco via webview)
- [ ] Demo: Hybrid native + web application

### Phase 9: Documentation & Testing
- [ ] Unit tests for all widgets
- [ ] Integration tests
- [ ] docs/index.html
- [ ] docs/cookbook.html
- [ ] README.md
- [ ] Example applications

---

## Deferred / Circle-Back Items

**Purpose:** Track items we've identified but intentionally deferred for later implementation. These are not forgotten - just prioritized for a future pass.

### Theming Integration (from Phase 6)
- [ ] **SV_WIDGET theme token integration** - Update all widget classes to use SV_TOKENS for colors, fonts, spacing instead of hardcoded values. This is a significant refactor affecting 20+ widget classes.
  - Priority: Medium (current widgets work, just not fully themed)
  - Effort: 1 session
  - Dependency: Stable SV_TOKENS API

### GUI Test Harness (from Phase 9)
- [ ] **Kamikaze agents for GUI simulation** - Short-lived agents that simulate user input
  - Priority: Low (manual testing works for now)
  - Effort: 1-2 sessions
  - Dependency: simple_process agent spawning

- [ ] **Screen pathway coverage** - State machine model for exhaustive UI path testing
  - Priority: Low (use-case JSONs cover happy paths)
  - Effort: 1 session

### Advanced Features
- [ ] **SV_OBSERVABLE / SV_BINDING** - Reactive data binding for widgets
  - Priority: Medium (enhances developer experience significantly)
  - Effort: 0.5 session

- [ ] **SV_FORM / SV_FIELD** - Form validation system
  - Priority: Medium
  - Effort: 1 session

- [ ] **SV_STATE_MACHINE** - UI state management
  - Priority: Low
  - Effort: 0.5 session

- [ ] **SV_CONSTRAINTS** - Constraint-based layout
  - Priority: Low (current layouts work well)
  - Effort: 0.5 session

### Platform-Specific
- [ ] **System dark mode detection** - Currently uses environment variable; could query Windows API
  - Priority: Low (manual toggle works)
  - Effort: 0.25 session

---

## Part 6: File Structure

```
simple_vision/
├── simple_vision.ecf
├── README.md
├── CHANGELOG.md
├── docs/
│   ├── index.html
│   ├── cookbook.html
│   └── css/style.css
├── src/
│   ├── core/
│   │   ├── sv_any.e
│   │   ├── sv_application.e
│   │   └── sv_quick.e
│   ├── widgets/                    -- Tier 1: EV2-based widgets
│   │   ├── sv_widget.e
│   │   ├── sv_control.e
│   │   ├── sv_button.e
│   │   ├── sv_checkbox.e
│   │   ├── sv_radio.e
│   │   ├── sv_text.e
│   │   ├── sv_text_field.e
│   │   ├── sv_text_area.e
│   │   ├── sv_dropdown.e
│   │   ├── sv_slider.e
│   │   ├── sv_progress.e
│   │   ├── sv_stepper.e
│   │   ├── sv_list.e
│   │   ├── sv_tree.e
│   │   ├── sv_data_grid.e
│   │   ├── sv_image.e
│   │   ├── sv_separator.e
│   │   └── sv_spacer.e
│   ├── containers/
│   │   ├── sv_container.e
│   │   ├── sv_window.e
│   │   ├── sv_dialog.e
│   │   ├── sv_row.e
│   │   ├── sv_column.e
│   │   ├── sv_grid.e
│   │   ├── sv_stack.e
│   │   ├── sv_card.e
│   │   ├── sv_tabs.e
│   │   ├── sv_scroll.e
│   │   └── sv_split.e
│   ├── dialogs/
│   │   ├── sv_alert.e
│   │   ├── sv_confirm.e
│   │   ├── sv_file_picker.e
│   │   ├── sv_color_picker.e
│   │   └── sv_font_picker.e
│   ├── builders/
│   │   ├── sv_window_builder.e
│   │   ├── sv_button_builder.e
│   │   ├── sv_row_builder.e
│   │   ├── sv_column_builder.e
│   │   └── ... (one per widget)
│   ├── styling/
│   │   ├── sv_style.e
│   │   └── sv_theme.e
│   ├── graphics/                   -- Tier 2: C-enhanced graphics
│   │   ├── sv_graphics.e           -- Unified API
│   │   ├── sv_cairo_canvas.e       -- Cairo rendering widget
│   │   ├── sv_gradient.e
│   │   └── sv_shadow.e
│   └── web/                        -- Tier 3: Web panel integration
│       ├── sv_web_view.e           -- Webview widget
│       ├── sv_web_bridge.e         -- Eiffel ↔ JS communication
│       ├── sv_rich_editor.e        -- TinyMCE/Quill wrapper
│       ├── sv_chart.e              -- Chart.js wrapper
│       └── sv_code_editor.e        -- Monaco wrapper
├── testing/
│   ├── lib_tests.e
│   └── test_set_base.e
└── examples/
    ├── hello_world/
    ├── login_form/
    ├── data_browser/
    ├── settings_panel/
    ├── hybrid_dashboard/           -- Tier 1 + Tier 3 example
    └── styled_cards/               -- Tier 2 Cairo example
```

### Related Libraries (Dependencies)

```
D:\prod\
├── simple_vision/      -- Main library (this plan)
├── simple_cairo/       -- Cairo wrapper (Tier 2) [TO BE CREATED]
├── simple_stb/         -- stb_image wrapper (Tier 2) [TO BE CREATED]
├── simple_webview/     -- webview wrapper (Tier 3) [TO BE CREATED]
├── simple_htmx/        -- HTMX HTML generation [EXISTS]
└── simple_alpine/      -- Alpine.js integration [EXISTS]
```

---

## Part 7: Design Principles

### 7.1 Simplicity First
- One-liner for common operations
- Sensible defaults everywhere
- Progressive disclosure of complexity

### 7.2 Fluent API
- All methods return Current for chaining
- Builder pattern for complex configuration
- No boilerplate required

### 7.3 Dual Naming
- Modern names (row, column, dropdown) for new developers
- Classic aliases (hbox, vbox, combo_box) for EV2 veterans
- Both resolve to same implementation

### 7.4 Consistent Patterns
- `.on_*()` for all event handlers
- `.children()` for container content
- `.content()` for single-child containers
- `.expand` for flexible sizing

### 7.5 DBC Throughout
- Preconditions on all inputs
- Postconditions on state changes
- Class invariants on all widgets

---

## Appendix A: EV2 Quick Reference (For Implementation)

### Creating Widgets
```eiffel
create button.make_with_text ("Click")
create box
box.extend (button)
```

### Event Handling
```eiffel
button.select_actions.extend (agent handler)
window.close_request_actions.extend (agent on_close)
```

### Layout
```eiffel
box.extend (widget)
box.disable_item_expand (widget)  -- Don't grow
box.set_padding (10)
box.set_border_width (5)
```

### Sizing
```eiffel
widget.set_minimum_size (100, 50)
window.set_size (800, 600)
```

### Visibility
```eiffel
widget.show
widget.hide
window.show_modal_to_window (parent)
```

---

## Appendix B: Modern GUI References

- [React Component Naming](https://www.sufle.io/blog/naming-conventions-in-react)
- [SwiftUI Cheat Sheet](https://fuckingswiftui.com/)
- [Flutter Layouts](https://docs.flutter.dev/ui/layout)
- [Material Design Components](https://m3.material.io/components)
- [Smashing Magazine Naming Best Practices](https://www.smashingmagazine.com/2024/05/naming-best-practices/)
- [Design System Naming](https://www.uxpin.com/studio/blog/design-system-naming-conventions/)

---

*simple_vision — Modern GUI development for Eiffel, simplified.*
