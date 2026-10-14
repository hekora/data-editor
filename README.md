# Transformice Data Editor · v1.4

An offline-first, browser-based editor for Transformice Adventures `data.bin` files and associated local game assets. The application is written in TypeScript, configured for Vite, and uses PixiJS and Howler.js for sprite preview and sound playback. No account, game server, admin permission or back-end API is required to edit your own files.

## Start the portable edition

1. Extract `Transformice-Data-Editor-v1.4-PORTABLE.zip` to a folder. **Keep the `tfmadv`, `vendor`, and `sample` folders next to `index.html`.**
2. Double-click `index.html` in Chrome or Edge. The portable distribution uses compiled, inline JavaScript and relative asset paths; it does not require Node.js.
3. The sample `data.bin` is bundled for preview. Choose **Open BIN** to import your own file.
4. Choose **Export data.bin** when finished. If you added or replaced PNG or MP3 files, choose **Export sprites/sounds ZIP** from Help (`?`) and keep the ZIP alongside the BIN when transferring the edited files.

The editor never writes directly to the original game directory without explicit user action. Some browser APIs (folder picker, direct save picker, and file-based image security rules) vary by browser and origin. If folder selection or direct overwrite is blocked, local file import and downloads remain available.

## Developer / Vite build

Requires Node.js 20+ and network access for the initial dependency installation:

```bash
npm install
npm run dev
npm run typecheck
npm run build
```

The production Vite output is in `dist/`, with `base: './'`. It can be published on any static HTTP host (including a subdirectory) **without Node.js in production**. Opening the Vite-generated ES-module output directly using `file://` may be restricted by browser security policies; use the separately provided portable edition for double-click startup.

**Build transparency:** npm registry DNS access was unavailable in this environment. As a result, the downloadable portable edition is a separate TypeScript-compiled, inline-script bundle produced by `npm run build:offline` (`tsc` plus `scripts/portable.py`), **not an executed `vite build`**. The Vite configuration and scripts are provided in source, but the production Vite build has not been verified here. The portable bundle was exercised using a Chromium integration environment that simulates local static hosting; direct `file://` navigation is restricted in this test environment.

## Interface

All editor-authored user-facing labels, dialogs, notifications, tooltips and help text are in **English**. Actual game asset names and IDs (often French in source `data.bin`) are displayed unchanged to preserve their references.

- **Anim:** Creature browser, nested symbols, multilayer timeline, keyframes, playback/loop, transform and easing controls, frame labels, copying and reordering, sprite/pivot tools, onion skin, zoom/pan and PNG snapshot.
- **VFX:** Effect list, IDs, nested clips and runtime flags.
- **Looks:** Appearance IDs and per-skin slots, image pivots, draw order, duplicate/copy pivots, and alpha-bounds pivot assistant.
- **Items:** Browse local equipment images and jump to the matching Look; add/replace PNGs.
- **SFX:** Sound IDs/names, audio playback and local MP3 replacement.
- **IDs:** Animation ID / name mapping.
- **Project:** Open BIN or JSON, optional remote BIN URL (CORS required), local folders when supported, undo/redo, exported BIN, exported animation JSON and optional asset ZIP.

The layout follows the previous Animation Studio's familiar organization: asset browser on the left, preview/player in the middle, timeline underneath, and properties on the right. Styling uses graphite with muted violet accents rather than the original bright-blue experimental theme.

## Create a custom part

1. Open a creature animation and choose a **Preview look** in the lower-right corner of the preview.
2. Click **New part #** on the timeline. The suggestion is generic (`#part_1`, `#part_2`, etc.); enter any valid custom name, e.g. `#head_extra`.
3. The editor adds a layer to the appropriate nested symbol, creates a keyframe if necessary, and registers the matching slot in the active Look. When a sibling part with the same base name exists, its transforms and pivot are used as an initial reference.
4. Import the matching PNG when prompted. For example, `#head_extra` in a Look with sprite ID `2` corresponds to `tfmadv/equipement/2/head_extra.png`.
5. The animation inspector reports whether the slot, PNG, and keyframe are available. Use **Repair part** if necessary, then adjust **Skin pivot X/Y** in the inspector or with the Pivot tool.
6. Switch **Preview look** to test other skins. A PNG and corresponding Look slot may be needed for each sprite ID.
7. Export both the `data.bin` and modified asset ZIP. A `data.bin` by itself does **not** contain the artwork.

**Nested symbols:** Use the symbol selector in the timeline or the small `↗` next to a symbol layer to open its internal layers. The **← Symbol** button returns to the previous level. Diamonds identify keyframes; occupied spans show where a sequence is active.

