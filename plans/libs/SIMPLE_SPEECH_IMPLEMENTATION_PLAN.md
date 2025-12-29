# simple_speech Implementation Plan

**Created:** 2025-12-28
**Status:** Phase 7 COMPLETE - Metadata embedding
**Research:** /d/prod/reference_docs/research/SIMPLE_SPEECH_RESEARCH.md

---

## Phase 0-7: COMPLETE

- Phase 0: Foundation Setup - COMPLETE
- Phase 1: Core Transcription - COMPLETE
- Phase 2: Format Export - COMPLETE
- Phase 3: Video Pipeline - COMPLETE
- Phase 4: AI Enhancement - COMPLETE
- Phase 5: Batch Processing - COMPLETE
- Phase 6: Transition Detection - COMPLETE
- Phase 7: Metadata Embedding - COMPLETE

---

## Phase 7: FFmpeg Metadata Embedding (COMPLETE)

### Design Philosophy: Self-Describing Media

Most STT tools stop at text files. We embed metadata INTO the container:
- Captions embedded directly in video
- Chapters as navigable metadata
- Result: Single intelligent media artifact

### 7.1 Container Support

| Container | Captions | Chapters |
|-----------|----------|----------|
| MP4/MOV   | mov_text | Yes      |
| MKV       | SRT/ASS  | Yes      |
| WebM      | WebVTT   | Yes      |

### 7.2 Classes

**SPEECH_METADATA_GENERATOR**
- [x] generate_ffmetadata (chapters): file content
- [x] generate_srt (segments): file content
- [x] generate_vtt (segments): file content
- [x] write_metadata_file (chapters, path)

**SPEECH_VIDEO_EMBEDDER**
- [x] make (pipeline: SPEECH_PIPELINE)
- [x] embed_captions (video, segments, output): boolean
- [x] embed_chapters (video, chapters, output): boolean
- [x] embed_all (video, segments, chapters, output): boolean

**Deliverable:** Transform video into self-describing media - DELIVERED

---

## Phase 8: Real-Time Streaming (NEXT)

### Design Philosophy: Never Block Transcription

SCOOP Architecture:
- CRITICAL: Whisper transcription (never blocked)
- HIGH: Transition detection (inline)
- LOW: AI enhancement (queued, shed under load)

### 8.1 Classes

**SPEECH_STREAM**
- [ ] make_from_microphone
- [ ] set_on_segment callback
- [ ] set_on_chapter callback
- [ ] start / stop

**SPEECH_SEGMENT_BUFFER**
- [ ] Ring buffer (100 segments default)
- [ ] Non-blocking put/take

**Deliverable:** Real-time transcription with live chaptering

---

## Approval

[x] Phase 0-1 (Core)
[x] Phase 2-3 (Export + Video)
[x] Phase 4-5 (AI + Batch)
[x] Phase 6 (Transitions)
[x] Phase 7 (Embedding) - COMPLETE
[ ] Phase 8 (Streaming) - PENDING
