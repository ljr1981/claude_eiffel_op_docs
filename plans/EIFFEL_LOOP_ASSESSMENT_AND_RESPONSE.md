# Assessment: simple_* Libraries and Eiffel-Loop Relationship

**Date:** 2026-01-20

---

## Executive Summary

Finnian Reilly's concerns in his January 20, 2026 email deserve a thoughtful response. After researching the full context, I can clarify:

1. **We are NOT forking Eiffel-Loop** - We created NEW libraries inspired by studying EL concepts
2. **We used the CURRENT master branch** - Not an old version (last commit Nov 18, 2025)
3. **Gobo dependencies confirmed removed** - The base.ecf we analyzed has only ISE dependencies
4. **Scope was limited to base library concepts** - The only EL library with no EL dependencies
5. **The simple_* libraries are original implementations** - Contracts, tests, and code are new

---

## Part 0: About Eiffel-Loop (Research Findings)

### History and Scope

| Property | Value |
|----------|-------|
| Started | 2002 (24 years of development) |
| Author | Finnian Reilly (Hex 11 Software, Dublin) |
| Professional Eiffel since | 2000 |
| Class count | 4,100+ classes |
| Library count | 50+ specialized libraries |
| GitHub | `https://github.com/finnianr/Eiffel-Loop` |

### Key Technical Achievements

- **EL_ZSTRING**: Dual-storage string (8-bit + compacted 32-bit) - developed fall 2015, released Jan 2016
- **EL_REFLECTION**: Comprehensive reflection framework
- **EL_FACTORY patterns**: Singleton, pool, lazy initialization patterns
- **Platform support**: Cross-platform (Windows, Linux, macOS)
- **Application frameworks**: Audio, database, graphics, multimedia

### Community Standing

Eiffel-Loop is one of the most comprehensive third-party Eiffel libraries. Finnian has contributed to Eiffel community discussions and his work is referenced in various Eiffel forums and documentation.

### Known Challenges

- **Void safety migration**: EL uses `void_safety = "none"` - migrating 4100+ classes is significant work
- **EiffelStudio compatibility**: Some historical ISE API changes affect compilation
- **Dependencies**: Complex inter-library dependencies (by design, for comprehensive features)

---

## Part 1: What We Actually Did

### Source Analyzed

| Property | Value |
|----------|-------|
| Repository | `https://github.com/finnianr/Eiffel-Loop.git` |
| Branch | master |
| Last Commit | `db7b4cffc` (Nov 18, 2025) |
| Commit Count | 1,886 total, 479 since Jan 2024 |
| Focus Area | `library/base/base.ecf` only |

### Why Base Library Only

The EL_base library was chosen because it is the **only** Eiffel-Loop library with no upstream EL dependencies:

```
Dependencies (from base.ecf):
- $ISE_LIBRARY/library/base/base.ecf
- $ISE_LIBRARY/library/base_extension/base_extension.ecf
- $ISE_LIBRARY/library/encoding/encoding.ecf
- $ISE_LIBRARY/library/i18n/i18n.ecf (Windows only)
- $ISE_LIBRARY/library/net/net.ecf
- $ISE_LIBRARY/library/time/time.ecf
- $ISE_LIBRARY/library/thread/thread.ecf
- $ISE_LIBRARY/library/uuid/uuid.ecf
```

**No Gobo.** **No other EL libraries.** This made it a clean boundary for study.

### What We Created (NEW Libraries)

We did NOT fork or modify Eiffel-Loop. We created **NEW libraries** with original code:

