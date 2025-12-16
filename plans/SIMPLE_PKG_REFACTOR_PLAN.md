# simple_pkg Refactor Plan: ECF as Single Source of Truth

## Design Philosophy

**ECF is the one-stop-shop for all package metadata.** No separate registry database, no package.json files. Everything lives in the ECF that every library already has.

GitHub provides:
- Repository listing (API)
- Raw ECF file access (raw.githubusercontent.com)
- Git tags for versions

simple_pkg provides:
- ECF parsing for metadata extraction
- Dependency resolution from `<library>` tags
- FTS5 in-memory search index (rebuilt from fetched ECFs)
- Local caching for performance

---

## ECF Metadata Standard

### Required Fields (add to all simple_* ECFs)

```xml
<system name="simple_json" uuid="..." library_target="simple_json">

  <!-- REQUIRED: Human-readable description -->
  <description>High-performance JSON parser with Schema, Patch, Pointer support</description>

  <!-- REQUIRED: Semantic version -->
  <version major="1" minor="2" release="0" build="0"/>

  <!-- NEW: Package metadata in note section -->
  <note>
    <package>
      <author>Larry Rix</author>
      <keywords>json, parser, serialization, schema, rfc8259</keywords>
      <category>data-formats</category>
      <license>MIT</license>
    </package>
  </note>

  <target name="simple_json">
    <!-- Dependencies detected from library tags -->
    <library name="simple_datetime" location="$SIMPLE_DATETIME/simple_datetime.ecf"/>
    <library name="simple_regex" location="$SIMPLE_REGEX/simple_regex.ecf"/>
  </target>
</system>
```

### Categories (predefined)
- `data-formats` - JSON, XML, YAML, CSV, etc.
- `networking` - HTTP, WebSocket, gRPC, SMTP
- `database` - SQL, MongoDB, Redis
- `security` - Encryption, JWT, hashing
- `platform` - Win32, system, process
- `utilities` - DateTime, UUID, validation
- `testing` - Testing framework, mocks
- `web` - Web framework, CORS, middleware
- `ai` - AI clients, embeddings

---

## Implementation Plan

### Phase 1: ECF Metadata Standardization

**Task 1.1: Create ECF_METADATA class**
- Location: `/d/prod/simple_pkg/src/ecf_metadata.e`
- Parse ECF using simple_xml
- Extract: name, description, version, author, keywords, category, dependencies

```eiffel
class ECF_METADATA
feature
  name: STRING
  description: STRING
  version: STRING  -- "1.2.0"
  author: STRING
  keywords: ARRAYED_LIST [STRING]
  category: STRING
  dependencies: ARRAYED_LIST [STRING]  -- simple_* names only

  make_from_ecf_content (a_xml: STRING)
    -- Parse ECF XML and populate fields
```

**Task 1.2: Update 69 ECF files with metadata**
- Add `<version>` tag to all ECFs (start at 0.1.0 or use git tag if exists)
- Add `<note><package>...</package></note>` section
- Script to batch-update ECFs

### Phase 2: Dynamic GitHub Integration

**Task 2.1: Enhance PKG_REGISTRY for ECF fetching**
- Location: `/d/prod/simple_pkg/src/pkg_registry.e`
- Add: `fetch_ecf_metadata (a_name: STRING): ECF_METADATA`
- Fetch raw ECF from: `https://raw.githubusercontent.com/simple-eiffel/{name}/main/{name}.ecf`
- Parse with ECF_METADATA class

**Task 2.2: Dependency extraction from ECF**
- Parse `<library location="$SIMPLE_*">` tags
- Filter: only extract `$SIMPLE_*` dependencies (not `$ISE_LIBRARY`)
- Return list of simple_* package names

**Task 2.3: Version detection**
- Primary: `<version>` tag in ECF
- Fallback: latest git tag via GitHub API
- Default: "main" if no version info

### Phase 3: Fuzzy Search with FTS5

**Task 3.1: Create in-memory search index**
- Location: `/d/prod/simple_pkg/src/pkg_search.e`
- Use simple_sql with in-memory SQLite (`:memory:`)
- FTS5 table: `packages_fts (name, description, keywords, category)`

```eiffel
class PKG_SEARCH
feature
  build_index (a_packages: ARRAYED_LIST [ECF_METADATA])
    -- Create FTS5 table and populate from metadata

  search (a_query: STRING): ARRAYED_LIST [SEARCH_RESULT]
    -- FTS5 MATCH with BM25 ranking

  browse_category (a_category: STRING): ARRAYED_LIST [ECF_METADATA]
    -- Filter by category
```

**Task 3.2: Search command enhancement**
- `simple search json` → fuzzy FTS5 search
- `simple search --category data-formats` → category browse
- `simple search --deps simple_http` → find packages using simple_http

### Phase 4: Automatic Dependency Installation

