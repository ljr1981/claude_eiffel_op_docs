# 12: Ecosystem Integration Analysis

## Purpose
After a library is production-ready and published, identify which existing simple_* libraries could benefit from integrating the new library.

## When to Use
- Library is published to GitHub (workflow 11 complete)
- Library provides reusable capabilities (encoding, factory patterns, reflection, etc.)
- Want to find integration opportunities across the ecosystem

## Prerequisites
- Library is v1.0.0 or higher
- All tests passing
- Published to GitHub
- README and docs complete

## Steps

| Step | File | Purpose |
|------|------|---------|
| E01 | E01-IDENTIFY-CAPABILITIES.md | Document what the library provides |
| E02 | E02-SCAN-CANDIDATES.md | Find libraries that could benefit |
| E03 | E03-ANALYZE-INTEGRATION.md | Analyze integration opportunities |
| E04 | E04-PRIORITIZE.md | Rank by impact and effort |
| E05 | E05-DOCUMENT-OPPORTUNITIES.md | Create integration report |

## Output
- `reference_docs/analyses/SIMPLE_{LIB}_INTEGRATION_OPPORTUNITIES.md`

## Verification Checkpoint
- [ ] Capabilities documented
- [ ] Candidate libraries identified
- [ ] Integration opportunities analyzed
- [ ] Priorities assigned
- [ ] Report saved to reference_docs/analyses/

## Next Workflow
None - this is the final workflow in the library lifecycle.

## Example Usage
```
Library: simple_reflection
Capabilities: type introspection, field access, object graph walking
Candidates: simple_json (auto-serialization), simple_mock (dynamic mocks), simple_diff (object comparison)
```
