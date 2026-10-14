# Transformice Data Editor · v1.6

A static, browser-based editor for Transformice Adventures `data.bin` files. By default, PNG and MP3 previews are loaded from `https://transformiceadventures.com/tfmadv/`, not bundled with the editor. Imported local assets take precedence. The application is written in TypeScript, configured for Vite, and uses PixiJS and Howler.js for sprite preview and sound playback. No account, game server, admin permission or back-end API is required to edit your own files.

## Start the portable edition

1. Extract `Transformice-Data-Editor-v1.6-PORTABLE.zip` to a folder. Keep `vendor/`, `sample/`, CSS files and `favicon.svg` beside `index.html`; there is intentionally **no `tfmadv/` directory** in the distribution.
2. Double-click `index.html` in Chrome or Edge. The portable distribution uses compiled, inline JavaScript and an external game-asset root; it does not require Node.js. Remote assets require network access, and browser CORS behavior may differ on `file://`.
3. On first launch, the editor checks `https://transformiceadventures.com/play/configADV.json` and loads the current official `data.bin` using the published `version` and `RESSOURCE`. If a local draft exists, you can choose to restore it instead. You can also choose **Open BIN** for a local file or **Game BIN** to reload the official version. A `sample/data.bin` fixture is bundled **only for offline tests**; it is not loaded automatically.
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

- **Anim:** Creature browser, nested symbols, multilayer timeline, keyframes, playback/loop, transform and easing controls, frame labels, copying and reordering, sprite/pivot tools, onion skin, native 100% zoom reset, 25–800% zoom slider/mouse wheel, and PNG snapshot. Interpolated timeline spans are clearly highlighted in purple, while static/hold spans are gray.
- **VFX:** Effect list, IDs, nested clips and runtime flags.
- **Looks:** Looks sorted numerically by displayed Look key (then image ID), categorized into expandable Fur / Hat / Face / Ears / Mouth / Tail / Shoulders / Clothing / Left hand / Right hand / Shield / Accessories / Effects / Other groups. Each matching Look appears under the relevant categories; the selected Look’s individual pieces are also grouped in the inspector. Expand with **+** and collapse with **−**. Per-skin slots/pivots, draw order, duplication/copy pivots, and alpha-bounds pivot assistant are retained. Groups are UI-only: the original BIN record order is untouched.
- **Items:** Browse equipment slots referenced by the loaded BIN and jump to the matching Look; add/replace PNGs. Without a remote directory listing or manifest, unreferenced images cannot be enumerated automatically.
- **SFX:** Sound IDs/names, audio playback and local MP3 replacement.
- **IDs:** Animation ID / name mapping.
- **Project:** Open BIN or JSON, optional remote BIN URL (CORS required), local folders when supported, undo/redo, exported BIN, exported animation JSON and optional asset ZIP.

The layout follows the previous Animation Studio's familiar organization: asset browser on the left, preview/player in the middle, timeline underneath, and properties on the right. **Drag the dock separators** to resize the left Library, right Inspector, timeline height, or layer-name column. Sizes persist locally when storage is available. On smaller screens, the sidebars become overlays. Styling uses graphite with muted violet accents rather than the original bright-blue experimental theme.

## Create a custom part

1. Open a creature animation and choose a **Preview look** in the lower-right corner of the preview.
2. Click **New part #** on the timeline. The suggestion is generic (`#part_1`, `#part_2`, etc.); enter any valid custom name, e.g. `#head_extra`.
3. The editor adds a layer to the appropriate nested symbol, creates a keyframe if necessary, and registers the matching slot in the active Look. When a sibling part with the same base name exists, its transforms and pivot are used as an initial reference.
4. Import the matching PNG when prompted. For example, `#head_extra` in a Look with sprite ID `2` corresponds to `tfmadv/equipement/2/head_extra.png`.
5. The animation inspector reports whether the slot, PNG, and keyframe are available. Use **Repair part** if necessary, then adjust **Skin pivot X/Y** in the inspector or with the Pivot tool.
6. Switch **Preview look** to test other skins. A PNG and corresponding Look slot may be needed for each sprite ID.
7. Export both the `data.bin` and modified asset ZIP. A `data.bin` by itself does **not** contain the artwork.

**Nested symbols:** Use the symbol selector in the timeline or the small `↗` next to a symbol layer to open its internal layers. The **← Symbol** button returns to the previous level. Diamonds identify keyframes. Purple bands with arrowheads show tween spans; gray bands show non-interpolated/hold spans. The highlighted band changes as the selected layer changes.

## Playback fixes

**The frame rate is fixed at 60 FPS.** The `data.bin` format does not store an independent FPS setting; this editor has no FPS selector and no FPS conversion/bake feature. The Speed menu (0.25×–3×) changes preview playback only, never the exported file. Playback is driven by PixiJS's ticker with the fixed 60 FPS animation timebase. **Play/Pause no longer reconstructs the toolbar or timeline during playback**: only the current frame indicator and canvas are updated, preventing the Pause click target from disappearing mid-interaction. Space toggles Play/Pause; the previous/next, first/last, playback speed and loop controls are independently supported. End-of-timeline playback can restart or stop according to the **Loop** checkbox, which changes the preview only.

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

## Error prevention and diagnostics (v1.4+)

