# simple_ffmpeg Design

## Overview

**simple_ffmpeg** provides multimedia processing capabilities for Eiffel applications by wrapping the FFmpeg C libraries (libavcodec, libavformat, libavutil). Supports video/audio encoding, decoding, transcoding, streaming, and metadata extraction.

## Design Decision: C Library Wrapper

**Choice: C library wrapper (libav*)**

| Approach | Pros | Cons |
|----------|------|------|
| libav* wrapper | Direct API, no subprocess, real-time capable, cross-platform | Complex C interop, memory management |
| CLI wrapper | Simple, stable | Subprocess overhead, parsing output |

C library approach provides maximum performance and enables real-time streaming. Uses Eric Bezault inline C pattern.

## Architecture

```
SIMPLE_FFMPEG (facade)
    ├── FFMPEG_DECODER (decode media)
    ├── FFMPEG_ENCODER (encode media)
    ├── FFMPEG_TRANSCODER (convert formats)
    ├── FFMPEG_PROBE (media info)
    ├── FFMPEG_FRAME (video/audio frame)
    └── FFMPEG_PACKET (compressed data)

C Libraries:
    └── libavcodec (encoding/decoding)
    └── libavformat (container formats)
    └── libavutil (utilities)
    └── libswscale (video scaling)
    └── libswresample (audio resampling)
```

## FFmpeg C Integration

### Required Headers
```c
#include <libavcodec/avcodec.h>
#include <libavformat/avformat.h>
#include <libavutil/avutil.h>
#include <libavutil/imgutils.h>
#include <libswscale/swscale.h>
#include <libswresample/swresample.h>
```

### Inline C Pattern
```eiffel
feature {NONE} -- C externals

    c_av_register_all
        external
            "C inline use <libavformat/avformat.h>"
        alias
            "av_register_all();"
        end

    c_avformat_open_input (a_ctx, a_url: POINTER): INTEGER
        external
            "C inline use <libavformat/avformat.h>"
        alias
            "[
                AVFormatContext **ctx = (AVFormatContext **)$a_ctx;
                return avformat_open_input(ctx, (const char *)$a_url, NULL, NULL);
            ]"
        end

    c_avcodec_find_decoder (a_codec_id: INTEGER): POINTER
        external
            "C inline use <libavcodec/avcodec.h>"
        alias
            "return avcodec_find_decoder((enum AVCodecID)$a_codec_id);"
        end
```

## Classes

### SIMPLE_FFMPEG
Main facade providing high-level operations.

```eiffel
class SIMPLE_FFMPEG

create
    make

feature {NONE} -- Initialization
    make
        -- Initialize FFmpeg libraries.
        do
            initialize_ffmpeg
            create decoders.make (5)
            create encoders.make (5)
        end

feature -- Status
    is_initialized: BOOLEAN
        -- Are FFmpeg libraries loaded?

    version: STRING_32
        -- FFmpeg version string.

    last_error: detachable STRING_32
        -- Last error message.

    has_error: BOOLEAN
        -- Did last operation fail?

feature -- Transcoding
    transcode (a_input, a_output: READABLE_STRING_GENERAL): BOOLEAN
        -- Convert media file with auto-detected settings.
        require
            input_exists: file_exists (a_input)
            output_valid: not a_output.is_empty
        ensure
            success_means_output_exists: Result implies file_exists (a_output)

    transcode_with_options (a_input, a_output: READABLE_STRING_GENERAL;
                            a_options: FFMPEG_OPTIONS): BOOLEAN
        -- Convert with explicit options.

feature -- Decoding
    open_input (a_file: READABLE_STRING_GENERAL): detachable FFMPEG_DECODER
        -- Open media file for decoding.

    open_stream (a_url: READABLE_STRING_GENERAL): detachable FFMPEG_DECODER
        -- Open network stream for decoding.

feature -- Encoding
    create_encoder (a_output: READABLE_STRING_GENERAL;
                    a_options: FFMPEG_ENCODER_OPTIONS): detachable FFMPEG_ENCODER
        -- Create encoder for output file.

feature -- Audio Operations
    extract_audio (a_video, a_audio: READABLE_STRING_GENERAL): BOOLEAN
        -- Extract audio track from video.

    convert_audio (a_input, a_output: READABLE_STRING_GENERAL;
                   a_format: READABLE_STRING_GENERAL): BOOLEAN
        -- Convert audio format (mp3, aac, flac, wav).

feature -- Video Operations
    extract_frame (a_video: READABLE_STRING_GENERAL;
                   a_time: REAL_64;
                   a_output: READABLE_STRING_GENERAL): BOOLEAN
        -- Extract single frame at timestamp.

    resize_video (a_input, a_output: READABLE_STRING_GENERAL;
                  a_width, a_height: INTEGER): BOOLEAN
        -- Resize video to specified dimensions.

feature -- Metadata
    probe (a_file: READABLE_STRING_GENERAL): detachable FFMPEG_MEDIA_INFO
        -- Get media file information.

feature -- Streaming
    stream_to_rtmp (a_input, a_rtmp_url: READABLE_STRING_GENERAL): detachable FFMPEG_STREAMER
        -- Start streaming to RTMP server.

end
```

