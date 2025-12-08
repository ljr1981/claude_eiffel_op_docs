# Eric's Feedback on simple_* Win32 Libraries

Date: December 2025

## Remarks

### 1. Cross-Platform Support
The simple_* Win32 libraries should provide implementations for Linux and macOS as well, not just Windows.

**Action Item:** When time permits, add platform-specific implementations for:
- Linux
- macOS

### 2. Inline C Code Instead of Separate .c Files
Don't put the C code in separate C files (which need separate compilation). Instead, include the C code directly in external routines using `C inline`.

**Example - Current approach (NOT recommended):**
```eiffel
c_sc_set_color (a_color: INTEGER): INTEGER
    external
        "C | %"simple_console.h%""
    alias
        "sc_set_color"
    end
```
This requires compiling `simple_console.c` into `simple_console.obj`.

**Recommended approach (inline C):**
```eiffel
c_sc_set_color (a_color: INTEGER): INTEGER
    external
        "C inline use %"simple_console.h%""
    alias
        "[
            save_default_color();
            return SetConsoleTextAttribute(get_stdout_handle(), (WORD)$a_color) ? 1 : 0;
        ]"
    end
```

**Benefits:**
- No separate `.obj` file needed
- Simpler build process
- All code in one place

## Affected Libraries
Libraries that may need this refactoring:
- simple_console
- simple_clipboard
- simple_registry
- simple_mmap
- simple_ipc
- simple_watcher
- simple_system
- simple_env

## Status
**Deferred** - Circle back to address when time permits.
