# Cross-Language FFI Guide for Eiffel

**Date**: 2024-12-26
**Status**: Reference Document

## Overview

Eiffel can interoperate with other programming languages via C FFI (Foreign Function Interface). This document catalogs which languages are accessible and how.

## How C FFI Works

FFI is **runtime linking**, not compile-time fusion:

```
┌─────────────────────────────────────────────────────┐
│                   AT RUNTIME                        │
│                                                     │
│  ┌──────────────┐      ┌──────────────────────┐    │
│  │ your_app.exe │─────>│ language_lib.dll     │    │
│  │ (Eiffel)     │ calls│ (C-compatible API)   │    │
│  └──────────────┘      └──────────────────────┘    │
└─────────────────────────────────────────────────────┘
```

- **Separate compilation**: Each language compiles independently
- **Dynamic linking**: Binaries connect at runtime
- **C ABI**: The universal calling convention

### Two Patterns

**Pattern A: Embedded Interpreter** (Python, Lua, JavaScript)
```
Eiffel exe  ──links──>  interpreter.dll  ──runs──>  script.py/.lua/.js
```

**Pattern B: Pre-compiled Library** (Rust, Zig, D, Fortran)
```
Eiffel exe  ──links──>  library.dll (already compiled from Rust/Zig/etc)
```

## Language Accessibility Tiers

### Tier 1: Excellent Candidates (Easy FFI)

| Language | Mechanism | Runtime Size | Use Case |
|----------|-----------|--------------|----------|
| **Python** | Embeddable C API | ~50MB | ML, data science, scripting |
| **Lua** | Embeddable C API | ~200KB | Lightweight scripting, games |
| **Rust** | `#[no_mangle] extern "C"` | 0 (no runtime) | Performance, safety |
| **Zig** | `export fn` | 0 (no runtime) | Systems, modern C replacement |
| **Nim** | `{.exportc.}` | Minimal | Scripting with native speed |
| **D** | `extern(C)` | Minimal | Systems programming |
| **Fortran** | ISO_C_BINDING | 0 | Scientific computing |
| **C/C++** | Native | 0 | Everything |

### Tier 2: Viable Candidates (Some Complexity)

| Language | Mechanism | Notes |
|----------|-----------|-------|
| **Ruby** | mruby C API | Similar to Python, smaller |
| **JavaScript** | QuickJS/Duktape | ~620KB embeddable engines |
| **Crystal** | Native compile | Ruby-like syntax, needs compiler |
| **Swift** | `@_cdecl` | Apple-centric |
| **Ada** | `pragma Import/Export` | Safety-critical systems |

### Tier 3: Possible But Complex

| Language | Mechanism | Why It's Hard |
|----------|-----------|---------------|
| **Haskell** | Foreign.C | Heavy runtime (10MB+) |
| **OCaml** | Ctypes | GC interaction tricky |
| **C#/.NET** | NativeAOT | New, Windows-centric |
| **Kotlin Native** | CInterop | Ecosystem is JVM-focused |

### Tier 4: Not Recommended

| Language | Why |
|----------|-----|
| **Go** | 30-40x cgo overhead, runtime conflicts |
| **Java** | JNI complexity, JVM in-process (50-100MB) |
| **Erlang/Elixir** | BEAM VM, NIFs are dangerous |
| **Scala** | JVM-based |

## Eiffel FFI Pattern

```eiffel
class SIMPLE_EXAMPLE

feature -- Public API

    call_foreign_function (x: INTEGER): INTEGER
        do
            Result := c_foreign_add (x, 10)
        end

feature {NONE} -- C Externals

    c_foreign_add (a, b: INTEGER): INTEGER
        external "C inline use %"foreign_lib.h%""
        alias "return foreign_add($a, $b);"
        end

end
```

## Recommended simple_* Libraries

### Priority 1: High Value

| Library | Language | Value Proposition |
|---------|----------|-------------------|
| simple_python | Python | ML/AI, 250K+ packages |
| simple_rust | Rust | Performance, safety |
| simple_lua | Lua | Tiny scripting (200KB) |
| simple_zig | Zig | Modern C, zero runtime |

### Priority 2: Niche Value

| Library | Language | Value Proposition |
|---------|----------|-------------------|
| simple_d | D | Systems programming |
| simple_fortran | Fortran | Scientific computing (BLAS/LAPACK) |
| simple_js | JavaScript | Web-like scripting |

## Quick Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                 LANGUAGE COMPARISON FOR FFI                      │
├──────────┬───────────┬────────────┬─────────────────────────────┤
│ Language │ Runtime   │ Complexity │ Best For                    │
├──────────┼───────────┼────────────┼─────────────────────────────┤
│ Lua      │ 200KB     │ Very Low   │ User scripting, config      │
│ Python   │ 50MB      │ Medium     │ ML, data science            │
│ Rust     │ 0         │ Low        │ Performance-critical code   │
│ Zig      │ 0         │ Low        │ Systems, C replacement      │
│ D        │ Minimal   │ Low        │ Systems programming         │
│ Fortran  │ 0         │ Low        │ Numerical computing         │
│ JS       │ 620KB     │ Medium     │ Web-like scripting          │
├──────────┼───────────┼────────────┼─────────────────────────────┤
│ Go       │ 2MB+      │ HIGH       │ ❌ Use HTTP/gRPC instead    │
│ Java     │ 50MB+     │ VERY HIGH  │ ❌ Use REST instead         │
└──────────┴───────────┴────────────┴─────────────────────────────┘
```

## References

- Research reports: `/d/prod/reference_docs/research/SIMPLE_*_RESEARCH.md`
- Python C API: https://docs.python.org/3/c-api/
- Lua C API: https://www.lua.org/manual/5.4/manual.html#4
- Rust FFI: https://doc.rust-lang.org/nomicon/ffi.html
- Zig C interop: https://ziglang.org/documentation/master/#C-Type-Coercion

---

*Part of Simple Eiffel cross-language interoperability initiative.*
