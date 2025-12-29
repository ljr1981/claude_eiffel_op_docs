# EiffelNotebook Implementation Plan

**Created:** 2025-12-18
**Status:** DRAFT - UX & Architecture Finalized
**Oracle Reference:** Query "notebook" or "repl"
**Research:** `D:\prod\reference_docs\research\EIFFEL_REPL_INTERPRETER_RESEARCH.md`

---

## Executive Summary

This plan delivers **EiffelNotebook** - an interactive notebook environment for Eiffel, similar to Jupyter for Python. Users write code in cells, execute incrementally, and see results inline.

**Two-Phase Strategy:**
1. **Phase 1: Accumulated Class MVP** (~1 day) - Ship quickly using melt compilation
2. **Phase 2: Full Interpreter** (~1 week) - Build proper bytecode interpreter while MVP is live

**Deliverables:**
1. `simple_notebook` - Core notebook library with multiple frontend targets
2. `simple_tui` - Reusable TUI framework (separate library)
3. Documentation site at simple-eiffel.github.io/simple_notebook

**Total Estimated LOC:** ~4,000 (Phase 1 core + REPL), ~2,000 (simple_tui framework)

---

## Project Structure (Finalized)

### simple_notebook - Main Library with Multiple Targets

```
simple_notebook/
├── simple_notebook.ecf
│   │
│   ├── target: simple_notebook         -- Core engine library
│   ├── target: simple_notebook_tests   -- Core tests
│   │
│   ├── target: simple_repl             -- Level 2: Enhanced CLI REPL
│   ├── target: simple_repl_tests       -- REPL tests
│   │
│   ├── target: simple_nb_tui           -- Level 4: TUI IDE
│   ├── target: simple_nb_tui_tests     -- TUI tests
│   │
│   ├── target: simple_nb_web           -- Web server (HTMX/Alpine)
│   ├── target: simple_nb_web_tests     -- Web tests
│   │
│   └── target: simple_nb_vsc           -- VS Code bridge (future)
│       └── simple_nb_vsc_tests         -- VS Code tests
│
├── src/                    -- Core engine classes
├── src_repl/               -- REPL-specific classes
├── src_tui/                -- TUI-specific classes
├── src_web/                -- Web-specific classes
├── src_vsc/                -- VS Code-specific classes (future)
├── testing/                -- Test classes
├── www/                    -- Web assets (HTML, JS, CSS)
└── templates/              -- Code generation templates
```

### simple_tui - Separate Reusable TUI Framework

```
simple_tui/
├── simple_tui.ecf
│   ├── target: simple_tui              -- TUI framework library
│   └── target: simple_tui_tests        -- Framework tests
│
├── src/
│   ├── simple_tui.e        -- Main facade
│   ├── tui_window.e        -- Top-level container
│   ├── tui_pane.e          -- Individual pane with border
│   ├── tui_layout.e        -- Split horizontal/vertical
│   ├── tui_text_buffer.e   -- Scrollable text content
│   ├── tui_input_line.e    -- Input with history
│   ├── tui_table.e         -- Tabular display
│   └── tui_event_loop.e    -- Key/resize handling
│
└── Built on: simple_console (colors, cursor, dimensions)
```

### Dependency Graph

```
simple_tui ──────────────────┐
    │                        │
    └── simple_console       │
                             │
simple_notebook ─────────────┤
    │                        │
    ├── simple_json          │
    ├── simple_file          │
    ├── simple_process       │
    ├── simple_uuid          │
    └── simple_template      │
                             │
simple_nb_tui ───────────────┘
    │
    ├── simple_notebook (core)
    └── simple_tui (framework)

simple_nb_web
    │
    ├── simple_notebook (core)
    └── simple_web

simple_repl
    │
    ├── simple_notebook (core)
    └── simple_console
```

---

## UX Architecture: Multi-Frontend Strategy

### Vision: One Engine, Three Interfaces

The notebook engine is UI-agnostic, enabling three distinct frontends:

```
                    ┌─────────────────────────┐
                    │    NOTEBOOK_ENGINE      │
                    │    (Core - UI Agnostic) │
                    └───────────┬─────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌───────────────┐      ┌───────────────┐      ┌───────────────┐
│  CLI REPL     │      │  Web UI       │      │  VS Code      │
│  eiffel_repl  │      │  HTMX/Alpine  │      │  Extension    │
│               │      │               │      │  (via LSP)    │
│  Level 2→4    │      │  Phase 1      │      │  Future       │
└───────────────┘      └───────────────┘      └───────────────┘
        │                       │                       │
        ▼                       ▼                       ▼
   Terminal              Web Browser              VS Code
```

### CLI Evolution Path

| Level | Name | Description | Status |
|-------|------|-------------|--------|
| 1 | Simple REPL | Basic `>>>` style prompt | Skip |
| 2 | Enhanced REPL | Syntax highlighting, history, multi-line | **Build First** |
| 3 | TUI Notebook | Single-pane cell editor | Stepping stone |
| 4 | TUI IDE | EiffelStudio-style split panes | **Ultimate Goal** |

**Decision:** Start with Level 2 (Enhanced REPL) to learn patterns, then evolve to Level 4 (TUI IDE).

---

## UX Design Decisions

This section documents the design choices presented, decisions made, and rationale for each.

### Decision 1: Prompt Style

**Options Presented:**

| Option | Example | Description |
|--------|---------|-------------|
| A | `e[1]>` | Simple, shows cell number |
| B | `e>` | Minimal, no context |
| C | `eiffel[1]>` | Explicit but verbose |
| D | `❯ [1]` | Modern Unicode style |
| E | `λ[1]>` | Lambda-inspired |

**Decision:** Option A - `e[1]>`

**Analysis:**

This is the right choice. Here's why:

1. **Information density:** The cell number `[1]` provides essential context without clutter. Users need to know which cell they're in for `:delete N`, `:edit N`, and mental model of session state.

2. **Brevity:** `e` is short enough to not waste horizontal space (critical in terminals), yet distinctive enough to signal "Eiffel."

3. **Familiarity:** Follows conventions from IPython (`In [1]:`) and Scala REPL (`scala>`) - experienced REPL users will recognize the pattern.

4. **Consistency across UIs:** The `[N]` cell numbering translates directly to web UI (`[1]`, `[2]`) and VS Code (cell indicators), maintaining mental model across frontends.

**Rejected alternatives:**

- Option B (`e>`) loses context - users can't tell which cell they're editing
- Option C (`eiffel[1]>`) wastes 6 characters per line
- Options D/E require Unicode support that some terminals lack

---

### Decision 2: Multi-line Input Termination

**Options Presented:**

| Option | Mechanism | Description |
|--------|-----------|-------------|
| A | Blank line | Empty line terminates input |
| B | Smart detection only | Parser decides completion |
| C | Ctrl+Enter | Explicit key combo to submit |
| D | `:run` command | Type command to execute buffer |
| E | Hybrid | Smart detection + Ctrl+Enter override |

**Decision:** Option E - Hybrid

**Analysis:**

Excellent choice. The hybrid approach is strictly superior:

1. **Smart detection handles 90% of cases:** The parser detects unclosed `if/do/class/loop` blocks and shows continuation prompt (`...`). Most of the time, users just type naturally and the system figures out when code is complete.

2. **Override prevents frustration:** When the parser gets it wrong (edge cases in complex expressions), Ctrl+Enter forces submission. Users never feel trapped.

3. **Supports intentional incomplete code:** Sometimes you want to submit partial code to see an error message. Ctrl+Enter enables this.

**Implementation detail:**

```
e[1]> if x > 0 then      -- Parser: unclosed 'if', show continuation
...       print("yes")   -- Still incomplete
...   end                -- Parser: complete! Auto-submit

e[2]> complex_expression  -- Parser thinks complete, but user wants more
...   Ctrl+Enter         -- Force submit even if parser disagrees
```

**Why other options fall short:**

- Option A (blank line) prevents blank lines in code - breaks formatting
- Option B (smart only) traps users when parser misjudges
- Option C (Ctrl+Enter only) requires extra keystrokes for every submission
- Option D (`:run`) is verbose and breaks flow

---

### Decision 3: Variable Display After Execution

**Options Presented:**

| Option | Display | Example |
|--------|---------|---------|
| A | Always show value | `--- x: INTEGER = 42` |
| B | Only on `:vars` | No automatic display |
| C | Show summary | `--- Defined: x, y, z` |
| D | Show with types + markers | `--- +x: INTEGER, ~y: STRING` |

**Decision:** Option D - Show with types and change markers

**Analysis:**

Strong choice for power users. Here's the refined specification:

```
e[1]> x := 42
      Compiling... done (1.2s, 1 feature)
      --- +x: INTEGER = 42

e[2]> x := x * 2
      y := "hello"
      Compiling... done (0.9s, 1 feature)
      --- ~x: INTEGER = 84, +y: STRING = "hello"
```

**Marker semantics:**
- `+` = newly defined variable
- `~` = modified existing variable
- (no marker) = unchanged (only shown if explicitly referenced)

**Why this is right:**

1. **Immediate feedback loop:** Users see exactly what their code did without typing `:vars`. This accelerates learning and debugging.

2. **Change tracking:** The `+`/`~` markers highlight what's different from the previous state. Critical for understanding mutations.

3. **Type visibility:** Showing types catches type errors early ("wait, that should be REAL not INTEGER").

**Potential concern and counter-argument:**

*Concern:* Could this be too verbose for large objects?

*Counter:* For complex objects, we truncate: `+p: POINT = <instance>` rather than dumping entire state. The `:vars` command provides full inspection when needed.

**Enhancement:** Add a `:quiet` toggle for users who find it too noisy. But default to verbose - it's easier to turn off than to not know it exists.

---

### Decision 4: Compilation Feedback Verbosity

**Options Presented:**

