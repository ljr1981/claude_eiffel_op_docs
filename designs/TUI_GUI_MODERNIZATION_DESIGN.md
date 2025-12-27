# TUI & GUI Modernization Design Document

**Created**: 2025-12-22
**Status**: Design Discussion
**Author**: Larry + Claude (Eiffel Expert)

---

## Executive Summary

This document explores modernizing Eiffel's GUI capabilities beyond the dated Eiffel Vision 2 library, and creating a new simple_tui library for console-based text user interfaces.

**Key Goals**:
1. Create `simple_tui` - text-based UI components for console applications
2. Explore modern GPU-accelerated GUI alternatives to Vision 2
3. Leverage 2025 technology (GPU acceleration, HiDPI, modern UX patterns)

---

## PART 1: simple_tui (Text User Interface)

### The Vision

Console-based widgets (text boxes, buttons, dropdowns, menus, tables, progress bars) that work cross-platform on Windows, Linux, and Mac terminals.

### Existing Tech to Study

| Library | Language | Platform | Notes |
|---------|----------|----------|-------|
| **ncurses** | C | Unix/Linux/Mac | The gold standard for TUI. Mature, well-documented |
| **PDCurses** | C | Windows/DOS | ncurses port for Windows |
| **notcurses** | C | Modern Unix | ncurses successor with sixel graphics, 24-bit color |
| **FTXUI** | C++ | Cross-platform | Modern, functional, header-only |
| **termbox2** | C | Cross-platform | Minimal, clean C API (~1500 lines) |
| **crossterm** | Rust | Cross-platform | Shows what's achievable |

### Recommended Architecture for simple_tui

```
Layer 1: simple_console (already exists) - low-level ANSI/Win32 console API
Layer 2: simple_tui_core - primitives: Box, Cursor, Color, Event loop
Layer 3: simple_tui_widgets - Text field, Button, Dropdown, List, Table, Progress
Layer 4: simple_tui_layout - Grid, Flex, Padding, Borders
```

### Key Design Decisions

1. **Pure Eiffel with inline C** for terminal handling (Win32 Console API / ANSI escapes)
2. **Event-driven architecture** matching Eiffel Vision 2's pattern (for familiarity)
3. **Component-based** with DBC contracts on all widgets
4. **Double-buffering** for flicker-free updates
5. **Unicode support** from day one

### Complexity Assessment

Medium. We already have simple_console. The challenge is building a proper widget hierarchy with layout management.

### Example API Vision

```eiffel
class MY_TUI_APP

inherit
    TUI_APPLICATION

feature -- UI

    build_ui
        local
            main_box: TUI_VBOX
            input: TUI_TEXT_FIELD
            btn: TUI_BUTTON
        do
            create main_box.make

            create input.make_with_label ("Name:")
            input.set_width (30)
            main_box.add (input)

            create btn.make_with_text ("Submit")
            btn.on_click (agent handle_submit)
            main_box.add (btn)

            set_root (main_box)
        end

    handle_submit
        do
            -- Handle button click
        end

end
```

---

## PART 2: Why Vision 2 is Dated

### Technical Limitations

- Based on GTK2 (released 2002) and Win32 GDI (released 1995)
- No GPU acceleration - all software rendering
- No modern animations/effects/transitions
- No HiDPI/4K awareness designed in from the start
- No dark mode support
- Pre-touch, pre-gestures, pre-modern-UX paradigms
- Component architecture from pre-2000s design patterns

### The Vision 2 Architecture

Vision 2 uses a "pick and drop" model with platform-specific implementations:
- Windows: Wraps WEL (Windows Eiffel Library) which wraps Win32
- Linux/Mac: Wraps GTK2

This was innovative in the late 1990s but hasn't kept pace with modern GUI development.

---

## PART 3: Modern GUI Landscape (2025)

### GPU-Accelerated Rendering Libraries

| Library | Backing | GPU Support | C API | Status |
|---------|---------|-------------|-------|--------|
| **Skia** | Google | Direct3D, Vulkan, Metal, OpenGL | Partial (via C wrapper) | Production (Chrome, Flutter, Android) |
| **bgfx** | Independent | All backends | Yes | Mature, active |
| **sokol** | Independent | All backends | Yes, minimal | Active, clean |
| **NanoVG** | Community | OpenGL/Vulkan | Yes | Stable, focused on vector graphics |

### Full Widget Toolkits

