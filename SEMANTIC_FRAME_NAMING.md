# Semantic Frame Naming Pattern for Eiffel

## Overview

This document describes a powerful refactoring pattern unique to Eiffel: **Semantic Frame Naming**. This pattern leverages Eiffel's ability to give a single feature multiple names, allowing classes to speak the vocabulary of whatever domain context they're used in.

## The Problem

### Semantic Frame Bias

Every programmer (human or AI) operates within a **semantic frame** - a mental context that biases them toward certain vocabulary. When working in a banking context, the mind reaches for `account_holder`. In gaming, it reaches for `gamertag`. In social media, `handle`.

This creates a predictable pattern:
1. Developer writes code using domain-natural vocabulary
2. Compiler rejects - feature name doesn't match supplier class
3. Developer looks up the "real" name
4. Developer either:
   - Conforms to supplier's vocabulary (cognitive friction)
   - Renames in inheritance clause (inheritance pollution)
   - Creates wrapper methods (code bloat)

This cycle wastes time and creates unnecessary friction.

### AI Amplification

AI coding assistants exhibit the same behavior probabilistically. Given a context, they "guess" feature names based on semantic frame patterns. When wrong, the Eiffel compiler forces correction. This isn't hallucination per se - it's **semantic frame bias** producing reasonable-but-wrong names.

The insight: these "wrong" names are (probablistically or model-frame) **predictable**. Given a context, we may be quite capable of anticipating what names will be reached for (by AI or human).

## The Solution

### Eiffel's Multi-Name Feature Syntax

Eiffel allows multiple names for a single feature:

```eiffel
name,
account_holder,
account_title,
username,
display_name,
handle,
gamertag,
profile_name: STRING_32
    attribute
        -- Human-readable identifier for this account
    end
```

All names resolve to the same implementation. No code duplication. No runtime cost.

### Pre-Salting with Semantic Frames

Instead of fighting semantic frame bias, **encode it into the supplier class upfront**:

1. Identify the potential or reasonable semantic frames (usage contexts) the class will serve
2. For each feature, determine domain-appropriate names for each frame
3. Add those names as comma-delimited aliases
4. Document the semantic frames in class and feature notes

The class becomes **polyglot across semantic frames**.

_NOTE: This also applies to semantic framing changes across time due to cultural or industry semantic drift._

## Benefits

### For Human Developers
- Write code in domain-natural vocabulary
- Less context-switching between business domain and code domain
- Reduced lookup friction
- Increate discoverability for reuse
- Self-documenting semantic roles

### For AI Assistants
- "Guesses" based on context are more likely to be valid
- Reduced correction cycles
- Faster code generation
- Less compiler-driven backtracking

### For the Codebase
- No wrapper method bloat
- No inheritance rename pollution
- Single implementation, multiple vocabularies
- Non-breaking enhancement (additive only)

## Implementation Guidelines

### 1. Identify Semantic Frames

For each class, ask: "In what contexts will this class be used?"

Example for `SIMPLE_JSON_VALUE`:
- HTTP/API context: payloads, responses, request bodies
- Configuration context: settings, preferences, options
- Data storage context: records, entries, documents
- Logging context: events, messages, structured logs

### 2. Map Features to Frame Vocabulary

For each feature, determine natural vocabulary per frame:

| Generic | HTTP/API | Config | Storage | Logging |
|---------|----------|--------|---------|---------|
| `value` | `payload` | `setting` | `record` | `entry` |
| `items` | `elements` | `options` | `rows` | `events` |
| `get` | `fetch` | `read` | `retrieve` | `extract` |

### 3. Add Names (Non-Breaking)

```eiffel
value,
payload,
setting,
record,
entry: detachable ANY
    -- The contained data
```

**Critical**: Never remove existing names. Only add. This ensures backward compatibility.

### 4. Document with Notes

Use Eiffel's `note` clause to document semantic frames:

