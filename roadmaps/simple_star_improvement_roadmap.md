# simple_* Improvement Roadmap

**Date:** December 7, 2025
**Updated:** December 9, 2025
**Based on:** simple_ecosystem_assessment.md + oracle stats

---

## Current Ecosystem Status

| Metric | Value |
|--------|-------|
| Libraries | 53 |
| Classes | 2,178 |
| Features (API surface) | 63,342 |
| DBC: Preconditions | 31% |
| DBC: Postconditions | 49% |
| SCOOP Compliance | 100% (53/53) |
| Docs Compliance | 98% (52/53) |

---

## Phase 1: Foundation Hardening ✅ COMPLETE

### Step 1.1: Standardize ECF Capabilities ✅
All 53 libraries now have explicit capability declarations:

```xml
<capability>
    <concurrency support="scoop" use="thread"/>
    <void_safety support="all"/>
</capability>
```

### Step 1.2: Void-Safety ✅
All libraries are void-safe by default. This is the Eiffel baseline - not optional.

### Step 1.3: Documentation ✅
52/53 libraries have full docs (README.md, CHANGELOG.md, docs/index.html).
- simple_oracle pending (not yet on GitHub)

---

## Phase 2: SCOOP Support ✅ COMPLETE

All 53 libraries have SCOOP capability declarations.

**Runtime SCOOP Blocker:** simple_http and simple_web depend on ISE's EWF (Eiffel Web Framework), which is not SCOOP-compatible. See Phase 7 for resolution.

---

## Phase 3: Wrapper Libraries ✅ COMPLETE

All planned wrapper libraries have been created:

| Library | Status | Notes |
|---------|--------|-------|
| simple_http | ✅ Created | Currently wraps http_client (EWF dependency) |
| simple_encryption | ✅ Created | Wraps eel |
| simple_compression | ✅ Created | Wraps zlib |
| simple_config | ✅ Created | Unified configuration |
| simple_cli | ✅ Created | Argument parsing |
| simple_archive | ✅ Created | Archive operations |
| simple_i18n | ✅ Created | Internationalization |
| simple_mongo | ✅ Created | MongoDB client |

**Additional libraries created beyond original plan:**
- simple_ai_client, simple_alpine, simple_app_api, simple_ci
- simple_clipboard, simple_console, simple_env, simple_file
- simple_foundation_api, simple_gui_designer, simple_ipc
- simple_mmap, simple_oracle, simple_registry, simple_service_api
- simple_showcase, simple_system, simple_watcher, simple_win32_api

---

## Phase 4: Distribution ✅ COMPLETE

### Step 4.1: simple_setup Package Manager ✅
- CLI tool for installing individual libraries
- Environment variable management
- Library manifest with all 53 libraries

### Step 4.2: INNO Setup Installer ✅
- Windows installer generation
- Full ecosystem bundle install
- Available at: https://github.com/simple-eiffel/simple_setup

### Step 4.3: GitHub Organization ✅
- All libraries published to https://github.com/simple-eiffel
- GitHub Pages documentation enabled

---

## Phase 5: CI/CD Pipeline 🔲 NOT STARTED

### Step 5.1: GitHub Actions Workflow
```yaml
- Compile with void_safety="all"
- Compile with concurrency="scoop"
- Run all tests
- Validate docs
```

### Step 5.2: Quality Gates
- No library merges without:
  - Void-safe compilation passing
  - SCOOP compilation passing
  - 100% test pass rate

---

## Phase 6: Documentation Standards ✅ MOSTLY COMPLETE

### Step 6.1: Every Library Gets ✅
- README.md (standardized format) - 53/53
- docs/index.html - 52/53
- CHANGELOG.md - 52/53

**Missing:** simple_oracle (not yet on GitHub)

### Step 6.2: API Documentation
- Generated via EiffelStudio -short command
- Documentation folders created for most libraries

---

## Phase 7: EWF Replacement 🔲 NEW - HIGH PRIORITY

**Goal:** Untether simple_http and simple_web from ISE's EWF to enable true SCOOP compatibility.

### Current Problem
- simple_http depends on: `net_http_client`
- simple_web depends on: `wsf`, `httpd`, `standalone`, `encoder`, `wsf_all`
- EWF is not SCOOP-compatible, blocking concurrent web applications

### Solution
Rewrite simple_http and simple_web with our own implementation:
- HTTP server using inline C for Win32 sockets
- Request parsing and response writing
- Routing and middleware
- Full SCOOP compatibility
- No ISE contrib dependencies

### Benefits
- Full control over implementation
- SCOOP-native concurrency
- Matches ecosystem pattern (inline C, no external .obj files)
- Removes last ISE dependency outside base/time/testing

