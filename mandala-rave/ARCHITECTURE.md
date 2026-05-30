# Architecture

Single-file architecture (~860 lines) covering everything from audio FFT to canvas rendering. This document maps that file and explains how the systems fit together.

---

## File Layout

```
index.html
├── <head>
│   ├── Fonts (Google Fonts: JetBrains Mono)
│   └── <style>   — all CSS inline (~190 lines)
│       ├── Stage scaling
│       ├── HUD positioning
│       ├── Mic-enable overlay
│       ├── Side LED meters (segmented bars)
│       └── CRT scanlines + vignette
├── <body>
│   ├── #stage (the 1920×1080 stage, transform-scaled to viewport)
│   │   ├── #bg (faint chamber glow)
│   │   ├── #c (the main <canvas>)
│   │   ├── #scanlines / #vignette overlays
│   │   ├── HUD divs (fold counter, status, controls)
│   │   └── .meter divs (left = bass, right = treble)
│   ├── #mic-overlay (click-to-enable)
│   └── <script> — all JS inline (~660 lines)
└── </body>
```

---

## Render Pipeline (per frame, 60 fps)

```
requestAnimationFrame → tick(now)
│
├── 1. analyzeAudio()
│      AnalyserNode.getByteFrequencyData() → bass / mid / treble / rms
│      Beat detection: bass > EMA × 1.35 → beatPulse = 1, decays × 0.82
│
├── 2. maybeTriggerGlitch(now)
│      Rising edge of beatPulse → advance PATTERN_CYCLE one step
│      Spawn the next glitch in the choreography
│
├── 3. Mouse mode fallback
│      If !audioReady: synthesize bass / mid / treble from mouse position
│
├── 4. Background gradient
│      6-stop radial gradient (HSL), black core → vivid neon edges
│      Lightness boosted by bass + beatPulse
│      Black veil (alpha 0.20) for atmospheric tone
│
├── 5. Mandala curves (N=7 closed Lissajous curves)
│      For each curve:
│        Sample STEPS=220 points in the wedge
│        sharpen() turns smooth sin into star-point peaks
│        Draw FOLD rotations × 2 (primary + mirror)
│        3-pass stroke: outer halo + mid glow + crisp core (lighter blend)
│        Chromatic aberration: red + cyan offset strokes on beat
│
├── 6. Laser spokes
│      FOLD radial beams from center, fired when beamE > 0.18
│      Outer halo + crisp white core
│
├── 7. renderOrbits()  (the orbital HUD layer)
│      0a. Spokes (30 → 800, FOLD count)
│      0b. Tick marks at 740 + faint guide ring
│      0c. Outer solid ring at 800
│      0d. Rhythmic particles oscillating between ring pairs (cosine ease)
│      0e. Orbit points on rings 740 + 800 (counter-rotating)
│       1. Tick ring at 480 (60 ticks, every 5th longer)
│       2. Inner curved text at 510 (Eva quotes)
│       3. Diamond markers at 530 (8 cardinal+intercardinal)
│       4. Satellite ring 550 (3 dots, beat-reactive counter-rotation)
│       5. Satellite ring 570 (5 dots, beat-reactive forward rotation)
│       6. Outer curved text at 600 (Eva quotes)
│
├── 8. renderGlitch()
│      For each active glitch in the queue:
│        Draw + decay (alpha-based lifetime)
│        Active types: shockwave / rings / radar / strobe
│
└── 9. HUD update
       Side LED meters: smoothed bass / treble fill the segmented bars
       CSS var --neon updated to sync meter color with mandala hue
```

---

## Code Map (line ranges)

| Lines | Section | What |
|---|---|---|
| 1–195 | `<head>` / `<style>` | All CSS — stage, HUD, meters, overlays |
| 196–207 | `<body>` markup | Stage, canvas, HUD, mic overlay |
| 208–227 | Stage scaling | Fit 1920×1080 to viewport |
| 228–250 | State + utilities | `FOLD`, `phase`, `hueT`, `hslToRgb()` |
| 251–280 | LED meters setup | Build 20-segment bars, smoothing |
| 281–335 | Audio pipeline | `enableMic()`, `analyzeAudio()`, beat detection |
| 336–355 | Glitch system header | `PATTERN_CYCLE`, state |
| 356–390 | `spawnGlitch()` | Parameters per glitch type |
| 391–405 | `maybeTriggerGlitch()` | Rising-edge detection + cycle advance |
| 406–482 | `drawOneGlitch()` | Render shockwave / rings / radar / strobe |
| 483–525 | Curve system | `seed()`, `sharpen()`, `STEPS`, `N`, `parts[]` |
| 526–547 | Input handlers | Pointer, keyboard, mic toggle |
| 548–752 | `renderOrbits()` | All 6 orbital ring layers + spokes + dots |
| 753–990 | `tick()` (main loop) | The full pipeline above |
| 991–1003 | Idle cursor hide | Hide cursor after 2.5s inactivity |

---

## Major Systems

### Audio Pipeline

