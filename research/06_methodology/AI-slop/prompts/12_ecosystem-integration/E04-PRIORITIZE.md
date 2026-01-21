# E04: Prioritize Integration Opportunities

## Objective
Rank candidates by impact and effort to create an actionable integration roadmap.

## Steps

### 1. Score Each Candidate

Use this scoring matrix:

**Impact Score (1-5)**
| Score | Description |
|-------|-------------|
| 5 | Transformative - enables major new capabilities |
| 4 | High - significant improvement to common use case |
| 3 | Medium - useful improvement, moderate usage |
| 2 | Low - minor improvement, niche usage |
| 1 | Minimal - nice to have, rarely used |

**Effort Score (1-5)**
| Score | Description |
|-------|-------------|
| 1 | Trivial - add dependency, use directly |
| 2 | Low - minor wrapper/adapter needed |
| 3 | Medium - moderate refactoring required |
| 4 | High - significant changes, API updates |
| 5 | Major - architectural changes, extensive testing |

### 2. Calculate Priority

```
Priority = Impact / Effort
```

Higher is better (high impact, low effort).

### 3. Create Priority Matrix

| Candidate | Impact | Effort | Priority | Tier |
|-----------|--------|--------|----------|------|
| {lib_1} | 5 | 2 | 2.5 | 1 |
| {lib_2} | 4 | 3 | 1.3 | 2 |
| {lib_3} | 3 | 4 | 0.75 | 3 |

### 4. Assign Tiers

| Tier | Priority Range | Recommendation |
|------|----------------|----------------|
| 1 | > 2.0 | Integrate immediately |
| 2 | 1.0 - 2.0 | Integrate soon |
| 3 | < 1.0 | Consider for future |

### 5. Identify Quick Wins

Quick wins are:
- Tier 1 candidates
- LOW complexity from E03
- Library is actively maintained

### 6. Identify Strategic Integrations

Strategic integrations are:
- HIGH impact even if HIGH effort
- Enable new ecosystem capabilities
- Justify a dedicated integration phase

## Output
Prioritized list with tiers and recommendations.

## Verification Checkpoint
- [ ] All candidates scored
- [ ] Priorities calculated
- [ ] Tiers assigned
- [ ] Quick wins identified
- [ ] Strategic integrations noted

## Next Step
â†’ E05-DOCUMENT-OPPORTUNITIES.md
