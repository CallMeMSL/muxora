# Project Description  
## Android Video Encoding Application (Tauri 2 + SolidJS + DaisyUI)

---

## 1. Overview and Objectives

The objective of this project is to develop a feature-rich **Android video encoding application** that allows users to select video files stored locally on their device and transcode them into different formats, qualities, and device-specific profiles. Conceptually, the application is inspired by desktop tools such as **HandBrake**, but redesigned for mobile usage patterns and constraints.

A **key and mandatory feature** of the application is the ability to **burn subtitles directly into the video stream** (also referred to as **hardcoded subtitles** or **subtitle burn-in**). This ensures subtitles are permanently embedded into the visual video frames and remain visible on any playback device without requiring external subtitle files.

The application will be implemented using:
- **Tauri v2 (Rust)** for the application core and backend logic  
- **SolidJS** for the reactive frontend UI  
- **DaisyUI (Tailwind-based)** for consistent styling and UI components  

---

## 2. Target Platform and User Workflow

### Platform
- **Android (primary target)**

### Typical User Workflow
1. Launch the application.
2. Select one or more video files from local device storage.
3. Choose an encoding configuration:
   - via **Templates / Presets**, or
   - via **Custom Encoding Settings**.
4. (Optional but critical) Select subtitle options:
   - choose subtitle files,
   - configure subtitle burn-in.
5. Add the job(s) to the encoding queue.
6. Start the encoding process.
7. Monitor progress and retrieve the encoded output.

---

## 3. Technology Stack and Architecture

### 3.1 Application Framework

#### Tauri 2 (Rust)
- Acts as the secure and performant application shell.
- Handles:
  - file system access,
  - background encoding jobs,
  - process management,
  - communication with the encoding engine.
- Implements job queue logic, progress parsing, and cancellation handling.

#### SolidJS (Frontend)
- Provides a highly reactive UI with minimal runtime overhead.
- Responsible for:
  - file selection UI,
  - settings forms,
  - template management,
  - queue visualization,
  - progress and error reporting.

#### DaisyUI
- Supplies standardized UI components such as:
  - forms,
  - dropdowns,
  - modals,
  - tabs,
  - progress indicators.
- Ensures a clean and maintainable UI layer with minimal custom CSS.

---

## 4. Video Encoding Engine (FFmpeg-Based)

The encoding pipeline will be based on **FFmpeg**, which is the industry-standard toolchain for video processing.

FFmpeg will be used to:
- transcode video and audio streams,
- resize and reframe video,
- apply filters,
- **burn subtitles directly into video frames**.

The Rust backend will:
- construct FFmpeg command pipelines,
- execute them safely,
- parse real-time progress output,
- handle errors and recovery.

---

## 5. Core Features and Functional Requirements

---

## 5.1 Local Video File Selection

- Browse and select video files stored on the Android device.
- Support:
  - single-file selection,
  - multi-file batch selection.
- Display basic metadata:
  - filename,
  - duration,
  - resolution,
  - codec/container (if available),
  - file size.

---

## 5.2 Encoding Configuration (Manual Mode)

### Video Settings
- Video codec (e.g. H.264 / H.265)
- Encoding mode:
  - constant quality (CRF-style),
  - target bitrate.
- Quality / bitrate value
- Encoder preset (speed vs. quality trade-off)
- Resolution handling:
  - keep original,
  - scale to predefined resolutions,
  - custom resolution.
- Frame rate:
  - keep original,
  - force custom FPS.

### Audio Settings
- Audio codec (e.g. AAC, Opus)
- Bitrate
- Channel configuration
- Audio passthrough (if supported)

### Output Settings
- Container format (MP4, MKV)
- Output directory
- File naming rules (suffixes, overwrite behavior)

---

## 5.3 Subtitle Handling (Critical Feature)

### Terminology
- **Subtitle Burn-In / Hardcoded Subtitles**  
  Subtitles are rendered directly into the video frames. They cannot be disabled during playback and do not require external subtitle support.

This feature is **mandatory** and must be fully supported.

---

### Subtitle Sources
The application must support:
- External subtitle files (e.g. `.srt`, `.ass`)
- Selecting one subtitle file per video
- (Optional extension) selecting from embedded subtitle tracks

---

### Subtitle Burn-In Configuration

Users must be able to configure subtitle burn-in behavior, including:

- Enable / disable subtitle burn-in per job
- Subtitle file selection
- Subtitle character encoding (UTF-8 default)
- Language label (for UI clarity)
- Timing offset (positive or negative delay)

---

### Subtitle Styling (If Supported by Subtitle Format)

For formats such as ASS/SSA, or via FFmpeg subtitle filters:

- Font selection
- Font size
- Text color
- Outline / shadow
- Positioning (bottom, top, custom margin)
- Safe area handling for different aspect ratios

If advanced styling is not feasible in the initial scope, the system must at minimum:
- reliably burn subtitles using default styling,
- document styling limitations clearly.

---

### Technical Implementation (Conceptual)