| Library | Classes | Tests | GitHub | Nature |
|---------|---------|-------|--------|--------|
| simple_factory | 8 | 53 | [simple-eiffel/simple_factory](https://github.com/simple-eiffel/simple_factory) | **NEW** - Factory patterns inspired by EL concepts |
| simple_reflection | 9 | 44 | [simple-eiffel/simple_reflection](https://github.com/simple-eiffel/simple_reflection) | **NEW** - Reflection utilities inspired by EL concepts |
| simple_zstring | 11 | 373 | [simple-eiffel/simple_zstring](https://github.com/simple-eiffel/simple_zstring) | **NEW** - Dual-storage string inspired by EL_ZSTRING concept |
| simple_encoding | 8 | 83 | [simple-eiffel/simple_encoding](https://github.com/simple-eiffel/simple_encoding) | **NEW** - Codec system inspired by EL encoding |

**Key point:** These are **new implementations**, not ports or forks. The contracts, tests, and implementations are original, designed for modern EiffelStudio with void safety.

---

## Part 2: Addressing Finnian's Concerns

### Concern 1: "Forking an old version"

**Clarification:** We did not fork Eiffel-Loop at all.

- We cloned the repository to **study the architecture**
- The clone is from master branch, last commit Nov 18, 2025
- We created entirely new libraries with new code
- The simple_* libraries do not contain any EL_* classes

**Evidence:** The simple_* repositories on GitHub contain no EL_* class names and have their own independent git history.

### Concern 2: "Gobo dependencies removed"

**Confirmed:** The `library/base/base.ecf` we analyzed has NO Gobo dependencies.

```xml
<!-- From base.ecf - ISE libraries only -->
<library name = "base" location = "$ISE_LIBRARY/library/base/base.ecf"/>
<library name = "base_ext" location = "$ISE_LIBRARY/library/base_extension/base_extension.ecf"/>
<library name = "encoding" location = "$ISE_LIBRARY/library/encoding/encoding.ecf"/>
<!-- ... etc, all ISE -->
```

Grep for "gobo" in base library: **No matches found.**

### Concern 3: "Missing new features, bug fixes, optimizations"

**Acknowledged:** The simple_* libraries are NOT intended to be Eiffel-Loop replacements. They are:

1. **Learning vehicles** - Understanding EL's architectural patterns
2. **Simplified implementations** - Focused on core concepts, not full feature parity
3. **Modern Eiffel** - Targeting EiffelStudio 25.02 with void safety from day one
4. **Original code** - Not ports of EL code

We do NOT claim feature parity with Eiffel-Loop.

### Concern 4: "Highly interdependent" (from Finnian's 2025 analysis)

**Our approach addressed this:** By focusing ONLY on base library concepts with no EL dependencies, we avoided the interdependency challenge. The simple_* libraries have clean, minimal dependency graphs:

```
simple_factory → (no simple_* dependencies)
simple_encoding → (no simple_* dependencies)
simple_reflection → simple_factory
simple_zstring → simple_encoding
```

### Concern 5: "Do not endorse"

**Understood and respected.** We will:
- Not claim to represent Eiffel-Loop
- Not claim the simple_* libraries are compatible with EL
- Credit EL as inspiration where appropriate
- Clarify in documentation that these are independent implementations

---

## Part 3: Why We Did This

### Motivation

1. **Modern Eiffel Learning** - EL is a rich source of Eiffel patterns, but compiling it on EiffelStudio 25.02 revealed issues (see compilation-assessment.md)

2. **Void Safety from Day 1** - EL_base has `void_safety = "none"`. The simple_* libraries are void-safe.

3. **Simpler Entry Point** - EL's 4100+ class ecosystem is valuable but large. Simpler libraries help learners.

4. **No Python/External Dependencies** - simple_* uses only ISE libraries.

### What We Learned from EL

Studying EL_base was valuable. We learned:

- **ZSTRING concept** - Dual-storage string (8-bit + compacted 32-bit) is elegant
- **Factory patterns** - EL's singleton and pool patterns are well-designed
- **Reflection approach** - Building on ISE INTERNAL with cleaner APIs
- **Codec architecture** - Extensible encoding system design

### What We Did Differently

| Aspect | Eiffel-Loop | simple_* |
|--------|-------------|----------|
| Void Safety | "none" | "conformance" or higher |
| Target ES | 16.05 | 25.02 |
| Class Count | 4100+ | ~40 (in 4 libraries) |
| Dependencies | Complex inter-EL | Minimal, flat |
| Goal | Comprehensive | Focused, educational |

---

## Part 4: The Trail to Where We Are

### Timeline

| Date | Activity |
|------|----------|
| Jan 18, 2026 | Cloned Eiffel-Loop from GitHub master |
| Jan 18, 2026 | Attempted compilation with void safety - documented issues |
| Jan 18, 2026 | Decided to study concepts rather than port code |
| Jan 19, 2026 | Created specifications based on EL concepts |
| Jan 19-20, 2026 | Built simple_factory, simple_reflection, simple_encoding, simple_zstring |
| Jan 20, 2026 | All 4 libraries published to GitHub with tests |
| Jan 20, 2026 | Received Finnian's email |

### Documents Created

All planning documents are in `D:\prod\reference_docs\plans\`:

- `EIFFEL_LOOP_INTEGRATION_EXECUTION_PLAN.md` - Original plan
- `EIFFEL_LOOP_INTEGRATION_STATUS.md` - Current status
- `EIFFEL_LOOP_REMAINING_WORK.md` - Remaining integration work

Specifications in `D:\prod\simple_loop\specs\`:
- `01_simple_factory.md` through `04_simple_zstring.md` - Library specs
- `08_gap_analysis.md` - Analysis of what exists vs needed

Compilation notes in `D:\eiffel_loop\`:
- `compilation-assessment.md` - Issues compiling EL_base
- `void-safety-conversion-plan.md` - Original migration plan

---

## Part 5: Recommended Response to Finnian

### Tone
- Respectful and appreciative
- Clear about what we did and didn't do
- Acknowledge his work and expertise
- Correct any misunderstanding about "forking"

### Key Points to Make

1. **Not a fork** - We created new libraries, not a fork
2. **Used current master** - Not an old version
3. **Inspiration, not port** - Studied concepts, wrote new code
4. **No endorsement claimed** - We don't represent EL
5. **Void-safe focus** - Our goal was modern Eiffel learning
6. **Appreciation** - EL is valuable, Finnian's work is impressive

---

## Conclusion

Finnian's concerns are understandable given the wording that may have implied forking. The reality is:

- **We studied EL to learn** - Not to compete or replace
- **We built new code** - Inspired by concepts, not ported
- **We used current source** - Master branch from Nov 2025
- **We respect EL** - It's a valuable Eiffel resource

The simple_* libraries are complementary learning tools, not Eiffel-Loop replacements.
