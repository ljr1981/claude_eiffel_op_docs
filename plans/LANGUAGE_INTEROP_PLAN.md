# Language Interoperability Libraries - Implementation Plan

**Date**: 2024-12-26
**Target**: 2024-12-27 (Tomorrow)
**Status**: Planned

## Objective

Build simple_* wrapper libraries for cross-language FFI, enabling Eiffel programs to leverage libraries from Python, Rust, Lua, Zig, and optionally D and Fortran.

## Priority Order

| # | Library | Feasibility | Value | Effort |
|---|---------|-------------|-------|--------|
| 1 | simple_python | HIGH | ML/AI ecosystem | Medium |
| 2 | simple_rust | HIGH | Performance libs | Low-Medium |
| 3 | simple_lua | HIGH | Tiny scripting | Low |
| 4 | simple_zig | HIGH | Modern C | Low |
| 5 | simple_d | MEDIUM | Systems | Low (if time) |
| 6 | simple_fortran | MEDIUM | Scientific | Low (if time) |

## Phase 1: simple_python

### Architecture

```
SIMPLE_PYTHON (Facade)
├── PY_INTERPRETER (singleton, lifecycle)
├── PY_MODULE (import, attribute access)
├── PY_OBJECT (reference-counted base wrapper)
├── PY_CALLABLE (function/method calls)
├── PY_INTEGER, PY_FLOAT, PY_STRING (primitives)
├── PY_LIST, PY_DICT, PY_TUPLE (collections)
└── PY_EXCEPTION (error handling)
```

### Key Classes

| Class | Purpose |
|-------|---------|
| SIMPLE_PYTHON | Facade - `make`, `import_module`, `run_script` |
| PY_INTERPRETER | Singleton - `Py_Initialize`, `Py_Finalize` |
| PY_OBJECT | Base - reference counting, `incref`, `decref` |
| PY_MODULE | Module ops - `get_attr`, `call_method` |
| PY_CALLABLE | Calls - `call`, `call_with_args` |

### C Externals Required

```eiffel
c_py_initialize external "C inline use <Python.h>" alias "Py_Initialize();"
c_py_finalize external "C inline use <Python.h>" alias "Py_Finalize();"
c_py_import (name: POINTER): POINTER alias "return PyImport_ImportModule($name);"
c_py_get_attr (obj, name: POINTER): POINTER alias "return PyObject_GetAttrString($obj, $name);"
c_py_call (callable, args: POINTER): POINTER alias "return PyObject_CallObject($callable, $args);"
c_py_incref (obj: POINTER) alias "Py_INCREF($obj);"
c_py_decref (obj: POINTER) alias "Py_DECREF($obj);"
```

### ECF Setup

```xml
<external_include location="$(PYTHON_HOME)/include"/>
<external_library location="$(PYTHON_HOME)/libs/python311.lib">
    <condition><platform value="windows"/></condition>
</external_library>
```

### Tests

1. Initialize/finalize interpreter
2. Import standard module (math, json)
3. Call function with primitives
4. Get/set attributes
5. Handle Python exceptions
6. Reference counting (no leaks)

---

## Phase 2: simple_rust

### Architecture

```
SIMPLE_RUST (Facade)
├── RUST_LIBRARY (dynamic loading)
├── RUST_FUNCTION (function handle)
├── RUST_STRING (owned string wrapper)
├── RUST_BUFFER (byte buffer wrapper)
└── RUST_RESULT (Result<T,E> pattern)
```

### Key Classes

| Class | Purpose |
|-------|---------|
| SIMPLE_RUST | Facade - `load_library`, `get_function` |
| RUST_LIBRARY | LoadLibrary/dlopen wrapper |
| RUST_FUNCTION | GetProcAddress/dlsym wrapper |
| RUST_STRING | Owned string with `rust_free` |

### C Externals Required

```eiffel
c_load_library (path: POINTER): POINTER
    alias "return LoadLibraryA($path);"  -- Windows

c_get_proc (lib, name: POINTER): POINTER
    alias "return GetProcAddress($lib, $name);"

c_free_library (lib: POINTER)
    alias "FreeLibrary($lib);"
```

### Rust Side Requirements

All Rust FFI functions must:
```rust
#[no_mangle]
pub extern "C" fn my_function(...) -> ... {
    std::panic::catch_unwind(|| {
        // implementation
    }).unwrap_or(default_value)
}
```

### Tests

1. Load Rust DLL
2. Call simple function (add integers)
3. String passing (with cleanup)
4. Error handling (null returns)

---

## Phase 3: simple_lua

### Architecture

