# 7-Step Research: simple_* CLI Candidates

**Date:** 2026-01-16
**Author:** Claude + Larry
**Status:** Research Complete
**Selected Project:** simple_chart

---

## Executive Summary

After analyzing the 88-library simple_* ecosystem and researching modern CLI tools, we identified **7 candidates** that:
- Fill genuine gaps in the ecosystem
- Have practical CLI app use cases
- Are technically interesting to build
- Leverage DBC/SCOOP advantages
- Build on existing simple_* dependencies

---

## Current Ecosystem Analysis (88 Libraries)

### Coverage by Domain

**CORE/FOUNDATION:**
simple_base64, simple_codec, simple_compression, simple_encryption,
simple_hash, simple_uuid, simple_datetime, simple_decimal, simple_fraction,
simple_math, simple_randomizer, simple_regex, simple_validation

**DATA FORMATS:**
simple_json, simple_xml, simple_yaml, simple_toml, simple_csv, simple_markdown

**FILE/SYSTEM:**
simple_file, simple_archive, simple_config, simple_env, simple_mmap,
simple_system, simple_watcher, simple_clipboard, simple_cache

**NETWORKING:**
simple_http, simple_websocket, simple_smtp, simple_grpc, simple_mq,
simple_cors, simple_rate_limiter

**DATABASE:**
simple_sql (SQLite), simple_mongo (planned)

**PROCESS/IPC:**
simple_process, simple_ipc, simple_serial, simple_usb, simple_bluetooth

**GRAPHICS/MEDIA:**
simple_cairo, simple_pdf, simple_stb (images), simple_ffmpeg, simple_audio,
simple_minifb, simple_vulkan, simple_shaderc, simple_sdf, simple_vision

**UI:**
simple_console, simple_tui, simple_alpine, simple_htmx, simple_browser,
simple_gui_designer (abandoned)

**AI/ML:**
simple_ai_client (Claude, Grok, Gemini, Ollama), simple_speech (STT/TTS)

**DEV TOOLS:**
simple_eiffel_parser, simple_kb, simple_oracle, simple_code (codegen),
simple_dev, simple_lsp, simple_testing, simple_logger, simple_telemetry

**DEVOPS/CLOUD:**
simple_docker, simple_k8s, simple_ci, simple_github_runner

**WEB:**
simple_web, simple_template, simple_jwt, simple_social

**MISC:**
simple_graph, simple_scheduler, simple_i18n, simple_setup, simple_pkg,
simple_registry, simple_showcase, simple_notebook, simple_rosetta

### Notable Gaps Identified

- No database besides SQLite (Postgres, Redis, etc.)
- No email parsing (have SMTP sending only)
- No PDF reading (have Cairo PDF writing)
- No spreadsheet/Excel
- No barcode/QR code
- No OCR
- No network diagnostics (ping, traceroute, etc.)
- No diff/patch utilities
- No terminal multiplexer
- No backup/sync utilities
- No screenshot/screen recording
- No system monitoring/metrics
- **No terminal data visualization/charting**

---

## Evaluation Matrix

| Candidate | Gap Filled | Fun Factor | DBC Value | Dependencies | Complexity |
|-----------|------------|------------|-----------|--------------|------------|
| **simple_qr** | QR/Barcode | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | stb, console | Medium |
| **simple_trippy** | Network diag | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | tui, process | High |
| **simple_chart** | Terminal viz | ⭐⭐⭐⭐ | ⭐⭐⭐ | console, csv | Medium |
| **simple_secrets** | Security scan | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | regex, file | Medium |
| **simple_bench** | Benchmarking | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | process, math | Low-Med |
| **simple_diff** | Diff/Patch | ⭐⭐⭐ | ⭐⭐⭐⭐ | file, console | Medium |
| **simple_sync** | Backup/Sync | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | file, hash, compression | High |

---

## All 7 Candidates

### 1. simple_qr - QR Code Generator/Scanner
**Inspired by:** qrrs, qrterminal

**What it does:**
- Generate QR codes rendered directly in terminal (Unicode half-blocks)
- Generate QR to PNG/SVG files
- Scan QR from image files
- Encode: URLs, WiFi configs, vCards, plain text

**CLI Example:**
```bash
simple_qr encode "https://simple-eiffel.github.io" --terminal
simple_qr encode --wifi "MyNetwork:WPA:password123" -o wifi.png
simple_qr decode screenshot.png
```

**Why it's interesting:**
- Visual output in terminal is satisfying
- Algorithmic challenge (Reed-Solomon encoding)
- Immediate practical use (share URLs, WiFi)

**Dependencies:** simple_stb (image I/O), simple_console

---

### 2. simple_trippy - Network Diagnostic Tool
**Inspired by:** Trippy, MTR, WinMTR

**What it does:**
- Combined ping + traceroute in one TUI
- Live-updating hop statistics (latency, packet loss)
- Multiple protocols (ICMP, UDP, TCP)
- Export results to JSON/CSV

**CLI Example:**
```bash
simple_trippy google.com           # Live TUI display
simple_trippy --report 8.8.8.8     # Text report
simple_trippy -p tcp -P 443 site   # TCP mode on port 443
```

