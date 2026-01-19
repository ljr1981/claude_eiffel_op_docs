# Simple Eiffel Ecosystem Gap Projects

**Research Date:** 2026-01-17
**Methodology:** 7-Step Deep Research (Market → Domain → Technical → Competitive → Ecosystem → Innovation → Synthesis)
**Status:** COMPREHENSIVE - 16 Gap-Projects Identified

---

## Executive Summary

This document identifies **16 gap-projects** for the simple_* Eiffel ecosystem:
- **10 NEW projects** from fresh research
- **6 REMAINING projects** from previous SIMPLE_CHART_RESEARCH.md (simple_chart was built)

Each project specification is formatted as a **plan-prompt for simple_code** to guide Claude-assisted development.

---

## Research Methodology

### 7-Step Analysis Performed:
1. **Market Research** - What do modern developers need?
2. **Domain Deep Dive** - What does each gap area require?
3. **Technical Assessment** - Is it feasible in Eiffel?
4. **Competitive Analysis** - What do other ecosystems offer?
5. **Ecosystem Assessment** - What simple_* libs exist? (78 catalogued)
6. **Innovation Opportunities** - Where can DBC add unique value?
7. **Strategic Synthesis** - Prioritize by impact and feasibility

### Ecosystem Baseline:
- **78 existing simple_* libraries** catalogued
- **12 major gaps** identified in previous research
- **Cross-referenced** against ECOSYSTEM_EXPANSION_ROADMAP_2025.md

---

## PART A: 10 NEW Gap-Projects

---

### 1. simple_email

**Gap Filled:** No email client in ecosystem (SMTP sending, IMAP/POP3 reading)

**Plan-Prompt for simple_code:**
```
Build simple_email - an Eiffel email client library for the simple_* ecosystem.

REQUIREMENTS:
1. SMTP client for sending emails (RFC 5321)
   - Plain text and HTML body support
   - Attachments via MIME multipart (RFC 2045-2049)
   - TLS/STARTTLS encryption
   - Authentication: PLAIN, LOGIN, CRAM-MD5

2. IMAP client for reading emails (RFC 3501)
   - Connect to mailbox, list folders
   - Fetch messages (headers, body, attachments)
   - Search messages by criteria
   - Mark as read/unread, delete, move

3. POP3 client for simple retrieval (RFC 1939)
   - List and fetch messages
   - Delete after retrieval option

CORE CLASSES:
- SIMPLE_EMAIL: Facade with quick send/receive methods
- EMAIL_MESSAGE: Represents an email (from, to, cc, bcc, subject, body, attachments)
- EMAIL_ATTACHMENT: File attachment with MIME type
- SMTP_CLIENT: Send emails via SMTP
- IMAP_CLIENT: Read/manage emails via IMAP
- POP3_CLIENT: Simple email retrieval

DEPENDENCIES: simple_file, simple_json (for config), simple_encryption (for auth)

CONTRACTS (DBC Focus):
- Precondition: Valid email addresses (RFC 5322 format)
- Precondition: SMTP server connected before send
- Postcondition: Message queued/sent confirmation
- Invariant: Connection state consistency

PATTERNS: Fluent builder for message composition, connection pooling for multiple sends

TESTING: Mock SMTP/IMAP servers for unit tests, integration tests against real servers
```

**Complexity:** Medium-High (protocol implementation)
**Dependencies:** simple_file, simple_json, simple_encryption
**Use Cases:** Notifications, reports, automated emails, email processing

---

### 2. simple_s3

**Gap Filled:** No cloud storage support (AWS S3 is the de facto standard)

**Plan-Prompt for simple_code:**
```
Build simple_s3 - an S3-compatible object storage client for the simple_* ecosystem.

REQUIREMENTS:
1. S3 API compatibility (works with AWS S3, MinIO, DigitalOcean Spaces, etc.)
2. Core operations:
   - List buckets
   - Create/delete bucket
   - List objects (with prefix filtering, pagination)
   - Put object (upload file or string)
   - Get object (download to file or string)
   - Delete object
   - Copy object
   - Head object (metadata only)

3. Advanced features:
   - Multipart upload for large files (>5MB)
   - Presigned URLs for temporary access
   - Object metadata (content-type, custom headers)

CORE CLASSES:
- SIMPLE_S3: Facade with quick upload/download methods
- S3_CLIENT: Full S3 API client
- S3_BUCKET: Bucket operations
- S3_OBJECT: Object representation with metadata
- S3_CREDENTIALS: AWS credentials (access key, secret key, region)
- S3_SIGNER: AWS Signature V4 request signing

DEPENDENCIES: simple_http, simple_json, simple_encryption (for HMAC-SHA256 signing)

CONTRACTS (DBC Focus):
- Precondition: Valid bucket name (DNS-compliant)
- Precondition: Object key not empty
- Postcondition: Upload returns ETag for verification
- Invariant: Credentials never logged/exposed

AUTHENTICATION: AWS Signature Version 4 (mandatory)

TESTING: MinIO container for local testing, mock responses for unit tests
```

