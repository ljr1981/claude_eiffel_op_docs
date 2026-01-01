# Cross-Platform C++ (MSVC/Linux) Embedded-Python Control of FreeCAD

**Goal:** Build a cross-platform (Windows MSVC + Linux) native C/C++ application that **directly integrates** with FreeCAD by **embedding Python** and driving FreeCAD’s **Python API** (no external `FreeCADCmd` process unless you choose it as a fallback).

This document is a **design + plan + implementation guide** with practical build instructions, runtime setup, hardening, and packaging considerations.

---

## 0) Executive Summary

FreeCAD’s most stable automation surface is Python. The most maintainable “direct integration” from C/C++ is:

- **Your C++ application hosts an embedded Python interpreter**
- Your C++ code imports a small Python “bridge module”
- That Python bridge imports `FreeCAD` and executes CAD operations
- C++ passes requests in **JSON** and receives results as **JSON**

This yields:
- One process
- Clean function calls (not CLI + stdout parsing)
- Full access to FreeCAD’s features
- Cross-platform portability

---

## 1) System Architecture

### 1.1 Components

1. **Host App (C++ / MSVC + Linux)**
   - CLI, service, or GUI
   - owns process lifetime
   - manages configuration, logging, and job scheduling

2. **Embedded Python Runtime**
   - Python 3.x embedded distribution (Windows) or system python (Linux) or bundled python (both)
   - used only as an embedded interpreter

3. **FreeCAD Runtime**
   - Installed FreeCAD (recommended) OR bundled FreeCAD runtime
   - provides `FreeCAD` Python module and native libraries

4. **Bridge Module (Python)**
   - `bridge/__init__.py` + `bridge/runner.py`
   - single function entrypoint: `run_job(job_json: str) -> str`

5. **Job Spec (JSON)**
   - describes CAD action: create doc, import, modify, recompute, export

6. **Output Artifacts**
   - STEP/STL/DXF files
   - a JSON report (status, timings, warnings, exported paths)

### 1.2 Data Flow

```
C++ App
  | (JSON request)
  v
Embedded Python
  | import bridge
  v
bridge.run_job()
  | import FreeCAD, Part, ImportGui...
  v
FreeCAD + OCCT
  | (files written)
  v
JSON response -> C++
```

### 1.3 Key Constraints

- Treat FreeCAD operations as **single-threaded** per process.
- If you need concurrency, run **multiple worker processes**.
- Avoid UI automation; keep it headless (even when FreeCAD GUI exists).

---

## 2) FreeCAD Integration Strategy

### 2.1 Why embedded Python is the “best” integration

- FreeCAD’s internal C++ API is not a stable public SDK.
- The community and workbenches primarily expose functionality through Python.
- Embedding Python keeps your core in C++ while using FreeCAD as a library/service.

### 2.2 Two operational modes

1. **Embedded Mode (primary)**
   - one process
   - C++ embeds Python
   - Python imports FreeCAD

2. **Fallback Mode (optional)**
   - spawn `FreeCADCmd` as a worker process
   - useful if environment setup fails
   - isolates crashes to a child process

---

## 3) Cross-Platform Folder Layout

Recommended repository layout:

```
project/
  CMakeLists.txt
  src/
    main.cpp
    pyembed.cpp
    pyembed.hpp
  python/
    bridge/
      __init__.py
      runner.py
      jobs.py
  examples/
    job_create_box.json
  runtime/
    windows/
      (optional bundled python + freecad runtime)
    linux/
      (optional appimage extraction or bundled runtime)
  docs/
```

---

## 4) Job Contract (JSON)

Example job: create a parametric box and export STEP.

```json
{
  "job_id": "demo-001",
  "action": "create_box",
  "params": {
    "length": 100,
    "width": 50,
    "height": 25
  },
  "export": {
    "format": "STEP",
    "path": "./out/box.step"
  }
}
```

Response JSON (string returned to C++):

```json
{
  "job_id": "demo-001",
  "ok": true,
  "exports": ["./out/box.step"],
  "warnings": [],
  "timing_ms": {
    "total": 143,
    "recompute": 37,
    "export": 62
  }
}
```

---

## 5) Python Bridge Implementation

### 5.1 `python/bridge/runner.py`

