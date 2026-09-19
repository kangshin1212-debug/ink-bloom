# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Ink Bloom is a single-file browser canvas drawing toy that simulates ink bleeding into paper. It's pure HTML/CSS/JavaScript with no build step, no package manager, and no dependencies — everything lives in `index.html`.

## Running it

There is no build, lint, or test tooling in this repo. To view changes:

```bash
npx serve .
```

Or simply open `index.html` directly in a browser (double-click it). There is no compile/bundle step — edits to `index.html` are visible on reload.

## Architecture

All logic lives in the single `<script>` block at the bottom of `index.html`, structured around one IIFE:

- **`PAPERS` config object** — defines the two paper modes (`light`/`dark`), each with a background color, a canvas `globalCompositeOperation` blend mode (`multiply` for light paper, `lighter` for dark paper so ink glows instead of darkening), and a set of named color palettes (pairs of HSL base colors). Switching paper mode resets the palette index and clears the canvas.
- **Bloom lifecycle** — `spawnBloom()` creates a bloom record (position, radius, HSL color with randomized jitter, alpha, an irregular polygon of radii for an organic blob shape, and a start timestamp/duration) and pushes it into the `activeBlooms` array. `scatterSplatter()` draws small immediate ink-speckle dots around the spawn point. Every animation frame (`frame()` via `requestAnimationFrame`), `drawBloom()` grows each active bloom's radius/alpha using easing curves (`easeOutCubic`/`easeOutQuad`) until its duration elapses, then it's removed from `activeBlooms`.
- **Blob shape** — `blobPath()` builds the organic (non-circular) ink-blot outline by scattering points around a circle with per-point radius jitter (`makeRadii`) and connecting them with quadratic curves through midpoints, with an optional time-based wobble.
- **Input handling** — Pointer Events (`pointerdown`/`pointermove`/`pointerup`/`pointercancel`/`pointerleave`) unify mouse and touch. Dragging spawns blooms as the pointer moves (throttled by distance/time since the last spawn, with size scaled by drag speed); a short tap/click with no movement spawns one larger bloom instead.
- **Canvas sizing** — `resize()` accounts for `devicePixelRatio` (capped at 2), snapshots the current canvas as a data URL, resizes, then redraws the snapshot back on top so in-progress drawings survive window resizes.
- **UI panel** — paper-mode buttons, palette swatches, a brush-size range input, and clear/save buttons are plain DOM elements wired up with event listeners at the bottom of the script; `saveBtn` exports the canvas via `canvas.toDataURL("image/png")`.

## Working in this codebase

- Keep everything in `index.html` — this project intentionally has no build system or module structure.
- When adding a new palette or paper mode, follow the existing `PAPERS` shape (bg, blend, palettes with `name` + `colors` HSL pairs) so `renderPalettes()` and `setPaperMode()` continue to work without changes.