| Option | Output | Example |
|--------|--------|---------|
| A | Spinner only | `...` |
| B | With timing | `Compiling... done (1.2s)` |
| C | With stats | `Compiling... done (1.2s, 3 features)` |
| D | Toggleable verbose | Option to show full compiler output |

**Decision:** Option C - With stats

**Analysis:**

Good choice, with one enhancement needed.

The stats (`3 features`) provide useful signal:
- Confirms the generator created expected structure
- Helps debug when "nothing happens" (0 features = code wasn't recognized)
- Educational: users learn that their code becomes features

**Recommended enhancement:** Also show compilation vs. execution split:

```
e[1]> -- complex code --
      Compiling... done (1.8s, 5 features)
      Running...   done (0.3s)
      output here
```

This distinguishes slow compilation (normal for first run) from slow execution (possible infinite loop). Critical diagnostic information.

**Full specification:**

| Stat | Meaning |
|------|---------|
| Time | Wall-clock compile + execute |
| Features | Number of cell features generated |
| (Optional) | "cached" if using compiled cache |

---

### Decision 5: Compilation Error Display

**Options Presented:**

| Option | Approach | Description |
|--------|----------|-------------|
| A | Raw compiler output | Pass through EiffelStudio errors as-is |
| B | Parsed and reformatted | Map line numbers to cells, clean up |
| C | Brief + expandable | Summary with `:error` for details |
| D | Colorized with context | Syntax-highlighted error context |

**Decision:** Option B - Parsed and reformatted

**Analysis:**

Correct choice. Raw EiffelStudio errors reference the generated class, not the user's cells. We must translate.

**The translation problem:**

EiffelStudio says:
```
Error: VEEN - Feature `undefined_var` is not defined.
Line 47 in class ACCUMULATED_SESSION_20251218_143052
```

But line 47 in the generated class is meaningless to the user. We need:
```
Error in cell [2], line 3:
  Feature 'undefined_var' is not defined.

  |  x := undefined_var + 1
         ^^^^^^^^^^^^^^^^^^
```

**Implementation requirements:**

1. **Line number mapping:** The generator must track `generated_line -> (cell_id, cell_line)` mapping.

2. **Error parser:** Parse EiffelStudio's output format (varies by error type).

3. **Context extraction:** Show the offending line from the original cell code.

**Trade-off acknowledged:**

Parsing EiffelStudio errors is fragile - error format may vary. Fallback strategy: if parsing fails, show raw output with a note: "Could not map to cell. Raw compiler output below."

**Suggestion for improvement:** Also implement Option D (colorization) as enhancement:
- Red for error text
- Yellow for warning
- Cyan for the offending code line
- This is additive, not instead of

---

### Decision 6: Session Persistence

**Options Presented:**

| Option | Behavior | Description |
|--------|----------|-------------|
| A | Explicit only | `:save` required |
| B | Auto-save always | Save to history file automatically |
| C | Ask on quit | Prompt if unsaved |
| D | Both + restore | Auto-checkpoint + explicit save + restore |

**Decision:** Option D - Both with restore capability

**Analysis:**

This is the power-user choice. Here's the full specification:

**Three-tier persistence:**

1. **Auto-checkpoint (crash recovery):**
   - Every N seconds (configurable, default 30), checkpoint to `~/.eiffel_repl/autosave.enb`
   - On crash/kill, user can recover: `eiffel_repl --recover`
   - Silent, invisible to user during normal operation

2. **Explicit save (user-controlled):**
   - `:save filename.enb` saves to named file
   - `:save` (no arg) saves to last used filename or prompts
   - Creates portable, shareable notebook files

3. **Session history (restore prior sessions):**
   - `:sessions` lists recent sessions with timestamps
   - `:restore N` or `:restore 2025-12-18` brings back that session
   - Stored in `~/.eiffel_repl/sessions/`

**Answering the question: "Can users bring back prior sessions?"**

Yes! This is the key feature:

```
e[1]> :sessions
Recent sessions:
  [1] 2025-12-18 14:30 - fibonacci.enb (5 cells)
  [2] 2025-12-17 09:15 - untitled (3 cells)
  [3] 2025-12-16 16:45 - data_analysis.enb (12 cells)

e[1]> :restore 2
Restoring session from 2025-12-17 09:15...
Loaded 3 cells.

e[1]> :cells
[1] x := 42
[2] y := x * 2
[3] print(y)
```

**Why this matters:**

1. **Exploratory work is never lost:** Users experiment freely knowing they can get back to any previous state.

2. **Teaching/demo use case:** Instructor can prepare a session, save it, and students can `:restore` it.

3. **Crash resilience:** The auto-checkpoint means even hard crashes lose at most 30 seconds of work.

**Storage structure:**

```
~/.eiffel_repl/
├── autosave.enb           # Current crash-recovery checkpoint
├── config.json            # User preferences
├── history.txt            # Command history (readline-style)
└── sessions/
    ├── 2025-12-18_143052.enb
    ├── 2025-12-17_091500.enb
    └── 2025-12-16_164500.enb
```

---

### Decision 7: Key Binding Style

**Options Presented:**

| Option | Style | Description |
|--------|-------|-------------|
| Vim | Modal | j/k navigate, Enter to edit, Esc to exit |
| Emacs | Chords | Ctrl+N/P navigate, Ctrl+Enter execute |
| Nano | Simple | Arrow keys, Ctrl+shortcuts |

**Decision:** Nano style (simple, non-modal)

**Analysis:**

Right choice for the target audience. Here's the reasoning:

**Who will use EiffelNotebook?**

1. **Eiffel learners:** New to the language, don't want to also learn vim
2. **Scientists/analysts:** Focused on their domain, not editor mastery
3. **Experienced devs:** Already have their editor, use notebook for experiments

All three groups benefit from obvious, discoverable keybindings.

**Nano-style specification:**

| Key | Action |
|-----|--------|
| Up/Down | Navigate history (in single-line mode) |
| Ctrl+Up/Down | Navigate between cells (in TUI mode) |
| Enter | Newline (multi-line mode) or submit (if complete) |
| Ctrl+Enter | Force submit (override smart detection) |
| Ctrl+R | Run current cell |
| Ctrl+Shift+R | Run all cells |
| Ctrl+S | Save session |
| Ctrl+L | Clear screen |
| Ctrl+C | Cancel current input / interrupt execution |
| Ctrl+D | Quit (with save prompt if unsaved) |
| Tab | Autocomplete (keywords, variables) |
| Ctrl+Z | Undo last edit |

**Why not vim-style?**

1. **Barrier to entry:** Modal editing confuses beginners
2. **Notebook is not a text editor:** You're writing small code snippets, not editing large files
3. **Consistency with web UI:** Web notebook uses standard text input, not vim mode

**Possible future enhancement:**

For power users who really want vim, add `:set keymap vim` option. But default to Nano.

---

## CLI REPL Specification (Level 2: Enhanced REPL)

Based on the decisions above, here's the complete REPL specification:

### Visual Layout

```
+------------------------------------------------------------+
|  Eiffel REPL v1.0 - :help for commands - Ctrl+D to quit    |
+------------------------------------------------------------+

e[1]> x := 42
      Compiling... done (1.2s, 1 feature)
      Running...   done (0.1s)
      --- +x: INTEGER = 42

e[2]> if x > 0 then
...       print("positive")
...   else
...       print("negative")
...   end
      Compiling... done (0.9s, 1 feature)
      Running...   done (0.1s)
      positive

e[3]> y := x * 2
      z := "result: " + y.out
      Compiling... done (0.8s, 1 feature)
      Running...   done (0.1s)
      --- +y: INTEGER = 84, +z: STRING = "result: 84"

e[4]> :vars
+--------------------------------------------------+
| Variable | Type    | Value          | Cell       |
+--------------------------------------------------+
| x        | INTEGER | 42             | [1]        |
| y        | INTEGER | 84             | [3]        |
| z        | STRING  | "result: 84"   | [3]        |
+--------------------------------------------------+

e[5]> :save my_session.enb
Saved: my_session.enb (4 cells)

e[6]> :quit
Session auto-saved. Goodbye!
```

### Command Reference

| Command | Alias | Description |
|---------|-------|-------------|
| `:help` | `:h` | Show all commands |
| `:quit` | `:q` | Exit REPL (prompts to save if unsaved) |
| `:clear` | `:c` | Reset session (clear all cells) |
| `:vars` | `:v` | Show all variables with types and values |
| `:cells` | `:ls` | List all cells with code preview |
| `:delete N` | `:d N` | Delete cell N |
| `:edit N` | `:e N` | Re-edit cell N (puts code at prompt) |
| `:run` | `:r` | Re-execute all cells |
| `:run N` | `:r N` | Re-execute cell N only |
| `:save [FILE]` | `:w` | Save session to file |
| `:load FILE` | `:o` | Load session from file |
| `:sessions` | `:ss` | List recent sessions |
| `:restore N` | `:rs N` | Restore session N |
| `:quiet` | | Toggle variable display after execution |
| `:verbose` | | Toggle full compiler output |
| `:time` | | Toggle timing display |

### Error Display Example

```
e[3]> x := undefined_variable + 1
      Compiling... FAILED (0.4s)

      Error in cell [3], line 1:
      +--------------------------------------------
      | VEEN: Feature 'undefined_variable' is not defined
      |
      |   x := undefined_variable + 1
      |        ^^^^^^^^^^^^^^^^^^
      |
      | Did you mean: x, y, z?
      +--------------------------------------------
```

---

## TUI IDE Vision (Level 4: Future)

The ultimate goal: EiffelStudio as a TUI.

### Layout Mockup

```
+-- Cells -----------------------------+-- Variables ------------------+
| [1] x := 42                          | x: INTEGER = 42              |
|     --- +x: INTEGER = 42             | y: INTEGER = 84              |
|                                      | p: POINT = <instance>        |
| [2] class POINT                      |   .x: REAL = 3.0             |
|     feature                          |   .y: REAL = 4.0             |
|         x, y: REAL                   |                              |
|         make (a, b: REAL)            +-- Output ---------------------+
|             do                       | 3.0, 4.0                     |
|                 x := a               |                              |
|                 y := b               |                              |
|             end                      |                              |
|     end                              |                              |
|     --- Defined: POINT               |                              |
|                                      +-- Errors ---------------------+
| [3]> p: POINT                        | (none)                       |
|      create p.make(3.0, 4.0)         |                              |
|      print(p.x.out + ", " + p.y.out) |                              |
|     -----------------                |                              |
|     Compiling...                     |                              |
+-- Command ---------------------------+------------------------------+
| e[3]> _                                                             |
+-- Status -----------------------------------------------------------+
| Cell 3/3 | Compiling | :help | Ctrl+R run | Ctrl+S save | Ctrl+Q   |
+---------------------------------------------------------------------+
```

### Pane Descriptions

| Pane | Purpose | Scrollable | Updates |
|------|---------|------------|---------|
| Cells | All cells with code and output | Yes | On execution |
| Variables | Live variable table with expandable objects | Yes | After each cell |
| Output | Stdout from current/last execution | Yes | Streaming during execution |
| Errors | Compilation and runtime errors | Yes | On error |
| Command | Input prompt (same as Level 2) | No | User input |
| Status | Current state, shortcuts reminder | No | Always visible |

### Navigation (Nano-style)

| Key | Action |
|-----|--------|
| Ctrl+Up/Down | Move between cells in Cells pane |
| Ctrl+Left/Right | Switch focus between panes |
| Ctrl+E | Expand/collapse current variable in Variables pane |
| PgUp/PgDn | Scroll current pane |
| F1-F4 | Focus specific pane (Cells, Vars, Output, Errors) |

### Required Infrastructure: simple_tui

To build Level 4, we need a TUI framework:

```
simple_tui/
├── src/
│   ├── tui_window.e        -- Top-level window container
│   ├── tui_pane.e          -- Individual pane with border, title
│   ├── tui_layout.e        -- Split horizontal/vertical
│   ├── tui_text_buffer.e   -- Scrollable text content
│   ├── tui_input_line.e    -- Single-line input with history
│   ├── tui_table.e         -- Tabular data display
│   ├── tui_event_loop.e    -- Key input, resize handling
│   └── simple_tui.e        -- Main facade
└── Built on: simple_console (colors, cursor, dimensions)
```

**Estimated LOC:** ~2,000 for simple_tui framework

This framework is reusable for other TUI tools (e.g., TUI database browser, TUI log viewer).

---

## Strategic Context

### Why Two Phases?

Based on research into EiffelStudio's Melting Ice bytecode interpreter (177 opcodes, stack-based VM), building a true Eiffel interpreter is feasible but non-trivial. The accumulated class approach lets us:

1. **Ship Fast** - Get product to market in ~1 day
2. **Gather Feedback** - Real users inform interpreter requirements
3. **Parallel Development** - Build interpreter while MVP serves users
4. **Risk Mitigation** - If interpreter takes longer, we still have working product

### Accumulated Class Pattern

Transform notebook cells into a single Eiffel class:

```eiffel
-- Generated from notebook: my_analysis.enb
class ACCUMULATED_SESSION_20251218_143052
inherit
    ANY
        redefine default_create end

create default_create

feature -- Cell Outputs
    cell_1_result: detachable ANY
    cell_2_result: detachable ANY
    cell_3_result: detachable ANY

feature {NONE} -- Initialization
    default_create
        do
            execute_all
        end

feature -- Execution
    execute_all
        do
            execute_cell_1
            execute_cell_2
            execute_cell_3
        end

    execute_cell_1
        local
            -- Cell 1 locals
            x: INTEGER
        do
            -- Cell 1 code
            x := 42
            cell_1_result := x
        end

    execute_cell_2
        local
            -- Cell 2 locals
            y: INTEGER
        do
            -- Cell 2 code (can reference cell_1_result)
            if attached {INTEGER} cell_1_result as prev then
                y := prev * 2
            end
            cell_2_result := y
        end

    execute_cell_3
        do
            -- Cell 3: print output
            if attached cell_2_result as r then
                io.put_string ("Result: " + r.out + "%N")
            end
            cell_3_result := "Done"
        end

end
```

**Compilation:** `ec -batch -melt -config notebook.ecf -target session`
**Execution:** Run generated executable, capture stdout/stderr

---

## Phase 1: Accumulated Class MVP

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      Web Browser                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              HTMX + Alpine.js UI                     │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐            │    │
│  │  │  Cell 1  │ │  Cell 2  │ │  Cell 3  │  [+ Add]   │    │
│  │  │  [Run]   │ │  [Run]   │ │  [Run]   │            │    │
│  │  │  Output  │ │  Output  │ │  Output  │            │    │
│  │  └──────────┘ └──────────┘ └──────────┘            │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ HTTP (HTMX partial updates)
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    simple_notebook Server                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │               NOTEBOOK_WEB_HANDLER                   │    │
│  │                                                      │    │
│  │  Routes:                                            │    │
│  │    GET  /                    → notebook list        │    │
│  │    GET  /notebook/:id        → load notebook        │    │
│  │    POST /notebook            → create notebook      │    │
│  │    POST /cell/:id/run        → execute cell         │    │
│  │    POST /cell/:id/run-all    → execute all cells    │    │
│  │    PUT  /cell/:id            → update cell code     │    │
│  │    POST /notebook/:id/cell   → add cell             │    │
│  │    DELETE /cell/:id          → remove cell          │    │
│  └─────────────────────────────────────────────────────┘    │
│                              │                               │
│                              ▼                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                   NOTEBOOK                          │    │
│  │  - cells: ARRAYED_LIST [NOTEBOOK_CELL]             │    │
│  │  - name, created_at, modified_at                   │    │
│  │  - to_json / from_json                             │    │
│  └─────────────────────────────────────────────────────┘    │
│                              │                               │
│                              ▼                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │           ACCUMULATED_CLASS_GENERATOR               │    │
│  │  - generate_class (notebook): STRING               │    │
│  │  - generate_ecf (notebook): STRING                 │    │
│  │  - collect_shared_state (cells): HASH_TABLE        │    │
│  └─────────────────────────────────────────────────────┘    │
│                              │                               │
│                              ▼                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                  CELL_EXECUTOR                      │    │
│  │  - compile (generated_class): COMPILATION_RESULT   │    │
│  │  - execute (exe_path): EXECUTION_RESULT            │    │
│  │  - timeout handling, process management            │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Class Structure

| Class | Purpose | LOC Est. |
|-------|---------|----------|
| `NOTEBOOK_CONFIG` | Configuration container with validation | ~200 |
| `CONFIG_DETECTOR` | Auto-detect EiffelStudio and paths | ~150 |
| `CONFIG_WIZARD` | Interactive first-run setup (CLI) | ~100 |
| `NOTEBOOK` | Container for cells with JSON serialization | ~200 |
| `NOTEBOOK_CELL` | Individual cell: code, output, type, order | ~150 |
| `ACCUMULATED_CLASS_GENERATOR` | Transforms cells into single Eiffel class | ~400 |
| `CELL_EXECUTOR` | Compiles/runs generated class via simple_process | ~350 |
| `COMPILATION_RESULT` | Holds compiler output, errors, warnings | ~100 |
| `EXECUTION_RESULT` | Holds runtime output, exit code, timing | ~100 |
| `NOTEBOOK_STORAGE` | File-based notebook persistence (.enb JSON) | ~200 |
| `NOTEBOOK_WEB_HANDLER` | HTTP routes using simple_web | ~500 |
| `NOTEBOOK_HTMX_RENDERER` | HTMX partial HTML generation | ~400 |
| `SIMPLE_NOTEBOOK` | Main facade with type anchors | ~150 |

**Subtotal Eiffel:** ~3,000 LOC

### Frontend Assets

| File | Purpose | LOC Est. |
|------|---------|----------|
| `index.html` | Main page template | ~150 |
| `notebook.js` | Alpine.js components | ~200 |
| `notebook.css` | Styling (CodeMirror integration) | ~150 |
| `partials/*.html` | HTMX partial templates | ~200 |

**Subtotal Frontend:** ~700 LOC

### ECF Dependencies

| File | Purpose | LOC Est. |
|------|---------|----------|
| `simple_notebook.ecf` | Main library config | ~80 |
| `notebook_session.ecf.template` | Template for generated sessions | ~60 |

**Subtotal Config:** ~140 LOC

### Total Phase 1: ~3,840 LOC

---

## Detailed Class Specifications

### NOTEBOOK

```eiffel
class NOTEBOOK

create
    make, make_from_json

feature -- Access
    id: STRING
    name: STRING
    cells: ARRAYED_LIST [NOTEBOOK_CELL]
    created_at: DATE_TIME
    modified_at: DATE_TIME

feature -- Queries
    cell_by_id (a_id: STRING): detachable NOTEBOOK_CELL
    cell_count: INTEGER
    code_cells: ARRAYED_LIST [NOTEBOOK_CELL]
        -- Only executable code cells (not markdown)

feature -- Commands
    add_cell (a_cell: NOTEBOOK_CELL)
    remove_cell (a_id: STRING)
    move_cell (a_id: STRING; a_new_position: INTEGER)
    update_cell_code (a_id: STRING; a_code: STRING)

feature -- Serialization
    to_json: JSON_OBJECT
    from_json (a_json: JSON_OBJECT)

invariant
    cells_ordered: across cells as c all c.item.order = c.cursor_index end
end
```

### NOTEBOOK_CELL

```eiffel
class NOTEBOOK_CELL

create
    make

feature -- Access
    id: STRING
    cell_type: STRING  -- "code" or "markdown"
    code: STRING
    output: detachable STRING
    error: detachable STRING
    execution_time_ms: INTEGER
    order: INTEGER
    status: STRING  -- "idle", "running", "success", "error"

feature -- Status
    is_code_cell: BOOLEAN
    is_markdown_cell: BOOLEAN
    has_output: BOOLEAN
    has_error: BOOLEAN

feature -- Commands
    set_code (a_code: STRING)
    set_output (a_output: STRING)
    set_error (a_error: STRING)
    set_status (a_status: STRING)
    clear_output

feature -- Serialization
    to_json: JSON_OBJECT
    from_json (a_json: JSON_OBJECT)
end
```

### ACCUMULATED_CLASS_GENERATOR

```eiffel
class ACCUMULATED_CLASS_GENERATOR

feature -- Generation
    generate_class (a_notebook: NOTEBOOK): STRING
        -- Generate complete Eiffel class from notebook cells
        require
            notebook_has_cells: a_notebook.cell_count > 0
        local
            code_cells: ARRAYED_LIST [NOTEBOOK_CELL]
            shared_vars: HASH_TABLE [STRING, STRING]  -- name -> type
        do
            code_cells := a_notebook.code_cells
            shared_vars := collect_shared_variables (code_cells)

            create Result.make_empty
            Result.append (generate_header (a_notebook))
            Result.append (generate_result_attributes (code_cells))
            Result.append (generate_shared_attributes (shared_vars))
            Result.append (generate_creation)
            Result.append (generate_execute_all (code_cells))
            across code_cells as c loop
                Result.append (generate_cell_feature (c.item, c.cursor_index))
            end
            Result.append (generate_footer)
        ensure
            valid_eiffel: is_valid_eiffel_class (Result)
        end

    generate_ecf (a_notebook: NOTEBOOK; a_class_name: STRING): STRING
        -- Generate ECF file for compilation

feature {NONE} -- Implementation
    collect_shared_variables (cells: ARRAYED_LIST [NOTEBOOK_CELL]): HASH_TABLE [STRING, STRING]
        -- Parse cells for `shared x: TYPE` declarations

    generate_cell_feature (cell: NOTEBOOK_CELL; index: INTEGER): STRING
        -- Generate execute_cell_N feature

    extract_locals (code: STRING): STRING
        -- Extract local declarations from cell code

    extract_body (code: STRING): STRING
        -- Extract executable statements from cell code
end
```

### CELL_EXECUTOR

```eiffel
class CELL_EXECUTOR

create
    make

feature -- Configuration
    workspace_path: PATH
        -- Directory for generated files

    timeout_seconds: INTEGER
        -- Maximum execution time (default: 30)

    eiffel_compiler_path: PATH
        -- Path to ec.exe

feature -- Execution
    compile_and_run (a_class: STRING; a_ecf: STRING): EXECUTION_RESULT
        require
            valid_class: not a_class.is_empty
            valid_ecf: not a_ecf.is_empty
        local
            class_file, ecf_file, exe_path: PATH
            compile_result: COMPILATION_RESULT
        do
            -- Write generated files
            class_file := write_class_file (a_class)
            ecf_file := write_ecf_file (a_ecf)

            -- Compile
            compile_result := compile (ecf_file)

            if compile_result.success then
                exe_path := compile_result.executable_path
                Result := execute (exe_path)
            else
                create Result.make_compilation_error (compile_result.errors)
            end
        end

feature {NONE} -- Implementation
    compile (ecf_path: PATH): COMPILATION_RESULT
        -- Run: ec -batch -melt -config {ecf_path} -c_compile
        local
            process: PROCESS
            cmd: STRING
        do
            cmd := eiffel_compiler_path.name + " -batch -melt -config "
                   + ecf_path.name + " -c_compile"
            create process.make_shell (cmd)
            process.set_timeout (timeout_seconds * 1000)
            process.launch
            process.wait_for_exit

            create Result.make (process.exit_code = 0,
                               process.output,
                               process.error_output)
        end

    execute (exe_path: PATH): EXECUTION_RESULT
        -- Run generated executable, capture output
        local
            process: PROCESS
            start_time: DATE_TIME
        do
            create start_time.make_now
            create process.make (exe_path.name)
            process.set_timeout (timeout_seconds * 1000)
            process.launch
            process.wait_for_exit

            create Result.make (
                process.exit_code,
                process.output,
                process.error_output,
                elapsed_ms (start_time)
            )
        end
end
```

### NOTEBOOK_WEB_HANDLER

```eiffel
class NOTEBOOK_WEB_HANDLER

inherit
    WEB_HANDLER

feature -- Routes
    handle_request (req: WEB_REQUEST; res: WEB_RESPONSE)
        do
            inspect req.method + " " + route_pattern (req.path)
            when "GET /" then
                handle_index (req, res)
            when "GET /notebook/:id" then
                handle_show_notebook (req, res)
            when "POST /notebook" then
                handle_create_notebook (req, res)
            when "POST /cell/:id/run" then
                handle_run_cell (req, res)
            when "POST /cell/:id/run-all" then
                handle_run_all_cells (req, res)
            when "PUT /cell/:id" then
                handle_update_cell (req, res)
            when "POST /notebook/:id/cell" then
                handle_add_cell (req, res)
            when "DELETE /cell/:id" then
                handle_delete_cell (req, res)
            when "POST /notebook/:id/save" then
                handle_save_notebook (req, res)
            else
                res.set_status (404)
                res.set_body ("Not found")
            end
        end

feature {NONE} -- Handlers
    handle_run_cell (req: WEB_REQUEST; res: WEB_RESPONSE)
        local
            cell_id: STRING
            notebook: NOTEBOOK
            cell: NOTEBOOK_CELL
            generator: ACCUMULATED_CLASS_GENERATOR
            executor: CELL_EXECUTOR
            result: EXECUTION_RESULT
            renderer: NOTEBOOK_HTMX_RENDERER
        do
            cell_id := req.path_parameter ("id")
            notebook := storage.notebook_for_cell (cell_id)
            cell := notebook.cell_by_id (cell_id)

            -- Mark as running (HTMX will poll for updates)
            cell.set_status ("running")

            -- Generate accumulated class up to this cell
            create generator
            generated_class := generator.generate_class_to_cell (notebook, cell)
            generated_ecf := generator.generate_ecf (notebook, class_name)

            -- Compile and execute
            create executor.make (workspace_path)
            result := executor.compile_and_run (generated_class, generated_ecf)

            -- Update cell with results
            if result.success then
                cell.set_output (result.stdout)
                cell.set_status ("success")
            else
                cell.set_error (result.combined_errors)
                cell.set_status ("error")
            end
            cell.set_execution_time_ms (result.execution_time_ms)

            -- Return HTMX partial for cell update
            create renderer
            res.set_header ("Content-Type", "text/html")
            res.set_body (renderer.render_cell (cell))
        end
end
```

---

## HTTP API Reference

### Notebook Operations

| Method | Path | Description | Request Body | Response |
|--------|------|-------------|--------------|----------|
| `GET` | `/` | List all notebooks | - | HTML page |
| `POST` | `/notebook` | Create notebook | `{"name": "..."}` | Redirect to notebook |
| `GET` | `/notebook/:id` | Load notebook | - | HTML page |
| `POST` | `/notebook/:id/save` | Save notebook | - | 200 OK |
| `DELETE` | `/notebook/:id` | Delete notebook | - | Redirect to index |

### Cell Operations

| Method | Path | Description | Request Body | Response |
|--------|------|-------------|--------------|----------|
| `POST` | `/notebook/:id/cell` | Add cell | `{"type": "code"}` | HTMX partial |
| `PUT` | `/cell/:id` | Update cell code | `{"code": "..."}` | 200 OK |
| `DELETE` | `/cell/:id` | Delete cell | - | 200 OK |
| `POST` | `/cell/:id/run` | Execute single cell | - | HTMX partial |
| `POST` | `/cell/:id/run-all` | Execute all cells to this one | - | HTMX partial |

---

## Frontend Design

### HTMX Integration

```html
<!-- Cell template -->
<div id="cell-{id}" class="notebook-cell" data-cell-id="{id}">
    <div class="cell-toolbar">
        <span class="cell-number">[{order}]</span>
        <button hx-post="/cell/{id}/run"
                hx-target="#cell-{id}"
                hx-swap="outerHTML"
                hx-indicator="#cell-{id}-spinner">
            ▶ Run
        </button>
        <button hx-post="/cell/{id}/run-all"
                hx-target="#notebook-cells"
                hx-swap="innerHTML">
            ▶▶ Run All
        </button>
        <button hx-delete="/cell/{id}"
                hx-target="#cell-{id}"
                hx-swap="outerHTML swap:1s">
            ✕
        </button>
        <span id="cell-{id}-spinner" class="htmx-indicator">⏳</span>
    </div>

    <div class="cell-input">
        <textarea x-data="cellEditor('{id}')"
                  x-on:blur="save()"
                  hx-put="/cell/{id}"
                  hx-trigger="blur"
                  hx-vals='{"code": this.value}'>{code}</textarea>
    </div>

    <div class="cell-output {status}">
        {#if output}
            <pre class="output-text">{output}</pre>
        {/if}
        {#if error}
            <pre class="output-error">{error}</pre>
        {/if}
        {#if execution_time_ms}
            <span class="execution-time">{execution_time_ms}ms</span>
        {/if}
    </div>
</div>
```

### Alpine.js Components

```javascript
// Cell editor with CodeMirror integration
function cellEditor(cellId) {
    return {
        cellId: cellId,
        editor: null,

        init() {
            this.editor = CodeMirror.fromTextArea(this.$el, {
                mode: 'eiffel',
                lineNumbers: true,
                theme: 'github',
                extraKeys: {
                    'Shift-Enter': () => this.runCell(),
                    'Ctrl-Enter': () => this.runAll()
                }
            });
        },

        save() {
            fetch(`/cell/${this.cellId}`, {
                method: 'PUT',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({code: this.editor.getValue()})
            });
        },

        runCell() {
            htmx.trigger(`#cell-${this.cellId} button[hx-post*="run"]`, 'click');
        },

        runAll() {
            htmx.trigger(`#cell-${this.cellId} button[hx-post*="run-all"]`, 'click');
        }
    };
}
```

---

## Edge Cases and Error Handling

### Compilation Failures

```eiffel
handle_compilation_error (cell: NOTEBOOK_CELL; errors: STRING)
    local
        parsed_errors: ARRAYED_LIST [COMPILER_ERROR]
        relevant_errors: ARRAYED_LIST [COMPILER_ERROR]
    do
        -- Parse EiffelStudio error output
        parsed_errors := parse_compiler_errors (errors)

        -- Map line numbers back to cell
        relevant_errors := map_errors_to_cell (parsed_errors, cell)

        -- Format for display
        cell.set_error (format_errors (relevant_errors))
        cell.set_status ("error")
    end
