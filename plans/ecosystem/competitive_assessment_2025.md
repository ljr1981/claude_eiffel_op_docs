# Eiffel + AI vs 2025 Tech Stacks: Honest Assessment

**Date:** December 9, 2025
**Purpose:** Unbiased, brutally honest comparison - no cheerleading

---

## CORRECTIONS (December 9, 2025)

**This document contained errors that have been corrected:**

1. **EWF DOES support SCOOP** - The original claim that EWF "blocks SCOOP" was **FALSE**.
   - Confirmed by Jocelyn Fiat (Eiffel Software maintainer): "EWF supports all the known Eiffel concurrency modes: none, thread and SCOOP"
   - Confirmed by Eric Bezault: EWF's CHANGELOG states "Made library ecf compilable in scoop concurrency mode by default"
   - simple_web.ecf already uses `concurrency support="scoop"`

2. **Gobo Eiffel VS Code Extension exists and works** - Eric Bezault's extension already provides:
   - Go to definition (DONE)
   - Hover documentation (DONE)  
   - Diagnostic errors as-you-type (DONE)
   - Find references (soon)
   - Code completion (soon)
   - With deep compiler integration (not just grep)

3. **"Phase 1: EWF Replacement" is NOT needed** - EWF is actively maintained and SCOOP-compatible.

**Lesson learned:** Verify claims with primary sources (ECF files, source code, maintainers) before publishing.

---

## The Question

Can Eiffel + simple_* + AI assistance compete with mainstream 2025 tech stacks for building production software?

**Short answer:** No, not yet. Here's why, and what would change that.

---

## Head-to-Head Comparison

### Web Development

| Capability | TypeScript/Node | Go | Rust | Python | Eiffel/simple_* |
|------------|-----------------|-----|------|--------|-----------------|
| HTTP server | Express, Fastify, Hono | net/http, Gin, Fiber | Actix, Axum | FastAPI, Django | simple_web (EWF wrapper) |
| Maturity | 10+ years | 10+ years | 5+ years | 20+ years | Wrapper only |
| Performance | Good | Excellent | Excellent | Moderate | Unknown (no benchmarks) |
| Async/concurrency | Native promises | Goroutines | async/await, tokio | asyncio | SCOOP (EWF supports it) |
| Middleware ecosystem | 1000s | 100s | 100s | 100s | ~5 |
| Production deployments | Millions | Millions | Thousands | Millions | Handful |

**Verdict:** Eiffel loses badly. simple_web is a thin wrapper around ISE's EWF, which is:
- SCOOP-compatible (confirmed by Jocelyn Fiat, Dec 2025)
- Poorly documented
- Minimally maintained
- No performance data

**What would change this:** Rewrite simple_http/simple_web from scratch with native sockets. This is Phase 7 in the roadmap but represents significant work.

---

### Database Access

| Capability | TypeScript | Go | Rust | Python | Eiffel/simple_* |
|------------|------------|-----|------|--------|-----------------|
| ORM | Prisma, TypeORM, Drizzle | GORM, sqlx | Diesel, SQLx | SQLAlchemy, Django ORM | None |
| Raw SQL | Multiple | database/sql | sqlx | psycopg2, etc | simple_sql |
| Connection pooling | Built-in | Built-in | Built-in | Built-in | Manual |
| Migrations | Yes | Yes | Yes | Yes | No |
| Query builder | Yes | Yes | Yes | Yes | Basic |

**Verdict:** simple_sql is functional for basic CRUD but lacks:
- ORM/object mapping
- Migrations
- Connection pooling
- Prepared statement caching

**Honest assessment:** Adequate for small projects. Not competitive for anything requiring serious database work.

---

### JSON Handling

| Capability | TypeScript | Go | Rust | Python | Eiffel/simple_* |
|------------|------------|-----|------|--------|-----------------|
| Parse/serialize | Native | encoding/json | serde_json | json | simple_json |
| Streaming | Yes | Yes | Yes | Yes | No |
| Schema validation | Zod, Yup | go-playground | serde | Pydantic | Contracts only |
| Performance | Fast | Fast | Very fast | Moderate | Unknown |

**Verdict:** simple_json works. It's one of the more complete simple_* libraries. But:
- No streaming for large documents
- No benchmarks vs other implementations
- Contract validation is different from schema validation