```
SIMPLE_LUA (Facade)
├── LUA_STATE (interpreter state)
├── LUA_VALUE (stack value wrapper)
├── LUA_TABLE (table operations)
├── LUA_FUNCTION (Lua function calls)
└── LUA_SCRIPT (file/string execution)
```

### Key Classes

| Class | Purpose |
|-------|---------|
| SIMPLE_LUA | Facade - `make`, `do_string`, `do_file` |
| LUA_STATE | State lifecycle, stack operations |
| LUA_TABLE | `get_field`, `set_field`, iteration |
| LUA_FUNCTION | `call`, register Eiffel functions |

### C Externals Required

```eiffel
c_lua_newstate: POINTER alias "return luaL_newstate();"
c_lua_close (L: POINTER) alias "lua_close($L);"
c_lua_dostring (L: POINTER; s: POINTER): INTEGER alias "return luaL_dostring($L, $s);"
c_lua_dofile (L: POINTER; f: POINTER): INTEGER alias "return luaL_dofile($L, $f);"
c_lua_getglobal (L: POINTER; name: POINTER) alias "lua_getglobal($L, $name);"
c_lua_setglobal (L: POINTER; name: POINTER) alias "lua_setglobal($L, $name);"
```

### ECF Setup

```xml
<external_include location="$SIMPLE_EIFFEL/simple_lua/Clib"/>
<external_library location="$SIMPLE_EIFFEL/simple_lua/Clib/lua54.lib">
    <condition><platform value="windows"/></condition>
</external_library>
```

### Tests

1. Create/close state
2. Execute Lua string
3. Execute Lua file
4. Get/set globals
5. Call Lua functions from Eiffel
6. Register Eiffel functions callable from Lua

---

## Phase 4: simple_zig

### Architecture

```
SIMPLE_ZIG (Facade - thin wrapper)
├── ZIG_LIBRARY (dynamic loading, same as Rust)
├── ZIG_FUNCTION (function handle)
└── ZIG_ALLOCATOR (optional: Zig allocator bridge)
```

### Notes

Zig is simpler than Rust because:
- No ownership/borrowing complexity
- Direct C interop (no `#[no_mangle]` needed with `export`)
- No runtime, no special cleanup

### Zig Side

```zig
export fn add(a: i32, b: i32) i32 {
    return a + b;
}

export fn greet(name: [*:0]const u8) [*:0]const u8 {
    // Return static or use provided allocator
}
```

### Tests

1. Load Zig DLL
2. Call exported functions
3. String passing
4. Array/slice handling

---

## Phase 5: simple_d (If Time Permits)

Similar to Rust/Zig pattern:
- D compiles to shared library with `extern(C)`
- Load via LoadLibrary/dlopen
- Call exported functions

---

## Phase 6: simple_fortran (If Time Permits)

### Value

Access to decades of optimized numerical libraries:
- BLAS (Basic Linear Algebra)
- LAPACK (Linear Algebra Package)
- Many scientific computing libraries

### Pattern

```fortran
subroutine my_sub(x, y, result) bind(C, name="my_sub")
    use iso_c_binding
    real(c_double), intent(in) :: x, y
    real(c_double), intent(out) :: result
    result = x + y
end subroutine
```

---

## Directory Structure

```
/d/prod/
├── simple_python/
│   ├── src/
│   │   ├── simple_python.e
│   │   ├── py_interpreter.e
│   │   ├── py_object.e
│   │   ├── py_module.e
│   │   └── ...
│   ├── testing/
│   ├── Clib/
│   └── simple_python.ecf
├── simple_rust/
├── simple_lua/
├── simple_zig/
├── simple_d/         (if time)
└── simple_fortran/   (if time)
```

## Success Criteria

Each library must have:
- [ ] Facade class with DBC
- [ ] Complete class diagram in docs
- [ ] README with usage examples
- [ ] ECF with proper externals
- [ ] Minimum 10 passing tests
- [ ] Inline C externals (no separate .c files)

## Dependencies

| Library | External Requirement |
|---------|---------------------|
| simple_python | Python 3.x + development headers |
| simple_rust | Pre-compiled Rust DLLs |
| simple_lua | Lua 5.4 (can embed source) |
| simple_zig | Pre-compiled Zig DLLs |
| simple_d | Pre-compiled D DLLs |
| simple_fortran | Pre-compiled Fortran libs or Intel MKL |

## References

- FFI Guide: `/d/prod/reference_docs/designs/CROSS_LANGUAGE_FFI_GUIDE.md`
- Research: `/d/prod/reference_docs/research/SIMPLE_*_RESEARCH.md`

---

*Plan created 2024-12-26 for implementation 2024-12-27*