```eiffel
class ACCOUNT

note
    description: "Generic account abstraction"
    semantic_frames: "banking, user_auth, gaming, social, accounting"
    refactored_for_frames: "2025-12-11"

feature -- Identity

    name,
    account_holder,    -- banking frame
    username,          -- user_auth frame
    gamertag,          -- gaming frame
    handle,            -- social frame
    account_name: STRING_32
            -- Human-readable identifier
        note
            banking: "Legal name on account"
            user_auth: "Login identifier"
            gaming: "Player display name"
            social: "Public @handle"
            accounting: "Ledger account name"
        attribute
        end
```

## Inter-Library Semantic Analysis

### Using the Dependency Graph

The library dependency graph is a **semantic frame generator**. When library A uses library B, it creates a semantic context.

Ask: "What vocabulary will library A naturally use when calling library B?"

### Complete Simple Eiffel Dependency Matrix

This matrix is derived from actual ECF dependencies across the ecosystem. Each row represents a semantic frame opportunity.

#### Core Data Libraries (High Fan-In - Priority Refactor Targets)

| Supplier | Client Libraries | Semantic Frames |
|----------|------------------|-----------------|
| **simple_json** | simple_ai_client | AI prompts, model responses, chat messages |
| | simple_ci | Build configs, pipeline definitions, status reports |
| | simple_codec | Serialization format, interchange data |
| | simple_config | Settings, preferences, configuration values |
| | simple_foundation_api | API contracts, domain objects, DTOs |
| | simple_gui_designer | UI definitions, component trees, layouts |
| | simple_http | HTTP payloads, request/response bodies |
| | simple_logger | Structured log entries, event records |
| | simple_lsp | LSP protocol messages, diagnostics, completions |
| | simple_oracle | Knowledge records, event logs, handoff data |
| | simple_setup | Installer config, package manifests |
| | simple_showcase | Demo data, example payloads |
| | simple_sql | Query results, row data, schema definitions |
| | simple_web | API payloads, REST responses, request bodies |
| **simple_file** | simple_eiffel_parser | Source files, parse targets |
| | simple_lsp | Project files, workspace documents |
| | simple_oracle | Knowledge base files, log files |
| | simple_ucf | Config files, universe definitions |
| **simple_sql** | simple_ai_client | Conversation history, model cache |
| | simple_lsp | Symbol database, index storage |
| | simple_oracle | Knowledge persistence, event store |
| | simple_service_api | User data, session storage, API persistence |
| | simple_showcase | Demo database, sample data |
| | simple_web | API data layer, user sessions |
| **simple_process** | simple_ai_client | Model subprocess calls, API bridges |
| | simple_ci | Build commands, test runners, deploy scripts |
| | simple_foundation_api | External tool integration |
| | simple_lsp | Compiler invocation, external tools |
| | simple_oracle | Git commands, file watchers |
| | simple_pdf | PDF generation subprocess |
| | simple_setup | Installer commands, system configuration |
| | simple_showcase | Demo process execution |
| | simple_web | External API calls, subprocess bridges |
| | simple_win32_api | System process management |

#### Infrastructure Libraries

| Supplier | Client Libraries | Semantic Frames |
|----------|------------------|-----------------|
| **simple_base64** | simple_compression | Encoded compressed data |
| | simple_encryption | Encoded ciphertext, key encoding |
| | simple_foundation_api | Data encoding, transport format |
| | simple_http | HTTP basic auth, binary payloads |
| | simple_pdf | Embedded resources, binary content |
| **simple_datetime** | simple_foundation_api | Timestamps, scheduling, durations |
| | simple_oracle | Event times, session timestamps |
| | simple_process | Execution timing, timeouts |
| **simple_env** | simple_setup | Installer environment, system paths |
| | simple_ucf | Environment-based config paths |
| | simple_win32_api | System environment access |
| **simple_regex** | simple_eiffel_parser | Token patterns, lexer rules |
| | simple_foundation_api | Validation patterns, parsing |
| | simple_yaml | YAML parsing patterns |
| **simple_toml** | simple_codec | TOML format handling |
| | simple_lsp | LSP config files, UCF format |
| | simple_oracle | Config file parsing |
| | simple_ucf | Universe config format |
| **simple_xml** | simple_codec | XML format handling |
| | simple_foundation_api | XML data interchange |
| | simple_lsp | XML-based configs |
| | simple_ucf | XML config format |
| **simple_yaml** | simple_codec | YAML format handling |
| **simple_logger** | simple_foundation_api | Application logging |
| | simple_oracle | Event logging, activity tracking |
| **simple_hash** | simple_foundation_api | Data integrity, checksums |

