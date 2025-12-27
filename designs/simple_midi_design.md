# simple_midi Design

## Overview

**simple_midi** provides MIDI device communication for Eiffel applications.

## API

| Class | Purpose |
|-------|---------|
| SIMPLE_MIDI | Main facade |
| MIDI_INPUT | MIDI input device |
| MIDI_OUTPUT | MIDI output device |
| MIDI_MESSAGE | MIDI message |

## Key Features

```eiffel
-- Enumerate devices
across midi.input_devices as d loop
    print ("Input: " + d.name)
end
across midi.output_devices as d loop
    print ("Output: " + d.name)
end

-- Receive MIDI
input := midi.open_input (0)
input.on_message (agent handle_midi)

-- Send MIDI
output := midi.open_output (0)
output.note_on (60, 127)  -- Middle C, full velocity
output.note_off (60)
output.program_change (0)  -- Piano
```

## MIDI Messages

| Type | Description |
|------|-------------|
| Note On | Key pressed |
| Note Off | Key released |
| Control Change | Knob/slider |
| Program Change | Instrument select |
| Pitch Bend | Pitch wheel |

## Windows API

- Windows MIDI: `midiInOpen`, `midiOutOpen`, `midiInStart`

## Phase 1 Scope

1. Enumerate MIDI devices
2. Open input/output
3. Receive messages
4. Send messages
5. Note on/off

## Dependencies

- winmm.lib