**Why it's interesting:**
- Raw socket programming (ICMP packets)
- Real-time TUI with simple_tui
- Actually useful for debugging network issues
- SCOOP for concurrent probes

**Dependencies:** simple_tui, simple_process (or inline C for raw sockets)

---

### 3. simple_chart - Terminal Data Visualization ⬅️ SELECTED
**Inspired by:** termgraph, YouPlot, datadash

**What it does:**
- Pipe data in, get charts out
- Bar charts, line graphs, scatter plots, histograms
- Sparklines for inline visualization
- Braille-mode for high resolution

**CLI Example:**
```bash
cat sales.csv | simple_chart bar --title "Q4 Sales"
simple_chart line data.json --x date --y value
echo "1 5 3 9 2 7" | simple_chart spark
```

**Why it's interesting:**
- Unicode/Braille rendering algorithms
- Automatic scaling and axis generation
- Pairs well with simple_csv, simple_json

**Dependencies:** simple_console, simple_csv, simple_json

---

### 4. simple_secrets - Secret Detection Scanner
**Inspired by:** TruffleHog, Gitleaks, GitGuardian

**What it does:**
- Scan files/directories for leaked secrets
- Detect API keys, passwords, tokens, private keys
- Entropy analysis for unknown secret patterns
- Git history scanning
- Pre-commit hook integration

**CLI Example:**
```bash
simple_secrets scan ./src            # Scan directory
simple_secrets scan --git .          # Include git history
simple_secrets verify .env           # Check specific file
simple_secrets hook install          # Pre-commit hook
```

**Why it's interesting:**
- Security is always engaging
- Pattern matching + entropy = interesting combo
- DBC shines: contracts ensure complete pattern coverage
- Real-world value (prevent embarrassing leaks)

**Dependencies:** simple_regex, simple_file, simple_hash, simple_process (for git)

---

### 5. simple_bench - Command Benchmarking
**Inspired by:** hyperfine

**What it does:**
- Run commands multiple times, calculate statistics
- Warm-up runs to prime caches
- Statistical analysis (mean, stddev, min, max, percentiles)
- Compare multiple commands
- Export results

**CLI Example:**
```bash
simple_bench "sleep 0.5"                    # Basic benchmark
simple_bench -w 3 -r 10 "my_program"        # 3 warmup, 10 runs
simple_bench "grep pattern" "rg pattern"    # Compare two
simple_bench --export json results.json
```

**Why it's interesting:**
- Statistical algorithms
- Process spawning and timing
- Useful for optimizing your own tools
- Clean, focused scope

**Dependencies:** simple_process, simple_math, simple_console

---

### 6. simple_diff - Diff and Patch Tool
**Inspired by:** delta, colordiff

**What it does:**
- Compare files with syntax highlighting
- Generate unified/context diffs
- Apply patches
- Side-by-side view
- Word-level diff highlighting

**CLI Example:**
```bash
simple_diff old.e new.e                # Show diff
simple_diff -u old.e new.e > fix.patch # Generate patch
simple_diff --apply fix.patch          # Apply patch
simple_diff --side-by-side a.txt b.txt
```

**Why it's interesting:**
- Classic algorithms (LCS, Myers diff)
- Syntax-aware diffing for Eiffel files
- Integration with simple_eiffel_parser for semantic diffs

**Dependencies:** simple_file, simple_console, simple_eiffel_parser (optional)

---

### 7. simple_sync - File Sync with Deduplication
**Inspired by:** BorgBackup, Rclone, zbackup

**What it does:**
- Sync directories with deduplication
- Content-defined chunking (only store unique blocks)
- Incremental backups
- Checksum verification
- Compression and encryption

**CLI Example:**
```bash
simple_sync init backup_repo            # Initialize repo
simple_sync backup ./projects backup_repo
simple_sync restore backup_repo::latest ./restore
simple_sync list backup_repo
simple_sync verify backup_repo
```

**Why it's interesting:**
- Rolling hash algorithms (Rabin fingerprinting)
- Content-addressed storage
- DBC ensures data integrity at every step
- Chunking algorithms are fascinating

**Dependencies:** simple_file, simple_hash, simple_compression, simple_encryption

---

## Research Sources

- [12 CLI Tools That Are Redefining Developer Workflows](https://www.qodo.ai/blog/best-cli-tools/)
- [9 Modern CLI Tools You Should Try in 2026](https://medium.com/the-software-journal/9-modern-cli-tools-you-should-try-in-2026-9c37c86ffea0)
- [14 Rust-based Alternative CLI Tools](https://itsfoss.com/rust-alternative-cli-tools/)
- [Trippy Network Diagnostic Tool](https://trippy.rs/)
- [qrrs - CLI QR code generator in Rust](https://github.com/Lenivaya/qrrs)
- [termgraph - Terminal graphs in Python](https://github.com/mkaz/termgraph)
- [YouPlot - Terminal plotting in Ruby](https://github.com/red-data-tools/YouPlot)
- [datadash - Terminal data visualization in Go](https://github.com/keithknott26/datadash)
- [Best Secret Scanning Tools in 2025](https://www.aikido.dev/blog/top-secret-scanning-tools)
- [BorgBackup](https://www.borgbackup.org/)
- [Rclone](https://rclone.org/)

---

*End of Research Report*
