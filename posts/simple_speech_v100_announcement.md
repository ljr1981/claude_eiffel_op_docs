# simple_speech v0.9.0-beta Release Announcement

**Date:** December 28, 2024
**Author:** Larry Rix
**Status:** Beta Release (Phases 0-7 Complete — Testers Welcome!)

---

## 🎉 Introducing simple_speech: Local-First Media Intelligence Pipeline

We are excited to announce the beta release of **simple_speech v0.9.0** — a local-first speech-to-text and media-structuring engine that transforms raw audio and video into navigable, captioned, chaptered media. We're looking for testers to help us shake out any remaining issues before the 1.0 release.

This is not just another transcription library. **simple_speech makes media self-describing — automatically.**

---

## What Makes simple_speech Different?

| Capability | Market Status | simple_speech |
|------------|---------------|---------------|
| Transcription | Commodity | ✅ Yes |
| Captions (SRT/VTT) | Expected | ✅ Yes |
| Auto-Chapters | Rare | ✅ Yes |
| Embedded Metadata | Extremely Rare | ✅ Yes |
| Offline Determinism | High-Trust Differentiator | ✅ Yes |

While many tools can transcribe speech, **simple_speech goes further**:

1. **Automatic Chapter Detection** — Analyzes transcription content to identify topic transitions
2. **Metadata Embedding** — Embeds captions AND chapters directly into video containers
3. **Complete Offline Operation** — No cloud APIs required; your data never leaves your machine
4. **Deterministic Results** — Same input always produces same output

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         SPEECH_QUICK (Facade)                       │
│        One-liner API for common workflows                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────┐  ┌────────────────────┐  ┌──────────────────┐ │
│  │ SPEECH_PIPELINE │  │ SPEECH_TRANSITION  │  │ SPEECH_VIDEO     │ │
│  │                 │  │ _DETECTOR          │  │ _EMBEDDER        │ │
│  │ • FFmpeg probe  │  │                    │  │                  │ │
│  │ • Audio extract │  │ • Gap analysis     │  │ • Captions       │ │
│  │ • Whisper STT   │  │ • Keyword signals  │  │ • Chapters       │ │
│  │ • Multi-format  │  │ • Sensitivity      │  │ • FFMETADATA     │ │
│  └────────┬────────┘  └─────────┬──────────┘  └────────┬─────────┘ │
│           │                     │                      │           │
│           └─────────────────────┼──────────────────────┘           │
│                                 │                                   │
│  ┌──────────────────────────────┴──────────────────────────────┐   │
│  │                     Export Layer                             │   │
│  │  VTT_EXPORTER │ SRT_EXPORTER │ JSON_EXPORTER │ TXT_EXPORTER │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                     │
├─────────────────────────────────────────────────────────────────────┤
│                     External Dependencies                           │
│  ┌──────────────────┐  ┌───────────────────┐  ┌─────────────────┐  │
│  │ whisper.cpp      │  │ FFmpeg            │  │ simple_ffmpeg   │  │
│  │ (via C DLL)      │  │ (CLI)             │  │ (Eiffel wrapper)│  │
│  └──────────────────┘  └───────────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## The SPEECH_QUICK Facade

The centerpiece of this release is the **SPEECH_QUICK** facade — a fluent API that wraps the entire pipeline for maximum simplicity:

### One-Liner: Full Video Processing

```eiffel
local
    quick: SPEECH_QUICK
do
    create quick.make_with_model ("models/ggml-base.en.bin")
    if quick.process_video ("raw.mp4", "finished.mp4") then
        print ("Success: " + quick.segment_count.out + " segments, "
               + quick.chapter_count.out + " chapters%N")
    end
end
```

This single call:
- Probes the video for audio streams
- Extracts audio as 16kHz mono WAV
- Transcribes with Whisper
- Detects chapter boundaries
- Embeds captions + chapters into output video
- Cleans up temp files

### Fluent Chain: Full Control