#### UI/Web Libraries

| Supplier | Client Libraries | Semantic Frames |
|----------|------------------|-----------------|
| **simple_htmx** | simple_alpine | Interactive UI components |
| | simple_foundation_api | Dynamic HTML generation |
| | simple_gui_designer | UI component library |
| | simple_showcase | Demo UI components |
| **simple_alpine** | simple_app_api | Reactive UI layer |
| | simple_showcase | Demo reactive components |
| **simple_web** | simple_app_api | HTTP server, API endpoints |
| | simple_gui_designer | Web-based designer UI |
| | simple_showcase | Demo web application |
| **simple_template** | simple_service_api | Email templates, report templates |
| | simple_setup | Installer templates, config templates |

#### API Layer Libraries

| Supplier | Client Libraries | Semantic Frames |
|----------|------------------|-----------------|
| **simple_foundation_api** | simple_jwt | Token generation, claims |
| | simple_service_api | Core API patterns |
| | simple_smtp | Email API integration |
| | simple_websocket | WebSocket API layer |
| **simple_service_api** | simple_app_api | Service layer patterns |
| **simple_validation** | simple_foundation_api | Input validation, constraints |

#### System Libraries

| Supplier | Client Libraries | Semantic Frames |
|----------|------------------|-----------------|
| **simple_console** | simple_setup | Installer console UI |
| | simple_win32_api | Console management |
| **simple_clipboard** | simple_win32_api | Clipboard operations |
| **simple_ipc** | simple_win32_api | Inter-process communication |
| **simple_mmap** | simple_win32_api | Memory-mapped files |
| **simple_registry** | simple_win32_api | Windows registry access |
| **simple_system** | simple_win32_api | System information |
| **simple_watcher** | simple_oracle | File system monitoring |
| | simple_win32_api | Directory watching |

#### Specialized Libraries

| Supplier | Client Libraries | Semantic Frames |
|----------|------------------|-----------------|
| **simple_eiffel_parser** | simple_lsp | Source parsing, AST |
| | simple_oracle | Code analysis, metrics |
| **simple_ucf** | simple_lsp | Universe configuration |
| | simple_oracle | Project configuration |
| **simple_cache** | simple_service_api | Response caching, data caching |
| **simple_cors** | simple_service_api | Cross-origin policies |
| **simple_csv** | simple_foundation_api | Tabular data, imports/exports |
| **simple_jwt** | simple_service_api | Authentication tokens |
| **simple_markdown** | simple_foundation_api | Documentation, rich text |
| **simple_pdf** | simple_service_api | Report generation |
| **simple_randomizer** | simple_foundation_api | Random data, test data |
| | simple_web | Session tokens, unique IDs |
| **simple_rate_limiter** | simple_service_api | API throttling |
| **simple_smtp** | simple_service_api | Email sending |
| **simple_uuid** | simple_foundation_api | Unique identifiers |
| **simple_websocket** | simple_service_api | Real-time communication |

### Priority Refactoring Order

Based on fan-in (number of clients), refactor suppliers in this order:

1. **simple_json** (14 clients) - Highest impact
2. **simple_process** (10 clients)
3. **simple_file** (4 clients)
4. **simple_sql** (6 clients)
5. **simple_base64** (5 clients)
6. **simple_foundation_api** (5 clients)
7. **simple_htmx** (4 clients)
8. **simple_env** (3 clients)
9. **simple_toml** (4 clients)
10. Remaining libraries by usage

### The Refactoring Sequence

**Phase 1: Enrich Suppliers (Non-Breaking)**
1. Analyze each supplier library
2. Identify all client libraries that use it
3. Determine semantic frames from those relationships
4. Add frame-appropriate feature names
5. Document with notes

**Phase 2: Refactor Clients (Optional)**
1. For each client library
2. Identify calls to enriched suppliers
3. Switch to frame-appropriate vocabulary
4. Code becomes more self-documenting

Phase 2 is optional because Phase 1 is already valuable - future code naturally uses appropriate names.

