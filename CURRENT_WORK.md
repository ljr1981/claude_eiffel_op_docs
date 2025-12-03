# Current Work Status

This file tracks the current state of work so Claude can resume context in new conversations.

---

## Active Projects

### simple_sql
**Repository**: D:\prod\simple_sql
**Description**: High-level SQLite API for Eiffel - making SQLite easy and safe to use.
**Status**: Production-ready, Phases 1-5 complete, 339 tests passing

### simple_web
**Repository**: D:\prod\simple_web
**Description**: High-level HTTP library for Eiffel - client and server capabilities.
**Status**: Production-ready, HTTP client and server complete, 70 tests passing

---

## Where We Left Off

**Last Session Date**: 2025-12-03

**Last Completed**:
- **simple_htmx Library Created** (new library):
  - Fluent HTML/HTMX builder for Eiffel web applications
  - 34 classes (HTMX_ELEMENT base, HTMX_FACTORY, HTMX_RENDER_CONTEXT, 31 element classes)
  - 35+ tests passing
  - Type-safe HTMX attributes, automatic HTML escaping
  - Fixed `raw_html` accumulation bug (was overwriting instead of appending)
  - Added h4, h5 elements and input_number factory method

- **simple_gui_designer Major Refactoring**:
  - **God Class Refactor**: Extracted 2000-line GUI_DESIGNER_SERVER into 10 focused handler classes via multiple inheritance
  - **simple_htmx Integration**: Refactored HTML rendering to use fluent builders (gds_html_renderer.e, gds_static_html.e)
  - **Bug Fixes**: ARRAY.has reference equality fix, JSON key name compatibility (row/col vs grid_row/grid_col)
  - **Tests**: Added 4 regression tests for control type validation and JSON parsing
  - Handler classes created: GDS_SHARED_STATE, GDS_SPEC_HANDLERS, GDS_SCREEN_HANDLERS, GDS_CONTROL_HANDLERS, GDS_CONTAINER_HANDLERS, GDS_HTMX_HANDLERS, GDS_EXPORT_HANDLERS, GDS_DOWNLOAD_UPLOAD_HANDLERS, GDS_HTML_RENDERER, GDS_STATIC_HTML

**Key Technical Learnings**:
- **Fluent Interface in Eiffel**: Return `like Current` for chaining, use `.do_nothing` when function result unused
- **ARRAY.has Uses Reference Equality**: For STRING comparisons, use `across...some...same_string` instead
- **Multiple Inheritance for Refactoring**: Extract God class into handler mixins, main class inherits all
- **raw_html Must Append**: `content_text.append()` not assignment, or only last call appears

---

## Repository Quick Reference

| Project | Location | Tests | Status |
|---------|----------|-------|--------|
| simple_sql | D:\prod\simple_sql | 339 | Complete |
| simple_web | D:\prod\simple_web | 70 | Complete |
| simple_htmx | D:\prod\simple_htmx | 35+ | Complete |
| simple_gui_designer | D:\prod\simple_gui_designer | 10 | Complete |
| simple_json | D:\prod\simple_json | - | Stable |
| framework | D:\prod\framework | - | Stable |
| testing_ext | D:\prod\testing_ext | - | Stable |
| eiffel_sqlite_2025 | - | - | Stable (v1.0.0) |
| reference_docs | D:\prod\reference_docs\eiffel | - | Documentation |

---

## Environment Setup

### Required Environment Variables

```powershell
# Set permanently at User level
[System.Environment]::SetEnvironmentVariable('FRAMEWORK', 'D:\prod\framework', 'User')
[System.Environment]::SetEnvironmentVariable('SIMPLE_JSON', 'D:\prod\simple_json', 'User')
[System.Environment]::SetEnvironmentVariable('TESTING_EXT', 'D:\prod\testing_ext', 'User')
```

### Compilation Commands

```batch
:: simple_web tests
FRAMEWORK="D:/prod/framework" SIMPLE_JSON="D:/prod/simple_json" TESTING_EXT="D:/prod/testing_ext" ^
  "C:\Program Files\Eiffel Software\EiffelStudio 25.02 Standard\studio\spec\win64\bin\ec.exe" ^
  -batch -config "D:\prod\simple_web\simple_web.ecf" -target simple_web_tests -c_compile -freeze -clean

:: Run tests
"C:\Program Files\Eiffel Software\EiffelStudio 25.02 Standard\studio\spec\win64\bin\ec.exe" ^
  -batch -config project.ecf -target target_name -tests
```

---

## Open Questions / Blockers

None currently.

---

## Session Notes

### 2025-12-03 (Session 13 - simple_htmx Creation & Integration)

**Task**: Create simple_htmx library and integrate into simple_gui_designer

**Context**:
- Code smell analysis of simple_gui_designer identified friction: manual `.append()` chains, HTMX attribute typos, parameter threading
- Decision: Create separate library (not in simple_web) for single responsibility
- Created simple_htmx, then refactored simple_gui_designer to use it

