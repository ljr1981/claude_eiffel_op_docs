# Eiffel Profiler Documentation

## Overview

The EiffelStudio profiler instruments compiled Eiffel code with C-level profiling code to collect performance data during execution. When the instrumented program runs, it generates a `profinfo` file containing raw timing data. EiffelStudio's Profiler Wizard then parses this into a visual GUI for analysis.

**Metrics collected:**
- Number of calls to each feature
- Time spent in each feature (self-time)
- Time spent in called features (descendant-time)
- Percentage of total execution time

## Enabling Profiling

### In ECF File

Add the `profile` attribute to your target's `<option>` element:
```xml
<option warning="warning" profile="true">
    <assertions .../>
</option>
```

**Note:** This is an attribute on `<option>`, NOT a `<setting>` element (verified via ECF schema 1-23-0).

### In EiffelStudio GUI

1. Project Settings > Target > Advanced Options
2. Check "Profile" checkbox
3. Recompile (freeze or finalize)

## Recommended Setup: Dedicated Profile Target

Create a dedicated target for profiling to keep the `profile="true"` setting isolated:

```xml
<target name="my_app_profile" extends="my_app">
    <root class="MY_PROFILE_APP" feature="make"/>
    <option warning="warning" profile="true">
        <assertions precondition="true" postcondition="true" check="true" invariant="true" loop="true" supplier_precondition="true"/>
    </option>
    <setting name="inlining" value="false"/>
    <setting name="inlining_size" value="0"/>
    <cluster name="profile_src" location=".\src\my_app\profile\" recursive="true"/>
</target>
```

**Key points:**
- Extends the main app target
- Has a dedicated profile harness app (non-interactive, exercises code paths)
- Disables inlining for potentially better call tree visibility
- Lives in its own cluster (excluded from main app via file_rule)

### Profile Harness App

Create a standalone application that:
- Runs without user interaction
- Exercises representative code paths (like tests, but without assertions)
- Exits cleanly (so profinfo gets written)
- Prints progress to console

See `simple_sql/src/wms/profile/wms_profile_app.e` for an example.

## Compilation Modes

| Mode | Use Case | profinfo Location |
|------|----------|-------------------|
| Freeze | Development profiling | `EIFGENs/<target>/W_code/` |
| Finalize | Production profiling | `EIFGENs/<target>/F_code/` |

**Note:** Finalized mode generates profinfo (e.g., `profinfo_AC`, `profinfo_C4` - the suffix varies). Frozen mode may NOT generate profinfo via batch compilation.

## Profiling Workflow

### Via Command Line (Finalized)

```batch
:: 1. Clean finalize compile
ec.exe -batch -config project.ecf -target my_profile -c_compile -finalize -clean

:: 2. Run the instrumented executable
EIFGENs\my_profile\F_code\my_app.exe

:: 3. Check for profinfo file
dir EIFGENs\my_profile\F_code\profinfo*
```

### Via EiffelStudio GUI

1. Open project, select profile target
2. Finalize compile
3. Run (F5)
4. After exit: Project menu → Profile
5. Profiler Wizard:
   - Select "Finalized mode"
   - Browse to profinfo file in F_code
   - Select profiler: "eiffel"
   - Configure output (Calls, Self, Descendants, Total, Percentage)
   - Query: `calls > 0`

## Profiler Query Window

| Option | Description |
|--------|-------------|
| Nest Output? | Tree view (checked) vs flat list (unchecked) |
| Eiffel Features | Show Eiffel feature names |
| C Functions | Show underlying C function names |
| Query | Filter, e.g., `calls > 0`, `percentage > 5` |

## Known Limitations

### 1. AutoTest Does Not Generate Useful Profile Data

Running profiled test executables (AutoTest) only captures the root `make` feature. The test harness launches tests in a way that bypasses profiling instrumentation.

**Solution:** Create a dedicated profile harness app that exercises code paths directly.

### 2. Shallow Call Tree in Finalized Mode (UNRESOLVED)

**Status:** Under investigation - ask Eiffel Google Group for help.

**Symptoms:** Profiler only shows features from the root class (e.g., `WMS_PROFILE_APP`), not features from called classes (`WMS_APP`, `SIMPLE_SQL_DATABASE`, etc.).

**Attempted fixes that did NOT help:**
- Disabling inlining (`<setting name="inlining" value="false"/>`)
- Setting inlining_size to 0
- Unchecking "Nest Output?" in Profiler Query Window
- Checking "C Functions" in Language type

**The profiler works** (generates valid profinfo, shows timing data), but only for top-level features in the root class cluster.

### 3. Frozen Mode May Not Generate profinfo (Batch)

Freeze-compiled executables run via batch command line may not generate profinfo files, even with `profile="true"`. Finalized mode works.

### 4. Segmentation Fault on Startup

**Likely Cause:** Corrupted EIFGENs from toggling profiling on/off.

**Solution:** Clean compile with `-clean` flag.

### 5. profinfo File Has Variable Suffix

The profinfo file may be named `profinfo_AC`, `profinfo_C4`, etc. - not just `profinfo`. Check the directory listing.

## Profiler Configuration Files

Located in: `$ISE_EIFFEL/studio/profiler/`

| File | Purpose |
|------|---------|
| `eiffel` | Internal Eiffel profiler configuration |
| `gprof` | GNU gprof external profiler configuration |
| `win32_ms` | Windows MS profiler configuration |

## Alternative Approaches

