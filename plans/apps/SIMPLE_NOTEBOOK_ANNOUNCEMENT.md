# Announcing Eiffel Notebook - Interactive Eiffel Programming

**Version 1.0.0-alpha.20** | December 2025

## The Vision

**Eiffel Notebook** brings the interactive notebook experience to the Eiffel programming language. Inspired by Jupyter notebooks and REPLs, it lets you write, execute, and iterate on Eiffel code interactively - perfect for learning, prototyping, and exploring the Simple Eiffel ecosystem.

---

## What We've Built (Alpha.20)

### Core Execution Engine
- **Accumulated class generation** - Cells compile into a single class that preserves state
- **Melt-based compilation** - Uses EiffelStudio's fast incremental compilation
- **Real-time streaming output** - Watch ec.exe compile in real-time, not just after
- **Auto-detect EiffelStudio** - Works out of the box via ISE_EIFFEL detection

### Interactive CLI
- **Smart multi-line input** - Automatic continuation for `do`, `if`, `class`, etc.
- **Syntax completeness detection** - Knows when your code is ready to execute
- **Dash commands** - `-help`, `-vars`, `-list`, `-save`, `-compile verbose`
- **Session logging** - Full DBC trace logging with caller/supplier contracts

### Variable Tracking
- **Cross-cell state** - Variables persist between cell executions
- **Shared variable detection** - Identifies variables meant to span cells
- **Type tracking** - Remembers variable types and defining cells

### Error Handling
- **Line mapping** - Maps compiler errors back to original cell lines
- **Underline formatting** - Visual indication of error location
- **Error classification** - Distinguishes syntax, type, and validity errors

### Persistence
- **JSON notebook format** - Save/load complete notebook state
- **Config file support** - Customize paths, timeouts, preferences
- **Workspace management** - Clean temporary files between runs

---

## Technical Highlights

| Component | Implementation |
|-----------|---------------|
| Compiler integration | SIMPLE_ASYNC_PROCESS for streaming |
| Code generation | ACCUMULATED_CLASS_GENERATOR |
| Variable analysis | VARIABLE_TRACKER with pattern matching |
| Error parsing | COMPILER_ERROR_PARSER for EiffelStudio output |
| Storage | JSON via SIMPLE_JSON |
| Process control | SIMPLE_PROCESS + SIMPLE_ASYNC_PROCESS |

**Test coverage:** 80 tests passing across all components

---

## What's Ahead

### Phase 2: Enhanced UX
- Variable change markers (highlight what changed between executions)
- Session persistence (resume exactly where you left off)
- Command history with search (up-arrow, Ctrl+R)
- Cell editing (modify previous cells)

### Phase 3: Web Interface
- Browser-based notebook UI
- Markdown cell rendering
- Syntax highlighting
- Output visualization

### Phase 4: Advanced Features
- Export to standalone Eiffel project
- Import existing .e files as cells
- Library exploration (introspect Simple Eiffel APIs)
- Notebook sharing format

---

## Get Started

**Download:** `eiffel_notebook_setup_1.0.0-alpha.20.exe`

**Repository:** [github.com/simple-eiffel/simple_notebook](https://github.com/simple-eiffel/simple_notebook)

```
Eiffel Notebook 1.0.0-alpha.20
Type Eiffel code to execute. Type -help for commands.

e[1]> name: STRING := "Eiffel"
e[2]> print ("Hello, " + name + "!")
Hello, Eiffel!
e[3]> -compile verbose
Compile mode: verbose (shows compiler output)
e[3]> x: INTEGER := 42
Eiffel Compilation Manager
Version 25.02.9.8732 - win64
Degree 6: Examining System
...
C compilation completed
e[4]>
```

---

## Part of Simple Eiffel

Eiffel Notebook is built on the **Simple Eiffel** ecosystem - 60+ libraries providing modern, DBC-compliant Eiffel components. It uses:
- `simple_process` - Async process execution with streaming
- `simple_json` - Notebook persistence
- `simple_file` - File operations
- `simple_datetime` - Timing and timestamps
- `simple_testing` - Test framework

---

*Built with Design by Contract. Powered by EiffelStudio.*