**Complexity:** Medium (HTTP-based, signature is the hard part)
**Dependencies:** simple_http, simple_json, simple_encryption
**Use Cases:** File storage, backups, static asset hosting, data lake

---

### 3. simple_postgres

**Gap Filled:** Only SQLite supported; PostgreSQL is the production standard

**Plan-Prompt for simple_code:**
```
Build simple_postgres - a PostgreSQL client for the simple_* ecosystem.

REQUIREMENTS:
1. PostgreSQL wire protocol (v3) implementation
2. Connection management:
   - Connect with host, port, database, user, password
   - SSL/TLS support
   - Connection pooling

3. Query execution:
   - Simple queries (text protocol)
   - Prepared statements (binary protocol)
   - Parameterized queries (prevent SQL injection)
   - Transaction support (BEGIN, COMMIT, ROLLBACK, SAVEPOINT)

4. Result handling:
   - Row-by-row iteration
   - Type conversion (PostgreSQL types to Eiffel)
   - NULL handling with detachable types

5. PostgreSQL-specific:
   - LISTEN/NOTIFY for pub-sub
   - COPY for bulk import/export
   - Array types
   - JSON/JSONB columns

CORE CLASSES:
- SIMPLE_POSTGRES: Facade with quick query methods
- PG_CONNECTION: Single database connection
- PG_POOL: Connection pool manager
- PG_QUERY: Query builder (similar to simple_sql)
- PG_RESULT: Query result set
- PG_ROW: Single result row
- PG_TRANSACTION: Transaction scope

DEPENDENCIES: simple_json (for JSONB), simple_encryption (for auth)

CONTRACTS (DBC Focus):
- Precondition: Connection established before query
- Precondition: Parameter count matches placeholders
- Postcondition: Transaction state consistent
- Invariant: Connection pool size within bounds

PATTERNS: Follow simple_sql API patterns for consistency

TESTING: PostgreSQL container for integration tests
```

**Complexity:** High (wire protocol implementation)
**Dependencies:** simple_json, simple_encryption
**Use Cases:** Production databases, enterprise applications, data-intensive apps

---

### 4. simple_template

**Gap Filled:** No template engine for dynamic content generation

**Plan-Prompt for simple_code:**
```
Build simple_template - a template engine for the simple_* ecosystem.

REQUIREMENTS:
1. Mustache-compatible syntax (logic-less templates):
   - Variable interpolation: {{name}}
   - Sections (conditionals): {{#show}}...{{/show}}
   - Inverted sections: {{^show}}...{{/show}}
   - Iteration: {{#items}}...{{/items}}
   - Comments: {{! comment }}
   - Partials: {{> partial_name}}

2. Extensions beyond Mustache:
   - Filters: {{name | uppercase}}
   - Default values: {{name | default:"Unknown"}}
   - HTML escaping (default) and raw output: {{{raw}}}

3. Template sources:
   - String templates
   - File templates
   - Template directory with partials

CORE CLASSES:
- SIMPLE_TEMPLATE: Facade with quick render methods
- TEMPLATE_ENGINE: Full template processor
- TEMPLATE_CONTEXT: Data context (key-value store, nested objects)
- TEMPLATE_PARSER: Parse template to AST
- TEMPLATE_RENDERER: Render AST with context
- TEMPLATE_FILTER: Built-in and custom filters

DEPENDENCIES: simple_file (for file templates), simple_json (for JSON context)

CONTRACTS (DBC Focus):
- Precondition: Template syntax valid (parse succeeds)
- Precondition: Required variables present in context
- Postcondition: Output contains no unresolved placeholders
- Invariant: Filters are pure functions

BUILT-IN FILTERS: uppercase, lowercase, trim, default, escape, json, date

TESTING: Mustache spec compliance tests, edge cases (empty, nested, recursive)
```