**Honest assessment:** Functional, probably competitive for typical use cases.

---

### Concurrency

| Capability | TypeScript | Go | Rust | Python | Eiffel/simple_* |
|------------|------------|-----|------|--------|-----------------|
| Model | Event loop | Goroutines + channels | async/await + ownership | GIL + asyncio | SCOOP |
| Learning curve | Low | Medium | High | Low | High |
| Safety guarantees | None | Race detector | Compile-time | None | Compile-time |
| Practical use | Proven | Proven | Proven | Proven | Limited |

**Verdict:** SCOOP is theoretically superior - data races impossible by design. But:
- EWF actually supports SCOOP (corrected Dec 2025)
- Limited real-world SCOOP codebases exist
- Documentation is academic, not practical
- Debugging tools are immature

**Honest assessment:** SCOOP could be a differentiator, but isn't yet because the ecosystem doesn't support it properly.

---

### Tooling

| Capability | TypeScript | Go | Rust | Python | Eiffel |
|------------|------------|-----|------|--------|--------|
| Package manager | npm/yarn/pnpm | go mod | cargo | pip/poetry | simple_setup (custom) |
| IDE support | VSCode, WebStorm | GoLand, VSCode | rust-analyzer | PyCharm, VSCode | EiffelStudio, VSCode (Gobo) |
| LSP | Excellent | Excellent | Excellent | Excellent | Gobo extension (partial) |
| Debugger | Chrome DevTools, VSCode | Delve | lldb/gdb | pdb, debugpy | EiffelStudio only |
| Formatter | Prettier | gofmt | rustfmt | Black | None standard |
| Linter | ESLint | golangci-lint | Clippy | Ruff, Flake8 | Compiler only |

**Verdict:** Eiffel tooling is a generation behind. No LSP means no VSCode/Cursor/modern editor support. EiffelStudio is functional but dated.

**This is a major adoption blocker.** Developers won't use a language they can't use in their preferred editor.

---

### Hiring & Community

| Metric | TypeScript | Go | Rust | Python | Eiffel |
|--------|------------|-----|------|--------|--------|
| Stack Overflow questions | 2M+ | 100K+ | 50K+ | 20M+ | ~2K |
| GitHub repos | 10M+ | 1M+ | 500K+ | 50M+ | ~5K |
| Job postings (2024) | 500K+ | 100K+ | 30K+ | 1M+ | ~50 |
| Reddit subscribers | 500K+ | 200K+ | 300K+ | 1M+ | ~1K |
| Books published (2020-2024) | 100+ | 50+ | 50+ | 200+ | 2-3 |

**Verdict:** Eiffel has effectively no community. This means:
- No Stack Overflow answers
- No blog posts solving common problems
- No conference talks
- No podcasts
- No YouTube tutorials
- No jobs (realistically)

**The AI-assistance argument:** "AI replaces community support" is partially true for coding but doesn't help with:
- Finding Eiffel developers to hire
- Getting management buy-in
- Finding hosting/deployment expertise
- Debugging production issues at 3am

---

### The AI Productivity Claim

**Claim:** AI + Eiffel + DBC = 98x productivity vs industry baseline.

**Reality check:**
1. The "industry baseline" of 10-50 LOC/day includes meetings, planning, code review, testing, deployment, maintenance. Raw LOC/day for focused coding is much higher in any language.
2. AI assistance provides similar productivity boosts in TypeScript, Python, Go, Rust. This isn't Eiffel-specific.
3. LOC is a terrible metric. 1000 lines of buggy code isn't better than 100 lines of correct code.
4. The DBC contracts are valuable but add lines that wouldn't exist in other languages - inflating the count.

**What's actually true:** AI + DBC catches bugs earlier. Contracts serve as executable documentation. This has real value. But it's not 98x.

---

## What Eiffel Actually Has Going For It

### 1. Null Safety (Real)
Void safety eliminates null pointer exceptions at compile time. This is genuine, valuable, and Eiffel did it before TypeScript, Kotlin, Swift, or Rust.

### 2. Design by Contract (Real)
Preconditions, postconditions, and invariants are first-class language features. No other mainstream language has this. It catches bugs and serves as documentation.

