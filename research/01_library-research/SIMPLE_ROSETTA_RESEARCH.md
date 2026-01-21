# simple_rosetta Research Notes

**Date:** 2025-12-21
**Status:** Step 1 Complete

---

## Step 1: Deep Web Research - Rosetta Code Specifications

### Rosetta Code Overview

| Metric | Value |
|--------|-------|
| Total Tasks | 1,338 programming tasks |
| Draft Tasks | 398 additional draft tasks |
| Languages Tracked | 982 programming languages |
| Eiffel Solutions | 159 tasks already have Eiffel solutions |
| Last Eiffel Update | December 20, 2025 |

### Task Categories (62 Subcategories)

**Core Programming Concepts**
- Basic language learning
- Functions and subroutines
- Scope, Initialization
- Memory management

**Data & Algorithms**
- Data Structures
- Sorting (50+ tasks)
- Recursion, Iteration
- Arrays

**Mathematics & Computation**
- Arithmetic
- Mathematics
- Geometry
- Number theory

**String & Text Operations**
- String manipulation (50 tasks)
- Text processing
- Regular expressions

**Systems & I/O**
- File handling
- Database operations
- Networking

**Advanced Topics**
- Encryption
- Compression
- Concurrency
- Graphics

### Programmatic Access Methods

#### Option 1: MediaWiki API (RECOMMENDED)

**Endpoint:** `https://rosettacode.org/w/api.php`

**List all tasks:**
```
GET /w/api.php?action=query&list=categorymembers&cmtitle=Category:Programming_Tasks&cmlimit=500&format=json
```

**Get raw page content:**
```
GET /w/index.php?action=raw&title={task_name}
```

**Page format:** Each task page has:
- Task description at top
- Language sections marked with `=={{header|LanguageName}}==`
- Code blocks within language sections

**Pagination:** Use `cmcontinue` parameter when results exceed 500

#### Option 2: RosettaCodeData Repository (ALTERNATIVE)

**Repository:** https://github.com/acmeism/RosettaCodeData

**Quick clone:**
```bash
git clone https://github.com/acmeism/RosettaCodeData --single-branch --depth=1
```

**Structure:**
- `/Lang/{LanguageName}/` - Solutions by language
- `/Task/{TaskName}/` - Solutions by task
- `/Conf/lang.yaml` - Language configuration

**Tools included:**
- `rcd-api-list-all-tasks` - List all tasks
- `rcd-api-list-all-langs` - List all languages

### Existing Eiffel Coverage

**159 tasks already solved in Eiffel, including:**

| Category | Examples |
|----------|----------|
| Algorithms | Factorial, Fibonacci, Sieve of Eratosthenes |
| Data Structures | Arrays, Stack, Tree traversal |
| Sorting | Bubble sort, Quicksort, Merge sort |
| Math | Prime decomposition, Perfect numbers |
| Strings | Reverse a string, Palindrome detection |

**Gap Analysis:** ~1,179 tasks WITHOUT Eiffel solutions

### Data Extraction Strategy

1. **Use MediaWiki API** to fetch task list
2. **For each task:**
   - Fetch raw wiki content
   - Parse language sections (regex: `=={{header|(.+?)}}==`)
   - Extract code blocks for Java, Python, C#, etc.
   - Check if Eiffel section exists
3. **Store in simple_sql database**
4. **Generate Eiffel solutions** for tasks without them (Claude Max)
5. **Validate** each Eiffel solution (compile + DBC)

### Key Insights for simple_rosetta

1. **API is stable** - MediaWiki API is well-documented, widely used
2. **1,179 opportunities** - Tasks without Eiffel solutions
3. **Structure is consistent** - Header pattern makes parsing reliable
4. **Multiple languages per task** - Can extract Java/Python/C# in parallel
5. **RosettaCodeData backup** - Pre-extracted data available if API fails

### Rate Limiting Considerations

- No explicit rate limits documented
- Recommend: 1 request/second to be polite
- Batch fetching with cmcontinue reduces calls
- Cache responses locally to avoid re-fetching

### Sources

