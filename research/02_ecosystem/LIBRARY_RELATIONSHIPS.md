# simple_* Library Ecosystem - Dependency Map

## Current State: Library Dependencies

```
                    ┌─────────────────────────────────────────────────────┐
                    │           APPLICATION LAYER (Consumers)              │
                    └─────────────────────────────────────────────────────┘
                              │              │              │
            ┌─────────────────┼──────────────┼──────────────┼──────────────┐
            ▼                 ▼              ▼              ▼              ▼
    ┌──────────────┐  ┌──────────────┐  ┌──────────┐  ┌───────────┐  ┌───────────┐
    │simple_showcase│  │simple_ai_client│  │simple_ci │  │simple_gui │  │    ...    │
    └──────┬───────┘  └──────┬───────┘  └────┬─────┘  │  _designer │  └───────────┘
           │                 │                │       └─────┬─────┘
           │                 │                │             │
    ┌──────┴─────────────────┴────────────────┴─────────────┴──────┐
    │                    WEB FRAMEWORK LAYER                        │
    └───────────────────────────────────────────────────────────────┘
                              │              │
            ┌─────────────────┴──────────────┴──────────────┐
            ▼                                                ▼
    ┌──────────────┐                                  ┌──────────────┐
    │ simple_web   │───┐                              │ simple_alpine│
    └──────┬───────┘   │                              └──────┬───────┘
           │           │                                     │
    ┌──────┴───────────┴─────────────────────────────────────┴──────┐
    │                    SERVICE LAYER                               │
    └───────────────────────────────────────────────────────────────┘
           │           │           │           │           │
    ┌──────┴───┐ ┌─────┴────┐ ┌────┴────┐ ┌────┴────┐ ┌────┴────────┐
    ▼          ▼ ▼          ▼ ▼         ▼ ▼         ▼ ▼              ▼
┌────────┐┌────────┐┌─────────┐┌─────────┐┌─────────┐┌──────────────┐
│simple_ ││simple_ ││ simple_ ││ simple_ ││ simple_ ││simple_rate_  │
│  jwt   ││ smtp   ││   sql   ││ cors    ││template ││   limiter    │
└───┬────┘└───┬────┘└────┬────┘└─────────┘└─────────┘└──────────────┘
    │         │          │
    │         │          │
    ┌─────────┴──────────┴─────────────────────────────────────────┐
    │                    FOUNDATION LAYER                           │
    └───────────────────────────────────────────────────────────────┘
    │         │         │          │          │          │          │
    ▼         ▼         ▼          ▼          ▼          ▼          ▼
┌────────┐┌────────┐┌─────────┐┌──────────┐┌──────────┐┌─────────┐┌──────────┐
│simple_ ││simple_ ││ simple_ ││ simple_  ││ simple_  ││simple_  ││ simple_  │
│ base64 ││ hash   ││  uuid   ││   json   ││ process  ││randomize││validation│
└────────┘└────────┘└─────────┘└──────────┘└──────────┘└─────────┘└──────────┘

Additional Foundation:
┌─────────┐┌─────────┐┌──────────┐┌──────────┐
│simple_  ││simple_  ││ simple_  ││ simple_  │
│  csv    ││ htmx    ││ markdown ││    ec    │
└─────────┘└─────────┘└──────────┘└──────────┘
```

## Detailed Dependency Graph

### Foundation Layer (No simple_* dependencies)
| Library           | External Deps      | Purpose                        |
|-------------------|-------------------|--------------------------------|
| simple_base64     | gobo_base         | Base64 encoding/decoding       |
| simple_hash       | gobo_base         | SHA-256/512, MD5, HMAC         |
| simple_uuid       | gobo_base         | UUID v4 generation             |
| simple_json       | gobo_base         | JSON parsing/generation        |
| simple_process    | base              | Process execution              |
| simple_randomizer | base              | Random number generation       |
| simple_csv        | base              | CSV parsing/writing            |
| simple_htmx       | base              | HTMX HTML generation           |
| simple_template   | base              | Mustache templates             |
| simple_cors       | base              | CORS header management         |
| simple_rate_limiter| base             | Rate limiting algorithms       |
| simple_markdown   | base, gobo_regexp | Markdown → HTML                |
| simple_validation | base, gobo_regexp | Data validation                |
| simple_ec         | base              | EiffelStudio compiler wrapper  |