**Complexity:** Medium (parsing and rendering)
**Dependencies:** simple_file, simple_json
**Use Cases:** Email templates, HTML generation, config file generation, code generation

---

### 5. simple_cron

**Gap Filled:** No job scheduling capability

**Plan-Prompt for simple_code:**
```
Build simple_cron - a job scheduler for the simple_* ecosystem.

REQUIREMENTS:
1. Cron expression parsing (standard 5-field format):
   - minute (0-59)
   - hour (0-23)
   - day of month (1-31)
   - month (1-12)
   - day of week (0-7, 0 and 7 are Sunday)
   - Support: *, ranges (1-5), lists (1,3,5), steps (*/15)

2. Job management:
   - Register jobs with cron expression and agent
   - One-time delayed jobs ("run in 5 minutes")
   - Enable/disable jobs
   - Job history and last run time

3. Execution:
   - Background scheduler thread (SCOOP separate)
   - Job timeout support
   - Error handling and retry
   - Concurrent job limit

4. Persistence (optional):
   - Save job definitions to simple_sql
   - Survive application restart

CORE CLASSES:
- SIMPLE_CRON: Facade with quick scheduling methods
- CRON_SCHEDULER: Main scheduler (SCOOP separate)
- CRON_EXPRESSION: Parse and evaluate cron expressions
- CRON_JOB: Job definition (expression, agent, enabled)
- CRON_HISTORY: Job execution history
- CRON_PERSISTENCE: Optional SQLite storage

DEPENDENCIES: simple_sql (for persistence), simple_datetime

CONTRACTS (DBC Focus):
- Precondition: Valid cron expression syntax
- Precondition: Job agent not Void
- Postcondition: Job registered and scheduled
- Invariant: Scheduler running while jobs exist

PATTERNS: Observer pattern for job completion, SCOOP for background execution

TESTING: Time-mocking for deterministic tests, integration tests for real scheduling
```

**Complexity:** Medium (cron parsing + SCOOP scheduling)
**Dependencies:** simple_sql, simple_datetime
**Use Cases:** Scheduled reports, cleanup tasks, periodic syncs, reminders

---

### 6. simple_oauth

**Gap Filled:** No OAuth2/OpenID Connect support for third-party authentication

**Plan-Prompt for simple_code:**
```
Build simple_oauth - an OAuth2/OIDC client for the simple_* ecosystem.

REQUIREMENTS:
1. OAuth 2.0 flows (RFC 6749):
   - Authorization Code (web apps)
   - Authorization Code with PKCE (mobile/SPA)
   - Client Credentials (server-to-server)
   - Refresh Token handling

2. OpenID Connect support:
   - ID token validation (JWT)
   - UserInfo endpoint
   - Discovery document (/.well-known/openid-configuration)

3. Provider presets:
   - Google
   - Microsoft/Azure AD
   - GitHub
   - Generic OAuth2 provider

CORE CLASSES:
- SIMPLE_OAUTH: Facade with quick auth methods
- OAUTH_CLIENT: OAuth2 client for any provider
- OAUTH_TOKEN: Access token, refresh token, expiry
- OAUTH_PKCE: PKCE code verifier/challenge generator
- OIDC_CLIENT: OpenID Connect client
- OIDC_CLAIMS: ID token claims (sub, email, name, etc.)
- OAUTH_PROVIDER: Provider configuration (endpoints, client_id, etc.)

DEPENDENCIES: simple_http, simple_json, simple_jwt, simple_encryption

CONTRACTS (DBC Focus):
- Precondition: Client ID and secret configured
- Precondition: Redirect URI matches registered
- Postcondition: Token response contains access_token
- Invariant: Tokens stored securely, never logged

SECURITY: State parameter for CSRF, PKCE for public clients, secure token storage

TESTING: Mock OAuth server, test against real providers in integration tests
```

**Complexity:** Medium (HTTP + JWT + security)
**Dependencies:** simple_http, simple_json, simple_jwt, simple_encryption
**Use Cases:** Social login, SSO, API authentication, third-party integrations

---

### 7. simple_graphql

**Gap Filled:** No GraphQL support (increasingly popular API pattern)