## Example: Refactoring SIMPLE_FILE for Multiple Frames

### Current State
```eiffel
class SIMPLE_FILE

feature -- Access
    path: STRING_32

feature -- Content
    read_text: STRING_32
    write_text (content: STRING_32)
```

### After Semantic Frame Analysis

Clients: simple_config, simple_logger, simple_archive, simple_backup, simple_template

```eiffel
class SIMPLE_FILE

note
    semantic_frames: "config, logging, archive, backup, template"

feature -- Access

    path,
    config_path,      -- simple_config frame
    log_path,         -- simple_logger frame
    archive_path,     -- simple_archive frame
    backup_path,      -- simple_backup frame
    template_path: STRING_32
        note
            config: "Path to configuration file"
            logging: "Path to log file"
            archive: "Path to archive member or source"
            backup: "Path to backup source or target"
            template: "Path to template file"
        attribute
        end

feature -- Content

    read_text,
    load_config,       -- simple_config frame
    read_log,          -- simple_logger frame
    extract_content,   -- simple_archive frame
    restore_content,   -- simple_backup frame
    load_template: STRING_32
        note
            config: "Load configuration content"
            logging: "Read log file content"
            archive: "Extract archived content"
            backup: "Restore backed up content"
            template: "Load template for processing"
        do
            -- single implementation
        end

    write_text,
    save_config,       -- simple_config frame
    append_log,        -- simple_logger frame (note: might need separate impl)
    archive_content,   -- simple_archive frame
    backup_content,    -- simple_backup frame
    write_template (content: STRING_32)
        note
            config: "Save configuration content"
            logging: "Write to log file"
            archive: "Store content in archive"
            backup: "Save backup content"
            template: "Write processed template"
        do
            -- single implementation
        end
```

## Inheritance and Semantic Frames

### The Principle: Names Flow Toward Specificity

Inheritance represents a **semantic frame shift** - the descendant class operates in a more specific context than its ancestor. This has clear implications for where multi-name features belong:

| Class Level | Semantic Scope | Name Style |
|-------------|----------------|------------|
| **Ancestor** | Generic, abstract | Domain-neutral: `name`, `value`, `identifier` |
| **Descendant** | Specific, concrete | Domain-specific: `account_holder`, `gamertag`, `handle` |

**Generic ancestors stay generic. Specialized descendants add specialized vocabulary.**

### Single Inheritance

When `BANK_ACCOUNT` inherits from `ACCOUNT`:

```eiffel
class ACCOUNT
feature
    name: STRING_32   -- Generic: stays here
```

```eiffel
class BANK_ACCOUNT
inherit
    ACCOUNT
feature
    account_holder: STRING_32
            -- Banking frame name for inherited `name`
        do
            Result := name
        end
```

Or, if the ancestor is being designed with semantic frames in mind:

```eiffel
class ACCOUNT
feature
    name,
    account_holder,    -- Available for banking descendants
    username,          -- Available for auth descendants
    gamertag: STRING_32
        attribute end
```

The descendant then simply uses the appropriate name without wrapping.

### Multiple Inheritance: Frame Convergence

Multiple inheritance is particularly interesting - the descendant class is literally **combining semantic frames**. It becomes the convergence point where those frames meet.

```eiffel
class AUTHENTICATED_BANK_CUSTOMER
inherit
    BANK_ACCOUNT      -- Banking frame
    USER_IDENTITY     -- Auth frame
    AUDIT_SUBJECT     -- Compliance frame
```

This descendant naturally needs vocabulary from all three frames. It's the right place to add bridging names that unify concepts across the inherited frames.

### Where to Add Names: Decision Guide

| Scenario | Add Names To |
|----------|--------------|
| Generic library class used in many contexts | **Supplier class** (pre-salt for common frames) |
| Abstract ancestor with known specializations | **Ancestor** (if frames are predictable) |
| Specialized descendant in specific domain | **Descendant** (domain-specific vocabulary) |
| Multiple inheritance convergence | **Descendant** (bridging vocabulary) |
| One dominant usage pattern across ecosystem | **Supplier** (optimize for common case) |

### Avoiding Pollution

**Don't** add domain-specific names to generic ancestors that shouldn't know about those domains:

