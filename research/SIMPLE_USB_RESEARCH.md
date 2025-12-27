# simple_usb Research Notes

**Date:** 2025-12-26
**Status:** Complete
**Goal:** Design an Eiffel USB device access library (HID, bulk transfer, device enumeration)

---

## Step 1: Deep Web Research - Existing USB Libraries

### Overview of USB Access Libraries

| Library | Language | Platform | Key Feature |
|---------|----------|----------|-------------|
| libusb | C | Cross-platform | Industry standard |
| WinUSB | C | Windows | Native Windows API |
| HIDAPI | C | Cross-platform | HID-focused |
| libusbp | C | Cross-platform | Pololu, simpler API |
| pyusb | Python | Cross-platform | Python wrapper for libusb |

### libusb Architecture

Source: [libusb GitHub](https://github.com/libusb/libusb)

**Key Features:**
- Cross-platform: Windows, macOS, Linux, BSD, WebAssembly
- User-mode: No special privileges needed
- Version-agnostic: USB 1.0 to 3.1 supported
- LGPL 2.1+ licensed

**Core Concepts:**
- **Device**: Physical USB device
- **Configuration**: Device configuration
- **Interface**: Function within configuration
- **Endpoint**: Data transfer point (IN/OUT)
- **Transfer**: Data exchange operation

### WinUSB Architecture

Source: [Microsoft WinUSB Documentation](https://learn.microsoft.com/en-us/windows-hardware/drivers/usbcon/using-winusb-api-to-communicate-with-a-usb-device)

**Key Functions:**
| Function | Purpose |
|----------|---------|
| `SetupDiGetClassDevs` | Enumerate device class |
| `SetupDiEnumDeviceInterfaces` | Iterate devices |
| `SetupDiGetDeviceInterfaceDetail` | Get device path |
| `CreateFile` | Open device handle |
| `WinUsb_Initialize` | Initialize WinUSB |
| `WinUsb_QueryDeviceInformation` | Get device info |
| `WinUsb_ReadPipe` | Read from endpoint |
| `WinUsb_WritePipe` | Write to endpoint |
| `WinUsb_Free` | Release WinUSB handle |

### HIDAPI Architecture

Source: [HIDAPI GitHub](https://github.com/libusb/hidapi)

**Key Functions:**
| Function | Purpose |
|----------|---------|
| `hid_enumerate` | List HID devices |
| `hid_open` | Open device by VID/PID |
| `hid_open_path` | Open device by path |
| `hid_read` | Read report |
| `hid_write` | Write report |
| `hid_get_feature_report` | Get feature report |
| `hid_send_feature_report` | Send feature report |
| `hid_close` | Close device |

---

## Step 2: Tech-Stack Research - Windows USB APIs

### SetupAPI for Device Enumeration

Source: [CodeProject SetupAPI](https://www.codeproject.com/articles/Enumerate-Installed-Devices-Using-Setup-API)

```c
// Get device info set for USB devices
HDEVINFO hDevInfo = SetupDiGetClassDevs(
    &GUID_DEVINTERFACE_USB_DEVICE,
    NULL, NULL,
    DIGCF_PRESENT | DIGCF_DEVICEINTERFACE
);

// Enumerate devices
SP_DEVICE_INTERFACE_DATA devInterfaceData;
devInterfaceData.cbSize = sizeof(SP_DEVICE_INTERFACE_DATA);

for (DWORD i = 0; SetupDiEnumDeviceInterfaces(
    hDevInfo, NULL, &GUID_DEVINTERFACE_USB_DEVICE, i, &devInterfaceData); i++)
{
    // Get device path
    PSP_DEVICE_INTERFACE_DETAIL_DATA pDetailData;
    // ... get detail and open device
}
```

### HID API (Windows)

```c
// Get HID GUID
GUID hidGuid;
HidD_GetHidGuid(&hidGuid);

// Enumerate HID devices
HDEVINFO hDevInfo = SetupDiGetClassDevs(
    &hidGuid, NULL, NULL,
    DIGCF_PRESENT | DIGCF_DEVICEINTERFACE
);

// Open device
HANDLE hDevice = CreateFile(
    devicePath,
    GENERIC_READ | GENERIC_WRITE,
    FILE_SHARE_READ | FILE_SHARE_WRITE,
    NULL, OPEN_EXISTING, 0, NULL
);

// Get device attributes
HIDD_ATTRIBUTES attrs;
attrs.Size = sizeof(HIDD_ATTRIBUTES);
HidD_GetAttributes(hDevice, &attrs);
// attrs.VendorID, attrs.ProductID, attrs.VersionNumber
```

### USB Device Classes

| Class | Code | Examples |
|-------|------|----------|
| Audio | 0x01 | Speakers, microphones |
| HID | 0x03 | Keyboards, mice, gamepads |
| Image | 0x06 | Scanners, cameras |
| Printer | 0x07 | Printers |
| Mass Storage | 0x08 | USB drives |
| Hub | 0x09 | USB hubs |
| CDC | 0x0A | Serial adapters |
| Smart Card | 0x0B | Card readers |
| Vendor Specific | 0xFF | Custom devices |

---

## Step 3: Eiffel Ecosystem Research - simple_* Coverage

### Available Dependencies

| Need | simple_* Library | Status |
|------|-----------------|--------|
| Serial ports | simple_serial | Available |
| Bluetooth | simple_bluetooth | Available |
| Registry | simple_registry | Available |
| Win32 API | simple_win32_api | Available |
| Logging | simple_logger | Available |

### ISE Libraries Needed

| Library | Purpose |
|---------|---------|
| base | Core classes |
| time | Timing |
| wel | Windows externals |

### Inline C Pattern for USB

```eiffel
feature {NONE} -- C externals

    c_setup_get_class_devs (a_guid: POINTER; a_flags: INTEGER): POINTER
        external
            "C inline use <setupapi.h>"
        alias
            "[
                return SetupDiGetClassDevs(
                    (LPGUID)$a_guid,
                    NULL, NULL,
                    (DWORD)$a_flags
                );
            ]"
        end

    c_hid_get_attributes (a_handle: POINTER; a_attrs: POINTER): BOOLEAN
        external
            "C inline use <hidsdi.h>"
        alias
            "[
                return HidD_GetAttributes(
                    (HANDLE)$a_handle,
                    (PHIDD_ATTRIBUTES)$a_attrs
                );
            ]"
        end
```

---

## Step 4: Developer Pain Points - Common USB Needs

### Most Common Use Cases (90% Coverage Target)

| Use Case | Frequency | Complexity |
|----------|-----------|------------|
| Enumerate USB devices | Very High | Low |
| Get device info (VID/PID/name) | Very High | Low |
| Communicate with HID devices | High | Medium |
| Game controller input | High | Medium |
| Arduino/custom hardware | High | Medium |
| USB serial devices | High | Low (use simple_serial) |
| Bulk data transfer | Medium | High |

### Developer Questions

1. "How do I list all connected USB devices?"
2. "How do I find a device by vendor/product ID?"
3. "How do I read from a game controller?"
4. "How do I communicate with my Arduino?"
5. "How do I detect when a USB device is connected?"
6. "How do I read/write to a HID device?"
7. "How do I get the device name and manufacturer?"

### Common Device Categories

| Category | Examples | API |
|----------|----------|-----|
| Game controllers | Xbox, PlayStation, joysticks | HID + XInput |
| Arduino/embedded | Arduino, ESP32, STM32 | Serial (CDC) or HID |
| Input devices | Keyboards, mice, touchpads | HID (raw) |
| Custom hardware | Lab equipment, sensors | HID or Bulk |
| LED controllers | RGB strips, keyboards | HID |

---

## Step 5: Innovation Hat - Unique Value Propositions

### Differentiators for simple_usb

1. **Device Discovery with DBC**
   ```eiffel
   across usb.devices as d loop
       require d.is_valid
       print (d.product_name + " [" + d.vendor_id.to_hex_string + ":" + d.product_id.to_hex_string + "]")
   end
   ```

2. **Type-Safe Device Categories**
   ```eiffel
   if attached {USB_HID_DEVICE} device as hid then
       hid.read_report
   end
   if attached {USB_GAMEPAD} device as pad then
       print (pad.left_stick_x.out)
   end
   ```

3. **Event-Based Hotplug**
   ```eiffel
   usb.on_device_connected (agent handle_connect)
   usb.on_device_disconnected (agent handle_disconnect)
   ```

4. **Built-in Gamepad Support**
   ```eiffel
   across usb.gamepads as gp loop
       print (gp.name + ": " + gp.button_count.out + " buttons")
   end
   ```

5. **Arduino Integration**
   ```eiffel
   if attached usb.find_arduino as arduino then
       -- Automatically detects Arduino devices
       arduino.open
       arduino.write ("LED ON")
   end
   ```

---

## Step 6: Design Strategy Synthesis - Key Decisions

### Decision 1: libusb vs Native APIs
**Choice:** Native Windows APIs (SetupAPI + WinUSB + HID)
**Rationale:** No external DLL dependencies, better integration

### Decision 2: HID Focus
**Choice:** HID-first design, bulk transfer secondary
**Rationale:** HID covers 80% of use cases (gamepads, custom hardware)

### Decision 3: Device Categories
**Choice:** Specialized classes for common device types
**Rationale:** Better UX for common scenarios

### Class Architecture

```
SIMPLE_USB (facade)
├── devices -> ARRAYED_LIST [USB_DEVICE]
├── hid_devices -> ARRAYED_LIST [USB_HID_DEVICE]
├── gamepads -> ARRAYED_LIST [USB_GAMEPAD]
├── find_device (vid, pid) -> USB_DEVICE
├── refresh
└── on_device_connected

USB_DEVICE (base)
├── vendor_id, product_id
├── product_name, manufacturer
├── device_path
├── is_connected
└── device_class

USB_HID_DEVICE extends USB_DEVICE
├── open, close
├── read_report -> USB_HID_REPORT
├── write_report
├── get_feature_report
├── send_feature_report
└── input_report_length, output_report_length

USB_GAMEPAD extends USB_HID_DEVICE
├── buttons -> ARRAY [BOOLEAN]
├── axes -> ARRAY [INTEGER]
├── left_stick_x, left_stick_y
├── right_stick_x, right_stick_y
├── triggers
└── dpad

USB_BULK_DEVICE extends USB_DEVICE
├── open, close
├── read (endpoint, length) -> ARRAY [NATURAL_8]
├── write (endpoint, data) -> INTEGER
└── control_transfer
```

### Phase 1 Scope (90% Use Cases)

1. ✅ Enumerate USB devices
2. ✅ Get device info (VID, PID, name, manufacturer)
3. ✅ Open/close HID devices
4. ✅ Read/write HID reports
5. ✅ Basic gamepad support
6. ✅ Device hotplug detection
7. ✅ Find device by VID/PID
8. ✅ Arduino detection

### Phase 2 (Future)

- Full gamepad mapping
- XInput integration
- Bulk transfer
- Control transfer
- Custom descriptors
- Linux support

---

## Step 7: Implementation Plan

### Files

```
simple_usb/
├── simple_usb.ecf
├── src/
│   ├── simple_usb.e             -- Main facade
│   ├── usb_device.e             -- Base device
│   ├── usb_hid_device.e         -- HID device
│   ├── usb_hid_report.e         -- HID report
│   ├── usb_gamepad.e            -- Gamepad
│   ├── usb_bulk_device.e        -- Bulk transfer
│   ├── usb_enumerator.e         -- Device enumeration
│   ├── usb_hotplug.e            -- Hotplug detection
│   └── usb_c_api.e              -- C externals
├── Clib/
│   ├── usb_bridge.h
│   └── Makefile.win
├── testing/
│   ├── test_app.e
│   ├── lib_tests.e
│   └── test_set_base.e
├── docs/
│   ├── index.html
│   └── css/style.css
└── README.md
```

### Test Plan

| Test | Description |
|------|-------------|
| test_enumerate | List USB devices |
| test_device_info | Get VID/PID/name |
| test_find_device | Find by VID/PID |
| test_hid_open | Open HID device |
| test_hid_read | Read HID report |
| test_hid_write | Write HID report |
| test_gamepad | Read gamepad state |
| test_hotplug | Detect connect/disconnect |

### Dependencies

| Dependency | Type |
|------------|------|
| simple_win32_api | simple_* |
| base | ISE stdlib |
| wel | ISE stdlib |

### Required Windows Libraries

```
setupapi.lib
hid.lib
winusb.lib
```

---

## Sources

- [libusb](https://libusb.info/)
- [libusb GitHub](https://github.com/libusb/libusb)
- [WinUSB Functions](https://learn.microsoft.com/en-us/windows-hardware/drivers/usbcon/using-winusb-api-to-communicate-with-a-usb-device)
- [HIDAPI GitHub](https://github.com/libusb/hidapi)
- [SetupAPI Device Enumeration](https://www.codeproject.com/articles/Enumerate-Installed-Devices-Using-Setup-API)
- [Arduino USB HID](https://github.com/tigoe/hid-examples)
