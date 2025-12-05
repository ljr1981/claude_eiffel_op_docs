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

Claude can compile and test Eiffel projects via command line.

**EiffelStudio ec.exe location** (Git Bash path):
```
"/c/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/studio/spec/win64/bin/ec.exe"
```

**CRITICAL**: Always use `-batch` flag for non-interactive compilation.

**Environment Variables**: New libraries need their env var set. Use `setx` for Windows:
```bash
# Set persistent Windows environment variable
"/c/Windows/System32/setx.exe" SIMPLE_ALPINE "D:\\prod\\simple_alpine"

# Then export for current session
export SIMPLE_ALPINE="D:\\prod\\simple_alpine"
```

**Compile commands** (run from project directory):
```bash
# Compile (freeze with C compilation) - ALWAYS use -batch
"/c/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/studio/spec/win64/bin/ec.exe" \
  -batch -config project.ecf -target target_name -c_compile

# Clean rebuild (delete EIFGENs first)
rm -rf EIFGENs && "/c/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/studio/spec/win64/bin/ec.exe" \
  -batch -config project.ecf -target target_name -c_compile

# Run tests
"/c/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/studio/spec/win64/bin/ec.exe" \
  -batch -config project.ecf -target target_name -tests
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

## AI + Verification: From Probable to Provable

Based on Bertrand Meyer's "AI for software engineering: from probable to provable" (CACM 2025):

### The Core Problem

AI produces *statistically likely* code, not *proven correct* code. With N modules at 99.9% correctness each:
- 1000 modules → 37% system correctness
- 5000 modules → <1% system correctness

**Solution**: Combine AI generation with formal verification via Design by Contract.

### The Process: Vibe-Contracting + Vibe-Coding

1. **Specification Hat** - Write contracts BEFORE implementation
2. **Feature Hat** - Implement to satisfy contracts
3. **Verify** - Compiler (types), Runtime (contracts), [Future: AutoProof (proofs)]
4. **Iterate** - Fix specification OR implementation on failure

### The "True but Incomplete" Trap

AI-generated contracts are often *true* but miss important guarantees:

```eiffel
-- INCOMPLETE (AI might generate this)
ensure
  has_item: items.has (a_item)

-- COMPLETE (what we need)
ensure
  has_item: items.has (a_item)
  count_increased: items.count = old items.count + 1
```

**Always ask: "What ELSE is guaranteed?"**

See `verification_process.md` and `contract_patterns.md` for details.

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
| `verification_process.md` | AI + DBC iterative workflow (Meyer's "probable to provable") |
| `contract_patterns.md` | Complete postcondition templates for common operations |
| `gotchas.md` | Generic Eiffel doc-vs-reality conflicts |
| `sqlite_gotchas.md` | SQLite/database-specific gotchas |
| `across_loops.md` | Iteration patterns and cursor behavior |
| `scoop.md` | SCOOP concurrency model |
| `profiler.md` | EiffelStudio profiler usage |
| `patterns.md` | Verified working code patterns |

---

## CI Tool: simple_ci

Use `simple_ci.exe` to compile and test Eiffel projects:

```batch
:: Run all configured projects
simple_ci.exe

:: Run specific project with verbose output
simple_ci.exe -p simple_web --verbose

:: Fast incremental build (skip clean)
simple_ci.exe -p simple_web --no-clean --verbose
```

Output files:
- `ci_report.json` - Machine-readable (for Claude parsing)
- `ci_report.txt` - Human-readable summary

### Adding Projects to CI

Edit `D:\prod\simple_ci\src\ci_config.e` to add new projects.

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
| 2025-12-03 | Added Meyer's "probable to provable" framework, verification_process.md, contract_patterns.md |
| 2025-12-03 | Added Specification Hat and contract completeness guidance |
| 2025-12-02 | Added simple_ci documentation, middleware patterns, Windows cmd patterns |
| 2025-12-02 | Genericized - removed project-specific refs, added ROADMAP.md convention |
| 2025-12-02 | Added profiler.md reference, expanded scoop.md |
| 2025-12-01 | Added EIS documentation patterns |
| 2025-11-29 | Initial structure created |
