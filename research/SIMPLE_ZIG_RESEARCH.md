# simple_zig Research Report

**Date**: 2024-12-26
**Feasibility**: HIGH
**Recommendation**: BUILD

## Executive Summary

Zig provides seamless C interoperability with zero runtime overhead. Unlike Rust, Zig doesn't require special annotations for C FFI - it's designed from the ground up to interoperate with C. Zig is an excellent choice for writing high-performance library code callable from Eiffel.

## Step 1: Problem Definition

**Goal**: Enable Eiffel programs to call Zig libraries, leveraging Zig's safety features and performance for systems-level code.

**Use Cases**:
- High-performance algorithms
- Memory-safe systems code without Rust's complexity
- Replacing C code with safer alternative
- Cross-compilation (Zig excels at this)

## Step 2: Technical Research

### Zig C Interoperability

Zig's `export` keyword creates C-compatible functions:

```zig
// lib.zig
const std = @import("std");

export fn add(a: i32, b: i32) i32 {
    return a + b;
}

export fn multiply(a: f64, b: f64) f64 {
    return a * b;
}

export fn greet(name: [*:0]const u8) [*:0]const u8 {
    // For simplicity, return static string
    // Real code would use allocator
    return "Hello!";
}
```

Build command:
```bash
zig build-lib -dynamic -O ReleaseFast lib.zig
# Produces: lib.dll (Windows), liblib.so (Linux)
```

### Key Zig FFI Features

1. **No runtime**: Zero-cost abstractions, no GC
2. **Direct C types**: `c_int`, `c_char`, `[*:0]const u8` (null-terminated)
3. **@cImport**: Can directly import and use C headers
4. **Automatic C ABI**: `export` functions use C calling convention

### Type Mapping

| Zig Type | C Type | Eiffel Type |
|----------|--------|-------------|
| `i32` | `int32_t` | `INTEGER_32` |
| `i64` | `int64_t` | `INTEGER_64` |
| `f32` | `float` | `REAL_32` |
| `f64` | `double` | `REAL_64` |
| `bool` | `_Bool` | `BOOLEAN` |
| `[*:0]const u8` | `const char*` | `POINTER` (C_STRING) |
| `[*]u8` | `uint8_t*` | `POINTER` |
| `?*T` | `T*` (nullable) | `detachable POINTER` |

### Eiffel Integration Pattern

```eiffel
class SIMPLE_ZIG

feature -- Initialization

    make (library_path: STRING)
        require
            path_valid: not library_path.is_empty
        local
            c_path: C_STRING
        do
            create c_path.make (library_path)
            library_handle := c_load_library (c_path.item)
        ensure
            loaded: is_loaded
        end

feature -- Status

    is_loaded: BOOLEAN
        do
            Result := library_handle /= default_pointer
        end

feature -- Operations

    add (a, b: INTEGER_32): INTEGER_32
        require
            loaded: is_loaded
        do
            Result := zig_add (a, b)
        end

    multiply (a, b: REAL_64): REAL_64
        require
            loaded: is_loaded
        do
            Result := zig_multiply (a, b)
        end

feature -- Cleanup

    unload
        do
            if is_loaded then
                c_free_library (library_handle)
                library_handle := default_pointer
            end
        ensure
            not_loaded: not is_loaded
        end

feature {NONE} -- Implementation

    library_handle: POINTER

feature {NONE} -- C Externals (Library Loading)

    c_load_library (path: POINTER): POINTER
        external "C inline use <windows.h>"
        alias "return LoadLibraryA((const char*)$path);"
        end

    c_free_library (lib: POINTER)
        external "C inline use <windows.h>"
        alias "FreeLibrary($lib);"
        end

feature {NONE} -- Zig Function Wrappers

    zig_add (a, b: INTEGER_32): INTEGER_32
        external "C inline use %"zig_lib.h%""
        alias "return add($a, $b);"
        end

    zig_multiply (a, b: REAL_64): REAL_64
        external "C inline use %"zig_lib.h%""
        alias "return multiply($a, $b);"
        end

end
```

### Header File (zig_lib.h)

```c
#ifndef ZIG_LIB_H
#define ZIG_LIB_H

#include <stdint.h>

#ifdef __cplusplus
extern "C" {
#endif

int32_t add(int32_t a, int32_t b);
double multiply(double a, double b);
const char* greet(const char* name);

#ifdef __cplusplus
}
#endif

#endif
```

## Step 3: Ecosystem Compatibility