**Task 4.1: Enhance PKG_RESOLVER**
- Location: `/d/prod/simple_pkg/src/pkg_resolver.e`
- Use ECF_METADATA.dependencies instead of manual tracking
- Recursive resolution with cycle detection (already exists)

**Task 4.2: Install with dependencies**
- `simple install simple_web` automatically installs:
  - simple_json (dependency)
  - simple_datetime (dependency)
  - simple_http (dependency)
  - simple_web (requested)

### Phase 5: Versioning Support

**Task 5.1: Version constraint syntax**
- `simple install simple_json@1.2.0` - exact version
- `simple install simple_json@^1.0` - compatible (future)
- Default: latest version (git tag or main)

**Task 5.2: Lock file generation**
- Location: `.simple-lock.json` in project root
- Records exact versions installed
- Format:
```json
{
  "packages": {
    "simple_json": "1.2.0",
    "simple_datetime": "1.0.0"
  },
  "locked_at": "2025-12-14T10:30:00Z"
}
```

**Task 5.3: `simple install` from lock file**
- If `.simple-lock.json` exists, use locked versions
- `simple install --update` to refresh lock file

### Phase 6: Local Caching

**Task 6.1: ECF cache**
- Location: `~/.simple/cache/ecf/`
- Cache fetched ECFs with TTL (1 hour default)
- `simple search --refresh` to bypass cache

**Task 6.2: Search index cache**
- Rebuild FTS5 index only when cache is stale
- Store index in `~/.simple/cache/search.db`

---

## File Changes Summary

### New Files
- `/d/prod/simple_pkg/src/ecf_metadata.e` - ECF parsing and metadata
- `/d/prod/simple_pkg/src/pkg_search.e` - FTS5 search engine
- `/d/prod/simple_pkg/src/pkg_lock.e` - Lock file management

### Modified Files
- `/d/prod/simple_pkg/src/pkg_registry.e` - Add ECF fetching
- `/d/prod/simple_pkg/src/pkg_resolver.e` - Use ECF dependencies
- `/d/prod/simple_pkg/src/pkg_cli.e` - Enhanced search/browse commands
- `/d/prod/simple_pkg/src/pkg_info.e` - Add version, keywords, category fields
- `/d/prod/simple_pkg/simple_pkg.ecf` - Add simple_sql dependency

### ECF Updates (all 69 libraries)
- Add `<version>` tag
- Add `<note><package>` metadata section

---

## Command Enhancements

| Command | Current | Enhanced |
|---------|---------|----------|
| `simple search json` | Substring match | FTS5 fuzzy + BM25 ranking |
| `simple search --cat web` | N/A | Browse by category |
| `simple install foo` | Clone only | Clone + install dependencies |
| `simple install foo@1.0` | N/A | Version-specific install |
| `simple info foo` | Basic | Full metadata from ECF |
| `simple outdated` | N/A | Compare installed vs latest |
| `simple lock` | N/A | Generate lock file |

---

## Data Flow

```
simple search "json parser"
  │
  ├─ Check cache (~/.simple/cache/search.db)
  │   └─ If fresh: query FTS5 index
  │
  ├─ If stale/missing:
  │   ├─ GitHub API: GET /orgs/simple-eiffel/repos
  │   ├─ For each repo:
  │   │   └─ Fetch raw ECF from GitHub
  │   │   └─ Parse with ECF_METADATA
  │   ├─ Build FTS5 index in memory
  │   └─ Cache to disk
  │
  └─ Return ranked results with snippets


simple install simple_web
  │
  ├─ Fetch ECF_METADATA for simple_web
  ├─ Extract dependencies: [simple_json, simple_datetime, simple_http]
  ├─ Recursive: fetch ECF for each dependency
  ├─ Topological sort (dependencies first)
  ├─ For each in order:
  │   ├─ Check if already installed
  │   ├─ Git clone from GitHub
  │   └─ Set environment variable
  └─ Update lock file
```

---

## Priority Order

1. **ECF_METADATA class** - Foundation for everything
2. **Dependency extraction from ECF** - Enables auto-install
3. **PKG_RESOLVER using ECF dependencies** - The "gimme" feature
4. **FTS5 search index** - Fuzzy search
5. **Version support** - @version syntax
6. **Lock file** - Reproducible installs
7. **ECF standardization script** - Update all 69 libraries

---

## Dependencies

simple_pkg will need:
- simple_xml (ECF parsing) - already available
- simple_sql (FTS5 search) - add to ECF
- simple_json (lock file, cache) - already included
- simple_http (GitHub API) - already included

---

## Success Criteria

- [ ] `simple search json` returns ranked results with snippets
- [ ] `simple search --cat database` shows simple_sql, simple_mongo
- [ ] `simple install simple_web` automatically installs 4+ dependencies
- [ ] `simple info simple_json` shows version, author, keywords, dependencies
- [ ] `.simple-lock.json` generated on install
- [ ] All 69 ECFs have standardized metadata
