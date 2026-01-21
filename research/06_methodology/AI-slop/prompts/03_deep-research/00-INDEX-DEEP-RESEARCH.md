# Deep Research Workflow: 7-Step Investigation Process

## CRITICAL: ANTI-SLOP ENFORCEMENT

**Before using this workflow, read: `../ANTI-SLOP-RULES.md`**

This workflow requires ACTUAL:
- Web searches with real URLs (WebSearch/WebFetch tools)
- Documentation written to actual files in `{lib}/research/`
- Sources cited with actual URLs

**FORBIDDEN**:
- Making up library names or features
- Citing documentation without actual URLs
- Claiming "research complete" without written deliverables

---

## Overview

7 sequential prompts for systematically researching a new library or feature idea before design and implementation. This workflow transforms a vague idea into a concrete, validated specification ready for the design phase.

## The Problem

Starting to code without research leads to:
- Reinventing existing solutions
- Missing important requirements
- Making uninformed design decisions
- Discovering problems late
- Building the wrong thing

**Solution:** A structured research process that validates ideas, explores alternatives, and makes informed decisions before any code is written.

## The 7-Step Framework

```
┌─────────────────────────────────────────────────────────────┐
│                    STEP 1: SCOPE                            │
│              (Define what we're researching)                │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    STEP 2: LANDSCAPE                        │
│              (What already exists?)                         │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    STEP 3: REQUIREMENTS                     │
│              (What must it do?)                             │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    STEP 4: DECISIONS                        │
│              (What design choices?)                         │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    STEP 5: INNOVATIONS                      │
│              (What's novel?)                                │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    STEP 6: RISKS                            │
│              (What could go wrong?)                         │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                    STEP 7: RECOMMENDATION                   │
│              (What should we do?)                           │
└─────────────────────────────────────────────────────────────┘
```

## Prompt Files

| # | File | Purpose | Key Questions |
|---|------|---------|---------------|
| 7S-01 | SCOPE | Define boundaries | What problem? Who for? What's in/out? |
| 7S-02 | LANDSCAPE | Survey existing | What exists? Strengths? Weaknesses? |
| 7S-03 | REQUIREMENTS | Define needs | Functional? Non-functional? Constraints? |
| 7S-04 | DECISIONS | Make choices | Options? Trade-offs? Rationale? |
| 7S-05 | INNOVATIONS | Identify novel | What's new? What's different? |
| 7S-06 | RISKS | Identify risks | What fails? Likelihood? Mitigation? |
| 7S-07 | RECOMMENDATION | Synthesize | Build? Buy? Adapt? How? |

## Research Philosophy

### Be Thorough
Don't skip steps. Each step builds on the previous ones.

### Be Critical
Challenge every assumption. Question every requirement.

### Be Honest
Acknowledge unknowns. Admit when alternatives are better.

### Be Practical
Focus on actionable insights. Avoid analysis paralysis.

## Input Format

Start with an IDEA:

```
IDEA: {one-sentence description}

CONTEXT: {why we want this}

INITIAL THOUGHTS: {any preliminary ideas}

CONSTRAINTS: {known limitations}
```

## Output Format

The workflow produces:

```
research-output/
├── STEP-1-SCOPE.md           # Problem definition
├── STEP-2-LANDSCAPE.md       # Existing solutions
├── STEP-3-REQUIREMENTS.md    # Detailed requirements
├── STEP-4-DECISIONS.md       # Design decisions
├── STEP-5-INNOVATIONS.md     # Novel approaches
├── STEP-6-RISKS.md           # Risk analysis
├── STEP-7-RECOMMENDATION.md  # Final recommendation
└── APPENDIX/                 # Supporting materials
    ├── ALTERNATIVES.md
    ├── REFERENCES.md
    └── PROTOTYPES/
```

## Integration with Other Workflows

```
┌──────────────────────┐
│    INITIAL IDEA      │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│  DEEP RESEARCH       │  ← This workflow
│  (7 Steps)           │
└──────────┬───────────┘
           │ produces research output
           ▼
┌──────────────────────┐
│  SPEC-FROM-RESEARCH  │  ← Next workflow
│  (8 Prompts)         │
└──────────┬───────────┘
           │ produces specification
           ▼
┌──────────────────────┐
│  IMPLEMENTATION      │
│  + MAINTENANCE       │
└──────────────────────┘
```

## When to Use This Workflow

1. **New library** - Before creating a new simple_* library
2. **Major feature** - Before adding significant capability
3. **Technology selection** - Choosing between approaches
4. **Problem understanding** - When problem is unclear
5. **Stakeholder alignment** - Getting agreement on direction

## Research Quality Criteria

### Step 1 (Scope)
- [ ] Problem clearly stated
- [ ] Boundaries defined
- [ ] Success criteria established

### Step 2 (Landscape)
- [ ] At least 3 alternatives researched
- [ ] Strengths/weaknesses documented
- [ ] Patterns identified

### Step 3 (Requirements)
- [ ] Functional requirements prioritized
- [ ] Non-functional requirements quantified
- [ ] Constraints identified

### Step 4 (Decisions)
- [ ] Options evaluated
- [ ] Trade-offs documented
- [ ] Rationale recorded

### Step 5 (Innovations)
- [ ] Novel aspects identified
- [ ] Differentiation clear
- [ ] Value proposition stated

### Step 6 (Risks)
- [ ] Risks identified
- [ ] Likelihood assessed
- [ ] Mitigations planned

### Step 7 (Recommendation)
- [ ] Clear recommendation
- [ ] Evidence-based
- [ ] Actionable next steps

## Time Investment

| Step | Typical Time | Notes |
|------|--------------|-------|
| 1. Scope | 1-2 hours | Get agreement on boundaries |
| 2. Landscape | 4-8 hours | Thorough research of alternatives |
| 3. Requirements | 2-4 hours | Detailed requirement analysis |
| 4. Decisions | 2-4 hours | Careful option evaluation |
| 5. Innovations | 1-2 hours | Identify differentiation |
| 6. Risks | 1-2 hours | Risk assessment |
| 7. Recommendation | 1-2 hours | Synthesis and recommendation |
| **Total** | **12-24 hours** | Spread over days recommended |

The investment saves multiples of this time by avoiding wrong paths.

CRITICAL: STOP here and tell me what the next step is and prompt me to tell you to continue with my added instruction.
