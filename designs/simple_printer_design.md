# simple_printer Design

## Overview

**simple_printer** provides direct printer control for Eiffel applications, including thermal and label printers.

## API

| Class | Purpose |
|-------|---------|
| SIMPLE_PRINTER | Main facade |
| PRINTER_DEVICE | Printer info |
| PRINT_JOB | Print job |
| PRINT_DOCUMENT | Document to print |

## Key Features

```eiffel
-- Enumerate printers
across printer.devices as p loop
    print (p.name + " [" + p.status.out + "]")
end

-- Print text
if attached printer.default as p then
    p.print_text ("Hello World")
end

-- Print document
job := printer.create_job ("My Document")
job.add_text ("Line 1")
job.add_text ("Line 2")
job.send

-- Thermal printer (ESC/POS)
thermal := printer.open_thermal ("COM3")
thermal.print_text ("Receipt Line 1")
thermal.print_barcode ("1234567890")
thermal.cut
```

## Printer Types

| Type | Protocol | Use Case |
|------|----------|----------|
| Standard | Windows GDI | Document printing |
| Thermal | ESC/POS | Receipts, labels |
| Label | ZPL/EPL | Shipping labels |
| Raw | Direct | Custom protocols |

## Windows API

- Winspool: `OpenPrinter`, `StartDocPrinter`, `WritePrinter`
- GDI: `CreateDC`, `StartDoc`, `TextOut`

## Phase 1 Scope

1. Enumerate printers
2. Get printer status
3. Print text
4. Raw printing
5. ESC/POS support

## Dependencies

- winspool.lib
