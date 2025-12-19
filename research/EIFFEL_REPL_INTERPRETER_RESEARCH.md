# Eiffel REPL & Interpreter Research

**Research Date:** December 2024
**Purpose:** Investigate feasibility of interactive Eiffel execution for EiffelNotebook
**Requested by:** Larry Rix

---

## Executive Summary

This research explores options for building an interactive Eiffel environment (REPL/notebook). Key findings:

| Option | Effort | License | Status |
|--------|--------|---------|--------|
| **EiffelStudio Melting Ice** | Low | GPL | Production-ready, bytecode VM exists |
| **Gobo Eiffel Compiler (gec)** | Medium | MIT | Active, compiles to C only |
| **tecomp Interpreter** | High | GPL | Abandoned (v0.11), has working VM |
| **Albatross** | Very High | Custom | Development phase, verification-focused |
| **Build from ECMA-367** | Extreme | N/A | Requires full parser + type system + VM |

**Recommendation:** Leverage EiffelStudio's existing Melting Ice bytecode VM via the accumulated class pattern. The infrastructure for interpretation already exists.

---

## 1. Gobo Eiffel Compiler (gec)

### Overview

The Gobo Eiffel Compiler is the only MIT-licensed open source Eiffel compiler.

- **Website:** [gobosoft.com/eiffel/gobo/gec](http://www.gobosoft.com/eiffel/gobo/gec/index.html)
- **GitHub:** [github.com/gobo-eiffel/gobo](https://github.com/gobo-eiffel/gobo)
- **License:** MIT (less restrictive than GPL)
- **Author:** Eric Bezault

### How It Works

```
Eiffel Source → gec → C Code → Zig/GCC/Clang → Native Binary
```

gec compiles Eiffel to C code, which is then compiled by a backend C compiler. Recent versions use the Zig toolchain by default (bundled in the delivery package).

### Recent Features (2024)

- SCOOP support added (wait conditions not yet supported)
- Boehm GC by default (included in delivery)
- Zig toolchain as default backend (bundled)
- Accept `.e` files directly without ECF
- Zero-configuration deployment

### REPL Potential

**None directly.** gec is a batch compiler with no interpreter mode. However:

- Could be used as the compilation backend for an accumulated-class REPL
- Very fast compilation makes it viable for notebook cells
- MIT license allows modification for custom uses

### Debugger Project

A separate project exists: [GOBO-Eiffel-debugger](https://github.com/wjansen/GOBO-Eiffel-debugger)

This could potentially be extended for breakpoint-based expression evaluation.

---

## 2. EiffelStudio Open Source Status

### License

EiffelStudio is released under **GPL v2.0** for open source development.

- **Official License:** [eiffel.org/license/eiffelstudio/gpl](https://www.eiffel.org/license/eiffelstudio/gpl)
- Commercial licenses also available

### Source Code Availability

| Repository | URL | Notes |
|------------|-----|-------|
| **SVN (Primary)** | `svn.eiffel.com/eiffelstudio-public` | Password-protected since 2019 |
| **GitHub Mirror** | [github.com/EiffelSoftware/EiffelStudio](https://github.com/EiffelSoftware/EiffelStudio) | **12-month delay** from trunk |
| **Libraries** | [github.com/EiffelSoftware/libraries](https://github.com/EiffelSoftware/libraries) | Updated daily |

### Current Status

- No new GPL release since 2019
- GitHub mirror continues with 12-month delay
- Full compiler source is available (but lagging)
- Libraries are current

### What's Included

The source code includes:
- Full compiler implementation
- **Melting Ice bytecode interpreter**
- Debugger with expression evaluation
- All standard libraries

---

## 3. EiffelStudio Melting Ice Technology

### The Hidden Gem

EiffelStudio already has a bytecode interpreter that runs Eiffel code without full C compilation. This is **exactly what we need** for a REPL.

- **Documentation:** [dev.eiffel.com/Melting_Ice_Technology](https://dev.eiffel.com/Melting_Ice_Technology)
- **How It Compiles:** [eiffel.org/doc/eiffelstudio/How_EiffelStudio_Compiles](https://www.eiffel.org/doc/eiffelstudio/How_EiffelStudio_Compiles)

### How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                    EiffelStudio System                      │
├─────────────────────────┬───────────────────────────────────┤
│     Frozen Code         │         Melted Code               │
│     (C → Native)        │         (Eiffel Bytecode)         │
│                         │                                   │
│  • Full optimization    │  • Interpreted by VM              │
│  • Slow to compile      │  • Fast to update (~seconds)      │
│  • Production use       │  • Development use                │
└─────────────────────────┴───────────────────────────────────┘
```

### Bytecode Format (EBC)

- **Storage:** `EIFGEN/W_CODE/project.melted`
- **Type:** Stack-based bytecode (similar to JVM/.NET IL)
- **Interpreter:** Built into EiffelStudio runtime

### Compilation Modes

| Mode | Speed | Output | Use Case |
|------|-------|--------|----------|
| **Melt** | Seconds | Bytecode (.melted) | Incremental dev |
| **Freeze** | Minutes | C + Native | Full rebuild |
| **Finalize** | Minutes | Optimized Native | Production |

### REPL Potential

**High.** The melting ice VM is exactly what's needed:

1. Parse Eiffel code
2. Generate EBC (bytecode)
3. Execute in existing VM
4. Return results

The challenge is accessing this programmatically vs. through the IDE.

### Research Questions

1. Can `ec` generate and execute melted code without full IDE?
2. Is the bytecode format documented?
3. Can the interpreter be invoked from command line?

---

## 4. tecomp: The Eiffel Compiler/Interpreter

### Overview

tecomp is a GPL-licensed Eiffel compiler that can **execute code directly** without C compilation.

- **SourceForge:** [sourceforge.net/projects/tecomp](https://sourceforge.net/projects/tecomp/)
- **License:** GPL v2.0 / LGPL v2.0
- **Author:** Helmut Brandl
- **Status:** Abandoned (last release: v0.11)

### How It Works

```
Eiffel Source → tecomp Parser → Internal Representation → VM Execution
                                        ↓
                              (Optional) C Code Generation
```

tecomp can:
1. **Interpret:** Execute directly in its VM
2. **Compile:** Generate C code for native execution

### Virtual Machine

- Architecture-independent (Intel, SPARC, PowerPC, Motorola)
- Runs on Unix and Windows (32/64 bit, big/little endian)
- Garbage collector implemented (v0.8+)

### ECMA Compliance

tecomp committed to implementing ECMA-367:
- Void safety (attached/detachable types)
- Object tests
- Inline agents
- Assigner commands
- Bracket notation

### REPL Potential

**Theoretical but impractical:**

- Has working interpreter infrastructure
- ECMA-compliant parsing
- BUT: Abandoned since ~2010
- No modern Eiffel features
- Would require significant revival effort

### Successor: Albatross

The author (Helmut Brandl) moved on to create Albatross, a verification-focused language inspired by Eiffel.

---

## 5. Albatross Programming Language

### Overview

Albatross is a statically verified programming language that evolved from "Modern Eiffel."

- **Website:** [albatross-lang.sourceforge.net](https://albatross-lang.sourceforge.net/)
- **GitHub:** [github.com/hbr/albatross](https://github.com/hbr/albatross)
- **Documentation:** [hbr.gitbooks.io/alba-lang-description](https://hbr.gitbooks.io/alba-lang-description/content/index.html)
- **License:** Custom
- **Author:** Helmut Brandl (same as tecomp)

### Philosophy

Albatross extends Eiffel's DBC into full formal verification:

```
Programs + Proofs in the same language
```

The compiler is part theorem prover, part compiler.

### Features

- Functional programming (algebraic data types, pattern matching)
- Imperative programming (arrays, loops, mutable state)
- Object-oriented (inheritance, abstract classes)
- Formal verification built-in

### REPL Potential

**None for Eiffel:**

- Different language (not Eiffel-compatible)
- Focused on verification, not rapid execution
- Still in development phase
- Interesting for future research, not current needs

---

## 6. LibertyEiffel / SmartEiffel

### Overview

The GNU Eiffel compiler(s), descended from SmartEiffel.

- **Website:** [liberty-eiffel.org](https://www.liberty-eiffel.org/)
- **GitHub:** [github.com/LibertyEiffel/Liberty](https://github.com/LibertyEiffel/Liberty)
- **License:** GPL
- **FSF:** [gnu.org/software/liberty-eiffel](https://www.gnu.org/software/liberty-eiffel/)

### How It Works

```
Eiffel Source → SmartEiffel/Liberty → C Code → Native Binary
                                   ↓
                            (SmartEiffel also) → Java Bytecode
```

### Dialect

LibertyEiffel implements a dialect between SmartEiffel and ECMA-367. Not fully standard-compliant.

### REPL Potential

**None.** Pure compilers with no interpreter mode.

SmartEiffel's Java bytecode output is interesting but doesn't help for REPL.

---

## 7. ECMA-367 Specification

### Documents

| Edition | Date | URL |
|---------|------|-----|
| 2nd Edition | June 2006 | [ecma-international.org PDF](https://www.ecma-international.org/wp-content/uploads/ECMA-367_2nd_edition_june_2006.pdf) |
| ETH Mirror | Current | [se.inf.ethz.ch PDF](https://se.inf.ethz.ch/~meyer/ongoing/etl/EIFFEL-STANDARD.pdf) |
| ISO | 2006 | ISO/IEC 25436 |

### What It Specifies

- Complete language syntax and semantics
- Type system (including void safety)
- Inheritance rules
- Generic types
- Agents
- SCOOP (partial)

### What It Does NOT Specify

- Compilation mechanism
- Bytecode format
- Runtime implementation
- Memory management

### Implementation Complexity

Building an Eiffel interpreter from ECMA-367 requires:

| Component | Complexity | Notes |
|-----------|------------|-------|
| Lexer | Low | Standard tokenization |
| Parser | Medium | Context-free grammar in spec |
| Type System | **High** | Covariance, generics, anchored types |
| Inheritance | **High** | Multiple inheritance, renaming, redefining |
| Void Safety | Medium | Static analysis required |
| Agents | High | First-class functions with closures |
| SCOOP | **Very High** | Concurrent model |
| Runtime | High | GC, exception handling, once features |

**Estimate:** 2-4 person-years for a minimal compliant interpreter.

---

## 8. Historical Resources (Wayback Machine)

### Notable Archives

The Internet Archive (web.archive.org) preserves historical Eiffel documentation:

- **archive.eiffel.com:** Old Eiffel Software documentation
- **gobosoft.com archives:** Historical Gobo releases
- **smarteiffel.loria.fr:** Original SmartEiffel site

### Useful Historical Documents

- Original Melting Ice papers (1990s)
- Early EiffelBench architecture
- SmartEiffel internals documentation

### Search Strategy

```
site:web.archive.org eiffel compiler internal
site:web.archive.org eiffelstudio bytecode
site:web.archive.org smarteiffel implementation
```

---

## 9. Feasibility Analysis: Building Our Own REPL

### Option A: Accumulated Class + Existing Compiler (Recommended)

**Approach:** Generate wrapper class, compile with `ec`, execute

```
Cell 1: x := 42      →  class NOTEBOOK
Cell 2: print(x)     →    feature x: INTEGER
                     →    feature cell_1 do ... end
                     →    feature cell_2 do ... end
                     →    feature make do cell_1; cell_2 end
                     →  end
                     →
                     →  ec -batch -config notebook.ecf -target run
```

| Aspect | Assessment |
|--------|------------|
| Effort | **Low** (weeks) |
| Dependencies | EiffelStudio or gec |
| Performance | ~1-3 seconds per cell |
| Compatibility | Full Eiffel |

**This is the EiffelNotebook vision approach.**

### Option B: Leverage Melting Ice Directly

**Approach:** Invoke EiffelStudio's bytecode interpreter programmatically

| Aspect | Assessment |
|--------|------------|
| Effort | **Medium** (months) |
| Dependencies | EiffelStudio source understanding |
| Performance | Sub-second execution |
| Compatibility | Full Eiffel |

**Requires research into:**
- ec command-line options for melted execution
- Debugger expression evaluation API
- Bytecode generation without full compile

### Option C: Revive tecomp

**Approach:** Resurrect and modernize tecomp's interpreter

| Aspect | Assessment |
|--------|------------|
| Effort | **High** (6+ months) |
| Dependencies | tecomp source (GPL) |
| Performance | Unknown (VM overhead) |
| Compatibility | Partial (2010-era Eiffel) |

**Challenges:**
- 14+ years of bitrot
- Missing modern Eiffel features
- Unknown code quality
- Sole maintainer moved on

### Option D: Build Interpreter from Scratch

**Approach:** Implement ECMA-367 interpreter from specification

| Aspect | Assessment |
|--------|------------|
| Effort | **Extreme** (2-4 person-years) |
| Dependencies | None |
| Performance | Likely slow |
| Compatibility | Could be full |

**Components needed:**
1. Lexer (use simple_eiffel_parser as base)
2. Parser (extend simple_eiffel_parser)
3. Type checker (complex: inheritance, generics)
4. Bytecode generator (design our own)
5. Virtual machine (stack or register based)
6. Runtime (GC, exceptions, once features)
7. Standard library bindings

**Not recommended** unless there's a multi-year commitment.

---

## 10. Recommendations

### For EiffelNotebook (Short Term)

**Use Option A: Accumulated Class Pattern**

1. Parse cells with simple_eiffel_parser (extend as needed)
2. Generate NOTEBOOK_SESSION class
3. Compile with `ec -batch` (melting mode for speed)
4. Execute and capture output
5. Maintain session state via class attributes

**Timeline:** 4-8 weeks for MVP

### For Performance (Medium Term)

**Investigate Option B: Melting Ice Integration**

1. Study EiffelStudio source (GitHub, 12-month delay acceptable)
2. Find bytecode generation APIs
3. Test command-line melted execution
4. Consider debugger expression evaluation

**Timeline:** 2-3 months research + prototype

### For Independence (Long Term)

**Consider Option C: tecomp Revival**

Only if:
- Eiffel Software cooperation is unavailable
- MIT licensing is required
- Community interest in maintenance

**Timeline:** 6-12 months for usable state

---

## 11. Source References

### Primary Sources

- [Gobo Eiffel Project - GitHub](https://github.com/gobo-eiffel/gobo)
- [Gobo Eiffel Compiler](http://www.gobosoft.com/eiffel/gobo/gec/index.html)
- [EiffelStudio - GitHub Mirror](https://github.com/EiffelSoftware/EiffelStudio)
- [EiffelStudio GPL License](https://www.eiffel.org/license/eiffelstudio/gpl)
- [Melting Ice Technology](https://dev.eiffel.com/Melting_Ice_Technology)
- [How EiffelStudio Compiles](https://www.eiffel.org/doc/eiffelstudio/How_EiffelStudio_Compiles)
- [tecomp - SourceForge](https://sourceforge.net/projects/tecomp/)
- [Albatross Language](https://albatross-lang.sourceforge.net/)
- [Albatross - GitHub](https://github.com/hbr/albatross)
- [LibertyEiffel](https://www.liberty-eiffel.org/)
- [LibertyEiffel - GitHub](https://github.com/LibertyEiffel/Liberty)
- [ECMA-367 Standard](https://ecma-international.org/publications-and-standards/standards/ecma-367/)
- [ECMA-367 PDF (2nd Edition)](https://www.ecma-international.org/wp-content/uploads/ECMA-367_2nd_edition_june_2006.pdf)
- [ETH ECMA-367 Mirror](https://se.inf.ethz.ch/~meyer/ongoing/etl/EIFFEL-STANDARD.pdf)
- [ECMA Implementation Status](https://dev.eiffel.com/ECMA_Implementation)
- [EiffelStudio Wikipedia](https://en.wikipedia.org/wiki/EiffelStudio)
- [Eiffel Language Wikipedia](https://en.wikipedia.org/wiki/Eiffel_(programming_language))

### Historical/Archival

- [Archive.eiffel.com - Gobo](https://archive.eiffel.com/products/shelf/bezault/page.html)
- [EiffelStudio History (archive)](https://archive.eiffel.com/doc/online/eiffel50/intro/studio/index-17.html)

### Community

- [Eiffel Users Google Group](https://groups.google.com/g/eiffel-users)
- [Lambda the Ultimate - Albatross](http://lambda-the-ultimate.org/node/5144)
- [Hacker News - Eiffel](https://news.ycombinator.com/item?id=22281615)

---

## 12. Appendix: tecomp Release History

| Version | Date | Notable Features |
|---------|------|------------------|
| 0.1 | 2008 | Initial release |
| 0.4 | ~2009 | Void safety, attached/detachable |
| 0.6.1 | ~2009 | Interpreter improvements |
| 0.7 | 2009 | Public announcement |
| 0.8 | ~2010 | Garbage collector |
| 0.11 | ~2010 | Final release |

---

## 13. Appendix: ECMA-367 Major Features

For reference, these features must be implemented by any compliant interpreter:

- **Basic Types:** INTEGER, REAL, CHARACTER, STRING, BOOLEAN
- **Reference Types:** Classes with attributes and routines
- **Inheritance:** Multiple, with renaming, redefining, selecting
- **Generics:** Constrained generic types
- **Agents:** Inline agents, call/item/apply
- **Contracts:** Preconditions, postconditions, invariants
- **Void Safety:** Attached/detachable types, object tests
- **Once Features:** Per-thread, per-object, per-process
- **Exceptions:** Retry, rescue
- **External:** C/C++ interop
- **SCOOP:** Separate types, wait-by-necessity (optional)

---

## 14. Appendix: EiffelStudio Bytecode Internals (Source Analysis)

**Source:** GitHub mirror at `github.com/EiffelSoftware/EiffelStudio` (GPL v2.0)

This section documents the internal architecture of EiffelStudio's Melting Ice bytecode interpreter, derived from direct source code analysis.

### 14.1 Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    EiffelStudio Runtime                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────────┐     │
│   │   IC        │    │  op_stack   │    │     iregs       │     │
│   │ (Interpreter│    │ (Operand    │    │   (Registers)   │     │
│   │  Counter)   │    │   Stack)    │    │                 │     │
│   │             │    │             │    │  [0] Current    │     │
│   │  Points to  │    │  RPN-style  │    │  [1] Result     │     │
│   │  next       │    │  stack for  │    │  [2] locnum     │     │
│   │  bytecode   │    │  operations │    │  [3] argnum     │     │
│   │  instruction│    │             │    │  [4+] locals    │     │
│   └─────────────┘    └─────────────┘    │  [n+] args      │     │
│                                          └─────────────────┘     │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    Dispatch Tables                       │   │
│   │                                                          │   │
│   │  egc_frozen[body_id] → C function pointer (frozen code)  │   │
│   │  melt[body_id]       → bytecode address (melted code)    │   │
│   │  pattern[pat_id]     → interpreter entry point           │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 14.2 Key Source Files

| File | Size | Purpose |
|------|------|---------|
| `Src/C/run-time/interp.c` | 229 KB | Main bytecode interpreter loop |
| `Src/C/run-time/wbench.c` | 21 KB | Workbench mode dispatch |
| `Src/C/run-time/include/rt_interp.h` | 12 KB | Opcode definitions (177 opcodes) |
| `Src/C/run-time/eif_interp.h` | 3 KB | Public interpreter API |
| `Src/C/run-time/eif_wbench.h` | 3 KB | Workbench API |
| `Src/C/bench/meltdump.c` | 22 KB | Tool to dump .melted files |
| `Src/C/bench/bytedump.c` | 34 KB | Bytecode disassembler |
| `Src/Eiffel/eiffel/byte_code/*.e` | ~180 files | Bytecode node classes |
| `Src/Eiffel/eiffel/generation/byte_array.e` | - | Bytecode emission |

### 14.3 Core Data Structures

#### Interpreter Counter (IC)
```c
rt_public unsigned char *IC = NULL;
// The interpreter counter is the location in byte code of the next
// instruction to be fetched. Its behaviour is similar to the one of
// PC in a central processing unit.
```

#### Operand Stack (op_stack)
```c
rt_shared struct opstack op_stack = {
    (struct stopchunk *) 0,  // st_hd - head
    (struct stopchunk *) 0,  // st_tl - tail
    (struct stopchunk *) 0   // st_cur - current
};
// This is the stack used by the virtual stack machine, in a reverse
// polish notation manner (RPN). All operations take arguments from
// the stack and push results back onto the stack.
```

#### Register Array (iregs)
```c
rt_private EIF_TYPED_VALUE **iregs = NULL;
#define icurrent    (*iregs)           // Value of Current
#define iresult     (*(iregs+1))       // Result of function
#define ilocnum     (*(iregs+2))       // Number of locals
#define iargnum     (*(iregs+3))       // Number of arguments
#define loc(n)      (*(iregs+3+(n)))   // Locals from 1 to locnum
#define arg(n)      (*(iregs+3+locnum+(n)))  // Arguments
```

### 14.4 Bytecode Opcodes (177 Total)

#### Control Flow (0-30)
| Code | Name | Description |
|------|------|-------------|
| 0 | `BC_START` | Routine entry |
| 1 | `BC_PRECOND` | Start preconditions |
| 2 | `BC_POSTCOND` | Start postconditions |
| 3 | `BC_DEFERRED` | Deferred routine marker |
| 17 | `BC_JMP_F` | Jump if false |
| 18 | `BC_JMP` | Unconditional jump |
| 19 | `BC_LOOP` | Loop marker |
| 78 | `BC_JMP_T` | Jump if true |

#### Assertions (5-16)
| Code | Name | Description |
|------|------|-------------|
| 5 | `BC_CHECK` | Check instruction |
| 6 | `BC_ASSERT` | Assertion |
| 8 | `BC_PRE` | Precondition |
| 9 | `BC_PST` | Postcondition |
| 11 | `BC_LINV` | Loop invariant |
| 13 | `BC_INV` | Class invariant |

#### Assignment & Creation (23-35)
| Code | Name | Description |
|------|------|-------------|
| 23 | `BC_RASSIGN` | Result assignment |
| 24 | `BC_LASSIGN` | Local assignment |
| 25 | `BC_ASSIGN` | General assignment |
| 26 | `BC_CREATE` | Object creation |
| 31 | `BC_CREATE_TYPE` | Typed creation |
| 34 | `BC_LREVERSE` | Local reverse assignment |
| 35 | `BC_RREVERSE` | Result reverse assignment |

#### Feature Calls (36-44)
| Code | Name | Description |
|------|------|-------------|
| 36 | `BC_FEATURE` | Feature call |
| 40 | `BC_FEATURE_INV` | Feature call with invariant |
| 41 | `BC_ATTRIBUTE` | Attribute access |
| 42 | `BC_ATTRIBUTE_INV` | Attribute with invariant |
| 43 | `BC_EXTERN` | External call |
| 44 | `BC_EXTERN_INV` | External with invariant |

#### Values & Registers (45-51)
| Code | Name | Description |
|------|------|-------------|
| 45 | `BC_CHAR` | Character constant |
| 46 | `BC_BOOL` | Boolean constant |
| 47 | `BC_INT32` | 32-bit integer constant |
| 48 | `BC_DOUBLE` | Double constant |
| 49 | `BC_RESULT` | Push Result |
| 50 | `BC_LOCAL` | Push local variable |
| 51 | `BC_ARG` | Push argument |

#### Operators (52-71)
| Code | Name | Description |
|------|------|-------------|
| 52 | `BC_UPLUS` | Unary plus |
| 53 | `BC_UMINUS` | Unary minus |
| 54 | `BC_NOT` | Boolean not |
| 55 | `BC_LT` | Less than |
| 56 | `BC_GT` | Greater than |
| 57 | `BC_MINUS` | Subtraction |
| 60 | `BC_EQ` | Equality |
| 61 | `BC_NE` | Not equal |
| 62 | `BC_STAR` | Multiplication |
| 68 | `BC_SLASH` | Division |
| 70 | `BC_PLUS` | Addition |

#### Exception Handling (80-82, 172-176)
| Code | Name | Description |
|------|------|-------------|
| 80 | `BC_RESCUE` | Rescue clause start |
| 81 | `BC_END_RESCUE` | Rescue clause end |
| 82 | `BC_RETRY` | Retry instruction |
| 172 | `BC_TRY` | Try block start |
| 173 | `BC_TRY_END` | Try block end |
| 175 | `BC_DO_RESCUE` | Do with rescue |

#### SCOOP (27-28, 114, 166)
| Code | Name | Description |
|------|------|-------------|
| 27 | `BC_START_SEPARATE` | Separate block start |
| 28 | `BC_END_SEPARATE` | Separate block end |
| 114 | `BC_SEPARATE` | Separate type marker |
| 166 | `BC_WAIT_ARG` | Wait condition argument |

### 14.5 Key Functions

#### Main Interpreter Entry
```c
// Public entry point - sets IC and calls interpret()
rt_public void xinterp(unsigned char *icval, rt_uint_ptr nb_pushed);

// Initialize the interpreter
rt_public void xinitint(void);

// IMPORTANT: Can update bytecode for a feature at runtime!
rt_public void eif_override_byte_code_of_body(
    int body_id,
    int pattern_id,
    unsigned char *bc,
    int count
);
```

#### Workbench Dispatch
```c
// Get function pointer for feature call
rt_public EIF_REFERENCE_FUNCTION wfeat(int routine_id, EIF_TYPE_INDEX dtype)
{
    BODY_INDEX body_id;
    CBodyId(body_id, routine_id, dtype);  // Get body index

    if (egc_frozen[body_id])
        return (EIF_REFERENCE_FUNCTION) egc_frozen[body_id];  // Frozen
    else {
        IC = melt[body_id];  // Position interpreter to bytecode
        return (EIF_REFERENCE_FUNCTION) pattern[MPatId(body_id)].toi;
    }
}
```

### 14.6 Melted File Format

The `.melted` file (e.g., `project.melted` in `EIFGEN/W_CODE/`) contains:

```
┌────────────────────────────────────────┐
│ Header                                  │
│  - Has content flag (1 byte)            │
│  - IEEE arithmetic flag (1 byte)        │
│  - Class types count (4 bytes)          │
│  - Class count (4 bytes)                │
│  - Original routine bodies (4 bytes)    │
│  - Profiling enabled (4 bytes)          │
├────────────────────────────────────────┤
│ Type Names Table                        │
│  - Count of feature table updates       │
│  - For each: dtype, name length, name   │
├────────────────────────────────────────┤
│ Bytecode Bodies                         │
│  - Compressed with LZO                  │
│  - Each body: pattern_id + bytecode     │
├────────────────────────────────────────┤
│ Metadata                                │
│  - Class nodes                          │
│  - Parent tables                        │
│  - CECIL info                           │
│  - Options                              │
│  - Routine info                         │
│  - Descriptors                          │
│  - Root class info                      │
└────────────────────────────────────────┘
```

### 14.7 Implications for EiffelNotebook

#### Potential Integration Points

1. **`eif_override_byte_code_of_body()`** - Could inject new bytecode at runtime!
2. **`xinterp()`** - Direct interpreter invocation with bytecode address
3. **`melt[]` array** - Register new melted features
4. **`pattern[]` table** - Add dispatch entries

#### Challenges

1. **Tight Coupling** - Interpreter expects full EiffelStudio infrastructure
2. **No Documentation** - Internal format undocumented
3. **Version Dependency** - Bytecode format may change between versions
4. **Initialization** - Requires full runtime initialization

#### Realistic Approach

Rather than trying to invoke the interpreter directly, the accumulated class pattern remains the most practical approach:

1. Generate Eiffel class with notebook cells
2. Use `ec -batch -melt` for fast compilation
3. Let EiffelStudio handle bytecode generation
4. Execute resulting system

This leverages the Melting Ice VM indirectly while avoiding the complexity of direct bytecode manipulation.

### 14.8 Tools Available

The EiffelStudio source includes useful diagnostic tools:

| Tool | Location | Purpose |
|------|----------|---------|
| `meltdump` | `Src/C/bench/meltdump.c` | Parse and dump .melted files |
| `bytedump` | `Src/C/bench/bytedump.c` | Disassemble bytecode |

These could be built and used to understand the bytecode format better.

---

*Document generated: December 2024*
*Updated: December 2024 - Added bytecode internals from source analysis*
*Status: Research Complete*