## Playback fixes

Playback is driven by PixiJS's ticker with a 60 FPS animation timebase. **Play/Pause no longer reconstructs the toolbar or timeline during playback**: only the current frame indicator and canvas are updated, preventing the Pause click target from disappearing mid-interaction. Space toggles Play/Pause; the previous/next, first/last, playback speed and loop controls are independently supported. End-of-timeline playback can restart or stop according to the **Loop** checkbox, which changes the preview only.

## Keyboard shortcuts

| Shortcut | Action |
|---|---|
| Space | Play / Pause |
| Left / Right | Previous / next frame |
| K | Add keyframe |
| Delete | Delete keyframe |
| P | Toggle Move / Pivot |
| G | Toggle grid |
| Ctrl/Cmd + Z / Y | Undo / Redo |
| Ctrl/Cmd + O | Open BIN |
| Ctrl/Cmd + S | Export BIN |

## Format and runtime limitations

The Rust `asset_tool` defines the binary structure (compressed payload with clips, sequences, Looks/pivots, effects, sounds and animation mappings). The BIN stores **references and definitions, not the actual PNG or MP3 bytes**. Custom parts or Looks may additionally require compatible client/runtime-side assets or identifiers. The PixiJS preview reproduces major sprite layering/transform behavior but is **not the game renderer**; accurate in-game acceptance of newly added runtime identifiers must be confirmed in a compatible client.

The editor intentionally does not implement the old site’s privileged PHP writes to server folders. It offers independent, local exports instead. Full parity with every undocumented feature of the legacy editor is not certified.

## Error prevention and diagnostics (v1.4)

- **Invalid edits are transactional.** All structural edits are checked against binary serialization and wire-format limits before being accepted. If a value exceeds the allowed range (for example, a skin pivot greater than 32767, duplicate Look slots or too many parts), the edit is rolled back and your previous data remains intact.
- **Custom `#` layer creation is linked automatically** to the selected Look and a keyframe, including when using the generic **+ Layer** command. This avoids the confusing "part added but not visible" failure of the earlier workflow.
- **PNG imports are verified by file signature and image dimensions,** rather than trusting the extension or browser MIME type. Invalid or renamed image formats are rejected without changing the current asset.
- **MP3 imports verify file headers.** Unsupported media is rejected with an English error message.
- **Broken / incompatible BIN files are rejected** without replacing the currently opened project. File size is capped at 32 MB before parsing.
- **Issues** appears in the top toolbar when any operation fails; click to inspect a session-only error log or export it as a JSON diagnostic file. No error log is uploaded.
- The playback controls stay stable after an error. The editor does not silently corrupt data.bin or modify game files on disk.

These safeguards reduce known failure modes; they do not guarantee compatibility with arbitrary future game formats or faulty assets. If an unseen game/runtime error remains, send the diagnostics JSON and the exact actions needed to reproduce it.

## Verification performed

- `tsc --noEmit` — passed.
- `tests/codec-test.cjs` — real BIN round trip with decompressed payload preserved byte-for-byte; 48 creatures, 113 VFX, 72 sounds and 204 Looks.
- `tests/browser-v3.py` — Chromium integration: standard legacy-style layout, neutral default `#part_1`, play/pause mouse and keyboard, stopped-frame stability, scrubbing, looping, custom slot PNG import, per-skin pivot, BIN export and re-import, item catalog, asset ZIP export and responsive width.
- `tests/browser-recovery.py` — intentionally invalid pivot, corrupt BIN, malformed PNG and custom-part creation via generic Layer; checks the failed changes were rolled back, the original project stayed open, playback still worked and diagnostics were downloadable.
- Direct disk navigation under `file://` and actual `vite build` were **not** independently verified due to environment limitations described above.

## Repository structure

- `src/codec.ts` — binary codec based on the server's `asset_tool` wire format.
- `src/main.ts` — editor state, UI, previews, asset import and export.
- `public/styles.css`, `public/studio-v2.css`, `public/studio-v3.css` — presentation.
- `public/vendor/` — bundled browser libraries for portable usage.
- `public/tfmadv/` — provided local sprite and audio samples.
- `tests/` — codec, browser integration and failure-recovery regression tests.
- `scripts/portable.py` — alternative no-server portable bundle builder (not the same as Vite).
- `vite.config.ts` — relative-base static Vite build configuration.

Game assets may be subject to the original rights holders' redistribution terms. Confirm permission before republishing the included artwork or audio.
