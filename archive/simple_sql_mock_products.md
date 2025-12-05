# SIMPLE_SQL Mock Product Ideas

These mock products would serve as tertiary-level testing by consuming the SIMPLE_SQL library in real-world scenarios. Unit tests (secondary) and DBC (primary) provide foundational coverage, but actual consumer code exposes integration issues and usage patterns.

---

## 1. Task Manager / Todo App

**Exercises**: Core CRUD, query builders, migrations, repository pattern

Features:
- Tasks with priorities, due dates, tags
- Projects containing tasks
- Schema migrations as features evolve
- Repository pattern for Task/Project entities

**Coverage Focus**:
- SIMPLE_SQL_DATABASE (CRUD)
- SIMPLE_SQL_QUERY_BUILDER (SELECT/INSERT/UPDATE/DELETE)
- SIMPLE_SQL_MIGRATION_RUNNER
- SIMPLE_SQL_REPOSITORY

---

## 2. Document Search Engine

**Exercises**: FTS5, BLOB handling, streaming, batch operations

Features:
- Store documents with full-text indexing
- Binary attachments (PDFs, images as BLOBs)
- BM25 ranking for search results
- Cursor-based iteration for large result sets

**Coverage Focus**:
- SIMPLE_SQL_FTS5, SIMPLE_SQL_FTS5_QUERY
- SIMPLE_SQL_PREPARED_STATEMENT (bind_blob)
- SIMPLE_SQL_CURSOR, SIMPLE_SQL_RESULT_STREAM
- SIMPLE_SQL_BATCH

---

## 3. Audit Log Viewer

**Exercises**: Audit/change tracking, JSON queries, export/import

Features:
- Track changes to entities over time
- Query change history with JSON diff analysis
- Export audit trails to CSV/JSON for compliance
- Import historical data from backups

**Coverage Focus**:
- SIMPLE_SQL_AUDIT
- SIMPLE_SQL_JSON
- SIMPLE_SQL_EXPORT, SIMPLE_SQL_IMPORT
- SIMPLE_SQL_ONLINE_BACKUP

---

## 4. ML Embedding Store (Semantic Search)

**Exercises**: Vector embeddings, similarity metrics, backup

Features:
- Store text chunks with vector embeddings
- K-nearest neighbor search for semantic similarity
- Online backup for model checkpoints
- Metadata queries via JSON

**Coverage Focus**:
- SIMPLE_SQL_VECTOR, SIMPLE_SQL_VECTOR_STORE
- SIMPLE_SQL_SIMILARITY
- SIMPLE_SQL_ONLINE_BACKUP
- SIMPLE_SQL_JSON (metadata)

---

## 5. Configuration Manager

**Exercises**: PRAGMA config, schema introspection, prepared statements

Features:
- Key-value settings with typed access
- Schema versioning and migrations
- Cached prepared statements for frequent lookups
- WAL mode for concurrent access

**Coverage Focus**:
- SIMPLE_SQL_PRAGMA_CONFIG
- SIMPLE_SQL_SCHEMA, SIMPLE_SQL_TABLE_INFO
- SIMPLE_SQL_PREPARED_STATEMENT
- SIMPLE_SQL_MIGRATION_RUNNER

---

## Implementation Priority

Recommended order based on coverage breadth:

1. **Task Manager** - Core patterns, good foundation
2. **Document Search Engine** - FTS5 + BLOB stress testing
3. **ML Embedding Store** - Vector/similarity edge cases
4. **Audit Log Viewer** - Export/import round-trip validation
5. **Configuration Manager** - PRAGMA and schema edge cases

---

## Notes

- Each mock should have its own ECF target
- Can be added to simple_sql repository or separate repo
- Should include realistic data volumes for performance testing
- Consider adding to CI/CD pipeline

Created: 2025-12-01
