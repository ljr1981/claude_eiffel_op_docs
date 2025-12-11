# Simple Eiffel README.md Standard

## Overview

This document defines the standard format for all README.md files in simple_* libraries.

## Template

```markdown
<p align="center">
  <img src="https://raw.githubusercontent.com/simple-eiffel/claude_eiffel_op_docs/main/artwork/LOGO.png" alt="simple_ library logo" width="400">
</p>

# simple_LIBNAME

**[Documentation](https://simple-eiffel.github.io/simple_LIBNAME/)** | **[GitHub](https://github.com/simple-eiffel/simple_LIBNAME)**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Eiffel](https://img.shields.io/badge/Eiffel-25.02-blue.svg)](https://www.eiffel.org/)
[![Design by Contract](https://img.shields.io/badge/DbC-enforced-orange.svg)]()

Short one-line description of what this library does.

Part of the [Simple Eiffel](https://github.com/simple-eiffel) ecosystem.

## Status

**Production** | **Beta** | **Alpha** | **Planned**

## Overview

2-3 sentences describing the library's purpose and key capabilities.

```eiffel
-- Quick code example showing basic usage
local
    obj: SIMPLE_LIBNAME
do
    create obj.make
    -- typical operation
end
```

## Features

- Feature 1
- Feature 2
- Feature 3

## Installation

1. Set environment variable:
```bash
export SIMPLE_LIBNAME=/path/to/simple_libname
```

2. Add to ECF:
```xml
<library name="simple_libname" location="$SIMPLE_LIBNAME/simple_libname.ecf"/>
```

## License

MIT License
```

## Required Elements

| Element | Description |
|---------|-------------|
| Logo | Centered, uses central URL from claude_eiffel_op_docs/artwork |
| Title | `# simple_LIBNAME` |
| Links | `**[Documentation](...) \| [GitHub](...)**` |
| Badges | License MIT, Eiffel 25.02, DbC enforced (minimum) |
| Description | One line, what it does |
| Ecosystem link | "Part of the Simple Eiffel ecosystem" with link |
| Status | Production/Beta/Alpha/Planned |
| Overview | 2-3 sentences + code example |
| Features | Bullet list |
| Installation | Env var + ECF |
| License | MIT |

## Badge Options

Standard badges (always include):
```markdown
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Eiffel](https://img.shields.io/badge/Eiffel-25.02-blue.svg)](https://www.eiffel.org/)
[![Design by Contract](https://img.shields.io/badge/DbC-enforced-orange.svg)]()
```

Optional badges:
```markdown
[![SCOOP](https://img.shields.io/badge/SCOOP-compatible-orange.svg)]()
[![Platform](https://img.shields.io/badge/platform-Windows-blue.svg)]()
[![Tests](https://img.shields.io/badge/tests-N%20passing-brightgreen.svg)]()
```

## Logo URL

Always use the central logo:
```
https://raw.githubusercontent.com/simple-eiffel/claude_eiffel_op_docs/main/artwork/LOGO.png
```

This ensures consistent branding across all libraries.

## Documentation Link

Points to GitHub Pages:
```
https://simple-eiffel.github.io/simple_LIBNAME/
```

## Examples

See:
- [simple_toml](https://github.com/simple-eiffel/simple_toml) - Minimal/Planned library
- [simple_process](https://github.com/simple-eiffel/simple_process) - Full/Production library

## Notes

1. Keep README concise - detailed docs belong in `/docs/`
2. Code examples should be valid Eiffel
3. Status should reflect actual state
4. Update badges when status changes
