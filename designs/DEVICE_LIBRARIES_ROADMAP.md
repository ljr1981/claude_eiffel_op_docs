# Device Libraries Roadmap

## Overview

This document outlines the planned device/hardware communication libraries for the Simple Eiffel ecosystem. All libraries follow the standard simple_* patterns: Design by Contract, void safety, SCOOP compatibility, and inline C externals.

## Current State

| Library | Status | Description |
|---------|--------|-------------|
| simple_serial | Phase 1 Complete | COM port communication |
| simple_bluetooth | Phase 1 Complete | Bluetooth SPP via virtual COM |

## Planned Libraries

### Priority: HIGH

| Library | Use Cases | Target API | Research |
|---------|-----------|------------|----------|
| simple_ffmpeg | Media encoding/decoding/streaming | libav* (C library) | Complete |
| simple_usb | HID devices, custom hardware | WinUSB, SetupAPI, HID | Complete |
| simple_audio | Real-time audio I/O | WASAPI | Complete |

### Priority: MEDIUM

| Library | Use Cases | Target API | Design |
|---------|-----------|------------|--------|
| simple_camera | Webcam capture | Media Foundation | Complete |
| simple_gamepad | Game controllers | XInput/DirectInput | Complete |
| simple_printer | Direct printer control | Winspool | Complete |
| simple_gps | Location services | Windows Location API | Complete |

### Priority: LOW

| Library | Use Cases | Target API | Design |
|---------|-----------|------------|--------|
| simple_midi | Musical instruments | Windows MIDI API | Complete |
| simple_scanner | Document scanning | WIA | Complete |
| simple_smartcard | Security tokens | PC/SC (winscard) | Complete |
| simple_nfc | Contactless tags | Windows Proximity API | Complete |

## Implementation Order

1. **simple_ffmpeg** - Widest applicability, C library wrapper
2. **simple_usb** - Foundation for many hardware projects
3. **simple_audio** - Real-time audio for games/VoIP
4. **simple_camera** - Video capture
5. **simple_gamepad** - Gaming input
6. **simple_printer** - Label/thermal printers
7. **simple_gps** - Location awareness
8. **simple_midi** - Music applications
9. **simple_scanner** - Document processing
10. **simple_smartcard** - Security applications
11. **simple_nfc** - Contactless applications

## Design Principles

All device libraries follow:

1. **Facade Pattern** - One main SIMPLE_X class as entry point
2. **Builder Pattern** - Fluent configuration APIs
3. **DBC** - Full preconditions, postconditions, invariants
4. **Void Safety** - All code void-safe
5. **SCOOP Compatible** - Concurrency-ready
6. **Inline C** - Win32/C calls via Eric Bezault pattern
7. **Error Handling** - last_error query, has_error status

## Cross-Platform Strategy

Phase 1: Windows only (Win32 APIs + external C libraries)
Phase 2: Linux support where applicable
Phase 3: macOS support where applicable

## Dependencies

| Library | Dependencies |
|---------|--------------|
| simple_ffmpeg | simple_file, FFmpeg dev libs |
| simple_usb | simple_win32_api |
| simple_audio | simple_file |
| simple_camera | simple_ffmpeg (optional) |
| simple_gamepad | (none) |
| simple_printer | (none) |
| simple_gps | (none) |
| simple_midi | (none) |
| simple_scanner | (none) |
| simple_smartcard | (none) |
| simple_nfc | (none) |

## Document Index

### Research (7-Step)
- [SIMPLE_FFMPEG_RESEARCH.md](../research/SIMPLE_FFMPEG_RESEARCH.md) - Complete
- [SIMPLE_USB_RESEARCH.md](../research/SIMPLE_USB_RESEARCH.md) - Complete
- [SIMPLE_AUDIO_RESEARCH.md](../research/SIMPLE_AUDIO_RESEARCH.md) - Complete

### Designs
- [simple_ffmpeg_design.md](simple_ffmpeg_design.md) - Complete
- [simple_camera_design.md](simple_camera_design.md) - Complete
- [simple_gamepad_design.md](simple_gamepad_design.md) - Complete
- [simple_printer_design.md](simple_printer_design.md) - Complete
- [simple_gps_design.md](simple_gps_design.md) - Complete
- [simple_midi_design.md](simple_midi_design.md) - Complete
- [simple_scanner_design.md](simple_scanner_design.md) - Complete
- [simple_smartcard_design.md](simple_smartcard_design.md) - Complete
- [simple_nfc_design.md](simple_nfc_design.md) - Complete