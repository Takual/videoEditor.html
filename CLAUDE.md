# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-file browser-based video editor (`VideoEditor.html`). No build step, no dependencies to install — open the file directly in a browser (Chrome/Edge recommended for full MediaRecorder support).

## Architecture

Everything lives in one HTML file with three sections:

1. **CSS** (lines 7–593) — CSS custom properties via `:root`, component-scoped class names
2. **HTML** (lines 595–673) — Static shell; UI is mostly static markup with JS-driven panels/overlays
3. **JavaScript** (lines 675–1346) — Vanilla JS, no framework

### Key data model

Each media item is a `clip` object:
```js
{ id, type: 'video'|'photo', file, url, name, color,
  trimStart, trimEnd, fullDuration,  // video only
  displayDuration }                  // photo only
```

`clips[]` is the source of truth for order and state. The DOM storyboard tiles are kept in sync via `syncOrder()` after drag-drop.

### Core flows

- **Import** → `addFiles()` → `loadDuration()` (video) → `renderTile()` → `updateUI()`
- **Trim** → `openTrim()` builds the bottom-sheet panel with range sliders wired directly to `clip.trimStart`/`trimEnd`/`displayDuration`
- **Preview** → `startPreview()` loops through `clips[]`, calling `pvPlayVideo()` or `pvShowPhoto()` sequentially using `async/await` + `requestAnimationFrame`
- **Export** → `startExport()` creates an offscreen `<canvas>`, captures it with `canvas.captureStream(30)`, feeds frames via `renderVideo()`/`renderPhoto()`, records with `MediaRecorder`, then triggers a download

### External dependency

SortableJS loaded from CDN (`cdn.jsdelivr.net/npm/sortablejs@1.15.0`) for drag-to-reorder.

## Browser compatibility notes

- Export requires `MediaRecorder` API — works in Chrome/Edge/Firefox; Safari has limited support
- Output format is WebM (VP9/VP8) or MP4 depending on browser support, detected at runtime via `MediaRecorder.isTypeSupported()`
- Audio is routed through Web Audio API (`createMediaElementSource` → `createMediaStreamDestination`)
