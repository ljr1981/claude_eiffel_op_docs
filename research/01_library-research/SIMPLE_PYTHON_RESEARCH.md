# simple_python Research Report

**Date**: 2024-12-26
**Feasibility**: HIGH
**Recommendation**: BUILD

## Executive Summary

Python interoperability via Eiffel's C FFI is highly feasible. Python provides a mature, stable C API (Python/C API) that allows embedding Python interpreters and calling Python functions from C code. Since Eiffel can call C, Eiffel can call Python.

## Step 1: Problem Definition

**Goal**: Enable Eiffel programs to leverage Python libraries (NumPy, Pandas, ML frameworks, etc.) through a simple_python library.

**Use Cases**:
- Data science and machine learning from Eiffel applications
- Access to Python's rich ecosystem (250,000+ PyPI packages)
- Scripting and automation within Eiffel applications
- Leverage existing Python codebases

## Step 2: Technical Research

### Python C API Overview

Python provides `Python.h` with functions for:
- **Interpreter Control**: `Py_Initialize()`, `Py_Finalize()`
- **Module Import**: `PyImport_ImportModule()`
- **Object Creation**: `PyLong_FromLong()`, `PyUnicode_FromString()`
- **Function Calls**: `PyObject_CallMethod()`, `PyObject_CallObject()`
- **Reference Counting**: `Py_INCREF()`, `Py_DECREF()`

### Example C Integration

```c
#include <Python.h>

int main() {
    Py_Initialize();

    // Import module
    PyObject *pModule = PyImport_ImportModule("math");

    // Call function
    PyObject *pResult = PyObject_CallMethod(pModule, "sqrt", "d", 16.0);

    double result = PyFloat_AsDouble(pResult);
    printf("sqrt(16) = %f\n", result);

    Py_DECREF(pResult);
    Py_DECREF(pModule);
    Py_Finalize();
    return 0;
}
```

### Eiffel Integration Pattern

```eiffel
class SIMPLE_PYTHON

feature -- Initialization

    make
        do
            c_py_initialize
        ensure
            is_initialized: is_ready
        end

feature -- Module Operations

    import_module (name: STRING): detachable PYTHON_MODULE
        require
            name_valid: not name.is_empty
        local
            c_name: C_STRING
        do
            create c_name.make (name)
            if attached c_py_import_module (c_name.item) as ptr then
                create Result.make_from_pointer (ptr)
            end
        end

feature {NONE} -- C Externals

    c_py_initialize
        external "C inline use <Python.h>"
        alias "Py_Initialize();"
        end

    c_py_import_module (name: POINTER): POINTER
        external "C inline use <Python.h>"
        alias "return PyImport_ImportModule((const char*)$name);"
        end

end
```

## Step 3: Ecosystem Compatibility

| Requirement | Solution |
|-------------|----------|
| SCOOP compatibility | Python GIL naturally serializes access |
| Void safety | Wrap PyObject* in detachable types |
| DBC | Contracts on all public features |
| Windows support | Python for Windows provides C headers/libs |

**Dependencies**:
- Python 3.x installation (python3.dll, Python.h)
- simple_file (for path resolution)
- simple_json (for data exchange)

## Step 4: Developer Pain Points

1. **Reference counting complexity** - Easy to leak PyObject references
2. **Type marshalling** - Converting Eiffel types to/from Python
3. **Error handling** - Python exceptions need translation
4. **GIL management** - Thread safety considerations

## Step 5: Innovation Opportunities

1. **Automatic reference counting** - RAII pattern in Eiffel wrappers
2. **Type-safe wrappers** - PYTHON_INTEGER, PYTHON_STRING, PYTHON_LIST
3. **Contract-based error checking** - PyErr_Occurred in postconditions
4. **JSON bridge** - simple_json for complex data structures
5. **Virtual environment support** - Auto-detect and activate venvs
6. **Module facades** - PY_NUMPY, PY_PANDAS pre-built wrappers
7. **GIL abstraction** - Sub-interpreters for SCOOP (Python 3.12+)

## Step 6: Design Decisions

1. **Embed vs Extend**: EMBED - Eiffel hosts Python, not vice versa
2. **Python version**: 3.x only (2.x deprecated), use Limited API for stability
3. **Object model**: PyObject wrapped in PYTHON_OBJECT descendants
4. **Memory management**: Explicit decref in dispose, or via once references
5. **Error model**: Python exceptions -> Eiffel exceptions

### Proposed Classes

| Class | Purpose |
|-------|---------|
| SIMPLE_PYTHON | Facade - interpreter lifecycle |
| PY_INTERPRETER | Singleton, lifecycle management |
| PYTHON_OBJECT | Base wrapper for PyObject* |
| PYTHON_MODULE | Module operations |
| PYTHON_CALLABLE | Function/method calls |
| PYTHON_INTEGER | Integer values |
| PYTHON_STRING | String values |
| PYTHON_LIST | List operations |
| PYTHON_DICT | Dictionary operations |
| PYTHON_ERROR | Exception handling |

### Architecture

```
SIMPLE_PYTHON (Facade)
├── PY_INTERPRETER (singleton, lifecycle)
├── PY_MODULE (import, attribute access)
├── PY_OBJECT (reference-counted wrapper)
├── PY_CALLABLE (function calls)
└── PY_EXCEPTION (error handling)
```

## Step 7: Conclusion

**Feasibility**: HIGH

**Recommendation**: BUILD simple_python

**Rationale**:
- Python C API is stable and well-documented (30+ years)
- High value: access to 250K+ Python packages (numpy, pandas, ML frameworks)
- Pattern proven in other ecosystems (Ruby, Lua wrappers exist)
- Reasonable complexity for inline C approach

**Implementation Priority**: Phase 1 (Core)
1. Interpreter init/finalize
2. Module import
3. Basic type wrappers (int, string, float)
4. Function calls with simple types
5. Error handling

**Estimated Effort**:
- Phase 1 (Core): 2-3 weeks
- Phase 2 (Types): 2 weeks
- Phase 3 (Facades): 2 weeks
- Phase 4 (Advanced): 2 weeks
- **Total: 8-10 weeks to production**

## Sources

- [Python C API Reference](https://docs.python.org/3/c-api/index.html)
- [Embedding Python](https://docs.python.org/3/extending/embedding.html)
- [PyO3 Design Patterns](https://pyo3.rs/)

---

*Research completed as part of cross-language interoperability investigation.*
