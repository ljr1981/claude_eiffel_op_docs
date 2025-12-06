# Unified API Facade Architecture for Eiffel

## Introduction

We've developed a three-tier API facade architecture that dramatically simplifies how Eiffel applications consume library functionality. Instead of managing 20+ individual library dependencies, applications can inherit from a single class and gain access to everything they need.

## The Problem

Traditional Eiffel library usage requires:
- Adding each library to your ECF individually
- Managing environment variables for each library
- Understanding the API of each separate library
- Dealing with inconsistent naming conventions across libraries

For a typical web application, this meant including and learning: simple_base64, simple_hash, simple_uuid, simple_json, simple_csv, simple_markdown, simple_validation, simple_process, simple_randomizer, simple_jwt, simple_smtp, simple_sql, simple_cors, simple_rate_limiter, simple_template, simple_websocket, simple_web, simple_alpine, simple_htmx...

## The Solution: Layered API Facades

We created three facade classes that bundle related functionality through inheritance:

```
+------------------------------------------+
| simple_app_api (Application Layer)       |
|   APP class                              |
|   - Web client, Alpine.js components     |
+------------------------------------------+
                   |
                   | inherits from
                   v
+------------------------------------------+
| simple_service_api (Service Layer)       |
|   SERVICE class                          |
|   - JWT, SMTP, SQL, CORS, Rate Limiting  |
|   - Templates, WebSocket                 |
+------------------------------------------+
                   |
                   | inherits from
                   v
+------------------------------------------+
| simple_foundation_api (Foundation Layer) |
|   FOUNDATION class                       |
|   - Base64, Hashing, UUID, JSON, CSV     |
|   - Markdown, Validation, Process, Random|
+------------------------------------------+
```

## How It Works

### Layer 1: FOUNDATION

The `FOUNDATION` class bundles 9 core utility libraries:

```eiffel
class FOUNDATION

feature -- Base64
    base64_encode (a_string: STRING): STRING
    base64_decode (a_encoded: STRING): STRING
    base64_url_encode (a_string: STRING): STRING

feature -- Hashing
    sha256 (a_string: STRING): STRING
    sha1 (a_string: STRING): STRING
    md5 (a_string: STRING): STRING
    hmac_sha256 (a_key, a_message: STRING): STRING
    secure_compare (a_string1, a_string2: STRING): BOOLEAN

feature -- UUID
    new_uuid: STRING
    new_uuid_compact: STRING
    is_valid_uuid (a_string: STRING): BOOLEAN

feature -- JSON
    parse_json (a_json: STRING): detachable JSON_VALUE
    new_json_object: JSON_OBJECT
    new_json_array: JSON_ARRAY

feature -- CSV
    parse_csv (a_csv: STRING): BOOLEAN
    csv_field (a_row, a_col: INTEGER): STRING

feature -- Markdown
    markdown_to_html (a_markdown: STRING): STRING

feature -- Validation
    new_validator: SIMPLE_VALIDATOR
    is_valid_email (a_email: STRING): BOOLEAN
    is_valid_url (a_url: STRING): BOOLEAN

feature -- Process
    execute_command (a_command: STRING): TUPLE [output: STRING; exit_code: INTEGER]

feature -- Random
    random_integer_in_range (a_min, a_max: INTEGER): INTEGER
    random_alphanumeric_string (a_length: INTEGER): STRING

end
```

### Layer 2: SERVICE

The `SERVICE` class inherits from `FOUNDATION` and adds 7 web service libraries:

```eiffel
class SERVICE

inherit
    FOUNDATION
        rename make as foundation_make end

feature -- JWT Authentication
    new_jwt (a_secret: STRING): SIMPLE_JWT
    create_token (a_secret, a_subject, a_issuer: STRING; a_expires: INTEGER): STRING
    verify_token (a_secret, a_token: STRING): BOOLEAN

feature -- SMTP Email
    new_smtp (a_host: STRING; a_port: INTEGER): SIMPLE_SMTP

feature -- SQL Database
    new_database (a_path: STRING): SIMPLE_DATABASE
    new_memory_database: SIMPLE_DATABASE

feature -- CORS
    new_cors: SIMPLE_CORS
    new_cors_permissive: SIMPLE_CORS

feature -- Rate Limiting
    new_rate_limiter (a_max_requests, a_window_seconds: INTEGER): SIMPLE_RATE_LIMITER

feature -- Templates
    new_template: SIMPLE_TEMPLATE
    render_template (a_template: STRING; a_data: HASH_TABLE [STRING, STRING]): STRING

feature -- WebSocket
    new_ws_handshake: WS_HANDSHAKE
    new_ws_text_frame (a_text: STRING; a_is_final: BOOLEAN): WS_FRAME
    new_ws_binary_frame (a_data: ARRAY [NATURAL_8]; a_is_final: BOOLEAN): WS_FRAME

end
```

### Layer 3: APP

The `APP` class inherits from `SERVICE` and adds 2 application-level libraries:

```eiffel
class APP

inherit
    SERVICE
        rename make as service_make end

feature -- Web Client
    new_web_client: SIMPLE_WEB_CLIENT
    new_get_request (a_url: STRING): SIMPLE_WEB_REQUEST
    new_post_request (a_url: STRING): SIMPLE_WEB_REQUEST
    web_client: SIMPLE_WEB_CLIENT  -- singleton

feature -- Alpine.js Components
    new_alpine_factory: ALPINE_FACTORY
    alpine: ALPINE_FACTORY  -- singleton

end
```