```python
import json
import time


def _now_ms():
    return int(time.time() * 1000)


def run_job(job_json: str) -> str:
    t0 = _now_ms()
    job = json.loads(job_json)
    job_id = job.get("job_id", "")

    result = {
        "job_id": job_id,
        "ok": False,
        "exports": [],
        "warnings": [],
        "timing_ms": {}
    }

    # Import FreeCAD late, so that import errors are cleanly reported.
    try:
        import FreeCAD
        import Part
    except Exception as e:
        result["warnings"].append(f"Failed to import FreeCAD/Part: {e}")
        result["timing_ms"]["total"] = _now_ms() - t0
        return json.dumps(result)

    try:
        action = job.get("action")
        params = job.get("params", {})
        export = job.get("export", {})

        doc = FreeCAD.newDocument(job_id or "Job")

        if action == "create_box":
            box = doc.addObject("Part::Box", "Box")
            box.Length = float(params.get("length", 10))
            box.Width = float(params.get("width", 10))
            box.Height = float(params.get("height", 10))

            t_recompute0 = _now_ms()
            doc.recompute()
            result["timing_ms"]["recompute"] = _now_ms() - t_recompute0

            fmt = (export.get("format") or "STEP").upper()
            out_path = export.get("path")
            if not out_path:
                raise ValueError("Missing export.path")

            t_exp0 = _now_ms()
            if fmt == "STEP" or fmt == "STP":
                import ImportGui
                ImportGui.export([box], out_path)
            elif fmt == "STL":
                import Mesh
                Mesh.export([box], out_path)
            else:
                raise ValueError(f"Unsupported export format: {fmt}")

            result["timing_ms"]["export"] = _now_ms() - t_exp0
            result["exports"].append(out_path)

        else:
            raise ValueError(f"Unsupported action: {action}")

        result["ok"] = True

    except Exception as e:
        # Do not throw—return structured failure for the host app.
        result["warnings"].append(str(e))

    result["timing_ms"]["total"] = _now_ms() - t0
    return json.dumps(result)
```

### 5.2 `python/bridge/__init__.py`

```python
from .runner import run_job
```

---

## 6) C++ Host Implementation (Core)

### 6.1 Key responsibilities

- Initialize Python
- Set Python paths so `bridge` and FreeCAD modules are found
- Call `bridge.run_job(job_json)`
- Capture exceptions/tracebacks
- Return structured output

### 6.2 C++ Example (single-file minimal)

**Note:** You will adapt path setup per OS (next sections cover this).

```cpp
#include <Python.h>
#include <iostream>
#include <fstream>
#include <sstream>
#include <string>

static std::string read_all(const std::string& path) {
    std::ifstream f(path);
    std::stringstream ss;
    ss << f.rdbuf();
    return ss.str();
}

static bool py_add_sys_path(const std::string& p) {
    std::string code = "import sys; sys.path.insert(0, r\"" + p + "\")";
    return PyRun_SimpleString(code.c_str()) == 0;
}

static std::string py_call_run_job(const std::string& job_json) {
    PyObject* bridge = PyImport_ImportModule("bridge");
    if (!bridge) {
        PyErr_Print();
        return "{\"ok\":false,\"warnings\":[\"Failed to import bridge\"]}";
    }

    PyObject* func = PyObject_GetAttrString(bridge, "run_job");
    if (!func || !PyCallable_Check(func)) {
        PyErr_Print();
        Py_XDECREF(func);
        Py_DECREF(bridge);
        return "{\"ok\":false,\"warnings\":[\"bridge.run_job not callable\"]}";
    }

    PyObject* args = PyTuple_New(1);
    PyTuple_SetItem(args, 0, PyUnicode_FromStringAndSize(job_json.c_str(), (Py_ssize_t)job_json.size()));

    PyObject* ret = PyObject_CallObject(func, args);
    Py_DECREF(args);
    Py_DECREF(func);
    Py_DECREF(bridge);

    if (!ret) {
        PyErr_Print();
        return "{\"ok\":false,\"warnings\":[\"Python exception in run_job\"]}";
    }

    PyObject* bytes = PyUnicode_AsUTF8String(ret);
    Py_DECREF(ret);

    if (!bytes) {
        PyErr_Print();
        return "{\"ok\":false,\"warnings\":[\"Failed converting return to UTF-8\"]}";
    }

    std::string out = PyBytes_AsString(bytes);
    Py_DECREF(bytes);
    return out;
}

int main(int argc, char** argv) {
    if (argc < 2) {
        std::cerr << "Usage: host_app <job.json>\n";
        return 2;
    }

    std::string job_json = read_all(argv[1]);

    Py_Initialize();

    // TODO: add platform-specific sys.path entries:
    // - path to python/ (bridge module)
    // - path to FreeCAD python modules
    // - DLL/SO search paths if needed

    // Example: make local bridge importable
    py_add_sys_path("./python");

    std::string result = py_call_run_job(job_json);
    std::cout << result << std::endl;

    Py_Finalize();
    return 0;
}
```