```eiffel
-- BAD: ACCOUNT shouldn't know about gaming
class ACCOUNT
feature
    name,
    gamertag,           -- Why does ACCOUNT know about gaming?
    battle_net_id,      -- This is domain pollution
    steam_username: STRING_32
```

**Do** let descendants introduce their domain vocabulary:

```eiffel
-- GOOD: GAME_PROFILE knows about gaming
class GAME_PROFILE
inherit
    ACCOUNT
feature
    gamertag,
    battle_net_id,
    steam_username: STRING_32
        do Result := name end
```

### Exception: High Fan-In Suppliers

When a class like `SIMPLE_JSON_VALUE` is used by 14+ client libraries across diverse semantic frames, pre-salting at the supplier level makes pragmatic sense:

- The class is already "aware" of multiple contexts through its widespread use
- Adding frame vocabulary reduces friction across the entire ecosystem
- The alternative (each client wrapping or renaming) creates more pollution

This is the case for our priority refactor targets: `simple_json`, `simple_process`, `simple_sql`, etc.

### Genericity and Semantic Range

Generic parameters act as a **semantic lens** - they focus an otherwise unbounded class into a specific domain.

| Expression | Semantic Range |
|------------|----------------|
| `ARRAYED_LIST [G]` | Unbounded - any list of anything |
| `ARRAYED_LIST [ACCOUNT]` | Lists within ACCOUNT's semantic scope |
| `ARRAYED_LIST [BANK_ACCOUNT]` | Lists within banking domain specifically |

**The principle**: Feature names must respect the semantic range of the enfolding class.

- Classes at the bottom of hierarchies (most specific) have the greatest semantic reach - they can extend vocabulary into their specific domain
- Ancestors bump into the semantic ranges of their descendants - they must stay neutral
- Generic parameters constrain or expand this range based on their bounds

#### Unconstrained Generics: Stay Maximally Generic

An unconstrained generic class knows nothing about what it holds:

```eiffel
class CONTAINER [G]
feature
    item: G           -- Generic: correct
    value: G          -- Generic: correct
    account: G        -- Domain-specific: WRONG (G could be anything)
```

The container should use vocabulary that makes sense regardless of what `G` turns out to be.

#### Constrained Generics: Vocabulary Reflects the Constraint

A constrained generic *knows* something about its parameter:

```eiffel
class ACCOUNT_REGISTRY [G -> ACCOUNT]
feature
    accounts,
    entries,
    members: ARRAYED_LIST [G]
        -- All valid: G is constrained to ACCOUNT's semantic scope
        attribute end

    primary_account,
    default_entry,
    active_member: detachable G
        -- Domain vocabulary is appropriate here
        attribute end
```

The constraint `G -> ACCOUNT` pulls the host class's semantic range into ACCOUNT's domain. Feature names can legitimately use ACCOUNT-related vocabulary.

#### Multiple Constraints

With multiple constraints, the semantic range is the intersection:

```eiffel
class AUDITABLE_ACCOUNT_STORE [G -> {ACCOUNT, AUDITABLE}]
```

This class operates in the semantic intersection of accounts AND auditable things. Feature names can draw from both domains.

#### Semantic Range Summary

| Class Type | Semantic Range | Vocabulary Style |
|------------|----------------|------------------|
| Unconstrained generic `[G]` | Unbounded | Maximally generic |
| Constrained generic `[G -> X]` | X's domain | X's vocabulary valid |
| Multiple constraints `[G -> {X, Y}]` | Intersection of X and Y | Both vocabularies valid |
| Concrete class | Its specific domain | Domain-specific |
| Abstract ancestor | Union of descendants | Must stay neutral |

## Other Language Constructs Affecting Semantic Naming

Beyond inheritance and genericity, several Eiffel constructs influence how we think about feature naming semantics.

### Agents: Functional Context

When features are used as agents, they shift into a functional/callback context:

```eiffel
my_list.do_all (agent print_item)
button.on_click (agent handle_click)
```

**Naming implications:**
- Action-oriented names work well: `process`, `handle`, `transform`
- Consider both direct call and agent contexts when naming
- Aliases can provide both: `process_item, item_processor: ...`