```

### Infinite Loop Protection

```eiffel
feature -- Execution Safety
    timeout_seconds: INTEGER = 30
        -- Maximum execution time before kill

    execute_with_timeout (exe: PATH): EXECUTION_RESULT
        local
            process: PROCESS
            timer: TIMEOUT_TIMER
        do
            create process.make (exe.name)
            create timer.make (timeout_seconds * 1000)

            timer.set_action (agent process.terminate)
            timer.start

            process.launch
            process.wait_for_exit

            timer.cancel

            if timer.was_triggered then
                create Result.make_timeout (
                    "Execution exceeded " + timeout_seconds.out + " seconds"
                )
            else
                create Result.make (process.exit_code, process.output, process.error_output, 0)
            end
        end
```

### Cell Reordering

When cells are reordered, shared state dependencies may break:

```eiffel
validate_cell_dependencies (notebook: NOTEBOOK): ARRAYED_LIST [DEPENDENCY_ERROR]
    local
        defined_vars: HASH_TABLE [INTEGER, STRING]  -- var_name -> defining_cell_order
        used_vars: ARRAYED_LIST [TUPLE [cell: INTEGER; var: STRING]]
    do
        create Result.make (0)
        create defined_vars.make (10)

        across notebook.code_cells as c loop
            -- Track variables defined in this cell
            across extract_defined_variables (c.item.code) as v loop
                defined_vars.force (c.cursor_index, v.item)
            end

            -- Check variables used in this cell
            across extract_used_variables (c.item.code) as v loop
                if defined_vars.has (v.item) then
                    if defined_vars.item (v.item) > c.cursor_index then
                        -- Variable used before defined!
                        Result.extend (create {DEPENDENCY_ERROR}.make (
                            c.cursor_index, v.item, defined_vars.item (v.item)
                        ))
                    end
                end
            end
        end
    end
