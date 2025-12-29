# simple_notebook Phase 2 Design

**Version:** 1.0  
**Date:** December 2025  
**Status:** Approved for Implementation

## Overview

Phase 2 enhances simple_notebook with three major features:
1. Variable change markers
2. Session persistence (Live Notebook model)
3. Command history

---

## 1. Variable Change Markers

### Description
After each cell execution, display which variables were added, modified, or removed.

### Output Format
```
e[3]> x := x + 10
...
e[3] Output:
  x: 42 → 52  (modified)
  y: 10       (new)
  z: --       (removed)
```

### Visual Indicators
| State | Format | Example |
|-------|--------|---------|
| New | `name: value (new)` | `y: 10 (new)` |
| Modified | `name: old → new (modified)` | `x: 42 → 52 (modified)` |
| Removed | `name: -- (removed)` | `z: -- (removed)` |

### Implementation Notes
- Compare variable state before/after cell execution
- Use existing VARIABLE_TRACKER infrastructure
- Only show changes, not all variables (use `-vars` for full list)

---

## 2. Session Persistence (Live Notebook Model)

### Concept
The session IS a file. Auto-saves continuously. Blends Jupyter's file-based model with REPL-style commands.

### File Format
- Extension: `.enb` (Eiffel Notebook)
- Format: JSON (same as existing notebook persistence)
- Default location: `~/.eiffel_notebook/notebooks/`

### Startup Modes
```bash
eiffel_notebook                    # resumes default.enb
eiffel_notebook mywork             # opens/creates mywork.enb  
eiffel_notebook --scratch          # ephemeral, no auto-save
```

### Runtime Commands

| Command | Alias | Description |
|---------|-------|-------------|
| `-save` | | Save current notebook |
| `-save name` | | Save As (copy to name, switch to it) |
| `-open name` | `-restore name` | Open/restore notebook |
| `-new` | | Fresh notebook |
| `-notebooks` | `-list` | List available notebooks |

### Auto-Save Behavior
- Auto-save after every cell execution
- Auto-save on `-quit`
- Dirty indicator in prompt: `e[3]*>` (unsaved) vs `e[3]>` (clean)
- Crash recovery from last auto-save

### Notebook Metadata
```json
{
  "name": "mywork",
  "created": "2025-12-20T15:30:00Z",
  "modified": "2025-12-20T16:45:00Z",
  "version": "1.0.0-alpha.21",
  "cells": [...],
  "history": [...]
}
```

---

## 3. Command History

### Features
- Up/Down arrows to navigate previous inputs
- `Ctrl+R` to search history (if terminal supports)
- Persistent across sessions
- `!N` to re-execute cell N

### Commands

| Input | Action |
|-------|--------|
| `↑` / `↓` | Navigate history |
| `!3` | Re-execute cell 3 |
| `!!` | Re-execute last cell |
| `-history` | Show recent commands |
| `-history N` | Show last N commands |

### Storage
- History file: `~/.eiffel_notebook/history`
- Format: One entry per line with timestamp
- Max entries: 1000 (configurable)

---

## New Commands Summary

### Session Management
```
-save [name]           Save current / Save As
-open name             Open notebook (alias: -restore)
-restore name          Open notebook (alias: -open)
-new                   Fresh notebook
-notebooks             List notebooks (alias: -list)
-list                  List notebooks (alias: -notebooks)
```

### History
```
-history [N]           Show command history
!N                     Re-execute cell N
!!                     Re-execute last cell
```

---

## Implementation Order

1. **Variable change markers** - Enhance VARIABLE_TRACKER, update CLI output
2. **Session persistence** - Add notebook file management, auto-save
3. **Command history** - Add history storage, navigation, `!N` syntax

---

## Precedents

| Feature | Precedent |
|---------|-----------|
| Variable markers | Swift Playgrounds, Observable |
| Session persistence | Jupyter, Smalltalk image |
| Command history | IPython, readline, Fish shell |

---

## Acknowledgments

Design discussion: Larry Rix, December 2025
