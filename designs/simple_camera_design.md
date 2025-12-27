# simple_camera Design

## Overview

**simple_camera** provides webcam and video capture for Eiffel applications using Windows Media Foundation.

## API

| Class | Purpose |
|-------|---------|
| SIMPLE_CAMERA | Main facade |
| CAMERA_DEVICE | Camera device info |
| CAMERA_FRAME | Captured frame |
| CAMERA_STREAM | Live stream |

## Key Features

```eiffel
-- Enumerate cameras
across camera.devices as d loop
    print (d.name + " [" + d.resolution_string + "]")
end

-- Capture single frame
if attached camera.capture_frame as frame then
    frame.save_as_jpeg ("photo.jpg", 85)
end

-- Live streaming
stream := camera.start_stream
stream.on_frame (agent process_frame)
-- later
stream.stop
```

## Windows API

- Media Foundation: `IMFSourceReader`, `IMFMediaSource`
- DirectShow (legacy): `IGraphBuilder`, `ICaptureGraphBuilder2`

## Phase 1 Scope

1. Enumerate cameras
2. Get camera properties
3. Capture single frame
4. Frame to image (JPEG/PNG)
5. Basic streaming

## Dependencies

- simple_ffmpeg (optional for encoding)
- Media Foundation SDK