### FFMPEG_DECODER
Decode media files/streams.

```eiffel
class FFMPEG_DECODER

create {SIMPLE_FFMPEG}
    make

feature -- Status
    is_open: BOOLEAN
    has_video: BOOLEAN
    has_audio: BOOLEAN
    duration: REAL_64
    position: REAL_64

feature -- Video Info
    video_width: INTEGER
    video_height: INTEGER
    video_frame_rate: REAL_64
    video_codec: STRING_32

feature -- Audio Info
    audio_sample_rate: INTEGER
    audio_channels: INTEGER
    audio_codec: STRING_32

feature -- Decoding
    read_frame: detachable FFMPEG_FRAME
        -- Read next decoded frame.

    seek (a_time: REAL_64)
        -- Seek to timestamp.

    read_video_frame: detachable FFMPEG_VIDEO_FRAME
        -- Read next video frame.

    read_audio_frame: detachable FFMPEG_AUDIO_FRAME
        -- Read next audio frame.

feature -- Cleanup
    close
        -- Close decoder and free resources.

end
```

### FFMPEG_ENCODER
Encode media files.

```eiffel
class FFMPEG_ENCODER

create {SIMPLE_FFMPEG}
    make

feature -- Status
    is_open: BOOLEAN
    frames_written: INTEGER

feature -- Configuration
    set_video_codec (a_codec: READABLE_STRING_GENERAL): like Current
    set_video_bitrate (a_bitrate: INTEGER): like Current
    set_resolution (a_width, a_height: INTEGER): like Current
    set_frame_rate (a_fps: REAL_64): like Current

    set_audio_codec (a_codec: READABLE_STRING_GENERAL): like Current
    set_audio_bitrate (a_bitrate: INTEGER): like Current
    set_sample_rate (a_rate: INTEGER): like Current
    set_channels (a_count: INTEGER): like Current

feature -- Encoding
    write_video_frame (a_frame: FFMPEG_VIDEO_FRAME): BOOLEAN
        -- Write video frame.

    write_audio_frame (a_frame: FFMPEG_AUDIO_FRAME): BOOLEAN
        -- Write audio frame.

feature -- Cleanup
    finalize
        -- Flush encoder and write trailer.

    close
        -- Close encoder and free resources.

end
```

### FFMPEG_FRAME
Base class for decoded frames.

```eiffel
deferred class FFMPEG_FRAME

feature -- Access
    timestamp: REAL_64
        -- Presentation timestamp in seconds.

    is_video: BOOLEAN
        deferred
        end

    is_audio: BOOLEAN
        deferred
        end

end
```

### FFMPEG_VIDEO_FRAME
Decoded video frame.

```eiffel
class FFMPEG_VIDEO_FRAME

inherit
    FFMPEG_FRAME

feature -- Access
    width: INTEGER
    height: INTEGER
    pixel_format: INTEGER

    is_video: BOOLEAN = True
    is_audio: BOOLEAN = False

feature -- Data Access
    data: MANAGED_POINTER
        -- Raw pixel data.

    to_rgb: ARRAY [NATURAL_8]
        -- Convert to RGB byte array.

    to_rgba: ARRAY [NATURAL_8]
        -- Convert to RGBA byte array.

    save_as_png (a_path: READABLE_STRING_GENERAL): BOOLEAN
        -- Save frame as PNG image.

    save_as_jpeg (a_path: READABLE_STRING_GENERAL; a_quality: INTEGER): BOOLEAN
        -- Save frame as JPEG image.

end
```