```eiffel
process_item,
item_processor,      -- Reads better in: agent item_processor
handle_record,
record_handler: PROCEDURE [TUPLE [ITEM]]
```

### Once Functions: Singleton/Cached Semantics

Once functions return the same value after first computation. Names should suggest this persistence:

```eiffel
shared_config,
global_config,
config_instance: SIMPLE_CONFIG
    once
        create Result.make
    end

cached_connection,
connection_pool,
db_connection: DATABASE_CONNECTION
    once
        create Result.make
    end
```

**Naming patterns:**
- `shared_X`, `global_X` - emphasizes shared nature
- `X_instance`, `the_X` - singleton pattern
- `cached_X` - emphasizes memoization
- Avoid names suggesting fresh creation: `new_X`, `create_X`

### Deferred vs Effective: Abstract to Concrete

Deferred features are promises - they define *what* without *how*. Names should stay abstract:

```eiffel
deferred class SERIALIZABLE
feature
    -- GOOD: Abstract vocabulary
    serialize,
    to_external_format,
    export_data: STRING
        deferred end

    -- BAD: Implementation-specific vocabulary in deferred
    to_json: STRING      -- What if descendant serializes to XML?
        deferred end
```

Effective (implementing) features can be more concrete:

```eiffel
class JSON_DOCUMENT
inherit
    SERIALIZABLE
feature
    serialize,
    to_external_format,
    export_data,
    to_json,            -- Now appropriate: we know it's JSON
    as_json_string: STRING
        do
            -- JSON-specific implementation
        end
```

**The pattern**: Deferred = abstract vocabulary. Effective = can add concrete aliases.

### Export Policies: Public vs Private Vocabulary

Export status affects appropriate vocabulary:

```eiffel
feature {NONE} -- Implementation

    internal_buffer,
    scratch_space,
    temp_storage: STRING
        -- Private: implementation vocabulary OK

    raw_process,
    unsafe_modify: ...
        -- Private: can expose "dangerous" operations

feature -- Public API

    content,
    value,
    data: STRING
        -- Public: clean, domain-appropriate vocabulary
        do
            Result := internal_buffer
        end
```

**Principle**: Private features can use implementation vocabulary. Public features should use domain vocabulary.

### Rename in Inheritance: The Old Way

Rename is Eiffel's existing mechanism for semantic frame shifts:

```eiffel
class BANK_ACCOUNT
inherit
    ACCOUNT
        rename
            name as account_holder,
            identifier as account_number
        end
```

**Our pattern reduces need for this** by pre-salting names in the supplier. But rename remains valuable for:
- Third-party libraries you can't modify
- Resolving name clashes in multiple inheritance
- Legacy code integration

### Redefine: Same Name, Different Semantics

When you redefine a feature, the name stays but behavior changes. This is a semantic shift that should be documented:

```eiffel
class PREMIUM_ACCOUNT
inherit
    ACCOUNT
        redefine
            calculate_fee
        end
feature
    calculate_fee: DECIMAL
            -- Redefined: Premium accounts get 50% discount
        note
            semantic_shift: "Applies premium discount vs standard calculation"
        do
            Result := Precursor * 0.5
        end
```

**Naming consideration**: If redefinition significantly changes semantics, consider whether the name still fits. The name is a promise about behavior.

### Preconditions and Postconditions: Contract Vocabulary

Contracts have their own naming conventions:

```eiffel
set_value (v: INTEGER)
    require
        value_non_negative: v >= 0
        value_in_range: v <= maximum
        not_frozen: not is_locked
    do
        internal_value := v
    ensure
        value_set: value = v
        value_updated: value /= old value or v = old value
        still_valid: is_valid
```

**Contract naming patterns:**
- `X_valid`, `valid_X` - validation checks
- `X_exists`, `has_X` - existence checks
- `not_X`, `X_not_Y` - negation
- `X_set`, `X_updated` - postcondition state changes
- `result_X` - postconditions about Result

Consider aliases that work in both feature and contract contexts:

```eiffel
is_valid,
valid,
satisfies_invariant: BOOLEAN

has_content,
not_empty,
content_exists: BOOLEAN
```