```eiffel
local
    quick: SPEECH_QUICK
    l_dummy: like quick
do
    create quick.make_with_model ("models/ggml-base.en.bin")
    l_dummy := quick.transcribe ("lecture.mp4")
                    .set_sensitivity (0.6)
                    .set_min_chapter_duration (60.0)
                    .detect_chapters
                    .export_vtt ("captions.vtt")
                    .export_srt ("captions.srt")
                    .export_chapters_json ("chapters.json")
                    .embed_to ("output.mp4")
end
```

---

## Export Formats

simple_speech exports to all major subtitle formats:

| Format | Extension | Use Case |
|--------|-----------|----------|
| WebVTT | .vtt | Web video, HTML5 |
| SubRip | .srt | Universal compatibility |
| JSON | .json | Programmatic processing |
| Plain Text | .txt | Reading, search indexing |

### Chapter Export

Chapters can be exported separately:
- **JSON** — Machine-readable chapter list with timestamps
- **VTT** — Chapter markers for players that support them
- **Embedded** — FFMETADATA format inside video containers

---

## Chapter Detection Algorithm

The `SPEECH_TRANSITION_DETECTOR` uses multiple signals to identify chapter boundaries:

1. **Silence Gaps** — Extended pauses often indicate topic changes
2. **Keyword Signals** — Words like "next", "now", "moving on", "chapter"
3. **Configurable Sensitivity** — Three levels (low/medium/high)
4. **Minimum Duration** — Prevents micro-chapters

```eiffel
-- Example: Detect ~5 chapters in an hour-long video
l_dummy := quick.set_sensitivity (0.7)           -- Higher = fewer chapters
                .set_min_chapter_duration (300.0) -- 5-minute minimum
                .detect_chapters
```

---

## Test Suite Results

All 14 library tests pass:

```
Running lib_tests...
✓ test_segment_creation
✓ test_segment_timing
✓ test_segment_confidence
✓ test_chapter_creation
✓ test_chapter_timing
✓ test_vtt_exporter
✓ test_srt_exporter
✓ test_json_exporter
✓ test_detector_creation
✓ test_detector_sensitivity
✓ test_detector_min_duration
✓ test_quick_creation
✓ test_quick_status
✓ test_memory_monitor

14 tests passed, 0 failed
```

---

## Demo Applications

Five demo applications showcase different capabilities:

### 1. demo_export
Basic transcription and export workflow.
```
Processing: sample.wav
Transcribed 21 segments in 3.2s
Exported: sample.vtt, sample.srt, sample.json, sample.txt
```

### 2. demo_pipeline
Full video processing with SPEECH_PIPELINE.
```
Processing: sintel.mp4
Duration: 888.02 seconds
Transcribed 92 segments
Exported to all formats
```

### 3. demo_batch
Memory-conscious sequential processing of multiple files.
```
Processing 3 videos...
✓ video1.mp4 → video1.vtt (0:45)
✓ video2.mp4 → video2.vtt (0:52)
✓ video3.mp4 → video3.vtt (1:07)
Batch complete: 3 videos in 2m 44s
```

### 4. demo_chapters
Chapter detection and export.
```
Processing: sintel.mp4
Detected 13 chapters:
  00:00:00 - Introduction
  00:01:15 - The Journey Begins
  00:03:42 - First Encounter
  ...
Exported: chapters.json, chapters.vtt
```

### 5. demo_embed
Full embedding with verification.
```
Input: sintel.mp4
Embedding captions (92 segments) + chapters (13)
Output: sintel_captioned.mp4
Verification: ✓ Subtitle stream present
              ✓ 13 chapters in metadata
```

---

## Installation

### Prerequisites

1. **EiffelStudio 25.02** — Compiler
2. **FFmpeg** — Audio extraction and video processing
3. **whisper.cpp** — Speech recognition (GGML models)

### Whisper Model Selection

| Model | Size | Speed | Accuracy | Recommended For |
|-------|------|-------|----------|-----------------|
| ggml-tiny.en.bin | 75 MB | Very Fast | Good | Testing |
| ggml-base.en.bin | 142 MB | Fast | Better | General use ★ |
| ggml-small.en.bin | 466 MB | Medium | Great | Production |
| ggml-medium.en.bin | 1.5 GB | Slow | Excellent | Quality priority |
| ggml-large-v3.bin | 3.1 GB | Very Slow | Best | Maximum accuracy |

