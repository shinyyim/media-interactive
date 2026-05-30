# cosmosbeater

> WebGL2 cosmic point-cloud, chaos field, sphere, and image backdrop — all reacting to live audio. Cycles through a library of generated images as texture sources.

A live audio-reactive 3D scene built directly on WebGL2 (no Three.js). Renders simultaneously: a star-like sphere of points, concentric rings, line pyramids, plus optional **chaos** (image-driven particle field), **backdrop** (image as bg plane), **pointcloud** (per-pixel sampled cloud), and **texsphere** (image mapped onto the sphere). Each of those four modes layers on top.

The image library is fetched from `manifest.json` and loaded on demand from `zeo/gemini-generation/Generations/_all_images/`.

---

## Controls

| Key / UI | Action |
|---|---|
| **ENTER** (gate) | Start scene |
| `g` / MIC | Toggle live mic (otherwise mouse drives audio bands) |
| `x` / CHAOS | Toggle chaos field (image-driven particles) |
| `y` / BACK | Toggle image backdrop (textured background plane) |
| `i` / CLOUD | Toggle pointcloud (per-pixel point sampling of current image) |
| `t` / TEXSPHERE | Toggle texsphere (image mapped onto the central sphere) |
| `j` / PREV, `k` / NEXT | Cycle to prev / next image |
| `←` / `→` | Spin bias (camera orbit speed) |
| `z` | Zoom in (camera closer) |
| `Space` | Pause / resume |
| `f` | Fullscreen toggle |
| `h` | Hide / show HUD |
| `Esc` | Exit fullscreen |

---

## What it does

- **Audio analyser** (FFT, 1024 bins) — splits into bass / mids / treble bands + beat detection. Drives brightness, glow, point size, ring color, line alpha.
- **Image library** — loads from `./zeo/gemini-generation/Generations/_all_images/manifest.json` (an array of filenames). First N images load eagerly, rest lazily in the background. Each becomes a WebGL texture + optional pointcloud (built on demand).
- **Render passes**:
  1. **Chaos** (optional) — image displayed as warped particles, modulated by audio
  2. **Backdrop** (optional) — image as billboard behind the scene
  3. **Pointcloud** (optional) — per-pixel sampled cloud floats in 3D
  4. **Sphere of points** — always on, brightness pulses with bass
  5. **Texsphere** (optional) — current image mapped onto the sphere, lit by bass
  6. **Rings + pyramids** — concentric line geometry, alpha follows mids
  7. **Post-process** — final composite to screen with tone curve

Each mode can be toggled independently, so you can run pointcloud + texsphere + rings + chaos all at once for maximum density.

---

## Files / dependencies

- `cosmosbeater.html` — single file, all GLSL inline
- **Required external data**: `./zeo/gemini-generation/Generations/_all_images/manifest.json` + the PNG library it references
- No CDN libs (Google Fonts only)

The image library is ~125 MB — gitignored by default. For deployment, Vercel and Cloudflare Pages both upload the local folder regardless of gitignore, so the library ships with the deploy.

---

## Tunable spots in `cosmosbeater.html`

| What | Where (search for) | Notes |
|---|---|---|
| Image manifest path | `imageManifestUrl:` | Point at any folder with a `manifest.json` |
| Eagerly-loaded count | `initialLoadCount:` | How many images to preload before lazy |
| Default modes on start | `modesOnStart:` | `backdrop: true` etc |
| Texsphere base brightness | `sphereBase = this.modeTexsphere ? 1.20 : 1.15` | Higher = more opaque |
| Sphere size | `sphereSize = this.modeTexsphere ? 9.0 : 4.0` | Larger = chunkier points |
| Camera distance start | `camDistTarget` initial value | Default zoom |
| Beat threshold | look for `bassKick` math | Beat sensitivity |
