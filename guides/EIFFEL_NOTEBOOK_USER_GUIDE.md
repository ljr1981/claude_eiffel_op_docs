# Eiffel Notebook User Guide

**Version 1.0.0-alpha.20** | December 2025

## Table of Contents

1. [Introduction](#introduction)
2. [Installation](#installation)
   - [Windows](#windows)
   - [Linux/WSL2](#linuxwsl2)
3. [Quick Start](#quick-start)
4. [Writing Code](#writing-code)
5. [Cell Types](#cell-types)
6. [Commands Reference](#commands-reference)
7. [Troubleshooting](#troubleshooting)

---

## Introduction

**Eiffel Notebook** brings interactive programming to Eiffel. Write code in cells, execute them, and see results immediately. Features real-time streaming compiler output, automatic EiffelStudio detection, and DBC trace logging.

Part of the [Simple Eiffel](https://github.com/simple-eiffel) ecosystem.

---

## Installation

### Windows

1. Download `eiffel_notebook_setup_1.0.0-alpha.20.exe`
2. Run the installer with administrator privileges
3. The CLI is added to PATH automatically
4. EiffelStudio is auto-detected

After installation, open a command prompt and run:
```
eiffel_notebook
```

### Linux/WSL2

**Prerequisites:**

1. Install build tools:
```bash
sudo apt update
sudo apt install build-essential gcc make
```

2. Install EiffelStudio from https://www.eiffel.com/eiffelstudio/download/

3. **Set environment variables** (add to `~/.bashrc` for persistence):
```bash
# EiffelStudio configuration (REQUIRED)
export ISE_EIFFEL=$HOME/Eiffel_25.02
export ISE_PLATFORM=linux-x86-64
export ISE_LIBRARY=$ISE_EIFFEL
export PATH=$ISE_EIFFEL/studio/spec/$ISE_PLATFORM/bin:$PATH

# Simple Eiffel ecosystem
export SIMPLE_EIFFEL=$HOME/simple_eiffel
```

> **CRITICAL:** `ISE_LIBRARY` must be set or you will get:
> `eif_langinfo.h: No such file or directory`

**Running the notebook:**
```bash
# Apply environment (or restart terminal after editing ~/.bashrc)
source ~/.bashrc

# Run notebook
~/simple_eiffel/simple_notebook/EIFGENs/notebook_cli/W_code/simple_notebook
```

**WSL2 Notes:**
- WSL2 works identically to native Linux
- Tested on Ubuntu 22.04 under WSL2
- Make sure all environment variables are set before running

---

## Quick Start

```
Eiffel Notebook 1.0.0-alpha.20
Type Eiffel code to execute. Type -help for commands.

e[1]> name: STRING := "World"
...
e[1] Output:

e[2]> print ("Hello, " + name + "!")
...
e[2] Output:
Hello, World!

e[3]> x: INTEGER := 21
...
e[3] Output:

e[4]> x * 2
...
e[4] Output:
42

e[5]> -quit
Goodbye!
```

---

## Writing Code

### Basic Syntax

Write standard Eiffel code:

```
e[1]> x: INTEGER           -- Declare attribute
e[2]> x := 42              -- Assign value  
e[3]> x * 2                -- Expression (printed)
```

### Multi-line Input

After typing code, press Enter to get continuation prompt (`...`).
Press Enter on empty line to execute.

```
e[1]> greet (name: STRING)
...       do
...           print ("Hello, " + name)
...       end
...
```

### Design by Contract

```
e[1]> safe_divide (a, b: INTEGER): INTEGER
...       require
...           b_not_zero: b /= 0
...       do
...           Result := a // b
...       ensure
...           positive_if_same_sign: (a > 0 and b > 0) implies Result >= 0
...       end
...
```

---

## Cell Types

Cells are automatically classified by their content:

| Content | Type | Action |
|---------|------|--------|
| `x: INTEGER` | Attribute | Added to class |
| `greet do ... end` | Routine | Added to class |
| `x := 42` | Instruction | Executed |
| `x * 2` | Expression | Printed |

---

## Commands Reference

### Session

| Command | Alias | Description |
|---------|-------|-------------|
| `-help` | `-h` | Show help |
| `-quit` | `-q` | Exit |
| `-clear` | `-c` | Clear all cells |

### Cell Management

| Command | Alias | Description |
|---------|-------|-------------|
| `-cells` | | List all cells |
| `-show N` | `-s N` | Show cell N |
| `-edit N` | `-e N` | Edit cell N |
| `-d N` | | Delete cell N |

### Execution

| Command | Description |
|---------|-------------|
| `-run` | Re-execute all cells |
| `-compile verbose` | Stream compiler output |
| `-compile silent` | Hide compiler output |

### Inspection

| Command | Description |
|---------|-------------|
| `-vars` | Show variables |
| `-class` | Show generated class |
| `-debug` | Show classifications |

---

## Troubleshooting

### Failed to start compiler
**Cause:** EiffelStudio not found.  
**Fix:** Set `ISE_EIFFEL` environment variable or create config.json.

### eif_langinfo.h: No such file or directory (Linux)
**Cause:** `ISE_LIBRARY` not set.  
**Fix:** Add `export ISE_LIBRARY=$ISE_EIFFEL` to your environment.

### Unknown identifier
**Cause:** Variable not declared.  
**Fix:** Declare variables first: `x: INTEGER` then `x := 42`

### Compilation hangs
**Windows:** Kill `notebook_session.exe` in Task Manager.  
**Linux:** Run `pkill -f notebook_session`

### Session Logs
Check the log file path shown at startup for detailed debugging information.

---

## Platform Support

| Platform | Status | Notes |
|----------|--------|-------|
| Windows 10/11 | Tested | Auto-detect EiffelStudio |
| Ubuntu 22.04 | Tested | Requires env vars |
| WSL2 Ubuntu | Tested | Same as native Linux |
| macOS | Untested | Should work with proper setup |

---

## Acknowledgments

- **Eric Bezault** (Gobo): Cell classification design
- **Javier Velilla**: Original project idea

---

*Built with Design by Contract. Powered by EiffelStudio.*
