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

3. ~~**`document.body` `touchmove` is `preventDefault`ed unconditionally**, so the overlay's `<code>` block cannot be scrolled on a narrow phone.~~ **Fixed** — scoped to `#ascii-wrap` in all three demos.

4. **~200 lines of CSS and the whole projection loop are duplicated four ways.** Every fix above had to be applied 3–4×. An `ascii.css` + `ascii.js` pair would end that — but it turns single-file demos into a build-ish thing, which is a call about what these files are *for*.

5. **`trefoil.html`'s tube frame degenerates.** `nx = -ty, ny = tx, nz = 0` is undefined when the tangent is near-vertical; the frame snaps and the tube shows a twist seam. **Fixed in `play.html`** with a parallel-transport frame (Gram-Schmidt the previous normal against each new tangent). A circular cross-section is rotationally symmetric, so the frame failing to close around the loop costs nothing. `trefoil.html` and `hold.html` still carry the old frame — porting it is mechanical and should happen next.

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


---

# Second pass — objects, void, and a code review of `play.html`

Reviewed the playtool as shipped in the previous pass. Findings below were
verified against a headless Chromium run over all objects and a hostile URL
hash; everything listed as fixed is in this branch.

## Fixed — correctness

| # | Finding | Fix |
|---|---------|-----|
| 15 | **Two `requestAnimationFrame` chains after a tab switch.** `visibilitychange` scheduled a fresh `loop` without cancelling the pending frame. The old callback is still queued — it just can't fire while hidden — so coming back ran two interleaved chains, doubling render work and halving the effective `dt`. Present in **all four** pages. | A single `rafId`, cancelled on hide, guarded on `start()`. `loop` clears it on entry. Verified with a triple `visibilitychange` round trip. |
| 16 | **The URL hash was unvalidated input.** `#density=999` rebuilt millions of points and hung the tab before the first frame; `#dist=0` put the camera inside the object; `#ramp=99` indexed past `RAMPS` and threw on every frame. | A `LIMITS` table clamps every numeric key to its slider's own range, and `ramp` is rounded to an integer index. Tested with `#density=999&dist=0&ramp=99&vreach=99999&obj=nope&rotX=99`. |
| 17 | **Lifting one finger of a pinch snapped the object.** `pointerup` dropped from two pointers to one but left `startX`/`startRotX` seeded from the *original* touch-down, so the next `pointermove` applied the whole accumulated delta at once. | Re-seed the drag origin from the pointer that is still down. |
| 18 | **`rotY` grows without bound** while spinning, so a link shared after a few minutes carried `rotY=847.221`. | Wrapped to ±π on the way into the hash. |
| 19 | **The auto-quality toast repeated every 1.5s** for as long as the device stayed slow — the one thing a slow device does not need. | Said once per slow patch, re-armed when quality recovers. |
| 20 | `keydown` called `e.target.matches(...)` unguarded; `e.target` is not always an `Element`. | `instanceof Element` guard, and `[role=radio]` added to the ignore list so the chips own their own arrow keys. |
| 25 | **Dragging to rotate selected the ASCII.** `touch-action: none` covered touch; nothing covered a mouse. Every drag highlighted the frame blue, and a drag that ended outside the wrap left it highlighted. Found by screenshotting a scripted drag. | `user-select: none` on both layers. `Copy frame` reads `textContent`, so it is unaffected. |

## Fixed — UI / UX

| # | Finding | Fix |
|---|---------|-----|
| 21 | **Object chips were a `radiogroup` in name only** — every chip was a tab stop and arrow keys did nothing, which is the opposite of what a radiogroup promises a keyboard user. | Roving `tabindex` (one stop for the group) and arrow-key navigation that moves the selection. |
| 22 | **The control panel took 430px of an 860px viewport**, leaving the art 20 character rows. | Capped at `min(40dvh, 340px)` and the rows tightened; the stage went 20 → 25 rows at 1280×860. |
| 23 | **Below ~470px the active nav link scrolled out of sight** behind the wordmark and the fps badge. | Wordmark text hides under 470px; the dot stays. |
| 24 | Eight sliders in one undifferentiated grid. | Grouped into **Form · Lens & depth · Light · Void** sections with labelled headings. |

## New

- **Six more objects** — cylinder, cone, pyramid, capsule, octahedron,
  icosahedron — bringing it to 12, grouped as Solids · Rings · Things.
  Polyhedra share one `mesh(verts, faces, R)` sampler; curved solids share a
  `ringStep` helper that holds arc spacing constant as the radius shrinks.
- **Depth fog** — luminance attenuated by depth within the object's own
  bounding range, normalised so it reads the same at any K2 or zoom.
- **The void** — a background point cloud the camera sits inside, projected
  through the same camera so it parallaxes against the object. Deterministic
  (a shared link reproduces the field), drawn into a second `<pre>` that shares
  the object's grid cell, and masked by the object's own z-buffer so it never
  bleeds through. Its point count is solved from the frustum's solid angle
  every frame, holding coverage near 3.5% of cells — measured stable across a
  123×25 desktop grid and a 69×69 phone grid.

## Still open

1. Items 1, 2, 4, 6, 7 from the first pass stand unchanged.
2. **The parallel-transport frame (item 5) is fixed in `play.html` only.**
   `trefoil.html` and `hold.html` still snap. Mechanical port, not done here.
3. **`VOID_MAX` (24000) saturates past ~2.5× zoom**, where the frustum narrows
   enough that the solved count would exceed the cloud. Coverage thins instead
   of holding. Raising the cap trades memory for a case few people reach.
4. **The void ignores `prefers-reduced-motion` for drift only** — drift starts
   at 0 under reduced motion, but nothing stops a user raising it. That is
   probably correct (explicit input beats a media query), but it is a choice.
5. **Duplication is now worse, not better.** `play.html` has the parallel
   transport frame, the flat `Float32Array` geometry, fog, and the void; the
   other three have none of it. First-pass item 4 is the blocker — until there
   is an `ascii.js`, every improvement widens the gap.
