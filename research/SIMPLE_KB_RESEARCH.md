# simple_kb Research Notes

**Date:** 2025-12-23
**Status:** In Progress
**Goal:** Design an Eiffel Knowledge Base CLI with FTS5 search

---

## Step 1: Deep Web Research - Existing Developer Search Tools

### Overview of Developer Documentation Tools

| Tool | Platform | Type | Key Feature |
|------|----------|------|-------------|
| rustup doc / docs.rs | Web + CLI | Language-specific | Type-aware search, press `s` to search |
| pkg.go.dev | Web + CLI | Language-specific | Symbol search with `#`, Postgres FTS |
| Dash | macOS | Aggregator | 200+ docsets, instant fuzzy search |
| Zeal | Windows/Linux | Aggregator | Dash-compatible, 207+ docsets |
| DevDocs.io | Web | Aggregator | 100+ docs, offline PWA, open source |
| Hoogle | Web + CLI | Language-specific | Type signature search for Haskell |
| Sourcegraph | Web | Code search | Enterprise scale, semantic navigation |
| ripgrep | CLI | File search | Fastest regex search, respects gitignore |

---

### Rust Documentation (rustup doc / docs.rs)

**Architecture:**
- `rustup doc` opens local documentation in browser
- `docs.rs` auto-builds docs for all crates on crates.io
- Uses `rustdoc` to generate searchable HTML

**Search Features:**
| Feature | Description |
|---------|-------------|
| In-page search | Press `s` or `/` to search |
| Type-based search | `fn:from_str` finds functions by signature |
| Prefix search | Filter by modules, structs, traits |
| Version switching | View docs for any published version |

**Key Insight:** Rust's search is deeply integrated with the type system - you can search by function signature, not just name.

---

### Go Documentation (pkg.go.dev)

**Architecture:**
- Frontend: HTTP server composing HTML from templates
- Backend: Postgres with text-search features
- Worker: Downloads modules, extracts READMEs, licenses, docs
- Index: `search_documents` table with importer counts

**Search Features:**
| Feature | Description |
|---------|-------------|
| Symbol search | Use `#` filter: `golang.org/x #error` |
| Natural search | Keyword search like "data router" |
| Local mode | `pkgsite -open .` for local docs |

**Database Schema Insight:**
```
Worker extracts:
- README files
- License files
- Documentation
- search_documents table (FTS-optimized)
- Importer counts (popularity ranking)
```

**Key Insight:** First sentence of package comment is indexed for search results - content structure matters.

---

### Dash (macOS) / Zeal (Windows/Linux)

**Architecture:**
- Local SQLite database per docset
- Pre-built docsets from documentation sources
- Fuzzy search across all enabled docsets

**Features:**
| Feature | Dash | Zeal |
|---------|------|------|
| Docsets | 200+ | 207+ (same source) |
| Offline | Yes | Yes |
| Fuzzy search | Yes | Yes |
| IDE integration | VS Code, Sublime, etc. | Atom, Emacs, Vim |
| Scope filter | `language:query` | `language:query` |

**Docset Format:**
- SQLite database with `searchIndex` table
- HTML files for content
- `Info.plist` for metadata

**Key Insight:** Docsets are just SQLite + HTML. We could create an Eiffel docset.

---

### DevDocs.io

**Architecture:**
- Ruby/Rails web application
- Scrapes documentation from sources
- Converts to uniform format
- Indexes for instant text search
- Service Worker for offline support

**Features:**
| Feature | Description |
|---------|-------------|
| Aggregation | 100+ documentation sources |
| Offline | Download for offline use |
| Fuzzy search | Real-time, as-you-type |
| Customization | Enable only what you need |
| Open source | github.com/freeCodeCamp/devdocs |

**Key Insight:** Uniform format is critical - all docs converted to consistent structure for search.

---

### Hoogle (Haskell)

**Architecture:**
- Parses Haddock documentation
- Generates binary `.hoo` index files
- Type unification for signature matching

**Search Features:**
| Query Type | Example | Finds |
|------------|---------|-------|
| Name | `map` | Functions named map |
| Type signature | `(a -> b) -> [a] -> [b]` | Functions matching signature |
| Partial | `[a] -> Int` | length, sum, etc. |

**Type Matching Algorithm:**
1. Parse query as type signature
2. Alpha-rename type variables
3. Unify against indexed signatures
4. Rank by subsumption (more specific = better match)

**Key Insight:** Type-based search is killer for functional languages. For Eiffel, we could search by feature signature.

---

### Sourcegraph

**Architecture:**
- `gitserver`: Sharded repository storage
- `worker`: Keeps repos up-to-date
- Trigram index + in-memory streaming (hybrid)
- LSP for code intelligence

