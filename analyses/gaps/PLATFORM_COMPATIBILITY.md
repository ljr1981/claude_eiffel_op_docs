# Simple Eiffel Ecosystem - Platform Compatibility

**Last Updated:** December 10, 2025

This document lists all simple_* libraries and their platform support status.

## Legend

| Symbol | Meaning |
|--------|---------|
| ✅ | Fully supported |
| 🔧 | Planned/In progress |
| ❌ | Not applicable (platform-specific by design) |
| ⚠️ | Partial support |

## Platform Compatibility Matrix

### Core Libraries (Cross-Platform)

These libraries use only pure Eiffel or standard ISE libraries:

| Library | Windows | Linux | macOS | Notes |
|---------|---------|-------|-------|-------|
| simple_base64 | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_cache | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_csv | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_datetime | ✅ | ✅ | ✅ | Uses ISE TIME |
| simple_hash | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_json | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_jwt | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_logger | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_markdown | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_randomizer | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_template | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_testing | ✅ | ✅ | ✅ | Uses ISE TESTING |
| simple_uuid | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_validation | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_xml | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_eiffel_parser | ✅ | ✅ | ✅ | Pure Eiffel |

### Web/Network Libraries

| Library | Windows | Linux | macOS | Notes |
|---------|---------|-------|-------|-------|
| simple_alpine | ✅ | ✅ | ✅ | Pure Eiffel (generates JS) |
| simple_cors | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_htmx | ✅ | ✅ | ✅ | Pure Eiffel (generates HTML) |
| simple_http | ✅ | 🔧 | 🔧 | Uses WinHTTP, needs libcurl |
| simple_rate_limiter | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_smtp | ✅ | 🔧 | 🔧 | Uses Winsock |
| simple_web | ✅ | ✅ | ✅ | Uses EWF (cross-platform) |
| simple_websocket | ✅ | 🔧 | 🔧 | Uses Winsock |

### Database Libraries

| Library | Windows | Linux | macOS | Notes |
|---------|---------|-------|-------|-------|
| simple_sql | ✅ | ✅ | ✅ | SQLite (cross-platform) |
| simple_mongo | ✅ | 🔧 | 🔧 | Needs MongoDB C driver |

### System Libraries (Windows-First)

These libraries use Win32 APIs and need platform-specific implementations:

| Library | Windows | Linux | macOS | Notes |
|---------|---------|-------|-------|-------|
| simple_clipboard | ✅ | 🔧 | 🔧 | Win32 clipboard API |
| simple_console | ✅ | 🔧 | 🔧 | Win32 console API |
| simple_env | ✅ | 🔧 | 🔧 | Win32 environment |
| simple_file | ✅ | ⚠️ | ⚠️ | Some Win32 features |
| simple_ipc | ✅ | 🔧 | 🔧 | Named pipes (Win32) |
| simple_mmap | ✅ | 🔧 | 🔧 | Memory-mapped files |
| simple_process | ✅ | 🔧 | 🔧 | CreateProcess vs fork |
| simple_registry | ✅ | ❌ | ❌ | Windows Registry only |
| simple_system | ✅ | 🔧 | 🔧 | System info APIs |
| simple_watcher | ✅ | 🔧 | 🔧 | ReadDirectoryChangesW |
| simple_win32_api | ✅ | ❌ | ❌ | Windows-specific by design |

### Application Libraries

| Library | Windows | Linux | macOS | Notes |
|---------|---------|-------|-------|-------|
| simple_ai_client | ✅ | 🔧 | 🔧 | HTTP dependency |
| simple_app_api | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_archive | ✅ | 🔧 | 🔧 | Compression library deps |
| simple_ci | ✅ | 🔧 | 🔧 | Process dependency |
| simple_cli | ✅ | 🔧 | 🔧 | Console dependency |
| simple_compression | ✅ | 🔧 | 🔧 | zlib binding |
| simple_config | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_encryption | ✅ | 🔧 | 🔧 | Win32 crypto or OpenSSL |
| simple_foundation_api | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_gui_designer | ✅ | 🔧 | 🔧 | Platform UI |
| simple_i18n | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_lsp | ✅ | 🔧 | 🔧 | Process/file deps |
| simple_oracle | ✅ | 🔧 | 🔧 | SQLite + process deps |
| simple_pdf | ✅ | 🔧 | 🔧 | Needs library bindings |
| simple_regex | ✅ | 🔧 | 🔧 | Uses PCRE |
| simple_service_api | ✅ | ✅ | ✅ | Pure Eiffel |
| simple_setup | ✅ | ❌ | ❌ | Inno Setup (Windows installer) |
| simple_showcase | ✅ | ✅ | ✅ | Pure Eiffel |

## Summary

| Category | Count | Cross-Platform | Windows-Only | Needs Porting |
|----------|-------|----------------|--------------|---------------|
| Core | 16 | 16 | 0 | 0 |
| Web/Network | 8 | 5 | 0 | 3 |
| Database | 2 | 1 | 0 | 1 |
| System | 11 | 0 | 2 | 9 |
| Application | 18 | 6 | 1 | 11 |
| **Total** | **55** | **28** | **3** | **24** |

## Cross-Platform Roadmap

Priority libraries for Linux/macOS support:

1. **simple_process** - Foundation for many other libraries
2. **simple_file** - Complete POSIX implementation
3. **simple_console** - ANSI terminal support
4. **simple_lsp** - Eiffel community benefit
5. **simple_http** - libcurl backend

## Notes

- **Windows-First Development:** We develop on Windows (Larry's environment), so Windows support comes first
- **SCOOP Compatibility:** All libraries are SCOOP-compatible regardless of platform
- **Inline C Pattern:** We use inline C pattern for native code - this makes porting easier as all platform-specific code is in Eiffel files
- **Community Contributions:** Linux/macOS implementations welcome!

## Platform-Specific Libraries

These libraries are intentionally Windows-only:

- **simple_registry** - Windows Registry has no Unix equivalent
- **simple_win32_api** - Low-level Win32 bindings
- **simple_setup** - Inno Setup installer generation

---

*Generated for the Simple Eiffel ecosystem*
*https://github.com/simple-eiffel*
