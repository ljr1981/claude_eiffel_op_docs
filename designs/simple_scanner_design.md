# simple_scanner Design

## Overview

**simple_scanner** provides document scanning for Eiffel applications using WIA (Windows Image Acquisition).

## API

| Class | Purpose |
|-------|---------|
| SIMPLE_SCANNER | Main facade |
| SCANNER_DEVICE | Scanner info |
| SCAN_SETTINGS | Scan configuration |
| SCANNED_IMAGE | Result image |

## Key Features

```eiffel
-- Enumerate scanners
across scanner.devices as s loop
    print (s.name)
end

-- Quick scan
if attached scanner.scan as image then
    image.save ("document.jpg")
end

-- Custom settings
settings := scanner.create_settings
settings.set_resolution (300)  -- DPI
settings.set_color_mode (Color_grayscale)
settings.set_area (0, 0, 8.5, 11)  -- inches

if attached scanner.scan_with_settings (settings) as image then
    image.save_as_pdf ("document.pdf")
end
```

## Scan Settings

| Property | Values |
|----------|--------|
| resolution | 75, 150, 300, 600 DPI |
| color_mode | Color, Grayscale, B&W |
| paper_size | Letter, A4, Legal, Custom |
| source | Flatbed, Feeder |

## Windows API

- WIA 2.0: `IWiaDevMgr2`, `IWiaItem2`
- TWAIN (legacy)

## Phase 1 Scope

1. Enumerate scanners
2. Basic scan
3. Resolution settings
4. Color mode
5. Save to file

## Dependencies

- wiaguid.lib