### Creation Procedures: Factory Semantics

Creation procedures have established conventions:

```eiffel
create {SIMPLE_CONFIG}
    make,
    make_empty,
    make_with_file,
    make_from_string,
    make_from_json
```

**Standard patterns:**
- `make` - default creation
- `make_X` - creation with specific input
- `make_from_X` - conversion/parsing creation
- `make_with_X` - creation with dependency injection

**Semantic frame aliases for creation:**

```eiffel
make_with_file,
load_from_file,      -- Config frame
parse_file,          -- Parser frame
open: ...            -- File frame
```

### Conversion: Type Bridge Vocabulary

Convert features bridge between types:

```eiffel
class SIMPLE_DECIMAL
convert
    to_string: {STRING},
    to_real: {REAL_64},
    make_from_string ({STRING})
feature
    to_string,
    as_string,
    string_value: STRING

    to_real,
    as_real,
    real_value,
    to_double,
    double_value: REAL_64
```

**Conversion naming patterns:**
- `to_X` - explicit conversion
- `as_X` - view/cast semantics
- `X_value` - accessor style
- Provide aliases for different mental models

### Expanded Types: Value Semantics

Expanded types have value (copy) semantics vs reference semantics. Names might reflect this:

```eiffel
expanded class POINT
    -- Value type: each variable has its own copy

class POINT_REF
    -- Reference type: variables can share

feature
    x, horizontal, h: INTEGER
    y, vertical, v: INTEGER
```

For libraries offering both:

```eiffel
point,
point_value,
location: POINT          -- Expanded, copied

point_ref,
point_reference,
shared_point: POINT_REF  -- Reference, shared
```

### Separate (SCOOP): Concurrency Context

SCOOP's `separate` keyword introduces concurrency semantics:

```eiffel
feature
    async_process,
    request_processing,
    queue_for_processing (item: separate ITEM)
        -- Name suggests asynchronous nature

    sync_result,
    await_result,
    blocking_fetch: RESULT
        -- Name suggests waiting/blocking

    concurrent_items,
    shared_collection,
    thread_safe_list: separate LIST [ITEM]
```

**SCOOP naming considerations:**
- Indicate async nature: `async_X`, `request_X`, `queue_X`
- Indicate blocking: `await_X`, `sync_X`, `blocking_X`
- Indicate thread-safety: `concurrent_X`, `shared_X`, `thread_safe_X`

### Frozen: Final/Locked Semantics

Frozen features cannot be redefined. Names might indicate this finality:

```eiffel
frozen core_algorithm,
frozen fundamental_process,
frozen locked_behavior: ...
    -- Implementation is fixed, not overridable

frozen standard_hash,
frozen immutable_id: INTEGER
    -- Value is architecturally fixed
```

**When to alias frozen features:**
- When the frozen behavior has domain-specific meanings
- When different clients conceptualize the fixed behavior differently

### Construct Impact Summary

| Construct | Semantic Impact | Naming Guidance |
|-----------|-----------------|-----------------|
| **Agents** | Functional context | Action verbs, `X_handler` aliases |
| **Once** | Singleton/cached | `shared_X`, `cached_X`, avoid `new_X` |
| **Deferred** | Abstract promise | Abstract vocabulary only |
| **Effective** | Concrete implementation | Can add concrete aliases |
| **Export {NONE}** | Private implementation | Implementation vocabulary OK |
| **Export public** | Public API | Domain vocabulary required |
| **Rename** | Frame shift mechanism | Our pattern reduces need |
| **Redefine** | Behavior change | Document semantic shift |
| **Contracts** | Validation vocabulary | `valid_X`, `has_X`, `X_set` |
| **Creation** | Factory semantics | `make_X`, `make_from_X` |
| **Convert** | Type bridging | `to_X`, `as_X`, `X_value` |
| **Expanded** | Value semantics | May need `_value` vs `_ref` aliases |
| **Separate** | Concurrency | `async_X`, `shared_X`, `concurrent_X` |
| **Frozen** | Final behavior | Indicates non-overridable |

## Best Practices