### 3. Multiple Inheritance Done Right (Real)
Eiffel's inheritance model (with renaming, redefining, undefining) is more powerful than single inheritance + interfaces. Whether this matters depends on your problem domain.

### 4. Readability
Eiffel code is verbose but readable. `feature`, `require`, `ensure`, `do`, `end` are clearer than braces and symbols.

### 5. SCOOP (Potential)
If SCOOP worked properly with the ecosystem, compile-time concurrency safety would be a genuine differentiator. It doesn't currently.

---

## What Would Make Eiffel Competitive

### Must Have (Blocking)
1. **LSP implementation** - Modern editor support is non-negotiable
2. **EWF replacement** - Native HTTP without ISE dependencies
3. **Real benchmarks** - Prove performance claims with data

### Should Have (Significant)
4. **Cross-platform installer** - Linux/Mac, not just Windows
5. **CI/CD templates** - GitHub Actions that work
6. **Connection pooling** - For any serious database work
7. **Async I/O** - Can't compete on web without it

### Nice to Have
8. **Package registry** - Like crates.io or npm
9. **Tutorial content** - YouTube, blog posts, books
10. **Success stories** - Production deployments with metrics

---

## Honest Conclusion

### Who Should Use Eiffel + simple_* Today?

1. **Eiffel enthusiasts** who value DBC and void safety above ecosystem
2. **Research projects** exploring formal methods
3. **Internal tools** where hiring isn't a concern
4. **Hobbyists** who enjoy the language

### Who Should Not?

1. **Startups** - Hiring will be impossible
2. **Web applications** - Ecosystem isn't ready
3. **Anyone needing community support** - It doesn't exist
4. **Teams requiring modern tooling** - No LSP, no VSCode

### The Path Forward

The simple_* ecosystem is impressive for what it is: one developer + AI building 53 libraries in a month. But "impressive given constraints" isn't the same as "competitive."

To actually compete with 2025 tech stacks, Eiffel needs:
1. Modern tooling (LSP)
2. Native web stack (not EWF)
3. Proven production deployments
4. A reason to choose it over TypeScript/Go/Rust that outweighs the ecosystem disadvantages

DBC + void safety + SCOOP could be that reason, but only if they work together in practice, not just theory.

---

## Appendix: What simple_* Actually Provides

### Complete and Functional
- JSON parsing/building (simple_json)
- SQL database access (simple_sql)
- UUID generation (simple_uuid)
- Base64 encoding (simple_base64)
- Hashing (simple_hash)
- CSV handling (simple_csv)
- Date/time operations (simple_datetime)
- Logging (simple_logger)
- Template rendering (simple_template)
- Environment variables (simple_env)
- File operations (simple_file)
- Process management (simple_process)
- Regex (simple_regex)
- XML parsing (simple_xml)
- Markdown rendering (simple_markdown)

### Wrappers (Depend on ISE Libraries)
- HTTP client/server (simple_http, simple_web) - EWF dependency
- Encryption (simple_encryption) - eel dependency
- Compression (simple_compression) - zlib wrapper
- i18n (simple_i18n) - ISE i18n wrapper

### Windows-Specific
- Clipboard (simple_clipboard)
- Registry (simple_registry)
- Console colors (simple_console)
- Win32 API (simple_win32_api)

### Experimental/Incomplete
- MongoDB (simple_mongo) - basic only
- GUI designer (simple_gui_designer) - prototype
- AI client (simple_ai_client) - Claude API only

---

## What It Would Take to Be Competitive

### Phase 1: Remove the EWF Dependency (Critical)

**Problem:** simple_http and simple_web wrap ISE's EWF, which is:
- Actually SCOOP-compatible (corrected Dec 2025 - confirmed by maintainers)
- Poorly maintained
- A black box we don't control

**Solution:** Build native HTTP from scratch using inline C for Win32/POSIX sockets.

**Deliverables:**
1. `SIMPLE_SOCKET` - TCP socket wrapper (inline C, no .obj files)
2. `SIMPLE_HTTP_PARSER` - HTTP/1.1 request/response parsing
3. `SIMPLE_HTTP_SERVER` - Listen, accept, route, respond
4. `SIMPLE_HTTP_CLIENT` - GET/POST/PUT/DELETE with headers
5. SCOOP-compatible design throughout

