# simple_d Research Report

**Date**: 2024-12-26
**Feasibility**: MEDIUM-HIGH
**Recommendation**: BUILD (if time permits)

## Executive Summary

D is a systems programming language with excellent C interoperability via `extern(C)`. It offers a middle ground between C++ complexity and C simplicity, with modern features like garbage collection (optional), generics, and contract programming (similar to Eiffel's DBC).

## Step 1: Problem Definition

**Goal**: Enable Eiffel to call D libraries for systems programming tasks.

**Use Cases**:
- High-performance systems code
- Projects already using D
- Alternative to C++ with cleaner syntax
- Leverage D's standard library (Phobos)

## Step 2: Technical Research

### D C Interoperability

```d
// mathlib.d
extern(C) int add(int a, int b) {
    return a + b;
}

extern(C) double multiply(double a, double b) {
    return a * b;
}

extern(C) const(char)* greet(const(char)* name) {
    import core.stdc.stdlib : malloc;
    import core.stdc.string : strlen, strcpy;

    // Allocate and return (caller frees)
    auto len = strlen(name) + 8;
    auto buf = cast(char*)malloc(len);
    strcpy(buf, "Hello, ");
    // ... append name
    return buf;
}
```

Build:
```bash
dmd -shared -of=mathlib.dll mathlib.d
# or with LDC (LLVM-based):
ldc2 --shared -of=mathlib.dll mathlib.d
```

### Key Features

1. **extern(C)**: Direct C ABI compatibility
2. **@nogc**: Functions that don't use GC
3. **Contracts**: `in`, `out`, `invariant` (like Eiffel!)
4. **BetterC**: Subset without runtime/GC

### D Contracts (Similar to Eiffel)

```d
int sqrt(int x)
in {
    assert(x >= 0, "x must be non-negative");
}
out(result) {
    assert(result * result <= x);
    assert((result + 1) * (result + 1) > x);
}
do {
    // implementation
}
```

## Step 3: Ecosystem Compatibility

| Requirement | Solution |
|-------------|----------|
| SCOOP compatibility | Use @nogc and extern(C) functions |
| Void safety | Nullable types in D |
| DBC | D has native contracts! |
| Windows support | DMD, LDC both support Windows |

## Step 4: Design Decisions

1. **Runtime**: Use BetterC or @nogc for minimal overhead
2. **Loading**: Dynamic library (same as Rust/Zig)
3. **Memory**: D manages D memory, explicit free for exports

### Proposed Classes

| Class | Purpose |
|-------|---------|
| SIMPLE_D | Facade - library loading |
| D_LIBRARY | Dynamic library wrapper |
| D_FUNCTION | Function caller |
| D_STRING | String wrapper |

## Step 5: Conclusion

**Feasibility**: MEDIUM-HIGH

**Recommendation**: BUILD (lower priority than Lua/Zig)

**Rationale**:
- D has native contracts (philosophically aligned with Eiffel)
- Clean C interop via extern(C)
- BetterC mode eliminates runtime concerns
- Smaller community than Rust but mature

**Estimated Effort**: 1 phase (similar to Zig)

---

*Research completed as part of cross-language interoperability investigation.*
