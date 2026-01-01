# Third-Party Attribution

The Simple Eiffel ecosystem gratefully acknowledges the following third-party projects, libraries, and standards that make this work possible.

---

## Core Dependencies

### Eiffel Software (ISE)
- **EiffelStudio Compiler** - The Eiffel compiler and IDE
- **ISE Base Library** - Core Eiffel classes
- **ISE Time Library** - Date/time handling
- **ISE Net Library** - Network primitives
- **ISE cURL Wrapper** - HTTP client foundation (used by simple_http)
- **ISE JSON Library** - JSON parsing foundation
- **EiffelWeb Framework (EWF)** - Web server foundation (used by simple_web)
- **License**: Eiffel Forum License v2 / GPL
- **Website**: https://www.eiffel.com

### Gobo Eiffel
- **Gobo Kernel** - Core utilities
- **Gobo Regexp** - Regular expression engine (used by simple_regex)
- **Gobo Parse/Lex** - Parser generators (used by simple_eiffel_parser)
- **Gobo Tools** - Eiffel source parsing
- **Author**: Eric Bezault
- **License**: MIT
- **Website**: https://github.com/gobo-eiffel/gobo

---

## External Libraries

### SQLite
- **Used by**: simple_sql, simple_kb
- **Via**: eiffel_sqlite_2025 (Simple Eiffel SQLite wrapper)
- **License**: Public Domain
- **Website**: https://sqlite.org

### FFmpeg
- **Used by**: simple_ffmpeg
- **Purpose**: Video/audio transcoding, metadata extraction
- **License**: LGPL 2.1+ or GPL 2+
- **Website**: https://ffmpeg.org

### Redis
- **Used by**: simple_cache (optional), simple_mq (optional)
- **Purpose**: Distributed caching and message queues
- **License**: BSD 3-Clause
- **Website**: https://redis.io

---

## Windows APIs

The following Microsoft Windows APIs are used via inline C externals:

### Windows System
- **Kernel32** - Process, file, memory management
- **User32** - Window management, input
- **Advapi32** - Security, registry
- **Shell32** - Shell operations
- **Ole32/OleAut32** - COM support

### Device APIs
- **SetupAPI** - USB device enumeration (simple_usb)
- **hid.lib** - HID device access (simple_usb)
- **WASAPI** - Audio I/O (simple_audio)
- **Win32 Serial** - COM port communication (simple_serial)
- **Bluetooth APIs** - SPP communication (simple_bluetooth)

### Cryptography
- **BCrypt** - Modern crypto API (simple_encryption)
- **CryptoAPI** - Legacy crypto support

---

## Standards and Specifications

### RFCs
- **RFC 7519** - JSON Web Token (JWT) - simple_jwt
- **RFC 4180** - CSV format - simple_csv
- **RFC 3986** - URI syntax - simple_http
- **RFC 6455** - WebSocket protocol - simple_websocket
- **RFC 5321** - SMTP - simple_smtp

### Other Standards
- **YAML 1.2** - simple_yaml
- **TOML v1.0** - simple_toml
- **JSON Schema Draft-07** - simple_json
- **ISO 8601** - Date/time formatting - simple_datetime
- **Unicode** - Text handling throughout

---

## Acknowledgments

Special thanks to:

- **Bertrand Meyer** - Creator of Eiffel and Design by Contract
- **Eiffel Software** - For EiffelStudio and ongoing Eiffel development
- **Eric Bezault** - For Gobo Eiffel and valuable feedback on this ecosystem
- **Javier Velilla** - For EiffelWeb Framework and community contributions
- **The Eiffel Community** - For decades of language evolution and support

---

## License Notes

- All Simple Eiffel libraries are released under **MIT License**
- Third-party dependencies retain their original licenses
- When using simple_ffmpeg with FFmpeg SDK, ensure GPL/LGPL compliance
- SQLite is public domain - no attribution legally required, but given gratefully

---

*Last updated: December 2024*
