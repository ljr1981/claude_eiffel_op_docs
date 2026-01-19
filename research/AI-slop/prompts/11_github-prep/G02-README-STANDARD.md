# G02: Standard README.md

## Objective
Create README.md following the Simple Eiffel ecosystem standard pattern.

## Prerequisites
- G01 complete (repository initialized)
- Library API documented in code

## Standard README Structure

```markdown
<p align="center">
  <img src="docs/images/logo.png" alt="{lib_name} logo" width="200">
</p>

<h1 align="center">{lib_name}</h1>

<p align="center">
  <a href="https://simple-eiffel.github.io/{lib_name}/">Documentation</a> •
  <a href="https://github.com/simple-eiffel/{lib_name}">GitHub</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License: MIT">
  <img src="https://img.shields.io/badge/Eiffel-25.02-purple.svg" alt="Eiffel 25.02">
  <img src="https://img.shields.io/badge/DBC-Contracts-green.svg" alt="Design by Contract">
</p>

**{One-line description}** — Part of the [Simple Eiffel](https://github.com/simple-eiffel) ecosystem.

## Status

✅ **Production Ready** — {version}
- {N} classes, {M} features
- {X} tests passing
- Full Design by Contract coverage

## Overview

{2-3 paragraphs describing what the library does, key features, and use cases}

## Quick Start

```eiffel
{Minimal working example showing primary use case}
```

## API Reference

### {MAIN_CLASS}

| Feature | Description |
|---------|-------------|
| `make` | {description} |
| `feature_1` | {description} |
| `feature_2` | {description} |

### {SECONDARY_CLASS}

| Feature | Description |
|---------|-------------|
| `make` | {description} |
| `feature_1` | {description} |

{Repeat for each public class}

## Features

- ✅ {Feature 1}
- ✅ {Feature 2}
- ✅ {Feature 3}
- ✅ Design by Contract throughout
- ✅ Void-safe
- ✅ SCOOP-compatible

## Installation

### Using as ECF Dependency

Add to your `.ecf` file:

```xml
<library name="{lib_name}" location="$SIMPLE_LIBS/{lib_name}/{lib_name}.ecf"/>
```

### Environment Setup

Set the `SIMPLE_LIBS` environment variable:
```bash
export SIMPLE_LIBS=/path/to/simple/libraries
```

## Dependencies

| Library | Purpose |
|---------|---------|
| {dep_1} | {why needed} |
| {dep_2} | {why needed} |

Or "None" if no external dependencies.

## License

MIT License - see [LICENSE](LICENSE) file.

---

Part of the [Simple Eiffel](https://github.com/simple-eiffel) ecosystem.
```

## Required Sections Checklist

- [ ] Logo (centered, 200px width)
- [ ] Title (h1, centered)
- [ ] Links (Documentation, GitHub)
- [ ] Badges (License, Eiffel version, DBC)
- [ ] One-line description with ecosystem link
- [ ] Status section with metrics
- [ ] Overview (2-3 paragraphs)
- [ ] Quick Start code example
- [ ] API Reference tables for each public class
- [ ] Features list with checkmarks
- [ ] Installation instructions with ECF snippet
- [ ] Dependencies table
- [ ] License section
- [ ] Footer with ecosystem link

## Logo Requirements

If no logo exists, create a simple placeholder at `docs/images/logo.png`:
- 200x200 pixels minimum
- Library name or simple icon
- Transparent background preferred
- Use "D:\prod\reference_docs\artwork\LOGO-2.png"

## Verification

- [ ] README.md exists in root directory
- [ ] All required sections present
- [ ] Code examples are syntactically correct
- [ ] API tables match actual class features
- [ ] No broken links

## Next Step
→ G03-DOCS-STANDARD.md
