# Transformice Data Editor

Transformice Data Editor is a browser-based workspace for inspecting, previewing and editing the `data.bin` asset definitions used by **Transformice Adventures**. Its main focus is character animation, with additional tools for effects, character appearances, equipment image references, sound references and animation identifiers.

The application runs entirely on the client. Editing does **not** require a game account, administrator privileges, a database, or a custom back-end. You can open a local `data.bin`, work with the official game data, and download your modified files. The editor does not silently write to the game's servers.

**Important distinction:** `data.bin` contains structured references and animation data; the PNG and MP3 assets referenced by those records are separate files. An exported BIN does not automatically include newly drawn sprites or replacement audio.

## Interface and navigation

The workspace is organized to resemble the familiar Animation Studio layout:

- **Library (left):** search and select creatures, effects, Looks, equipment references, sounds and animation IDs.
- **Preview (center):** inspect the selected clip with game sprites; move parts, adjust pivots, change the active Look, inspect onion-skin frames and navigate around the canvas.
- **Timeline (below preview):** enter nested Poses, select layers, create and edit keyframes, set interpolations and control playback.
- **Inspector (right):** edit properties of the selected asset, animation, Pose, layer or keyframe.

Resize the library, inspector, timeline height and layer-name column by dragging their separators. On narrow screens the side panels can be collapsed. The application uses a single stylesheet, `studio.css`, for the entire interface.

All interface labels, dialogs, help text and diagnostics are written in **English**. Original creature, clip, image and effect names from the game remain unchanged, including names written in French, because these are meaningful identifiers.

## Animation editor

### Playback and scene preview

- The game animation timebase is **fixed at 60 FPS**. There is no FPS selector and no frame-rate conversion operation.
- **Speed** changes only the preview playback rate (from 0.25× to 3×). It does not change keyframes or the exported BIN.
- Use Play/Pause, frame stepping, loop and direct frame entry to inspect poses and transitions.
- The preview renders sprites with PixiJS and uses persistent scene objects where possible, rather than rebuilding every sprite on each tick.
- **Preview FPS** reports the actual rate at which the editor is drawing on the current computer. This performance measurement is separate from the game's 60 FPS timebase.
- Canvas controls include Move, Pivot, grid, flip, onion skin, zoom, reset to **100%**, and a zoom slider.
- **Right mouse drag** pans the camera without changing the animation. Middle drag and available empty-space drag gestures also support navigation.
- A PNG snapshot of the preview can be exported.

The preview aims to resemble the in-game appearance, but it is a browser renderer, not an embedded instance of the game's full rendering engine.

### Timeline and keyframes

The timeline uses a track-based layout with a numbered frame ruler, a prominent playhead, a distinct selected-keyframe state and horizontal duration indicators:

- Create, select and remove layers and keyframes within a Pose.
- Move keyframes by **dragging their diamonds horizontally** on the same layer. Frame snapping, occupied-frame restrictions and boundary checks prevent invalid placements.
- Edit keyframe transforms (position, rotation, scale, skew, opacity, tint when supported) and interpolation/easing properties in the Inspector.
- Distinguish interpolated (**Tween**) spans from non-interpolated (**Hold**) spans without hiding keyframe markers.
- Scrub by dragging on the ruler, jump directly to a frame, and use independent timeline zoom controls.
- Navigate nested Poses using the timeline Pose selector, layer links and the return control.
- Use Undo/Redo for supported edits, including keyframe movement.

Keyframe moves do not automatically edit other layers, and overlapping legacy frame intervals are handled conservatively rather than rewritten without confirmation.

### Poses and layers

A **Pose** is the editor's user-facing name for an animation symbol (`SymbolClip` in the data model). This is terminology used by the interface; no additional binary type is introduced.

**New Pose** provides three starting points:

1. **Empty Pose:** create a blank Pose with a custom name and duration.
2. **From statique:** initialize a Pose from the layers and initial transforms of a creature's available `statique` animation. Referenced nested Pose dependencies are copied as needed.
3. **Duplicate current:** duplicate the currently selected Pose and its existing animation data.

You may insert a new Pose as a layer of the current Pose to see it in context. The editor checks references to avoid Pose dependency cycles.

**Add Layer** provides three explicit types:

- **Part:** a named Look slot beginning with `#`, for example `#head_extra`.
- **Pose:** a reference to an existing nested Pose.
- **Image:** a PNG sprite reference, usually expressed as `@filename.png`.

