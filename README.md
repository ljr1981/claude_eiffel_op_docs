# Eiffel Reference Documentation

Reference documentation for Eiffel development, Claude AI workflows, and strategic planning.

---

## Quick Start

**Starting a new Claude session?** Read these first:
1. [`claude/CONTEXT.md`](claude/CONTEXT.md) - Session startup protocol
2. Look for `ROADMAP.md` in the project you're working on
3. Ask "What would you like to work on this session?"

---

## Folder Structure

### [`claude/`](claude/) - AI Session Management

Documents for Claude AI workflow and collaboration patterns.

| File | Purpose |
|------|---------|
| [CONTEXT.md](claude/CONTEXT.md) | **Read first** - Session startup, general Eiffel knowledge |
| [HATS.md](claude/HATS.md) | Focused work modes (refactoring, contracting, testing, etc.) |
| [contract_patterns.md](claude/contract_patterns.md) | Design by Contract patterns for AI-assisted development |
| [verification_process.md](claude/verification_process.md) | Code verification workflow |
| [mentoring_mode.md](claude/mentoring_mode.md) | Teaching and explanation mode |

### [`language/`](language/) - Eiffel Language Reference

Practical knowledge about Eiffel that supplements official documentation.

| File | Purpose |
|------|---------|
| [gotchas.md](language/gotchas.md) | "Docs say X but reality is Y" corrections |
| [patterns.md](language/patterns.md) | Verified working code patterns |
| [across_loops.md](language/across_loops.md) | Iteration constructs, cursors, `.item` behavior |
| [scoop.md](language/scoop.md) | SCOOP concurrency model |
| [profiler.md](language/profiler.md) | EiffelStudio profiler usage |
| [sqlite_gotchas.md](language/sqlite_gotchas.md) | SQLite/database-specific issues |

### [`strategy/`](strategy/) - Business & Strategy

Market analysis, competitive positioning, and development roadmaps.

| File | Purpose |
|------|---------|
| [LIBRARY_ROADMAP.md](strategy/LIBRARY_ROADMAP.md) | **NEW** - Gap analysis vs. modern ecosystem |
| [COMPETITIVE_ANALYSIS.md](strategy/COMPETITIVE_ANALYSIS.md) | Market positioning analysis |
| [AI_PRODUCTIVITY.md](strategy/AI_PRODUCTIVITY.md) | AI productivity research and data |
| [REFACTORING_CASE_STUDY.md](strategy/REFACTORING_CASE_STUDY.md) | Real-world refactoring case study |

### [`archive/`](archive/) - Completed Project Plans

Historical project planning documents (reference only).

| File | Project |
|------|---------|
| simple_showcase_content.md | Content blueprint for showcase site |
| simple_showcase_plan.md | Initial project plan |
| simple_alpine_plan.md | Alpine.js wrapper project |
| simple_htmx_creation.md | HTMX wrapper project |
| simple_sql_*.md | SQL library planning |
| current_work.md | Historical work tracking |

---

## Project Convention

Each Eiffel project should have a `ROADMAP.md` in its root containing:
- Project overview and current status
- Dependencies and build commands
- Current focus and backlog
- Session notes

This keeps reference docs generic while project details live with each project.

---

## Contributing

When adding new learnings:
1. Add to the appropriate folder/file
2. Include verification date and EiffelStudio version
3. Provide working code examples
4. Follow existing format in each file

---

## Last Updated

2025-12-05 - Reorganized into folders: claude/, language/, strategy/, archive/
