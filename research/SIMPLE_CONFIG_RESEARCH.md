# simple_config Research Report

**Date:** 2025-12-08
**Library:** simple_config (Configuration File Management)

---

## Step 1: Specifications Research

### Configuration File Format Standards

| Format | Specification | Key Characteristics |
|--------|--------------|---------------------|
| **JSON** | RFC 8259, ECMA-404 | Universal, fast parsing, no comments |
| **TOML** | TOML v1.0.0 | Human-readable, comments, native datetime |
| **YAML** | YAML 1.2 | Indentation-based, complex data, comments |
| **INI** | No formal spec | Simple key=value, legacy, Windows heritage |
| **.env** | 12-Factor App | Environment variables, KEY=value |

### 12-Factor App Configuration Principles
- Store config in the environment
- Strict separation of config from code
- Config varies between deploys; code doesn't
- Never store secrets in code repository

### Key Findings
- **TOML** is rising in popularity for app config (Rust/Python ecosystems)
- **JSON** remains universal but lacks comments
- **YAML** dominates DevOps (Kubernetes, Docker Compose, CI/CD)
- **Environment variables** are standard for secrets and deployment-specific config

**Sources:**
- [Configuration Format Comparison](https://schoenwald.aero/posts/2025-05-03_configuration-format-comparison/)
- [JSON vs YAML vs TOML 2025](https://dev.to/leapcell/json-vs-yaml-vs-toml-vs-xml-best-data-format-in-2025-5444)
- [Best Config File Formats](https://jhall.io/posts/best-config-file-formats/)

---

## Step 2: Tech-Stack Library Analysis

### Go: Viper (Gold Standard)
**Features:**
- Multiple formats: JSON, TOML, YAML, HCL, INI, envfile, Java properties
- **Hierarchical configuration**: defaults → file → env → flags
- **Watch and reload**: File change detection, `OnConfigChange` callbacks
- **Remote providers**: etcd, Consul, Firestore
- Case-insensitive keys
- Pflags integration (CLI args override config)
- Struct unmarshaling

**API Pattern:**
```go
viper.SetDefault("port", 8080)
viper.SetConfigName("config")
viper.AddConfigPath(".")
viper.ReadInConfig()
viper.WatchConfig()
port := viper.GetInt("port")
```

### Node.js Options
| Library | Focus |
|---------|-------|
| **dotenv** | .env file loading, lightweight |
| **config** | JSON/YAML files, environment-specific |
| **convict** | Schema validation, type coercion |
| **nconf** | Hierarchical (CLI → env → file) |

### Python: python-dotenv
- Loads .env file into `os.environ`
- 12-factor app friendly
- Simple and focused

### Rust: config crate
- Layered configuration
- Multiple sources
- Strong 12-factor support

### Cross-Platform: dotenvx
- Released June 2024
- Consistent behavior across Python, Node, Rust
- Next-generation .env management

**Sources:**
- [Viper GitHub](https://github.com/spf13/viper)
- [Advanced Viper Features](https://tillitsdone.com/blogs/advanced-viper-configuration/)
- [dotenvx](https://dotenvx.com/blog/2024/06/24/dotenvx-next-generation-config-management.html)

---

## Step 3: Eiffel Ecosystem Analysis

### ISE Preferences Library
- Location: `$ISE_LIBRARY/library/preferences/`
- Main class: `PREFERENCES`
- Storage backends: XML file or Windows Registry
- Default values from XML files
- Hidden preferences support
- UI integration via `PREFERENCE_VIEW` / `PREFERENCES_WINDOW`

**Architecture:**
1. Storage layer: `PREFERENCES_STORAGE_I` interface
2. Preference types: Various `PREFERENCE` descendants
3. Manager: `PREFERENCE_MANAGER` for creation
4. Versioning support

**Key Features:**
- 5 levels of configuration control
- Default value files
- Persistence between sessions
- Type-safe preferences

**Limitations:**
- Heavy API (designed for GUI apps)
- No JSON support (XML or Registry only)
- No environment variable support
- No hierarchical override pattern
- No watch/reload

### simple_json Library (Our Foundation)
- `SIMPLE_JSON_OBJECT` for nested structures
- Basic parsing and serialization
- Can be used as config storage backend

---

## Step 4: Developer Pain Points

### Security Concerns with Environment Variables
- Globally accessible to all processes
- Plain text storage
- Accidentally logged in debugging
- No audit trail
- No version history
- Cross-platform quoting issues

### Configuration Management Nightmares
1. **Scattered configuration**: Files + env vars + CLI args with no clear precedence
2. **Environment discrepancies**: Dev vs staging vs production drift
3. **Secret rotation**: Updating secrets requires restarts
4. **No validation**: Invalid config discovered at runtime
5. **Synchronization burden**: Keeping config in sync across environments

### Common Complaints
- "Where's the audit trail for that API key?"
- "Configuration and secrets are mixed together"
- "Different environments have different secrets, requiring careful coordination"
- "Manual synchronization is error-prone"
- "As applications grow, managing secrets becomes more complex"

**Sources:**
- [Pitfalls of Environment Variables](https://cloudtruth.com/blog/the-pitfalls-of-using-environment-variables-for-config-and-secrets/)
- [OWASP Secrets Management](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [Don't Use Secrets in Environment Variables](https://www.nodejs-security.com/blog/do-not-use-secrets-in-environment-variables-and-here-is-how-to-do-it-better)

---

## Step 5: Innovation Opportunities

### What Would Make Developers' Lives Easier?

1. **Hierarchical Override Pattern (like Viper)**
   ```eiffel
   -- Priority: CLI args > env vars > config file > defaults
   config.set_default ("port", 8080)
   config.load_file ("config.json")
   config.load_environment_prefix ("MYAPP_")  -- MYAPP_PORT=9000 overrides

   port := config.integer_value ("port")  -- Returns 9000
   ```

2. **Dot Notation for Nested Access**
   ```eiffel
   -- Instead of: config.section ("database").string_value ("host")
   host := config.string_value ("database.host")
   ```

3. **Environment Variable Fallback**
   ```eiffel
   -- Check config file, fall back to env var
   api_key := config.string_or_env ("api_key", "MY_API_KEY")
   ```

4. **Schema Validation**
   ```eiffel
   config.require_string ("database.host")
   config.require_integer ("database.port").in_range (1, 65535)
   config.optional_string ("database.password")
   ```

5. **Multiple Config File Overlay**
   ```eiffel
   config.load_file ("defaults.json")
   config.load_file ("production.json")  -- Overrides defaults
   config.load_file ("local.json")       -- Overrides production
   ```

6. **File Watch and Reload**
   ```eiffel
   config.watch_for_changes
   config.on_change (agent handle_config_update)
   ```

7. **Typed Array Access**
   ```eiffel
   servers := config.string_array ("servers")
   ports := config.integer_array ("allowed_ports")
   ```

---

## Step 6: Design Strategy Synthesis

### Core Design Principles
1. **JSON-first**: Use simple_json as backend (consistent with ecosystem)
2. **Hierarchical**: Support defaults → file → env override
3. **Simple API**: Dot notation, type-safe accessors
4. **Environment-aware**: First-class env var support

### What simple_config Should Be
- Simple JSON configuration management
- Integration with environment variables
- Type-safe value access
- File persistence

### What simple_config Should NOT Be
- A full Viper clone
- Multi-format (TOML/YAML/INI) - stay JSON focused
- A secrets manager
- A remote configuration system

### API Surface Design

**Current API (Good Foundation):**
```eiffel
create config.make_with_file ("config.json")
value := config.string_value ("key")
value := config.string_value_or_default ("key", "default")
config.set_string ("key", "value")
config.save
```

**Recommended Additions:**
```eiffel
-- Dot notation
host := config.string_value ("database.host")

-- Environment integration
config.load_environment_prefix ("MYAPP_")
-- or per-key:
api_key := config.string_value_or_env ("api_key", "MY_API_KEY")

-- Typed arrays
servers := config.string_list ("servers")

-- Merge files
config.merge_file ("overrides.json")

-- Validation
config.require ("database.host")
if config.is_valid then ...
```

---

## Step 7: Gap Analysis - Current Implementation

### What Current Implementation Has
- ✅ JSON-based storage (via simple_json)
- ✅ Type-safe accessors: string, integer, boolean, real
- ✅ Default value variants: `*_value_or_default`
- ✅ Nested sections via `section` method
- ✅ Modification tracking (`is_modified`)
- ✅ File load/save operations
- ✅ Remove operation

### What Current Implementation Lacks
- ❌ Dot notation for nested access (`database.host`)
- ❌ Environment variable integration
- ❌ Hierarchical override (defaults < file < env)
- ❌ Multiple file merge/overlay
- ❌ Typed arrays (`string_list`, `integer_list`)
- ❌ Validation/required fields
- ❌ File watch/reload
- ❌ Default values built into schema

### API Ergonomics Issues
- Nested access requires chaining: `config.section("db").string_value("host")`
- No environment variable fallback
- No schema definition
- `to_json_pretty` returns same as `to_json` (not actually pretty)

---

## Recommendations

### Phase 1: Essential Improvements (Recommended)
1. **Dot notation** for nested access
2. **Environment variable** integration (`string_value_or_env`)
3. **Typed arrays** (`string_list`, `integer_list`)
4. **Fix `to_json_pretty`** to actually format

### Phase 2: Nice-to-Have
1. **File merge** (overlay configs)
2. **Load from environment prefix** (MYAPP_*)
3. **Required field validation**

### Phase 3: Future (Out of Scope)
1. File watch/reload
2. Remote configuration
3. Schema definition language

---

## Conclusion

The current simple_config implementation provides a functional JSON-based configuration system. However, compared to modern standards (Viper, dotenv), it lacks several expected features:

**Must Have:**
- Dot notation for nested keys
- Environment variable fallback
- Typed arrays

**Should Have:**
- Environment prefix loading
- File merge/overlay
- Actual pretty-printing

The implementation is a reasonable starting point but should be enhanced with environment variable integration to be considered production-ready for the simple_* ecosystem. The hierarchical override pattern (defaults → file → env) is a standard expectation in modern configuration libraries.
