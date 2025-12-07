# Building an XML Library in Eiffel: Taming Gobo's XM_* Classes with AI Assistance

**Date:** December 7, 2025
**Library:** simple_xml
**Repository:** https://github.com/ljr1981/simple_xml

---

## Executive Summary

This post documents a development session where we designed and built a high-level XML parsing and building library for Eiffel. The library wraps Gobo's complex XM_* classes with a simple, fluent API and passes 10 automated tests. It's now integrated into FOUNDATION_API with 3 additional integration tests (40 total tests passing).

This session also uncovered a critical gotcha about ECF UUIDs that causes mysterious "wrong classes loaded" behavior when libraries share similar UUIDs.

---

## The Problem

Eiffel has powerful XML support through Gobo's XM_* classes, but the API is verbose and requires understanding multiple cursor types, callback pipes, and factory patterns. Simple tasks like parsing XML and extracting a value require significant boilerplate.

**Requirements:**
- Parse XML strings and files with a single method call
- Build XML documents using a fluent API
- Query elements with XPath-like syntax
- Modify documents (add/remove/update elements and attributes)
- Follow the `simple_*` library patterns
- Be immediately usable via ECF reference

---

## Architecture Design

We chose a **facade pattern** with fluent builders:

```
SIMPLE_XML (parser facade)
    ├── parse (string) → SIMPLE_XML_DOCUMENT
    ├── parse_file (path) → SIMPLE_XML_DOCUMENT
    ├── build (root_name) → SIMPLE_XML_BUILDER
    └── new_document (root_name) → SIMPLE_XML_DOCUMENT

SIMPLE_XML_DOCUMENT (document wrapper)
    ├── root → SIMPLE_XML_ELEMENT
    ├── element_at (xpath) → element
    ├── text_at (xpath) → string
    └── to_string → xml output

SIMPLE_XML_ELEMENT (element wrapper)
    ├── name, text, attr(name)
    ├── child(name), children, children_named(name)
    ├── set_text, set_attr (fluent)
    ├── add_child, remove_child
    └── to_string → element as xml

SIMPLE_XML_BUILDER (fluent construction)
    ├── element(name) → moves into child
    ├── attr(name, value) → adds attribute
    ├── text(content) → sets text
    ├── done → returns to parent
    └── to_document / to_string
```

**Why this design?**

1. **Facade pattern** - Hide Gobo's complexity behind simple methods
2. **Fluent API** - Chain calls for readable code: `builder.element("item").attr("id", "1").text("value").done`
3. **Wrapper classes** - Provide Eiffel-friendly interface over XM_* internals
4. **XPath-like queries** - Navigate with `doc.text_at("root/child/item")` instead of cursor iteration

---

## Implementation Timeline

| Time | Activity |
|------|----------|
| 0:00 | Design discussion - architecture, Gobo wrapper approach |
| 0:15 | Created project structure, ECF with Gobo dependencies |
| 0:25 | Implemented SIMPLE_XML (parser facade) |
| 0:35 | Implemented SIMPLE_XML_DOCUMENT (document wrapper) |
| 0:50 | Implemented SIMPLE_XML_ELEMENT (element wrapper) |
| 1:05 | Implemented SIMPLE_XML_BUILDER (fluent builder) |
| 1:15 | First compile - discovered DS_LIST iteration gotcha |
| 1:25 | Fixed ARRAYED_LIST across iteration (`ic` IS the item) |
| 1:35 | Fixed HASH_TABLE key access (`@ic.key` syntax) |
| 1:45 | All 10 tests passing |
| 1:55 | Created README.md and docs |
| 2:05 | Integrated into FOUNDATION_API |
| 2:15 | **UUID collision bug** - classes not visible |
| 2:25 | Fixed UUID, added 3 integration tests (40 total passing) |

---

## Code Examples

### Parsing XML

```eiffel
-- Parse and query
create xml.make
doc := xml.parse ("<config><db host='localhost' port='5432'/></config>")

if doc.is_valid then
    if attached doc.element_at ("config/db") as db then
        print (db.attr ("host"))  -- "localhost"
        print (db.attr ("port"))  -- "5432"
    end
end

-- XPath-like text extraction
value := doc.text_at ("config/setting/value")
```

### Building XML