- **Invalid edits are transactional.** All structural edits are checked against binary serialization and wire-format limits before being accepted. If a value exceeds the allowed range (for example, a skin pivot greater than 32767, duplicate Look slots or too many parts), the edit is rolled back and your previous data remains intact.
- **Custom `#` layer creation is linked automatically** to the selected Look and a keyframe, including when using the generic **+ Layer** command. This avoids the confusing "part added but not visible" failure of the earlier workflow.
- **PNG imports are verified by file signature and image dimensions,** rather than trusting the extension or browser MIME type. Invalid or renamed image formats are rejected without changing the current asset.
- **MP3 imports verify file headers.** Unsupported media is rejected with an English error message.
- **Broken / incompatible BIN files are rejected** without replacing the currently opened project. File size is capped at 32 MB before parsing.
- **Issues** appears in the top toolbar when any operation fails; click to inspect a session-only error log or export it as a JSON diagnostic file. No error log is uploaded.
- The playback controls stay stable after an error. The editor does not silently corrupt data.bin or modify game files on disk.

These safeguards reduce known failure modes; they do not guarantee compatibility with arbitrary future game formats or faulty assets. If an unseen game/runtime error remains, send the diagnostics JSON and the exact actions needed to reproduce it.

## Game assets / remote path (v1.5+)

The editor uses `https://transformiceadventures.com/tfmadv/` as the default root for *both* sprites and SFX (for example `equipement/2/head.png` and `sons/soins3.mp3`). The editor is suitable for static hosting under `/data-editor/` without deploying a second copy of the game's resources. From **Help → Asset source URL**, the root can be changed; optionally choose a local assets folder in supported browsers. Files imported in the editor override remote resources and can be exported as an Asset ZIP.

The editor remembers remote image-loading failures within a session and displays missing images transparently instead of repeatedly requesting the same failed URL. Unknown remote assets are shown as **not checked yet**, not falsely flagged missing before a load attempt. It also includes `favicon.svg` to avoid the old implicit `favicon.ico` request. An unavailable resource on the game server may still return an initial HTTP 404, and it must be uploaded to the game server to resolve the problem permanently. CORS for cross-origin PixiJS/WebGL previews depends on the game server; this was **not** independently verified. Once the editor is hosted on the same origin as the game, cross-origin restrictions should normally not apply (provided both host and protocol match).

Changing visual sorting, zoom and timeline display does not modify `data.bin`; they are UI-only features.

## Official game BIN startup (v1.6)

- Without an existing draft, the editor fetches the official `configADV.json`, checks its `version`, and then fetches `${RESSOURCE}/data.bin?v${version}`. The config currently advertises game version `1.11.8`, but the URL is resolved at runtime so newer versions do not require editor updates.
- If IndexedDB contains a draft, the editor offers restoration. Declining loads the current game BIN. A manually opened file takes precedence over an in-flight automatic network request, so slower responses will not replace local work.
- When using **Game BIN** to replace a modified project, the editor requires explicit `YES` confirmation. There is no server-side write; editing and exporting are fully local.
- The remote host must permit the fetch; cross-origin CORS may block the initial download while the editor is hosted on another domain. **Open BIN** remains available; do not assume that a successful mock test proves remote CORS works.
- No configurable FPS field exists. `GAME_FPS` is a fixed `60` constant and preview speed has no effect on serialization.

## Verification performed

- `tsc --noEmit` — passed.
- `tests/codec-test.cjs` — real BIN round trip with decompressed payload preserved byte-for-byte; 48 creatures, 113 VFX, 72 sounds and 204 Looks.
- `tests/browser-v3.py` — Chromium integration: standard legacy-style layout, neutral default `#part_1`, play/pause mouse and keyboard, stopped-frame stability, scrubbing, looping, custom slot PNG import, per-skin pivot, BIN export and re-import, item catalog, asset ZIP export and responsive width.
- `tests/browser-recovery.py` — intentionally invalid pivot, corrupt BIN, malformed PNG and custom-part creation via generic Layer; checks the failed changes were rolled back, the original project stayed open, playback still worked and diagnostics were downloadable.
- `tests/browser-v15.py` — Chromium network-mocked integration: absolute game asset URLs, numeric Look ordering without BIN mutations, native 100% zoom, slider, tween/hold spans and Play/Pause.
- `tests/browser-v16.py` — Chromium network-mocked integration: `configADV.json` / versioned BIN default startup, grouped Looks and left/right category controls, fixed 60 FPS with preview-only Speed, drag-to-resize Library, and playback consistency.
- `tests/browser-v16-origin.py` — additional test for local-origin draft restore/decline, resize persistence, and project replacement. **Skipped in this environment** because Chromium navigation to localhost is blocked by administrator policy; run it on an unrestricted workstation.
- Direct disk navigation under `file://`, live remote-server availability/CORS, and actual `vite build` were **not** independently verified due to environment limitations described above.

## Repository structure

- `src/codec.ts` — binary codec based on the server's `asset_tool` wire format.
- `src/main.ts` — editor state, UI, previews, asset import and export.
- `public/styles.css`, `public/studio-v2.css`, `public/studio-v3.css` — presentation.
- `public/vendor/` — bundled browser libraries for portable usage.
- `public/favicon.svg` — editor icon. There is **no bundled `public/tfmadv/` directory**; PNGs and MP3s load from the game host by default.
- `tests/` — codec, browser integration and failure-recovery regression tests.
- `scripts/portable.py` — alternative no-server portable bundle builder (not the same as Vite).
- `vite.config.ts` — relative-base static Vite build configuration.

No game sprites or audio are bundled with this edition; only their references in the example BIN are included. Confirm permission before publishing any separate custom art or audio.