**Plan-Prompt for simple_code:**
```
Build simple_graphql - a GraphQL client and server for the simple_* ecosystem.

REQUIREMENTS:
1. GraphQL Client:
   - Execute queries and mutations
   - Variable substitution
   - Fragment support
   - Error handling (GraphQL errors vs network errors)

2. GraphQL Server:
   - Schema definition (SDL or programmatic)
   - Query execution engine
   - Resolver registration
   - Introspection support (__schema, __type)

3. Type system:
   - Scalar types (Int, Float, String, Boolean, ID)
   - Custom scalars (Date, DateTime, JSON)
   - Object types, Input types, Enums
   - Lists and Non-null modifiers

CORE CLASSES:
- SIMPLE_GRAPHQL: Facade for client operations
- GRAPHQL_CLIENT: HTTP client for GraphQL endpoints
- GRAPHQL_SCHEMA: Schema definition and validation
- GRAPHQL_RESOLVER: Resolver function registration
- GRAPHQL_QUERY: Parsed query representation
- GRAPHQL_EXECUTOR: Query execution engine
- GRAPHQL_RESULT: Query result with data and errors

DEPENDENCIES: simple_http (client), simple_web (server), simple_json

CONTRACTS (DBC Focus):
- Precondition: Query syntax valid (parse succeeds)
- Precondition: Query matches schema (validation)
- Postcondition: Result shape matches query
- Invariant: Resolvers registered for all fields

PATTERNS: DataLoader pattern for N+1 prevention, middleware for auth/logging

TESTING: GraphQL spec compliance, introspection tests, resolver unit tests
```

**Complexity:** High (parser, type system, executor)
**Dependencies:** simple_http, simple_web, simple_json
**Use Cases:** Modern APIs, mobile backends, aggregation layer, BFF pattern

---

### 8. simple_openapi

**Gap Filled:** No API documentation generation

**Plan-Prompt for simple_code:**
```
Build simple_openapi - OpenAPI specification generator for the simple_* ecosystem.

REQUIREMENTS:
1. OpenAPI 3.0 spec generation:
   - Auto-generate from simple_web routes
   - Manual schema definition support
   - JSON and YAML output

2. Route introspection:
   - Extract path, method, parameters from simple_web
   - Infer request/response schemas
   - Support path parameters, query params, headers

3. Documentation features:
   - Operation descriptions and tags
   - Request/response examples
   - Authentication schemes
   - Server definitions

4. Validation:
   - Validate requests against spec
   - Validate responses against spec

CORE CLASSES:
- SIMPLE_OPENAPI: Facade with quick spec generation
- OPENAPI_SPEC: Full OpenAPI 3.0 document
- OPENAPI_PATH: Path item with operations
- OPENAPI_OPERATION: Single operation (GET, POST, etc.)
- OPENAPI_SCHEMA: JSON Schema for types
- OPENAPI_GENERATOR: Generate spec from simple_web routes
- OPENAPI_VALIDATOR: Validate requests/responses

DEPENDENCIES: simple_web, simple_json, simple_yaml

CONTRACTS (DBC Focus):
- Precondition: Routes registered before generation
- Postcondition: Generated spec is valid OpenAPI 3.0
- Invariant: Spec and routes stay synchronized

OUTPUT: spec.json, spec.yaml, Swagger UI HTML

TESTING: OpenAPI spec validation, round-trip tests, Swagger UI rendering
```

**Complexity:** Medium (introspection + JSON Schema)
**Dependencies:** simple_web, simple_json, simple_yaml
**Use Cases:** API documentation, client generation, contract testing, API gateways

---

### 9. simple_xlsx

**Gap Filled:** No spreadsheet/Excel support

