# quintessa / stage

> Audio-reactive, camera-driven interactive visuals — four self-contained scenes wired up behind a single stage entry, switchable on number keys.

A media-interactive playground that combines live face tracking, audio-reactive geometry, particle systems, and chromatic glitch over the browser's webcam + mic. Each scene is a single HTML file; `stage.html` loads all four in iframes and lets you switch between them with `1`–`4`.

---

## Scenes

| Key | Scene | What it is |
|---|---|---|
| `1` | [faces](./faces.README.md) | Live face tracking — grayscale crops inside red neon boxes with random labels, audio-reactive pixelized background, RGB glitch on beats |
| `2` | [mandala](./mandala-rave/README.md) | 12-fold audio-reactive radial sigil with multi-pass neon glow and choreographed glitch cycle |
| `3` | [cosmosbeater](./cosmosbeater.README.md) | WebGL2 point-cloud / sphere / chaos with image-texture cycling, all driven by audio analyser |
| `4` | [boundary](./boundary.README.md) | Sparse line / particle field that surges with bass beats |

---

## Run locally

This project uses `getUserMedia` (camera + mic), which browsers block on `file://`. Serve over HTTP:

```bash
cd /path/to/this/folder
python3 -m http.server 8765
# then open http://localhost:8765/stage.html
```

Then click **ENTER** on the boot screen and the camera/mic permission prompt comes up once (it's shared across all four iframes since they're same-origin).

---

## Stage controls

| Key | Action |
|---|---|
| `1` `2` `3` `4` | Switch to scene |
| `f` | Toggle fullscreen |
| `Esc` | Exit fullscreen (browser-native) |

Each scene has its own additional shortcuts — see the per-scene README.

---

## Deploy

### Recommended: Vercel (1-line)

```bash
npx vercel --prod
```

CLI uploads your local folder directly (so `.gitignored` files like the `zeo/` image library still ship). You'll get a `*.vercel.app` URL with HTTPS automatically.

### Alternative: Cloudflare Pages (unlimited bandwidth)

```bash
npm i -g wrangler
wrangler login
wrangler pages deploy . --project-name=quintessa
```

Best if you expect heavy traffic — bandwidth is unlimited on the free tier.

### Notes

- `getUserMedia` only works on HTTPS or `localhost`. Both Vercel and Cloudflare Pages give HTTPS automatically.
- The `zeo/gemini-generation/Generations/_all_images/` folder (~125MB of PNGs) is required only by **cosmosbeater**. It's gitignored — see [Repo layout](#repo-layout) below.

---

## Repo layout

```
.
├── stage.html                       # Entry — iframe shell + 1-4 switcher
├── faces.html                       # Scene 1
├── cosmosbeater.html                # Scene 3
├── boundary.html                    # Scene 4
├── mandala-rave/
│   ├── index.html                   # Scene 2
│   └── README.md
├── zeo/gemini-generation/Generations/_all_images/    # gitignored
│   ├── manifest.json                # required by cosmosbeater
│   └── *.png                        # ~99 source images
├── README.md                        # this file
├── faces.README.md
├── cosmosbeater.README.md
└── boundary.README.md
```

`.gitignore`:
```
zeo/
obsidian/
ref/
ref_frames/
.DS_Store
node_modules/
.vercel/
src/
```

Push to GitHub for code versioning; deploy to Vercel for the live site. Vercel uploads everything from your local folder regardless of `.gitignore`, so the large image library ships with the deploy without polluting your git history.

---

## Tech stack

- Vanilla JS / Canvas 2D / WebGL2 / Web Audio API
- [pico.js](https://github.com/tehnokv/picojs) for cheap face bbox detection (faces)
- No build step. No frameworks. Each scene is a single HTML file.
