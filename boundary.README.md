# boundary

> Minimal line / particle scene that surges to bass beats. Sparse, monochrome, contemplative.

A spare visual layer — long lines stretching across the canvas, accented by particle bursts that fire on bass beats. Lower-energy than the other scenes; reads like a screensaver / contemplative interlude.

---

## Controls

| Key / UI | Action |
|---|---|
| Click the gate | Begin (request mic) |
| `m` | Toggle mic on / off (mouse fallback if off) |
| `Space` | Manual beat trigger (test the burst) |
| `f` | Fullscreen toggle (or browser-native exit on `Esc`) |

---

## What it does

- **Audio analyser** measures mic input, derives bass band + beat events
- On each beat: `onBeat(intensity)` fires a particle burst and advances the line state
- The line geometry is laid out as sparse strokes across the canvas — width / opacity track audio energy
- Mouse position acts as a fallback driver when mic is off

The whole thing is built around `MIC` vs `MOUSE` as the source, switchable at runtime.

---

## Files / dependencies

- `boundary.html` — single file
- No local assets
- No CDN libs (Google Fonts only)

Self-contained. Just needs HTTPS / localhost for the mic.

---

## Tunable spots in `boundary.html`

Single file (~540 lines). Worth scanning end-to-end if you want to tweak — most numeric constants are at the top of the `<script>` block. Look for:

- `--mg` and other CSS color vars at the top of `<style>` — palette
- `onBeat(...)` — what happens on each beat
- `setLine(...)` — line update logic
- `loop()` — render loop
