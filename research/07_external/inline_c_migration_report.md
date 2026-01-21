# Inline C Migration Report

**Date:** December 8, 2025
**Context:** Inlining C code into Eiffel classes

---

## Summary

We attempted to migrate external C code from `Clib/` folders into Eiffel inline C externals across the simple_* ecosystem. This eliminates the need for pre-compiled `.obj` files and simplifies distribution.

**Results:**
- 3 libraries successfully migrated
- 6 libraries cannot be migrated (architectural limitations)

---

## Successes

The following libraries were successfully converted from external C to inline C:

### 1. simple_env
- **Functions:** `c_se_get_env`, `c_se_set_env`, `c_se_unset_env`, `c_se_expand_env`
- **Pattern:** Stateless Win32 API wrappers (`GetEnvironmentVariableA`, `SetEnvironmentVariableA`, `ExpandEnvironmentStringsA`)
- **Status:** Compiles and tests pass

### 2. simple_clipboard
- **Functions:** `c_sc_open`, `c_sc_close`, `c_sc_empty`, `c_sc_set_text`, `c_sc_get_text`, `c_sc_has_text`
- **Pattern:** Stateless Win32 clipboard API calls
- **Status:** Compiles and tests pass

### 3. simple_console
- **Functions:** 15 inline C functions for color, cursor, and screen manipulation
- **Pattern:** Stateless Win32 console API calls (`SetConsoleTextAttribute`, `SetConsoleCursorPosition`, `FillConsoleOutputCharacterA`, etc.)
- **Status:** Compiles and tests pass

### Common Pattern for Success

All successful migrations share these characteristics:
1. **No structs** passed between Eiffel and C
2. **No static state** maintained across calls
3. **Single Win32 API calls** or simple sequences of calls
4. **Local variables only** within each function

Example of inline C that works:

```eiffel
c_se_set_env (a_name, a_value: POINTER): INTEGER
    external
        "C inline use <windows.h>"
    alias
        "return SetEnvironmentVariableA((const char*)$a_name, (const char*)$a_value) ? 1 : 0;"
    end
```

---

## Failures

### 1. simple_ipc (Named Pipes)

**Problem:** The C API defines and uses a struct:

```c
typedef struct {
    HANDLE handle;
    char* name;
    int is_server;
    int is_connected;
} sipc_pipe;
```

Functions pass `sipc_pipe*` pointers. When inlined, each `external "C inline"` block is compiled independently. The struct definition in one block is not visible to other blocks.

**Attempted workaround:** Preprocessor guards:
```c
#ifndef SIPC_PIPE_STRUCT_DEFINED
#define SIPC_PIPE_STRUCT_DEFINED
typedef struct { ... } sipc_pipe;
#endif
```

**Result:** Failed. C compiler error: `sipc_pipe: undeclared identifier`. The preprocessor state does not persist across inline blocks.

**Conclusion:** Must keep external C file for struct-based APIs.

### 2. simple_system

**Problem:** Uses static state for OS version caching:

```c
static RTL_OSVERSIONINFOW* get_os_version_info(void) {
    static RTL_OSVERSIONINFOW osvi = {0};
    static int initialized = 0;
    // ... lazy initialization ...
}
```

Multiple inline functions call this helper. Since each inline block is compiled independently, they cannot share the static helper function or its static variables.

**Conclusion:** Must keep external C file for APIs with static state.

### 3-6. Other struct-based libraries

The following libraries also have struct-based C APIs and cannot be inlined:
- **simple_mmap** - `smmap_handle` struct
- **simple_process** - `sproc_handle` struct
- **simple_registry** - `HKEY` handle management with struct-like state
- **simple_watcher** - `swatcher` struct

---

## Open Questions for the Eiffel Community

### Question 1: Struct visibility across inline blocks

Is there a way to make a C struct definition visible to all inline C externals in a class?

**Use case:** Multiple functions need to work with the same struct type:
```eiffel
feature {NONE} -- C externals

    c_create_pipe: POINTER
        external "C inline use <windows.h>"
        alias "[
            sipc_pipe* p = malloc(sizeof(sipc_pipe));  -- needs struct def
            return p;
        ]"
        end

    c_connect (a_pipe: POINTER): INTEGER
        external "C inline use <windows.h>"
        alias "[
            sipc_pipe* p = (sipc_pipe*)$a_pipe;  -- needs same struct def
            return p->is_connected;
        ]"
        end
```

Currently fails because each block is compiled independently.

### Question 2: Static state across inline blocks

Is there a way to share static helper functions or static variables across inline C externals?

**Use case:** Lazy initialization / singleton pattern:
```c
static RTL_OSVERSIONINFOW* get_os_version_info(void) {
    static RTL_OSVERSIONINFOW osvi = {0};
    static int initialized = 0;
    if (!initialized) { /* ... */ initialized = 1; }
    return &osvi;
}
```

Currently impossible because each inline block has its own compilation scope.

### Question 3: EiffelStudio inline C compilation model

What exactly happens during EiffelStudio's C code generation for inline externals?

- Are all inline blocks from a class concatenated?
- Can we add a "shared header" that gets included before all inline blocks?
- Is there an ECF option to inject shared declarations?

### Question 4: Alternative approaches

Given the limitations, what do experienced Eiffel developers recommend?

**Option A:** Keep external C files for complex APIs (current approach)
**Option B:** Redesign the C API to be stateless (significant work)
**Option C:** Use MANAGED_POINTER with raw struct bytes (loses type safety)
**Option D:** Something else?

---

## Recommendations

### For libraries that CAN be inlined:
1. Use inline C when: stateless functions, no structs, simple Win32 calls
2. Benefits: No .obj distribution, automatic recompilation, simpler packaging

### For libraries that CANNOT be inlined:
1. Keep external C files when: structs, static state, complex multi-function interactions
2. Document the dependency clearly in README
3. Consider providing platform-specific pre-compiled objects

### Summary Table

| Library | Status | Reason |
|---------|--------|--------|
| simple_env | Inlined | Stateless API |
| simple_clipboard | Inlined | Stateless API |
| simple_console | Inlined | Stateless API |
| simple_ipc | Keep Clib | Struct-based API |
| simple_mmap | Keep Clib | Struct-based API |
| simple_process | Keep Clib | Struct-based API |
| simple_registry | Keep Clib | Handle-based state |
| simple_system | Keep Clib | Static state pattern |
| simple_watcher | Keep Clib | Struct-based API |

---

## Next Steps

1. Wait for community feedback on Questions 1-4
2. If answers enable more inlining, proceed with remaining libraries
3. If not, document the pattern clearly for future reference

---

*Report prepared for the Eiffel community discussion.*
