# simple_codegen Issues Found During simple_chart Build

**Date:** 2026-01-16
**Test Project:** simple_chart (terminal data visualization CLI)
**Result:** Project compiled and runs successfully after manual fixes

## Summary

simple_codegen successfully scaffolded a project and generated all classes, but required significant manual fixes before compilation. The issues fall into two categories:
1. **CLI/Assembly bugs** - Issues with simple_codegen itself
2. **Generated code issues** - API mismatches between generated code and actual simple_* library APIs

---

## Issue #1: SQLite KB Tracking Fails (Non-Fatal)

**Severity:** Low
**Type:** CLI bug

**Description:** When running `simple_codegen new`, the tool outputs an error about KB (knowledge base) tracking:

```
Error: Failed to update KB tracking
```

**Impact:** Non-fatal. Project creation continues successfully.

**Cause:** SQLite database connection or query issue in KB tracking module.

**Fix:** Investigate `scg_session.e` or KB tracking code. Consider making KB tracking optional or failing silently.

**Related:** This ties into Issue #3 below - simple_sql usage pattern issues.

---

## Issue #2: `assemble` Creates Nested `src/src/` Directory

**Severity:** Medium
**Type:** CLI bug
**Status:** FIXED

**Description:** When running `simple_codegen assemble`, if the user provided an output path ending with `/src`, the generated `.e` files were placed in:
```
simple_chart/src/src/*.e
```

Instead of:
```
simple_chart/src/*.e
```

**Root Cause:** Line 316 in `scg_session.e` used string concatenation (`a_output_path + "/src/"`) instead of proper path handling. If user provided `--output ./simple_chart/src`, it would create `src/src/`.

**Fix Applied:** Modified `assemble_project` in `scg_session.e`:
1. Sanitize output path: strip trailing `/src` or `\src` if present
2. Strip trailing slashes for consistency
3. Use `SIMPLE_PATH.add()` for proper path construction instead of string concatenation

**Files Changed:** `D:\prod\simple_code\src\cli\scg_session.e` lines 293-340

---

## Issue #3: simple_sql / SCG_KB Database Missing
**Severity:** Medium
**Type:** Deployment issue
**Status:** FIXED

**Description:** The `SCG_KB` class uses simple_sql for SQLite knowledge base storage. During simple_codegen operations, KB tracking silently failed because the KB database file was not deployed with the executable.

**Symptoms:**
- KB tracking during `simple_codegen new` silently skips
- `simple_codegen projects --stats` fails with precondition violation
- No "[KB] Project registered" message after project creation

**Root Cause:**
SCG_KB looks for the database at `<exe_dir>/data/scg_kb.db`. This file must be pre-populated with Eiffel knowledge base content (classes, features, patterns, examples). The database doesn't create itself - it needs to be deployed.

**Fix Applied:**
1. Create `data/` directory next to simple_codegen executable
2. Copy `kb.db` from simple_kb's W_code directory to `data/scg_kb.db`
   ```bash
   mkdir -p <exe_dir>/data/
   cp /d/prod/simple_kb/EIFGENs/kb_cli/W_code/kb.db <exe_dir>/data/scg_kb.db
   ```

**Deployment Note:**
The KB database must be included in simple_codegen installer/distribution. The database should be refreshed periodically using simple_kb's ingest command to keep API documentation current.

---

## Issue #4: Generated Code Has Wrong API Calls

**Severity:** High
**Type:** Claude knowledge gap / API documentation issue

This is the biggest problem. Claude (generating the class code) produces syntactically valid Eiffel but uses incorrect API calls for simple_* libraries.

### 3a. SIMPLE_CSV API Mismatch

**Generated (wrong):**
```eiffel
across l_csv as l_row loop
    l_label := l_row.item.field (l_x_idx)
end
```

**Actual API:**
```eiffel
l_csv.start_iteration
from until not l_csv.next_row loop
    l_label := l_csv.current_field (l_x_idx)
end
```

Claude assumed a `SIMPLE_CSV_ROW` class exists. It doesn't - iteration is done via methods on `SIMPLE_CSV` itself.

### 3b. SIMPLE_JSON_ARRAY Not ITERABLE

**Generated (wrong):**
```eiffel
across l_arr as ic loop
    if ic.item.is_number then ...
end
```

**Actual API:**
```eiffel
from i := 1 until i > l_arr.count loop
    if l_arr.item (i).is_number then ...
    i := i + 1
end
```

