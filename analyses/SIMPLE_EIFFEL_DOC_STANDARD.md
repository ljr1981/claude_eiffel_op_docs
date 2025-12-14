# Simple Eiffel Documentation Standard

This document defines the standard format and structure for all `simple_*` library documentation.

## index.html Standard Format

All library `docs/index.html` files MUST follow this template:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>simple_LIBNAME - Short Description for Eiffel</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <header>
        <div class="logo-container">
            <img src="https://raw.githubusercontent.com/simple-eiffel/claude_eiffel_op_docs/main/artwork/LOGO.png" alt="simple_* logo" class="logo">
        </div>
        <h1>simple_LIBNAME</h1>
        <p class="tagline">Short Description for Eiffel</p>
        <div class="badges">
            <span class="badge badge-version">v1.0.0</span>
            <span class="badge badge-license">MIT</span>
        </div>
    </header>

    <nav>
        <ul>
            <li><a href="#overview">Overview</a></li>
            <li><a href="user-guide.html">User Guide</a></li>
            <li><a href="api-reference.html">API Reference</a></li>
            <li><a href="architecture.html">Architecture</a></li>
            <li><a href="cookbook.html">Cookbook</a></li>
            <li><a href="https://github.com/simple-eiffel/simple_LIBNAME">GitHub</a></li>
        </ul>
    </nav>

    <main>
        <!-- Content sections -->
    </main>

    <footer>
        <p>&copy; 2025 simple_* ecosystem. MIT License.</p>
        <p>
            <a href="https://github.com/simple-eiffel/simple_LIBNAME">GitHub</a> |
            <a href="https://simple-eiffel.github.io">Documentation</a>
        </p>
    </footer>
</body>
</html>
```

## Key Requirements

### 1. Logo - Use Local Image
```html
<img src="images/logo.png" alt="simple_* logo" class="logo">
```

Each library should have `docs/images/logo.png` with the simple_* logo.

### 2. Badges
```html
<div class="badges">
    <span class="badge badge-version">v1.0.0</span>
    <span class="badge badge-license">MIT</span>
</div>
```

Only version and license badges. No "DbC Enforced" or other badges.

### 3. Navigation
- First link: `#overview` (anchor on same page)
- User Guide, API Reference, Architecture, Cookbook: Link to separate files
- GitHub: External link to repository

```html
<nav>
    <ul>
        <li><a href="#overview">Overview</a></li>
        <li><a href="user-guide.html">User Guide</a></li>
        <li><a href="api-reference.html">API Reference</a></li>
        <li><a href="architecture.html">Architecture</a></li>
        <li><a href="cookbook.html">Cookbook</a></li>
        <li><a href="https://github.com/simple-eiffel/simple_LIBNAME">GitHub</a></li>
    </ul>
</nav>
```

### 4. Footer - Standard Format
```html
<footer>
    <p>&copy; 2025 simple_* ecosystem. MIT License.</p>
    <p>
        <a href="https://github.com/simple-eiffel/simple_LIBNAME">GitHub</a> |
        <a href="https://simple-eiffel.github.io">Documentation</a>
    </p>
</footer>
```

### 5. Ecosystem Link in Overview
Always include this line in overview:
```html
<p>Part of the <a href="https://github.com/simple-eiffel">Simple Eiffel</a> ecosystem.</p>
```

## 5-Document IUARC Standard

For libraries with full documentation:

| File | Purpose |
|------|---------|
| `index.html` | Overview, Quick Start, Features, Documentation links |
| `user-guide.html` | Complete tutorial with examples |
| `api-reference.html` | Full API with contracts |
| `architecture.html` | Internal design and rationale |
| `cookbook.html` | Ready-to-use recipes and patterns |

## CSS File

All libraries use `docs/css/style.css` for styling. Standard classes:
- `.feature-grid`, `.feature-card` - Feature showcase
- `.api-table` - API tables
- `.doc-links` - Documentation link list
- `.keyword`, `.type`, `.string`, `.comment` - Code highlighting
- `.badge`, `.badge-version`, `.badge-license` - Header badges

## Reference Examples

Good examples to follow:
- `simple_toml/docs/index.html` - Clean, minimal format
- `simple_process/docs/index.html` - With feature grid
- `simple_web/docs/index.html` - Full IUARC with all 5 docs

## DO NOT

- Add extra badges like "DbC Enforced"
- Use `index.html` in nav instead of `#overview`
- Use different footer format
- Omit the ecosystem link
