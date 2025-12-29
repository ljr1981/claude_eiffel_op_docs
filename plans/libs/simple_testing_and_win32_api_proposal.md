# Proposal: simple_testing and simple_win32_api Libraries

## Executive Summary

This proposal outlines the consolidation of testing infrastructure into a `simple_testing` library and the creation of a `simple_win32_api` aggregator library that provides unified access to all Win32 wrapper functionality.

---

## Part 1: simple_testing Library

### Current State

The `testing_ext` library provides:
- `TEST_SET_BASE` - Extended assertions inheriting from EQA_TEST_SET
- `TEST_SET_BRIDGE` - Bridge class for compatibility

### Proposed Structure

```
simple_testing/
├── src/
│   ├── core/
│   │   ├── test_set_base.e          -- Current TEST_SET_BASE (assertions)
│   │   └── test_set_bridge.e        -- Bridge class
│   └── win32/
│       ├── test_console_helper.e    -- Console-aware test helpers
│       ├── test_clipboard_helper.e  -- Clipboard test utilities
│       └── test_process_helper.e    -- Process execution for tests
├── testing/
└── simple_testing.ecf
```

### ECF Targets

```xml
<!-- Core testing (platform-agnostic assertions) -->
<target name="simple_testing">
    <cluster name="core" location=".\src\core\" recursive="true"/>
</target>

<!-- Win32-specific testing capabilities -->
<target name="simple_testing_win32" extends="simple_testing">
    <library name="simple_console" location="$SIMPLE_CONSOLE\simple_console.ecf"/>
    <library name="simple_clipboard" location="$SIMPLE_CLIPBOARD\simple_clipboard.ecf"/>
    <library name="simple_process" location="$SIMPLE_PROCESS\simple_process.ecf"/>
    <cluster name="win32" location=".\src\win32\" recursive="true"/>
</target>
```

### Win32 Testing Features

1. **TEST_CONSOLE_HELPER**
   - Colored test output (PASS=green, FAIL=red)
   - Progress indicators
   - Clear screen between test suites

2. **TEST_CLIPBOARD_HELPER**
   - Clipboard backup/restore for tests
   - Safe clipboard testing without affecting user data

3. **TEST_PROCESS_HELPER**
   - Execute commands and capture output for verification
   - Timeout handling for hung processes
   - Environment variable manipulation for tests

---

## Part 2: simple_win32_api Library

### Purpose

Aggregate all Win32 wrapper libraries into a single convenient import, similar to how `simple_foundation_api` and `simple_service_api` work.

### Current Win32 Libraries (Existing + New)

| Library | Status | Purpose |
|---------|--------|---------|
| simple_process | DONE | Process execution, output capture |
| simple_env | DONE | Environment variables |
| simple_system | DONE | System info (CPU, memory, OS) |
| simple_console | DONE | Console colors, cursor control |
| simple_clipboard | DONE | Clipboard text access |
| simple_registry | PENDING | Windows registry access |
| simple_mmap | PENDING | Memory-mapped files |
| simple_ipc | PENDING | Named pipes / IPC |
| simple_watcher | PENDING | File system change monitoring |

### Proposed Structure

```
simple_win32_api/
├── src/
│   └── simple_win32.e    -- Optional: Facade class with common operations
├── testing/
└── simple_win32_api.ecf
```

### ECF Design

```xml
<system name="simple_win32_api" library_target="simple_win32_api">
    <target name="simple_win32_api">
        <library name="simple_process" location="$SIMPLE_PROCESS\simple_process.ecf"/>
        <library name="simple_env" location="$SIMPLE_ENV\simple_env.ecf"/>
        <library name="simple_system" location="$SIMPLE_SYSTEM\simple_system.ecf"/>
        <library name="simple_console" location="$SIMPLE_CONSOLE\simple_console.ecf"/>
        <library name="simple_clipboard" location="$SIMPLE_CLIPBOARD\simple_clipboard.ecf"/>
        <library name="simple_registry" location="$SIMPLE_REGISTRY\simple_registry.ecf"/>
        <library name="simple_mmap" location="$SIMPLE_MMAP\simple_mmap.ecf"/>
        <library name="simple_ipc" location="$SIMPLE_IPC\simple_ipc.ecf"/>
        <library name="simple_watcher" location="$SIMPLE_WATCHER\simple_watcher.ecf"/>
        <cluster name="src" location=".\src\" recursive="true"/>
    </target>
</system>
```

### Usage

Instead of:
```xml
<library name="simple_process" location="..."/>
<library name="simple_env" location="..."/>
<library name="simple_system" location="..."/>
<library name="simple_console" location="..."/>
```

Simply:
```xml
<library name="simple_win32_api" location="$SIMPLE_WIN32_API\simple_win32_api.ecf"/>
```

---

## Part 3: Implementation Plan

### Phase 1: Complete Win32 Libraries (Current Task)
1. ~~simple_env~~ DONE
2. ~~simple_system~~ DONE
3. ~~simple_console~~ DONE
4. ~~simple_clipboard~~ DONE
5. simple_registry (MODERATE)
6. simple_mmap (MODERATE)
7. simple_ipc (MODERATE)
8. simple_watcher (HARDER)

### Phase 2: Create simple_win32_api
1. Create aggregator ECF
2. Create optional SIMPLE_WIN32 facade class
3. Add tests verifying all libraries load correctly

### Phase 3: Create simple_testing
1. Migrate testing_ext content to simple_testing/src/core/
2. Create Win32 test helpers in simple_testing/src/win32/
3. Update all simple_* libraries to use simple_testing
4. Deprecate testing_ext (or make it re-export simple_testing)

### Phase 4: Integration
1. Update simple_foundation_api to optionally include simple_win32_api
2. Update simple_service_api as needed
3. Documentation updates

---

## Benefits

### For Developers
- Single import for all Win32 functionality
- Consistent SCOOP-compatible Win32 access
- Enhanced testing capabilities with Win32 features

### For Maintenance
- Centralized Win32 wrapper code
- Consistent patterns across all Win32 libraries
- Easier to add new Win32 features

### For Testing
- Rich console output for test results
- Safe clipboard testing
- Process-based integration testing

---

## Questions to Consider

1. Should simple_win32_api include ALL Win32 libraries, or offer subset targets?
   - `simple_win32_api` (all)
   - `simple_win32_api_core` (process, env, system only)
   - `simple_win32_api_ui` (console, clipboard only)

2. Should simple_testing Win32 helpers be optional?
   - Platform-agnostic core always available
   - Win32 features only on Windows

3. Naming convention: `simple_testing` vs `simple_test`?

---

## Recommendation

Proceed with the current Win32 library development (Phase 1), then create simple_win32_api as soon as all 8 libraries are complete. The simple_testing refactor can happen in parallel or after, as it's less urgent.

**Priority:**
1. HIGH: Finish simple_registry, simple_mmap, simple_ipc, simple_watcher
2. MEDIUM: Create simple_win32_api aggregator
3. MEDIUM: Create simple_testing from testing_ext
4. LOW: Win32 test helpers (nice-to-have)