```
getUserMedia({audio: true})
       ↓
MediaStreamSource → AnalyserNode (fftSize 2048, smoothing 0.55)
       ↓ getByteFrequencyData() each frame
freqData[1024]
       ↓ band averaging
{ bass, mid, treble, rmsLv }
       ↓ envelope detection
bassEMA (exponential moving average)
       ↓ threshold
beatPulse (1.0 spike, decays × 0.82)
```

**Beat detection** is intentionally simple: when `bass` exceeds its smoothed envelope by 35% AND is above 0.18 absolute, fire `beatPulse = 1`. Each frame it decays by × 0.82. Rising edge of beatPulse (crossing from < 0.55 to > 0.65) triggers the choreographed glitch.

**Mouse fallback** (when no mic): `bass = mouseY × 0.7`, `mid = mouseX × 0.7`, `treble = (1 − mouseY) × 0.5`. Random fake beats fire at rate `mouseY × 0.008` per frame.

### Mandala Curve Generation

Each particle is a **closed Lissajous-like curve** in polar coordinates inside one wedge (slice of the FOLD-fold symmetry):

```js
// for s in 0..STEPS:
const t = (s / STEPS) * 2π;
const aRaw = sin(t * aFreq + aPhase + phaseSkew);
const rRaw = sin(t * rFreq + rPhase);
const localA = (0.5 + 0.5 * sharpen(aRaw, sharpA)) * slice * angleMod;
const r      = (rBase + sharpen(rRaw, sharpR) * rAmp) * radiusMod;
const x = cos(localA - slice/2 + π/2) * r;
const y = sin(localA - slice/2 + π/2) * r;
```

- `aFreq` / `rFreq` are **integers (1–5)** → the curve closes cleanly (no chaotic open path)
- `sharpen(x, k) = sign(x) × |x|^k` with `k < 1` turns smooth sinusoidal peaks into star points / spikes
- The wedge-local curve is then drawn `FOLD` times (rotated by `slice` each time) plus a mirror copy

Three stroke passes (`globalCompositeOperation = 'lighter'`):
1. **Outer halo** — 5.5× width, 10% alpha → soft glow
2. **Mid glow** — 2.4× width, 30% alpha → bloom
3. **Crisp core** — 1× width, full alpha, color pushed toward white

On beat (`beatPulse > 0.30`), two extra passes draw the curve with **red (+x offset) and cyan (-x offset)** for chromatic aberration.

### Color System

Two independent hue cycles:

**Mandala hue** — restricted to "cool / neon" range (no yellow / red):
```js
hueT += 0.00009;
const oscT = (sin(hueT * 2π) + 1) * 0.5;
const baseHue = 120 + oscT * 200;  // 120° (green) ↔ 320° (pink)
```
Period ~3 min. Audio shifts mainHue by ±14° via `bass` and `treble`.

**Background hue** — restricted to "warm cyberpunk" range:
```js
const bgOsc   = (sin(hueT * 2π * 0.5) + 1) * 0.5;
const pinkHue = 312 + bgOsc * 28;  // 312° ↔ 340° (magenta → hot pink)
```
Period ~6 min. The BG gradient uses `pinkHue` and `violetHue = 270 + bgOsc * 18` for opposing-end stops.

HSL→RGB conversion is implemented inline (no library):
```js
hslToRgb(h, s, l) → [r, g, b]
```

### Glitch Choreography

A fixed 8-step pattern cycles through these glitch types, one per detected beat:

```
1. shockwave   2. rings       3. shockwave   4. radar
5. shockwave   6. rings       7. radar       8. strobe
```

**Trigger logic** (in `maybeTriggerGlitch`):
- Rising edge: `beatPulse > 0.65 && prevBeatPulse < 0.55`
- Minimum interval: 90 ms between fires
- On fire, `spawnGlitch(PATTERN_CYCLE[beatIdx % length], beatPulse)`

Active glitches live in `glitches[]`. Each has its own decay rate; when `t <= 0`, removed.

**Active types:**
- `shockwave` — 3–5 concentric rings exploding from center
- `rings` — slower concentric rings
- `radar` — single rotating beam, ~70 frames
- `strobe` — full white flash, ~4 frames

### Orbital Detail Layer

Six evenly-spaced rings (at radii 480 / 510 / 530 / 550 / 570 / 600), plus an outer "frame ring" at 740 / 800:

```
       ↓ closer to mandala
[480] tick marks (60 ticks, every 5th longer)
[510] inner curved text (Eva quotes, rotates +3.6°/sec)
[530] 8 diamond markers (cardinal + intercardinal)
[550] satellite ring — 3 dots, beat-reactive counter-rotation
[570] satellite ring — 5 dots, beat-reactive forward rotation
[600] outer curved text (Eva quotes, rotates −2.4°/sec)
       ↓ outer frame
[740] tick ring + faint guide circle + 1 orbit dot (counter-rotating)
[800] solid ring + 1 orbit dot (forward-rotating)
       ↑ farther from center
```

