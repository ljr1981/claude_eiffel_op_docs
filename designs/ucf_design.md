# UCF - Universe Configuration File

**Universe Configuration File (UCF)** defines the scope of an Eiffel ecosystem for cross-project tooling in simple_lsp.

## Purpose

While ECF (Eiffel Configuration File) defines a single project, UCF defines an **ecosystem** - a collection of related projects/libraries that should be analyzed together.

Use cases:
- **Cross-library navigation** - Jump from simple_json to simple_sql definition
- **DbC Heat Mapping** - Visualize contract coverage across all 55+ simple_* libraries
- **Dependency Analysis** - Find circular dependencies, hub classes
- **Ecosystem Search** - "Find all features named `make` across entire ecosystem"
- **Version Compatibility** - Track which libraries work together

## File Format: TOML

UCF uses TOML for clean, readable configuration:

```toml
# simple_universe.ucf

[universe]
name = "simple-eiffel"
description = "Simple Eiffel Ecosystem"
version = "1.0.0"
root = "D:/prod"

# Standard library (ISE EiffelStudio)
[stdlib]
location = "$ISE_LIBRARY"
include = ["base", "time", "testing"]

# Ecosystem libraries
[[library]]
name = "simple_json"
location = "$SIMPLE_JSON"
ecf = "simple_json.ecf"
tags = ["data", "serialization"]

[[library]]
name = "simple_sql"
location = "$SIMPLE_SQL"
ecf = "simple_sql.ecf"
tags = ["data", "database"]

[[library]]
name = "simple_xml"
location = "$SIMPLE_XML"
ecf = "simple_xml.ecf"
tags = ["data", "serialization"]

[[library]]
name = "simple_toml"
location = "$SIMPLE_TOML"
ecf = "simple_toml.ecf"
tags = ["data", "config"]

[[library]]
name = "simple_yaml"
location = "$SIMPLE_YAML"
ecf = "simple_yaml.ecf"
tags = ["data", "config"]

[[library]]
name = "simple_codec"
location = "$SIMPLE_CODEC"
ecf = "simple_codec.ecf"
tags = ["data", "serialization", "conversion"]

[[library]]
name = "simple_lsp"
location = "$SIMPLE_LSP"
ecf = "simple_lsp.ecf"
tags = ["tooling", "ide"]

# ... more libraries ...

# Heat map configuration
[heatmap]
enabled = true
metrics = ["preconditions", "postconditions", "invariants", "check_assertions"]
colors = { none = "#FF0000", low = "#FFA500", medium = "#FFFF00", high = "#00FF00" }
thresholds = { low = 25, medium = 50, high = 75 }

# Visualization settings
[visualization]
dependency_graph = true
inheritance_tree = true
cluster_view = true
max_depth = 5

# Analysis settings
[analysis]
detect_circular_deps = true
detect_hub_classes = true
hub_threshold = 10  # Classes with >10 clients are "hubs"
```

## Discovery

UCF files can be:
1. **Explicit** - `simple_lsp --ucf path/to/universe.ucf`
2. **Workspace root** - Look for `*.ucf` in workspace
3. **Parent directories** - Search upward for `universe.ucf`
4. **Auto-generated** - Scan `$SIMPLE_*` environment variables

## Auto-Generation

When no UCF exists, simple_lsp can generate one:

```bash
simple_lsp --generate-ucf > simple_universe.ucf
```

This scans:
- `SIMPLE_*` environment variables
- `D:/prod/simple_*` directories
- Each library's ECF file

## Integration with LSP Features

### DbC Heat Map
UCF enables ecosystem-wide contract analysis:

```
[heatmap view]
┌─────────────────────────────────────────┐
│ simple_json      ████████░░ 82%        │
│ simple_sql       ██████████ 100%       │
│ simple_xml       ████████░░ 80%        │
│ simple_process   █████████░ 90%        │
│ simple_file      ██████░░░░ 60%        │
│ simple_http      ████░░░░░░ 40%        │
│ simple_crypto    ██████████ 95%        │
└─────────────────────────────────────────┘
```

### Cross-Library Navigation
With UCF, "Go to Definition" works across libraries:

1. In simple_lsp, hover over `SIMPLE_JSON`
2. UCF tells LSP where simple_json is located
3. LSP indexes simple_json's symbols
4. Ctrl+Click jumps to `simple_json/src/simple_json.e`

### Ecosystem Search
Search all libraries at once:

- `Ctrl+T` → "make" → Shows all `make` features across ecosystem
- `Ctrl+Shift+F` → "JSON" → Find all references to JSON types

### Dependency Graph
Visualize which libraries depend on which:

```
                simple_lsp
               /    |    \
    simple_json  simple_sql  simple_xml
         |          |
    [gobo_json] [sqlite]
```

## API Design

```eiffel
class UCF_PARSER
feature
    parse_file (a_path: STRING_32): UCF_UNIVERSE
    parse_string (a_toml: STRING_32): UCF_UNIVERSE
    generate_from_env: UCF_UNIVERSE  -- Auto-discover from SIMPLE_* vars

class UCF_UNIVERSE
feature
    name: STRING_32
    root: STRING_32
    libraries: ARRAYED_LIST [UCF_LIBRARY]
    library_by_name (a_name: STRING_32): detachable UCF_LIBRARY
    libraries_with_tag (a_tag: STRING_32): ARRAYED_LIST [UCF_LIBRARY]
    heatmap_config: UCF_HEATMAP_CONFIG

class UCF_LIBRARY
feature
    name: STRING_32
    location: STRING_32        -- With $ENV_VAR
    resolved_path: STRING_32   -- Expanded
    ecf_path: STRING_32
    tags: ARRAYED_LIST [STRING_32]
    parsed_ecf: detachable LSP_ECF_PARSER
```

## Implementation Plan

1. **Phase 1: UCF Parser** (uses simple_toml)
   - UCF_PARSER class
   - UCF_UNIVERSE, UCF_LIBRARY data classes
   - Auto-discovery from SIMPLE_* environment variables

2. **Phase 2: Cross-Library Indexing**
   - On UCF load, index all library ECFs
   - Build unified symbol database
   - Enable cross-library "Go to Definition"

3. **Phase 3: Heat Mapping**
   - Contract metric extraction
   - Heat map computation
   - VS Code webview visualization

4. **Phase 4: Dependency Analysis**
   - Build dependency graph
   - Circular dependency detection
   - Hub class identification

## File Locations

Standard UCF search paths:
1. Workspace root: `./universe.ucf`, `./.eiffel_lsp/universe.ucf`
2. User home: `~/.eiffel_lsp/universe.ucf`
3. System: `/etc/eiffel_lsp/universe.ucf`

## VS Code Integration

UCF settings in VS Code:

```json
{
    "eiffel.lsp.ucfPath": "D:/prod/simple_universe.ucf",
    "eiffel.lsp.autoDiscoverUniverse": true,
    "eiffel.lsp.heatmapEnabled": true
}
```

## Generated: December 10, 2025