For cases where the built-in profiler is insufficient:

- **Manual timing**: Use `TIME` class for targeted measurements
- **valgrind/callgrind** (Linux): Use with C-compiled Eiffel
- **External profilers**: Configure via custom profiler configuration files

## Claude Code Integration

**What Claude can do:**
- Enable/disable profiling in ECF files
- Create profile targets and harness apps
- Compile and run finalized profiled executables
- Discuss profiling results you share

**What requires EiffelStudio GUI:**
- Converting `profinfo` to `.pfi` via Profiler Wizard
- Interactive profile analysis and querying

---

## Quick Reference

### Enable Profiling
```xml
<option profile="true">
```

### Compile and Run (Finalized)
```batch
ec.exe -batch -config project.ecf -target my_profile -c_compile -finalize -clean
EIFGENs\my_profile\F_code\my_app.exe
```

### Analyze in EiffelStudio
Project menu → Profile → Finalized mode → Browse to profinfo_XX → eiffel → Finish

### Disable Profiling
Tell Claude to remove `profile="true"` from the ECF.

---

## Update Log

| Date | Change |
|------|--------|
| 2025-12-02 | Added dedicated profile target pattern with WMS_PROFILE_APP example |
| 2025-12-02 | Documented shallow call tree issue (unresolved) - escalate to Eiffel Google Group |
| 2025-12-02 | Confirmed finalized mode generates profinfo, frozen mode (batch) may not |
| 2025-12-02 | Noted profinfo files have variable suffixes (profinfo_AC, profinfo_C4, etc.) |
| 2025-12-02 | Added inlining disable settings (did not resolve shallow call tree) |
| 2025-12-02 | Initial documentation |

---

## Session Log: 2025-12-02 Profiler Investigation

### Environment
- EiffelStudio 25.02 Standard (win64)
- Windows 11
- Project: simple_sql (SQLite wrapper library with WMS mock application)

### Goal
Get deep call tree profiling showing features in `WMS_APP`, `SIMPLE_SQL_DATABASE`, etc. - not just the root application class.

### What We Tried

#### Attempt 1: Profile on wms_tests target (AutoTest)
- Added `profile="true"` to `wms_tests` target
- Ran tests via AutoTest
- **Result:** No profinfo generated. AutoTest bypasses profiling.

#### Attempt 2: Profile on various targets via batch compile
- Tried `profile="true"` on `simple_sql`, `wms`, `wms_tests`
- Used `ec.exe -batch -freeze -c_compile`
- **Result:** Segmentation fault on startup (corrupted EIFGENs likely)

#### Attempt 3: Created dedicated wms_profile target
- New target extending `wms` with `profile="true"`
- Created `WMS_PROFILE_APP` - non-interactive harness exercising:
  - 5 warehouses, 100 products, 200 locations
  - 300 stock receive operations
  - 50 stock transfers
  - 30 reservations (create/release)
  - Extensive query operations
- **Result (finalize):** Generated `profinfo_AC` in F_code ✓

#### Attempt 4: Analyze in Profiler Wizard
- Selected Finalized mode, browsed to profinfo_AC, profiler "eiffel"
- **Result:** Only shows `WMS_PROFILE_APP` features (make, run_stock_operations, etc.)
- No visibility into `WMS_APP.receive_stock`, `SIMPLE_SQL_DATABASE.execute`, etc.

#### Attempt 5: Disable inlining
- Added to ECF:
  ```xml
  <setting name="inlining" value="false"/>
  <setting name="inlining_size" value="0"/>
  ```
- Recompiled finalized with clean
- **Result:** Same shallow call tree. Inlining wasn't the cause.

#### Attempt 6: Try different Profiler Query options
- Unchecked "Nest Output?" - got flat list but still only root class features
- Checked "C Functions" - no additional features revealed
- **Result:** No improvement

### Final State
- `wms_profile` target exists with `profile="true"` and inlining disabled
- `WMS_PROFILE_APP` harness created at `src/wms/profile/wms_profile_app.e`
- Profiler generates valid profinfo and shows timing data
- **UNRESOLVED:** Only root class features visible, no call tree depth

### Profiler Output (Final Attempt)
```
Function                                    Calls   Self      Descendants  Total     Percentage
WMS_PROFILE_APP.run_stock_operations        1       0.03125   0.0          0.03125   100.0
WMS_PROFILE_APP.run_query_operations        1       0.0       0.0          0.0       0.0
WMS_PROFILE_APP.run_product_setup           1       0.0       0.0          0.0       0.0
WMS_PROFILE_APP.run_location_setup          1       0.0       0.0          0.0       0.0
... (only WMS_PROFILE_APP features shown)
```

### Next Steps
- Ask Eiffel Google Group: "How to get deep call tree in EiffelStudio profiler?"
- Mention: EiffelStudio 25.02, finalized mode, inlining disabled, profinfo generated correctly but only shows root class features

---

## Sources

- [Profile Menu](https://www.eiffel.org/doc/eiffelstudio/Profile_menu)
- [Advanced Options](https://www.eiffel.org/doc/eiffelstudio/Advanced_Options)
- [How to set up a Profiler Configuration File](https://www.eiffel.org/doc/eiffelstudio/How_to_set_up_a_Profiler_Configuration_File)
- [Reuse or Generate an Execution Profile](https://www.eiffel.org/doc/eiffelstudio/Reuse_or_Generate_an_Execution_Profile)