```

### SCOOP Compatibility

All execution happens in a subprocess, so SCOOP concerns are minimal. The web server uses standard EWF patterns.

---

## File Structure

```
D:\prod\simple_notebook\
├── src\
│   ├── simple_notebook.e              -- Main facade
│   ├── notebook_config.e              -- Configuration container
│   ├── config_detector.e              -- Auto-detect EiffelStudio
│   ├── config_wizard.e                -- First-run setup (CLI)
│   ├── notebook.e                     -- Notebook container
│   ├── notebook_cell.e                -- Individual cell
│   ├── accumulated_class_generator.e  -- Code generation
│   ├── cell_executor.e                -- Compilation/execution
│   ├── compilation_result.e           -- Compiler output
│   ├── execution_result.e             -- Runtime output
│   ├── notebook_storage.e             -- File persistence
│   ├── notebook_web_handler.e         -- HTTP routes
│   └── notebook_htmx_renderer.e       -- HTML rendering
├── templates\
│   ├── notebook_session.ecf.template  -- ECF template
│   └── accumulated_class.e.template   -- Class template
├── www\
│   ├── index.html                     -- Main page
│   ├── notebook.js                    -- Alpine.js components
│   ├── notebook.css                   -- Styles
│   ├── codemirror\                    -- CodeMirror assets
│   │   ├── codemirror.min.js
│   │   ├── codemirror.min.css
│   │   └── mode\
│   │       └── eiffel.js              -- Eiffel syntax highlighting
│   └── partials\
│       ├── cell.html                  -- Cell partial
│       ├── cell_output.html           -- Output partial
│       └── notebook_list.html         -- Index partial
├── testing\
│   ├── lib_tests.e                    -- Test runner
│   ├── test_config.e                  -- Configuration tests
│   ├── test_notebook.e                -- Notebook tests
│   ├── test_generator.e               -- Code gen tests
│   ├── test_executor.e                -- Execution tests
│   └── test_web_handler.e             -- HTTP tests
├── docs\
│   └── index.html                     -- IUARC documentation
├── simple_notebook.ecf                -- Library ECF
├── README.md
├── CHANGELOG.md
└── package.json
```

---

## ECF Configuration

### simple_notebook.ecf

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<system xmlns="http://www.eiffel.com/developers/xml/configuration-1-23-0"
        name="simple_notebook"
        uuid="[GENERATE-NEW-UUID]"
        library_target="simple_notebook">

    <description>Interactive Eiffel notebook environment</description>

    <target name="simple_notebook">
        <root all_classes="true"/>
        <version major="1" minor="0" release="0" build="0"/>

        <file_rule>
            <exclude>/EIFGENs$</exclude>
            <exclude>/testing$</exclude>
        </file_rule>

        <option warning="warning" syntax="standard" manifest_array_type="mismatch_warning">
            <assertions precondition="true" postcondition="true"
                       check="true" invariant="true"/>
        </option>

        <setting name="console_application" value="true"/>
        <setting name="concurrency" value="scoop"/>
        <setting name="void_safety" value="all"/>

        <capability>
            <concurrency support="scoop"/>
            <void_safety support="all"/>
        </capability>

        <!-- Dependencies -->
        <library name="base" location="$ISE_LIBRARY/library/base/base.ecf"/>
        <library name="time" location="$ISE_LIBRARY/library/time/time.ecf"/>

        <!-- Simple Eiffel -->
        <library name="simple_json" location="$SIMPLE_EIFFEL/simple_json/simple_json.ecf"/>
        <library name="simple_file" location="$SIMPLE_EIFFEL/simple_file/simple_file.ecf"/>
        <library name="simple_process" location="$SIMPLE_EIFFEL/simple_process/simple_process.ecf"/>
        <library name="simple_web" location="$SIMPLE_EIFFEL/simple_web/simple_web.ecf"/>
        <library name="simple_uuid" location="$SIMPLE_EIFFEL/simple_uuid/simple_uuid.ecf"/>
        <library name="simple_template" location="$SIMPLE_EIFFEL/simple_template/simple_template.ecf"/>

        <cluster name="src" location=".\src\" recursive="true"/>
    </target>

    <target name="simple_notebook_tests" extends="simple_notebook">
        <root class="LIB_TESTS" feature="make"/>
        <library name="testing" location="$ISE_LIBRARY/library/testing/testing.ecf"/>
        <library name="simple_testing" location="$SIMPLE_EIFFEL/simple_testing/simple_testing.ecf"/>
        <cluster name="testing" location=".\testing\"/>
    </target>

</system>
```