- [Rosetta Code Main](https://rosettacode.org/wiki/Rosetta_Code)
- [Category:Programming Tasks](https://rosettacode.org/wiki/Category:Programming_Tasks)
- [Category:Eiffel](https://rosettacode.org/wiki/Category:Eiffel)
- [Category:Solutions by Programming Task](https://rosettacode.org/wiki/Category:Solutions_by_Programming_Task)
- [Rosetta Code API/MediaWiki](https://rosettacode.org/wiki/Rosetta_Code:API/MediaWiki)
- [RosettaCodeData GitHub](https://github.com/acmeism/RosettaCodeData)
- [Rosetta Code/Count examples](https://rosettacode.org/wiki/Rosetta_Code/Count_examples)

---

## Step 2: Research Tech-Stack Libraries

### Existing Multilingual Code Datasets

#### xCodeEval (NTU-NLP, 2023)

| Feature | Value |
|---------|-------|
| Scale | 25 million document-level coding examples |
| Problems | 7,500 unique problems |
| Languages | Up to 17 programming languages |
| Tasks | 7 (synthesis, translation, repair, retrieval, etc.) |
| Evaluation | Execution-based with ExecEval engine |
| Access | Hugging Face: `NTU-NLP-sg/xCodeEval` |
| License | CC BY-NC 4.0 |

**Key files:**
- `problem_descriptions.jsonl` - Problem metadata
- `unittest_db.json` - Test cases mapped by src_uid

#### CodeXGLUE (Microsoft, 2021)

| Feature | Value |
|---------|-------|
| Tasks | 10 tasks across 14 datasets |
| Scope | Code understanding, generation, translation |
| Includes | BigCloneBench, POJ-104, Defects4J, CodeSearchNet |
| Focus | ML benchmark for code intelligence |

### AI Code Translation Tools

#### Research Tools

| Tool | Description | Accuracy |
|------|-------------|----------|
| **CoTran** (ECAI 2024) | LLM + compiler feedback + symbolic execution | 48.68% functional equivalence (Py->Java) |
| **LLMLift** (Dec 2024) | Formal verification for LLM outputs | Java/C/C++ to various targets |
| **TransCoder** (Facebook) | Unsupervised deep learning, 2.8M projects | Up to 90% accuracy |
| **SteloCoder** | Decoder-only LLM, multi-language to Python | No input language needed |

#### Commercial Tools

- [AI Code Convert](https://aicodeconvert.com/) - Free web-based converter
- [CodeConvert AI](https://www.codeconvert.ai/) - Commercial conversion service

### Code Parsing Libraries

#### Tree-sitter (Recommended)

| Feature | Value |
|---------|-------|
| Type | Incremental parsing system |
| Written in | Rust (generates C/WASM parsers) |
| Languages | 40+ grammars available |
| Performance | Sub-millisecond incremental updates |
| Used by | GitHub, Neovim, Helix |

**Rust usage:**
```rust
let mut parser = Parser::new();
parser.set_language(&tree_sitter_java::LANGUAGE.into())?;
let tree = parser.parse(source_code, None)?;
```

**Language grammars available:**
- tree-sitter-java, tree-sitter-python, tree-sitter-javascript
- tree-sitter-c-sharp, tree-sitter-go, tree-sitter-rust
- No tree-sitter-eiffel (opportunity!)

#### Python Scraping/Parsing Stack

| Library | Purpose |
|---------|---------|
| **requests/httpx** | HTTP client for API calls |
| **BeautifulSoup** | HTML/XML parsing |
| **lxml** | Fast XML/HTML parsing (C-based) |
| **Selectolax** | Ultra-fast HTML parser |
| **Scrapy** | Full scraping framework |

### Polyglot Transpilers

#### awesome-polyglot Collection

**To JavaScript:**
- Opal, ruby2js (Ruby)
- Transcrypt, pyjs (Python)
- Scala.js (Scala)
- ClojureScript (Clojure)
- Fable (F#)

**Multi-target:**
- **Haxe** - Cross-compiles to JS, C++, C#, Java, Python
- **Nim** - Compiles to C, C++, JS
- **Oczor** - Compiles to JS, Lua, Ruby, Emacs Lisp

### Polyglot Runtime Environments

#### GraalVM

- Run multiple languages in same JVM
- Direct interop between Java, JavaScript, Python, Ruby, R
- Share data in same memory space

#### VS Code Polyglot Notebooks

- Multiple languages in same notebook
- Powered by .NET Interactive
- Share variables between languages

### Code Comparison Tools

| Tool | Capability |
|------|------------|
| **srclib** (Sourcegraph) | Polyglot code analysis, common output format |
| **SwapCode AI** | Cross-language semantic comparison |
| **PolyglotPiranha** | Automated code transformations, tree-sitter based |

### Key Insights for simple_rosetta

1. **No Eiffel in existing datasets** - xCodeEval/CodeXGLUE don't include Eiffel
2. **Tree-sitter is the standard** - Use for parsing Java/Python/C# code
3. **No tree-sitter-eiffel** - Could create one (separate project)
4. **Execution-based validation** - xCodeEval's approach matches our compile+test plan
5. **LLM translation works** - 50-90% accuracy depending on approach
6. **Claude Max advantage** - Skip the complex fine-tuning, use frontier model directly

### Recommended Tech Stack for simple_rosetta

| Component | Technology |
|-----------|------------|
| API Client | simple_http (Eiffel) |
| Database | simple_sql (SQLite) |
| Wiki Parsing | Regex for MediaWiki format |
| Code Parsing | Tree-sitter (if needed for analysis) |
| Eiffel Generation | Claude Max API |
| Validation | ec -batch compile |

### Sources

- [xCodeEval GitHub](https://github.com/ntunlp/xCodeEval)
- [CodeXGLUE GitHub](https://github.com/microsoft/CodeXGLUE)
- [CoTran GitHub](https://github.com/PrithwishJana/CoTran)
- [Tree-sitter](https://github.com/tree-sitter/tree-sitter)
- [awesome-polyglot](https://github.com/lxsmnsyc/awesome-polyglot)
- [srclib](https://github.com/sourcegraph/srclib)
- [LLM Code Translation](https://lokalise.com/blog/llm-code-translation/)

---

## Step 3: Eiffel Ecosystem Research (simple_* First)

### simple_rosetta Requirements Mapping

| Requirement | simple_* Library | Status | Notes |
|-------------|-----------------|--------|-------|
| HTTP API calls | simple_http | ✅ COVERED | `SIMPLE_HTTP_QUICK.get_json()`, authentication support |
| JSON parsing | simple_json | ✅ COVERED | `JSON_PARSER`, `JSON_OBJECT`, JSONPath, Schema validation |
| SQLite database | simple_sql | ✅ COVERED | `SIMPLE_SQL_QUICK`, fluent API, prepared statements |
| AI code generation | simple_ai_client | ✅ COVERED | Claude, Ollama, OpenAI; `SIMPLE_AI_QUICK` facade |
| Wiki markup parsing | simple_regex | ✅ COVERED | PCRE regex, `=={{header|(.+?)}}==` pattern matching |
| Compiler validation | simple_process | ✅ COVERED | `SIMPLE_PROCESS_QUICK.execute()`, SCOOP-compatible |
| Rate limiting | simple_rate_limiter | ✅ COVERED | Token bucket, sliding window for polite API access |
| Caching | simple_cache | ✅ COVERED | In-memory cache to avoid re-fetching pages |
| Logging | simple_logger | ✅ COVERED | Progress tracking, error logging |
| Testing | simple_testing | ✅ COVERED | Test framework for validation suite |

### Library Details

#### simple_http (API Client)
```eiffel
-- Fetch Rosetta Code task list
http: SIMPLE_HTTP_QUICK
result := http.get_json ("https://rosettacode.org/w/api.php?action=query&list=categorymembers&cmtitle=Category:Programming_Tasks&cmlimit=500&format=json")

-- Fetch raw wiki content
raw_content := http.get_text ("https://rosettacode.org/w/index.php?action=raw&title=" + task_name)
```

#### simple_sql (Task Database)
```eiffel
-- Schema for tasks and solutions
db.execute ("CREATE TABLE tasks (id INTEGER PRIMARY KEY, name TEXT, description TEXT, has_eiffel BOOLEAN)")
db.execute ("CREATE TABLE solutions (id INTEGER PRIMARY KEY, task_id INTEGER, language TEXT, code TEXT)")

-- Query tasks without Eiffel
missing := db.query ("SELECT name FROM tasks WHERE has_eiffel = 0")
```

#### simple_json (API Response Parsing)
```eiffel
-- Parse MediaWiki API response
parser: JSON_PARSER
json := parser.parse (api_response)
tasks := json.object_at ("query").array_at ("categorymembers")
across tasks as t loop
    task_name := t.item.string_at ("title")
end
```

#### simple_regex (Wiki Markup Parsing)
```eiffel
-- Extract language sections from wiki
regex: SIMPLE_REGEX
regex.compile ("=={{header\|([^}]+)}}==")
across regex.all_matches (wiki_content) as m loop
    language := m.item.group (1)  -- "Java", "Python", "C#", etc.
end
```

#### simple_ai_client (Eiffel Generation)
```eiffel
-- Generate Eiffel solution from other language examples
ai: SIMPLE_AI_QUICK
prompt := "Translate this Java code to Eiffel with full DBC contracts:%N" + java_code
eiffel_code := ai.ask_claude (prompt)
```

#### simple_process (Compiler Validation)
```eiffel
-- Validate generated Eiffel compiles
proc: SIMPLE_PROCESS_QUICK
result := proc.execute ("ec -batch -config test.ecf -target validate -c_compile")
if result.exit_code = 0 then
    -- Valid Eiffel code
end
```

### Gap Analysis

| Gap | Solution | Priority |
|-----|----------|----------|
| Wiki markup → AST | Use simple_regex (sufficient for header patterns) | Not needed |
| Eiffel code parser | simple_eiffel_parser exists | ✅ Available |
| Diff/comparison | Could use simple_diff or basic string comparison | Low |
| Template generation | simple_template for instruction pair generation | ✅ Available |

### No Gobo/ISE Dependencies Required

All simple_rosetta requirements are covered by existing simple_* libraries:

1. **simple_http** - HTTP client for MediaWiki API
2. **simple_json** - Parse API responses
3. **simple_sql** - SQLite database for tasks/solutions
4. **simple_regex** - Parse wiki markup patterns
5. **simple_ai_client** - Generate Eiffel via Claude Max
6. **simple_process** - Run `ec -batch` for validation
7. **simple_cache** - Cache API responses (1 req/sec politeness)
8. **simple_rate_limiter** - Enforce rate limits
9. **simple_logger** - Progress tracking
10. **simple_testing** - Validation test suite

### Recommended simple_rosetta Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      SIMPLE_ROSETTA                              │
│  (Facade: ROSETTA_CLIENT, ROSETTA_QUICK)                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │ TASK_FETCHER │  │ WIKI_PARSER  │  │EIFFEL_GENERATOR│          │
│  │ (simple_http)│  │(simple_regex)│  │(simple_ai_client)│        │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │ TASK_STORE   │  │  VALIDATOR   │  │INSTRUCTION_GEN│           │
│  │ (simple_sql) │  │(simple_process)│ │(simple_template)│        │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Classes to Create

| Class | Purpose | Depends On |
|-------|---------|------------|
| `ROSETTA_CLIENT` | MediaWiki API wrapper | simple_http, simple_json |
| `ROSETTA_TASK` | Task data model | - |
| `ROSETTA_SOLUTION` | Solution data model | - |
| `WIKI_PARSER` | Parse wiki markup | simple_regex |
| `TASK_STORE` | SQLite persistence | simple_sql |
| `EIFFEL_GENERATOR` | AI translation | simple_ai_client |
| `EIFFEL_VALIDATOR` | Compile validation | simple_process |
| `INSTRUCTION_PAIR_GEN` | Training data | simple_template |
| `ROSETTA_QUICK` | One-liner facade | All above |

### Conclusion

**No gaps requiring new simple_* libraries.** The existing ecosystem fully covers simple_rosetta requirements. The project can proceed with:

1. Core dependencies: simple_http, simple_json, simple_sql, simple_regex
2. AI/Validation: simple_ai_client, simple_process
3. Infrastructure: simple_cache, simple_rate_limiter, simple_logger, simple_testing

---

## Step 4: Developer Pain Points Research

### Primary Pain Points (from Hacker News, forums, documentation)

| Pain Point | Severity | simple_rosetta Addresses? |
|------------|----------|---------------------------|
| **No working library ecosystem** | Critical | ✅ Yes - provides curated, working examples |
| **Simple things take too much effort** | High | ✅ Yes - shows idiomatic Eiffel for common tasks |
| **Finding examples is hard** | High | ✅ Yes - 1,179 new Eiffel examples |
| **Translating knowledge from Java/Python/C#** | High | ✅ Yes - side-by-side comparisons |
| **Outdated tooling compared to modern IDEs** | Medium | ❌ No (separate concern) |
| **Learning both Method and Language** | Medium | ⚠️ Partially - examples show DBC in practice |
| **Syntax differences (no curly braces, Result)** | Low | ✅ Yes - many examples normalize patterns |

### Key Quotes from Developer Community

> "There was no working ecosystem of libraries... getting them to actually work with your version of Eiffel was a huge hassle."
> — [Hacker News discussion](https://news.ycombinator.com/item?id=22281615)

> "Too many simple things just took way too much effort to do."
> — [Hacker News discussion](https://news.ycombinator.com/item?id=22281615)

> "Simple programming tasks like file IO, regex matching, printing a text table, making HTTP requests required disproportionate effort."
> — Developer feedback

### The "Coming From X" Problem

Developers arriving from Java, Python, C# face these translation challenges:

| Concept | Java/C#/Python | Eiffel | Confusion Level |
|---------|----------------|--------|-----------------|
| Return values | `return x` | `Result := x` | High |
| Loops | `while (cond)` | `until cond` (inverted) | High |
| Code blocks | `{ }` | `do ... end` | Medium |
| Switch/case | `switch/case/break` | `inspect/when` (no break) | Medium |
| Null handling | `null`/`None` | `Void` + attached/detachable | High |
| Contracts | Annotations/asserts | Built-in `require`/`ensure` | Medium |
| Inheritance | Single + interfaces | Multiple inheritance | High |
| Properties | Get/set methods | Uniform access (queries) | Medium |

### What simple_rosetta Solves

**The Core Value Proposition:**
> "Show me how to do [X task] in Eiffel, when I already know how in Java/Python/C#"

**Example Use Case:**
1. Developer knows how to implement "Bubble Sort" in Python
2. Searches simple_rosetta for "Bubble Sort"
3. Sees Python solution side-by-side with Eiffel solution
4. Eiffel solution includes:
   - Full DBC contracts (preconditions, postconditions)
   - Proper Eiffel idioms (across loops, Result)
   - Comments explaining "In Python you'd do X, in Eiffel we do Y"

### Rosetta Code Gap = Opportunity

| Metric | Value |
|--------|-------|
| Total Rosetta Code tasks | 1,338 |
| Tasks WITH Eiffel solutions | 159 (12%) |
| Tasks WITHOUT Eiffel | **1,179 (88%)** |
| Potential instruction pairs | ~5,000-10,000 |

**Existing Eiffel contributor:** [Javier Velilla](https://github.com/jvelilla/RosettaCode) has GitHub repo with some Rosetta Code examples.

### Target Developer Personas

| Persona | Pain Point | simple_rosetta Value |
|---------|------------|----------------------|
| **Java Developer** | "How do I do generics/collections in Eiffel?" | Side-by-side Java→Eiffel translations |
| **Python Developer** | "Eiffel syntax is verbose, show me patterns" | Compact examples with idioms |
| **C# Developer** | "I understand LINQ, how do agents work?" | Cross-language pattern mapping |
| **CS Student** | "Learning DBC, need many examples" | 1,179 examples all with contracts |
| **Eiffel Newcomer** | "I want to solve X, what's the Eiffel way?" | Searchable task database |

### What We're NOT Solving

These remain separate concerns:
- IDE/tooling modernization (EiffelStudio roadmap)
- Language specification (ISE/ECMA)
- Core library maintenance (ISE/Gobo)
- Marketing/community building

simple_rosetta focuses narrowly on: **Eiffel examples with cross-language context**

### Sources

- [Learning Eiffel](https://www.eiffel.org/doc/eiffel/Learning_Eiffel)
- [Hacker News: Eiffel programming language](https://news.ycombinator.com/item?id=22281615)
- [Eiffel Syntax Guide](https://eiffel-guide.com/)
- [Rosetta Code](https://rosettacode.org/wiki/Rosetta_Code)
- [jvelilla/RosettaCode GitHub](https://github.com/jvelilla/RosettaCode)

---

## Step 5: Innovation Hat - Unique Value Propositions

### What Makes simple_rosetta Unique?

#### Innovation 1: The Eiffel Sieve

**Concept:** Use Eiffel's strict compiler + Design by Contract as a **quality filter** for AI-generated code.

```
AI-Generated Code --> Eiffel Sieve (compile + DBC) --> Verified Solution
                           |
                           v
                    Reject + Log (learn errors)
```

**Why this matters:**
- LLMs generate plausible-looking code that often has subtle bugs
- Eiffel's void safety catches null pointer errors at compile time
- DBC preconditions catch logic errors (wrong inputs)
- DBC postconditions catch incorrect results
- DBC invariants catch invalid state

**The virtuous cycle:**
1. Generate Eiffel code with Claude Max
2. Compile with `ec -batch` -> catches syntax/type errors
3. Run with contracts enabled -> catches logic errors
4. Failed code becomes **negative training examples**
5. Successful code becomes **positive training examples**

**No other language can do this as effectively.** Java/Python rely on runtime exceptions. Eiffel catches errors before they happen.

#### Innovation 2: Dual-Purpose Output

| Output | Purpose | Consumer |
|--------|---------|----------|
| **Rosetta Eiffel Database** | Searchable task/solution repository | Developers learning Eiffel |
| **LLM Instruction Pairs** | Fine-tuning data for Polyglot Translator | Qwen2.5-Coder training |

One pipeline, two valuable outputs. The database is immediately useful; the training data enables the next phase.

#### Innovation 3: Cross-Language Context Preservation

Unlike simple code translation, we preserve **pedagogical context**:

```json
{
  "task": "Bubble Sort",
  "java_solution": "...",
  "python_solution": "...",
  "eiffel_solution": "...",
  "translation_notes": [
    "Java uses while(true) with break; Eiffel uses from/until loop",
    "Python swap idiom a,b = b,a becomes Eiffel 3-line swap",
    "Eiffel adds require: array not void; ensure: sorted"
  ],
  "eiffel_idioms": ["across loop", "Result assignment", "class invariant"]
}
```

This creates **instructional value** beyond mere code translation.

#### Innovation 4: Validated Training Data

| Dataset | Validation Level | Eiffel Coverage |
|---------|------------------|-----------------|
| CodeXGLUE | None (raw code) | None |
| xCodeEval | Unit tests | None |
| **simple_rosetta** | **Compile + DBC + tests** | **100%** |

Our training data is **proven correct** by the strictest compiler in mainstream use.

#### Innovation 5: Community Contribution Model

**Contribution paths:**
1. **AI-first:** Claude Max generates, humans review
2. **Community:** Eiffel developers submit solutions
3. **Import:** Pull existing 159 Rosetta Code Eiffel solutions

All paths feed through the Eiffel Sieve for validation.

#### Innovation 6: Polyglot Translator Specialist

Not a general-purpose Eiffel model, but a **specialist** that excels at:

| Skill | Training Source |
|-------|-----------------|
| Java -> Eiffel | Java + Eiffel solution pairs |
| Python -> Eiffel | Python + Eiffel solution pairs |
| C# -> Eiffel | C# + Eiffel solution pairs |
| "How do I do X in Eiffel?" | Instructional pairs |

This focused model can be **smaller and faster** than a generalist, while being **more accurate** for its specialty.

### Competitive Differentiation

| Competitor | What They Offer | What We Offer Better |
|------------|-----------------|----------------------|
| **Rosetta Code (raw)** | Unstructured wiki | Structured database + API |
| **ChatGPT/Claude** | General code help | Specialist Eiffel focus, validated |
| **CodeXGLUE** | Training data | Eiffel coverage, contract validation |
| **Copilot** | Code completion | Cross-language pedagogy |

### The "Small Group vs Goliath" Strategy

We can't outspend Google, Microsoft, or Meta. But we can:

1. **Focus narrowly** on what they ignore (Eiffel)
2. **Leverage what we have** (strictest compiler as validator)
3. **Build what they can't** (Eiffel-specific instruction pairs)
4. **Serve who they don't** (Eiffel learners and developers)

> "We need to answer THEIR questions."

When a Java developer asks "How do I do X in Eiffel?", the big LLMs often hallucinate. Our specialist won't.

---

## Next: Step 6 - Synthesize Design Strategy

## Step 6: Design Strategy Synthesis

### Core Design Decisions

Based on Steps 1-5 research, here are the strategic design decisions:

#### Decision 1: Database-First, Model-Second

**Order of operations:**
1. Build the Rosetta Eiffel Database (immediate value)
2. Use database to generate instruction pairs
3. Fine-tune Polyglot Translator later

**Rationale:** The database is useful on day one. The model needs the database to exist first.

#### Decision 2: MediaWiki API as Primary Source

**Why MediaWiki API over RosettaCodeData repo:**
- Real-time access (repo may be stale)
- Structured JSON responses
- Pagination support for 1,338 tasks
- Can detect existing Eiffel solutions

**Fallback:** Clone RosettaCodeData if API is unavailable

#### Decision 3: SQLite Database Schema

```sql
-- Core tables
CREATE TABLE tasks (
    id INTEGER PRIMARY KEY,
    name TEXT UNIQUE NOT NULL,
    description TEXT,
    category TEXT,
    has_eiffel BOOLEAN DEFAULT FALSE,
    eiffel_validated BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE solutions (
    id INTEGER PRIMARY KEY,
    task_id INTEGER REFERENCES tasks(id),
    language TEXT NOT NULL,
    code TEXT NOT NULL,
    source TEXT, -- 'rosetta', 'generated', 'community'
    validated BOOLEAN DEFAULT FALSE,
    validation_log TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(task_id, language)
);

CREATE TABLE instruction_pairs (
    id INTEGER PRIMARY KEY,
    task_id INTEGER REFERENCES tasks(id),
    instruction TEXT NOT NULL,
    input TEXT,
    output TEXT NOT NULL,
    pair_type TEXT, -- 'translate', 'explain', 'generate'
    source_languages TEXT, -- JSON array
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE validation_runs (
    id INTEGER PRIMARY KEY,
    solution_id INTEGER REFERENCES solutions(id),
    compile_success BOOLEAN,
    compile_errors TEXT,
    contract_violations TEXT,
    test_passed BOOLEAN,
    run_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Decision 4: Pipeline Architecture

Phase A: Data Collection (TASK_FETCHER -> WIKI_PARSER -> SOLUTION_EXTRACTOR -> TASK_STORE)
Phase B: Eiffel Generation (EIFFEL_GENERATOR -> EIFFEL_VALIDATOR -> SOLUTION_STORE)
Phase C: Training Data (PAIR_GENERATOR -> PAIR_VALIDATOR -> JSONL_EXPORTER)

#### Decision 5: Phased Implementation

| Phase | Deliverable | Dependencies | Effort |
|-------|-------------|--------------|--------|
| **1A** | Task fetcher + parser | simple_http, simple_json, simple_regex | 1 week |
| **1B** | SQLite storage | simple_sql | 1 week |
| **1C** | Import existing Eiffel | Wiki parser | 1 week |
| **2A** | Eiffel generator | simple_ai_client | 2 weeks |
| **2B** | Eiffel validator | simple_process | 1 week |
| **2C** | Validation pipeline | Generator + Validator | 1 week |
| **3A** | Instruction pair templates | simple_template | 1 week |
| **3B** | JSONL exporter | simple_json | 1 week |
| **3C** | Quality filters | DBC validation | 1 week |

**Total estimated effort:** 10 weeks for complete pipeline

#### Decision 6: Key Classes

| Class | Responsibility | simple_* Dependencies |
|-------|----------------|----------------------|
| ROSETTA_CLIENT | MediaWiki API wrapper | simple_http, simple_json |
| ROSETTA_TASK | Task data model | - |
| ROSETTA_SOLUTION | Solution data model | - |
| WIKI_PARSER | Parse wiki markup | simple_regex |
| TASK_STORE | SQLite persistence | simple_sql |
| EIFFEL_GENERATOR | Claude Max translation | simple_ai_client |
| EIFFEL_VALIDATOR | Compile + DBC check | simple_process |
| INSTRUCTION_PAIR | Training pair model | - |
| PAIR_GENERATOR | Create instruction pairs | simple_template |
| JSONL_EXPORTER | Export for training | simple_json |
| ROSETTA_QUICK | One-liner facade | All above |

#### Decision 7: Priority Languages

| Priority | Language | Rosetta Coverage | Reason |
|----------|----------|------------------|--------|
| 1 | Python | ~900 tasks | Most popular, most examples |
| 2 | Java | ~700 tasks | Enterprise standard, OOP |
| 3 | C | ~600 tasks | Systems programming |
| 4 | C# | ~400 tasks | .NET ecosystem |
| 5 | JavaScript | ~500 tasks | Web development |

#### Decision 8: Validation Levels

- Level 0: Failed to compile
- Level 1: Compiles, contracts fail
- Level 2: Compiles, contracts pass (VALID)
- Level 3: Compiles, contracts pass, tests pass (FULLY VALIDATED)

#### Decision 9: Rate Limiting Strategy

1 request per second to Rosetta Code. Cache responses in SQLite. Exponential backoff on errors.

#### Decision 10: Success Metrics

| Metric | Target |
|--------|--------|
| Tasks imported | 1,338 |
| Eiffel solutions generated | 1,000 |
| Validation success rate | >70% |
| Instruction pairs | 5,000 |
| Model accuracy improvement | >20% |

---

## Next: Step 7 - Produce Design/Implementation Report

## Step 7: Design/Implementation Report

### Report Location

The complete design and implementation report has been produced:

**File:** `/d/prod/reference_docs/designs/SIMPLE_ROSETTA_DESIGN.md`

### Report Contents

1. **Executive Summary** - Core innovation (Eiffel Sieve), strategic context
2. **Problem Statement** - The 1,179 task gap, developer pain points
3. **Technical Architecture** - Dependencies, schema, class structure
4. **Pipeline Flow** - 3-phase process diagram
5. **Implementation Plan** - 10-week phased approach with code examples
6. **API Design** - ROSETTA_QUICK facade, CLI interface
7. **Validation Strategy** - 4 validation levels, Eiffel Sieve process
8. **Risk Mitigation** - Rate limits, costs, validation rate
9. **Success Metrics** - Targets for tasks, solutions, pairs
10. **Next Steps** - Concrete action items
11. **Appendix** - Prompt template for Eiffel generation

### Key Deliverables

| Deliverable | Location |
|-------------|----------|
| Research notes | `/d/prod/reference_docs/research/SIMPLE_ROSETTA_RESEARCH.md` |
| Design report | `/d/prod/reference_docs/designs/SIMPLE_ROSETTA_DESIGN.md` |
| Architecture update | `/d/prod/reference_docs/designs/EIFFEL_SPECIALIST_MODELS_ARCHITECTURE.md` |

---

## Research Complete

**7-Step Research Process for simple_rosetta: COMPLETE**

| Step | Status | Key Finding |
|------|--------|-------------|
| 1. Rosetta Code Specs | ✅ | 1,338 tasks, MediaWiki API, 1,179 gap |
| 2. Tech-Stack Libraries | ✅ | Tree-sitter, xCodeEval (no Eiffel) |
| 3. Eiffel Ecosystem | ✅ | 10 simple_* libs cover all needs |
| 4. Developer Pain Points | ✅ | 5/7 pain points addressed |
| 5. Innovation Hat | ✅ | 6 unique value propositions |
| 6. Design Strategy | ✅ | 10 design decisions |
| 7. Implementation Report | ✅ | Complete design document |

**Ready to begin implementation.**