**Plan-Prompt for simple_code:**
```
Build simple_xlsx - Excel spreadsheet library for the simple_* ecosystem.

REQUIREMENTS:
1. XLSX reading:
   - Open .xlsx files (Office Open XML format)
   - Read sheets, rows, cells
   - Support cell types: string, number, date, boolean, formula (value only)
   - Handle merged cells
   - Read cell formatting (basic: bold, color)

2. XLSX writing:
   - Create new workbooks
   - Add/remove sheets
   - Write cells with types
   - Basic formatting (bold, colors, column width)
   - Save to file

3. CSV interop:
   - Import CSV to sheet
   - Export sheet to CSV

CORE CLASSES:
- SIMPLE_XLSX: Facade with quick read/write methods
- XLSX_WORKBOOK: Excel workbook (multiple sheets)
- XLSX_SHEET: Single worksheet
- XLSX_ROW: Row of cells
- XLSX_CELL: Single cell with value and format
- XLSX_STYLE: Cell formatting (font, color, border)
- XLSX_READER: Parse .xlsx files (ZIP + XML)
- XLSX_WRITER: Generate .xlsx files

DEPENDENCIES: simple_file, simple_xml, simple_archive (for ZIP), simple_datetime

CONTRACTS (DBC Focus):
- Precondition: File exists and is valid .xlsx
- Precondition: Sheet index/name exists
- Postcondition: Written file is valid .xlsx (can reopen)
- Invariant: Cell coordinates within sheet bounds

FORMAT: .xlsx is ZIP containing XML files (follow Office Open XML spec)

TESTING: Read/write round-trip, compatibility with Excel/LibreOffice
```

**Complexity:** Medium-High (ZIP + XML parsing)
**Dependencies:** simple_file, simple_xml, simple_archive, simple_datetime
**Use Cases:** Report generation, data import/export, ETL, business automation

---

### 10. simple_mock

**Gap Filled:** No HTTP mock server for testing

**Plan-Prompt for simple_code:**
```
Build simple_mock - HTTP mock server for testing in the simple_* ecosystem.

REQUIREMENTS:
1. Mock server:
   - Start/stop mock HTTP server on configurable port
   - Register mock responses for URL patterns
   - Support all HTTP methods
   - Response delays for timeout testing

2. Request matching:
   - Exact URL match
   - URL pattern with wildcards
   - Method matching
   - Header matching
   - Body matching (exact, contains, JSON path)

3. Response definition:
   - Status code
   - Headers
   - Body (string, JSON, file)
   - Delays

4. Verification:
   - Assert request was made
   - Assert request count
   - Assert request body/headers
   - Get all received requests

CORE CLASSES:
- SIMPLE_MOCK: Facade with quick mock setup
- MOCK_SERVER: HTTP server for testing
- MOCK_EXPECTATION: Expected request + response pair
- MOCK_REQUEST: Received request details
- MOCK_RESPONSE: Response to return
- MOCK_MATCHER: URL/header/body matching rules
- MOCK_VERIFIER: Assertion helpers

DEPENDENCIES: simple_web (HTTP server), simple_json

CONTRACTS (DBC Focus):
- Precondition: Server started before registering mocks
- Precondition: At least one expectation registered
- Postcondition: Verification reports unmatched requests
- Invariant: Server port not in use by another process

PATTERNS: Builder pattern for expectations, fluent assertions

TESTING: Self-testing (mock server tests), integration with simple_http client
```

**Complexity:** Low-Medium (wraps simple_web)
**Dependencies:** simple_web, simple_json
**Use Cases:** Unit testing HTTP clients, integration testing, CI/CD pipelines

---

## PART B: 6 Remaining Projects from Previous Research

These projects were identified in SIMPLE_CHART_RESEARCH.md but not yet built.

---

### 11. simple_qr

**Gap Filled:** QR code and barcode generation/scanning

**Plan-Prompt for simple_code:**
```
Build simple_qr - QR code and barcode library for the simple_* ecosystem.

REQUIREMENTS:
1. QR Code generation:
   - Generate QR codes from text/URLs
   - Multiple error correction levels (L, M, Q, H)
   - Output: ASCII art, PNG image, SVG
   - Configurable size and colors

2. Barcode generation:
   - Code 128, Code 39, EAN-13, UPC-A
   - Output: ASCII, PNG, SVG

3. QR Code reading (optional, Phase 2):
   - Decode QR from image file
   - Camera capture (if simple_camera exists)

CORE CLASSES:
- SIMPLE_QR: Facade with quick generation methods
- QR_ENCODER: QR code matrix generation
- QR_RENDERER: Render to ASCII/PNG/SVG
- BARCODE_ENCODER: Linear barcode generation
- BARCODE_RENDERER: Render barcodes

DEPENDENCIES: simple_file, simple_console (for ASCII), simple_stb or simple_cairo (for PNG)

CONTRACTS (DBC Focus):
- Precondition: Input data fits in QR capacity
- Precondition: Valid barcode format for data type
- Postcondition: Output is scannable (valid encoding)
- Invariant: Error correction level applied correctly

CLI USAGE:
  echo "https://example.com" | simple_qr generate --format=png -o qr.png
  simple_qr barcode --type=ean13 "5901234123457" -o barcode.png

TESTING: Round-trip encode/decode, scanner app verification
```

