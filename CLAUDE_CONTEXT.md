# Claude Context File - READ FIRST

**Purpose**: This file provides essential context for Claude at the start of each new conversation. Read this file before writing any Eiffel code.

---

## Critical Corrections (Compiler-Verified)

### Across Loops
- Status: UNDER INVESTIGATION
- Official docs say `ic.item` accesses elements
- User reports needing to remove `.item` in practice
- **Action**: When writing across loops, be prepared for corrections and document findings

```eiffel
-- Official syntax (may need adjustment):
across my_list as ic loop
    print (ic.item)
end
```

---

## Project Context

- Main repository: `D:\prod\simple_sql`
- SQLite wrapper: `D:\prod\eiffel_sqlite_2025` (v1.0.0, SQLite 3.51.1 with FTS5)
- Reference docs: `D:\prod\reference_docs\`
- User has hands-on Eiffel experience and compiler access
- Trust compiler errors over documentation when conflicts arise
- **Important**: simple_sql depends on eiffel_sqlite_2025 v1.0.0+ for FTS5 and modern SQLite features

---

## Working Relationship

1. Claude writes code based on current knowledge
2. User compiles and tests
3. Errors are debugged collaboratively
4. Learnings are captured in this reference folder
5. Both benefit in future conversations

---

## Key Files to Check

| File | Contents |
|------|----------|
| `HATS.md` | Focused work modes - tell Claude which "hat" to wear for specific tasks |
| `gotchas.md` | Known doc-vs-reality conflicts (including EIS annotations) |
| `across_loops.md` | Iteration patterns |
| `scoop.md` | SCOOP concurrency model - separate keyword, processors, wait conditions |
| `profiler.md` | EiffelStudio profiler usage and configuration |
| `patterns.md` | Verified working code |

---

## Operational Notes

**Always read this reference documentation** when starting work on any Eiffel project. Key capabilities:

1. **Compilation**: Can compile via `ec.exe -batch` command line
2. **Testing**: Can run tests via `ec.exe -batch ... -tests`
3. **Profiling**: Can enable profiling in ECF, but full analysis requires EiffelStudio GUI
4. **SCOOP**: Can implement concurrency using `separate` keyword (see `scoop.md`)

## EIS (Eiffel Information System) Quick Reference

When adding documentation to Eiffel classes:
```eiffel
note
    EIS: "name=API Reference", "src=../docs/api/myclass.html", "protocol=URI", "tag=documentation"
```

HTML links back to EiffelStudio:
```html
<a href="eiffel:?class=MY_CLASS&feature=my_feature">View Source</a>
```

---

## Update Log

| Date | Change |
|------|--------|
| 2025-12-02 | Added profiler.md reference, expanded scoop.md with comprehensive docs |
| 2025-12-02 | Added Operational Notes section with compilation/testing capabilities |
| 2025-12-01 | Added EIS (Eiffel Information System) documentation patterns |
| 2025-11-30 | Added eiffel_sqlite_2025 dependency, SQLite 3.51.1 context |
| 2025-11-29 | Initial structure created |

