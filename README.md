# Eiffel + AI Reference Documentation

**Start here:** Run `oracle-cli.exe boot` - Oracle handles session context now

---

## Structure

```
reference_docs/
├── README.md                ← You are here
│
├── claude/                  ← AI workflow patterns
│   ├── CONTEXT.md           - Session startup context
│   ├── HATS.md              - Focused work modes
│   ├── EIFFEL_MENTAL_MODEL.md - Core Eiffel knowledge for AI
│   ├── contract_patterns.md - DBC patterns
│   ├── simple_library_design_process.md
│   └── verification_process.md
│
├── language/                ← Eiffel language knowledge
│   ├── gotchas.md           - Doc vs reality corrections
│   ├── sqlite_gotchas.md    - SQLite/DB specific issues
│   ├── patterns.md          - Verified working code
│   ├── across_loops.md      - Iteration constructs
│   ├── scoop.md             - Concurrency (SCOOP)
│   └── profiler.md          - Performance analysis
│
├── plans/                   ← Active planning docs
│   ├── competitive_assessment_2025.md - Honest head-to-head vs 2025 stacks
│   ├── simple_star_improvement_roadmap.md - Ecosystem status & roadmap
│   ├── simple_testing_migration_plan.md
│   └── simple_testing_and_win32_api_proposal.md
│
├── research/                ← Tech stack research (for building wrappers)
│   ├── SIMPLE_*_RESEARCH.md - How Go/Rust/Python do it
│   ├── LIBRARY_RELATIONSHIPS.md
│   └── inline_c_migration_report.md
│
├── deployment/              ← Deployment guides
│   └── ORACLE_CLOUD_DEPLOYMENT.md
│
├── posts/                   ← Blog post drafts
│   ├── simple_pdf_development_post.md
│   └── simple_xml_development_post.md
│
├── ECMA-367_Full.md         ← Eiffel language specification
├── eiffel_build_rules.md    ← Build guidance
└── eric_feedback_win32_libs.md ← Eric Bezault feedback
```

---

## Quick Reference

| Need | Location |
|------|----------|
| Session context | `oracle-cli.exe boot` |
| Eiffel gotchas | `language/gotchas.md` |
| SQLite issues | `language/sqlite_gotchas.md` |
| Code patterns | `language/patterns.md` |
| SCOOP concurrency | `language/scoop.md` |
| Ecosystem status | `plans/simple_star_improvement_roadmap.md` |
| Competitive analysis | `plans/competitive_assessment_2025.md` |

---

## Oracle Integration

The oracle (`simple_oracle`) now handles:
- Session handoff (replaces RESUME_POINT.md)
- Knowledge base (ingests these docs)
- Event logging (compiles, tests, git)
- Ecosystem stats

**Key commands:**
```bash
oracle-cli.exe boot           # Session start
oracle-cli.exe query "..."    # Search knowledge
oracle-cli.exe handoff "..."  # End of session
oracle-cli.exe check          # When Larry says "see oracle"
oracle-cli.exe ingest         # Re-ingest these docs
```

---

## Adding Knowledge

When you learn something:
1. **Gotcha?** → Add to `language/gotchas.md`
2. **Pattern?** → Add to `language/patterns.md`
3. **Rule/Decision?** → `oracle-cli.exe learn rule/decision "title" "content"`

---

*Last updated: 2025-12-09*
