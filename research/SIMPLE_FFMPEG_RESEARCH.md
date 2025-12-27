# simple_ffmpeg Research Notes

**Date:** 2025-12-26
**Status:** Complete
**Goal:** Design an Eiffel multimedia library wrapping FFmpeg's libav* C libraries

---

## Step 1: Deep Web Research - Existing Multimedia Libraries

### Overview of Multimedia Frameworks

| Library | Language | Type | Key Feature |
|---------|----------|------|-------------|
| FFmpeg/libav* | C | Framework | Industry standard, decode/encode/transcode |
| GStreamer | C | Framework | Pipeline-based, plugins |
| OpenCV | C++ | Vision | Image/video processing |
| VLC libvlc | C | Player | Playback-focused |
| MediaFoundation | C++ | Windows | Windows native |
| AVFoundation | Obj-C | macOS/iOS | Apple native |

### FFmpeg/libav* Architecture

Source: [ffmpeg-libav-tutorial](https://github.com/leandromoreira/ffmpeg-libav-tutorial)

**Core Data Structures:**
- **AVFormatContext**: Container abstraction (MKV, MP4, AVI, etc.)
- **AVStream**: Individual media stream (audio, video, subtitle)
- **AVCodecContext**: Codec configuration (bitrate, resolution, sample rate)
- **AVPacket**: Compressed data slice ready for decoding
- **AVFrame**: Uncompressed raw data (pixels for video, PCM for audio)

**Basic Decoding Workflow:**
1. Load container: `avformat_open_input()` → AVFormatContext
2. Find streams: `avformat_find_stream_info()`
3. Find decoder: `avcodec_find_decoder()` → AVCodec
4. Open decoder: `avcodec_alloc_context3()` + `avcodec_open2()`
5. Read packets: `av_read_frame()` → AVPacket
6. Decode frames: `avcodec_send_packet()` + `avcodec_receive_frame()` → AVFrame

**Key Libraries:**
| Library | Purpose |
|---------|---------|
| libavcodec | Encoding/decoding |
| libavformat | Container formats (mux/demux) |
| libavutil | Utilities, memory, math |
| libswscale | Video scaling/conversion |
| libswresample | Audio resampling/conversion |
| libavfilter | Video/audio filters |

---

### Industry Adoption

Source: [FFmpeg Wikipedia](https://en.wikipedia.org/wiki/FFmpeg)

- **VLC**: Uses FFmpeg for decoding
- **YouTube**: FFmpeg in processing pipeline
- **Chrome/Firefox**: FFmpeg for media playback
- **Audacity**: FFmpeg for import/export
- **OBS Studio**: FFmpeg for encoding
- **NASA Perseverance**: FFmpeg on Mars rover

---

## Step 2: Tech-Stack Research - FFmpeg C API

### Modern FFmpeg API (Post-3.0)

Source: [decode_video.c example](https://github.com/FFmpeg/FFmpeg/blob/master/doc/examples/decode_video.c)

**New Send/Receive API (Recommended):**
```c
// Send packet to decoder
avcodec_send_packet(codec_ctx, packet);

// Receive decoded frame
avcodec_receive_frame(codec_ctx, frame);
```

**Old API (Deprecated):**
```c
// Don't use - deprecated
avcodec_decode_video2(codec_ctx, frame, &got_frame, packet);
```

### Key Functions

| Function | Purpose |
|----------|---------|
| `av_register_all()` | Register all codecs (deprecated in 4.0+) |
| `avformat_open_input()` | Open media file |
| `avformat_find_stream_info()` | Read stream metadata |
| `avcodec_find_decoder()` | Find decoder by codec ID |
| `avcodec_alloc_context3()` | Allocate codec context |
| `avcodec_open2()` | Initialize decoder |
| `av_read_frame()` | Read next packet |
| `avcodec_send_packet()` | Send packet for decoding |
| `avcodec_receive_frame()` | Get decoded frame |
| `av_frame_alloc()` | Allocate frame |
| `av_packet_alloc()` | Allocate packet |
| `sws_getContext()` | Create scaler context |
| `sws_scale()` | Scale/convert video frame |
| `swr_alloc_set_opts()` | Configure audio resampler |
| `swr_convert()` | Convert audio samples |

### Memory Management

FFmpeg uses reference counting:
```c
av_frame_unref(frame);     // Release frame data
av_packet_unref(packet);   // Release packet data
avcodec_free_context(&ctx);// Free codec context
avformat_close_input(&fmt);// Close format context
```

### Error Handling

FFmpeg returns negative values on error:
```c
int ret = avformat_open_input(&fmt_ctx, filename, NULL, NULL);
if (ret < 0) {
    char errbuf[256];
    av_strerror(ret, errbuf, sizeof(errbuf));
    // Handle error
}
```

---

## Step 3: Eiffel Ecosystem Research - simple_* Coverage

### Available Dependencies

| Need | simple_* Library | Status |
|------|-----------------|--------|
| File paths | simple_file | Available |
| JSON metadata | simple_json | Available |
| Logging | simple_logger | Available |
| Testing | simple_testing | Available |

### ISE Libraries Needed

| Library | Purpose |
|---------|---------|
| base | Core classes, ARRAY, STRING |
| time | Timing operations |

### Inline C Pattern

From simple_serial, simple_bluetooth:
```eiffel
feature {NONE} -- C externals

    c_open_input (a_url: POINTER): POINTER
        external
            "C inline use <libavformat/avformat.h>"
        alias
            "[
                AVFormatContext *ctx = NULL;
                int ret = avformat_open_input(&ctx, (const char *)$a_url, NULL, NULL);
                if (ret < 0) return NULL;
                return ctx;
            ]"
        end
```

### ECF Configuration

```xml
<external_include location="$(FFMPEG_INCLUDE)"/>
<external_library location="$(FFMPEG_LIB)/avcodec.lib"/>
<external_library location="$(FFMPEG_LIB)/avformat.lib"/>
<external_library location="$(FFMPEG_LIB)/avutil.lib"/>
<external_library location="$(FFMPEG_LIB)/swscale.lib"/>
<external_library location="$(FFMPEG_LIB)/swresample.lib"/>
```

---

## Step 4: Developer Pain Points - Common FFmpeg Needs

### Most Common Use Cases (90% Coverage Target)

| Use Case | Frequency | Complexity |
|----------|-----------|------------|
| Get media info (duration, resolution, codecs) | Very High | Low |
| Extract video thumbnails | Very High | Medium |
| Transcode video (format conversion) | High | Medium |
| Extract audio from video | High | Low |
| Convert audio formats | High | Low |
| Resize/scale video | Medium | Medium |
| Capture frames from stream | Medium | Medium |
| Record to file | Medium | High |

### Developer Questions

1. "How do I get the duration of a video file?"
2. "How do I extract a frame at a specific timestamp?"
3. "How do I convert MP4 to WebM?"
4. "How do I extract audio track from video?"
5. "How do I resize a video?"
6. "How do I get video resolution and codec info?"
7. "How do I create thumbnails?"
8. "How do I convert audio to MP3?"

### Pain Points with Raw FFmpeg API

1. **Complex initialization** - Many setup steps
2. **Memory management** - Easy to leak
3. **Error handling** - Cryptic error codes
4. **Codec/format matching** - Many combinations
5. **Threading** - Multi-threaded decoding complex
6. **Pixel format conversion** - YUV to RGB
7. **Audio sample format** - Planar vs interleaved

---

## Step 5: Innovation Hat - Unique Value Propositions

### Differentiators for simple_ffmpeg

1. **Design by Contract**
   - Preconditions prevent invalid operations
   - Postconditions guarantee results
   - Invariants ensure object consistency

2. **Void Safety**
   - No null pointer crashes
   - Explicit handling of missing data

3. **SCOOP Ready**
   - Async transcoding operations
   - Progress callbacks
   - Cancelable operations

4. **High-Level Facade**
   - One-liner for common operations
   - Complex operations hidden

5. **Automatic Resource Management**
   - RAII-style cleanup
   - No manual memory freeing

6. **Type Safety**
   - Strongly typed frame/packet classes
   - Compile-time codec validation

### API Design Goals

```eiffel
-- One-liner operations
ffmpeg.transcode ("input.avi", "output.mp4")
ffmpeg.extract_audio ("video.mp4", "audio.mp3")
ffmpeg.extract_frame ("video.mp4", 5.0, "thumb.jpg")

-- Info queries with DBC
if attached ffmpeg.probe ("video.mp4") as info then
    print (info.duration.out)
    print (info.video_width.out + "x" + info.video_height.out)
end

-- Frame-by-frame with safety
if attached ffmpeg.open_input ("video.mp4") as decoder then
    across decoder as frame loop
        frame.save_as_jpeg ("frame_" + frame.timestamp.out + ".jpg", 85)
    end
    decoder.close
end
```

---

## Step 6: Design Strategy Synthesis - Key Decisions

### Decision 1: C Library vs CLI
**Choice:** C library wrapper (libav*)
**Rationale:** No external dependencies, real-time capable, cross-platform

### Decision 2: Sync vs Async
**Choice:** Sync first, async Phase 2
**Rationale:** Simpler API, SCOOP enables async later

### Decision 3: High-level vs Low-level
**Choice:** Both - facade + raw access
**Rationale:** 90% use cases via facade, power users get raw access

### Decision 4: Frame Format
**Choice:** RGB/RGBA for video, PCM for audio
**Rationale:** Most usable format for applications

### Decision 5: Error Handling
**Choice:** last_error + has_error pattern
**Rationale:** Consistent with simple_* ecosystem

### Class Architecture

```
SIMPLE_FFMPEG (facade)
├── probe() -> FFMPEG_MEDIA_INFO
├── transcode() -> BOOLEAN
├── extract_audio() -> BOOLEAN
├── extract_frame() -> BOOLEAN
├── open_input() -> FFMPEG_DECODER
└── create_encoder() -> FFMPEG_ENCODER

FFMPEG_DECODER
├── read_video_frame() -> FFMPEG_VIDEO_FRAME
├── read_audio_frame() -> FFMPEG_AUDIO_FRAME
├── seek()
└── close()

FFMPEG_ENCODER
├── write_video_frame()
├── write_audio_frame()
├── finalize()
└── close()

FFMPEG_VIDEO_FRAME
├── width, height
├── timestamp
├── to_rgb() -> ARRAY [NATURAL_8]
├── save_as_jpeg()
└── save_as_png()

FFMPEG_AUDIO_FRAME
├── sample_rate, channels
├── timestamp
├── to_pcm_s16() -> ARRAY [INTEGER_16]
└── to_pcm_float() -> ARRAY [REAL_32]
```

### Phase 1 Scope (90% Use Cases)

1. ✅ Initialize FFmpeg
2. ✅ Probe media info (duration, resolution, codecs)
3. ✅ Open video/audio files
4. ✅ Decode video frames
5. ✅ Decode audio frames
6. ✅ Save frames as images (JPEG, PNG)
7. ✅ Basic transcode
8. ✅ Extract audio
9. ✅ Seek to timestamp
10. ✅ Resource cleanup

### Phase 2 (Future)

- Streaming (RTMP, HLS)
- Hardware acceleration
- Filter graphs
- Screen capture
- SCOOP async operations

---

## Step 7: Implementation Plan

### Files

```
simple_ffmpeg/
├── simple_ffmpeg.ecf
├── src/
│   ├── simple_ffmpeg.e          -- Main facade
│   ├── ffmpeg_decoder.e         -- Decode media
│   ├── ffmpeg_encoder.e         -- Encode media
│   ├── ffmpeg_transcoder.e      -- High-level transcode
│   ├── ffmpeg_frame.e           -- Base frame class
│   ├── ffmpeg_video_frame.e     -- Video frame
│   ├── ffmpeg_audio_frame.e     -- Audio frame
│   ├── ffmpeg_media_info.e      -- Media metadata
│   ├── ffmpeg_stream_info.e     -- Stream details
│   └── ffmpeg_c_api.e           -- C externals
├── Clib/
│   ├── ffmpeg_bridge.h          -- C header
│   └── Makefile.win             -- Build config
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
| test_init | FFmpeg initializes |
| test_probe_video | Read video metadata |
| test_probe_audio | Read audio metadata |
| test_decode_frames | Decode video frames |
| test_frame_to_jpeg | Save frame as JPEG |
| test_frame_to_png | Save frame as PNG |
| test_seek | Seek to timestamp |
| test_transcode | Convert format |
| test_extract_audio | Extract audio track |
| test_error_handling | Invalid file |

### Dependencies

| Dependency | Type |
|------------|------|
| simple_file | simple_* |
| base | ISE stdlib |
| time | ISE stdlib |
| FFmpeg dev libs | External |

---

## Sources

- [FFmpeg libav tutorial](https://github.com/leandromoreira/ffmpeg-libav-tutorial)
- [FFmpeg decode_video.c](https://github.com/FFmpeg/FFmpeg/blob/master/doc/examples/decode_video.c)
- [Libavcodec Documentation](https://www.ffmpeg.org/libavcodec.html)
- [Libavformat Documentation](https://www.ffmpeg.org/libavformat.html)
- [How to decode audio streams](https://etiand.re/posts/2025/01/how-to-decode-audio-streams-in-c-cpp-using-libav/)
- [FFmpeg Wikipedia](https://en.wikipedia.org/wiki/FFmpeg)
- [FFmpeg Ultimate Guide](https://img.ly/blog/ultimate-guide-to-ffmpeg/)