### FFMPEG_AUDIO_FRAME
Decoded audio frame.

```eiffel
class FFMPEG_AUDIO_FRAME

inherit
    FFMPEG_FRAME

feature -- Access
    sample_rate: INTEGER
    channels: INTEGER
    sample_count: INTEGER
    sample_format: INTEGER

    is_video: BOOLEAN = False
    is_audio: BOOLEAN = True

feature -- Data Access
    data: MANAGED_POINTER
        -- Raw audio samples.

    to_pcm_s16: ARRAY [INTEGER_16]
        -- Convert to signed 16-bit PCM.

    to_pcm_float: ARRAY [REAL_32]
        -- Convert to 32-bit float PCM.

end
```

### FFMPEG_MEDIA_INFO
Media file metadata.

```eiffel
class FFMPEG_MEDIA_INFO

feature -- General
    filename: STRING_32
    format_name: STRING_32
    duration: REAL_64
    size: INTEGER_64
    bit_rate: INTEGER

feature -- Video
    has_video: BOOLEAN
    video_codec: detachable STRING_32
    video_width: INTEGER
    video_height: INTEGER
    video_frame_rate: REAL_64
    video_bit_rate: INTEGER

feature -- Audio
    has_audio: BOOLEAN
    audio_codec: detachable STRING_32
    audio_sample_rate: INTEGER
    audio_channels: INTEGER
    audio_bit_rate: INTEGER

feature -- Streams
    stream_count: INTEGER
    streams: ARRAYED_LIST [FFMPEG_STREAM_INFO]

feature -- Metadata
    title: detachable STRING_32
    artist: detachable STRING_32
    album: detachable STRING_32
    metadata: STRING_TABLE [STRING_32]

end
```

### FFMPEG_TRANSCODER
High-level transcoding operations.

```eiffel
class FFMPEG_TRANSCODER

create
    make

feature -- Initialization
    make (a_input, a_output: READABLE_STRING_GENERAL)
        require
            input_valid: not a_input.is_empty
            output_valid: not a_output.is_empty

feature -- Configuration (Fluent)
    set_video_codec (a_codec: READABLE_STRING_GENERAL): like Current
    set_video_bitrate (a_bitrate: INTEGER): like Current
    set_resolution (a_width, a_height: INTEGER): like Current
    set_audio_codec (a_codec: READABLE_STRING_GENERAL): like Current
    set_audio_bitrate (a_bitrate: INTEGER): like Current
    copy_video: like Current
    copy_audio: like Current
    no_video: like Current
    no_audio: like Current

feature -- Execution
    run: BOOLEAN
        -- Execute transcoding synchronously.

    run_async: FFMPEG_TASK
        -- Execute transcoding asynchronously.

feature -- Progress
    progress: REAL_64
        -- Current progress (0.0 to 1.0).

    on_progress (a_callback: PROCEDURE [REAL_64])
        -- Set progress callback.

end
```

## Usage Examples

### Basic Transcoding
```eiffel
local
    ffmpeg: SIMPLE_FFMPEG
do
    create ffmpeg.make

    if ffmpeg.transcode ("input.avi", "output.mp4") then
        print ("Conversion complete%N")
    else
        print ("Error: " + ffmpeg.last_error + "%N")
    end
end
```

### Custom Transcoding
```eiffel
local
    transcoder: FFMPEG_TRANSCODER
do
    create transcoder.make ("input.mov", "output.mp4")
    transcoder
        .set_video_codec ("libx264")
        .set_video_bitrate (2_000_000)
        .set_resolution (1920, 1080)
        .set_audio_codec ("aac")
        .set_audio_bitrate (128_000)

    if transcoder.run then
        print ("Done%N")
    end
end
```

