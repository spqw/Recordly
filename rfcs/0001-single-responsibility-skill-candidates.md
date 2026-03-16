# RFC 0001: Single-Responsibility Skill Candidates From Recordly

## Status

Draft

## Context

Recordly is not a single-purpose app from an implementation perspective. It combines:

- Electron window orchestration and IPC
- Platform-specific capture backends
- Native macOS and Windows helper builds
- A renderer-driven export pipeline
- Timeline and playback editing
- Project persistence
- Localization validation

That makes it a good source for Codex skills, but only if the boundaries stay narrow. A useful skill should map to a workflow that is:

- recurrent inside the repo
- instruction-heavy enough to justify a skill
- separable from unrelated UI work
- testable or verifiable with a short checklist

This RFC identifies the strongest candidates for single-responsibility skills based on the current repository layout and architecture.

## Selection Criteria

The candidates below were favored when they met most of these conditions:

1. The workflow crosses multiple files or tools and benefits from a repeatable procedure.
2. The workflow has platform quirks, data contracts, or validation steps that are easy to forget.
3. The workflow can be triggered from a clear user request without dragging in the whole app architecture.
4. The workflow is more procedural than product-specific.

The candidates below were rejected when they were too broad, too UI-coupled, or mostly ordinary React editing.

## Recommended Skill Candidates

### 1. `recordly-capture-backends`

**Responsibility**

Own changes and debugging for screen/audio capture flows across macOS, Windows, and Linux.

**Why this is a good skill**

- The workflow spans renderer hooks, Electron IPC, native subprocess orchestration, permissions, and per-platform fallbacks.
- The architecture is fragile and platform-specific.
- A skill can encode the investigation path instead of requiring re-discovery each time.

**Primary code surfaces**

- `src/hooks/useScreenRecorder.ts`
- `electron/ipc/handlers.ts`
- `electron/main.ts`
- `electron/preload.ts`
- `electron/windows.ts`

**Likely bundled references or scripts**

- Capture flow map by platform
- Permission checklist for macOS
- WGC / FFmpeg fallback notes for Windows
- Expected temp-file and telemetry outputs

**Typical requests**

- Fix broken window recording on Windows
- Diagnose missing permissions on macOS
- Adjust microphone or system-audio behavior
- Trace the stop-recording handoff into the editor

**Boundary**

Do not include export rendering, timeline editing, or localization.

### 2. `recordly-export-pipeline`

**Responsibility**

Own video and GIF export pipeline work from decode through render, encode, mux, and progress reporting.

**Why this is a good skill**

- The export stack is dense, specialized, and performance-sensitive.
- The workflow includes WebCodecs, demuxing, PIXI rendering, muxing, and audio processing.
- A skill can carry the mental model and verification checklist for safe changes.

**Primary code surfaces**

- `src/lib/exporter/videoExporter.ts`
- `src/lib/exporter/gifExporter.ts`
- `src/lib/exporter/frameRenderer.ts`
- `src/lib/exporter/streamingDecoder.ts`
- `src/lib/exporter/muxer.ts`
- `src/lib/exporter/audioEncoder.ts`

**Likely bundled references or scripts**

- Export pipeline sequence diagram
- Performance and memory triage checklist
- Codec/container notes
- Minimal test commands for exporter-focused changes

**Typical requests**

- Fix export hangs or timing drift
- Improve GIF output quality or speed
- Investigate audio/video sync problems
- Change progress reporting or output sizing logic

**Boundary**

Do not include capture acquisition or editor-side timeline authoring except where export consumes those structures.

### 3. `recordly-project-files`

**Responsibility**

Own `.recordly` project serialization, normalization, versioning, migration, and open/save flows.

**Why this is a good skill**

- The workflow has a crisp data contract and clear compatibility concerns.
- The persistence layer is reusable and narrow.
- A skill can preserve migration discipline and validation rules.

**Primary code surfaces**

- `src/components/video-editor/projectPersistence.ts`
- `src/components/video-editor/VideoEditor.tsx`
- `electron/ipc/handlers.ts`

**Likely bundled references or scripts**

- Project schema reference
- Backward-compatibility checklist
- File URL and path normalization notes
- Sample legacy and current project fixtures

**Typical requests**

- Add a new editor field to saved projects
- Fix broken project reopening on Windows paths
- Migrate legacy `.openscreen` behavior
- Validate save/load integrity after editor changes

**Boundary**

Do not include export internals or generic editor UI work beyond save/load integration.

### 4. `recordly-i18n-maintainer`

**Responsibility**

Own locale file updates, namespace discipline, fallback behavior, and translation-structure verification.

**Why this is a good skill**

- The workflow is narrow and procedural.
- The repo already has explicit contributor guidance and a validation script.
- This is a strong candidate for a low-risk, high-repeatability maintenance skill.