## Usage Examples

### Before: Traditional Approach

```eiffel
class MY_APPLICATION

inherit
    ANY

feature {NONE} -- Initialization

    make
        local
            l_base64: SIMPLE_BASE64
            l_hash: SIMPLE_HASH
            l_uuid: SIMPLE_UUID
            l_json: SIMPLE_JSON
            l_jwt: SIMPLE_JWT
            l_smtp: SIMPLE_SMTP
            l_db: SIMPLE_DATABASE
            l_cors: SIMPLE_CORS
        do
            create l_base64
            create l_hash
            create l_uuid
            create l_json
            create l_jwt.make ("secret")
            create l_smtp.make ("smtp.example.com", 587)
            create l_db.make (":memory:")
            create l_cors

            -- Use each library separately...
            encoded := l_base64.encode ("data")
            hash := l_hash.sha256 ("password")
            -- etc.
        end

end
```

**ECF required:**
```xml
<library name="simple_base64" location="$SIMPLE_BASE64\simple_base64.ecf"/>
<library name="simple_hash" location="$SIMPLE_HASH\simple_hash.ecf"/>
<library name="simple_uuid" location="$SIMPLE_UUID\simple_uuid.ecf"/>
<library name="simple_json" location="$SIMPLE_JSON\simple_json.ecf"/>
<library name="simple_jwt" location="$SIMPLE_JWT\simple_jwt.ecf"/>
<library name="simple_smtp" location="$SIMPLE_SMTP\simple_smtp.ecf"/>
<library name="simple_sql" location="$SIMPLE_SQL\simple_sql.ecf"/>
<library name="simple_cors" location="$SIMPLE_CORS\simple_cors.ecf"/>
<!-- ... and more -->
```

### After: Facade Approach

```eiffel
class MY_APPLICATION

feature {NONE} -- Initialization

    make
        local
            app: APP
        do
            create app.make

            -- Everything through one object!
            encoded := app.base64_encode ("data")
            hash := app.sha256 ("password")
            uuid := app.new_uuid
            token := app.create_token ("secret", "user", "app", 3600)
            db := app.new_memory_database
            cors := app.new_cors

            -- Alpine.js component
            div := app.alpine.div
            div.x_data ("{count: 0}")

            -- Web request
            request := app.new_get_request ("https://api.example.com/data")
        end

end
```

**ECF required:**
```xml
<library name="simple_app_api" location="$SIMPLE_APP_API\simple_app_api.ecf"/>
```

## Choosing the Right Layer

| If your application needs... | Use this class |
|------------------------------|----------------|
| Core utilities only (encoding, hashing, JSON, etc.) | `FOUNDATION` |
| Web services (JWT, email, database, etc.) | `SERVICE` |
| Full web application (client, UI components) | `APP` |

Each layer includes everything from the layers below it, so `APP` gives you access to all `SERVICE` and `FOUNDATION` features.

## Design Principles

### 1. Factory Methods Over Direct Construction

Instead of exposing raw classes, the facades provide factory methods:

```eiffel
-- Good: Factory method
jwt := service.new_jwt ("secret")

-- Avoided: Direct construction requiring knowledge of class internals
create {SIMPLE_JWT} jwt.make_with_secret ("secret")
```

### 2. Convenience Methods for Common Operations

The facades add convenience methods that combine multiple operations:

```eiffel
-- Convenience: One call does it all
token := service.create_token ("secret", "user@example.com", "my-app", 3600)

-- Without convenience method (more verbose)
jwt := service.new_jwt ("secret")
jwt.set_subject ("user@example.com")
jwt.set_issuer ("my-app")
jwt.set_expiration (3600)
token := jwt.generate
```

### 3. Singleton Access for Shared Resources

Some objects benefit from singleton access:

```eiffel
-- Get the same Alpine factory throughout your application
div := app.alpine.div
span := app.alpine.span
-- Both use the same factory instance
```

### 4. Design by Contract Throughout

All features include proper contracts:

```eiffel
new_get_request (a_url: STRING): SIMPLE_WEB_REQUEST
    require
        url_not_void: a_url /= Void
        url_not_empty: not a_url.is_empty
    do
        create Result.make_get (a_url)
    ensure
        result_not_void: Result /= Void
    end
```

## Benefits

1. **Simplified Dependencies** - One ECF entry instead of 20+
2. **Consistent API** - All features follow the same naming conventions
3. **Discoverability** - IDE auto-completion shows all available features
4. **Reduced Boilerplate** - No need to create and manage multiple library objects
5. **Layered Architecture** - Choose the appropriate level for your needs
6. **Future-Proof** - New libraries can be added to facades without changing client code

## Repository Links

- [simple_foundation_api](https://github.com/ljr1981/simple_foundation_api) - Core utilities
- [simple_service_api](https://github.com/ljr1981/simple_service_api) - Web services
- [simple_app_api](https://github.com/ljr1981/simple_app_api) - Application layer
- [simple_showcase](https://github.com/ljr1981/simple_showcase) - Example application using simple_app_api

## Conclusion

The three-tier API facade architecture demonstrates how Eiffel's inheritance model can be leveraged to create clean, unified APIs. By bundling related libraries behind facade classes, we've made it dramatically easier to build Eiffel applications while maintaining the language's strong typing and Design by Contract principles.

The approach is particularly powerful for web applications, where the `APP` class provides one-stop access to everything from low-level encoding to high-level UI component generation.

---

*This architecture was developed as part of the simple_* library ecosystem for Eiffel.*
