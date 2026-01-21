# simple_lua Research Report

**Date**: 2024-12-26
**Feasibility**: HIGH
**Recommendation**: BUILD

## Executive Summary

Lua is the ideal embeddable scripting language. With a ~200KB footprint, sub-millisecond startup, and a C API designed specifically for embedding, Lua is easier to integrate than Python and perfect for user-extensible Eiffel applications.

## Step 1: Problem Definition

**Goal**: Enable Eiffel programs to embed Lua scripting for configuration, plugins, and user extensibility.

**Use Cases**:
- User-defined scripts in Eiffel applications
- Configuration files (Lua as config language)
- Game scripting (Lua is industry standard)
- Plugin systems
- REPL/interactive shells

## Step 2: Technical Research

### Lua C API Overview

Lua's C API is stack-based:

```c
#include <lua.h>
#include <lauxlib.h>
#include <lualib.h>

int main() {
    lua_State *L = luaL_newstate();  // Create state
    luaL_openlibs(L);                 // Load standard libs

    // Execute Lua code
    luaL_dostring(L, "print('Hello from Lua!')");

    // Set global variable
    lua_pushnumber(L, 42);
    lua_setglobal(L, "answer");

    // Get global variable
    lua_getglobal(L, "answer");
    double val = lua_tonumber(L, -1);
    lua_pop(L, 1);

    lua_close(L);
    return 0;
}
```

### Stack-Based Operations

Lua uses a virtual stack for all data exchange:

```
Push operations:     lua_pushnil, lua_pushnumber, lua_pushstring, lua_pushboolean
Pop/read operations: lua_tonumber, lua_tostring, lua_toboolean, lua_pop
Stack management:    lua_gettop, lua_settop, lua_pushvalue, lua_remove
```

### Calling Lua Functions from C

```c
lua_getglobal(L, "my_function");  // Push function
lua_pushnumber(L, 10);             // Push arg 1
lua_pushnumber(L, 20);             // Push arg 2
lua_call(L, 2, 1);                 // Call: 2 args, 1 result
double result = lua_tonumber(L, -1);
lua_pop(L, 1);
```

### Registering C Functions in Lua

```c
int my_eiffel_function(lua_State *L) {
    double x = lua_tonumber(L, 1);
    double y = lua_tonumber(L, 2);
    lua_pushnumber(L, x + y);
    return 1;  // Number of return values
}

lua_pushcfunction(L, my_eiffel_function);
lua_setglobal(L, "eiffel_add");

// Now Lua can call: result = eiffel_add(10, 20)
```

### Eiffel Integration Pattern

```eiffel
class SIMPLE_LUA

feature -- Initialization

    make
        do
            state := c_lua_newstate
            c_lua_openlibs (state)
        ensure
            state_created: state /= default_pointer
        end

feature -- Execution

    do_string (code: STRING): BOOLEAN
        require
            code_valid: not code.is_empty
        local
            c_code: C_STRING
        do
            create c_code.make (code)
            Result := c_lua_dostring (state, c_code.item) = 0
        end

    do_file (path: STRING): BOOLEAN
        require
            path_exists: (create {RAW_FILE}.make (path)).exists
        local
            c_path: C_STRING
        do
            create c_path.make (path)
            Result := c_lua_dofile (state, c_path.item) = 0
        end

feature -- Variables

    set_global_number (name: STRING; value: REAL_64)
        local
            c_name: C_STRING
        do
            create c_name.make (name)
            c_lua_pushnumber (state, value)
            c_lua_setglobal (state, c_name.item)
        end

    get_global_number (name: STRING): REAL_64
        local
            c_name: C_STRING
        do
            create c_name.make (name)
            c_lua_getglobal (state, c_name.item)
            Result := c_lua_tonumber (state, -1)
            c_lua_pop (state, 1)
        end

feature -- Cleanup

    close
        do
            c_lua_close (state)
            state := default_pointer
        end

feature {NONE} -- Implementation

    state: POINTER

feature {NONE} -- C Externals

    c_lua_newstate: POINTER
        external "C inline use <lua.h>, <lauxlib.h>"
        alias "return luaL_newstate();"
        end

    c_lua_close (L: POINTER)
        external "C inline use <lua.h>"
        alias "lua_close((lua_State*)$L);"
        end

    c_lua_openlibs (L: POINTER)
        external "C inline use <lua.h>, <lualib.h>"
        alias "luaL_openlibs((lua_State*)$L);"
        end

    c_lua_dostring (L: POINTER; s: POINTER): INTEGER
        external "C inline use <lua.h>, <lauxlib.h>"
        alias "return luaL_dostring((lua_State*)$L, (const char*)$s);"
        end

    c_lua_dofile (L: POINTER; f: POINTER): INTEGER
        external "C inline use <lua.h>, <lauxlib.h>"
        alias "return luaL_dofile((lua_State*)$L, (const char*)$f);"
        end

    c_lua_pushnumber (L: POINTER; n: REAL_64)
        external "C inline use <lua.h>"
        alias "lua_pushnumber((lua_State*)$L, $n);"
        end

    c_lua_tonumber (L: POINTER; idx: INTEGER): REAL_64
        external "C inline use <lua.h>"
        alias "return lua_tonumber((lua_State*)$L, $idx);"
        end

    c_lua_getglobal (L: POINTER; name: POINTER)
        external "C inline use <lua.h>"
        alias "lua_getglobal((lua_State*)$L, (const char*)$name);"
        end

    c_lua_setglobal (L: POINTER; name: POINTER)
        external "C inline use <lua.h>"
        alias "lua_setglobal((lua_State*)$L, (const char*)$name);"
        end

    c_lua_pop (L: POINTER; n: INTEGER)
        external "C inline use <lua.h>"
        alias "lua_pop((lua_State*)$L, $n);"
        end

end
```

