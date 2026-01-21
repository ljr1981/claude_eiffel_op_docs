# MML Integration Across the Methodology Workflow

## Executive Summary

Model-Based Contracts using simple_mml should be integrated from **the earliest specification phases**, not added later. If MML models are defined in steps 01-04, they flow naturally through maintenance and hardening phases.

```
EARLY INTEGRATION (Steps 01-04)     →     AUTOMATIC BENEFIT (Steps 05-12)
┌─────────────────────────────┐           ┌─────────────────────────────┐
│ Define model queries        │           │ Design audit has models     │
│ Write model postconditions  │     →     │ Maintenance has contracts   │
│ Express invariants via MML  │           │ Xtreme has assault targets  │
└─────────────────────────────┘           └─────────────────────────────┘
```

---

## Integration Points by Workflow Step

### 01_project-creation: **FOUNDATIONAL** (Highest Impact)

| Step | Current | MML Enhancement |
|------|---------|-----------------|
| 02-DEFINE-CLASS-STRUCTURE | Signatures only | **Add model query signatures** |
| 04-WRITE-POSTCONDITIONS | Primitive contracts | **Model-based postconditions** |
| 05-WRITE-INVARIANTS | Basic invariants | **Model-based invariants** |

**NEW: Add step 02.5 - DEFINE-MODEL-QUERIES**

```eiffel
-- CURRENT (primitive postcondition)
extend (x: G)
    ensure
        count_incremented: count = old count + 1
        has_new_element: has (x)

-- MML ENHANCED (complete specification)
feature -- Model
    model: MML_SEQUENCE [G]
        -- Abstract mathematical model of this container
        do
            create Result.make_from_list (internal_storage.twin)
        end

feature -- Modification
    extend (x: G)
        ensure
            model_extended: model |=| old model & x
```

**Why this matters:** The MML postcondition `model |=| old model & x` says EXACTLY what extend does - the new model equals the old model with x appended. No ambiguity. Complete specification.

---

### 02_spec-extraction: **ADD MODEL IDENTIFICATION**

| Step | Current | MML Enhancement |
|------|---------|-----------------|
| S03-EXTRACT-CLASS-SPECS | Query/Command specs | **Identify model candidates** |

**Add to S03 template:**

```markdown
═══════════════════════════════════════════════════════════════
MODEL SPECIFICATION
What abstract mathematical structure represents this class?
═══════════════════════════════════════════════════════════════

MODEL ANALYSIS:
  Data structure: [SEQUENCE | SET | BAG | MAP | RELATION | INTERVAL]
  Key type: {element type}
  Value type: {if applicable}

  Recommended model query:
    model: MML_{TYPE} [{PARAMS}]
      -- Abstract state as mathematical model

  Contracts that could use model:
    - {feature}: postcondition could be model-based
    - {feature}: invariant could reference model
```

---

### 04_spec-from-research: **CRITICAL INTEGRATION POINT**

| Step | Current | MML Enhancement |
|------|---------|-----------------|
| R04-CLASS-DESIGN | Design classes | **Design model queries** |
| R05-CONTRACT-DESIGN | Design contracts | **Use MML in contracts** |

**Enhance R05-CONTRACT-DESIGN with MML section:**

```markdown
═══════════════════════════════════════════════════════════════
MODEL-BASED CONTRACT DESIGN
Complete specifications using mathematical models
═══════════════════════════════════════════════════════════════

For container/collection classes:

1. IDENTIFY ABSTRACT MODEL
   Class: {CLASS_NAME}
   Contains: {what elements}
   Order matters: [YES → MML_SEQUENCE | NO → MML_SET/MML_BAG]
   Duplicates allowed: [YES → MML_BAG | NO → MML_SET]
   Key-value pairs: [YES → MML_MAP]

2. DEFINE MODEL QUERY
   ```eiffel
   model: MML_{TYPE} [{PARAMS}]
       -- Abstract mathematical model
       do
           -- Construct from internal state
       end
   ```

3. WRITE MODEL-BASED POSTCONDITIONS

   For queries:
   ```eiffel
   item (i: INTEGER): G
       ensure
           consistent_with_model: Result = model [i]
   ```

   For commands:
   ```eiffel
   extend (x: G)
       ensure
           model_updated: model |=| old model & x

   remove (x: G)
       ensure
           model_updated: model |=| old model / x

   clear
       ensure
           model_empty: model.is_empty
   ```

4. WRITE MODEL-BASED INVARIANTS
   ```eiffel
   invariant
       model_count_consistent: model.count = count
       model_contains_all: ∀ i ∈ 1..count: model.has (item(i))
   ```
```

---

### 05_design-audit: **ASSESS MODEL COVERAGE**

**Add to audit criteria:**

```markdown
═══════════════════════════════════════════════════════════════
MODEL-BASED CONTRACT AUDIT
Are contracts complete specifications?
═══════════════════════════════════════════════════════════════

AUDIT CHECKLIST:

[ ] Does class have a model query? (if container-like)
[ ] Are postconditions model-based? (not just count checks)
[ ] Do invariants reference the model?
[ ] Is model query frozen/stable? (doesn't change specification)

PRIMITIVE vs MODEL-BASED:

| Primitive (Incomplete) | Model-Based (Complete) |
|------------------------|------------------------|
| count = old count + 1  | model |=| old model & x |
| has (x)                | model.has (x)          |
| is_empty               | model.is_empty         |

GAPS FOUND:
  {class}.{feature}: Uses primitive postcondition, could be model-based
```

