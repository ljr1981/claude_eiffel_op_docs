# simple_smartcard Design

## Overview

**simple_smartcard** provides smart card reader access for Eiffel applications using PC/SC.

## API

| Class | Purpose |
|-------|---------|
| SIMPLE_SMARTCARD | Main facade |
| SMARTCARD_READER | Card reader |
| SMARTCARD | Connected card |
| APDU_COMMAND | Card command |
| APDU_RESPONSE | Card response |

## Key Features

```eiffel
-- Enumerate readers
across smartcard.readers as r loop
    print (r.name + " [" + r.state.out + "]")
end

-- Connect to card
if attached smartcard.connect (smartcard.readers.first) as card then
    -- Get card ATR
    print ("ATR: " + card.atr_hex)

    -- Send APDU command
    cmd := create {APDU_COMMAND}.make_select ("A0000000031010")
    if attached card.transmit (cmd) as resp then
        print ("Response: " + resp.data_hex)
        print ("SW: " + resp.status_word.to_hex_string)
    end

    card.disconnect
end

-- Card events
smartcard.on_card_inserted (agent handle_insert)
smartcard.on_card_removed (agent handle_remove)
```

## APDU Commands

| Command | Description |
|---------|-------------|
| SELECT | Select application |
| READ BINARY | Read file data |
| GET DATA | Get data object |
| VERIFY | Verify PIN |

## Windows API

- PC/SC: `SCardEstablishContext`, `SCardConnect`, `SCardTransmit`

## Phase 1 Scope

1. Enumerate readers
2. Connect to card
3. Get ATR
4. Send APDU commands
5. Card events

## Dependencies

- winscard.lib