- Subtitle burn-in is implemented using FFmpeg video filters (e.g. `subtitles` filter).
- Subtitles are rendered **before encoding**, becoming part of the final video stream.
- Burn-in settings must be compatible with:
  - templates,
  - batch processing,
  - queue execution.

---

## 5.4 Templates / Presets System

Templates are a core usability feature.

### Template Capabilities
- Built-in templates for common use cases:
  - Mobile playback (high compatibility),
  - Small file size,
  - High-quality archive,
  - Videos with burned-in subtitles.
- User-defined templates:
  - save full encoding + subtitle configuration,
  - rename, edit, delete templates,
  - reuse templates across sessions.

Templates must include:
- video settings,
- audio settings,
- **subtitle burn-in configuration**.

---

## 5.5 Batch Processing and Job Queue

- Queue-based encoding system
- Each job contains:
  - input file,
  - selected template or custom settings,
  - subtitle configuration.
- Job states:
  - pending,
  - running,
  - completed,
  - failed,
  - cancelled.

---

## 5.6 Progress Monitoring and Error Handling

- Real-time progress reporting per job
- Estimated remaining time (best effort)
- Cancel running jobs
- Clear error messages
- Optional access to detailed FFmpeg logs for debugging

---

## 6. Non-Functional Requirements

### Performance
- Encoding is CPU-intensive and must be managed carefully.
- Sensible defaults to avoid overheating or excessive battery drain.
- Optional warnings for aggressive encoding presets.

### Stability
- Long-running tasks must not crash the app.
- Graceful failure handling and cleanup of partial outputs.

### Security and Privacy
- Local-only file processing
- No external uploads
- Minimal permission usage
- Transparent storage access handling

### Maintainability
- Modular Rust services
- Clear separation between UI, job management, and encoding logic
- Extensible design for future features (e.g. watermarking, trimming, hardware acceleration)

---

## 7. Technical Feasibility and Risk: FFmpeg on Android via Rust

### Open Technical Question
A **key open point** in this project is the **technical feasibility and implementation strategy** for running **FFmpeg on Android via a Rust-based backend (Tauri 2)**.

This aspect must be explicitly considered as part of the project scope and risk assessment.

---

### Feasibility Considerations

- **FFmpeg on Android is technically possible**, but non-trivial.
- FFmpeg must be:
  - cross-compiled for Android (ARM64, potentially ARMv7),
  - packaged correctly with the application,
  - invoked in a way compatible with Android process and permission constraints.
- Rust can:
  - invoke FFmpeg as an external binary,
  - or link against FFmpeg libraries via FFI (more complex, higher risk).

---

### Integration Approaches (Conceptual)

1. **Bundled FFmpeg Binary (Preferred for MVP)**
   - Precompiled FFmpeg binaries for Android architectures.
   - Rust backend spawns FFmpeg processes and parses stdout/stderr.
   - Advantages:
     - simpler integration,
     - closer to desktop FFmpeg usage,
     - easier debugging.
   - Risks:
     - binary size,
     - Android execution restrictions,
     - performance and battery usage.

2. **FFmpeg via Rust FFI (Advanced / High Risk)**
   - FFmpeg compiled as shared libraries.
   - Rust calls FFmpeg APIs directly.
   - Advantages:
     - tighter integration,
     - potentially better control.
   - Risks:
     - significantly higher complexity,
     - harder maintenance,
     - increased development time.

---

### Risk Mitigation Strategy

- Include an **early technical spike / proof-of-concept phase**:
  - verify FFmpeg execution on Android,
  - validate subtitle burn-in functionality,
  - confirm acceptable performance and stability.
- Treat FFmpeg-on-Android as a **validated assumption**, not a given.
- Clearly separate:
  - **MVP scope** (basic FFmpeg execution, subtitle burn-in),
  - **future optimizations** (hardware acceleration, deeper integration).

---

### Explicit Project Risk Statement

> The project depends on the successful execution of FFmpeg-based video encoding on Android devices via a Rust/Tauri backend.  
> This dependency represents a **technical risk** and must be validated early.  
> If constraints arise, alternative architectures or reduced feature scope may be required.

---

## 8. Suggested UI Structure

1. **Home / Dashboard**
2. **File Selection Screen**
3. **Encoding Settings**
   - Video
   - Audio
   - Subtitles (Burn-In)
4. **Template Management**
5. **Encoding Queue**
6. **Completed Jobs / Output Viewer**

---

## 9. Deliverables

- Android application built with:
  - Tauri 2 (Rust)
  - SolidJS
  - DaisyUI
- FFmpeg-based encoding pipeline
- Full subtitle burn-in support
- Template and batch encoding system
- Stable progress tracking and error reporting
- Technical documentation

---

## 10. Success Criteria

The project is considered successful if:

- Users can encode videos reliably on Android devices.
- Subtitle burn-in works consistently across formats and devices.
- FFmpeg execution on Android is validated and stable.
- Templates significantly reduce repetitive configuration effort.
- The application remains stable during long encoding sessions.
- The system is extensible for future feature growth.