**Search Features (2024):**
| Feature | Description |
|---------|-------------|
| Keyword search | Natural language queries |
| Point-in-time | `rev:at.time()` for historical search |
| Diff/commit search | Search across git history |
| Precise navigation | Go-to-definition, find references |
| Language detection | File contents, not just extension |

**Key Insight:** Hybrid trigram + streaming handles both indexed and real-time search.

---

### ripgrep

**Architecture:**
- Rust workspace with 9 member crates
- `grep` facade coordinates sub-crates
- `ignore` crate handles gitignore + parallel iteration
- `regex-automata` with SIMD acceleration

**Features:**
| Feature | Description |
|---------|-------------|
| Speed | Fastest grep alternative |
| Smart defaults | Respects gitignore, skips binary |
| Unicode | Full UTF-8 support |
| PCRE2 optional | Lookaround, backreferences |
| Parallel | Multi-threaded directory traversal |

**Key Insight:** Speed comes from respecting ignores (fewer files) + SIMD regex + parallelism.

---

### Comparative Analysis for Eiffel KB

| Need | Best Model | Why |
|------|-----------|-----|
| Offline local search | Dash/Zeal | SQLite + HTML docset format |
| Code examples | DevDocs | Uniform format, fuzzy search |
| Type/signature search | Hoogle | Could adapt for Eiffel signatures |
| Fast file search | ripgrep | Speed for codebase exploration |
| Natural queries | pkg.go.dev | Postgres FTS, keyword search |

### Key Takeaways for simple_kb

1. **SQLite FTS is the standard** - Dash, Zeal, DevDocs all use it
2. **Uniform format matters** - Convert all sources to consistent structure
3. **Fuzzy search expected** - Users expect typo tolerance
4. **Scope filters useful** - `language:query` or `lib:query` pattern
5. **Type search is advanced** - Eiffel signature search would be unique
6. **First sentence matters** - Index summary/description prominently
7. **Offline is critical** - Developers want it to work without internet

### Sources