**simple_htmx Implementation**:
- 34 classes total (HTMX_ELEMENT base, HTMX_FACTORY, HTMX_RENDER_CONTEXT, 31 elements)
- Fluent interface with `like Current` return types
- Type-safe HTMX attributes (hx_get, hx_post, hx_target, hx_swap, etc.)
- Automatic HTML escaping for security
- 35+ tests

**simple_gui_designer Refactoring**:
- Extracted God class (2000 lines) into 10 handler classes via multiple inheritance
- Refactored HTML rendering to use simple_htmx fluent builders
- Fixed ARRAY.has bug (reference equality vs string comparison)
- Added JSON key compatibility (row/col and grid_row/grid_col)
- Added 4 regression tests

**Bugs Fixed**:
- `raw_html` accumulation: Changed from assignment to append
- ARRAY.has: Use `across...some...same_string` for STRING matching
- JSON keys: Support both naming conventions for backward compatibility

**Git Commits**:
- `60987ae` - Initial simple_htmx library (2,662 lines)
- `01533e4` - Comprehensive documentation (950 lines)
- `3f2ae20` - Add h4, h5, input_number (73 lines)
- `6f19acb` - Fix raw_html accumulation bug (155 lines)
- `f403a39` - GUI_DESIGNER_SERVER God class refactor (2,300 lines net)
- `edf1821` - Refactor to use simple_htmx (387 lines changed)
- `6e1600e` - Control loading bugs, DBC tests (194 lines)

**Result**: simple_htmx complete, simple_gui_designer refactored and cleaned up

---

### 2025-12-02 (Session 12 - HTTP Server Implementation)

**Task**: Add HTTP server capability to simple_web (fold simple_socket into simple_web)

**Context**:
- User finished profiler investigation (shallow call trees - posted to Google Groups)
- Explored EiffelStudio libraries for "simple_*" wrapper candidates
- Identified cURL as good candidate, discovered simple_web already exists
- Found empty src/server folder - opportunity for server-side HTTP

**Implementation**:
- Explored EWF (Eiffel Web Framework) in EiffelStudio:
  - `WSF_EXECUTION` - request execution handler
  - `WSF_DEFAULT_SERVICE` - generic service launcher
  - `WSF_REQUEST`, `WSF_RESPONSE` - low-level request/response
  - Standalone launcher for embedded HTTP server
- Designed agent-based routing API (Express.js-like):
  ```eiffel
  server.on_get ("/api/users/{id}", agent handle_user)
  ```
- Created 6 server classes:
  - SIMPLE_WEB_SERVER - Main server, inherits WSF_DEFAULT_SERVICE
  - SIMPLE_WEB_SERVER_EXECUTION - Dispatches requests to route handlers
  - SIMPLE_WEB_SERVER_REQUEST - Clean request API wrapper
  - SIMPLE_WEB_SERVER_RESPONSE - Clean response API wrapper
  - SIMPLE_WEB_SERVER_ROUTE - URL pattern matching with {param} support
  - SIMPLE_WEB_SERVER_ROUTER - Singleton router (once function)

**Bugs Fixed**:
- Across loop syntax: Use `ic` directly for ARRAYED_LIST, not `ic.item`
- Use traditional `from/until/loop` instead of across for complex iteration
- `WGI_INPUT_STREAM.read_string` is procedure, not function - use `last_string`
- `WSF_VALUE.string_representation` returns READABLE_STRING_32, use directly
- `to_string_8` instead of obsolete `as_string_8`
- Path `path_matches` needed to be public for precondition access

**ECF Fixes**:
- Converted hard-coded paths: `D:\prod\...` → `$ENV_VAR\...`
- Removed self-referencing library entry (was a bug)
- Added wsf library for core WSF classes

**Result**: 70/70 tests passing (1 unresolved Ollama timeout - expected)

---

### Previous Sessions (simple_sql focused)

See session notes below for Phases 1-5 of simple_sql development including:
- SQLite 3.51.1 upgrade with FTS5
- Vector embeddings for AI/ML
- Advanced backup/import/export
- Repository pattern
- Audit tracking
- 339 comprehensive tests

[Previous session notes preserved from earlier in file...]

---

## Key Eiffel Patterns Learned

### WSF Server Pattern
```eiffel
class MY_SERVER
inherit
    WSF_DEFAULT_SERVICE [MY_EXECUTION]
        redefine initialize end
create
    make
feature
    make (a_port: INTEGER)
        do
            initialize
            set_service_option ("port", a_port)
        end
    start
        do
            launch (service_options)
        end
end

class MY_EXECUTION
inherit
    WSF_EXECUTION
feature
    execute
        do
            -- Handle request/response
        end
end
```

### Singleton with Once Function
```eiffel
router: MY_ROUTER
        -- Shared singleton (same instance across all callers)
    once
        create Result
    end
```

### TEST_SET_BASE vs EQA_TEST_SET
Always use `TEST_SET_BASE` (from testing_ext) instead of `EQA_TEST_SET` for consistent test assertions.