---

## 7) Windows (MSVC) Implementation Details

Windows is the hardest environment because:
- DLL discovery matters
- Python embedding needs explicit configuration
- FreeCAD installs vary by version

### 7.1 Choose a runtime strategy

**Strategy A (recommended for dev):** Depend on installed FreeCAD + installed Python
- Pro: simplest to start
- Con: user machine must match

**Strategy B (recommended for distribution):** Bundle Python + bundle FreeCAD runtime
- Pro: deterministic, no “works on my machine”
- Con: larger installer

This document provides both.

---

### 7.2 Discover FreeCAD install path (Windows)

Common locations:
- `C:\Program Files\FreeCAD 0.21\` (or similar)

Key subpaths:
- `bin\` (DLLs and executables)
- `lib\` (sometimes)
- `Mod\` (Python workbenches)

The FreeCAD Python module is typically importable when Python can find FreeCAD’s `lib`/`Mod` paths and the DLLs in `bin` are loadable.

#### Practical approach
- Let the user configure `FREECAD_HOME` environment variable
- Fall back to probing known paths

Example:
- `FREECAD_HOME=C:\Program Files\FreeCAD 0.21`

---

### 7.3 DLL search path configuration (Windows)

For embedded Python importing FreeCAD, Windows must locate FreeCAD DLLs.

**Best practice (Windows 10+):** call `AddDllDirectory` / `SetDefaultDllDirectories` early.

In C++:

```cpp
#ifdef _WIN32
#include <windows.h>
#include <string>

static void add_dll_dir(const std::wstring& dir) {
    SetDefaultDllDirectories(LOAD_LIBRARY_SEARCH_DEFAULT_DIRS | LOAD_LIBRARY_SEARCH_USER_DIRS);
    AddDllDirectory(dir.c_str());
}
#endif
```

Then add:
- `FREECAD_HOME\bin`

Also add `sys.path` entries for Python:
- `FREECAD_HOME\Mod`
- sometimes `FREECAD_HOME\lib`

---

### 7.4 Python embedding on Windows (MSVC)

#### Option 1: Link against Python installed on the system
- Install Python matching your build arch (x64)
- In MSVC:
  - Add include path: `...\Python311\include`
  - Link lib: `python311.lib`
  - Ensure `python311.dll` is findable at runtime

#### Option 2: Use the official Python embeddable package (recommended for distribution)
- Download Python embeddable zip
- Ship it alongside your app
- Set `Py_SetPythonHome()` / `Py_SetPath()`

**Recommended:** Use a bundled python to avoid mismatch.

---

### 7.5 MSVC build with CMake

`CMakeLists.txt` (simplified):

```cmake
cmake_minimum_required(VERSION 3.20)
project(freecad_host CXX)

set(CMAKE_CXX_STANDARD 17)

find_package(Python3 REQUIRED COMPONENTS Development)

add_executable(freecad_host src/main.cpp)

target_include_directories(freecad_host PRIVATE ${Python3_INCLUDE_DIRS})
target_link_libraries(freecad_host PRIVATE ${Python3_LIBRARIES})
```

Build:

```powershell
cmake -S . -B build -G "Visual Studio 17 2022" -A x64
cmake --build build --config Release
```

---

### 7.6 Windows runtime initialization (recommended sequence)

1. Determine `FREECAD_HOME`
2. Add `FREECAD_HOME\bin` to DLL search directories
3. Initialize Python
4. Set `sys.path`:
   - `./python` (your bridge)
   - `FREECAD_HOME\Mod`
   - optionally other FreeCAD python paths
5. Import bridge and run

Pseudo-code:

```cpp
// 1) read FREECAD_HOME
// 2) AddDllDirectory(FREECAD_HOME\bin)
Py_Initialize();
py_add_sys_path("./python");
py_add_sys_path(FREECAD_HOME + "\\Mod");
...
```

---

## 8) Linux Implementation Details

Linux is simpler because:
- shared library discovery is standardized
- FreeCAD packages and AppImage options exist

### 8.1 Strategy choices

**Strategy A (recommended):** Depend on distro FreeCAD and Python

- Install via package manager:
  - Debian/Ubuntu: `sudo apt install freecad`
  - Fedora: `sudo dnf install freecad`

Your app embeds `libpython3.x` and imports `FreeCAD` from the installed location.

**Strategy B:** Bundle a FreeCAD AppImage extraction
- Extract AppImage
- Set `PYTHONPATH`/`LD_LIBRARY_PATH` equivalent inside your launcher

---

### 8.2 Linux build (CMake)

Same `find_package(Python3 COMPONENTS Development)` approach.

Build:

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j
```

