# 01: Analyze Requirements (Build Specification Layer)

## Context
Working Hat: **SPECIFICATION** (Phase 1 of 4)
Layer Being Built: **SPECIFICATION LAYER**

## Input Required
- Natural language requirements OR existing code to wrap OR API to implement
- Problem domain description

## Prompt

```
TASK: Build the SPECIFICATION LAYER by extracting formal requirements.

INPUT:
{requirements}
{problem_domain}

═══════════════════════════════════════════════════════════════
SPECIFICATION EXTRACTION
The specification layer defines WHAT the system should do.
═══════════════════════════════════════════════════════════════

OUTPUT FORMAT:

1. DOMAIN MODEL
   - Key domain concepts and their relationships
   - Domain rules that must ALWAYS hold
   - Domain rules that must NEVER be violated

2. ENTITIES: List each noun as potential class
   - Entity: {name}
   - Domain meaning: {what it represents}
   - Domain rules: {rules that apply to this entity}

3. ACTIONS: List each verb as potential feature
   - Action: {name}
   - Domain meaning: {what it accomplishes}
   - Valid when: {domain conditions}
   - Result: {domain effect}

4. CONSTRAINTS: Classify as specification element
   - "must X" → postcondition candidate (feature guarantees X)
   - "cannot X" → precondition candidate (caller ensures not X)
   - "always X" → invariant candidate (class maintains X)
   - "never X" → precondition or invariant candidate

5. RELATIONSHIPS: List inheritance/composition relationships
   - Relationship: {A} → {B}
   - Type: [inheritance | composition | uses]
   - Domain justification: {why this relationship}

6. QUERIES vs COMMANDS: Classify each action
   - Query: returns value, no state change
   - Command: modifies state, no return value

═══════════════════════════════════════════════════════════════
SPECIFICATION QUALITY CHECKS
═══════════════════════════════════════════════════════════════

[ ] Every domain concept has clear definition?
[ ] Every domain rule captured?
[ ] Every feature has domain meaning?
[ ] Ambiguities explicitly flagged?

RULES:
- Do NOT write code
- Do NOT implement anything
- ONLY extract and classify requirements
- Flag ambiguities as UNCLEAR: [item]
- Everything traces back to domain requirements
```

## Success Criteria
- Domain model explicitly captured
- All nouns mapped to classes with domain meaning
- All verbs mapped to features with domain effect
- All constraints captured for contract writing
- Domain rules identified for invariants
- Ambiguities explicitly flagged

## Next Step
→ 02-DEFINE-CLASS-STRUCTURE.md
