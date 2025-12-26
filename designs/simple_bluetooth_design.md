# simple_bluetooth Design Report

## Executive Summary

**Recommendation: PROCEED WITH CAUTION - Phased Approach**

Based on comprehensive 7-step research, a Bluetooth library for Eiffel is feasible but presents significant challenges. The recommended approach is a phased implementation starting with the most practical use case (Serial Port Profile via COM ports), with optional BLE support via Windows.Devices.Bluetooth API.

## Research Findings

### Step 1: Specifications Review

**Bluetooth Core Specification 5.4** (February 2023) is the current standard from Bluetooth SIG. Key aspects:

1. **Bluetooth Classic (BR/EDR)** - Traditional Bluetooth for audio, file transfer, serial communication
2. **Bluetooth Low Energy (BLE)** - Low-power protocol for IoT, sensors, beacons
3. **Key Profiles:**
   - **SPP (Serial Port Profile)** - Emulates RS-232 serial connection via RFCOMM (most practical for Eiffel)
   - **GATT (Generic Attribute Profile)** - BLE data exchange via Services/Characteristics
   - **A2DP** - Audio streaming
   - **HID** - Human Interface Devices (keyboards, mice)

**Critical Finding:** SPP is NOT supported on iOS without MFi certification. BLE (GATT) is the only universal mobile option.

### Step 2: Modern Library Analysis

| Language | Library | Focus | Status |
|----------|---------|-------|--------|
| Python | **Bleak** | BLE only | Active, async, cross-platform |
| Python | PyBluez | Classic + BLE | Unmaintained, C bindings required |
| Rust | btleplug | BLE only | Active, Windows/macOS/Linux/iOS/Android |
| Go | tinygo-org/bluetooth | BLE | Experimental on Windows |
| C#/.NET | Windows.Devices.Bluetooth | Both | Windows 10+ UWP/WinRT |

**Key Insight:** Nearly all modern libraries focus on BLE only. Bluetooth Classic support is fragmented and platform-specific.

### Step 3: Eiffel Ecosystem

**Finding: NO existing Bluetooth library for Eiffel.**

Related simple_* libraries that could be leveraged:
- **simple_ipc** - Named pipes, Unix sockets (no serial port support currently)
- **simple_http** - HTTP client patterns
- **simple_process** - Process execution for shell-out approach

Options for implementation:
1. **WrapC** - Eiffel wrapper generator for C libraries (could wrap BlueZ on Linux)
2. **Inline C** - Eric Bezault pattern for Win32 API calls
3. **Shell out** - Execute platform tools like `bluetoothctl`

### Step 4: Developer Pain Points

From research on developer forums and articles:

1. **Platform API Fragmentation** - Each OS has completely different Bluetooth APIs
2. **Windows BLE Pairing UX** - Users must pair via Settings before app can connect (unlike macOS/iOS)
3. **Android Device Fragmentation** - Chipset/firmware variations cause inconsistent behavior
4. **Background Processing** - Maintaining connections while app is backgrounded is complex
5. **No Pure Cross-Platform Solution** - Every library requires platform-specific code
6. **Evolving Stacks** - Bluetooth implementations change with OS updates
7. **Learning Curve** - Terms like CBPeripheral, CBCentralManager, GATT, UUIDs confuse newcomers

### Step 5: Innovation Opportunities

For Eiffel's simple_bluetooth:

1. **Serial-First Design** - Focus on SPP-over-COM as the primary use case (works on Windows/Linux)
2. **DBC for Connection State** - Contracts that enforce proper connection lifecycle
3. **Fluent API** - `bluetooth.discover.filter_by_name ("Device").connect.send ("data")`
4. **Abstracted Profiles** - Hide GATT complexity behind domain-specific classes
5. **SCOOP Integration** - Separate processor for async Bluetooth operations
6. **Automatic Reconnection** - Built-in retry logic with configurable backoff

### Step 6: Design Strategy

