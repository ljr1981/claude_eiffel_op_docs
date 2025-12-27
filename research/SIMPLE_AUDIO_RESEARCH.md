# simple_audio Research Notes

**Date:** 2025-12-26
**Status:** Complete
**Goal:** Design an Eiffel real-time audio I/O library for playback and recording

---

## Step 1: Deep Web Research - Existing Audio Libraries

### Overview of Audio Libraries

| Library | Language | Platform | Key Feature |
|---------|----------|----------|-------------|
| WASAPI | C | Windows | Native, low-latency |
| CoreAudio | C | macOS/iOS | Apple native |
| ALSA | C | Linux | Linux native |
| PortAudio | C | Cross-platform | Mature, Audacity uses it |
| RtAudio | C++ | Cross-platform | Simple C++ API |
| miniaudio | C | Cross-platform | Single-file, feature-rich |
| OpenAL | C | Cross-platform | 3D audio, games |
| FMOD | C++ | Cross-platform | Commercial, games |

### miniaudio Architecture

Source: [miniaudio](https://miniaud.io/)

**Key Features:**
- Single source file, no dependencies
- Public domain license
- Playback, capture, full-duplex
- Built-in decoders: WAV, FLAC, MP3
- Sample rate/format conversion
- 3D spatialization (high-level API)
- Node graph for mixing/effects

**API Levels:**
| Level | Purpose | Complexity |
|-------|---------|------------|
| Low-level | Direct device access | High |
| High-level | Sound management, mixing | Medium |
| Node graph | Advanced effects | High |

### WASAPI Architecture

Source: [Microsoft WASAPI Documentation](https://learn.microsoft.com/en-us/windows/win32/coreaudio/wasapi)

**Core Interfaces:**
| Interface | Purpose |
|-----------|---------|
| IMMDeviceEnumerator | Enumerate audio devices |
| IMMDevice | Single audio device |
| IAudioClient | Audio stream management |
| IAudioRenderClient | Playback buffer access |
| IAudioCaptureClient | Recording buffer access |
| IAudioEndpointVolume | Volume control |

**Operating Modes:**
| Mode | Description | Latency |
|------|-------------|---------|
| Shared | Multiple apps share device | ~10-30ms |
| Exclusive | Single app owns device | ~3-10ms |

### PortAudio Architecture

Source: [PortAudio](https://portaudio.com/)

**Key Features:**
- Cross-platform: Windows, macOS, Linux
- MIT license
- Callback or blocking I/O
- Used by Audacity, JACK
- Supports: WASAPI, DirectSound, ASIO, CoreAudio, ALSA

---

## Step 2: Tech-Stack Research - WASAPI C API

### Device Enumeration

```c
// Initialize COM
CoInitializeEx(NULL, COINIT_MULTITHREADED);

// Get device enumerator
IMMDeviceEnumerator *pEnumerator = NULL;
CoCreateInstance(
    &CLSID_MMDeviceEnumerator, NULL, CLSCTX_ALL,
    &IID_IMMDeviceEnumerator, (void**)&pEnumerator
);

// Get default playback device
IMMDevice *pDevice = NULL;
pEnumerator->GetDefaultAudioEndpoint(eRender, eConsole, &pDevice);

// Or enumerate all devices
IMMDeviceCollection *pCollection = NULL;
pEnumerator->EnumAudioEndpoints(eRender, DEVICE_STATE_ACTIVE, &pCollection);
```

### Audio Client Initialization

```c
// Activate audio client
IAudioClient *pAudioClient = NULL;
pDevice->Activate(
    &IID_IAudioClient, CLSCTX_ALL, NULL, (void**)&pAudioClient
);

// Get mix format
WAVEFORMATEX *pwfx = NULL;
pAudioClient->GetMixFormat(&pwfx);

// Initialize audio client (shared mode)
pAudioClient->Initialize(
    AUDCLNT_SHAREMODE_SHARED,
    0, // flags
    10000000, // buffer duration (100ns units)
    0, // periodicity
    pwfx,
    NULL // session GUID
);

// Get render client
IAudioRenderClient *pRenderClient = NULL;
pAudioClient->GetService(&IID_IAudioRenderClient, (void**)&pRenderClient);
```

### Playback Loop

```c
// Get buffer
UINT32 numFramesAvailable;
pAudioClient->GetBufferSize(&numFramesAvailable);

// Start playback
pAudioClient->Start();

// Render loop
while (playing) {
    UINT32 numFramesPadding;
    pAudioClient->GetCurrentPadding(&numFramesPadding);

    UINT32 numFramesToWrite = numFramesAvailable - numFramesPadding;

    BYTE *pData;
    pRenderClient->GetBuffer(numFramesToWrite, &pData);

    // Fill pData with audio samples
    FillBuffer(pData, numFramesToWrite, pwfx);

    pRenderClient->ReleaseBuffer(numFramesToWrite, 0);

    Sleep(10); // Or use event-based timing
}

// Stop
pAudioClient->Stop();
```

### Capture Loop

```c
// Get capture client
IAudioCaptureClient *pCaptureClient = NULL;
pAudioClient->GetService(&IID_IAudioCaptureClient, (void**)&pCaptureClient);

// Capture loop
while (recording) {
    UINT32 packetLength = 0;
    pCaptureClient->GetNextPacketSize(&packetLength);

    while (packetLength != 0) {
        BYTE *pData;
        UINT32 numFramesAvailable;
        DWORD flags;

        pCaptureClient->GetBuffer(&pData, &numFramesAvailable, &flags, NULL, NULL);

        // Process audio data
        ProcessCapturedData(pData, numFramesAvailable);

        pCaptureClient->ReleaseBuffer(numFramesAvailable);
        pCaptureClient->GetNextPacketSize(&packetLength);
    }

    Sleep(10);
}
```

---

## Step 3: Eiffel Ecosystem Research - simple_* Coverage

### Available Dependencies

| Need | simple_* Library | Status |
|------|-----------------|--------|
| FFmpeg decoding | simple_ffmpeg | Planned |
| File I/O | simple_file | Available |
| Threading | SCOOP | Built-in |
| Logging | simple_logger | Available |

### ISE Libraries Needed

| Library | Purpose |
|---------|---------|
| base | Core classes |
| time | Timing |
| thread | Threading (for callbacks) |

### COM Integration

WASAPI uses COM. Need proper initialization:
```eiffel
feature {NONE} -- COM

    c_co_initialize: INTEGER
        external
            "C inline use <objbase.h>"
        alias
            "return CoInitializeEx(NULL, COINIT_MULTITHREADED);"
        end

    c_co_uninitialize
        external
            "C inline use <objbase.h>"
        alias
            "CoUninitialize();"
        end
```

---

## Step 4: Developer Pain Points - Common Audio Needs

### Most Common Use Cases (90% Coverage Target)

| Use Case | Frequency | Complexity |
|----------|-----------|------------|
| Play audio file | Very High | Medium |
| Record from microphone | High | Medium |
| System sound output | High | Medium |
| Real-time audio processing | Medium | High |
| Audio visualization | Medium | Medium |
| Voice chat | Medium | High |
| Game audio | Medium | High |
| Notification sounds | High | Low |

### Developer Questions

1. "How do I play an audio file?"
2. "How do I record from the microphone?"
3. "How do I list available audio devices?"
4. "How do I control volume?"
5. "How do I play audio in the background?"
6. "How do I get audio levels for visualization?"
7. "How do I mix multiple sounds?"
8. "What audio formats are supported?"

### Audio Format Considerations

| Format | Sample Rate | Bits | Channels | Use Case |
|--------|-------------|------|----------|----------|
| CD Quality | 44100 Hz | 16-bit | Stereo | Music |
| Voice | 16000 Hz | 16-bit | Mono | VoIP |
| High-Res | 96000 Hz | 24-bit | Stereo | Studio |
| DVD | 48000 Hz | 16-bit | 5.1 | Movies |

---

## Step 5: Innovation Hat - Unique Value Propositions

### Differentiators for simple_audio

1. **High-Level Playback**
   ```eiffel
   audio.play ("music.mp3")  -- One-liner
   audio.play_async ("music.mp3")  -- Non-blocking
   ```

2. **Design by Contract**
   ```eiffel
   play (a_file: READABLE_STRING_GENERAL)
       require
           file_exists: file_exists (a_file)
           not_playing: not is_playing
       ensure
           now_playing: is_playing
   ```

3. **Event-Based Recording**
   ```eiffel
   recorder.on_data_available (agent process_audio)
   recorder.start
   ```

4. **Audio Level Monitoring**
   ```eiffel
   across audio.devices as d loop
       print (d.name + ": " + d.peak_level.out)
   end
   ```

5. **SCOOP Integration**
   ```eiffel
   separate audio as a do
       a.play_async ("background.mp3")
   end
   ```

6. **Simple Mixing**
   ```eiffel
   mixer.add_track ("music.mp3", 0.8)  -- 80% volume
   mixer.add_track ("effects.wav", 1.0)
   mixer.play
   ```

---

## Step 6: Design Strategy Synthesis - Key Decisions

### Decision 1: WASAPI vs miniaudio vs PortAudio
**Choice:** WASAPI (native) for Phase 1, miniaudio consideration for Phase 2
**Rationale:** No external dependencies, Windows-native, low-latency

### Decision 2: Callback vs Blocking
**Choice:** Callback with SCOOP integration
**Rationale:** Better for real-time, non-blocking operation

### Decision 3: Format Support
**Choice:** PCM native, use simple_ffmpeg for encoded formats
**Rationale:** Separation of concerns, leverage existing library

### Decision 4: Shared vs Exclusive Mode
**Choice:** Shared mode default, exclusive optional
**Rationale:** Better compatibility, exclusive for pro audio

### Class Architecture

```
SIMPLE_AUDIO (facade)
├── devices -> ARRAYED_LIST [AUDIO_DEVICE]
├── default_playback_device -> AUDIO_DEVICE
├── default_recording_device -> AUDIO_DEVICE
├── play (file) -- High-level playback
├── record (file) -- High-level recording
├── create_player -> AUDIO_PLAYER
├── create_recorder -> AUDIO_RECORDER
└── create_mixer -> AUDIO_MIXER

AUDIO_DEVICE
├── id, name
├── is_playback, is_recording
├── sample_rate, channels, bits_per_sample
├── is_default
├── peak_level
└── volume

AUDIO_PLAYER
├── open (device)
├── play, pause, stop
├── is_playing, is_paused
├── position, duration
├── volume
├── on_finished (callback)
└── close

AUDIO_RECORDER
├── open (device)
├── start, stop
├── is_recording
├── on_data_available (callback)
├── buffer -> AUDIO_BUFFER
└── close

AUDIO_BUFFER
├── data -> ARRAY [INTEGER_16] or ARRAY [REAL_32]
├── sample_rate, channels
├── frame_count
├── duration
└── to_wav (file)

AUDIO_MIXER
├── add_track (source, volume)
├── remove_track
├── set_track_volume
├── play, stop
└── output -> AUDIO_BUFFER
```

### Phase 1 Scope (90% Use Cases)

1. ✅ Enumerate audio devices
2. ✅ Get device properties
3. ✅ Play PCM audio
4. ✅ Record PCM audio
5. ✅ Volume control
6. ✅ Peak level monitoring
7. ✅ Callback-based audio
8. ✅ WAV file I/O

### Phase 2 (Future)

- Encoded format playback (via simple_ffmpeg)
- Audio mixing
- Effects processing
- 3D spatialization
- ASIO support
- Linux support (ALSA)

---

## Step 7: Implementation Plan

### Files

```
simple_audio/
├── simple_audio.ecf
├── src/
│   ├── simple_audio.e           -- Main facade
│   ├── audio_device.e           -- Device info
│   ├── audio_player.e           -- Playback
│   ├── audio_recorder.e         -- Recording
│   ├── audio_buffer.e           -- Audio data
│   ├── audio_format.e           -- Format specs
│   ├── audio_mixer.e            -- Mixing
│   ├── audio_wav.e              -- WAV file I/O
│   └── audio_c_api.e            -- C externals
├── Clib/
│   ├── audio_bridge.h
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
| test_enumerate | List audio devices |
| test_device_info | Get device properties |
| test_play_pcm | Play PCM buffer |
| test_record | Record to buffer |
| test_wav_read | Read WAV file |
| test_wav_write | Write WAV file |
| test_volume | Set/get volume |
| test_peak_level | Monitor levels |
| test_callback | Callback invocation |

### Dependencies

| Dependency | Type |
|------------|------|
| simple_file | simple_* |
| base | ISE stdlib |
| thread | ISE stdlib |

### Required Windows Libraries

```
ole32.lib (COM)
uuid.lib (GUIDs)
```

### Required Headers

```c
#include <audioclient.h>
#include <mmdeviceapi.h>
#include <endpointvolume.h>
#include <functiondiscoverykeys_devpkey.h>
```

---

## Sources

- [WASAPI Documentation](https://learn.microsoft.com/en-us/windows/win32/coreaudio/wasapi)
- [miniaudio](https://miniaud.io/)
- [miniaudio GitHub](https://github.com/mackron/miniaudio)
- [PortAudio](https://portaudio.com/)
- [RtAudio](https://github.com/thestk/rtaudio)
- [Audio APIs Comparison](https://bastibe.de/2017-07-10-audio-apis-wasapi.html)
- [WASAPI Tutorial](https://medium.com/@shahidahmadkhan86/sound-in-windows-the-wasapi-in-c-23024cdac7c6)
