# Iron Package Publishing Guide for simple_* Libraries

**Date:** December 7, 2025
**Purpose:** Document precise steps to publish all simple_* libraries to Eiffel's Iron repository

---

## Part 1: Iron System Overview

### What is Iron?
Iron is Eiffel's package management system, similar to npm (Node.js), pip (Python), or Cargo (Rust). It enables:
- **Package discovery**: Search and browse packages at [iron.eiffel.com](https://iron.eiffel.com)
- **Simplified installation**: `iron install <package>`
- **URI-based references**: Use `iron:package-name:file.ecf` instead of environment variables
- **Version management**: Packages are organized by EiffelStudio version (e.g., 25.02)

### Why Iron Matters
**Before Iron:**
```xml
<library name="json" location="$EIFFEL_LIBRARY/contrib/library/json/json.ecf"/>
```
Requires: Manual git clone, environment variable setup, team coordination.

**With Iron:**
```xml
<library name="json" location="iron:json:library/json.ecf"/>
```
Requires: Nothing. EiffelStudio resolves it automatically.

---

## Part 2: Package.iron File Format

### Basic Structure
```
package <package_name>

project
    <library_name> = "<relative_path_to.ecf>"

note
    title: <Display Title>
    description: "<Multi-line description>"
    tags: tag1, tag2, tag3
    license: <License Name>
    copyright: <Copyright Notice>
    link[home]: <URL>
    link[doc]: "Documentation" <URL>
    link[source]: <URL>
    link[license]: <URL>

setup
    compile_library = <Clib_folder>

end
```

### Complete Example (from espec)
```
package espec

project
    espec = "library/espec.ecf"

note
    description: "[
ESpec: Eiffel Specification Library
(simplified version without the GUI facilities).
    ]"
    title: ESpec, an Eiffel Specification Library
    tags: specification,test
    license: Eiffel Forum License v2 (see http://www.eiffel.com/licensing/forum.txt)
    link[license]: http://www.eiffel.com/licensing/forum.txt
    copyright: Software Engineering Lab (York University)
    link[home]: https://wiki.eecs.yorku.ca/project/eiffel/
    link[doc]: "Documentation" https://svn.eiffel.com/...README.txt

end
```

### Key Notes
- **Package name**: Must be unique in the repository
- **Project section**: Maps library names to ECF files (can have multiple)
- **Tags**: Comma-separated, used for search
- **Links**: Support categories like `[home]`, `[doc]`, `[source]`, `[license]`
- **Setup section**: For C library compilation (only if needed)

---

## Part 3: Iron Command Line Tool

Location: `<EiffelStudio>/tools/spec/win64/bin/iron.exe`

### Essential Commands

```bash
# List registered repositories
iron repository --list

# Add a repository
iron repository --add https://iron.eiffel.com/25.02

# Search for packages
iron search json

# Get package info
iron info json

# Install a package
iron install json

# Get installation path
iron path json

# Update all packages
iron update
```

### Share/Publish Commands

```bash
# Create a package on the repository
iron share create \
    --username <your_username> \
    --password <your_password> \
    --repository https://iron.eiffel.com/25.02 \
    --package "/path/to/package.iron" \
    --package-name "simple_json"

# Update an existing package
iron share update \
    --username <your_username> \
    --password <your_password> \
    --repository https://iron.eiffel.com/25.02 \
    --package "/path/to/package.iron" \
    --package-name "simple_json"

# List your packages
iron share list \
    --username <your_username> \
    --password <your_password> \
    --repository https://iron.eiffel.com/25.02
```

### Auto-Generate package.iron
```bash
# Generate package.iron from a folder containing ECF files
iron update_package_file /path/to/library
```

---

## Part 4: Publishing Workflow

### Step 1: Create Iron Account
1. Visit [https://iron.eiffel.com/repository/account/?register](https://iron.eiffel.com/repository/account/?register)
2. Register with email, username, password
3. Verify email (if required)

### Step 2: Create package.iron for Each Library

For each simple_* library, create a `package.iron` file in the library root:

**Example: simple_json/package.iron**
```
package simple_json

project
    simple_json = "simple_json.ecf"

note
    title: Simple JSON
    description: "[
Simple JSON - High-level JSON parsing and building for Eiffel.
Provides one-liner parsing, fluent builder API, and type-safe access.
SCOOP-compatible and void-safe.
    ]"
    tags: json, parser, builder, serialization, simple
    license: MIT
    copyright: 2024-2025 Larry Rix
    link[home]: https://github.com/ljr1981/simple_json
    link[source]: https://github.com/ljr1981/simple_json
    link[doc]: "README" https://github.com/ljr1981/simple_json/blob/main/README.md

end
```

### Step 3: Create Archive (if not using GitHub source)

Iron can pull from:
1. **Direct archive upload**: `.tar.gz` or `.zip` file
2. **GitHub/URL source**: Repository cloned on install

For GitHub-hosted packages, no archive is needed.

### Step 4: Publish to Iron

```bash
iron share create \
    --username ljr1981 \
    --password <password> \
    --repository https://iron.eiffel.com/25.02 \
    --package "D:/prod/simple_json/package.iron" \
    --package-name "simple_json" \
    --package-archive-source "D:/prod/simple_json" \
    --verbose
```

### Step 5: Verify and Document

1. Visit `https://iron.eiffel.com/repository/25.02/package/?name=simple_json`
2. Review metadata, update if needed via web interface
3. Test installation: `iron install simple_json`
4. Update library README with Iron installation instructions

---

## Part 5: simple_* Package.iron Templates

### Foundation Layer Libraries

Each needs a package.iron file in its root directory:

| Library | Tags |
|---------|------|
| simple_base64 | base64, encoding, decoding, simple |
| simple_csv | csv, parser, generator, tabular, simple |
| simple_datetime | date, time, duration, age, calendar, simple |
| simple_hash | hash, sha256, sha1, md5, hmac, crypto, simple |
| simple_htmx | htmx, html, components, web, simple |
| simple_json | json, parser, builder, serialization, simple |
| simple_logger | logging, structured, json, debug, simple |
| simple_markdown | markdown, html, parser, simple |
| simple_process | process, shell, command, execution, simple |
| simple_randomizer | random, generator, uuid, simple |
| simple_regex | regex, pattern, matching, fluent, simple |
| simple_uuid | uuid, guid, identifier, simple |
| simple_validation | validation, fluent, rules, simple |
| simple_xml | xml, parser, builder, simple |

### Service Layer Libraries

| Library | Tags |
|---------|------|
| simple_cache | cache, memory, storage, simple |
| simple_cors | cors, http, headers, web, simple |
| simple_jwt | jwt, token, authentication, simple |
| simple_pdf | pdf, generation, document, simple |
| simple_rate_limiter | rate-limit, throttle, api, simple |
| simple_smtp | smtp, email, mail, simple |
| simple_sql | sql, database, sqlite, simple |
| simple_template | template, engine, rendering, simple |
| simple_websocket | websocket, realtime, connection, simple |

### Application Layer Libraries

| Library | Tags |
|---------|------|
| simple_foundation_api | foundation, facade, utilities, simple |
| simple_service_api | service, facade, web, simple |
| simple_app_api | application, facade, full-stack, simple |
| simple_web | web, server, http, ewf, simple |
| simple_testing | testing, assertions, framework, simple |

---

## Part 6: Batch Publishing Script

Create a script to publish all libraries:

**publish_all.sh**
```bash
#!/bin/bash
IRON="C:/Program Files/Eiffel Software/EiffelStudio 25.02 Standard/tools/spec/win64/bin/iron.exe"
REPO="https://iron.eiffel.com/25.02"
USER="ljr1981"
# PASSWORD should be set via environment variable or prompt

LIBS=(
    simple_base64 simple_csv simple_datetime simple_hash
    simple_htmx simple_json simple_logger simple_markdown
    simple_process simple_randomizer simple_regex simple_uuid
    simple_validation simple_xml simple_cache simple_cors
    simple_jwt simple_pdf simple_rate_limiter simple_smtp
    simple_sql simple_template simple_websocket
    simple_foundation_api simple_service_api simple_app_api
    simple_web simple_testing
)

for lib in "${LIBS[@]}"; do
    echo "Publishing $lib..."
    "$IRON" share create \
        --username "$USER" \
        --password "$IRON_PASSWORD" \
        --repository "$REPO" \
        --package "D:/prod/$lib/package.iron" \
        --package-name "$lib" \
        --package-archive-source "D:/prod/$lib" \
        --batch
done
```

---

## Part 7: Post-Publishing Tasks

### Update ECF Files for Iron URIs (Optional)

The `iron update_ecf` command can convert library references:

```bash
iron update_ecf simple_showcase.ecf
```

This converts:
```xml
<library name="simple_json" location="$SIMPLE_JSON\simple_json.ecf"/>
```
To:
```xml
<library name="simple_json" location="iron:simple_json:simple_json.ecf"/>
```

### Update README Files

Add installation instructions to each library's README:

```markdown
## Installation

### Via Iron (Recommended)
```bash
iron install simple_json
```

Then in your ECF:
```xml
<library name="simple_json" location="iron:simple_json:simple_json.ecf"/>
```

### Via Git
```bash
git clone https://github.com/ljr1981/simple_json.git
export SIMPLE_JSON=/path/to/simple_json
```
```

---

## Part 8: Action Plan

### Immediate Tasks
1. [ ] Register Iron account at iron.eiffel.com
2. [ ] Create package.iron file template
3. [ ] Generate package.iron for all 32 simple_* libraries

### Short-Term Tasks
4. [ ] Test publishing with one library (e.g., simple_json)
5. [ ] Verify installation works
6. [ ] Publish remaining libraries in batches

### Post-Publishing Tasks
7. [ ] Update README files with Iron installation instructions
8. [ ] Consider converting ECF files to use Iron URIs
9. [ ] Set up automated publishing for future updates

---

## References

- [IRON Documentation](https://iron.eiffel.com/doc/)
- [IRON Repository](https://iron.eiffel.com/repository/)
- [IRON: Eiffel package repository](https://www.eiffel.org/doc/solutions/IRON-_Eiffel_package_repository)
- [Eiffel Software Iron Announcement](https://www.eiffel.com/2014/iron-facility-easily-share-eiffel-libraries/)
