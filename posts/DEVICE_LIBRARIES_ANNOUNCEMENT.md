# Simple Eiffel Device Libraries - Hardware Access for Eiffel

I'm pleased to announce three new libraries bringing hardware and multimedia capabilities to Eiffel: **simple_ffmpeg**, **simple_usb**, and **simple_audio**.

## Why Device Libraries?

Eiffel has always excelled at correctness and reliability. Now it can talk to your hardware with the same Design by Contract rigor. These libraries follow the Simple Eiffel patterns: facade classes, fluent APIs, inline C (Eric Bezault pattern), and comprehensive contracts.

## simple_ffmpeg - Multimedia Processing

Video/audio transcoding, metadata extraction, and format conversion using FFmpeg.

```eiffel
local
    ffmpeg: SIMPLE_FFMPEG
    opts: FFMPEG_OPTIONS
do
    create ffmpeg.make
    
    -- Probe media file
    if attached ffmpeg.probe ("video.mp4") as info then
        print ("Duration: " + info.duration.out + "s%N")
        print ("Resolution: " + info.resolution_string + "%N")
    end
    
    -- Transcode with options
    create opts.make
    opts := opts.set_video_codec ("libx265")
               .set_resolution (1920, 1080)
               .preset_quality
    
    ffmpeg.transcode_with_options ("input.mov", "output.mp4", opts)
end
```

**Key features:**
- CLI mode (uses ffmpeg.exe) - no SDK required
- SDK mode available for native FFmpeg access
- Fluent options API with presets (web, fast, quality)
- Audio extraction, frame extraction, video resizing
- 19 tests passing

**GitHub**: https://github.com/simple-eiffel/simple_ffmpeg

---

## simple_usb - USB Device Access

Enumerate USB devices, read HID reports, and work with gamepads.

```eiffel
local
    usb: SIMPLE_USB
do
    create usb.make
    
    -- List all USB devices
    across usb.devices as d loop
        print (d.product_name + " [" + d.id_string + "]%N")
    end
    
    -- Read from HID device
    across usb.hid_devices as hid loop
        if hid.open then
            if attached hid.read_report as report then
                print ("Data: " + report.to_hex_string + "%N")
            end
            hid.close
        end
    end
    
    -- Gamepad input
    across usb.gamepads as gp loop
        if gp.open and then gp.poll then
            print ("Left stick: " + gp.left_stick_x.out + "%N")
        end
    end
end
```

**Key features:**
- USB device enumeration via Windows SetupAPI
- HID device read/write with report parsing
- Gamepad support with axes and buttons
- Arduino detection by known VID/PIDs
- Inline C pattern (no external DLLs)
- 29 tests passing

**GitHub**: https://github.com/simple-eiffel/simple_usb

---

## simple_audio - Real-time Audio I/O

Low-latency audio playback and recording using Windows WASAPI.

```eiffel
local
    audio: SIMPLE_AUDIO
    buffer: AUDIO_BUFFER
do
    create audio.make
    
    -- List audio devices
    across audio.output_devices as d loop
        print (d.display_name + "%N")
    end
    
    -- Play a 440Hz sine wave
    if attached audio.default_output as dev then
        if attached audio.create_output_stream (dev, 44100, 2, 16) as stream then
            create buffer.make (44100, 2, 16)
            buffer.fill_sine_wave (440.0, 44100)
            
            stream.start
            stream.write (buffer)
            stream.stop
            stream.close
        end
    end
    
    audio.dispose
end
```

**Key features:**
- WASAPI streaming (shared mode, low latency)
- Device enumeration (input/output)
- PCM buffers with 8/16/24/32-bit support
- Built-in sine wave generation for testing
- Inline C pattern (no external DLLs)
- 17 tests passing

**GitHub**: https://github.com/simple-eiffel/simple_audio

---

## Installation

All three libraries are Windows-only (using Windows APIs). Add to your ECF:

```xml
<library name="simple_ffmpeg" location="$SIMPLE_EIFFEL/simple_ffmpeg/simple_ffmpeg.ecf"/>
<library name="simple_usb" location="$SIMPLE_EIFFEL/simple_usb/simple_usb.ecf"/>
<library name="simple_audio" location="$SIMPLE_EIFFEL/simple_audio/simple_audio.ecf"/>
```

For simple_ffmpeg CLI mode, ensure `ffmpeg.exe` and `ffprobe.exe` are in your PATH.

## What's Next?

The device library roadmap includes:
- **simple_serial** - COM port communication (available now)
- **simple_bluetooth** - Bluetooth SPP (available now)
- **simple_midi** - MIDI input/output
- **simple_camera** - Webcam capture
- **simple_gamepad** - XInput game controllers

## Links

- **Simple Eiffel**: https://github.com/simple-eiffel
- **Device Roadmap**: See reference_docs/designs/DEVICE_LIBRARIES_ROADMAP.md

---

Part of the **Simple Eiffel** ecosystem - modern, contract-driven Eiffel libraries.

Larry