```eiffel
-- Fluent construction
create xml.make
builder := xml.build ("catalog")
builder := builder
    .element ("book")
        .attr ("id", "bk101")
        .element ("title").text ("Eiffel Programming").done
        .element ("author").text ("Bertrand Meyer").done
        .element ("price").text ("49.95").done
    .done
    .element ("book")
        .attr ("id", "bk102")
        .element ("title").text ("Design by Contract").done
    .done

doc := builder.to_document
print (doc.to_string)
```

**Output:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<catalog>
  <book id="bk101">
    <title>Eiffel Programming</title>
    <author>Bertrand Meyer</author>
    <price>49.95</price>
  </book>
  <book id="bk102">
    <title>Design by Contract</title>
  </book>
</catalog>
```

### Modifying XML

```eiffel
-- Update existing document
doc := xml.parse (existing_xml)
if attached doc.root as root then
    -- Update attribute (fluent)
    root.set_attr ("version", "2.0")

    -- Add new child
    root.add_element ("timestamp").set_text (current_time)

    -- Remove child
    if attached root.child ("deprecated") as old then
        root.remove_child (old)
    end
end

updated_xml := doc.to_string
```

---

## Gotchas and Lessons Learned

### 1. DS_LIST Requires Cursor-Based Iteration

Gobo's `DS_LIST` doesn't support `across` loops:

```eiffel
-- WRONG: Compile error
l_attrs: DS_LIST [XM_ATTRIBUTE]
across l_attrs as ic loop ... end

-- RIGHT: Use cursor-based iteration
l_cursor := l_attrs.new_cursor
from l_cursor.start until l_cursor.after loop
    print (l_cursor.item.name)
    l_cursor.forth
end
```

### 2. ARRAYED_LIST across - `ic` IS the Item

```eiffel
-- WRONG: ic.item doesn't exist
l_items: ARRAYED_LIST [XM_CHARACTER_DATA]
across l_items as ic loop
    xm_element.delete (ic.item)  -- ERROR!
end

-- RIGHT: ic IS the item directly
across l_items as ic loop
    xm_element.delete (ic)  -- ic IS the XM_CHARACTER_DATA
end
```

### 3. HASH_TABLE Key Access with `@` Prefix

```eiffel
-- WRONG: No 'key' on cursor
l_table: HASH_TABLE [STRING, STRING]
across l_table as ic loop
    print (ic.key)   -- ERROR: unknown identifier
    print (ic.item)  -- ERROR: calling 'item' on STRING
end

-- RIGHT: Use @ prefix for key, ic IS the value
across l_table as ic loop
    print (@ic.key)  -- The key (STRING)
    print (ic)       -- The value (STRING) - ic IS the value
end
```

### 4. VOIT(2) - Loop Variable Name Conflicts

```eiffel
class MY_CLASS
feature
    attr (a_name: STRING): STRING  -- Feature named 'attr'

    process
        local
            l_attrs: HASH_TABLE [STRING, STRING]
        do
            -- WRONG: 'attr' conflicts with feature name
            across l_attrs as attr loop ... end  -- VOIT(2) error!

            -- RIGHT: Use different name
            across l_attrs as ic_attr loop ... end
        end
```

### 5. ECF UUID MUST Be Unique (Critical!)

This one caused significant debugging time. I copied `simple_validation.ecf` as a template and modified it, but forgot to generate a new UUID:

```xml
<!-- WRONG: Copied UUID from another library -->
<system ... uuid="A1B2C3D4-E5F6-7890-ABCD-EF1234567890" ...>

<!-- What happens: EiffelStudio uses UUIDs to identify libraries.
     When two libraries share similar UUIDs, it loads classes from
     the WRONG library! We saw SIMPLE_VALIDATOR classes when expecting
     SIMPLE_XML classes. -->

<!-- RIGHT: Generate a fresh UUID -->
<system ... uuid="A1B2C3D4-E5F6-7890-ABCD-EF1234567801" ...>
```

**How to generate a proper UUID:**
```powershell
[guid]::NewGuid().ToString()
```

**Symptom:** When you reference a library and get "unknown class" errors, or see completely wrong classes appearing in EiffelStudio's library view, check for UUID collision.

---

## Final Test Results

### simple_xml Standalone (10 tests)
```
=== Simple XML Tests ===

  PASS: test_parse_simple
  PASS: test_parse_with_attributes
  PASS: test_invalid_xml
  PASS: test_element_navigation
  PASS: test_text_at_query
  PASS: test_modification
  PASS: test_builder_simple
  PASS: test_builder_nested
  PASS: test_builder_with_attributes
  PASS: test_document_to_string

