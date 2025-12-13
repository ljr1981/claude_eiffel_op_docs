# simple_alpine — Constitutional Audit (Eiffel Expert v1.1)

## Scope
Audit and review of the **simple_alpine** repository against the **EIFFEL_EXPERT_BRIEFING v1.1 (OOSC2-Integrated)**.

This audit evaluates:
- Design by Contract (DbC)
- Void safety
- SCOOP correctness and liveness
- Ecosystem standards (ECF, testing, library usage)
- Restricted / radioactive zones

---

## Executive Summary

**Overall status:** ✅ *Conditionally compliant*

The repository aligns well with the Simple Eiffel ecosystem and OOSC2 principles, but requires a small number of **mandatory corrections** to fully conform to the Eiffel Expert Constitution.

---

## Confirmed Passes

### 1. ECF Configuration
- `void_safety support="all"` is enabled.
- `concurrency support="scoop"` is enabled.
- Environment-variable–based library paths are used.

**Status:** PASS

---

### 2. Testing Standards
- Test classes inherit from `TEST_SET_BASE`.
- Test root application does **not** inherit from a test base.
- Tests are located under `testing/`.

**Status:** PASS

---

### 3. General DbC Usage
- Core domain classes (e.g., `ALPINE_ELEMENT`) demonstrate correct Eiffel idioms.
- Postconditions are used to confirm semantic state, not implementation detail.

**Status:** PASS

---

## Required Corrections (Constitutional)

### 1. ECF Test Cluster Naming (Hard Failure)

**Observed:**
```xml
<cluster name="tests" location=".\testing\" recursive="true"/>
```

**Required by Constitution:**
- Cluster name must be `test_classes`

**Correction:**
```xml
<cluster name="test_classes" location=".\testing\" recursive="true"/>
```

**Rationale:**
This is a standardized ecosystem invariant. Deviations increase friction across `simple_*` libraries.

**Status:** ❌ FAIL (must fix)

---

### 2. Missing DbC on Public Factory API

**Observed:**
Public factory routines in `ALPINE_FACTORY` expose part of the library’s public ADT but lack contracts.

**Why this matters (OOSC2):**
Factories define *abstract creation semantics*. Without contracts, they become implementation-defined and brittle under evolution.

**Minimum Required Pattern:**
```eiffel
some_factory_feature (...): SOME_TYPE
    require
        semantic_precondition: <low-risk semantic rule>
    do
        ...
    ensure
        result_attached: Result /= Void
    end
```

**Guidance:**
- Preconditions must express *semantic necessity*, not defensive checks.
- Always include `ensure Result /= Void`.

**Status:** ❌ FAIL (must fix)

---

## Advisory Risks (Not Hard Failures)

### STRING_32 → STRING_8 Boundary

**Observed:**
Frequent use of:
```eiffel
a_string.to_string_8
```

**Risk:**
- Potential Unicode truncation
- Semantic drift if non-ASCII identifiers are later permitted

**Recommendation:**
- Treat STRING_8 conversion as a **serialization boundary**
- Prefer STRING_32 internally
- Document the boundary explicitly if intentional

**Status:** ⚠️ ADVISORY

---

## Restricted / Radioactive Zones Check

No violations detected in:
- Inline C ABI
- Encoding / byte-level I/O
- Persistence schema
- ECF capability flags (aside from cluster naming)

**Status:** PASS

---

## Priority Action List

1. Rename ECF test cluster to `test_classes`.
2. Add minimal DbC to all public factory features.
3. Decide and document STRING_32 → STRING_8 boundary policy.

---

## Final Judgment

With the above corrections applied, **simple_alpine** will be fully compliant with the **Eiffel Expert Constitution v1.1** and safe for unsupervised agent modification.

> Any future change that weakens ADT semantics, substitutability, or SCOOP liveness is a regression — even if tests pass.