**Radial spokes**: `FOLD` rays from `r=30` to `r=800`, very slow rotation.

**Rhythmic particles**: one per spoke, oscillating between a randomly-assigned pair of rings (e.g., [120, 800] or [480, 740]). Position uses cosine ease which naturally pauses at the extremes (the rings).

```js
const [rIn, rOut] = RING_PAIRS[i % RING_PAIRS.length];
const k = (cos(t) + 1) * 0.5;     // 0..1, slow at extremes
const r = rIn + (rOut - rIn) * k;
```

### Background Gradient

Six-stop radial gradient drawn fresh every frame:

```
[0.00]  black core               (lightness 0)
[0.42]  soft violet emerging     (lightness 5  + audio)
[0.72]  magenta band             (lightness 24 + audio × 0.65)
[0.95]  hot pink peak            (lightness 52 + audio × 0.75)  ← brightest
[1.00]  edge violet ease         (lightness 44 + audio × 0.55)
```

`audioBoost = bass × 16 + beatPulse × 26 + rmsLv × 4` makes the outer pink ring strongly pulse on every kick.

A `rgba(0, 0, 0, 0.20)` veil is painted on top so the colors feel atmospheric rather than opaque.

### Side LED Meters

Two segmented vertical bar meters on the left (bass) and right (treble) edges of the stage:

- 20 segments each, stacked bottom-up via `flex-direction: column-reverse`
- Each segment is a CSS `.seg` div that lights up (`.seg.on`) based on audio level
- The lit color uses CSS variable `--neon` which JavaScript updates each frame to match the mandala's current hue

```js
document.documentElement.style.setProperty('--neon', `${colR}, ${colG}, ${colB}`);
```

---

## Key State Variables

| Variable | Type | Purpose |
|---|---|---|
| `FOLD` | `let` | Mandala rotational symmetry (default 12, range 3–24) |
| `phase` | `let` | Time variable, increments with mid + treble + drift |
| `hueT` | `let` | Slow time variable for hue cycles |
| `bass`, `mid`, `treble` | `let` | Current audio band levels (0..1) |
| `beatPulse` | `let` | Beat envelope (0..1, spikes to 1 on beat) |
| `bassEMA` | `let` | Smoothed bass for beat detection |
| `energy`, `drift` | `let` | Smoothed control inputs (audio or mouse) |
| `parts[]` | `const` | Mandala curves (N=7 elements) |
| `glitches[]` | `const` | Active glitch elements |
| `beatIdx` | `let` | Glitch cycle position |
| `audioReady` | `let` | True when mic is connected |
| `satAng1`, `satAng2` | `let` | Beat-reactive satellite rotation angles |

---

## Audio Reactivity Matrix

| Audio source | Affects |
|---|---|
| `bass` | Mandala radius, `radiusMod`; rhythmic particle phase nudge; BG outer ring lightness; satellite rotation speed |
| `beatPulse` | Glitch trigger; chromatic aberration; line thickness ×1.5; BG lightness; satellite rotation; brightness flash on multiple elements |
| `mid` | `phase` acceleration; angular range; petal bloom; phase skew |
| `treble` | Outer radial expansion; sparkle frequency; hue shift toward cyan; laser beam intensity; drift |
| `rmsLv` | Overall energy boost across multiple parameters |

---

## Performance Notes

- **N=7 curves × FOLD=12 × 2 mirrors × 3 stroke passes** ≈ 504 strokes/frame for the mandala alone
- Plus orbital detail (~150 strokes/frame) and glitches (variable)
- Roughly 700–1500 stroke operations per frame at 60 fps — well within Canvas 2D budget on modern hardware
- Most expensive single op: long curved text rendering (each character is `save / translate / rotate / fillText / restore`)
- No off-screen canvas, no `getImageData`, no shadow blur — kept simple

---

## Extending

### Adding a new glitch type

1. Add the name to `PATTERN_CYCLE` array
2. Add a case in `spawnGlitch()` setting `g.params` and `g.decay`
3. Add a case in `drawOneGlitch()` rendering the glitch
4. Test by triggering manually: in console run `spawnGlitch('your_type', 1)`

### Changing the color palette

- **Mandala hue range** — edit `baseHue = 120 + oscT * 200` (start hue + range)
- **BG palette** — edit `pinkHue = 312 + bgOsc * 28` (and stops below)
- **Side meter color** — automatic, follows mandala hue via `--neon` CSS var

### Adding a new orbital ring

In `renderOrbits()`, add a new layer:
1. Define a new `R_*` constant for the radius
2. Draw your element (ring / ticks / text / dots) at that radius
3. Pick a rotation speed multiplier of `baseRot` for parallax variety

### Wiring an audio file instead of mic

Replace `MediaStreamSource` source in `enableMic()`:
```js
const audio = new Audio('your-track.mp3');
audio.crossOrigin = 'anonymous';
audio.play();
const src = ac.createMediaElementSource(audio);
src.connect(ac.destination);   // hear it
src.connect(analyser);         // analyze it
```