---

## Implementation Phases (Bottom-Up, Test-Driven)

The implementation follows a strict bottom-up approach: each layer is proven solid through comprehensive tests before building the next layer on top.

> **No plan survives contact with the enemy!**
> This plan will evolve as we learn. Use:
> - `[x]` = completed
> - `[ ]` = pending
> - `~~strikethrough~~` = abandoned/changed
> - **PIVOT:** notes for significant changes

```
Phase 5: simple_nb_vsc ────────────────────────── Future
              │
Phase 4: simple_nb_tui + simple_tui ───────────── TUI IDE
              │
Phase 3: simple_nb_web ────────────────────────── Web UI
              │
Phase 2: simple_repl ──────────────────────────── CLI REPL
              │
Phase 1: simple_notebook (core) ───────────────── Foundation
              │
         simple_notebook_tests ────────────────── Proof
```

---

### Phase 1: Core Engine (simple_notebook + simple_notebook_tests)

- [ ] **Phase 1 Complete**

**Goal:** Build and prove the notebook engine with full ec.exe integration.

**Principle:** Tests exercise real compilation. No mocks for the compiler - if tests pass, we know ec.exe integration works.

#### Phase 1.0: Configuration

- [ ] `NOTEBOOK_CONFIG` - Configuration container
- [ ] `CONFIG_DETECTOR` - Auto-detect compiler paths
- [ ] `CONFIG_WIZARD` - First-run setup (CLI)
- [ ] `config.json` schema defined
- [ ] `test_config.e` - Configuration tests passing

**Config file location:** `~/.eiffel_notebook/config.json`

**config.json Schema:**

```json
{
    "eiffel_compiler": "C:/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/studio/spec/win64/bin/ec.exe",
    "ise_library": "C:/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/library",
    "simple_eiffel": "D:/prod",
    "workspace_dir": "~/.eiffel_notebook/workspace",
    "timeout_seconds": 30,
    "autosave_interval_seconds": 30,
    "history_size": 1000,
    "repl": {
        "prompt_style": "e[N]>",
        "show_timing": true,
        "show_variable_changes": true,
        "quiet_mode": false
    },
    "web": {
        "port": 8080,
        "host": "localhost"
    }
}
```

**NOTEBOOK_CONFIG Class:**

