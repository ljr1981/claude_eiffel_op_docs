# simple_go Research Report

**Date**: 2024-12-26
**Feasibility**: LOW
**Recommendation**: SKIP (use HTTP/gRPC instead)

## Executive Summary

Go can produce C-compatible shared libraries via cgo and `-buildmode=c-shared`, but the overhead and complexity make FFI impractical. Go's runtime (garbage collector, goroutine scheduler) adds 30-40x overhead for FFI calls. HTTP/gRPC interop is the recommended approach.

## Step 1: Problem Definition

**Goal**: Evaluate whether Eiffel can call Go libraries via C FFI.

**Potential Use Cases**:
- Go's excellent concurrency primitives
- Cloud-native tooling (Docker, K8s client libraries)
- Network programming

## Step 2: Technical Research

### Go cgo Pattern

Go can export C functions:

```go
package main

import "C"

//export GoAdd
func GoAdd(a, b C.int) C.int {
    return a + b
}

//export GoGreet
func GoGreet(name *C.char) *C.char {
    goName := C.GoString(name)
    greeting := "Hello, " + goName + "!"
    return C.CString(greeting)  // Caller must free
}

func main() {}  // Required but unused
```

Build: `go build -buildmode=c-shared -o mylib.dll`

### The Go Runtime Problem

Unlike Rust or C, Go has a significant runtime:
- **Garbage Collector**: Runs in background, can pause
- **Goroutine Scheduler**: M:N threading model
- **Stack Management**: Goroutines have growable stacks

When you call into Go via cgo:
1. Go runtime must be initialized (first call)
2. Each call crosses the cgo boundary (expensive)
3. Go GC may run at any time

### Performance Overhead

Benchmarks show **30-40x overhead** for cgo calls vs native Go:

```
BenchmarkGoNative:     2.5 ns/op
BenchmarkCgoCall:      80-100 ns/op
```

This makes fine-grained FFI impractical. You can't call Go functions in a tight loop.

### Memory Management Issues

1. **Go GC can move objects** - Can't pass Go pointers to C safely
2. **C.CString allocates on C heap** - Must be freed by caller
3. **Pointer passing rules** are complex and error-prone

```go
// This is UNSAFE - Go might move the data
//export UnsafeGetBuffer
func UnsafeGetBuffer() *C.char {
    data := make([]byte, 100)
    return (*C.char)(unsafe.Pointer(&data[0]))  // WRONG!
}
```

## Step 3: Ecosystem Compatibility

| Requirement | Challenge |
|-------------|-----------|
| SCOOP compatibility | Go runtime conflicts with SCOOP threading |
| Performance | 30-40x FFI overhead |
| Memory safety | Complex pointer rules |
| Windows support | Works but adds DLL + Go runtime |

## Step 4: Developer Pain Points

1. **Massive overhead** - 80-100ns per call vs 2-3ns
2. **Runtime initialization** - First call is slow
3. **No fine-grained control** - Go GC runs when it wants
4. **Complex pointer rules** - Easy to create dangling pointers
5. **Large binary size** - Go runtime adds ~2MB minimum

## Step 5: Alternative Approaches

### HTTP/REST (Recommended)

```
[Eiffel App] <--HTTP--> [Go Service]
```

- Use simple_http to call Go services
- Go excels at HTTP servers
- Clean process isolation
- No FFI complexity

### gRPC (Recommended for Performance)

```
[Eiffel App] <--gRPC--> [Go Service]
```

- Use simple_grpc for efficient RPC
- Protocol Buffers for type safety
- HTTP/2 multiplexing
- Better than cgo for most use cases

### When FFI Might Work

- **Batch operations**: Call Go once with large data
- **Coarse-grained APIs**: Single call does lots of work
- **Initialization only**: Go sets up resources, C uses them

## Step 6: Design Decisions

**Decision: Do NOT build simple_go for FFI**

**Rationale**:
1. 30-40x overhead makes it impractical for normal use
2. Go's strengths (goroutines, channels) don't translate to FFI
3. HTTP/gRPC are idiomatic for Go interop
4. simple_http and simple_grpc already exist

### Recommended Patterns

| Use Case | Solution |
|----------|----------|
| Call Go libraries | HTTP microservice |
| High-performance RPC | gRPC |
| Leverage Go concurrency | Separate process |
| Docker/K8s integration | CLI or HTTP API |

## Step 7: Conclusion

**Feasibility**: LOW

**Recommendation**: SKIP simple_go FFI

**Rationale**:
- 30-40x FFI overhead is prohibitive
- Go's runtime (GC, scheduler) conflicts with Eiffel control
- Pointer passing rules are complex and error-prone
- HTTP/gRPC are the idiomatic Go integration patterns

**Alternative**: Use existing libraries
- `simple_http` for REST APIs to Go services
- `simple_grpc` for high-performance RPC to Go services
- `simple_docker` / `simple_k8s` already wrap Go-based CLIs

**When Go FFI Might Make Sense**:
- Batch processing (call Go once with megabytes of data)
- One-time initialization (Go sets up state, returns handle)
- If you absolutely need a single-process solution

---

*Research completed as part of cross-language interoperability investigation.*
