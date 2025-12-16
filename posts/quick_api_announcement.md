# QUICK API Release: Zero-Configuration Facades for Beginners

**16 libraries now have QUICK classes - one-liner operations for the 80% use case.**

---

## TL;DR

We've added zero-configuration "QUICK" APIs to 16 libraries across two batches:

**Batch 1 (Services):**
- simple_sql, simple_http, simple_smtp, simple_jwt
- simple_encryption, simple_mq, simple_ai_client, simple_web

**Batch 2 (Data Formats):**
- simple_json, simple_cache, simple_validation, simple_template
- simple_csv, simple_xml, simple_yaml, simple_regex

Every QUICK class provides:
- Sensible defaults (no configuration needed)
- One-liner operations for common tasks
- Debug logging via simple_logger
- Access to full API when needed (`quick.underlying` attribute)

---

## The Problem We Solved

Eiffel's power comes with learning curves. New developers face:

```eiffel
-- Full API: 6+ lines to validate an email
create validator.make
result := validator.required.email.for_field ("email").validate (input)
if result.is_valid then
    -- handle valid
else
    -- handle invalid
end
```

**With QUICK API: 1 line**
```eiffel
if v.email (input) then ...
```

---

## QUICK API Pattern

Every QUICK class follows the same pattern:

```eiffel
class SIMPLE_*_QUICK

feature {NONE} -- Initialization
    make
        do
            create underlying.make  -- Full API
            create logger.make ("*_quick")  -- Debug logging
        end

feature -- One-liner Operations
    -- Simple operations returning BOOLEAN, STRING, or collections

feature -- Advanced Access
    underlying: SIMPLE_*
        -- Access full API when needed
```

---

## Highlights by Library

### SIMPLE_JSON_QUICK
```eiffel
create json.make

-- Parse and query in one call
name := json.get_string (json_text, "$.users[0].name")

-- Build JSON fluently
print (json.object.put ("name", "Alice").put ("age", 30).to_json)
-- {"name":"Alice","age":30}

-- Quick validation
if json.is_valid (input) then ...
if json.is_object (input) then ...
```

### SIMPLE_CACHE_QUICK - The "Remember" Pattern
```eiffel
create cache.make

-- The killer feature: get-or-compute
value := cache.remember ("expensive_key", agent compute_value)
-- Returns cached value if exists, otherwise computes and caches

-- Counters
cache.increment ("page_views")
cache.increment_by ("score", 10)

-- Stats
print (cache.stats)  -- "Hits: 42, Misses: 8, Rate: 84%"
```

### SIMPLE_VALIDATION_QUICK
```eiffel
create v.make

-- One-liner validators
if v.email ("user@example.com") then ...
if v.url ("https://example.com") then ...
if v.length_ok (input, 3, 20) then ...
if v.in_range (age, 18, 65) then ...
if v.one_of (status, <<"active", "pending", "closed">>) then ...

-- Error message on failure
if not v.email (input) then
    print (v.last_error)
end
```

### SIMPLE_REGEX_QUICK - Built-in Validators
```eiffel
create rx.make

-- Common validators included
if rx.is_email ("user@example.com") then ...
if rx.is_url ("https://example.com") then ...
if rx.is_phone ("555-123-4567") then ...
if rx.is_ipv4 ("192.168.1.1") then ...

-- Extract all from text
emails := rx.extract_emails (document)
urls := rx.extract_urls (document)
numbers := rx.extract_numbers (document)

-- Core operations
if rx.matches ("[a-z]+", "hello") then ...
words := rx.find_all ("\w+", "Hello World")
result := rx.replace_all ("\s+", " ", text)
```

### SIMPLE_YAML_QUICK - Config File Access
```eiffel
create yaml.make

-- Dot-path access (like config files!)
config := yaml.load ("config.yml")
host := yaml.get_string (config, "database.host")
port := yaml.get_integer (config, "database.port")
debug := yaml.get_boolean (config, "logging.debug")

-- One-liner file access
host := yaml.string_from_file ("config.yml", "database.host")
```

### SIMPLE_CSV_QUICK
```eiffel
create csv.make

-- Read/write one-liners
rows := csv.read ("data.csv")
csv.write ("output.csv", rows)

-- Headers as maps
across csv.read_with_headers ("data.csv") as rec loop
    print (rec ["name"] + " is " + rec ["age"])
end

-- Format variants
csv.use_tabs          -- TSV
csv.use_semicolons    -- European CSV
```

### SIMPLE_XML_QUICK - XPath Queries
```eiffel
create xml.make

-- XPath one-liners
titles := xml.xpath (book_xml, "//book/title")
title := xml.first (book_xml, "//book/title")
id := xml.attr (book_xml, "//book", "id")

-- Simple element access
title := xml.text (html, "title")
paragraphs := xml.texts (html, "p")

-- Quick build
print (xml.element ("name", "Alice"))
-- <name>Alice</name>
```

### SIMPLE_TEMPLATE_QUICK
```eiffel
create tpl.make

-- One-liner render
html := tpl.render ("Hello {{name}}!", <<["name", "World"]>>)

-- File rendering
html := tpl.file ("templates/email.html", <<["user", "Alice"]>>)

-- Conditional
html := tpl.render_if (is_logged_in, "Welcome back!", <<>>)

-- List rendering
html := tpl.render_list ("<li>{{name}}</li>", items)
```

---

## When to Use QUICK vs Full API

| Use QUICK when... | Use Full API when... |
|-------------------|----------------------|
| Learning Eiffel | Need maximum control |
| Prototyping | Performance-critical code |
| Scripts and tools | Complex validation chains |
| 80% of use cases | Advanced features |
| Config file parsing | Schema validation |
| Quick data extraction | Streaming/large files |

---

## Documentation

Each library now has:
- **README.md** with Quick Start (Zero-Configuration) section
- **docs/quick.html** with full QUICK API reference
- Access via `https://simple-eiffel.github.io/{library}/quick.html`

---

## What's Next

Potential QUICK candidates:
- simple_http (QUICK already done)
- simple_pdf (QUICK candidate)
- simple_docker (QUICK done in v1.4.0)

---

## Links

- **Organization**: [github.com/simple-eiffel](https://github.com/simple-eiffel)
- **Documentation**: [simple-eiffel.github.io](https://simple-eiffel.github.io)

---

*Released December 16, 2025*
*Human+AI collaboration: Larry Rix with Claude Opus 4.5 (Anthropic)*