```eiffel
class NOTEBOOK_CONFIG

create
    make, make_from_file, make_with_defaults

feature -- Access
    eiffel_compiler: PATH
        -- Path to ec.exe

    ise_library: PATH
        -- Path to ISE_LIBRARY

    simple_eiffel: PATH
        -- Path to SIMPLE_EIFFEL root

    workspace_dir: PATH
        -- Working directory for generated files

    timeout_seconds: INTEGER
        -- Maximum execution time

    autosave_interval_seconds: INTEGER
        -- Auto-checkpoint frequency

    history_size: INTEGER
        -- Command history limit

feature -- Status
    is_valid: BOOLEAN
        -- All required paths exist and are accessible

    validation_errors: ARRAYED_LIST [STRING]
        -- List of configuration problems

feature -- REPL Settings
    prompt_style: STRING
    show_timing: BOOLEAN
    show_variable_changes: BOOLEAN
    quiet_mode: BOOLEAN

feature -- Web Settings
    web_port: INTEGER
    web_host: STRING

feature -- Commands
    validate
        -- Check all paths exist, populate validation_errors

    save (a_path: PATH)
        -- Persist to JSON file

    reload
        -- Re-read from file

feature -- Serialization
    to_json: JSON_OBJECT
    from_json (a_json: JSON_OBJECT)

invariant
    timeout_positive: timeout_seconds > 0
    history_positive: history_size > 0
    port_valid: web_port >= 1024 and web_port <= 65535
end
```

**CONFIG_DETECTOR Class:**

```eiffel
class CONFIG_DETECTOR
    -- Auto-detect EiffelStudio and Simple Eiffel paths

feature -- Detection
    detect_eiffel_compiler: detachable PATH
        -- Find ec.exe via:
        -- 1. ISE_EIFFEL environment variable
        -- 2. Common installation paths
        -- 3. PATH search

    detect_ise_library: detachable PATH
        -- Find ISE_LIBRARY via:
        -- 1. ISE_LIBRARY environment variable
        -- 2. Relative to detected compiler

    detect_simple_eiffel: detachable PATH
        -- Find SIMPLE_EIFFEL via:
        -- 1. SIMPLE_EIFFEL environment variable
        -- 2. Common paths (D:/prod, ~/simple-eiffel)

feature -- Platform-specific paths
    windows_search_paths: ARRAYED_LIST [PATH]
        do
            create Result.make (3)
            Result.extend (create {PATH}.make_from_string ("C:/Program Files/Eiffel Software"))
            Result.extend (create {PATH}.make_from_string ("C:/EiffelStudio"))
            Result.extend (create {PATH}.make_from_string (user_home + "/EiffelStudio"))
        end

    linux_search_paths: ARRAYED_LIST [PATH]
        do
            create Result.make (3)
            Result.extend (create {PATH}.make_from_string ("/opt/eiffelstudio"))
            Result.extend (create {PATH}.make_from_string ("/usr/local/eiffelstudio"))
            Result.extend (create {PATH}.make_from_string (user_home + "/eiffelstudio"))
        end

feature -- Full Detection
    detect_all: NOTEBOOK_CONFIG
        -- Create config with all auto-detected values
        local
            ec, ise, simple: detachable PATH
        do
            ec := detect_eiffel_compiler
            ise := detect_ise_library
            simple := detect_simple_eiffel

            create Result.make_with_defaults
            if attached ec as e then Result.set_eiffel_compiler (e) end
            if attached ise as i then Result.set_ise_library (i) end
            if attached simple as s then Result.set_simple_eiffel (s) end
        end
end
```

**CONFIG_WIZARD Class:**

```eiffel
class CONFIG_WIZARD
    -- Interactive first-run setup for CLI

feature -- Wizard Flow
    run: NOTEBOOK_CONFIG
        -- Interactive setup returning completed config
        do
            print_welcome
            Result := run_detection
            if not Result.is_valid then
                Result := prompt_for_missing (Result)
            end
            confirm_and_save (Result)
        ensure
            valid_config: Result.is_valid
        end

feature {NONE} -- Steps
    print_welcome
        do
            io.put_string ("╔═══════════════════════════════════════════════════════════╗%N")
            io.put_string ("║           Eiffel Notebook - First Run Setup               ║%N")
            io.put_string ("╚═══════════════════════════════════════════════════════════╝%N%N")
        end

    run_detection: NOTEBOOK_CONFIG
        local
            detector: CONFIG_DETECTOR
        do
            io.put_string ("Detecting EiffelStudio installation...%N")
            create detector
            Result := detector.detect_all

            if attached Result.eiffel_compiler as ec then
                io.put_string ("  ✓ Found compiler: " + ec.name + "%N")
            else
                io.put_string ("  ✗ Compiler not found%N")
            end

            if attached Result.simple_eiffel as se then
                io.put_string ("  ✓ Found Simple Eiffel: " + se.name + "%N")
            else
                io.put_string ("  ✗ Simple Eiffel not found%N")
            end
        end

    prompt_for_missing (a_config: NOTEBOOK_CONFIG): NOTEBOOK_CONFIG
        do
            Result := a_config
            if not attached Result.eiffel_compiler then
                io.put_string ("%NEnter path to EiffelStudio ec.exe: ")
                io.read_line
                Result.set_eiffel_compiler (create {PATH}.make_from_string (io.last_string))
            end
            if not attached Result.simple_eiffel then
                io.put_string ("Enter path to Simple Eiffel root (or press Enter to skip): ")
                io.read_line
                if not io.last_string.is_empty then
                    Result.set_simple_eiffel (create {PATH}.make_from_string (io.last_string))
                end
            end
        end

    confirm_and_save (a_config: NOTEBOOK_CONFIG)
        do
            io.put_string ("%NConfiguration complete! Save to ~/.eiffel_notebook/config.json? [Y/n] ")
            io.read_line
            if io.last_string.is_empty or io.last_string.item (1).as_lower = 'y' then
                a_config.save (default_config_path)
                io.put_string ("✓ Configuration saved.%N")
            end
        end
end
```

**Test file:** `testing/test_config.e`

```eiffel
test_config_defaults
    local
        config: NOTEBOOK_CONFIG
    do
        create config.make_with_defaults
        assert_equal ("default timeout", 30, config.timeout_seconds)
        assert_equal ("default port", 8080, config.web_port)
        assert_equal ("default host", "localhost", config.web_host)
    end

test_config_json_roundtrip
    local
        config, loaded: NOTEBOOK_CONFIG
        json: JSON_OBJECT
    do
        create config.make_with_defaults
        config.set_timeout_seconds (60)
        config.set_web_port (3000)
        json := config.to_json

        create loaded.make_from_json (json)
        assert_equal ("timeout preserved", 60, loaded.timeout_seconds)
        assert_equal ("port preserved", 3000, loaded.web_port)
    end

test_config_validation_missing_compiler
    local
        config: NOTEBOOK_CONFIG
    do
        create config.make_with_defaults
        -- Don't set eiffel_compiler
        config.validate
        assert ("not valid", not config.is_valid)
        assert ("has error", config.validation_errors.count > 0)
    end

test_detector_finds_env_var
    local
        detector: CONFIG_DETECTOR
        ec_path: detachable PATH
    do
        -- This test passes if ISE_EIFFEL env var is set
        create detector
        ec_path := detector.detect_eiffel_compiler
        if attached (create {EXECUTION_ENVIRONMENT}).get ("ISE_EIFFEL") then
            assert ("found compiler", ec_path /= Void)
        end
    end

test_config_file_save_load
    local
        config, loaded: NOTEBOOK_CONFIG
        test_path: PATH
    do
        create test_path.make_from_string (test_workspace + "/test_config.json")

        create config.make_with_defaults
        config.set_timeout_seconds (45)
        config.save (test_path)

        create loaded.make_from_file (test_path)
        assert_equal ("timeout loaded", 45, loaded.timeout_seconds)
    end
```

**Exit criteria:** Configuration loads, saves, validates. Auto-detection works when env vars present.

---

#### Phase 1.1: Data Structures

- [ ] `NOTEBOOK_CELL` - Single cell: id, code, output, status
- [ ] `NOTEBOOK` - Cell container with ordering
- [ ] `NOTEBOOK_STORAGE` - File I/O for .enb files
- [ ] `test_data_structures.e` - All tests passing

**Test file:** `testing/test_data_structures.e`

```eiffel
test_cell_creation
    local
        cell: NOTEBOOK_CELL
    do
        create cell.make ("cell_001", "code")
        cell.set_code ("x := 42")
        assert_equal ("code set", "x := 42", cell.code)
        assert_equal ("status idle", "idle", cell.status)
    end

test_notebook_json_roundtrip
    local
        nb, nb2: NOTEBOOK
        json: JSON_OBJECT
    do
        create nb.make ("test_notebook")
        nb.add_code_cell ("x := 42")
        nb.add_code_cell ("print(x)")
        json := nb.to_json
        create nb2.make_from_json (json)
        assert_equal ("cell count preserved", 2, nb2.cell_count)
        assert_equal ("code preserved", "x := 42", nb2.cells[1].code)
    end

test_notebook_storage_save_load
    local
        storage: NOTEBOOK_STORAGE
        nb, loaded: NOTEBOOK
    do
        create storage.make (test_workspace)
        create nb.make ("my_notebook")
        nb.add_code_cell ("x := 42")
        storage.save (nb, "test.enb")
        loaded := storage.load ("test.enb")
        assert_equal ("name preserved", "my_notebook", loaded.name)
    end
```

**Exit criteria:** All data structure tests pass. Notebooks persist correctly.

---

#### Phase 1.2: Code Generation

| Class | Purpose | Tests |
|-------|---------|-------|
| `ACCUMULATED_CLASS_GENERATOR` | Cells → Eiffel class | Valid class generation |
| `LINE_MAPPING` | Track generated line → cell line | Error mapping |

**Test file:** `testing/test_code_generation.e`