### ECF Integration

```xml
<library name="simple_speech"
         location="$SIMPLE_EIFFEL/simple_speech/simple_speech.ecf"/>
```

---

## Documentation

Comprehensive documentation is available:

- **README.md** — Quick start and installation
- **docs/index.html** — Full API reference
- **docs/cookbook.html** — 13 ready-to-use recipes

### Cookbook Highlights

| Recipe | Description |
|--------|-------------|
| #1 | Transcribe video to VTT |
| #4 | Auto-chapter a long video |
| #7 | Create self-describing video |
| #10 | Full fluent workflow |
| #13 | Error handling patterns |

---

## Related Library Updates

### simple_ffmpeg

Added configurable path setters for FFmpeg binaries:

```eiffel
ffmpeg.set_ffmpeg_path ("D:\ffmpeg\bin\ffmpeg.exe")
ffmpeg.set_ffprobe_path ("D:\ffmpeg\bin\ffprobe.exe")
```

This allows simple_speech to locate FFmpeg when it's not in PATH.

### simple_sql

Expanded todo_app support with:
- Task dependencies
- Tags/labels
- Enhanced data model
- SQLite persistence

### simple_tui (Major Feature Release!)

**New: Full Task Manager TUI Application** (~4,000 lines added):

| Component | Description |
|-----------|-------------|
| TASK_MANAGER_APP | Complete task management interface |
| TUI_QUICK | Fluent builder API for rapid UI construction |
| TUI_INPUT_DIALOG | Multi-field input dialogs with combo boxes |

**Task Manager Capabilities:**
- Create/edit/delete tasks with title, description, priority, due date
- Status workflow: pending -> in_progress -> waiting -> completed -> archived
- Context tagging: office, home, phone, errands
- Subtask support with parent/child relationships
- Multiple view filters (all, pending, in progress, completed, by context)

**AI Integration (Claude, Grok, Ollama):**
- Natural language task parsing
- Automatic subtask suggestions
- AI-powered blocker resolution
- Configurable provider routing

---

## Development Journey

simple_speech followed our standard 8-phase development cycle:

| Phase | Description | Status |
|-------|-------------|--------|
| Phase 0 | Research & RFC | ✅ Complete |
| Phase 1 | Core STT (Whisper) | ✅ Complete |
| Phase 2 | Pipeline Architecture | ✅ Complete |
| Phase 3 | Export Formats | ✅ Complete |
| Phase 4 | Chapter Detection | ✅ Complete |
| Phase 5 | SPEECH_QUICK Facade | ✅ Complete |
| Phase 6 | Batch Processing | ✅ Complete |
| Phase 7 | Metadata Embedding | ✅ Complete |

### Key Technical Decisions

1. **DLL-based Whisper** — whisper.cpp compiled as DLL for Eiffel interop
2. **Temp File Strategy** — Extract audio to temp WAV, auto-cleanup
3. **FFMETADATA Format** — Standard format for chapter embedding
4. **Fluent API Pattern** — All methods return Current for chaining

---

## What's Next?

Potential future enhancements:

1. **AI-Enhanced Chapters** — Use LLMs for smarter title generation
2. **Speaker Diarization** — Identify different speakers
3. **Language Detection** — Auto-detect source language
4. **Real-time Mode** — Stream processing for live content
5. **GPU Acceleration** — CUDA support for faster transcription

---

## Acknowledgments

simple_speech builds on excellent open-source foundations:

- **whisper.cpp** by Georgi Gerganov — Core speech recognition
- **FFmpeg** — Media processing backbone
- **EiffelStudio** — Rock-solid compiler and runtime

---

## Get Started Today

```eiffel
-- Transform any video in 3 lines
local quick: SPEECH_QUICK
do
    create quick.make_with_model ("models/ggml-base.en.bin")
    if quick.process_video ("raw.mp4", "captioned.mp4") then
        print ("Done! Your video is now self-describing.%N")
    end
end
```

**Repository:** [github.com/simple-eiffel/simple_speech](https://github.com/simple-eiffel/simple_speech)
**License:** MIT
**Version:** 0.9.0-beta

---

*simple_speech — Because media should describe itself.*