`SIMPLE_JSON_ARRAY` doesn't inherit from `ITERABLE` so `across` loops don't work.

### 3c. Method Name Mismatches

| Generated | Actual |
|-----------|--------|
| `as_real_64` | `as_real` |
| `set_has_header` | `make_with_header` |
| `is_valid` | (doesn't exist) |

### 3d. ARRAYED_LIST Iteration Pattern

`across` loops with `ic.item` sometimes fail on `ARRAYED_LIST`. Use index-based iteration:

```eiffel
from i := 1 until i > list.count loop
    ... list.i_th(i) ...
    i := i + 1
end
```

---

## Issue #5: Inheritance Constraints Not Considered

**Severity:** Medium
**Type:** Claude knowledge gap

**Description:** Generated `SIMPLE_CHART` class inherits from `ARGUMENTS_32` with:
```eiffel
inherit
    ARGUMENTS_32
        rename
            print as arguments_print
        end
```

But then uses `print (...)` calls throughout the code. Since `print` was renamed, this causes VEEN errors.

**Fix:** Replace `print` with `io.put_string` when inheriting from ARGUMENTS classes.

---

## Issue #6: No Claude Action Logging
**Severity:** High
**Type:** Missing feature
**Status:** IMPLEMENTED

**Description:** There's no mechanism for Claude to log what it's doing while following simple_codegen prompts. This makes it impossible to:
- Track gap-fills (actions Claude took because prompts were incomplete)
- Track rebellions (actions Claude took contrary to instructions)
- Perform after-action analysis to improve prompts

**Implementation:**
1. Created `SCG_ACTION_LOGGER` class at `src/cli/scg_action_logger.e`
2. Added `log` command handler to `SCG_CLI_APP`
3. Categories: INSTRUCTED, GAP_FILL, REBEL, ERROR_FIX, DECISION

**Usage:**
```bash
simple_codegen log --session <name> --category <CATEGORY> --action "description"
```

**Output:** JSON log file at `sessions/<name>/claude_actions.log`

**See:** `SIMPLE_CODEGEN_LOGGING_SPEC.md` for full specification.

---

## Recommendations for simple_codegen

### Short-term Fixes

1. ~~**Fix `assemble` path bug**~~ - DONE (Issue #2)
2. ~~**Make KB tracking work**~~ - DONE (Issue #3 - database deployment)

### Medium-term Improvements

3. **API Documentation Injection** - When generating classes that use simple_* libraries, include actual API signatures in the prompt context
4. **Validation Prompts** - After initial generation, validate against known API patterns and prompt for corrections
5. **Library Usage Examples** - Include example code snippets in prompts for each dependency

### Long-term Enhancements

6. **Simple_kb Integration** - Query simple_kb for actual class/feature signatures when generating code
7. **Compilation Feedback Loop** - After assemble, attempt compilation and feed errors back as refinement prompts
8. **Iteration Pattern Library** - Maintain patterns for each simple_* library's iteration idioms

---

## Test Commands Used

```bash
# Project creation
simple_codegen new simple_chart --type cli --lib simple_console --lib simple_csv --lib simple_json --lib simple_cli

# Session init
simple_codegen init --session simple_chart --level system

# Class generation (for each class)
simple_codegen process --session simple_chart --input response.json

# Assembly
simple_codegen assemble --session simple_chart

# Compilation
ec.sh -batch -config simple_chart.ecf -target simple_chart -c_compile
```

---

## Classes Generated

| Class | Purpose | Manual Fixes Required |
|-------|---------|----------------------|
| SIMPLE_CHART | Main CLI app | `print` → `io.put_string` |
| CHART_CONFIG | Configuration | None |
| CHART_DATA | Data container | Index-based iteration |
| CHART_DATA_LOADER | CSV/JSON/stdin loader | Full API rewrite for CSV/JSON |
| CHART_RENDERER | Base renderer | None |
| BAR_CHART_RENDERER | Bar chart output | None |
| SPARKLINE_RENDERER | Sparkline output | None |
| CHART_SCALER | Value scaling | None |
| UNICODE_BLOCKS | Block characters | None |

---

## Conclusion

simple_codegen successfully drives the Claude-in-the-loop workflow. The main gap is Claude's knowledge of actual simple_* library APIs. This could be addressed by:

1. Injecting API documentation into prompts
2. Using simple_kb to retrieve real signatures
3. Adding a validation/refinement step after initial generation

The `assemble` path bug should be an easy fix in the assembler code.