**Primary code surfaces**

- `src/i18n/config.ts`
- `src/contexts/I18nContext.tsx`
- `src/i18n/locales/*/*.json`
- `scripts/i18n-check.mjs`
- `TRANSLATION_GUIDE.md`

**Likely bundled references or scripts**

- Namespace map
- Key naming rules
- Locale sync checklist
- Validation command usage

**Typical requests**

- Add strings for a new feature
- Audit missing or extra translation keys
- Introduce another locale
- Move UI strings into the existing namespace system

**Boundary**

Do not include broader copywriting or unrelated React refactors.

### 5. `recordly-native-helper-builds`

**Responsibility**

Own native helper compilation flows and packaging integration for macOS Swift helpers and Windows C++ helpers.

**Why this is a good skill**

- The workflow combines platform toolchains, build scripts, binary naming, and packaging expectations.
- Failures are often environment-specific and expensive to re-learn.
- The responsibility is narrow enough to avoid becoming a generic “build everything” skill.

**Primary code surfaces**

- `scripts/build-native-helpers.mjs`
- `scripts/build-wgc-capture.mjs`
- `scripts/build-cursor-monitor.mjs`
- `electron/native/`
- `package.json`

**Likely bundled references or scripts**

- Toolchain prerequisites by platform
- Binary output locations and expected names
- Common failure modes for `swiftc` and CMake
- Packaging touchpoints in Electron build scripts

**Typical requests**

- Fix native helper compilation failures
- Rename or add helper binaries
- Update packaging expectations after native changes
- Diagnose missing helper artifacts in local builds

**Boundary**

Do not include capture logic itself unless the issue is clearly in the build or packaging path.

### 6. `recordly-cursor-telemetry-and-rendering`

**Responsibility**

Own cursor telemetry ingestion, smoothing, looping, visual asset selection, and playback/export rendering behavior.

**Why this is a good skill**

- Cursor behavior is a distinct product differentiator in this repo.
- The implementation is spread across native telemetry, playback math, editor controls, and export rendering.
- The domain has enough specialized terminology and heuristics to justify a dedicated skill.

**Primary code surfaces**

- `src/components/video-editor/videoPlayback/`
- `src/components/video-editor/VideoPlayback.tsx`
- `src/components/video-editor/SettingsPanel.tsx`
- `src/hooks/useScreenRecorder.ts`
- `electron/ipc/handlers.ts`
- `electron/native/NativeCursorMonitor.swift`
- `electron/native/SystemCursorAssets.swift`

**Likely bundled references or scripts**

- Cursor telemetry format reference
- Smoothing and motion-blur tuning checklist
- Loop-behavior edge cases
- Asset fallback rules across platforms

**Typical requests**

- Fix cursor jitter or missing clicks
- Tune cursor loop behavior
- Diagnose wrong cursor asset selection
- Keep playback and export cursor rendering consistent

**Boundary**

Do not include generic zoom logic unless the request explicitly couples cursor and zoom behavior.

## Secondary Candidates

These are viable, but weaker than the six above.

### `recordly-timeline-editing`

Pros:

- Clear file cluster under `src/components/video-editor/timeline/`
- Contains non-trivial editing rules and interaction logic

Concerns:

- Too coupled to the monolithic `VideoEditor.tsx`
- Risk of turning into a generic “editor feature” skill

### `recordly-annotation-tools`

Pros:

- Fairly bounded feature area
- Clear user-facing responsibility

Concerns:

- Smaller workflow surface
- Less reusable than export, capture, or project persistence

### `recordly-window-orchestration`

Pros:

- Electron window types and lifecycle are distinct

Concerns:

- Useful mostly for internal architecture work
- Too narrow unless window-management work recurs often

## Candidates That Should Not Become Skills

### “Recordly frontend”

Too broad. This would collapse multiple unrelated responsibilities into a vague app-wide skill.

### “Video editor everything”

Too large and too coupled to one monolithic component. It would not help with context hygiene.

### “Recordly maintenance”

Too generic. The skill would provide little guidance beyond ordinary repository exploration.

## Recommended Rollout Order

1. `recordly-i18n-maintainer`
2. `recordly-project-files`
3. `recordly-native-helper-builds`
4. `recordly-export-pipeline`
5. `recordly-capture-backends`
6. `recordly-cursor-telemetry-and-rendering`

This order starts with the most bounded, reusable, and easy-to-validate skills before moving into platform-fragile or performance-sensitive workflows.

## Proposed Next Step

Create the first two skills first:

- `recordly-i18n-maintainer`
- `recordly-project-files`

Those two have the cleanest boundaries, the lowest coordination cost, and the highest chance of producing immediately reusable skill folders without overfitting to transient implementation details.