- [pkg.go.dev](https://pkg.go.dev/)
- [pkg.go.dev About](https://pkg.go.dev/about)
- [pkgsite design](https://github.com/golang/pkgsite/blob/master/doc/design.md)
- [DevDocs.io](https://devdocs.io/)
- [DevDocs GitHub](https://github.com/freeCodeCamp/devdocs)
- [Zeal](https://zealdocs.org/)
- [Zeal GitHub](https://github.com/zealdocs/zeal)
- [Sourcegraph Architecture](https://sourcegraph.com/docs/admin/architecture)
- [Sourcegraph Code Search](https://sourcegraph.com/docs/code-search/features)
- [ripgrep GitHub](https://github.com/BurntSushi/ripgrep)
- [Hoogle GitHub](https://github.com/ndmitchell/hoogle)

---

## Step 2: Tech-Stack Research - FTS5, Semantic Search, Code Search

*(In Progress)*


### SQLite FTS5 Full-Text Search

**Overview:**
FTS5 is SQLite's latest full-text search extension (succeeds FTS3/FTS4). Enables efficient text search with built-in relevance ranking.

**Key Features:**

| Feature | Description |
|---------|-------------|
| BM25 ranking | Best Match 25 algorithm for relevance scoring |
| Column weights | Custom weights per column: `bm25(table, 2.0, 1.0)` |
| Tokenizers | Porter (stemming), Unicode61, ASCII |
| Boolean operators | AND, OR, NOT support |
| Phrase search | "exact phrase" matching |
| Prefix search | `word*` for prefix matching |
| Auxiliary functions | `highlight()`, `snippet()` for result formatting |

**BM25 Scoring:**
```sql
-- Lower scores = more relevant (negative by design)
SELECT title, content, bm25(articles, 2.0, 1.0) as score
FROM articles
WHERE articles MATCH 'sqlite database'
ORDER BY score;
```

**Column Weighting Example:**
```sql
-- Title matches worth 10x content matches
SELECT * FROM docs 
WHERE docs MATCH 'query'
ORDER BY bm25(docs, 10.0, 1.0);
```

**Tokenizer Options:**
| Tokenizer | Use Case |
|-----------|----------|
| unicode61 | General multilingual (default) |
| porter | English with stemming (happen → happens, happened) |
| ascii | ASCII-only, faster |

---

### Trigram Indexing (Zoekt/Google Code Search)

**Zoekt** (Sourcegraph/Google):
- Fast trigram-based code search
- Supports substring and regexp matching
- Boolean operators (and, or, not)
- Used by Sourcegraph for enterprise code search

**How Trigrams Work:**
1. Split text into 3-character sequences: "hello" → "hel", "ell", "llo"
2. Build inverted index: trigram → list of documents
3. Query: extract trigrams from search, intersect document lists
4. Post-filter with actual pattern match

**Why 3-grams:**
- 2-grams: too few distinct values (collisions)
- 4-grams: too many distinct values (index size)
- 3-grams: sweet spot for code search

**Performance:**
- Very fast for substring search
- Enables regex search without full scan
- Google Code Search indexed millions of files

---

### Semantic/Vector Search

**Concept:**
- Embed text/code into high-dimensional vectors
- Similar items cluster together in vector space
- Query: embed query, find nearest neighbors

**For Code Search:**
| Approach | Description |
|----------|-------------|
| Function-level chunks | Index functions/methods as units |
| jina-embeddings-v2-base-code | Specialized code embeddings |
| sentence-transformers | General NLP embeddings |

**Vector Databases:**
- Qdrant, Pinecone, Milvus, FAISS
- Approximate nearest neighbor (ANN) search
- Sub-millisecond retrieval at scale

**Trade-offs:**
| Aspect | FTS5 | Vector Search |
|--------|------|---------------|
| Setup | Simple, built into SQLite | Requires embedding model |
| Queries | Exact keywords | Semantic similarity |
| "How do I X?" | Poor (keyword mismatch) | Good (intent matching) |
| Code search | Good for names/identifiers | Good for concepts |
| Offline | Yes | Needs embedding model |

---

### Fuzzy Matching

**Levenshtein Distance:**
- Minimum edits (insert, delete, substitute) to transform one string to another
- "kitten" → "sitting" = 3 edits
- O(n*m) dynamic programming

**Related Algorithms:**
| Algorithm | Feature |
|-----------|---------|
| Levenshtein | Basic edit distance |
| Damerau-Levenshtein | Adds transposition (swap adjacent) |
| Jaro-Winkler | Optimized for short strings, typos |

**FTS5 + Fuzzy:**
- FTS5 doesn't have built-in fuzzy
- Common approach: generate spelling variants, search all
- Or: use trigrams for candidate retrieval, then fuzzy rank

---

### Architecture Patterns for Search

**Strategy Pattern:**
- Swap search algorithms (FTS5, trigram, vector) at runtime
- Common interface: `search(query) → results`

**Pipeline Pattern:**
```
Query → Tokenize → Expand (synonyms) → Search → Rank → Filter → Format
```

**Hybrid Search:**
1. Fast candidate retrieval (trigrams or FTS5)
2. Re-rank with more expensive scoring (BM25, vectors)
3. Post-filter (permissions, recency)

---

### Key Decisions for simple_kb

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Primary search | FTS5 | Built into simple_sql, proven, offline |
| Ranking | BM25 with column weights | Title/name matches > content |
| Fuzzy support | Trigram fallback | For typo tolerance |
| Semantic search | Future phase | Requires embedding infrastructure |
| Query syntax | Simple keywords first | Lower barrier, upgrade later |

### Sources

- [SQLite FTS5](https://sqlite.org/fts5.html)
- [FTS5 BM25](https://www.slingacademy.com/article/ranking-full-text-search-results-in-sqlite-explained/)
- [Zoekt](https://github.com/sourcegraph/zoekt)
- [Google Code Search (Russ Cox)](https://swtch.com/~rsc/regexp/regexp4.html)
- [Qdrant Code Search](https://qdrant.tech/documentation/advanced-tutorials/code-search/)
- [Hugging Face Code Search](https://huggingface.co/learn/cookbook/en/code_search)
- [Meilisearch Fuzzy Search](https://www.meilisearch.com/blog/fuzzy-search)
- [Levenshtein Distance](https://en.wikipedia.org/wiki/Levenshtein_distance)

---

## Step 3: Eiffel Ecosystem Research - simple_* Coverage

*(In Progress)*


### Requirements Mapping to simple_* Libraries

| Requirement | simple_* Library | Status | Key Classes/Features |
|-------------|-----------------|--------|---------------------|
| FTS5 full-text search | simple_sql | ✅ COVERED | `SIMPLE_SQL_FTS5`, `SIMPLE_SQL_FTS5_QUERY` |
| BM25 ranking | simple_sql | ✅ COVERED | `search_ranked()`, `order_by_rank` |
| CLI framework | simple_cli | ✅ COVERED | `SIMPLE_CLI` |
| Code examples DB | simple_rosetta | ✅ COVERED | `SOLUTION_STORE`, `TASK_STORE` |
| Eiffel code parsing | simple_eiffel_parser | ✅ COVERED | `EIFFEL_PARSER`, `EIFFEL_AST` |
| Knowledge base pattern | simple_oracle | ✅ COVERED | `knowledge` FTS5 table, query patterns |
| JSON handling | simple_json | ✅ COVERED | For metadata, config |
| File operations | simple_file | ✅ COVERED | Read source files |
| HTTP (for docs fetch) | simple_http | ✅ COVERED | Fetch online resources |
| Caching | simple_cache | ✅ COVERED | Cache query results |
| Logging | simple_logger | ✅ COVERED | Query logging |

---

### Existing Patterns to Reuse

**simple_oracle FTS5 Pattern:**
```sql
-- FTS5 virtual table for knowledge
CREATE VIRTUAL TABLE IF NOT EXISTS knowledge USING fts5(
    category,
    title,
    content
);

-- Query with MATCH and snippet highlighting
SELECT category, title, 
       snippet(knowledge, 2, '>>>', '<<<', '...', 50) as snippet, 
       rank 
FROM knowledge 
WHERE knowledge MATCH ? 
ORDER BY rank 
LIMIT 10
```

**simple_sql FTS5 Query Builder:**
```eiffel
-- Fluent API for FTS5 queries
results := db.fts5.query_builder ("documents")
    .match ("search terms")
    .in_column ("body")
    .with_rank
    .with_snippets ("body", "<mark>", "</mark>")
    .order_by_rank
    .limit (20)
    .execute
```

**simple_rosetta Data Model:**
```eiffel
-- Solutions already structured with:
-- - task_name, language, code, tier, validated
-- Perfect for "how do I do X" queries
```

---

### Content Sources Available Locally

| Source | Location | Content Type | Volume |
|--------|----------|--------------|--------|
| simple_rosetta solutions | `/d/prod/simple_rosetta/` | Code examples | 274 solutions |
| simple_* libraries (59) | `/d/prod/simple_*/src/` | Eiffel classes | ~1000+ classes |
| ISE EiffelBase | `$ISE_LIBRARY/library/base/` | Core library | ~300 classes |
| Gobo libraries | `$GOBO/library/` | Extended functionality | ~500+ classes |
| ECMA 367 | PDF extraction needed | Language spec | Chapters |
| oracle knowledge base | `/d/prod/simple_oracle/` | Patterns, gotchas | Growing |

---

### simple_eiffel_parser Capabilities

| Feature | Status | Use Case |
|---------|--------|----------|
| Lexer | ✅ | Tokenize Eiffel code |
| Parser | ✅ | Build AST |
| Class nodes | ✅ | Extract class metadata |
| Feature nodes | ✅ | Extract feature signatures |
| Contract extraction | ✅ | Get require/ensure clauses |
| DBC metrics | ✅ | Analyze contract coverage |

**Could extract for each class:**
- Class name, description (note clause)
- Feature names, signatures, return types
- Preconditions, postconditions
- Inheritance hierarchy

---

### Gap Analysis

| Gap | Solution | Priority |
|-----|----------|----------|
| Compiler error database | Build from experience + docs | High |
| ECMA 367 content | PDF extraction or manual | Medium |
| OOSC-2 excerpts | Manual curation | Low |
| Gobo parsing | Use simple_eiffel_parser | Medium |
| Signature search (Hoogle-style) | Build on parser feature extraction | Future |

---

### Architecture from Existing Patterns

```
┌─────────────────────────────────────────────────────────────────┐
│                      SIMPLE_KB (Facade)                          │
│  (KB_CLI, KB_QUICK, KB_DATABASE)                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐                     │
│  │   Content Tables  │  │   FTS5 Index     │                    │
│  │   (classes,       │  │   (kb_search)    │                    │
│  │    features,      │  │   BM25 ranking   │                    │
│  │    examples,      │  │   Snippets       │                    │
│  │    errors,        │  └──────────────────┘                    │
│  │    patterns)      │                                           │
│  └──────────────────┘                                            │
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐                     │
│  │   Ingesters       │  │   Query Engine   │                    │
│  │   - Source files  │  │   - NL parsing   │                    │
│  │   - Rosetta       │  │   - FTS5 MATCH   │                    │
│  │   - Oracle KB     │  │   - Result format│                    │
│  └──────────────────┘  └──────────────────┘                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

### Conclusion: All Requirements Covered

**No new simple_* libraries needed.** Existing ecosystem provides:

1. **simple_sql** - FTS5, BM25 ranking, query builder
2. **simple_oracle** - Proven knowledge base pattern
3. **simple_rosetta** - 274 code examples ready
4. **simple_eiffel_parser** - Code parsing for indexing
5. **simple_cli** - CLI framework
6. **simple_file** - Source file reading

Build simple_kb as a **new library** that combines these existing capabilities.

---

## Step 4: Developer Pain Points - Actual Eiffel Programmer Questions

*(In Progress)*


### Categories of Developer Questions

Based on research, Eiffel developers ask questions in these categories:

| Category | Example Questions | Priority |
|----------|-------------------|----------|
| **Reuse/Discovery** | "What library does X?", "Is there a regex library?" | Critical |
| **How-to/Patterns** | "How do I iterate a list?", "How do I read a file?" | Critical |
| **Syntax Translation** | "What's the Eiffel equivalent of X?", "How do I return a value?" | High |
| **Compiler Errors** | "What does VEVI mean?", "How do I fix VD89?" | High |
| **Void Safety** | "How do I handle detachable?", "Why is my attached failing?" | High |
| **Contracts/DBC** | "When to use require vs check?", "How to write postconditions?" | Medium |
| **SCOOP** | "How do I make this concurrent?", "What is separate?" | Medium |
| **Migration** | "How do I convert from Python?", "What's the Eiffel way?" | Medium |

---

### Common Pain Points from Research

**From Hacker News & Forums:**

| Pain Point | Description | KB Solution |
|------------|-------------|-------------|
| "Simple things take too much effort" | File I/O, regex, HTTP require digging | Show simple_* one-liners |
| "No working ecosystem of libraries" | Hard to find what exists | Index all simple_* + ISE + Gobo |
| Syntax differences | No curly braces, `Result` vs `return` | Translation examples |
| Loops are inverted | `until` vs `while` | Side-by-side comparisons |
| Multiple inheritance clashes | Rename/undefine confusion | Pattern examples |
| Finding examples is hard | Scattered documentation | Centralized search |

---

### Compiler Error Codes (High Value Target)

**ECMA/ISO Validity Errors (VXXX):**

| Code | Meaning | Common Cause |
|------|---------|--------------|
| VEVI | Variable not properly set | Attached variable not initialized |
| VUAR | Argument mismatch | Wrong number/types of arguments |
| VTCT | Type constraint violation | Attachment mark on expanded type |
| VD89 | Dependency cycle | Duplicate ECF UUIDs (simple_kb insight!) |
| VOMB | Invalid inspect/when | Type mismatch in inspect clause |

**These are perfect for the KB because:**
1. Error code appears in compiler output
2. User copies error code
3. KB returns: meaning + common causes + fixes + examples

---

### Specific Question Types to Support

**Type 1: Reuse Questions**
```
User: "What library handles JSON?"
KB: simple_json - JSON_PARSER, JSON_OBJECT, JSONPath
    Example: parser.parse (text).object_at ("key")
```

**Type 2: How-To Questions**
```
User: "How do I read a file line by line?"
KB: simple_file - FILE_QUICK.read_lines ("path")
    Example code with contracts
    Rosetta Code: "Read a file line by line" solution
```

**Type 3: Error Questions**
```
User: "VEVI error"
KB: Variable not properly set
    Common cause: Attached Result not assigned in all paths
    Fix: Make Result detachable OR ensure all paths assign
    Example of fix
```

**Type 4: Translation Questions**
```
User: "Python: for i in range(10)"
KB: Eiffel equivalent:
    across 1 |..| 10 as ic loop
        -- ic.item is the value
    end
```

**Type 5: Pattern Questions**
```
User: "singleton pattern eiffel"
KB: Use once function:
    instance: MY_CLASS
        once
            create Result.make
        end
```

---

### Categorized Question Inventory

**Must Answer (Day 1):**
- What does error X mean?
- How do I do X in Eiffel?
- What library does X?
- Show me example of X

**Should Answer (Week 1):**
- What's the difference between X and Y?
- Is X thread-safe / SCOOP-compatible?
- What preconditions does X require?
- How do I convert from Python/Java/C#?

**Nice to Answer (Future):**
- What's the Eiffel best practice for X?
- Why was X designed this way?
- What are common mistakes with X?

---

### Sources

- [Eiffel FAQs](https://www.eiffel.com/resources/faqs/eiffel-language/)
- [Learning Eiffel](https://www.eiffel.org/doc/eiffel/Learning_Eiffel)
- [Eiffel Syntax Guide](https://eiffel-guide.com/)
- [Handling Syntax and Validity Errors](https://www.eiffel.org/doc/eiffelstudio/Handling_Syntax_and_Validity_Errors)
- [Error Wizard](https://www.eiffel.org/doc/eiffelstudio/Error_wizard)
- [Converting to Void-Safety](https://www.eiffel.org/doc/eiffel/Converting_existing_software_to_void-safety)
- [Hacker News Eiffel Discussion](https://news.ycombinator.com/item?id=22281615)

---

## Step 5: Innovation Hat - Unique Value Propositions

*(In Progress)*


### Why Not "Just Ask Claude"?

| Aspect | Generic LLM | simple_kb |
|--------|-------------|-----------|
| **Accuracy** | May hallucinate libraries that don't exist | Indexed from actual codebase |
| **Currency** | Training cutoff (Jan 2025) | Live from local files |
| **simple_* specific** | Doesn't know simple_json, simple_http | Full inventory |
| **Offline** | Requires internet | Works offline |
| **Speed** | API latency | Local FTS5 milliseconds |
| **Cost** | API tokens | Free after build |
| **Error codes** | Generic explanation | Specific fixes from experience |

---

### Innovation 1: Authoritative Local Source

**The Problem:** LLMs sometimes hallucinate Eiffel APIs that don't exist.

**The Solution:** simple_kb indexes actual source code.
- If it's in the DB, it exists
- Feature signatures extracted from real code
- Contracts come from actual implementations

```
User: "What methods does SIMPLE_HTTP have?"
KB: [Extracted from actual class file, not hallucinated]
    - get(url): Response
    - post(url, body): Response  
    - get_json(url): JSON_OBJECT
    ...
```

---

### Innovation 2: Error Code Decoder

**The Problem:** EiffelStudio shows "VEVI" but user doesn't know what to do.

**The Solution:** Error code → explanation + fix + example

```
User: "VEVI"
KB: VARIABLE NOT PROPERLY SET
    
    Cause: Attached variable not initialized on all paths
    
    Bad:
        my_feature: STRING
            do
                if condition then
                    Result := "yes"
                end
                -- Missing else branch!
            end
    
    Fix Option 1: Add else branch
        else
            Result := ""
        end
    
    Fix Option 2: Make detachable
        my_feature: detachable STRING
```

---

### Innovation 3: Cross-Reference simple_* Ecosystem

**The Problem:** 59 libraries, hard to know what's where.

**The Solution:** Unified search across all simple_* libraries

```
User: "json"
KB: 
    simple_json - Full JSON parsing library
        JSON_PARSER, JSON_OBJECT, JSON_ARRAY
        
    simple_sql - JSON1 extension support
        simple_sql_json.e - JSON path queries in SQLite
        
    simple_http - JSON response helpers
        get_json() returns JSON_OBJECT
```

---

### Innovation 4: Rosetta Code Integration

**The Problem:** "How do I do X?" needs working examples.

**The Solution:** 274 Rosetta Code solutions indexed

```
User: "bubble sort"
KB:
    Task: Bubble Sort
    Tier: TIER1 (fundamental)
    
    class BUBBLE_SORT
    feature
        sort (a: ARRAY[COMPARABLE]): ARRAY[COMPARABLE]
            require
                a_not_void: a /= Void
            local
                ...
```

---

### Innovation 5: Pattern Library from Experience

**The Problem:** Common patterns scattered in memory/docs.

**The Solution:** Curated pattern database from oracle knowledge

```
User: "singleton pattern"
KB: EIFFEL SINGLETON PATTERN
    
    Use once function (not class):
    
    shared_instance: MY_SINGLETON
        once
            create Result.make
        end
    
    Why: Once functions guarantee single execution
    per thread (or system with once ("PROCESS"))
```

---

### Innovation 6: Signature Search (Future/Advanced)

**Inspired by Hoogle:** Search by feature signature

```
User: "(ARRAY[G], G): INTEGER"
KB: Finds features matching that signature:
    - ARRAY.index_of (v: G): INTEGER
    - ARRAY.occurrences (v: G): INTEGER
    - LIST.index_of (v: G): INTEGER
```

---

### Innovation 7: Translation Bridge

**The Problem:** Developers come from Java/Python/C#

**The Solution:** Cross-language pattern mappings

```
User: "python list comprehension"
KB: EIFFEL EQUIVALENT
    
    Python: [x*2 for x in range(10) if x > 3]
    
    Eiffel:
    create result.make (0)
    across 1 |..| 10 as ic loop
        if ic.item > 3 then
            result.extend (ic.item * 2)
        end
    end
    
    Or with ITERABLE_OPERATIONS (Gobo):
    (1 |..| 10).filtered (agent (x: INTEGER): BOOLEAN do Result := x > 3 end)
               .mapped (agent (x: INTEGER): INTEGER do Result := x * 2 end)
```

---

### Innovation 8: Contract Pattern Library

**Unique to Eiffel:** DBC pattern examples

```
User: "postcondition for add to list"
KB: STANDARD POSTCONDITIONS FOR COLLECTION ADD
    
    ensure
        count_increased: count = old count + 1
        item_added: has (item)
        item_is_last: last = item
        others_unchanged: -- frame condition
```

---

### Competitive Advantage Summary

| Feature | ChatGPT | Copilot | simple_kb |
|---------|---------|---------|-----------|
| Eiffel-specific | Generic | Generic | **Specialist** |
| Offline | No | No | **Yes** |
| simple_* aware | No | No | **Full index** |
| Error decoder | Generic | No | **Specific fixes** |
| Rosetta examples | Limited | No | **274 solutions** |
| Local code aware | No | Partial | **Full AST** |
| Contract patterns | No | No | **DBC library** |
| Free after setup | No | No | **Yes** |

---

## Step 6: Design Strategy Synthesis - Key Decisions

*(In Progress)*


### Decision 1: Name and Location

**Name:** `simple_kb` (Knowledge Base)
**Location:** `/d/prod/simple_kb/`
**GitHub:** `simple-eiffel/simple_kb`

**Rationale:** Follows simple_* naming convention. "KB" is clear and short.

---

### Decision 2: Database Schema

```sql
-- Core content tables
CREATE TABLE classes (
    id INTEGER PRIMARY KEY,
    library TEXT NOT NULL,        -- 'simple_json', 'base', 'gobo'
    name TEXT NOT NULL,           -- 'JSON_PARSER'
    description TEXT,             -- From note clause
    file_path TEXT,               -- Source file location
    UNIQUE(library, name)
);

CREATE TABLE features (
    id INTEGER PRIMARY KEY,
    class_id INTEGER REFERENCES classes(id),
    name TEXT NOT NULL,           -- 'parse'
    signature TEXT,               -- '(text: STRING): JSON_OBJECT'
    description TEXT,             -- From feature comment
    kind TEXT,                    -- 'query', 'command', 'creation'
    preconditions TEXT,           -- JSON array of contracts
    postconditions TEXT,          -- JSON array of contracts
    UNIQUE(class_id, name)
);

CREATE TABLE examples (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,          -- 'Bubble Sort'
    source TEXT,                  -- 'rosetta', 'manual', 'generated'
    code TEXT NOT NULL,
    tags TEXT,                    -- JSON array: ['sorting', 'array']
    tier TEXT                     -- 'TIER1', 'TIER2', etc.
);

CREATE TABLE errors (
    id INTEGER PRIMARY KEY,
    code TEXT UNIQUE NOT NULL,    -- 'VEVI', 'VD89'
    meaning TEXT NOT NULL,        -- Short description
    explanation TEXT,             -- Full explanation
    common_causes TEXT,           -- JSON array
    fixes TEXT,                   -- JSON array of fix strategies
    examples TEXT                 -- JSON: bad code → good code
);

CREATE TABLE patterns (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,           -- 'singleton', 'factory', 'observer'
    description TEXT,
    code TEXT NOT NULL,           -- Example implementation
    when_to_use TEXT,
    eiffel_idioms TEXT            -- JSON array of Eiffel-specific notes
);

CREATE TABLE translations (
    id INTEGER PRIMARY KEY,
    source_lang TEXT NOT NULL,    -- 'python', 'java', 'csharp'
    source_pattern TEXT NOT NULL, -- 'list comprehension'
    eiffel_pattern TEXT NOT NULL, -- Eiffel equivalent code
    notes TEXT                    -- Explanation
);

-- FTS5 virtual table for full-text search
CREATE VIRTUAL TABLE kb_search USING fts5(
    content_type,    -- 'class', 'feature', 'example', 'error', 'pattern'
    content_id,      -- Reference to source table
    title,           -- Searchable title/name
    body,            -- Searchable content
    tags,            -- Searchable tags
    tokenize='porter unicode61'
);
```

---

### Decision 3: Content Priority Order

| Phase | Content | Source | Volume |
|-------|---------|--------|--------|
| 1 | Compiler errors | Manual curation + docs | ~50 codes |
| 1 | simple_* class inventory | Parser extraction | ~500 classes |
| 1 | Rosetta solutions | simple_rosetta DB | 274 examples |
| 2 | Feature signatures | Parser extraction | ~3000 features |
| 2 | ISE EiffelBase | Parser extraction | ~300 classes |
| 2 | Patterns | Manual curation | ~30 patterns |
| 3 | Gobo libraries | Parser extraction | ~500 classes |
| 3 | Translations | Manual curation | ~50 mappings |
| 3 | ECMA 367 excerpts | Manual/PDF | Selected sections |

---

### Decision 4: CLI Interface

```
Usage: eiffel-kb <command> [options]

Commands:
  search <query>          Full-text search across all content
  class <name>            Show class details + features
  feature <class.feature> Show feature signature + contracts
  error <code>            Explain compiler error code
  example <topic>         Find code examples
  pattern <name>          Show design pattern
  translate <lang:code>   Show Eiffel equivalent

Options:
  --limit N       Max results (default: 10)
  --json          Output as JSON
  --verbose       Include full code snippets

Examples:
  eiffel-kb search "json parsing"
  eiffel-kb class JSON_PARSER
  eiffel-kb feature SIMPLE_HTTP.get_json
  eiffel-kb error VEVI
  eiffel-kb example "bubble sort"
  eiffel-kb pattern singleton
  eiffel-kb translate "python:for x in range(10)"
```

---

### Decision 5: Query Processing Pipeline

```
User Query
    │
    ▼
┌─────────────────┐
│ Query Parser    │  Detect: error code? class name? natural language?
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ Query Expander  │  Add synonyms, handle typos
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ FTS5 Search     │  MATCH with BM25 ranking
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ Result Enricher │  Fetch full content from source tables
└─────────────────┘
    │
    ▼
┌─────────────────┐
│ Formatter       │  CLI text, JSON, or HTML
└─────────────────┘
```

---

### Decision 6: Key Classes

| Class | Responsibility | Dependencies |
|-------|----------------|--------------|
| `KB_DATABASE` | SQLite + FTS5 management | simple_sql |
| `KB_INGESTER` | Load content from sources | simple_eiffel_parser, simple_file |
| `KB_SEARCH` | Query processing + FTS5 | simple_sql |
| `KB_CLI` | Command-line interface | simple_cli |
| `KB_RESULT` | Search result model | - |
| `KB_FORMATTER` | Output formatting | simple_json |
| `KB_QUICK` | One-liner facade | All above |
| `SIMPLE_KB` | Main facade (type refs) | All above |

---

### Decision 7: Ingestion Strategy

**Source Code Ingestion:**
```eiffel
ingester: KB_INGESTER
ingester.ingest_directory ("/d/prod/simple_json/src")
-- Parses all .e files
-- Extracts class metadata, features, contracts
-- Inserts into classes/features tables
-- Rebuilds FTS5 index
```

**Rosetta Integration:**
```eiffel
ingester.import_rosetta ("/d/prod/simple_rosetta/rosetta.db")
-- Copies solutions to examples table
-- Adds to FTS5 index
```

**Error Code Manual Entry:**
```eiffel
ingester.add_error ("VEVI", "Variable not properly set", ...)
-- Or bulk load from JSON file
ingester.load_errors_from_json ("/d/prod/simple_kb/data/errors.json")
```

---

### Decision 8: Phased Implementation

| Phase | Deliverable | Effort |
|-------|-------------|--------|
| **1A** | Schema + KB_DATABASE | 2 days |
| **1B** | Error code database (manual curation) | 2 days |
| **1C** | CLI with `error` command | 1 day |
| **2A** | Source ingester (simple_eiffel_parser) | 3 days |
| **2B** | `search`, `class`, `feature` commands | 2 days |
| **2C** | Rosetta import | 1 day |
| **3A** | Pattern library curation | 2 days |
| **3B** | Translation mappings | 2 days |
| **3C** | Documentation + tests | 2 days |

**Total estimated effort:** ~17 days

---

### Decision 9: Success Metrics

| Metric | Target |
|--------|--------|
| Classes indexed | 500+ (all simple_*) |
| Features indexed | 3000+ |
| Error codes documented | 50+ |
| Examples available | 274 (Rosetta) |
| Patterns documented | 30+ |
| Query response time | <100ms |
| Offline capability | 100% |

---

### Decision 10: Future Enhancements

| Enhancement | Priority | Dependencies |
|-------------|----------|--------------|
| Signature search (Hoogle-style) | Medium | Parser feature extraction |
| Semantic/vector search | Low | Embedding model |
| Web UI | Medium | simple_web |
| IDE integration (LSP) | Medium | simple_lsp |
| Auto-update on source change | Low | simple_watcher |

---

## Step 7: Implementation Report

*(Writing design document...)*


### Design Document Produced

**Location:** `/d/prod/reference_docs/designs/SIMPLE_KB_DESIGN.md`

**Contents:**
1. Executive Summary
2. Problem Statement
3. Architecture Diagram
4. Database Schema
5. CLI Interface Specification
6. Key Classes
7. Sample Interactions
8. Implementation Plan (17 days, 3 phases)
9. Dependencies
10. Success Metrics
11. Future Enhancements
12. Risk Mitigation
13. Appendix: Error Code Template

---

## Research Complete

**7-Step Research Process for simple_kb: COMPLETE**

| Step | Status | Key Finding |
|------|--------|-------------|
| 1. Developer Tools | ✅ | 8 tools analyzed (Rust docs, pkg.go.dev, Dash, etc.) |
| 2. Tech Stack | ✅ | FTS5 + BM25 primary, trigram fallback |
| 3. Eiffel Ecosystem | ✅ | All requirements covered by existing simple_* |
| 4. Pain Points | ✅ | 8 question categories, error codes high value |
| 5. Innovations | ✅ | 8 unique differentiators vs generic LLM |
| 6. Design Decisions | ✅ | 10 key decisions, 17-day implementation |
| 7. Design Document | ✅ | Complete at SIMPLE_KB_DESIGN.md |

**Ready to begin implementation.**