**Complexity:** Medium
**Dependencies:** simple_file, simple_console, simple_stb/simple_cairo
**Use Cases:** Inventory, tickets, payments, contact sharing, URLs

---

### 12. simple_trippy

**Gap Filled:** Network diagnostic tools (ping, traceroute, DNS lookup)

**Plan-Prompt for simple_code:**
```
Build simple_trippy - network diagnostic CLI for the simple_* ecosystem.

REQUIREMENTS:
1. Ping (ICMP Echo):
   - Ping host by name or IP
   - Configurable count, interval, timeout
   - Statistics: min/max/avg latency, packet loss

2. Traceroute:
   - Trace route to destination
   - Show each hop with latency
   - Resolve hostnames (optional)
   - UDP and ICMP modes

3. DNS Lookup:
   - Resolve hostname to IP (A, AAAA records)
   - Reverse DNS (PTR)
   - Query specific record types (MX, TXT, NS)
   - Custom DNS server support

4. Port scanning (basic):
   - Check if port is open
   - Scan port range

CORE CLASSES:
- SIMPLE_TRIPPY: Facade with quick diagnostic methods
- ICMP_PING: ICMP echo request/reply
- TRACEROUTE: Trace packet path
- DNS_RESOLVER: DNS query client
- PORT_SCANNER: TCP port checker

DEPENDENCIES: simple_console (for output formatting), simple_process (raw sockets may need elevation)

CONTRACTS (DBC Focus):
- Precondition: Valid hostname or IP address
- Precondition: Sufficient privileges for ICMP
- Postcondition: Results include timing information
- Invariant: Timeout respected

NOTES: ICMP requires raw sockets (admin/root on most systems)

CLI USAGE:
  simple_trippy ping google.com -c 5
  simple_trippy trace 8.8.8.8
  simple_trippy dns example.com --type=MX

TESTING: Local loopback tests, integration against known hosts
```

**Complexity:** High (raw sockets, ICMP protocol)
**Dependencies:** simple_console, simple_process
**Use Cases:** Network debugging, connectivity testing, DNS troubleshooting

---

### 13. simple_secrets

**Gap Filled:** Secret detection and security scanning

**Plan-Prompt for simple_code:**
```
Build simple_secrets - secret detection scanner for the simple_* ecosystem.

REQUIREMENTS:
1. Secret patterns:
   - AWS keys (AKIA...)
   - API tokens (various formats)
   - Private keys (RSA, SSH)
   - Passwords in config files
   - Database connection strings
   - JWT tokens
   - Custom regex patterns

2. Scanning:
   - Scan files and directories
   - Git history scanning (find secrets in old commits)
   - Exclude patterns (.gitignore style)
   - Configurable rule sets

3. Reporting:
   - Show file, line, matched pattern
   - Severity levels (high, medium, low)
   - Output formats: text, JSON, SARIF

CORE CLASSES:
- SIMPLE_SECRETS: Facade with quick scan methods
- SECRET_SCANNER: Main scanner engine
- SECRET_PATTERN: Regex pattern with metadata
- SECRET_FINDING: Detected secret location
- SECRET_RULESET: Collection of patterns
- SECRET_REPORTER: Format and output findings

DEPENDENCIES: simple_file, simple_regex, simple_json, simple_process (for git)

CONTRACTS (DBC Focus):
- Precondition: Path exists and is readable
- Postcondition: All files in scope scanned
- Postcondition: Findings include location info
- Invariant: Actual secret values redacted in output

BUILT-IN RULES: AWS, Azure, GCP, GitHub, Slack, Stripe, generic patterns

CLI USAGE:
  simple_secrets scan ./src
  simple_secrets scan . --git-history
  simple_secrets scan . --format=sarif -o results.sarif

TESTING: Test files with planted secrets, false positive testing
```

**Complexity:** Medium
**Dependencies:** simple_file, simple_regex, simple_json, simple_process
**Use Cases:** CI/CD security gates, pre-commit hooks, security audits

---

### 14. simple_bench

**Gap Filled:** Command and code benchmarking

