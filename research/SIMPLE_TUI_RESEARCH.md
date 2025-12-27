# simple_tui Research Notes

**Date:** 2025-12-27
**Status:** Complete
**Goal:** Design a modern TUI library for Eiffel with maximum innovation

---

## Step 1: Deep Web Research - Existing TUI Libraries

### Modern TUI Landscape (2024-2025)

| Library | Language | Architecture | Key Innovation |
|---------|----------|--------------|----------------|
| **Ratatui** | Rust | Immediate mode | 60fps, double-buffering, minimal diffs |
| **Textual** | Python | CSS-styled | Web-like development, flexbox layouts |
| **Bubble Tea** | Go | Elm (MVU) | Functional, message-based, composable |
| **FTXUI** | C++ | Functional/React-like | Three layers: screen/dom/component |
| **Notcurses** | C | Planes/cells | Sixel graphics, 24-bit color, multimedia |
| **Ink** | JS/React | Virtual DOM | React patterns in terminal |
| **FINAL CUT** | C++ | Widget tree | Qt-like, full mouse support |
| **termbox2** | C | Minimal | ~1500 lines, clean API |
| **Phoenix** | Go | DDD + Elm | 10x perf claims, differential rendering |

### Why ncurses Is Dated

1. **Portability issues**: Behaves differently across terminals
2. **Limited UTF-8/emoji**: Wide characters problematic
3. **Blocking I/O**: Hard to integrate async
4. **No 24-bit color**: Limited to 256 colors
5. **No multimedia**: No sixel/kitty graphics
6. **Manual refresh**: Easy to cause flickering

### Key Takeaways

1. **Elm Architecture dominates**: Model-View-Update pattern proven across Go/Rust/Python
2. **CSS-like styling wins**: Textual, r3bl_tui, reactive_tui all use CSS
3. **Immediate mode rising**: React-inspired rebuild-every-frame
4. **60fps is achievable**: With proper double-buffering and diff rendering
5. **Cross-platform via abstractions**: crossterm, termion, termwiz backends

---

## Step 2: Tech-Stack Research - Terminal Fundamentals

### Terminal Output Protocols

| Protocol | Platform | Features |
|----------|----------|----------|
| **ANSI/VT100** | All modern | Colors, cursor, basic styling |
| **VT220+** | All modern | 256 colors, more escape codes |
| **24-bit RGB** | Modern terminals | True color (16.7M colors) |
| **Sixel** | xterm, mlterm, WezTerm | Bitmap graphics |
| **Kitty Graphics** | Kitty, WezTerm | Superior bitmap protocol |

### Windows Console Evolution

- **Pre-2016**: Console API only (no ANSI)
- **Windows 10 1607+**: ANSI via ENABLE_VIRTUAL_TERMINAL_PROCESSING
- **Windows Terminal (2019+)**: Full VT100/ANSI, 24-bit color

### Input Handling

Event types to handle:
- Keyboard: Regular keys, modifiers (Ctrl/Alt/Shift), function keys
- Mouse: Click, drag, scroll, movement
- Terminal: Resize, focus gain/loss
- Paste: Bracketed paste mode

### Double Buffering

Benefits:
- No flickering
- Minimal I/O (only changed cells written)
- Smooth animation possible

---

## Step 3: Eiffel Ecosystem Research

### Existing Libraries We Can Leverage

| Library | Use For | Notes |
|---------|---------|-------|
| **simple_console** | Low-level terminal I/O | ANSI codes, cursor, Win32 API |
| **simple_file** | Config file loading | .tcss style files |
| **simple_json** | Serialization | Widget state persistence |
| **simple_toml** | Config parsing | Theme files |
| **simple_process** | Subprocesses | Spawning child terminals |
| **simple_datetime** | Timing | Animation frame timing |

---

## Step 4: Developer Pain Points

| Pain Point | Solution Approach |
|------------|-------------------|
| **Flickering** | Double buffering, synchronized output |
| **Layout complexity** | Flexbox-like system |
| **Event handling** | Elm-style message passing |
| **Cross-platform** | ANSI + Win32 abstraction |
| **Unicode width** | wcwidth algorithm |
| **State management** | Immutable models, contracts |
| **Testing** | Mock terminal, headless mode |
| **Debugging** | Logging to file |

---

## Step 5: Innovation Hat - Unique Value Propositions

### 10 Key Innovations for simple_tui

1. **Contract-Driven Widgets** - DBC for widget state validation
2. **SCOOP-Ready Async** - Non-blocking data loading via actors
3. **Eiffel-Style Layout DSL** - Fluent builder pattern
4. **Hot-Reloadable Styles** - TOML themes, live reload
5. **Accessibility-First** - ARIA-like attributes, cursor placement
6. **Terminal Capability Detection** - Auto-detect and degrade
7. **Snapshot Testing** - Golden-file testing for TUI output
8. **Animation Framework** - Property animation with easing
9. **Theming System** - Semantic colors, built-in themes
10. **Component Inspector** - DevTools-like debug mode

---

## Step 6: Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Architecture | Elm (MVU) | Proven, testable, SCOOP-friendly |
| Rendering | Immediate + double buffer | Simple, 60fps capable |
| Layout | Flexbox-inspired | Web developers familiar |
| Styling | TOML themes | Leverages simple_toml |
| Events | Agents + messages | Flexible, type-safe |
| Backend | Unified trait | Windows + ANSI |

### Phase 1 Widget Set

TUI_LABEL, TUI_BUTTON, TUI_TEXT_FIELD, TUI_CHECKBOX, TUI_PROGRESS, TUI_LIST, TUI_BOX

---

## Step 7: Implementation Roadmap

### Phase 1: Core Infrastructure (2-3 weeks)
- TUI_BACKEND abstraction
- TUI_BUFFER (double buffering)
- TUI_EVENT system
- TUI_STYLE

### Phase 2: Widget Foundation (2 weeks)
- TUI_WIDGET base class
- TUI_BOX layout
- Basic widgets (Label, Button, Progress)

### Phase 3: Interactive Widgets (2 weeks)
- TUI_TEXT_FIELD
- TUI_LIST
- TUI_CHECKBOX / TUI_RADIO

### Phase 4: Polish (1 week)
- Theming, Animation, Docs, Examples

**Estimated Total: 7-8 weeks for v1.0**

---

## References

- [Ratatui](https://ratatui.rs/) - Rust TUI framework
- [Textual](https://textual.textualize.io/) - Python TUI with CSS
- [Bubble Tea](https://github.com/charmbracelet/bubbletea) - Go TUI with Elm
- [FTXUI](https://github.com/ArthurSonzogni/FTXUI) - C++ functional TUI
- [Notcurses](https://github.com/dankamongmen/notcurses) - C TUI with graphics
- [7 Things Learned](https://www.textualize.io/blog/7-things-ive-learned-building-a-modern-tui-framework/)
- [awesome-tuis](https://github.com/rothgar/awesome-tuis)

---

*Research completed: December 27, 2024*
*Produced for: Simple Eiffel ecosystem - simple_tui library design*