**Effort:** Significant. This is building a web framework from scratch. But it's the single biggest blocker to Eiffel being taken seriously for web development.

**Benchmark:** Must demonstrate comparable performance to Go's net/http or Node's http module. No benchmarks = no credibility.

---

### Phase 2: LSP Implementation (Critical for Adoption)

**Problem:** No developer will adopt a language they can't use in VSCode/Cursor/their preferred editor. EiffelStudio-only is a non-starter for most teams.

**Solution:** Implement Language Server Protocol for Eiffel.

**Note:** Eric Bezault is currently building LSP for VSCode. But even without that, this is doable.

**Minimum viable LSP:**
1. Go to definition - parse .e files, index class/feature locations
2. Find references - grep for usage patterns
3. Hover documentation - extract contracts from parsed source
4. Diagnostic errors - run `ec.exe -batch`, parse output
5. Code completion - index feature names by class

**Reality check:** This is NOT a massive undertaking. LSP is just:
- A parser (Eiffel syntax is regular and well-defined)
- A symbol index (class names, feature names, locations)
- JSON-RPC over stdio (well-documented protocol)

We don't need deep compiler integration. We need to parse source files, index symbols, and answer queries. The same AI+human team that built 53 libraries in a month can build a functional LSP in weeks.

**Approach:**
1. `simple_eiffel_parser` - Parse .e files into AST
2. `simple_lsp` - Symbol indexing + JSON-RPC server
3. VSCode extension - Thin client that spawns the LSP

Not perfect, but functional. Good enough for 80% of use cases. Iterate from there.

---

### Phase 3: Prove It Works (Credibility)

**Problem:** No production deployments with public metrics. "Trust us, it's fast" isn't evidence.

**Solution:** Build and deploy real applications with published metrics.

**Needed:**
1. **Benchmarks** - simple_json vs serde_json vs encoding/json. simple_web vs Gin vs Actix. Published, reproducible, honest.
2. **Production case study** - A real application (not a demo) running in production with uptime, latency, and throughput data.
3. **Load testing results** - How many requests/second? How does it scale? What breaks first?

**Without this:** Every claim is just marketing.

---

### Phase 4: Fill Remaining Gaps

**Database:**
- Connection pooling for simple_sql
- Migration support
- Query builder improvements

**Async I/O:**
- Non-blocking file operations
- Async HTTP client
- SCOOP-native async patterns

**Observability:**
- Metrics export (Prometheus format)
- Distributed tracing
- Structured logging improvements

---

### Phase 5: Community Building (Long-term)

**This can't be manufactured, but it can be seeded:**

1. **Tutorial content** - YouTube videos, blog posts, "Getting Started" guides
2. **Example applications** - Not toy demos, real patterns people can copy
3. **Stack Overflow presence** - Answer Eiffel questions (there are few, but they exist)
4. **Conference talks** - Present the AI+DBC workflow at programming conferences
5. **Open contribution** - Make it easy for others to contribute to simple_*

---

### Realistic Assessment

| Phase | Feasibility | Impact |
|-------|-------------|--------|
| 1. EWF Replacement | NOT NEEDED | EWF already supports SCOOP |
| 2. LSP | Doable (weeks) | Critical for adoption |
| 3. Prove It Works | Doable | High - credibility |
| 4. Fill Gaps | Doable | Medium - completeness |
| 5. Community | Slow, organic | Long-term sustainability |

**Honest timeline:**
- Phase 1: Months of focused work (but doable)
- Phase 2: Weeks (Eric is already on it, or we build our own)
- Phases 3-5: Ongoing, not "done"

**The uncomfortable truth:** Even with all phases complete, Eiffel would still be competing against languages with 10-20 years of ecosystem maturity, millions of developers, and massive corporate backing. The question isn't "can Eiffel be competitive?" but "is the DBC+void safety+SCOOP value proposition strong enough to overcome these disadvantages?"

For most teams, the answer will be no. For teams where correctness is paramount and they're willing to invest in training and tooling, it might be yes.

---

**Bottom line:** The simple_* ecosystem is a proof of concept that AI can accelerate Eiffel development. It's not yet a production-ready alternative to mainstream stacks. The gap is tooling, ecosystem, and community - not the language itself.
