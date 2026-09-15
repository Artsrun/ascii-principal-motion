# UI/UX review — ASCII · Principal Motion

Reviewed `index.html`, `trefoil.html`, `hold.html` at 1280×860 and 390×844.
Everything under **Fixed** is in this branch. Everything under **Open** is a real
finding I did not act on — with the reason.

---

## Fixed

### Blocking

| # | Finding | Fix |
|---|---------|-----|
| 1 | **Pinch-zoom was disabled** — `maximum-scale=1, user-scalable=no` on all three pages. Fails WCAG 1.4.4 (Resize Text); on 6px glyphs that is not a small thing. | Dropped both tokens from the viewport meta. |
| 2 | **Demos could not reach each other.** No link from any page to any other. `/trefoil.html` and `/hold.html` were reachable only by typing the URL. | Shared `<nav>` in every header, `aria-current="page"` on the active one. |
| 3 | **Two buttons lied.** `#to-points` ("Points preview") and `#to-donut` ("← back to donut") only rewrote a status string — no navigation. | Real `<a>` elements. "Points preview" now opens the playtool it was promising. |
| 4 | **`hold.html` framed every object at ~27% of the grid.** `K1 = COLS * 0.46` was tuned for the classic torus at `K2 = 5`; at `K2 = 6.5` with a smaller torus the object sat in the middle of a mostly empty screen. | `K1` is derived from the object's bounding radius, so each shape fills ~84% of the tighter grid axis. Torus ink went 258 → 720 cells at 1280×860. |

### Accessibility

| # | Finding | Fix |
|---|---------|-----|
| 5 | The `<pre>` mutates ~60×/s. Screen readers re-announce the whole ASCII buffer every frame. | `aria-hidden="true"` on the `<pre>`; the already-visible status line became the `role="status" aria-live="polite"` region. |
| 6 | No focus ring anywhere — `:focus-visible` was never styled, so keyboard users had no idea what was focused. | Shared `:focus-visible` outline. |
| 7 | `prefers-reduced-motion` ignored: auto-spin and the pulsing status dot ran regardless. | Global reduced-motion block; `index`/`trefoil` now boot paused when it is set. |
| 8 | `hold.html`'s object chips were four unrelated `<button>`s and the spin button had no state. | `role="radiogroup"` + `aria-checked`, and `aria-pressed` on the spin toggle. |

### Correctness / perf

| # | Finding | Fix |
|---|---------|-----|
| 9 | **Grid parity bug.** `Math.max(40, Math.min(120, Math.floor(w/cellW) \| 1))` — the `\| 1` forced an odd column count, then the clamp could hand back an even one, putting the projection centre on a cell boundary. | Clamp first, `\| 1` second. |
| 10 | `@import url(fonts.googleapis.com)` inside `<style>` — render-blocking, serialised behind the CSS parse, and a third-party request on every load. | System font stacks via `--mono`. Nothing leaves the origin now. |
| 11 | `requestAnimationFrame` kept running on a hidden tab. | Pause on `visibilitychange`, resume with a fresh timestamp. |
| 12 | `resize` fired in bursts during mobile URL-bar show/hide, reallocating both buffers each time. | Debounced to 120ms. |
| 13 | `localStorage` unguarded — throws in a private window or with site data blocked, killing the first-run overlay. | `try/catch`, overlay still shows. |
| 14 | No favicon → a 404 on every page load. | Inline SVG data-URI. |

---

## Open — not fixed, with reasons

1. **Double-tap fights single-tap** (`index`, `trefoil`). The first tap of a double-tap toggles pause *immediately*, so a reset is always preceded by a visible pause flicker. Correct fix is a ~300ms deferred single-tap, which adds latency to the primary gesture — that's a feel decision, not a bug fix. Worth trying both.

2. **Long-press is the only way to reopen the help overlay**, and nothing on screen says so. A visible `?` in the header would cost one button.

3. **`document.body` `touchmove` is `preventDefault`ed unconditionally**, so the overlay's `<code>` block cannot be scrolled on a narrow phone. Should be scoped to `#ascii-wrap`.

4. **~200 lines of CSS and the whole projection loop are duplicated four ways.** Every fix above had to be applied 3–4×. An `ascii.css` + `ascii.js` pair would end that — but it turns single-file demos into a build-ish thing, which is a call about what these files are *for*.

5. **`trefoil.html`'s tube frame degenerates.** `nx = -ty, ny = tx, nz = 0` is undefined when the tangent is near-vertical; the frame snaps and the tube shows a twist seam. A parallel-transport frame fixes it. Same code path exists in `hold.html`'s trefoil and in the playtool's `tube()` — I kept it consistent rather than fixing one of three.

6. **Dark-only.** A `prefers-color-scheme: light` visitor gets a black slab with no acknowledgement. Fine as an aesthetic choice, worth being deliberate about.

7. **`index`/`trefoil` still build geometry inline per frame** with an array-of-objects in `hold.html`. The playtool's flat `Float32Array` (stride 6) with hoisted trig is the faster shape — porting it back is mechanical but touches the code these demos exist to show.

8. **README says GitHub Pages deploys from `main`.** The default branch is `prod`. One of the two is wrong.

---

## New: `play.html` — the playtool

A single page that makes the pipeline's parameters *manipulable* instead of hard-coded, so the demo can answer "what does K2 actually do" by letting you drag it.

**Live controls**: object (torus · trefoil · sphere · mug · cube · spring) · density · zoom ·
distance (K2) · spin rate · light azimuth/elevation · charset ramp · Y-stretch · invert.

**Interaction**
- drag to turn, pinch **or** wheel to zoom, double-tap to reset
- full keyboard: `←→↑↓` turn (`shift` = coarse) · `space` spin · `R` reset · `C` copy · `S` share · `[` `]` density · `-` `+` zoom · `1`–`6` object · `H` panel

**Things the other three don't do**
- **Shareable state.** Every parameter lives in the URL hash. "Share link" copies a URL that reopens the exact frame — e.g. `play.html#obj=cube&ramp=2&rotX=0.5&rotY=0.9`.
- **Copy frame.** Puts the current ASCII on the clipboard as text.
- **Auto quality.** Tracks an EMA of fps; below 28 it thins the point cloud and says so, and creeps back up above 55. The demo stays interactive on a weak phone instead of dropping to 9fps.
- **Measured cell metrics.** Glyph width/height are measured from a probe element rather than assumed to be `1.55`, so circles are round in whatever monospace font the OS actually supplies.
- **Auto-framing.** `K1` comes from the object's bounding radius and the grid — every object fills the frame at every distance without per-object constants.
- **Font size from a target column count**, so the art stays legible at 390px and detailed at 1280px.

**Perf shape**: geometry is a flat `Float32Array` (stride 6, position + normal), rasterised into a `Uint8Array` char-index buffer and a `Float32Array` z-buffer, joined once per frame. No per-frame allocation, no per-point object.