| Requirement | Solution |
|-------------|----------|
| SCOOP compatibility | Zig functions are inherently thread-safe (no global state) |
| Void safety | Zig optionals map to detachable |
| DBC | Contracts on Eiffel side |
| Windows support | Zig has excellent Windows cross-compilation |

**Dependencies**:
- Pre-compiled Zig libraries (DLLs)
- Header files for Eiffel externals
- simple_file (for DLL path resolution)

## Step 4: Developer Pain Points

1. **String handling** - Null-terminated vs slices
2. **Memory ownership** - Who allocates/frees?
3. **Build integration** - Need Zig toolchain for library compilation
4. **Error handling** - Zig errors don't cross FFI

## Step 5: Innovation Opportunities

1. **Allocator bridge** - Pass Eiffel allocator to Zig
2. **Slice wrappers** - Safe handling of Zig slices
3. **Error translation** - Zig error unions to Eiffel exceptions
4. **Build integration** - Simple build script for Zig libs
5. **Header generation** - Auto-generate .h from Zig exports

## Step 6: Design Decisions

1. **Loading model**: Dynamic loading (LoadLibrary/dlopen)
2. **Build model**: Pre-compile Zig libs, ship DLLs
3. **Memory model**: Zig owns Zig memory, provide explicit free
4. **String model**: Null-terminated for simplicity

### Proposed Classes

| Class | Purpose |
|-------|---------|
| SIMPLE_ZIG | Facade - library loading |
| ZIG_LIBRARY | Dynamic library wrapper |
| ZIG_FUNCTION | Generic function caller |
| ZIG_STRING | Null-terminated string wrapper |
| ZIG_SLICE | Slice/array wrapper |
| ZIG_ALLOCATOR | Optional allocator bridge |

### Architecture

```
SIMPLE_ZIG (Facade)
├── ZIG_LIBRARY (LoadLibrary wrapper)
├── ZIG_FUNCTION (GetProcAddress wrapper)
├── ZIG_STRING (null-terminated string)
└── ZIG_SLICE (pointer + length)
```

## Step 7: Conclusion

**Feasibility**: HIGH

**Recommendation**: BUILD simple_zig

**Rationale**:
- Zig's C interop is seamless (designed for it)
- No runtime overhead (unlike Go, Java)
- Simpler than Rust (no borrow checker complexity)
- Excellent cross-compilation
- Growing ecosystem of quality libraries

**Implementation Priority**: Phase 1 (Core)
1. Library loading (LoadLibrary/dlopen)
2. Basic type marshalling (integers, floats)
3. String handling
4. Function calling

**Estimated Effort**: 1 phase (simpler than Rust due to cleaner C interop)

## Comparison: Zig vs Rust for FFI

| Aspect | Zig | Rust |
|--------|-----|------|
| C interop | Native (`export`) | Requires `#[no_mangle] extern "C"` |
| Runtime | None | None |
| Memory safety | Compile-time checks | Borrow checker |
| Learning curve | Lower | Higher |
| Ecosystem | Growing | Large |
| Panic handling | Configurable | Must catch_unwind |

## Example: Full Integration

### Zig Library (mathlib.zig)

```zig
const std = @import("std");

export fn factorial(n: u64) u64 {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

export fn fibonacci(n: u32) u64 {
    if (n <= 1) return n;
    var a: u64 = 0;
    var b: u64 = 1;
    var i: u32 = 2;
    while (i <= n) : (i += 1) {
        const temp = a + b;
        a = b;
        b = temp;
    }
    return b;
}

export fn is_prime(n: u64) bool {
    if (n < 2) return false;
    if (n == 2) return true;
    if (n % 2 == 0) return false;
    var i: u64 = 3;
    while (i * i <= n) : (i += 2) {
        if (n % i == 0) return false;
    }
    return true;
}
```

### Eiffel Usage

```eiffel
class APPLICATION

create
    make

feature
    make
        local
            zig: SIMPLE_ZIG
        do
            create zig.make ("mathlib.dll")

            print ("10! = " + zig.factorial (10).out + "%N")
            print ("fib(20) = " + zig.fibonacci (20).out + "%N")
            print ("is_prime(17) = " + zig.is_prime (17).out + "%N")

            zig.unload
        end

end
```

## Sources

- [Zig Language Reference](https://ziglang.org/documentation/master/)
- [Zig C Interop](https://ziglang.org/documentation/master/#C-Type-Coercion)
- [Zig Build System](https://ziglang.org/documentation/master/#Build-System)

---

*Research completed as part of cross-language interoperability investigation.*
