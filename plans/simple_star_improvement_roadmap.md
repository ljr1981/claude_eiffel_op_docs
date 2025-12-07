# simple_* Improvement Roadmap

**Date:** December 7, 2025
**Based on:** simple_ecosystem_assessment.md

---

## Phase 1: Foundation Hardening (Critical)

### Step 1.1: Standardize ECF Capabilities
Every simple_* library needs explicit capability declarations:

```xml
<capability>
    <concurrency support="scoop" use="thread"/>
    <void_safety support="all"/>
</capability>
```

**Libraries requiring updates:**
- simple_csv
- simple_datetime
- simple_hash
- simple_json
- simple_logger
- simple_markdown
- simple_xml
- simple_process

### Step 1.2: Void-Safety Audit
Compile each library with `void_safety support="all"` and fix any errors:
- Add `attached` keywords where needed
- Use `detachable` for truly optional values
- Fix all VEVI/VJAR/VUTA errors

### Step 1.3: Create package.iron for Every Library
Each library gets a manifest file:

```
package simple_json

project
    simple_json = "simple_json.ecf"

note
    title: Simple JSON
    description: "High-level JSON parsing for Eiffel"
    collection: simple
    tags: json,parser,builder
    license: MIT
    link[source]: https://github.com/ljr1981/simple_json

end
```

---

## Phase 2: SCOOP Variants

### Step 2.1: Create Multi-Variant ECFs
Follow EWF pattern - ship 3 variants minimum:

```
simple_json.ecf           -- Default (thread)
simple_json-safe.ecf      -- Void-safe only
simple_json-scoop.ecf     -- SCOOP + void-safe
```

### Step 2.2: Priority Libraries for SCOOP
1. simple_json (most used)
2. simple_logger (concurrent logging)
3. simple_cache (thread-safe caching)
4. simple_sql (connection pooling)

---

## Phase 3: Missing Wrapper Libraries

### Step 3.1: simple_http
Wrap `http_client` contrib library:
```eiffel
-- Target API
http.get ("https://api.example.com/users").json
http.post (url).body (json).send
http.with_header ("Authorization", token).get (url)
```

### Step 3.2: simple_encryption
Wrap `eel` (Eiffel Encryption Library):
```eiffel
-- Target API
encrypt (plaintext, key)
decrypt (ciphertext, key)
aes_256_encrypt (data, key, iv)
```

### Step 3.3: simple_compression
Wrap `wsf_compression` or zlib:
```eiffel
-- Target API
gzip (data): STRING
gunzip (compressed): STRING
deflate (data): STRING
```

### Step 3.4: simple_config
Unified configuration loading:
```eiffel
-- Target API
config.load_env (".env")
config.load_yaml ("config.yml")
config.get ("database.host")
config.get_or_default ("port", "8080")
```

---

## Phase 4: Iron Publication

### Step 4.1: Register on iron.eiffel.com
1. Create account
2. Register simple_* collection

### Step 4.2: Publish Foundation Libraries First
1. simple_json
2. simple_uuid
3. simple_base64
4. simple_hash
5. simple_validation

### Step 4.3: Publish Remaining Libraries
- Staged rollout as void-safety/SCOOP audits complete

---

## Phase 5: CI/CD Pipeline

### Step 5.1: GitHub Actions Workflow
```yaml
- Compile with void_safety="all"
- Compile with concurrency="scoop"
- Run all tests
- Validate package.iron
```

### Step 5.2: Quality Gates
- No library merges without:
  - Void-safe compilation passing
  - SCOOP compilation passing
  - 100% test pass rate

---

## Phase 6: Documentation Standards

### Step 6.1: Every Library Gets
- README.md (standardized format)
- docs/index.html
- docs/css/style.css
- docs/images/logo.png
- CHANGELOG.md

### Step 6.2: API Documentation
- Generate from code comments
- Include usage examples for every public feature

---

## Execution Priority

| Priority | Task | Impact |
|----------|------|--------|
| P0 | ECF capability standardization | Prevents undefined behavior |
| P0 | Void-safety audit | Production safety |
| P1 | package.iron files | Enables distribution |
| P1 | simple_http wrapper | Most requested feature |
| P2 | SCOOP variants | Concurrent applications |
| P2 | Iron publication | Discoverability |
| P3 | CI/CD pipeline | Quality enforcement |
| P3 | Additional wrappers | Ecosystem completeness |

---

## Library-by-Library Checklist

| Library | ECF Fixed | Void-Safe | SCOOP | package.iron | Iron Published |
|---------|-----------|-----------|-------|--------------|----------------|
| simple_base64 | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_cache | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_cors | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_csv | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_datetime | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_hash | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_htmx | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_json | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_jwt | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_logger | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_markdown | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_pdf | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_process | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_randomizer | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_rate_limiter | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_regex | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_smtp | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_sql | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_template | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_uuid | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_validation | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_websocket | [ ] | [ ] | [ ] | [ ] | [ ] |
| simple_xml | [ ] | [ ] | [ ] | [ ] | [ ] |

---

## New Libraries to Create

| Library | Wraps | Priority |
|---------|-------|----------|
| simple_http | http_client | P1 |
| simple_encryption | eel | P2 |
| simple_compression | wsf_compression/zlib | P2 |
| simple_config | preferences | P2 |
| simple_cli | argument_parser | P3 |
| simple_archive | etar | P3 |
| simple_i18n | i18n | P3 |
| simple_mongo | mongo | P3 |
