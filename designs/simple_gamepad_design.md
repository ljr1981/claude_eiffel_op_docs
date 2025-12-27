# simple_gamepad Design

## Overview

**simple_gamepad** provides game controller input for Eiffel applications using XInput and DirectInput.

## API

| Class | Purpose |
|-------|---------|
| SIMPLE_GAMEPAD | Main facade |
| GAMEPAD | Connected controller |
| GAMEPAD_STATE | Button/axis state |

## Key Features

```eiffel
-- Enumerate gamepads
across gamepad.controllers as c loop
    print (c.name + " [" + c.type.out + "]")
end

-- Read state
if attached gamepad.first as gp then
    state := gp.state
    if state.button_a then
        print ("A pressed")
    end
    print ("Left stick: " + state.left_x.out + ", " + state.left_y.out)
end

-- Event-based
gamepad.on_button_pressed (agent handle_button)
gamepad.on_axis_changed (agent handle_axis)
```

## Gamepad State

| Property | Type | Range |
|----------|------|-------|
| button_a/b/x/y | BOOLEAN | True/False |
| left_x/y | REAL_32 | -1.0 to 1.0 |
| right_x/y | REAL_32 | -1.0 to 1.0 |
| left_trigger | REAL_32 | 0.0 to 1.0 |
| right_trigger | REAL_32 | 0.0 to 1.0 |
| dpad_up/down/left/right | BOOLEAN | True/False |

## Windows API

- XInput: Xbox controllers (recommended)
- DirectInput: Generic controllers

## Phase 1 Scope

1. Enumerate controllers
2. Read button states
3. Read analog sticks
4. Read triggers
5. D-pad support

## Dependencies

- XInput.lib, dinput8.lib