| Toolkit | Language | GPU | Cross-Platform | Eiffel Friendliness |
|---------|----------|-----|----------------|---------------------|
| **Dear ImGui** | C++ | Yes | Yes | Good - simple C-like API, immediate mode |
| **Nuklear** | C | Yes | Yes | Excellent - pure C, single-header |
| **GTK4** | C | Partial | Yes | Good - proper C API |
| **SDL2 + UI** | C | Yes | Yes | Good - we already use SDL patterns |
| **raylib** | C | Yes | Yes | Good - game-focused but capable |
| **lvgl** | C | Yes (embedded focus) | Yes | Good - designed for embedded but works on desktop |

### Platform-Specific Modern APIs

**Windows (WEL 2025 candidates)**:
- **Direct2D/DirectWrite** - GPU-accelerated 2D, current Microsoft recommendation
- **WinUI 3** - Modern Windows 11 look, but C++/WinRT heavy
- **Win2D** - UWP canvas, GPU accelerated

**Linux (LEL candidates)**:
- **GTK4** - Modern, GPU-accelerated, proper C API
- **Wayland protocols** - For truly modern Linux graphics

**Mac (MEL candidates)**:
- **Core Graphics/Core Animation** - GPU accelerated, Objective-C
- **SwiftUI** - Modern but Swift-only
- **Metal** - For raw GPU access

---

## PART 4: Recommended Implementation Paths

### Option A: Immediate Mode GUI (simple_imgui)

**Wrap Dear ImGui** (or Nuklear):

```eiffel
class SIMPLE_IMGUI_WINDOW

feature -- Widgets

    begin_window (title: STRING; flags: INTEGER)
    end_window

    button (label: STRING): BOOLEAN
        -- Returns True if clicked

    text_input (label: STRING; buffer: STRING): BOOLEAN
        -- Returns True if changed

    slider_float (label: STRING; value: REAL_REF; min, max: REAL): BOOLEAN

    combo_box (label: STRING; items: ARRAY [STRING]; selected: INTEGER_REF): BOOLEAN

end
```

**Pros**:
- Immediate mode is simple to wrap
- GPU accelerated (OpenGL/DirectX/Vulkan backends)
- Single C++ header, easy to compile
- Incredible community, tons of widgets
- Very fast development cycle

**Cons**:
- Doesn't look "native" (though themes exist)
- Retained state requires careful design in Eiffel
- Game-oriented aesthetic by default

### Option B: Vector Graphics Foundation (simple_canvas)

**Wrap NanoVG or Skia**:

```eiffel
class SIMPLE_CANVAS

feature -- Path Drawing

    begin_path
    move_to (x, y: REAL)
    line_to (x, y: REAL)
    bezier_to (c1x, c1y, c2x, c2y, x, y: REAL)
    arc (cx, cy, r, a0, a1: REAL; dir: INTEGER)
    close_path
    fill
    stroke

feature -- Text & Images

    draw_text (x, y: REAL; text: STRING)
    draw_image (x, y, w, h: REAL; image: SIMPLE_IMAGE)

feature -- State

    save
    restore
    translate (x, y: REAL)
    rotate (angle: REAL)
    scale (x, y: REAL)

end
```

Then build widgets on top. This is how Flutter works.

**Pros**:
- Maximum control over appearance
- Can match any platform aesthetic
- GPU accelerated
- Foundation for future mobile port

**Cons**:
- Significant work to build widget library
- Need to handle layout, hit testing, accessibility ourselves

### Option C: Platform-Specific with Shared Interface

**Create an abstract GUI interface, implement per-platform**:

```
SIMPLE_GUI_INTERFACE (deferred)
    ├── SIMPLE_GUI_WIN32 (Direct2D)
    ├── SIMPLE_GUI_LINUX (GTK4)
    └── SIMPLE_GUI_MAC (Core Graphics)
```

**Pros**:
- Native look on each platform
- Best performance per-platform
- Can leverage platform-specific features

**Cons**:
- 3x the implementation work
- Consistency challenges across platforms
- Feature parity difficult to maintain

---

## PART 5: Recommended Path Forward

### Two-Track Approach

#### Track 1: simple_tui (Near-term, 2-4 weeks)

- Pure Eiffel with inline C
- No external dependencies beyond simple_console
- Immediately useful for CLI tools
- Establishes patterns for widget architecture
- Good learning ground for event-driven UI patterns

#### Track 2: simple_gui_core (Medium-term)