#### Recommended Phased Approach:

**Phase 1: Serial Port Bridge (Priority)**
```
simple_bluetooth/
  src/
    serial/
      bt_serial_port.e       -- Windows COM port / Linux rfcomm
      bt_serial_connection.e -- Read/write bytes
    simple_bluetooth.e       -- Facade
```
- Uses existing OS serial port APIs
- Requires pre-pairing via OS settings
- Works with SPP-compatible devices (Arduino, HC-05, etc.)

**Phase 2: Windows BLE (Optional)**
```
    ble/
      bt_le_scanner.e        -- Device discovery
      bt_le_device.e         -- BLE device connection
      bt_gatt_service.e      -- GATT service wrapper
      bt_gatt_characteristic.e
```
- Wraps Windows.Devices.Bluetooth via inline C
- Windows 10+ only

**Phase 3: Cross-Platform BLE (Future)**
- Would require wrapping BlueZ (Linux), CoreBluetooth (macOS)
- Significant complexity

#### Architecture Decision: Serial Port First

```eiffel
class BLUETOOTH_SERIAL_PORT

create
    make

feature -- Connection
    connect (a_port: STRING)
        require
            valid_port: not a_port.is_empty
            not_connected: not is_connected
        do
            -- Open COM port (Windows) or /dev/rfcomm0 (Linux)
        ensure
            connected: is_connected
        end

    disconnect
        require
            connected: is_connected
        ensure
            not_connected: not is_connected
        end

feature -- Communication
    send (a_data: STRING)
        require
            connected: is_connected
        end

    receive: STRING
        require
            connected: is_connected
        end

feature -- Status
    is_connected: BOOLEAN
    last_error: detachable STRING

invariant
    error_only_when_failed: is_connected implies last_error = Void
end
```

#### Dependencies
- **simple_ipc** (extend with serial port support)
- **simple_process** (for device discovery via shell commands)
- **simple_win32_api** (for inline C Win32 calls)

## Competitive Analysis

| Approach | Pros | Cons |
|----------|------|------|
| Serial Port Bridge | Simple, works now, no C library | Requires pre-pairing, SPP only |
| Windows BLE Native | Full BLE support, modern API | Windows-only, complex WinRT wrapping |
| BlueZ Wrapper | Full Linux support | Linux-only, C binding maintenance |
| Shell Out | Quick, no C code | Slow, fragile parsing, limited control |

## Test Strategy

1. **Unit Tests** - Mock serial port for connection state tests
2. **Integration Tests** - Real HC-05/HC-06 module communication
3. **BLE Tests** - Heart rate monitor as standard test device

## Implementation Roadmap

| Phase | Scope | Est. LOC |
|-------|-------|----------|
| 1 | Serial port (Windows/Linux) | ~500 |
| 2 | Windows BLE discovery | ~800 |
| 3 | Windows BLE GATT | ~1200 |
| 4 | Linux BlueZ (future) | ~1500 |

## Verdict: Should We Build simple_bluetooth?

**YES, but with Phase 1 only initially.**

### Rationale:

1. **Real Developer Need** - KB query shows users asking about Bluetooth
2. **Achievable Scope** - Serial port bridge is practical with existing patterns
3. **Low Risk** - Phase 1 requires minimal C code, leverages OS serial ports
4. **Progressive Enhancement** - BLE can be added later if demand exists

### NOT Recommended:
- Full cross-platform BLE library (too complex for value)
- iOS/Android support (requires mobile development infrastructure)

## Next Steps

1. **Extend simple_ipc** with serial port support (or create simple_serial)
2. Create BLUETOOTH_SERIAL_PORT class
3. Add device discovery helpers (list COM ports, parse Bluetooth names)
4. Document pairing requirements clearly
5. Test with common SPP devices (HC-05, ESP32-SPP)

---

**Report Generated:** 2025-12-26
**Research Method:** 7-Step Library Design Process
