# simple_speech Research Notes

**Date:** 2025-12-28
**Status:** Complete
**Goal:** Design a Swiss Army knife speech-to-text library for Eiffel

---

## Step 1: Deep Web Research - Speech-to-Text Landscape

### Major Open-Source STT Engines (2025)

| Engine | Type | Accuracy | Speed | Offline | Languages |
|--------|------|----------|-------|---------|-----------|
| **whisper.cpp** | C/C++ port of Whisper | 7.4% WER avg | CPU/GPU | Yes | 99+ |
| **faster-whisper** | CTranslate2 optimized | Same | 4x faster | Yes | 99+ |
| **Whisper Large V3 Turbo** | Reduced decoder | Within 1-2% | 6x faster | Yes | 99+ |
| **Vosk** | Lightweight offline | Good | Fast | Yes | 20+ |
| **Moonshine** | Edge/mobile optimized | Good | Very fast | Yes | Limited |
| **Kaldi** | Customizable toolkit | Excellent | Varies | Yes | Any |

### Key Insight
whisper.cpp is the clear choice for local/offline use due to:
- Pure C/C++ with no dependencies
- Windows support (MSVC)
- Clean C API suitable for Eiffel wrapping
- Active development with stable API
- Model size options (tiny to large)

---

## Step 2: Technical Deep-Dive - whisper.cpp API

### Core C API Functions

- whisper_init_from_file_with_params() - Load model
- whisper_full() - PCM -> text (all-in-one)
- whisper_full_n_segments() - Get segment count
- whisper_full_get_segment_text() - Get segment text
- whisper_full_get_segment_t0/t1() - Get timing
- whisper_free() - Cleanup

### Integration Requirements

| Item | Details |
|------|---------|
| Build | CMake, links to whisper.lib |
| Header | whisper.h (~1000 lines) |
| Models | 39MB (tiny) to 2.9GB (large-v3) |
| Audio Format | 16-bit PCM, 16kHz mono |

---

## Step 3: Ecosystem Integration Analysis

### Existing simple_* Libraries for Integration

| Library | Integration Point |
|---------|-------------------|
| **simple_ffmpeg** | Extract audio from video |
| **simple_audio** | WAV loading, PCM buffers |
| **simple_ai_client** | Translation via Claude/Grok/Ollama |
| **simple_sql** | Store transcriptions in SQLite |
| **simple_json** | Parse/generate VTT/JSON formats |

---

## Step 4: User Pain Points

### Developer Pain Points
- Setup complexity (builds, models, dependencies)
- Audio preprocessing (format conversion, resampling)
- Memory management (large models)
- Streaming complexity (real-time is hard)

### End-User Pain Points
- Inaccurate auto-captions
- Poor timing sync
- No speaker identification
- Translation quality loss
- Slow processing

### Statistics
- 80% users more likely to watch video with captions
- 12% higher view time with captions
- 430 million people with disabling hearing loss
- EAA accessibility deadline June 2025

---

## Step 5: Innovation Opportunities (10 Ideas)

1. **Video Pipeline** - One-command video captioning with simple_ffmpeg
2. **Real-Time Streaming** - Live transcription with callbacks
3. **Multi-Format Export** - VTT/SRT/JSON/TXT/ASS
4. **AI Enhancement** - LLM post-processing for correction/translation
5. **Speaker Diarization** - Who said what
6. **Batch Processing** - Process entire video libraries
7. **Quality Scoring** - Confidence levels for manual review
8. **Caption Burning** - Embed captions into video
9. **Voice Commands** - Simple voice control for apps
10. **Transcript Search** - FTS5 search across transcriptions

---

## Step 6: Design Synthesis

### Core Classes

| Class | Responsibility |
|-------|----------------|
| SIMPLE_SPEECH | Main facade |
| SPEECH_SEGMENT | Segment with timing |
| SPEECH_ENGINE | Whisper.cpp wrapper |
| SPEECH_STREAM | Real-time streaming |
| SPEECH_EXPORTER | VTT/SRT/JSON export |
| SPEECH_PIPELINE | Video workflow |

### Feature Phases

1. Core Transcription (whisper.cpp, WAV)
2. Format Export (VTT/SRT/JSON)
3. Video Pipeline (simple_ffmpeg)
4. AI Enhancement (simple_ai_client)
5. Real-Time Streaming
6. Advanced (diarization, search)

---

## Step 7: Final Recommendations

### Go/No-Go: GO

### Integration Strategy
1. Do not fork whisper.cpp - just vendor whisper.h
2. Provide prebuilt whisper.lib for Windows x64
3. Document tested version (e.g., v1.7.3)
4. Model download helper script

### Priority Innovations

| Priority | Innovation | Integration |
|----------|------------|------------|
| 1 | Video Pipeline | simple_ffmpeg + simple_audio |
| 2 | Multi-Format Export | simple_json |
| 3 | AI Translation | simple_ai_client |
| 4 | Batch Processing | simple_file |
| 5 | Real-Time Stream | simple_audio |
| 6 | Search Index | simple_sql |

### Success Metrics
- Transcribe WAV in <10 lines of code
- Video to VTT in <15 lines
- Support 10+ output languages
- Real-time streaming works
- Batch 100 videos without memory leaks

---

## Sources

- https://modal.com/blog/open-source-stt
- https://github.com/ggml-org/whisper.cpp
- https://www.getsubly.com/post/srt-vtt
- https://www.w3.org/WAI/media/av/captions/
- https://www.assemblyai.com/blog/top-speaker-diarization-libraries-and-apis