**Plan-Prompt for simple_code:**
```
Build simple_bench - benchmarking tool for the simple_* ecosystem.

REQUIREMENTS:
1. Command benchmarking:
   - Run command multiple times
   - Measure wall time, user time, system time
   - Statistical analysis: mean, median, stddev, min, max
   - Warmup runs (discarded)

2. Comparison mode:
   - Benchmark multiple commands
   - Show relative performance
   - Statistical significance testing

3. Code benchmarking:
   - Benchmark Eiffel agents
   - Micro-benchmark support
   - Memory usage tracking (if available)

4. Output:
   - Table format
   - JSON export
   - Markdown table

CORE CLASSES:
- SIMPLE_BENCH: Facade with quick benchmark methods
- BENCHMARK_RUNNER: Execute benchmarks
- BENCHMARK_RESULT: Timing statistics
- BENCHMARK_COMPARISON: Compare multiple results
- BENCHMARK_REPORTER: Format output

DEPENDENCIES: simple_process (for commands), simple_json, simple_math (for statistics)

CONTRACTS (DBC Focus):
- Precondition: Run count >= 1
- Precondition: Command or agent provided
- Postcondition: Results include all statistical measures
- Invariant: Warmup runs not included in statistics

CLI USAGE:
  simple_bench "sleep 0.1" --runs=10
  simple_bench compare "cmd1" "cmd2" "cmd3" --runs=100
  simple_bench --export=json -o results.json

TESTING: Known-duration commands, statistical validation
```

**Complexity:** Low-Medium
**Dependencies:** simple_process, simple_json, simple_math
**Use Cases:** Performance testing, optimization validation, CI regression detection

---

### 15. simple_diff

**Gap Filled:** Diff and patch utilities

**Plan-Prompt for simple_code:**
```
Build simple_diff - diff and patch library for the simple_* ecosystem.

REQUIREMENTS:
1. Text diff:
   - Line-by-line diff (unified format)
   - Word-level diff
   - Character-level diff
   - Configurable context lines

2. Diff output:
   - Unified diff format
   - Side-by-side display
   - HTML diff with highlighting
   - JSON diff representation

3. Patch:
   - Apply unified diff to file
   - Reverse patch
   - Dry-run mode
   - Reject file for failed hunks

4. Directory diff:
   - Compare directories recursively
   - Show added/removed/modified files

CORE CLASSES:
- SIMPLE_DIFF: Facade with quick diff methods
- DIFF_ENGINE: Myers diff algorithm implementation
- DIFF_HUNK: Single change hunk
- DIFF_LINE: Line with status (add, remove, context)
- PATCH_APPLIER: Apply diffs to files
- DIFF_RENDERER: Format diff output

DEPENDENCIES: simple_file, simple_console (for colored output)

CONTRACTS (DBC Focus):
- Precondition: Input files exist and readable
- Postcondition: Diff can reconstruct target from source
- Postcondition: Patch application is reversible
- Invariant: Line numbers accurate

ALGORITHM: Myers diff algorithm (standard for git, diff)

CLI USAGE:
  simple_diff file1.txt file2.txt
  simple_diff --side-by-side old/ new/
  simple_diff patch -i changes.diff target.txt

TESTING: Known diff/patch pairs, round-trip verification
```

**Complexity:** Medium (Myers algorithm)
**Dependencies:** simple_file, simple_console
**Use Cases:** Version control, code review, config management, merge tools

---

### 16. simple_sync

**Gap Filled:** File synchronization with deduplication

**Plan-Prompt for simple_code:**
```
Build simple_sync - file sync and backup tool for the simple_* ecosystem.

REQUIREMENTS:
1. File synchronization:
   - Sync source to destination
   - Bidirectional sync option
   - Incremental sync (only changed files)
   - Delete files not in source (optional)

2. Change detection:
   - Timestamp comparison
   - Content hash comparison (SHA-256)
   - Size comparison

3. Deduplication:
   - Content-addressable storage
   - Hard links for identical files
   - Block-level dedup (optional, Phase 2)

4. Features:
   - Include/exclude patterns
   - Dry-run mode
   - Progress reporting
   - Conflict resolution strategies

CORE CLASSES:
- SIMPLE_SYNC: Facade with quick sync methods
- SYNC_ENGINE: Main synchronization logic
- SYNC_PLAN: Planned operations before execution
- SYNC_OPERATION: Single file operation (copy, delete, link)
- FILE_HASHER: Content hashing for comparison
- DEDUP_STORE: Content-addressable storage

DEPENDENCIES: simple_file, simple_hash, simple_json (for state)

CONTRACTS (DBC Focus):
- Precondition: Source path exists
- Precondition: Destination writable
- Postcondition: Destination mirrors source (based on strategy)
- Invariant: No data loss without explicit delete flag

PATTERNS: Rsync-like algorithm (send only differences)

CLI USAGE:
  simple_sync /source /dest
  simple_sync /source /dest --bidirectional
  simple_sync /source /dest --dry-run --delete

TESTING: Round-trip sync verification, conflict scenarios
```