```eiffel
test_single_cell_generation
    local
        gen: ACCUMULATED_CLASS_GENERATOR
        nb: NOTEBOOK
        code: STRING
    do
        create nb.make ("test")
        nb.add_code_cell ("x := 42%Nprint(x)")
        create gen.make
        code := gen.generate_class (nb)

        -- Verify structure
        assert ("has class header", code.has_substring ("class ACCUMULATED_SESSION_"))
        assert ("has execute_cell_1", code.has_substring ("execute_cell_1"))
        assert ("has cell code", code.has_substring ("x := 42"))
    end

test_shared_variable_collection
    local
        gen: ACCUMULATED_CLASS_GENERATOR
        nb: NOTEBOOK
        code: STRING
    do
        create nb.make ("test")
        nb.add_code_cell ("shared x: INTEGER%Nx := 42")
        nb.add_code_cell ("print(x)")  -- uses x from cell 1
        create gen.make
        code := gen.generate_class (nb)

        -- x should be a class attribute, not local
        assert ("x is attribute", code.has_substring ("x: INTEGER"))
        assert ("not local in cell 2", not has_local_x_in_cell_2 (code))
    end

test_line_mapping
    local
        gen: ACCUMULATED_CLASS_GENERATOR
        nb: NOTEBOOK
        mapping: LINE_MAPPING
    do
        create nb.make ("test")
        nb.add_code_cell ("x := 42%Ny := x * 2")  -- 2 lines
        nb.add_code_cell ("print(y)")              -- 1 line
        create gen.make
        gen.generate_class (nb)
        mapping := gen.line_mapping

        -- Generated line 47 might map to cell 1, line 2
        assert_equal ("maps to cell 1", "cell_001", mapping.cell_id_for_line (47))
        assert_equal ("maps to line 2", 2, mapping.cell_line_for_line (47))
    end
```

**Exit criteria:** Generator produces valid Eiffel class structure. Line mapping works.

---

#### Phase 1.3: Compilation Integration (ec.exe)

| Class | Purpose | Tests |
|-------|---------|-------|
| `CELL_EXECUTOR` | Invoke ec.exe, run result | Real compilation |
| `COMPILATION_RESULT` | Compiler output/errors | Parse ec.exe output |
| `EXECUTION_RESULT` | Runtime output/timing | Capture stdout/stderr |
| `COMPILER_ERROR_PARSER` | Parse EiffelStudio errors | Extract line/message |

**Test file:** `testing/test_compilation.e`

**Critical:** These tests invoke real ec.exe. They prove the integration works.

```eiffel
test_simple_compilation_succeeds
    -- This test actually compiles Eiffel code!
    local
        executor: CELL_EXECUTOR
        nb: NOTEBOOK
        result: EXECUTION_RESULT
    do
        create nb.make ("test")
        nb.add_code_cell ("print(%"Hello from notebook!%N%")")

        create executor.make (test_workspace, ec_path)
        result := executor.execute_notebook (nb)

        assert ("compilation succeeded", result.compilation_succeeded)
        assert ("has output", result.stdout.has_substring ("Hello from notebook!"))
    end

test_compilation_with_variable
    local
        executor: CELL_EXECUTOR
        nb: NOTEBOOK
        result: EXECUTION_RESULT
    do
        create nb.make ("test")
        nb.add_code_cell ("x: INTEGER%Nx := 42")
        nb.add_code_cell ("print(x.out)")

        create executor.make (test_workspace, ec_path)
        result := executor.execute_notebook (nb)

        assert ("succeeded", result.compilation_succeeded)
        assert ("output is 42", result.stdout.has_substring ("42"))
    end

test_compilation_error_mapped_to_cell
    local
        executor: CELL_EXECUTOR
        nb: NOTEBOOK
        result: EXECUTION_RESULT
    do
        create nb.make ("test")
        nb.add_code_cell ("x := 42")                    -- OK
        nb.add_code_cell ("y := undefined_variable")   -- ERROR

        create executor.make (test_workspace, ec_path)
        result := executor.execute_notebook (nb)

        assert ("compilation failed", not result.compilation_succeeded)
        assert ("error in cell 2", result.errors.first.cell_id.is_equal ("cell_002"))
        assert ("mentions undefined", result.errors.first.message.has_substring ("undefined"))
    end

test_runtime_error_caught
    local
        executor: CELL_EXECUTOR
        nb: NOTEBOOK
        result: EXECUTION_RESULT
    do
        create nb.make ("test")
        nb.add_code_cell ("x: INTEGER%Nx := 1 // 0")  -- Division by zero

        create executor.make (test_workspace, ec_path)
        result := executor.execute_notebook (nb)

        assert ("compilation succeeded", result.compilation_succeeded)
        assert ("runtime failed", not result.execution_succeeded)
    end

test_timeout_protection
    local
        executor: CELL_EXECUTOR
        nb: NOTEBOOK
        result: EXECUTION_RESULT
    do
        create nb.make ("test")
        nb.add_code_cell ("from until False loop end")  -- Infinite loop

        create executor.make (test_workspace, ec_path)
        executor.set_timeout_seconds (2)
        result := executor.execute_notebook (nb)

        assert ("timed out", result.timed_out)
    end
```

**Exit criteria:**
- Real ec.exe compilation works
- Compiler errors are caught and mapped to cells
- Runtime errors are caught
- Infinite loops are terminated

---

#### Phase 1.4: Variable Tracking

| Class | Purpose | Tests |
|-------|---------|-------|
| `VARIABLE_TRACKER` | Track defined variables across cells | State introspection |
| `VARIABLE_INFO` | Name, type, value, source cell | Display in UI |

**Test file:** `testing/test_variable_tracking.e`

```eiffel
test_variable_extraction
    local
        tracker: VARIABLE_TRACKER
        nb: NOTEBOOK
        vars: ARRAYED_LIST [VARIABLE_INFO]
    do
        create nb.make ("test")
        nb.add_code_cell ("x: INTEGER%Nx := 42")
        nb.add_code_cell ("y: STRING%Ny := %"hello%"")

        create tracker.make
        vars := tracker.extract_variables (nb)

        assert_equal ("two vars", 2, vars.count)
        assert_equal ("x is INTEGER", "INTEGER", vars[1].type_name)
        assert_equal ("y is STRING", "STRING", vars[2].type_name)
    end

test_variable_change_detection
    local
        tracker: VARIABLE_TRACKER
        before, after: ARRAYED_LIST [VARIABLE_INFO]
        changes: ARRAYED_LIST [VARIABLE_CHANGE]
    do
        -- Simulate before/after state
        create before.make (1)
        before.extend (create {VARIABLE_INFO}.make ("x", "INTEGER", "42", "cell_001"))

        create after.make (2)
        after.extend (create {VARIABLE_INFO}.make ("x", "INTEGER", "84", "cell_002"))  -- Modified
        after.extend (create {VARIABLE_INFO}.make ("y", "STRING", "new", "cell_002"))  -- New

        create tracker.make
        changes := tracker.detect_changes (before, after)

        assert_equal ("two changes", 2, changes.count)
        assert ("x modified", changes[1].is_modified)
        assert ("y is new", changes[2].is_new)
    end
```

**Exit criteria:** Can track variables across cells, detect new/modified.

---

#### Phase 1.5: NOTEBOOK_ENGINE Facade

| Class | Purpose | Tests |
|-------|---------|-------|
| `NOTEBOOK_ENGINE` | Unified API for all operations | Integration tests |
| `SIMPLE_NOTEBOOK` | Public facade with type anchors | API usability |

**Test file:** `testing/test_engine.e`

```eiffel
test_engine_full_workflow
    -- Integration test: create, edit, execute, inspect
    local
        engine: NOTEBOOK_ENGINE
        session_id: STRING
        result: EXECUTION_RESULT
        vars: ARRAYED_LIST [VARIABLE_INFO]
    do
        create engine.make (test_workspace, ec_path)

        -- Create session
        session_id := engine.new_session ("My Analysis")

        -- Add cells
        engine.add_cell (session_id, "x: INTEGER%Nx := 42")
        engine.add_cell (session_id, "y := x * 2%Nprint(y.out)")

        -- Execute
        result := engine.execute_all (session_id)
        assert ("succeeded", result.execution_succeeded)
        assert ("output 84", result.stdout.has_substring ("84"))

        -- Inspect variables
        vars := engine.variables (session_id)
        assert_equal ("two vars", 2, vars.count)

        -- Save
        engine.save_session (session_id, "analysis.enb")

        -- Load in new engine
        create engine.make (test_workspace, ec_path)
        session_id := engine.load_session ("analysis.enb")
        assert_equal ("cells preserved", 2, engine.cell_count (session_id))
    end
```

**Exit criteria:** Complete workflow works end-to-end through engine API.

---

### Phase 1 Summary

| Component | Classes | Tests | LOC Est. |
|-----------|---------|-------|----------|
| Configuration | 3 | ~5 | ~400 |
| Data Structures | 3 | ~15 | ~550 |
| Code Generation | 2 | ~10 | ~500 |
| Compilation | 4 | ~12 | ~600 |
| Variable Tracking | 2 | ~6 | ~250 |
| Engine Facade | 2 | ~5 | ~300 |
| **Total Phase 1** | **16** | **~53** | **~2,600** |

**Phase 1 Exit Criteria:**
- [ ] All 53+ tests pass
- [ ] Configuration auto-detects EiffelStudio
- [ ] Real ec.exe integration proven
- [ ] Notebooks save/load correctly
- [ ] Compilation errors map to cells
- [ ] Variables tracked across cells
- [ ] Engine API is clean and usable

---

### Phase 2: CLI REPL (simple_repl + simple_repl_tests)

- [ ] **Phase 2 Complete**

**Prerequisite:** Phase 1 complete and proven.

**Goal:** Build enhanced REPL (Level 2) on proven engine.

#### Phase 2.1: Input Handling

- [ ] `REPL_INPUT_HANDLER` - Read lines, handle multi-line
- [ ] `SYNTAX_COMPLETENESS_CHECKER` - Detect incomplete code
- [ ] `COMMAND_PARSER` - Parse `:commands`
- [ ] Input handling tests passing

#### Phase 2.2: Output Formatting

- [ ] `REPL_OUTPUT_FORMATTER` - Format results, errors, vars
- [ ] `ERROR_FORMATTER` - Pretty-print mapped errors
- [ ] `VARIABLE_FORMATTER` - Format with +/~ markers
- [ ] Output formatting tests passing

#### Phase 2.3: Session Management

