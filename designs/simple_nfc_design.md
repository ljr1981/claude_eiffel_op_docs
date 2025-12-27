# simple_nfc Design

## Overview

**simple_nfc** provides NFC tag reading/writing for Eiffel applications using Windows Proximity API.

## API

| Class | Purpose |
|-------|---------|
| SIMPLE_NFC | Main facade |
| NFC_DEVICE | NFC reader |
| NFC_TAG | NFC tag |
| NDEF_MESSAGE | NDEF data |
| NDEF_RECORD | NDEF record |

## Key Features

```eiffel
-- Check NFC availability
if nfc.is_available then
    print ("NFC ready")
end

-- Read tag
nfc.on_tag_detected (agent (tag: NFC_TAG)
    do
        print ("Tag ID: " + tag.id_hex)
        if attached tag.read_ndef as msg then
            across msg.records as r loop
                print ("Record: " + r.type + " = " + r.payload_text)
            end
        end
    end
)

-- Write tag
tag.write_ndef (create {NDEF_MESSAGE}.make_text ("Hello NFC"))
tag.write_ndef (create {NDEF_MESSAGE}.make_uri ("https://example.com"))
```

## NDEF Record Types

| Type | Description |
|------|-------------|
| Text | Plain text |
| URI | URL/link |
| Smart Poster | URL with metadata |
| MIME | Custom data |

## Windows API

- Proximity API: `Windows.Networking.Proximity`
- PC/SC (for advanced access)

## Phase 1 Scope

1. Detect NFC availability
2. Tag detection events
3. Read tag ID
4. Read NDEF messages
5. Write NDEF messages

## Dependencies

- Windows 10+ with NFC support