- Start with **sokol_gfx** or **bgfx** for GPU abstraction
- Add **NanoVG** for 2D vector graphics
- Build widget library on top
- Single codebase, multiple GPU backends (OpenGL, D3D, Vulkan, Metal)

### Proposed Architecture

```
simple_gui_core
    ├── simple_gpu        -- sokol or bgfx wrapper (GPU abstraction)
    ├── simple_canvas     -- NanoVG wrapper for 2D vector graphics
    ├── simple_font       -- Font loading/rendering (stb_truetype)
    ├── simple_input      -- Unified input handling
    └── simple_gui_widgets
        ├── GUI_WIDGET (deferred base)
        ├── GUI_BUTTON
        ├── GUI_TEXT_FIELD
        ├── GUI_DROPDOWN
        ├── GUI_TABLE
        ├── GUI_SCROLL_VIEW
        └── GUI_LAYOUT_MANAGER
```

### Why sokol + NanoVG

1. Both are single-file C headers (easy to include)
2. Both have clean C APIs (easy to wrap with Eiffel inline-C)
3. Both are actively maintained
4. sokol handles all GPU backends transparently
5. NanoVG gives us high-quality 2D rendering
6. Together they're under 50K lines of C
7. No build system complexity

---

## PART 6: Open Design Questions

### 1. Native Look vs Custom Look

Do we want widgets that look like Windows/GTK native controls, or are we okay with a custom "Simple" aesthetic (like VS Code, Electron apps, game UIs)?

**Native**: Harder to implement, platform-specific code, but familiar to users
**Custom**: Consistent across platforms, full control, but different from OS

### 2. Immediate vs Retained Mode

**Immediate Mode** (ImGui style):
- UI is rebuilt every frame
- No widget state management
- Easier to wrap, different paradigm
- Used by: Dear ImGui, Nuklear, raylib

**Retained Mode** (Vision 2 style):
- Widget objects persist
- State is managed
- More complex but familiar to Eiffel developers
- Used by: GTK, Qt, WinForms

### 3. Priority

- **TUI first**: Useful for tooling (simple_notebook, simple_pkg CLI), lower risk
- **GUI first**: Bigger impact, more complex, higher reward

### 4. Scope for v1

- **Minimal**: Button, Label, TextInput, List (proof of concept)
- **Moderate**: Above + Dropdown, Checkbox, Radio, Table, Tabs
- **Comprehensive**: Full widget set rivaling Vision 2

---

## PART 7: Research Resources

### Libraries to Evaluate

1. **sokol** - https://github.com/floooh/sokol
   - Single-file cross-platform libs
   - sokol_gfx.h, sokol_app.h, sokol_audio.h

2. **NanoVG** - https://github.com/memononen/nanovg
   - Antialiased 2D vector drawing
   - OpenGL/Vulkan backends

3. **Dear ImGui** - https://github.com/ocornut/imgui
   - Immediate mode GUI
   - Massive widget library

4. **Nuklear** - https://github.com/Immediate-Mode-UI/Nuklear
   - Single-header ANSI C GUI
   - Similar to ImGui but C

5. **bgfx** - https://github.com/bkaradzic/bgfx
   - Cross-platform rendering
   - More comprehensive than sokol

### Eiffel Vision 2 Study

- Location: `$ISE_LIBRARY/library/vision2`
- Study the EV_WIDGET hierarchy
- Understand pick-and-drop model
- Learn from the platform abstraction layer

---

## PART 8: Next Steps

1. **Prototype simple_tui** with basic widgets (Box, Label, Button)
2. **Evaluate sokol_app.h** for window creation and input
3. **Evaluate NanoVG** for 2D rendering quality
4. **Design the abstract widget interface** that works for both TUI and GUI
5. **Build proof-of-concept** with 3-4 widgets

---

## Appendix: Comparison with Vision 2

| Aspect | Vision 2 | Proposed Modern |
|--------|----------|-----------------|
| Rendering | Software (GDI/GTK) | GPU (OpenGL/Vulkan/D3D) |
| Resolution | Fixed DPI | HiDPI aware |
| Effects | None | Animations, shadows, blur |
| Themes | Platform-dependent | Consistent custom themes |
| Performance | Moderate | High (GPU accelerated) |
| Mobile Ready | No | Foundation for future |
| Code Size | Large (full EiffelStudio) | Small (embedded C libs) |
| Dependencies | GTK2/Win32 | Self-contained |

---

**Document End**
