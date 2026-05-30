# faces

> Real-time face tracking with neon-glow boxes, random labels, audio-reactive pixelized stipple background, neon grid, and RGB-split glitch.

A live webcam scene where every face the detector finds gets cropped in grayscale inside a colored neon box with a random adjective/occupation label (`KIND`, `LEATHERWORKER`, `EXTREME IRONER`...). The whole frame outside the boxes turns into a stipple of dots whose color tints reflect the dominant audio band (bass → crimson, mid → magenta, treble → cyan). A white neon grid drifts behind it. Beats punch RGB chromatic aberration inside the face boxes.

---

## Controls

| Key / UI | Action |
|---|---|
| **ENTER** (gate) | Request camera + mic, start scene |
| `p` / PIXEL | Toggle pixelized stipple background |
| `a` / AUDIO | Toggle audio analyser (mic in) |
| `g` / GLITCH | Toggle RGB chromatic-split + spark sparks |
| `k` / GRID | Toggle audio-reactive neon grid |
| `m` / MIRROR | Toggle selfie mirror |
| `r` | Re-roll all labels |
| `c` | Clear tracked faces |
| `[` / `]` | Decrease / increase pixel cell size |

Sliders on the bottom bar: **SIZE** (4-24 px cell), **DIM** (0-100% black overlay on stipple bg).

---

## What's happening under the hood

- **pico.js** runs a Haar-cascade-style face detector on a downsampled 440-wide grayscale frame every ~60 ms (~16 fps). Each detection becomes a tracked face with a stable ID + label.
- **Tracker** matches new detections to existing tracks by proximity (within face-size × 0.7). Tracks that disappear for >350 ms are dropped.
- **Each track** gets a single color sampled from a Rei moon-portrait palette (lavender, crimson, electric blue, hot rose, violet, icy cyan) — kept stable for the life of the track.
- **Background** is rendered as a black canvas, optionally with stipple dots sampled from the live video (dot color tinted by audio: weighted average of crimson/magenta/cyan).
- **Face boxes** clip the video into grayscale shapes. When glitch is on, the clip is filled with two additive SVG-filter passes (`feColorMatrix` → red-only / cyan-only) offset opposite directions — offset grows with bass beats.
- **Grid** is a flat orthogonal lattice scrolling at audio-driven speed, with random row/column highlights flashing on beats.

---

## Files / dependencies

- `faces.html` — single file
- [pico.js](https://github.com/tehnokv/picojs) loaded from `cdn.jsdelivr.net/gh/tehnokv/picojs/pico.js` (~10 KB)
- Face cascade fetched from `cdn.jsdelivr.net/gh/nenadmarkus/pico/.../facefinder` (~270 KB binary)
- Google Fonts: JetBrains Mono

No images or local assets required. Just needs HTTPS or `localhost` so the browser allows `getUserMedia`.

---

## Tunable spots in `faces.html`

| What | Where (search for) | Notes |
|---|---|---|
| Label word lists | `const ADJ`, `const NOUN` | Add / remove words |
| Neon palette | `NEON_PALETTE` | Per-face box colors |
| Spark color rules | `function sparkColor()` | Which band → which color |
| Stipple color blend | `function audioTintedColor()` | Eva palette weights |
| Box scale | `const BOX_SCALE = 0.9` | Visible box / clip size |
| Beat sensitivity | `bas > audio.basEMA + 1.0 * sigma` | Lower = more sensitive |
| Grid base cell | `const baseCell = 64` (in `drawGrid`) | Lattice density |
