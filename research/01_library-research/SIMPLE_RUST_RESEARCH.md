# simple_rust Research Report

**Date**: 2024-12-26
**Feasibility**: HIGH
**Recommendation**: BUILD

## Executive Summary

Rust interoperability via Eiffel's C FFI is highly feasible. Rust's `cdylib` crate type produces C-compatible shared libraries with `#[no_mangle] pub extern "C"` functions. This is Rust's primary FFI mechanism and is production-ready.

## Step 1: Problem Definition

**Goal**: Enable Eiffel programs to call Rust libraries, leveraging Rust's performance and safety guarantees for systems-level code.

**Use Cases**:
- High-performance algorithms (parsing, compression, crypto)
- Memory-safe systems code
- Leverage Rust crates (cargo ecosystem)
- Gradual migration of performance-critical code to Rust

## Step 2: Technical Research

### Rust cdylib Pattern

Rust can export C-compatible functions:

```rust
// lib.rs
#[no_mangle]
pub extern "C" fn rust_add(a: i32, b: i32) -> i32 {
    a + b
}

#[no_mangle]
pub extern "C" fn rust_greet(name: *const c_char) -> *mut c_char {
    let c_str = unsafe { CStr::from_ptr(name) };
    let greeting = format!("Hello, {}!", c_str.to_str().unwrap());
    CString::new(greeting).unwrap().into_raw()
}

#[no_mangle]
pub extern "C" fn rust_free_string(s: *mut c_char) {
    if !s.is_null() {
        unsafe { drop(CString::from_raw(s)); }
    }
}
```

```toml
# Cargo.toml
[lib]
crate-type = ["cdylib"]
```

Build produces: `target/release/mylib.dll` (Windows)

### Eiffel Integration Pattern

```eiffel
class SIMPLE_RUST

feature -- Basic Operations

    add (a, b: INTEGER_32): INTEGER_32
        do
            Result := c_rust_add (a, b)
        end

    greet (name: STRING): STRING
        local
            c_name: C_STRING
            c_result: POINTER
        do
            create c_name.make (name)
            c_result := c_rust_greet (c_name.item)
            if not c_result.is_default_pointer then
                create Result.make_from_c (c_result)
                c_rust_free_string (c_result)
            else
                Result := ""
            end
        end

feature {NONE} -- C Externals

    c_rust_add (a, b: INTEGER_32): INTEGER_32
        external "C inline use %"rust_lib.h%""
        alias "return rust_add($a, $b);"
        end

    c_rust_greet (name: POINTER): POINTER
        external "C inline use %"rust_lib.h%""
        alias "return rust_greet((const char*)$name);"
        end

    c_rust_free_string (s: POINTER)
        external "C inline use %"rust_lib.h%""
        alias "rust_free_string($s);"
        end

end
```

## Step 3: Ecosystem Compatibility

| Requirement | Solution |
|-------------|----------|
| SCOOP compatibility | Rust functions are thread-safe by default |
| Void safety | Null pointer checks on all returns |
| DBC | Contracts on Eiffel side |
| Windows support | `cargo build --release` produces DLL |

**Dependencies**:
- Rust toolchain (rustc, cargo)
- Pre-built Rust libraries (DLLs)
- simple_file (for DLL path resolution)

## Step 4: Developer Pain Points

1. **String ownership** - Rust strings must be freed by Rust
2. **Memory management** - Who owns heap allocations?
3. **Panic safety** - Rust panics across FFI are UB
4. **Build complexity** - Need cargo + rustc for Rust side

## Step 5: Innovation Opportunities

1. **Automatic string cleanup** - Track Rust allocations, free in dispose
2. **Result wrappers** - Rust Result<T,E> -> Eiffel detachable
3. **Panic catching** - catch_unwind at FFI boundary
4. **Pre-built crates** - Ship common Rust libs as DLLs
5. **Bindgen integration** - Auto-generate Eiffel wrappers

## Step 6: Design Decisions

1. **Library model**: Load pre-built Rust DLLs, not compile on demand
2. **Memory model**: Rust owns Rust memory, provide free functions
3. **Error model**: C-style return codes, not panics across FFI
4. **Async model**: Not initially (Rust async complicates FFI)

### Proposed Classes

| Class | Purpose |
|-------|---------|
| SIMPLE_RUST | Facade - library loading, common patterns |
| RUST_LIBRARY | Dynamic library loader |
| RUST_STRING | Owned Rust string wrapper |
| RUST_RESULT | Result<T,E> pattern |
| RUST_SLICE | Rust slice/Vec wrapper |

### Safety Requirements

All Rust FFI functions must:
1. Use `catch_unwind` to prevent panics crossing FFI
2. Return null/error codes instead of panicking
3. Provide explicit free functions for heap allocations
4. Document ownership clearly

```rust
#[no_mangle]
pub extern "C" fn safe_operation(input: *const c_char) -> *mut c_char {
    std::panic::catch_unwind(|| {
        // actual implementation
    })
    .unwrap_or(std::ptr::null_mut())
}
```

## Step 7: Conclusion

**Feasibility**: HIGH

**Recommendation**: BUILD simple_rust

**Rationale**:
- Rust cdylib is production-ready FFI mechanism
- High value: Rust's performance + safety for critical code
- Clear ownership semantics (Rust owns Rust memory)
- Growing cargo ecosystem of high-quality crates

**Implementation Priority**: Phase 1 (Core)
1. Library loading (LoadLibrary/dlopen)
2. Basic type marshalling (integers, floats)
3. String handling with ownership
4. Error/Result handling

**Use Cases for simple_rust**:
- Wrap high-performance Rust crates (regex, serde, compression)
- Cryptography (ring, rustls)
- Parsing (nom, pest)

**Estimated Effort**: 2 phases (simpler than Python due to C-native types)

---

*Research completed as part of cross-language interoperability investigation.*