These selections use the game's existing clip/layer structures and remain compatible with the data model.

### Custom parts, pivots and Looks

A part beginning with `#` resolves through the currently selected Look. Creating an entry in the timeline alone does not supply an image; the Look and sprite must also be available.

To create a new part:

1. Open the intended creature and animation, then choose a **Preview look**.
2. Enter the nested Pose that contains the character's relevant body layers.
3. Select **New part #** or **Add Layer → Part**, then choose a unique name. Suggestions are generic, such as `#part_1`.
4. Check the new layer and its keyframe. The editor registers its Look slot in the active appearance and can use a corresponding part as an initial alignment reference.
5. Import or provide its actual PNG. For a Look with sprite/image ID `2`, `#head_extra` typically resolves to `tfmadv/equipement/2/head_extra.png`.
6. Use the Inspector or the Pivot tool to adjust the appearance-specific anchor. Test other Looks to see whether their matching slots and images are present.
7. Export the BIN **and** the added/replaced images when sharing your work.

The editor distinguishes an untested image from a genuinely unavailable image. Problems such as missing slots, sprites or keyframes are displayed in the selected-part context. It can also copy/reuse pivots between appearances and help find a pivot from the visible pixel bounds of an imported PNG.

The **Looks** section sorts entries numerically by displayed ID and organizes them into expandable groups such as **Fur, Hat, Face, Ears, Mouth, Tail, Shoulders, Clothing, Left Hand, Right Hand, Shield, Accessories, Effects and Other**. These categories are search/navigation aids and do not change the original ordering or encoding of the BIN records. A Look may appear in more than one applicable group.

## Other data sections

| Section | Purpose |
|---|---|
| **VFX** | Browse and modify effect definitions, their animation clips and supported effect properties. |
| **Looks** | Inspect appearance records, part slots, sprite IDs, ordering and pivot information. |
| **Items** | Browse equipment sprite references associated with available data and jump to the related Look or image. |
| **SFX** | Find sound references by name/ID, preview supported audio with Howler.js and import replacement MP3s. |
| **IDs** | Inspect and edit supported animation name/ID mappings. |

The Items browser is reference-based: it cannot discover every unreferenced asset on the game server without a separate directory index or manifest.

## Loading projects and resolving assets

### Official game data

When there is no restorable local draft, the editor attempts to load the official game BIN. It first requests:

`https://transformiceadventures.com/play/configADV.json`

It reads the resource base URL and published game version, then builds the corresponding versioned `data.bin` URL. The application does not hard-code a particular game version. **Game BIN** lets you request a fresh copy later; a project with unexported changes requires explicit confirmation before replacement. If you open a local BIN while the automatic request is running, the delayed remote response must not replace your local work.

### Local and custom sources

- **Open BIN:** import a local `data.bin`.
- **JSON / URL:** import or inspect supported JSON representations or load a user-supplied BIN URL, subject to browser access restrictions.
- **Local assets:** provide a local folder or individual PNG/MP3 resources when supported by the browser.
- **Draft:** restore edits stored in local browser storage when available and accepted by the user.

Remote PNG and MP3 references default to:

`https://transformiceadventures.com/tfmadv/`

For example, a part may use `equipement/2/head_extra.png`, and a sound may use `sons/soins3.mp3`. Imported local resources override matching remote resources. The editor does not include a duplicate `tfmadv/` tree in its distribution. A remote resource can still be missing, and cross-origin restrictions depend on the game server's response headers and on where the editor is hosted.

## Saving and exporting

- **Export data.bin** serializes the supported edited records to a new binary file for download.
- Export functions for an animation's data or a supported JSON representation are available in the relevant editor controls.
- **Export sprites/sounds ZIP** packages imported or replaced media files using their expected game-relative paths so they can be deployed alongside the edited BIN.
- The original file and remote game installation are not automatically overwritten.

The editor validates input formats and the binary representation before accepting supported mutations. Invalid PNG/MP3 files and incompatible BIN files should be rejected without replacing the current project. An **Issues** panel records encountered errors in the current session and can export diagnostic JSON. Undo/Redo and local drafts provide additional recovery mechanisms, but you should keep backups of important BINs and artwork.

## Keyboard and mouse reference

