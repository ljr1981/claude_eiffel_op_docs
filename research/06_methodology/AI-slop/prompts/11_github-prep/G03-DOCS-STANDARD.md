# G03: Standard /docs Folder

## Objective
Create the standard documentation folder for GitHub Pages hosting.

## Prerequisites
- G01, G02 complete
- Library API known

## Directory Structure

```
docs/
├── index.html          # Main documentation page
├── css/
│   └── style.css       # Shared stylesheet
└── images/
    └── logo.png        # Library logo
```

## index.html Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{lib_name} - Simple Eiffel Library</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <header>
        <div class="container">
            <img src="images/logo.png" alt="{lib_name}" class="logo">
            <h1>{lib_name}</h1>
            <p class="tagline">{One-line description}</p>
            <div class="badges">
                <img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="MIT License">
                <img src="https://img.shields.io/badge/Eiffel-25.02-purple.svg" alt="Eiffel 25.02">
                <img src="https://img.shields.io/badge/DBC-Contracts-green.svg" alt="Design by Contract">
            </div>
        </div>
    </header>

    <nav>
        <div class="container">
            <a href="#overview">Overview</a>
            <a href="#quick-start">Quick Start</a>
            <a href="#features">Features</a>
            <a href="#api-reference">API Reference</a>
            <a href="https://github.com/simple-eiffel/{lib_name}">GitHub</a>
        </div>
    </nav>

    <main>
        <div class="container">
            <section id="overview">
                <h2>Overview</h2>
                <p>{Overview paragraph 1}</p>
                <p>{Overview paragraph 2}</p>
            </section>

            <section id="quick-start">
                <h2>Quick Start</h2>
                <pre><code class="language-eiffel">{Quick start code example}</code></pre>
            </section>

            <section id="features">
                <h2>Features</h2>
                <ul>
                    <li>{Feature 1}</li>
                    <li>{Feature 2}</li>
                    <li>{Feature 3}</li>
                    <li>Design by Contract throughout</li>
                    <li>Void-safe implementation</li>
                    <li>SCOOP-compatible</li>
                </ul>
            </section>

            <section id="api-reference">
                <h2>API Reference</h2>

                <h3>{MAIN_CLASS}</h3>
                <p>{Class description}</p>
                <table>
                    <thead>
                        <tr>
                            <th>Feature</th>
                            <th>Description</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td><code>make</code></td>
                            <td>{description}</td>
                        </tr>
                        <tr>
                            <td><code>feature_1</code></td>
                            <td>{description}</td>
                        </tr>
                    </tbody>
                </table>

                <!-- Repeat for each public class -->
            </section>

            <section id="installation">
                <h2>Installation</h2>
                <p>Add to your ECF file:</p>
                <pre><code class="language-xml">&lt;library name="{lib_name}" location="$SIMPLE_LIBS/{lib_name}/{lib_name}.ecf"/&gt;</code></pre>
            </section>
        </div>
    </main>

    <footer>
        <div class="container">
            <p>&copy; 2026 Simple Eiffel. <a href="https://github.com/simple-eiffel">GitHub</a></p>
        </div>
    </footer>
</body>
</html>
```

## css/style.css Template

```css
:root {
    --primary-color: #6B46C1;
    --secondary-color: #805AD5;
    --bg-color: #FFFFFF;
    --text-color: #1A202C;
    --code-bg: #F7FAFC;
    --border-color: #E2E8F0;
}

@media (prefers-color-scheme: dark) {
    :root {
        --bg-color: #1A202C;
        --text-color: #F7FAFC;
        --code-bg: #2D3748;
        --border-color: #4A5568;
    }
}

* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    line-height: 1.6;
    color: var(--text-color);
    background-color: var(--bg-color);
}

.container {
    max-width: 900px;
    margin: 0 auto;
    padding: 0 20px;
}

header {
    background: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
    color: white;
    padding: 60px 0;
    text-align: center;
}

header .logo {
    width: 120px;
    height: auto;
    margin-bottom: 20px;
}

header h1 {
    font-size: 2.5rem;
    margin-bottom: 10px;
}

header .tagline {
    font-size: 1.2rem;
    opacity: 0.9;
    margin-bottom: 20px;
}

header .badges img {
    margin: 0 5px;
}

nav {
    background-color: var(--code-bg);
    border-bottom: 1px solid var(--border-color);
    position: sticky;
    top: 0;
    z-index: 100;
}

nav .container {
    display: flex;
    gap: 20px;
    padding: 15px 20px;
    overflow-x: auto;
}

nav a {
    color: var(--text-color);
    text-decoration: none;
    white-space: nowrap;
}

nav a:hover {
    color: var(--primary-color);
}

main {
    padding: 40px 0;
}

section {
    margin-bottom: 60px;
}

h2 {
    color: var(--primary-color);
    border-bottom: 2px solid var(--primary-color);
    padding-bottom: 10px;
    margin-bottom: 20px;
}

h3 {
    color: var(--secondary-color);
    margin: 30px 0 15px;
}

pre {
    background-color: var(--code-bg);
    border: 1px solid var(--border-color);
    border-radius: 8px;
    padding: 20px;
    overflow-x: auto;
    margin: 15px 0;
}

code {
    font-family: "Fira Code", "Consolas", monospace;
    font-size: 0.9rem;
}

table {
    width: 100%;
    border-collapse: collapse;
    margin: 15px 0;
}

th, td {
    padding: 12px;
    text-align: left;
    border-bottom: 1px solid var(--border-color);
}

th {
    background-color: var(--code-bg);
    font-weight: 600;
}

ul {
    margin: 15px 0;
    padding-left: 25px;
}

li {
    margin: 8px 0;
}

footer {
    background-color: var(--code-bg);
    border-top: 1px solid var(--border-color);
    padding: 30px 0;
    text-align: center;
}

footer a {
    color: var(--primary-color);
}
```

## Logo Placeholder

If no custom logo exists, create a simple SVG placeholder:

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200" viewBox="0 0 200 200">
  <rect fill="#6B46C1" width="200" height="200" rx="20"/>
  <text x="100" y="110" font-family="Arial, sans-serif" font-size="24" fill="white" text-anchor="middle" font-weight="bold">{LIB}</text>
</svg>
```

Save as `docs/images/logo.png` (convert SVG to PNG if needed) or use as `logo.svg`.

## Verification Checklist

- [ ] docs/ directory exists
- [ ] docs/index.html exists with all sections
- [ ] docs/css/style.css exists
- [ ] docs/images/logo.png exists (or logo.svg)
- [ ] All placeholders replaced with actual content
- [ ] Links work correctly
- [ ] Page renders correctly in browser

## GitHub Pages Setup

After pushing, enable GitHub Pages:
1. Go to repository Settings
2. Pages section
3. Source: Deploy from branch
4. Branch: main, folder: /docs
5. Save

Site will be available at: https://simple-eiffel.github.io/{lib_name}/

## Next Step
→ G04-FINAL-CHECKLIST.md