### Service Layer
| Library        | Depends On                              | Purpose               |
|----------------|----------------------------------------|----------------------|
| simple_sql     | simple_json                            | SQL database access   |
| simple_jwt     | simple_base64, simple_hash, simple_uuid| JWT token handling    |
| simple_smtp    | simple_base64, simple_uuid             | SMTP email sending    |
| simple_alpine  | simple_htmx                            | Alpine.js integration |

### Web Framework Layer
| Library    | Depends On                                           | Purpose           |
|------------|-----------------------------------------------------|-------------------|
| simple_web | simple_process, simple_json, simple_randomizer, simple_sql | HTTP server      |

### Application Layer
| Library            | Depends On                                                    | Purpose              |
|--------------------|--------------------------------------------------------------|---------------------|
| simple_ci          | simple_json, simple_process                                  | CI/CD runner        |
| simple_ai_client   | simple_json, simple_process, simple_sql                      | AI API client       |
| simple_gui_designer| simple_json, simple_web, simple_htmx                         | GUI design tool     |
| simple_showcase    | simple_htmx, simple_alpine, simple_web, simple_json, simple_process, simple_sql | Demo website |

---

## Proposed: Better Integration

### Current Issues

1. **Missing Integrations**:
   - `simple_web` doesn't use `simple_template` for templating
   - `simple_web` doesn't use `simple_cors` for CORS handling
   - `simple_web` doesn't use `simple_rate_limiter` for rate limiting
   - `simple_web` doesn't use `simple_validation` for input validation
   - `simple_jwt` isn't integrated with `simple_web` for auth

2. **Duplication Potential**:
   - Multiple libraries may have their own error handling patterns
   - Validation logic may be scattered across libraries

3. **Missing Links**:
   - No WebSocket support yet (`simple_websocket` planned)
   - No OAuth2 support (could use `simple_jwt`)

### Proposed Enhanced Architecture

```
                    ┌─────────────────────────────────────────────────────┐
                    │           APPLICATION LAYER                          │
                    └─────────────────────────────────────────────────────┘
                                          │
                    ┌─────────────────────┴─────────────────────┐
                    │            SIMPLE_WEB (Enhanced)          │
                    │  ┌──────────────────────────────────────┐ │
                    │  │  Built-in support for:               │ │
                    │  │  • simple_template (view rendering)  │ │
                    │  │  • simple_cors (CORS middleware)     │ │
                    │  │  • simple_rate_limiter (throttling)  │ │
                    │  │  • simple_validation (input checks)  │ │
                    │  │  • simple_jwt (auth middleware)      │ │
                    │  │  • simple_websocket (real-time)      │ │
                    │  └──────────────────────────────────────┘ │
                    └─────────────────────────────────────────────┘
                              │              │              │
            ┌─────────────────┴──────────────┴──────────────┴──────┐
            │                    SERVICE LAYER                      │
            └───────────────────────────────────────────────────────┘
            │         │         │          │          │          │
    ┌───────┴──┐┌─────┴───┐┌────┴────┐┌────┴────┐┌────┴────┐┌────┴────┐
    │ simple_  ││ simple_ ││ simple_ ││ simple_ ││ simple_ ││ simple_ │
    │   jwt    ││  smtp   ││ cors    ││rate_lim ││template ││websocket│
    └───┬──────┘└───┬─────┘└─────────┘└─────────┘└─────────┘└────┬────┘
        │           │                                             │
        │           │                                             │
    ┌───┴───────────┴─────────────────────────────────────────────┴─┐
    │                    FOUNDATION LAYER                            │
    └───────────────────────────────────────────────────────────────┘
    │         │         │          │          │          │          │
    ▼         ▼         ▼          ▼          ▼          ▼          ▼
┌────────┐┌────────┐┌─────────┐┌──────────┐┌──────────┐┌─────────┐┌──────────┐
│simple_ ││simple_ ││ simple_ ││ simple_  ││ simple_  ││simple_  ││ simple_  │
│ base64 ││ hash   ││  uuid   ││   json   ││ process  ││validate ││ markdown │
└────────┘└────────┘└─────────┘└──────────┘└──────────┘└─────────┘└──────────┘
```

---

## Implementation Plan