### DO
- Add names that are genuinely natural for each semantic frame
- Document every semantic frame in class notes
- Document each alias with its frame context in feature notes
- Keep the original/generic name as the first in the list
- Consider the actual library dependency graph for realistic frames
- Let generic ancestors stay generic
- Add specialized names in specialized descendants
- Pre-salt high fan-in suppliers for ecosystem-wide benefit

### DON'T
- Remove existing feature names (breaking change)
- Add frivolous aliases without semantic justification
- Forget to document the semantic context
- Add names for hypothetical frames with no real use case
- Create ambiguous names that could mean different things
- Pollute generic ancestors with domain-specific vocabulary they shouldn't know about

## Measuring Success

After applying this pattern:

1. **Reduced compile errors** - AI and human "guesses" hit valid names more often
2. **Cleaner client code** - No rename clauses needed for vocabulary matching
3. **Faster development** - Less time looking up "correct" names
4. **Self-documenting code** - Feature names reflect usage context
5. **Easier onboarding** - New developers use natural vocabulary immediately

## Conclusion

Semantic Frame Naming transforms Eiffel classes from rigid single-vocabulary APIs into flexible polyglot interfaces. By anticipating how different contexts will naturally name things, we eliminate friction between developer intent and implementation reality.

This is not syntactic sugar - it's a fundamental improvement in how libraries communicate across domain boundaries. The supplier speaks every client's language.

---

*Part of the Simple Eiffel ecosystem documentation.*
*Pattern developed through AI-human collaborative insight, December 2025.*

---

## Lessons from Practice: simple_json Refactor

### The Two-Phase Workflow

**Phase 1: Pre-Salt the Supplier**
Add semantic aliases to the supplier class features. This is non-breaking - existing code continues to work.

**Phase 2: Refactor Client Consumers**  
Walk through each client library and update calls to use context-appropriate aliases. Compile and test after each client.

### Feature-Level Semantic Context

**Critical Insight**: Semantic framing operates at the *feature level*, not just the *class level*.

Within a single client class, different features may have different semantic contexts when calling the same supplier:

```eiffel
class HTTP_CLIENT

feature -- Request building
    send_request
        do
            -- Building outbound JSON: "encode" semantic
            body := json.new_object  -- or json.build_object
        end

feature -- Response handling  
    handle_response (a_body: STRING)
        do
            -- Parsing inbound JSON: "decode" semantic
            data := json.decode_payload (a_body)
        end
end
```

The same `SIMPLE_JSON` supplier is called, but the semantic context differs by feature purpose.

### Real-World Mapping from simple_json Refactor

| Client Library | Feature Context | Original Call | Semantic Alias |
|----------------|-----------------|---------------|----------------|
| simple_config | Loading config files | `parse_file` | `load_config` |
| simple_http | Decoding HTTP response | `parse` | `decode_payload` |
| simple_ai_client | Parsing AI API response | `parse` | `parse_response` |
| simple_ai_client | Deserializing embeddings | `parse` | `deserialize` |
| simple_web (response) | Decoding HTTP body | `parse` | `decode_payload` |
| simple_web (request) | Decoding request body | `parse` | `decode_payload` |
| simple_lsp | Parsing JSON-RPC message | `parse` | `parse_message` |
| simple_sql | Deserializing stored JSON | `parse` | `deserialize` |
| simple_codec | Format conversion | `parse` | `decode` |
| simple_ci | Loading CI config | `parse` | `load_config` |
| simple_showcase | Decoding HTTP request | `parse` | `decode_payload` |

### Patterns Observed

1. **Config contexts** gravitate toward: `load_config`, `read_settings`
2. **HTTP/API contexts** gravitate toward: `decode_payload`, `decode`, `encode`
3. **Serialization contexts** gravitate toward: `serialize`, `deserialize`
4. **Message/protocol contexts** gravitate toward: `parse_message`, `encode_message`
5. **AI/LLM contexts** gravitate toward: `parse_response`, `parse_completion`

### The Compile-Test Loop

For each client:
1. `sed` or edit to change calls to semantic aliases
2. Clean compile (`rm -rf EIFGENs && ec.exe -batch ...`)
3. Run tests
4. Log to oracle
5. Move to next client

This systematic approach ensures no regressions while improving semantic clarity across the ecosystem.