| Control | Action |
|---|---|
| `Space` | Play / Pause |
| `Left` / `Right` | Previous / next frame |
| `K` | Add keyframe |
| `Delete` | Delete the selected keyframe |
| `P` | Toggle Move / Pivot |
| `G` | Toggle grid |
| `Ctrl/Cmd + Z` / `Ctrl/Cmd + Y` | Undo / Redo |
| `Ctrl/Cmd + O` | Open BIN |
| `Ctrl/Cmd + S` | Export BIN |
| Right mouse drag on preview | Pan the camera without editing |
| Mouse wheel on preview | Zoom the canvas |
| Drag timeline ruler | Scrub the current frame |
| Drag timeline keyframe diamond | Move the keyframe horizontally |

Shortcuts are ignored in input fields where their normal text-editing behavior is needed.

## Running the application

### Portable static distribution

Extract the **portable ZIP**, keeping its `index.html`, `studio.css`, `vendor/`, `favicon.svg` and any other supplied files together. Open `index.html` in a supported browser, or publish those files on a static HTTP host such as GitHub Pages. The portable edition uses JavaScript compiled from TypeScript and bundled into the HTML; it does not need Node.js or a separate application back-end to serve its own files.

Some browsers restrict `file://` resources, remote fetches, folder selection and direct-save APIs. Static HTTP hosting is generally more predictable, particularly when loading the official BIN or external assets. The browser also needs an internet connection to fetch the game-hosted resources.

### Developer environment and Vite

Node.js 20+ is recommended for the editable source project:

```bash
npm install
npm run dev
npm run typecheck
npm run build
```

`npm run build` performs a TypeScript typecheck and a Vite production build in `dist/`. Vite is configured with `base: './'` for subdirectory hosting. Production hosting serves **static output only**; a Node.js server is not required in production.

There is also an offline-style portable builder when a TypeScript compiler is already available:

```bash
npm run build:offline
```

This runs TypeScript compilation followed by `scripts/portable.py`. The portable builder is **not** the same operation as a Vite production build. Both source and portable distributions use the same `studio.css` source file.

### Static site deployment

Upload the **contents** of the portable distribution into the intended directory, e.g. `/data-editor/`, rather than uploading the ZIP file itself. The deployment needs one HTML entrypoint and **one** stylesheet named `studio.css`; it does not require any `studio-v*.css` or `styles.css` files. The HTML reference may include a query-string revision to refresh browser/CDN caches while the filename remains stable.

Do not place your private working BIN or custom sprites on a publicly readable website unless you intend to publish them. When hosted on a different origin from the game, remote BIN retrieval may require cross-origin access allowed by the game server.

## Source code structure

```text
src/
  codec.ts          Binary parsing, validation and serialization
  main.ts           Application state, UI, preview, asset handling and export
  keyframes.ts      Timeline/keyframe movement logic
  poses.ts          Pose/preset and nested-reference operations
public/
  studio.css        Single complete stylesheet
  favicon.svg       Application icon
  vendor/           Browser distributions of required runtime libraries
  sample/data.bin   Sample fixture for optional local tests
scripts/
  portable.py       Offline-style static launcher packaging
  package-release.py  Source and portable ZIP packaging
tests/              Binary and browser interaction regression tests
index.html          Vite source HTML entrypoint
vite.config.ts      Vite configuration for relative-path deployment
```

PixiJS handles scene preview, Howler.js handles sound, the TypeScript application maintains edit state, and the binary codec handles reading/writing the asset file. The editor's UI categories and workspace preferences are not serialized as new fields in the game BIN.

## Testing and limitations

Use these commands to run the included test suites, provided their dependencies (including a Chromium/Playwright environment for browser tests) are available:

```bash
npm run test:codec
npm run test:keyframes
node tests/pose-presets-test.mjs
npm run test:layout
npm run test:browser-keyframes
```

Important limitations to consider when using the editor:

- It cannot introduce a different in-game FPS without modifying the game runtime; animations remain based on 60 FPS.
- A custom `#` part may need a corresponding PNG and Look slot for **each** target appearance, as well as support from the actual game client.
- `data.bin` does not bundle external artwork or audio; exporting a BIN alone cannot deploy new PNG/MP3 assets.
- Browser preview is not a guarantee of pixel-perfect behavior for all game-side effects and render states.
- Remote assets and the official BIN depend on server availability, correct resource URLs and applicable browser CORS policy.
- Binary validation covers the recognized format; compatibility with future, undocumented game-format changes is not guaranteed.
- The editor does not implement privileged live writes to the game's server. Publishing the exported files to a game installation or content server is a separate operation.

Keep independent backups when working on valuable production data and test newly added parts and poses in the intended game client before release.