Results: 10 passed, 0 failed
ALL TESTS PASSED
```

### FOUNDATION_API Integration (40 tests)
```
...
XML Tests
---------
  PASS: test_parse_xml
  PASS: test_build_xml
  PASS: test_new_xml_document
...
Results: 40 passed, 0 failed
ALL TESTS PASSED
```

---

## Library Contents

```
simple_xml/
├── simple_xml.ecf
├── README.md
├── src/
│   ├── simple_xml.e           -- Parser facade
│   ├── simple_xml_builder.e   -- Fluent builder
│   ├── simple_xml_document.e  -- Document wrapper
│   └── simple_xml_element.e   -- Element wrapper
├── tests/
│   ├── simple_xml_test_app.e
│   └── simple_xml_test_set.e
└── docs/
    ├── index.html
    └── css/style.css
```

---

## FOUNDATION_API Integration

`simple_xml` is now integrated into FOUNDATION_API:

```eiffel
class FOUNDATION_API

feature -- XML Processing

    parse_xml (a_xml: STRING): SIMPLE_XML_DOCUMENT
            -- Parse `a_xml' string and return document.
        do
            Result := xml_processor.parse (a_xml)
        end

    build_xml (a_root_name: STRING): SIMPLE_XML_BUILDER
            -- Create XML builder with root element.
        do
            Result := xml_processor.build (a_root_name)
        end

    new_xml_document (a_root_name: STRING): SIMPLE_XML_DOCUMENT
            -- Create empty XML document with root.
        do
            Result := xml_processor.new_document (a_root_name)
        end

    xml: SIMPLE_XML
            -- Direct access for advanced operations.
```

### Using via FOUNDATION_API

```eiffel
class MY_CONFIG_READER

inherit
    FOUNDATION_API

feature
    load_config (a_path: STRING)
        local
            doc: SIMPLE_XML_DOCUMENT
        do
            doc := parse_xml (read_file (a_path))
            if doc.is_valid then
                database_host := doc.text_at ("config/database/host")
                database_port := doc.text_at ("config/database/port").to_integer
            end
        end
end
```

### Using Standalone

1. Clone or reference the repository
2. Set environment variable: `SIMPLE_PDF=D:\path\to\simple_xml`
3. Add to your ECF:

```xml
<library name="simple_xml" location="$SIMPLE_XML/simple_xml.ecf"/>
```

---

## Gobo XM_* Classes Wrapped

For reference, here's what simple_xml wraps:

| Gobo Class | Complexity | simple_xml Equivalent |
|------------|------------|----------------------|
| XM_EIFFEL_PARSER | Callbacks, pipes, factories | `SIMPLE_XML.parse` |
| XM_TREE_CALLBACKS_PIPE | Event routing | Hidden in parser |
| XM_DOCUMENT | Document navigation | `SIMPLE_XML_DOCUMENT` |
| XM_ELEMENT | Element access, iteration | `SIMPLE_XML_ELEMENT` |
| XM_ATTRIBUTE | Attribute handling | `element.attr(name)` |
| DS_LINEAR_CURSOR | Manual iteration | `across` / `children` |
| XM_NAMESPACE | Namespace management | Simplified/hidden |

---

## Conclusion

AI-assisted development allowed us to:

- **Design** a clean facade over Gobo's complex XM_* classes
- **Implement** 4 classes with fluent APIs
- **Debug** Eiffel-specific gotchas (DS_LIST, ARRAYED_LIST, HASH_TABLE iteration)
- **Discover** the critical ECF UUID collision bug
- **Test** with 10 unit tests + 3 integration tests
- **Integrate** into FOUNDATION_API for immediate ecosystem use
- **Document** gotchas for future reference (5 new entries in gotchas.md)

The key insights from this session:

1. **Gobo's DS_* collections behave differently than standard Eiffel collections** - cursor-based iteration required
2. **`across` loop variable semantics vary by collection type** - sometimes `ic` is the cursor, sometimes it IS the item
3. **ECF UUIDs are identity tokens** - reusing them causes silent library confusion
4. **AI-generated UUIDs are fake** - always generate real UUIDs with proper tools

The library is now part of the `simple_*` ecosystem and has been integrated into FOUNDATION_API for immediate use.

**Repository:** https://github.com/ljr1981/simple_xml

---

*This development session was conducted using Claude Code (Anthropic's CLI tool) with Claude Opus 4.5.*
