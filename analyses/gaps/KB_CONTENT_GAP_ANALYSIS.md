# KB Content Gap Analysis

**Date:** 2025-12-25
**Status:** Phase 1 Complete
**Purpose:** Identify content gaps to guide FAQ/instructional pair creation

---

## Current State

### Indexed Content (Excellent)
| Content Type | Count | Quality |
|--------------|-------|---------|
| Libraries | 182 | High - Full ECF metadata |
| Classes | 4,613 | High - Signatures, descriptions, parents |
| Features | 87,780 | High - Contracts, kinds, descriptions |
| Examples (Rosetta) | 273 | High - Complete working code |

### Seeded Content (Medium)
| Content Type | Count | Quality | Notes |
|--------------|-------|---------|-------|
| Error Codes | 31 | Medium | Good coverage, need more examples |
| Patterns | 28 (14 unique) | Medium | Duplicates exist, need cleanup |

### Missing Content (Critical Gap)
| Content Type | Count | Status |
|--------------|-------|--------|
| FAQs | 0 | **Empty - Primary gap** |
| Instructional Pairs | 0 | Not implemented yet |
| Cross-language translations | 0 | Not implemented yet |

---

## Top 20 Questions Developers Would Ask

### Newcomer Questions (High Priority)
1. "How do I create a class in Eiffel?"
2. "What is Design by Contract?"
3. "How do I declare variables in Eiffel?"
4. "What are preconditions and postconditions?"
5. "How do I create an object in Eiffel?"
6. "What is the difference between once and regular features?"
7. "How do I handle strings in Eiffel?"
8. "What is an invariant?"
9. "How do I write a loop in Eiffel?"
10. "What does 'attached' mean?"

### Language Migrator Questions (High Priority)
11. "How do I do X in Eiffel?" (where X is common in Java/Python/C#)
12. "What's the Eiffel equivalent of an interface?"
13. "How does Eiffel handle null/None/nil?"
14. "Does Eiffel have generics/templates?"
15. "How do I handle exceptions in Eiffel?"

### Intermediate/Advanced Questions
16. "How do I fix VEVI error?"
17. "What is SCOOP and how do I use it?"
18. "How do I use agents (lambdas/closures)?"
19. "How do I implement multiple inheritance correctly?"
20. "How do I make my code thread-safe?"

---

## Content Categories to Create

### Category 1: Newcomer FAQ (Target: 100 pairs)
**Hat:** Newcomer
**Topics:**
- Basic syntax (class, feature, variable declaration)
- Object creation (create, default_create)
- Control structures (if, loop, across)
- Basic DBC (require, ensure, invariant)
- String handling
- Collections basics

### Category 2: Language Migration (Target: 120 pairs)
**Hats:** Java Migrator, Python Migrator, C# Migrator

| Source Concept | Eiffel Equivalent |
|----------------|-------------------|
| interface | deferred class |
| abstract class | deferred class with deferred features |
| null checks | void safety (attached/detachable) |
| try/catch | rescue/retry |
| generics | constrained generics [G -> CONSTRAINT] |
| properties | attributes + setters |
| annotations | notes |
| lambda | agent |
| foreach | across |
| enum | expanded class or INTEGER_x constants |

### Category 3: DBC Deep Dive (Target: 60 pairs)
**Hat:** DBC Expert
**Topics:**
- Precondition design (require)
- Postcondition design (ensure)
- Class invariant design
- Old expression usage
- Contract inheritance
- Check instructions
- Loop variants/invariants

### Category 4: Void Safety (Target: 40 pairs)
**Hat:** Void Safety Expert
**Topics:**
- attached vs detachable
- Object test (attached x as lx)
- CAP (Certified Attachment Patterns)
- Converting legacy code
- VUTA, VEVI, VJAR errors

### Category 5: SCOOP Concurrency (Target: 40 pairs)
**Hat:** SCOOP Expert
**Topics:**
- separate keyword
- Region model
- Inline separate syntax
- Wait conditions
- Concurrency patterns

### Category 6: Error Code Explanations (Target: 93 pairs)
**Hat:** Debugger
**Format:** 3 pairs per error code (31 codes)
- What does this error mean?
- Show me an example that causes this error
- How do I fix this error?

### Category 7: Pattern Implementation (Target: 84 pairs)
**Hat:** Architect
**Format:** 3 pairs per pattern (28 patterns - deduplicated to 14)
- When should I use this pattern?
- Show me the Eiffel implementation
- What are Eiffel-specific considerations?

### Category 8: Class-Driven FAQ (Target: 600 pairs)
**Source:** Top 200 most-used classes from KB
**Format:** 3 pairs per class
- What is this class used for?
- Show me basic usage example
- What are the key features?

---

## Priority Order for Content Creation

### Phase A: Seed Content (Days 1-2)
1. Clean up pattern duplicates (28 -> 14)
2. Create basic FAQ seeder with top 20 questions
3. Extend schema for instructional pair format

### Phase B: Core FAQ (Days 3-5)
1. Newcomer FAQ: 100 pairs
2. Error explanations: 93 pairs
3. Pattern FAQ: 84 pairs

### Phase C: Migration Content (Days 6-8)
1. Java migration: 40 pairs
2. Python migration: 40 pairs
3. C# migration: 40 pairs

### Phase D: Advanced Topics (Days 9-11)
1. DBC deep dive: 60 pairs
2. Void safety: 40 pairs
3. SCOOP: 40 pairs

### Phase E: Class-Driven (Days 12-14)
1. Top 100 simple_* classes: 300 pairs
2. Top 100 ISE stdlib classes: 300 pairs

---

## Issues Found

### Pattern Duplicates
The patterns table has 28 entries but only 14 unique patterns:
- singleton (x2)
- factory (x2)
- builder (x2)
- template_method (x2)
- observer (x2)
- command (x2)
- visitor (x2)
- iterator (x2)
- null_object (x2)
- once_per_object (x2)
- expanded_value (x2)
- agent_callback (x2)
- dbc_validation (x2)
- attachment_check (x2)

**Action:** Run cleanup to remove duplicates

### Missing FAQ Seeder
No KB_FAQ_SEEDER class exists. Need to create one that populates initial FAQ content.

---

## Estimated Content Totals

| Category | Target Count |
|----------|-------------|
| Newcomer FAQ | 100 |
| Language Migration | 120 |
| DBC Deep Dive | 60 |
| Void Safety | 40 |
| SCOOP | 40 |
| Error Explanations | 93 |
| Pattern FAQ | 84 (42 after dedup) |
| Class-Driven | 600 |
| **Total** | **~1,137 pairs** |

---

## Success Metrics

1. **Coverage:** At least one FAQ per major topic
2. **Quality:** FAQs compile-tested where code is included
3. **Accessibility:** Questions phrased as developers naturally ask
4. **RAG Effectiveness:** 4.0/5.0 average answer quality

---

## Next Step

Proceed to Phase 2: Schema Enhancement
- Add instructional pair fields to FAQ table
- Create KB_FAQ_SEEDER class
- Clean up pattern duplicates
