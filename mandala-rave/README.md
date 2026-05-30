# mandala-rave

> Audio-reactive radial sigil — a cyberpunk / EDM rave visual instrument built in a single HTML file.

**Live:** https://shinyyim.github.io/mandala-rave/

A 12-fold mandala drawn as closed Lissajous curves, surrounded by an orbital HUD (rotating tick rings, curved text labels, satellites), reacting to live microphone audio with multi-pass neon glow, chromatic aberration on beats, and a choreographed glitch cycle.

---

## Features

- **Live audio reactivity** via Web Audio API — bass / mid / treble bands + beat detection
- **Choreographed glitch cycle** — 8-step pattern (shockwave / rings / radar / strobe) advancing on each detected bass beat
- **Gradual HSL color cycle** — slow ~3 min sweep through neon pink → magenta → violet → blue → cyan → green
- **Concentric neon background** — 5-stop radial gradient (black core → vivid edges)
- **Orbital detail layer** — 6 evenly-spaced rings (40px apart) with tick marks, rotating curved text, diamond markers, satellite dots, and orbit points
- **Side LED meters** — segmented vertical bass/treble gauges on both edges, color synced to mandala hue
- **Mouse fallback** — when no mic, mouse X/Y synthesizes fake bass/mid/treble so the visual still reacts
- **Single file, no build** — open `index.html` and it runs

---

## Quick Start

### Run locally

```bash
git clone https://github.com/shinyyim/mandala-rave.git
cd mandala-rave
open index.html
```

That's it. No build, no dependencies. Works in any modern browser.

> **Microphone note:** browsers block `getUserMedia` on `file://` URLs in some configurations. If the mic doesn't activate locally, either use the [live URL](https://shinyyim.github.io/mandala-rave/) or run a local HTTP server (`python3 -m http.server`).

### View live

Visit https://shinyyim.github.io/mandala-rave/ in any modern browser. Click the overlay to grant microphone access, then play music nearby.

---

## Controls

| Key / Input | Action |
|---|---|
| Click overlay | Enable microphone (live audio reactivity) |
| `Esc` | Skip mic, use mouse mode |
| `M` | Re-open mic overlay |
| `Space` | Reseed all mandala curves (new shape) |
| `C` | Clear canvas |
| `+` / `-` | Change mandala fold count (3–24) |
| `A` | Toggle auto-evolve (slow parameter drift) |
| Mouse X | Drift (rotation phase speed) |
| Mouse Y | Energy (overall intensity) |

---

## How the Audio Reactivity Works

Every frame, audio is sampled via FFT (2048-bin) and split into three bands:

- **bass** (20–230 Hz, bins 1–10) — kick drums, sub bass
- **mid** (230 Hz–2 kHz, bins 10–90) — vocals, snare, leads
- **treble** (2–8 kHz, bins 90–350) — hi-hats, cymbals, brightness

Beat detection runs on the bass envelope (EMA × 1.35 threshold). Each detected beat advances the choreographed glitch cycle by one step.

The audio levels drive:

| Audio | Visual effect |
|---|---|
| bass | Mandala radius pulse, beat detection, BG ring lightness |
| beatPulse | Choreographed glitch fires, chromatic aberration, line thickening |
| mid | Phase progression, angular range, petal bloom |
| treble | Outer radial expansion, sparkle frequency, BG color shift toward cyan |

Without a microphone, mouse position synthesizes these values (`mouseY → bass`, `mouseX → mid`, `1 - mouseY → treble`) so the visual still responds.

---

## Project Structure

```
mandala-rave/
├── index.html          # Everything: HTML + CSS + JS in one file (~860 lines)
├── README.md           # This file
├── ARCHITECTURE.md     # Detailed architecture and code map
├── LICENSE             # MIT
└── .gitignore
```

Everything is in `index.html`. CSS is inline in the `<head>`, JavaScript is inline at the bottom. No external dependencies except Google Fonts (JetBrains Mono).

---

## Development Workflow

```bash
# 1. Edit
$EDITOR index.html

# 2. Reload (just refresh the browser — no build step)
# 3. Commit + push when ready
git add index.html
git commit -m "your change"
git push
```

GitHub Pages automatically rebuilds on push (~1–2 min). Hard refresh (`Cmd+Shift+R`) the live site if cached.

---

## Quick Customization

Most common tweaks and where to find them in `index.html`:

| What | Where (approximate line) | Notes |
|---|---|---|
| Mandala symmetry (fold count) | `let FOLD = 12;` (~ line 230) | Or use `+`/`-` keys at runtime |
| Curve count / density | `const N = 7;` in `// CURVES` section | Higher = denser mandala |
| Color cycle range | `baseHue = 120 + oscT * 200;` in `tick()` | 120° (green) ↔ 320° (pink) |
| BG palette | `pinkHue = 312 + bgOsc * 28;` in `tick()` | Cyberpunk pink range |
| Glitch sequence | `PATTERN_CYCLE` array in glitch section | 8-step choreography |
| Ring radii (orbital detail) | `R_TICK`, `R_TEXT_IN`, etc. in `renderOrbits()` | 480, 510, 530, 550, 570, 600 |
| Outer text content | `farLabel = '…'` in `renderOrbits()` | Curved text on outer orbit |
| Inner text content | `label = '…'` in `renderOrbits()` | Curved text on inner orbit |
| Beat sensitivity | `bass > bassEMA * 1.35` in `analyzeAudio()` | Lower = more sensitive |

For deeper architecture, see [ARCHITECTURE.md](./ARCHITECTURE.md).

---

## Tech Stack

- **Vanilla JavaScript** (ES2020+) — no frameworks
- **Canvas 2D API** — all rendering (no WebGL/Three.js)
- **Web Audio API** — `AudioContext`, `AnalyserNode`, `getUserMedia`
- **Plain HTML + CSS** — single-file, inline
- **GitHub Pages** — static hosting with auto HTTPS

No build tooling, no bundler, no npm.

---

## Browser Support

- Modern Chrome, Edge, Firefox, Safari (last 2 major versions)
- Microphone input requires HTTPS or `localhost` (browsers block `getUserMedia` on plain HTTP)
- Best at 1920×1080 — canvas auto-scales to fit smaller windows

---

## Credits

- Curved text content references *Neon Genesis Evangelion* (SEELE / Instrumentality philosophy)
- Built with [Claude Code](https://claude.com/claude-code)

---

## License

MIT — see [LICENSE](./LICENSE).