## Step 3: Ecosystem Compatibility

| Requirement | Solution |
|-------------|----------|
| SCOOP compatibility | Each lua_State is independent, thread-safe by isolation |
| Void safety | Stack operations checked, detachable for optional values |
| DBC | Contracts on stack depth, state validity |
| Windows support | Lua compiles easily on Windows |

**Dependencies**:
- Lua 5.4 headers and library (can embed source directly)
- simple_file (for script loading)

### Embedding Options

1. **Dynamic linking**: Link against lua54.dll
2. **Static linking**: Compile Lua source into Eiffel project
3. **Embed source**: Include lua.c (single-file amalgamation)

## Step 4: Developer Pain Points

1. **Stack management** - Must balance pushes and pops
2. **Type checking** - Lua is dynamically typed
3. **Error handling** - Protected calls needed for safety
4. **Memory** - Lua GC vs Eiffel memory model

## Step 5: Innovation Opportunities

1. **Stack guards** - Automatic stack depth verification in postconditions
2. **Type-safe wrappers** - LUA_NUMBER, LUA_STRING, LUA_TABLE
3. **Eiffel function registration** - Easy callback mechanism
4. **Coroutine support** - Expose Lua coroutines
5. **Sandboxing** - Restricted Lua environment for untrusted scripts

## Step 6: Design Decisions

1. **Lua version**: 5.4 (latest stable)
2. **Embedding**: Static link or DLL (configurable)
3. **State management**: One LUA_STATE per SIMPLE_LUA instance
4. **Error handling**: Protected calls (`lua_pcall`) by default

### Proposed Classes

| Class | Purpose |
|-------|---------|
| SIMPLE_LUA | Facade - create state, execute code |
| LUA_STATE | State wrapper with stack operations |
| LUA_VALUE | Base for typed value wrappers |
| LUA_NUMBER | Number (REAL_64) wrapper |
| LUA_STRING | String wrapper |
| LUA_TABLE | Table operations (get/set/iterate) |
| LUA_FUNCTION | Function handle, call with args |
| LUA_COROUTINE | Coroutine control |
| LUA_REGISTRY | Register Eiffel functions |

## Step 7: Conclusion

**Feasibility**: HIGH

**Recommendation**: BUILD simple_lua

**Rationale**:
- Lua was designed for embedding (unlike Python which added it later)
- Tiny footprint (~200KB vs Python's ~50MB)
- Fast startup (<1ms vs Python's 100-500ms)
- Simple, clean C API
- Industry standard for game scripting

**Implementation Priority**: Phase 1 (Core)
1. State creation/destruction
2. Execute string/file
3. Get/set globals (numbers, strings)
4. Call Lua functions
5. Error handling

**Estimated Effort**: 1-2 phases (simpler than Python)

## Comparison: Lua vs Python for Embedding

| Aspect | Lua | Python |
|--------|-----|--------|
| Runtime size | ~200KB | ~50MB |
| Startup time | <1ms | 100-500ms |
| API complexity | Low | Medium |
| Reference counting | No (GC) | Yes (manual) |
| Package ecosystem | Smaller | 250,000+ |
| Best for | Scripting, config | Data science, ML |

## Sources

- [Lua Reference Manual](https://www.lua.org/manual/5.4/)
- [Programming in Lua (PIL)](https://www.lua.org/pil/)
- [Lua C API Tutorial](https://www.lua.org/pil/24.html)

---

*Research completed as part of cross-language interoperability investigation.*