---

### 07_maintenance-xtreme: **ASSAULT WITH MODELS**

**Enhance X03-CONTRACT-ASSAULT:**

```markdown
═══════════════════════════════════════════════════════════════
MODEL-BASED ASSAULT
Use models to write complete postconditions
═══════════════════════════════════════════════════════════════

If class has no model query, ADD ONE:

ASSAULT: Add model query to {CLASS}
  ```eiffel
  model: MML_SEQUENCE [G]
      do
          -- Construct from internal state
      end
  ```

Then assault with model-based contracts:

ASSAULT: {CLASS}.{feature} model postcondition
  BEFORE: ensure count = old count + 1
  AFTER:  ensure model |=| old model & x

  Why stronger: Model postcondition specifies COMPLETE behavior,
                not just one property. If implementation is wrong
                in ANY way, model equality fails.

MODEL ASSAULT CATEGORIES:

1. EXACT TRANSFORMATION
   model |=| old model.{operation}
   -- Specifies exactly how model changes

2. CONTAINMENT
   model >= old model  -- superset
   model <= old model  -- subset

3. CARDINALITY + CONTENTS
   model.count = expected AND model.has (expected_elements)

4. FRAME CONDITIONS (model-based)
   other_model |=| old other_model
   -- Nothing else changed
```

---

## MML Types and Their Use Cases

| MML Type | Use For | Example Domain |
|----------|---------|----------------|
| MML_SEQUENCE | Ordered, duplicates OK | List, Stack, Queue |
| MML_SET | Unordered, no duplicates | Unique IDs, Tags |
| MML_BAG | Unordered, duplicates counted | Word frequency, Inventory |
| MML_MAP | Key → Value | Dictionary, Cache |
| MML_RELATION | Key → Multiple Values | Graph edges, Permissions |
| MML_INTERVAL | Integer range | Indices, Bounds |

---

## Example: Full MML-Enhanced Class

```eiffel
class MY_STACK [G]

feature -- Model (specification layer)

    model: MML_SEQUENCE [G]
            -- Abstract stack as sequence (top = last)
        do
            create Result.make_from_list (storage.twin)
        end

feature -- Queries

    count: INTEGER
        ensure
            consistent: Result = model.count
        end

    top: G
        require
            not_empty: not is_empty
        ensure
            is_model_last: Result = model.last
        end

    is_empty: BOOLEAN
        ensure
            model_based: Result = model.is_empty
        end

feature -- Commands

    push (x: G)
        ensure
            model_extended: model |=| old model & x
        end

    pop
        require
            not_empty: not is_empty
        ensure
            model_reduced: model |=| old model.but_last
        end

    clear
        ensure
            model_empty: model.is_empty
        end

feature {NONE} -- Implementation

    storage: ARRAYED_LIST [G]

invariant
    model_storage_sync: model.count = storage.count

end
```

---

## Workflow Summary: Where MML Applies

```
┌────────────────────────────────────────────────────────────────────────┐
│ 01 PROJECT-CREATION                                                    │
│    02: Define model query signatures            ◄── MML FOUNDATION     │
│    04: Write model-based postconditions         ◄── MML CONTRACTS      │
│    05: Write model-based invariants             ◄── MML INVARIANTS     │
├────────────────────────────────────────────────────────────────────────┤
│ 02 SPEC-EXTRACTION                                                     │
│    S03: Identify model candidates               ◄── MML DISCOVERY      │
├────────────────────────────────────────────────────────────────────────┤
│ 04 SPEC-FROM-RESEARCH                                                  │
│    R04: Design model queries                    ◄── MML DESIGN         │
│    R05: Use MML in contract design              ◄── MML CONTRACTS      │
├────────────────────────────────────────────────────────────────────────┤
│ 05 DESIGN-AUDIT                                                        │
│    Assess model coverage                        ◄── MML AUDIT          │
│    Identify primitive→model upgrade opportunities                      │
├────────────────────────────────────────────────────────────────────────┤
│ 06 MAINTENANCE                                                         │
│    Verify model consistency when changing code  ◄── MML GUARDS         │
├────────────────────────────────────────────────────────────────────────┤
│ 07 MAINTENANCE-XTREME                                                  │
│    X03: Assault with model-based contracts      ◄── MML ASSAULT        │
│    X04: Adversarial tests against models                               │
├────────────────────────────────────────────────────────────────────────┤
│ 08 BUG-HUNTING                                                         │
│    Look for model/implementation mismatches     ◄── MML DETECTION      │
├────────────────────────────────────────────────────────────────────────┤
│ 09 BUG-FIXING                                                          │
│    Fix to satisfy model contracts               ◄── MML VERIFICATION   │
└────────────────────────────────────────────────────────────────────────┘
```

---

## Conclusion

**Bake MML into the spec from step 01.**

If model queries and model-based contracts exist from project creation:
- Design audit (05) verifies model coverage automatically
- Maintenance (06) has rigorous contracts to preserve
- Maintenance-xtreme (07) already has assault targets
- Bug hunting (08) can compare model vs implementation
- Bug fixing (09) has clear specification to satisfy

**MML is not a maintenance add-on. It's a specification foundation.**
