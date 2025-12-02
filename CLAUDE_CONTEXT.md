# Claude Context File - READ FIRST

**Purpose**: This file provides essential context for Claude when working on any Eiffel project. Read this file before writing any Eiffel code.

---

## Session Startup Protocol

**At the start of each Eiffel session, Claude should:**

1. **Read this file** for general Eiffel knowledge
2. **Ask which project** we're working on (if not obvious)
3. **Look for ROADMAP.md** in the project root - this contains project-specific context
4. **If no ROADMAP.md exists**, offer to create one

---

## ROADMAP.md Convention

Each Eiffel project should have a `ROADMAP.md` in its root directory containing:

```markdown
# Project Name - Roadmap

## Project Overview
Brief description of what this project is.

## Current Status
What state is the project in? (active development, maintenance, stable, etc.)

## Dependencies
- Library dependencies and versions
- External tools required

## Build & Test Commands
- How to compile
- How to run tests

## Current Focus
What are we working on right now?

## Backlog / Future Work
Items to address later.

## Session Notes
Temporary notes for current work session.
```

---

## General Eiffel Knowledge

### Compiler Access

Claude can compile and test Eiffel projects via command line:

```batch
:: Compile (freeze with C compilation)
ec.exe -batch -config "project.ecf" -target target_name -c_compile -freeze

:: Run tests
ec.exe -batch -config "project.ecf" -target target_name -tests

:: Clean rebuild
ec.exe -batch -config "project.ecf" -target target_name -c_compile -freeze -clean
```

### Assertion Configuration in ECF

```xml
<option warning="warning">
    <assertions precondition="true" postcondition="true" check="true"
                invariant="true" loop="true" supplier_precondition="true"/>
</option>
```

### Contract Priority Order

When reviewing or adding contracts, work in this order:
1. **Class invariants** - What's always true about objects of this class?
2. **Loop invariants/variants** - Loop correctness and termination
3. **Check assertions** - Internal sanity checks within routines
4. **Preconditions/Postconditions** - Caller/callee guarantees

---

## Working Relationship

1. Claude writes code based on current knowledge + these reference docs
2. User compiles and tests (or Claude compiles via ec.exe -batch)
3. Errors are debugged collaboratively
4. Learnings are captured in this reference folder
5. Both benefit in future conversations

**Key principle**: Trust compiler errors over documentation when conflicts arise.

---

## Reference Files

| File | Contents |
|------|----------|
| `HATS.md` | Focused work modes ("hats") for specific tasks |
| `gotchas.md` | Generic Eiffel doc-vs-reality conflicts |
| `sqlite_gotchas.md` | SQLite/database-specific gotchas |
| `across_loops.md` | Iteration patterns and cursor behavior |
| `scoop.md` | SCOOP concurrency model |
| `profiler.md` | EiffelStudio profiler usage |
| `patterns.md` | Verified working code patterns |

---

## EIS (Eiffel Information System) Quick Reference

Bidirectional linking between Eiffel source and external documentation:

```eiffel
-- Outgoing link (source → docs):
note
    EIS: "name=API Reference", "src=../docs/api.html", "protocol=URI", "tag=documentation"

-- Incoming link (docs → source) in HTML:
<a href="eiffel:?class=MY_CLASS&feature=my_feature">View Source</a>
```

---

## Update Log

| Date | Change |
|------|--------|
| 2025-12-02 | Genericized - removed project-specific refs, added ROADMAP.md convention |
| 2025-12-02 | Added profiler.md reference, expanded scoop.md |
| 2025-12-01 | Added EIS documentation patterns |
| 2025-11-29 | Initial structure created |