**Complexity:** Medium-High
**Dependencies:** simple_file, simple_hash, simple_json
**Use Cases:** Backups, deployment, workspace sync, data migration

---

## Summary Matrix

| # | Library | Gap Filled | Complexity | Key Dependencies | Priority |
|---|---------|------------|------------|------------------|----------|
| 1 | simple_email | Email client | Medium-High | http, encryption | 07 |
| 2 | simple_s3 | Cloud storage | Medium | http, encryption | 14 |
| 3 | simple_postgres | PostgreSQL DB | High | json, encryption | **01** |
| 4 | simple_template | Template engine | Medium | file, json | 06 |
| 5 | simple_cron | Job scheduler | Medium | sql, datetime | 13 |
| 6 | simple_oauth | OAuth2/OIDC | Medium | http, jwt | 12 |
| 7 | simple_graphql | GraphQL API | High | http, web, json | **02** |
| 8 | simple_openapi | API docs | Medium | web, json, yaml | 15 |
| 9 | simple_xlsx | Excel files | Medium-High | file, xml, archive | **03** |
| 10 | simple_mock | Mock server | Low-Medium | web, json | 08 |
| 11 | simple_qr | QR/barcodes | Medium | file, console | **04** |
| 12 | simple_trippy | Network diag | High | console, process | 09 |
| 13 | simple_secrets | Secret scan | Medium | file, regex, json | 10 |
| 14 | simple_bench | Benchmarking | Low-Medium | process, json | 11 |
| 15 | simple_diff | Diff/patch | Medium | file, console | **05** |
| 16 | simple_sync | File sync | Medium-High | file, hash, json | 16 |

---

## Larry's Build Order (Priority)

| Priority | Library | Gap Filled | Complexity |
|----------|---------|------------|------------|
| **01** | simple_postgres | PostgreSQL database driver | High |
| **02** | simple_graphql | GraphQL client/server | High |
| **03** | simple_xlsx | Excel spreadsheet read/write | Medium-High |
| **04** | simple_qr | QR code/barcode generation | Medium |
| **05** | simple_diff | Diff and patch utilities | Medium |
| **06** | simple_template | Mustache-style templates | Medium |
| **07** | simple_email | Email client (SMTP/IMAP/POP3) | Medium-High |
| **08** | simple_mock | Mock HTTP server for testing | Low-Medium |
| **09** | simple_trippy | Network diagnostics | High |
| **10** | simple_secrets | Secret detection scanner | Medium |
| **11** | simple_bench | Command benchmarking | Low-Medium |
| **12** | simple_oauth | OAuth2/OIDC authentication | Medium |
| **13** | simple_cron | Job scheduler with cron syntax | Medium |
| **14** | simple_s3 | S3-compatible cloud storage | Medium |
| **15** | simple_openapi | OpenAPI spec generator | Medium |
| **16** | simple_sync | File sync with deduplication | Medium-High |

**Rationale:** Production database (postgres) and modern APIs (graphql) first, followed by business essentials (xlsx, qr, diff, template, email), then testing/diagnostics, and infrastructure last

---

## Usage with simple_code

Each plan-prompt above is designed to be given directly to simple_code:

```bash
simple_code new-session --name "build-simple-email"
simple_code set-plan "Build simple_email - an Eiffel email client..."
simple_code generate
```

The prompts include:
- Clear requirements
- Core class structure
- Dependencies to use
- DBC contract focus
- Testing approach

---

**Document Author:** Claude (AI Research Assistant)
**Methodology:** 7-Step Deep Research
**Ecosystem Baseline:** 78 simple_* libraries catalogued
**Output:** 16 gap-project specifications