### Decode and Process Frames
```eiffel
local
    ffmpeg: SIMPLE_FFMPEG
    decoder: FFMPEG_DECODER
    frame: FFMPEG_VIDEO_FRAME
do
    create ffmpeg.make

    if attached ffmpeg.open_input ("video.mp4") as dec then
        from
        until
            not attached dec.read_video_frame as f
        loop
            -- Process each frame
            if f.timestamp > 5.0 and f.timestamp < 10.0 then
                f.save_as_jpeg ("frame_" + f.timestamp.out + ".jpg", 85)
            end
        end
        dec.close
    end
end
```

### Extract Audio
```eiffel
local
    ffmpeg: SIMPLE_FFMPEG
do
    create ffmpeg.make
    if ffmpeg.extract_audio ("video.mp4", "audio.mp3") then
        print ("Audio extracted%N")
    end
end
```

### Get Media Info
```eiffel
local
    ffmpeg: SIMPLE_FFMPEG
do
    create ffmpeg.make
    if attached ffmpeg.probe ("video.mp4") as info then
        print ("Duration: " + info.duration.out + " seconds%N")
        print ("Resolution: " + info.video_width.out + "x" + info.video_height.out + "%N")
        print ("Video codec: " + info.video_codec + "%N")
        print ("Audio codec: " + info.audio_codec + "%N")
    end
end
```

### Stream to RTMP
```eiffel
local
    ffmpeg: SIMPLE_FFMPEG
    streamer: FFMPEG_STREAMER
do
    create ffmpeg.make
    if attached ffmpeg.stream_to_rtmp ("webcam", "rtmp://server/live/stream") as s then
        s.start
        -- Stream runs until stopped
        execution_environment.sleep (60_000_000_000) -- 60 seconds
        s.stop
    end
end
```

## Clib Structure

```
simple_ffmpeg/
├── Clib/
│   ├── ffmpeg_bridge.h      -- Header with inline helpers
│   ├── Makefile.win         -- Windows build (links to FFmpeg DLLs)
│   └── Makefile.linux       -- Linux build
```

### ffmpeg_bridge.h
```c
#ifndef FFMPEG_BRIDGE_H
#define FFMPEG_BRIDGE_H

#include <libavcodec/avcodec.h>
#include <libavformat/avformat.h>
#include <libavutil/avutil.h>
#include <libswscale/swscale.h>
#include <libswresample/swresample.h>

// Helper macros for common operations
#define FFMPEG_VERSION_STRING av_version_info()

#endif
```

## Dependencies

| Library | Usage |
|---------|-------|
| simple_file | File path handling |

External:
- FFmpeg development libraries (libavcodec, libavformat, libavutil, libswscale, libswresample)

## Phase 1 Scope

- Initialize FFmpeg libraries
- Open/close media files
- Basic decoding (video frames, audio samples)
- Basic encoding (video frames, audio samples)
- Simple transcode operation
- Media info/probe
- Frame extraction (save as image)
- Audio extraction

## Phase 2 (Future)

- Streaming (RTMP, HLS input/output)
- Hardware acceleration (NVENC, QSV, VAAPI)
- Filter graph support
- Real-time capture (webcam, screen)
- Async processing with SCOOP

## Test Plan

1. FFmpeg initialization
2. Open video file
3. Read video frames
4. Read audio frames
5. Video frame to image
6. Basic transcode
7. Audio extraction
8. Media info parsing
9. Seek operations
10. Error handling

## Files

```
simple_ffmpeg/
├── simple_ffmpeg.ecf
├── src/
│   ├── simple_ffmpeg.e
│   ├── ffmpeg_decoder.e
│   ├── ffmpeg_encoder.e
│   ├── ffmpeg_transcoder.e
│   ├── ffmpeg_frame.e
│   ├── ffmpeg_video_frame.e
│   ├── ffmpeg_audio_frame.e
│   ├── ffmpeg_media_info.e
│   ├── ffmpeg_stream_info.e
│   ├── ffmpeg_options.e
│   ├── ffmpeg_encoder_options.e
│   └── ffmpeg_streamer.e
├── Clib/
│   ├── ffmpeg_bridge.h
│   ├── Makefile.win
│   └── Makefile.linux
├── testing/
│   ├── test_app.e
│   ├── lib_tests.e
│   └── test_set_base.e
├── docs/
│   ├── index.html
│   └── css/style.css
└── README.md
```