- [ ] `REPL_SESSION_MANAGER` - Auto-save, restore, history
- [ ] `COMMAND_HISTORY` - Up/down navigation
- [ ] Session management tests passing

#### Phase 2.4: REPL Controller

- [ ] `REPL_CONTROLLER` - Main loop, dispatch
- [ ] Integration tests passing
- [ ] Manual testing: full REPL workflow works

**Phase 2 LOC Estimate:** ~1,200

---

### Phase 3: Web UI (simple_nb_web + simple_nb_web_tests)

- [ ] **Phase 3 Complete**

**Prerequisite:** Phase 1 complete. (Phase 2 optional - can build in parallel)

**Goal:** Web notebook interface using HTMX/Alpine.js.

#### Phase 3.1: HTTP Handlers

- [ ] `NOTEBOOK_WEB_HANDLER` - Route dispatch
- [ ] `CELL_HANDLER` - Cell CRUD operations
- [ ] `EXECUTION_HANDLER` - Run cell/all
- [ ] HTTP handler tests passing

#### Phase 3.2: HTML Rendering

- [ ] `HTMX_RENDERER` - Generate partials
- [ ] `CELL_RENDERER` - Render single cell
- [ ] Rendering tests passing

#### Phase 3.3: Frontend Assets

- [ ] `index.html` - Main page
- [ ] `notebook.js` - Alpine components
- [ ] `notebook.css` - Styling
- [ ] `eiffel.js` - CodeMirror mode
- [ ] Manual testing: web UI works end-to-end

**Phase 3 LOC Estimate:** ~1,500 (Eiffel) + ~700 (Frontend)

---

### Phase 4: TUI IDE (simple_tui + simple_nb_tui)

- [ ] **Phase 4 Complete**

**Prerequisite:** Phase 1 complete. Phase 2 recommended (shares input handling patterns).

**Goal:** Full TUI IDE with split panes.

#### Phase 4.0: simple_tui Framework (Separate Library)

- [ ] `TUI_WINDOW` - Top-level container
- [ ] `TUI_PANE` - Individual pane with border
- [ ] `TUI_LAYOUT` - Split H/V
- [ ] `TUI_TEXT_BUFFER` - Scrollable text
- [ ] `TUI_INPUT_LINE` - Command input
- [ ] `TUI_TABLE` - Tabular display
- [ ] `TUI_EVENT_LOOP` - Main loop
- [ ] simple_tui framework tests passing

**simple_tui LOC Estimate:** ~2,000

#### Phase 4.1: Notebook TUI

- [ ] `NB_TUI_APP` - Main application
- [ ] `CELLS_PANE` - Cell list display
- [ ] `VARIABLES_PANE` - Variable table
- [ ] `OUTPUT_PANE` - Stdout display
- [ ] `ERRORS_PANE` - Error display
- [ ] `COMMAND_PANE` - Input prompt
- [ ] TUI notebook tests passing
- [ ] Manual testing: TUI IDE works end-to-end

**simple_nb_tui LOC Estimate:** ~1,000

---

### Phase 5: VS Code Integration (simple_nb_vsc) - Future

- [ ] **Phase 5 Complete** (Deferred)

**Prerequisite:** Phases 1-2 complete. simple_lsp operational.

**Goal:** VS Code notebook support via LSP extension.

#### Options (to be decided):
- [ ] Option 1: Extend simple_lsp with notebook methods
- [ ] Option 2: Standalone bridge process
- [ ] Option 3: VS Code Notebook API integration

**Deferred** until core platform stable.

---

### Phase 6: Full Interpreter - Future

- [ ] **Phase 6 Complete** (Deferred)

After Phases 1-4 ship, replace accumulated class approach with true interpreter.

**Approach Options:**

1. **Custom Stack VM** (Recommended)
   - Parse with simple_eiffel_parser
   - Generate custom bytecode
   - Implement stack interpreter
   - ~3,000 LOC

2. **AST Interpreter**
   - Direct tree walking
   - Simpler but slower
   - ~2,000 LOC

3. **EiffelStudio Bytecode**
   - Hook into existing VM
   - Complex integration
   - ~2,500 LOC

---

## Implementation Timeline

```
Phase 1: simple_notebook core     ████████░░░░░░░░░░░░  ~2,200 LOC
         └─ simple_notebook_tests (proves ec.exe integration)

Phase 2: simple_repl              ░░░░░░░░████░░░░░░░░  ~1,200 LOC
         └─ simple_repl_tests

Phase 3: simple_nb_web            ░░░░░░░░░░░░████░░░░  ~2,200 LOC
         └─ simple_nb_web_tests

Phase 4: simple_tui framework     ░░░░░░░░░░░░░░░░████  ~2,000 LOC
         simple_nb_tui            ░░░░░░░░░░░░░░░░░░██  ~1,000 LOC
         └─ tests for both

Phase 5: simple_nb_vsc            ░░░░░░░░░░░░░░░░░░░░  Future
Phase 6: Full interpreter         ░░░░░░░░░░░░░░░░░░░░  Future
```

**Total LOC (Phases 1-4):** ~8,600

---

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Compilation too slow | Medium | High | Cache compiled binaries, incremental compilation |
| Error messages hard to map | Medium | Medium | Line number tracking in generator |
| Complex Eiffel breaks generator | Low | Medium | Start simple, expand supported syntax |
| HTMX complexity | Low | Low | Use proven patterns from other Simple libs |
| CodeMirror Eiffel mode incomplete | Medium | Low | Enhance as needed, basic highlighting sufficient |

---

## Success Criteria

### Phase 1 MVP

- [ ] Create notebook with 5+ cells
- [ ] Execute cells individually and run-all
- [ ] See output/errors inline
- [ ] Cells persist across browser refresh
- [ ] Sub-second UI response for edits
- [ ] < 10 second full compilation + execution
- [ ] Works on Windows (Linux/macOS future)

### Quality Gates

- [ ] All tests pass
- [ ] No compiler warnings
- [ ] DBC contracts on all features
- [ ] SCOOP compatible
- [ ] Documentation generated

---

## Appendix A: Eiffel Syntax Highlighting for CodeMirror

```javascript
// codemirror/mode/eiffel.js
CodeMirror.defineMode("eiffel", function() {
    var keywords = /^(across|agent|alias|all|and|as|assign|attached|attribute|check|class|convert|create|Current|debug|deferred|detachable|do|else|elseif|end|ensure|expanded|export|external|False|feature|from|frozen|if|implies|indexing|inherit|inspect|invariant|like|local|loop|not|note|obsolete|old|once|only|or|Precursor|redefine|rename|require|rescue|Result|retry|select|separate|some|then|True|TUPLE|undefine|until|variant|Void|when|xor)\b/;
    var types = /^(ANY|ARRAY|BOOLEAN|CHARACTER|COMPARABLE|DOUBLE|HASHABLE|INTEGER|ITERABLE|ITERATION_CURSOR|LINKED_LIST|LIST|NUMERIC|POINTER|REAL|SPECIAL|STRING|STRING_32)\b/;
    var builtins = /^(io|print|put_string|put_integer|put_new_line|out|default_create|make|make_empty|make_from_string)\b/;

    return {
        startState: function() {
            return {inString: false, inComment: false};
        },
        token: function(stream, state) {
            if (state.inComment) {
                if (stream.match("*/")) {
                    state.inComment = false;
                    return "comment";
                }
                stream.next();
                return "comment";
            }

            if (stream.match("--")) {
                stream.skipToEnd();
                return "comment";
            }

            if (stream.match("/*")) {
                state.inComment = true;
                return "comment";
            }

            if (stream.match('"')) {
                while (!stream.eol()) {
                    if (stream.next() === '"' && stream.peek() !== '"') break;
                }
                return "string";
            }

            if (stream.match("'")) {
                stream.next();
                stream.match("'");
                return "string";
            }

            if (stream.match(/^[0-9]+(\.[0-9]+)?/)) {
                return "number";
            }

            if (stream.match(keywords)) {
                return "keyword";
            }

            if (stream.match(types)) {
                return "type";
            }

            if (stream.match(builtins)) {
                return "builtin";
            }

            if (stream.match(/^[A-Z][A-Z0-9_]*/)) {
                return "type";
            }

            if (stream.match(/^[a-z_][a-z0-9_]*/)) {
                return "variable";
            }

            stream.next();
            return null;
        }
    };
});

CodeMirror.defineMIME("text/x-eiffel", "eiffel");
```

---

## Appendix B: Sample Notebook JSON Format

```json
{
    "id": "nb_20251218_143052",
    "name": "Fibonacci Analysis",
    "created_at": "2025-12-18T14:30:52Z",
    "modified_at": "2025-12-18T14:45:30Z",
    "cells": [
        {
            "id": "cell_001",
            "cell_type": "markdown",
            "order": 1,
            "code": "# Fibonacci Sequence\n\nLet's compute Fibonacci numbers.",
            "output": null,
            "error": null,
            "status": "idle",
            "execution_time_ms": 0
        },
        {
            "id": "cell_002",
            "cell_type": "code",
            "order": 2,
            "code": "shared fib: ARRAY [INTEGER]\n\ncreate fib.make_filled (0, 0, 10)\nfib[0] := 0\nfib[1] := 1\nacross 2 |..| 10 as i loop\n    fib[i.item] := fib[i.item - 1] + fib[i.item - 2]\nend",
            "output": null,
            "error": null,
            "status": "success",
            "execution_time_ms": 2341
        },
        {
            "id": "cell_003",
            "cell_type": "code",
            "order": 3,
            "code": "-- Print results\nacross fib as f loop\n    io.put_integer (f.item)\n    io.put_string (\" \")\nend\nio.put_new_line",
            "output": "0 1 1 2 3 5 8 13 21 34 55 \n",
            "error": null,
            "status": "success",
            "execution_time_ms": 1823
        }
    ]
}
```

---

**Document Version:** 1.0
**Last Updated:** 2025-12-18
**Author:** Claude (with Larry)