---

## Phase 8: Oracle Enhancement 🔲 NEW

### Planned Commands
- `sync` - Commit and push all repos with changes (replace sync_all_repos.sh)
- `docs` - Generate API documentation (replace generate_api_docs.sh)
- `status` period filtering (bug fix)

### Current Oracle Capabilities
- boot, query, log, compile, test, git, commits
- status, stats, scan, ingest, learn, handoff, check

---

## Execution Priority (Updated)

| Priority | Task | Status |
|----------|------|--------|
| ~~P0~~ | ~~ECF capability standardization~~ | ✅ Done |
| ~~P0~~ | ~~Void-safety audit~~ | ✅ Done (default) |
| ~~P1~~ | ~~simple_setup manifest~~ | ✅ Done |
| ~~P1~~ | ~~simple_http wrapper~~ | ✅ Done |
| ~~P2~~ | ~~SCOOP declarations~~ | ✅ Done |
| ~~P2~~ | ~~GitHub Pages docs~~ | ✅ Done |
| P1 | **EWF replacement** | 🔲 New priority |
| P2 | CI/CD pipeline | 🔲 Not started |
| P2 | Oracle enhancements | 🔲 Planned |
| P3 | simple_oracle GitHub + docs | 🔲 Pending |

---

## Library Inventory (53 Total)

All libraries have: ECF ✅ | Void-Safe ✅ | SCOOP ✅ | README ✅

| Library | CHANGELOG | docs/index.html | Notes |
|---------|-----------|-----------------|-------|
| simple_ai_client | ✅ | ✅ | |
| simple_alpine | ✅ | ✅ | |
| simple_app_api | ✅ | ✅ | |
| simple_archive | ✅ | ✅ | |
| simple_base64 | ✅ | ✅ | |
| simple_cache | ✅ | ✅ | |
| simple_ci | ✅ | ✅ | |
| simple_cli | ✅ | ✅ | |
| simple_clipboard | ✅ | ✅ | |
| simple_compression | ✅ | ✅ | |
| simple_config | ✅ | ✅ | |
| simple_console | ✅ | ✅ | |
| simple_cors | ✅ | ✅ | |
| simple_csv | ✅ | ✅ | |
| simple_datetime | ✅ | ✅ | |
| simple_encryption | ✅ | ✅ | |
| simple_env | ✅ | ✅ | |
| simple_file | ✅ | ✅ | |
| simple_foundation_api | ✅ | ✅ | |
| simple_gui_designer | ✅ | ✅ | |
| simple_hash | ✅ | ✅ | |
| simple_htmx | ✅ | ✅ | |
| simple_http | ✅ | ✅ | EWF dependency |
| simple_i18n | ✅ | ✅ | |
| simple_ipc | ✅ | ✅ | |
| simple_json | ✅ | ✅ | |
| simple_jwt | ✅ | ✅ | |
| simple_logger | ✅ | ✅ | |
| simple_markdown | ✅ | ✅ | |
| simple_mmap | ✅ | ✅ | |
| simple_mongo | ✅ | ✅ | |
| simple_oracle | ❌ | ❌ | Not on GitHub yet |
| simple_pdf | ✅ | ✅ | |
| simple_process | ✅ | ✅ | |
| simple_randomizer | ✅ | ✅ | |
| simple_rate_limiter | ✅ | ✅ | |
| simple_regex | ✅ | ✅ | |
| simple_registry | ✅ | ✅ | |
| simple_service_api | ✅ | ✅ | |
| simple_setup | ✅ | ✅ | |
| simple_showcase | ✅ | ✅ | |
| simple_smtp | ✅ | ✅ | |
| simple_sql | ✅ | ✅ | |
| simple_system | ✅ | ✅ | |
| simple_template | ✅ | ✅ | |
| simple_testing | ✅ | ✅ | |
| simple_uuid | ✅ | ✅ | |
| simple_validation | ✅ | ✅ | |
| simple_watcher | ✅ | ✅ | |
| simple_web | ✅ | ✅ | EWF dependency |
| simple_websocket | ✅ | ✅ | |
| simple_win32_api | ✅ | ✅ | |
| simple_xml | ✅ | ✅ | |

---

## Summary

**Completed:** Phases 1-4, 6 (98%)
**In Progress:** Phase 7 (EWF replacement), Phase 8 (Oracle)
**Not Started:** Phase 5 (CI/CD)

The ecosystem is production-ready for most use cases. The main remaining work is:
1. Replacing EWF dependency for true SCOOP web applications
2. Adding CI/CD for automated quality enforcement
3. Completing simple_oracle GitHub publication