### Phase 1: Complete Week 3 Libraries
1. ✅ `simple_validation` - Completed (49 tests passing)
2. ✅ `simple_websocket` - RFC 6455 WebSocket implementation (20 tests passing)

### Phase 2: Integration Updates (Future)

#### Update simple_web to integrate new libraries:

1. **Add simple_template support**
   - Add optional template engine middleware
   - Allow views to render via Mustache templates

2. **Add simple_cors support**
   - Create CORS middleware using simple_cors
   - Apply to routes automatically

3. **Add simple_rate_limiter support**
   - Create rate limiting middleware
   - Configurable per-route rate limits

4. **Add simple_validation support**
   - Request body validation middleware
   - Automatic error responses for invalid input

5. **Add simple_jwt support**
   - Authentication middleware
   - Token validation and extraction

6. **Add simple_websocket support**
   - WebSocket upgrade handler
   - Message routing

#### Create a "Meta Package"

Consider creating `simple_framework` that bundles common dependencies:
```eiffel
-- simple_framework.ecf would include:
-- simple_web, simple_template, simple_cors, simple_rate_limiter
-- simple_validation, simple_jwt, simple_websocket, simple_json
```

### Phase 3: Documentation

Update all library READMEs to show:
1. Which libraries they depend on
2. Which libraries can optionally use them
3. Example integration code

---

## Summary Stats

| Category           | Count |
|-------------------|-------|
| Foundation libs    | 14    |
| Service libs       | 4     |
| Framework libs     | 1     |
| Application libs   | 4     |
| **Total**         | **23** |

| With Dependencies | Without |
|-------------------|---------|
| 9                 | 14      |

---

## Phase 2 Update: API Layer Libraries ⭐

### Vision: Three Unified API Libraries

Create three unified API libraries that bundle their respective layers, providing clean interfaces for application development:

```
┌─────────────────────────────────────────────────────────────────┐
│ ★ simple_app_api                                                │
│   Bundles: simple_web, simple_alpine, simple_showcase,          │
│            simple_ci, simple_ai_client, simple_gui_designer     │
│   Purpose: Single import for building applications              │
└───────────────────────────────┬─────────────────────────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ ★ simple_service_api                                            │
│   Bundles: simple_jwt, simple_smtp, simple_sql, simple_cors,    │
│            simple_rate_limiter, simple_template, simple_websocket│
│   Purpose: Common services for web applications                 │
└───────────────────────────────┬─────────────────────────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│ ★ simple_foundation_api                                         │
│   Bundles: simple_base64, simple_hash, simple_uuid, simple_json,│
│            simple_process, simple_randomizer, simple_csv,       │
│            simple_htmx, simple_markdown, simple_validation      │
│   Purpose: Core utilities for any Eiffel project                │
└─────────────────────────────────────────────────────────────────┘
```

### Benefits of API Layer Approach

1. **Single Import**: Instead of 10+ library imports, use just `simple_foundation_api`
2. **Versioned Bundles**: API libraries can version their bundle composition
3. **Clean Architecture**: Clear layer separation with defined boundaries
4. **Easier Onboarding**: New developers import one library per layer
5. **Dependency Management**: API libraries handle internal dependencies

### Example Usage

```eiffel
-- Before: Many imports
library "simple_base64" location="..."
library "simple_hash" location="..."
library "simple_uuid" location="..."
library "simple_json" location="..."
-- ... 6 more imports

-- After: Single import
library "simple_foundation_api" location="$SIMPLE_FOUNDATION_API/simple_foundation_api.ecf"
```

### ECF Structure Example

```xml
<!-- simple_foundation_api.ecf -->
<system name="simple_foundation_api" uuid="..." library_target="simple_foundation_api">
    <target name="simple_foundation_api">
        <library name="simple_base64" location="$SIMPLE_BASE64/simple_base64.ecf"/>
        <library name="simple_hash" location="$SIMPLE_HASH/simple_hash.ecf"/>
        <library name="simple_uuid" location="$SIMPLE_UUID/simple_uuid.ecf"/>
        <library name="simple_json" location="$SIMPLE_JSON/simple_json.ecf"/>
        <library name="simple_validation" location="$SIMPLE_VALIDATION/simple_validation.ecf"/>
        <!-- ... additional foundation libraries -->
    </target>
</system>
```

This architectural enhancement creates a clean, maintainable structure for the growing simple_* ecosystem.
