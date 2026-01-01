# Simple Zig Design Document

## Overview

**Goal**: Use Zig as an alternative C compiler for EiffelStudio-generated code, eliminating MSVC dependency on Windows.

**Status**: SHELVED - POC complete, not adopted (binary size unacceptable)

## Phase 1: POC (Complete)

**Library**: simple_uuid (no inline C)
**Result**: 502 C files compiled, 45/45 tests pass

Key discoveries:
- Zig 0.13.0 compatible with EiffelStudio C code
- `eif_config.mingw.h` required for `ui64` → `ULL` suffix conversion
- MSVC-compiled objects (esize.obj, eskelet.obj) link with Zig output

## Phase 2: Inline C Compatibility (Complete)

### Test Results

| Library | C Files | Inline C | Clib .c | Tests | Result |
|---------|---------|----------|---------|-------|--------|
| simple_uuid | 21 | No | No | 45/45 | ✅ PASS |
| simple_console | 33 | Yes (Win32) | No | 31/31 | ✅ PASS |
| simple_process | 31 | Yes (Win32) | Yes | 17/17 | ✅ PASS |
| simple_sql | 35 | Yes (SQLite) | Yes + .obj | 46/46 | ✅ PASS |

**Total: 139 tests passed, 0 failed**

### Inline C Patterns Tested

1. **Pure inline C (simple_console)**
   - Windows.h console APIs (GetStdHandle, SetConsoleTextAttribute, etc.)
   - All inline - no separate .c files

2. **Separate Clib/*.c (simple_process)**
   - Complex C functions in simple_process.c
   - Compiled separately with Zig, linked together

3. **External .obj linking (simple_sql)**
   - Pre-compiled sqlite3.obj (MSVC)
   - esqlite.c wrapper compiled with Zig
   - Hybrid MSVC + Zig linking works!

### Key Findings

1. **eif_config.mingw.h required** - for ui64→ULL suffix conversion
2. **MSVC-compiled objects compatible** - esize.obj, eskelet.obj, sqlite3.obj all link with Zig
3. **MSVC .lib files compatible** - mtfinalized.lib, mtnet.lib, mtwel.lib all work
4. **Size difference** - Zig executables ~5x larger (static C runtime)

### Libraries with Inline C (15 total)

**Heavy inline C (10+ externals):**
- simple_audio (18) - Windows audio APIs ✅ TESTED (17/17)
- simple_ffmpeg (14) - FFmpeg wrapper
- simple_ipc (21) - Unix domain sockets, Windows named pipes
- simple_process (19) - Process management ✅ TESTED (17/17)
- simple_serial (14) - Serial port communication ✅ TESTED (6/6)
- simple_usb (20) - USB device access
- simple_watcher (11) - File system watching

**Light inline C (1-10 externals):**
- simple_clipboard - Clipboard access
- simple_console - Console/terminal operations ✅ TESTED
- simple_env - Environment variables
- simple_lsp (2) - Language Server Protocol
- simple_mmap - Memory-mapped files
- simple_oracle (1) - Oracle system
- simple_registry - Windows registry
- simple_system - System information

## Compilation Requirements

```bash
zig cc -c source.c \
  -include "$ISE/studio/spec/win64/include/eif_config.mingw.h" \
  -I"$ISE/studio/spec/win64/include" \
  -I<project_clib> \
  -D_WIN32_WINNT=0x0500 -DWINVER=0x0500 \
  -D_CRT_SECURE_NO_DEPRECATE -D_UNICODE -DUNICODE \
  -DWIN32_LEAN_AND_MEAN -DEIF_THREADS \
  -target x86_64-windows-msvc
```

## Tools Created

| Tool | Purpose |
|------|---------|
| `/d/prod/tools/zig/zig_build.py` | **Unified build** - finalize, compile, link, test |
| `/d/prod/tools/zig/compile_fcode.py` | Compile F_code directory with Zig |
| `/d/prod/tools/zig/link_fcode.py` | Link compiled objects with Zig |
| `/d/prod/tools/zig/add_inline_c_faqs.py` | Add inline C FAQs to kb.db |

## Phase 3: Unified Build System (Complete)

### Test Results

| Library | C Files | Tests | Result |
|---------|---------|-------|--------|
| simple_uuid | 21 | 45/45 | ✅ PASS |
| simple_console | 33 | 31/31 | ✅ PASS |
| simple_process | 31 | 17/17 | ✅ PASS |
| simple_sql | 35 | 46/46 | ✅ PASS |
| simple_audio | 27 | 17/17 | ✅ PASS |
| simple_serial | 31 | 6/6 | ✅ PASS |

**Total: 6 libraries, 162 tests passed, 0 failed**

### Unified Build Script

**Usage:**
```bash
python3 /d/prod/tools/zig/zig_build.py <library_path> [target_name]

# Examples:
python3 /d/prod/tools/zig/zig_build.py /d/prod/simple_console
python3 /d/prod/tools/zig/zig_build.py /d/prod/simple_sql simple_sql_tests
```

**Features:**
- Auto-detects ECF and test target
- Finds Clib directories and .c/.obj files
- Runs EiffelStudio finalize + finish_freezing
- Compiles all C code with Zig
- Links and runs tests

### Size Analysis

| Library | MSVC | Zig | Zig -O2 -s | Ratio |
|---------|------|-----|------------|-------|
| simple_audio | 1.6 MB | 7.9 MB | 6.7 MB | 4.2x |
| simple_console | 2.1 MB | 11.4 MB | ~9.7 MB | 4.6x |
| simple_process | 1.8 MB | 9.0 MB | ~7.6 MB | 4.2x |
| simple_sql | 3.5 MB | 13.9 MB | ~11.8 MB | 3.4x |

**Findings:**
- `-O2 -s` reduces size by ~15%
- Zig statically links C runtime (MSVC uses dynamic msvcrt.dll)
- Size acceptable for development; production can use MSVC if needed

## EiffelStudio Integration

### Workflow

1. **Development**: Use normal EiffelStudio workflow (melting/freezing)
2. **MSVC-free Build**: Run `zig_build.py` for CI or MSVC-free environments
3. **Production**: Choose based on needs:
   - MSVC: Smaller binaries, Windows SDK required
   - Zig: Larger binaries, no external dependencies

### CI/CD Integration

```yaml
# Example GitHub Actions
- name: Build with Zig
  run: |
    python3 /d/prod/tools/zig/zig_build.py /d/prod/simple_console
```

### ECF Requirements

All simple_* libraries should use `$SIMPLE_EIFFEL` for dependencies:
```xml
<library name="simple_foo" location="$SIMPLE_EIFFEL\simple_foo\simple_foo.ecf"/>
```

**NOT:**
```xml
<library name="simple_foo" location="$SIMPLE_FOO\simple_foo.ecf"/>  <!-- WRONG -->
```

## Future Work

1. **Test remaining libraries**: simple_ipc, simple_usb, simple_watcher, simple_ffmpeg
2. **Cross-platform**: Test Zig on Linux/macOS
3. **Automation**: Integrate with EiffelStudio ec.sh wrapper
4. **Size optimization**: Investigate LTO and other techniques