---

### 8.3 Linux runtime path setup

On Linux, `import FreeCAD` typically works if FreeCAD is installed and Python can find the module.

However, depending on distro packaging, you may need to add one or more paths:

- `/usr/lib/freecad/lib` or `/usr/lib/freecad/Mod`
- `/usr/lib64/freecad/lib` (on some distros)

Practical approach:
- Let user specify `FREECAD_HOME`
- Add `${FREECAD_HOME}/lib` and `${FREECAD_HOME}/Mod` to `sys.path`

LD library path is usually handled by system packaging.

---

## 9) Error Handling and Tracebacks

### 9.1 Always return structured JSON

- Python exceptions should be caught in the bridge and returned as warnings
- C++ exceptions should return a JSON object with `ok=false`

### 9.2 Capture Python tracebacks in C++ (optional)

If you want full tracebacks programmatically:
- import `traceback`
- call `traceback.format_exc()` in a Python helper

Simpler: call `PyErr_Print()` to stderr and return a generic JSON error.

---

## 10) Concurrency Model (Recommended)

### 10.1 Single worker per process

Because FreeCAD/OCCT operations are not designed for safe parallel mutation:

- Run a **single CAD worker thread** in your process
- Or better: run **multiple worker processes** each handling one job at a time

### 10.2 Multi-process job queue

- Host spawns N workers
- Each worker embeds Python and runs jobs
- Host distributes jobs via:
  - pipes
  - sockets
  - file queue
  - shared memory

This also isolates crashes.

---

## 11) Packaging and Distribution

### 11.1 Windows packaging (best practice)

- Bundle:
  - your exe
  - Python embeddable distribution
  - your `python/bridge` modules
  - FreeCAD runtime OR require installed FreeCAD

- Provide a launcher that:
  - sets `FREECAD_HOME`
  - sets DLL directories
  - runs your exe

### 11.2 Linux packaging

- If targeting servers/desktops:
  - depend on distro packages (simplest)
- If targeting portable distribution:
  - bundle AppImage extraction
  - ship wrapper script to set env vars

---

## 12) Security and Hardening Checklist

- Keep FreeCAD operations offline where required
- Log job inputs/outputs (redact sensitive content)
- Enforce an allowlist of:
  - actions
  - export formats
  - output directories
- Prevent arbitrary Python execution from user inputs
  - never `eval`
  - never pass raw code
- Use structured JSON contract only

---

## 13) Test Plan

### 13.1 Unit tests (bridge)

- Create box
- Export STEP
- Validate file exists
- Confirm JSON `ok=true`

### 13.2 Integration tests (host)

- Run job JSON end-to-end
- Validate output
- Run with missing FreeCAD (expect graceful error)

### 13.3 Regression tests

- Pin FreeCAD version(s) you support
- Run basic job suite for each version

---

## 14) Next Implementation Deliverables

If you want this turned into working starter code, the next step is producing:

1. Full CMake project with:
   - OS-specific runtime path logic
   - `FREECAD_HOME` discovery

2. A Windows-specific module:
   - `AddDllDirectory` setup
   - robust Python path configuration

3. A Linux-specific module:
   - `FREECAD_HOME` detection
   - sys.path setup

4. Sample jobs:
   - create box
   - import STEP, apply transform, export
   - mesh export

---

## Appendix A: Recommended OS-Specific Path Rules

### A.1 Windows

- Require `FREECAD_HOME` or probe common install paths
- Add DLL directory:
  - `${FREECAD_HOME}\bin`
- Add Python paths:
  - `./python`
  - `${FREECAD_HOME}\Mod`
  - (optionally) `${FREECAD_HOME}\lib`

### A.2 Linux

- Prefer distro install
- Optionally set `FREECAD_HOME`
- Add Python paths:
  - `./python`
  - `${FREECAD_HOME}/Mod`
  - `${FREECAD_HOME}/lib`

---

**End of report**

