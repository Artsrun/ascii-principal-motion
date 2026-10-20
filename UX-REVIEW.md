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

## Still open (after the second pass)

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


---

# Third pass — ten more objects, seven shading models, ten charsets

## Objects: 12 → 22

New: **tetrahedron · dodecahedron · prism · gem** (Solids),
**knot · Möbius** (Rings), **gear · vase · bowl · hourglass** (Things).

The interesting part is what *went away*. The previous pass shipped
hand-written face tables — `[[0,11,5], [0,5,1], …]` for the icosahedron, and
I checked each triple's winding by hand against a cross product in a
scratch buffer. That does not scale to a dodecahedron, and a single
transposed pair is a silently inside-out face.

So faces are now derived. `hull(verts)` takes a bare list of points: for every
triple it tests whether all remaining vertices lie on one side of that plane,
keeps the plane if so and orients it outward, then gathers the coplanar
vertices per plane, sorts them by angle around the plane's centroid and
fan-triangulates. Every polyhedron is now a vertex list and nothing else.

It is unit-tested against the five Platonic solids before wiring:

| | planes | triangles | outward |
|---|---|---|---|
| tetrahedron | 4/4 | 4/4 | ✅ |
| cube | 6/6 | 12/12 | ✅ |
| octahedron | 8/8 | 8/8 | ✅ |
| icosahedron | 20/20 | 20/20 | ✅ |
| dodecahedron | 12/12 | 36/36 | ✅ |

Two more generic samplers joined `mesh` and `tube`:

- **`lathe(prof, steps)`** — surface of revolution, outward normal taken as the
  profile tangent turned a quarter turn. Because that falls out of the
  profile's *direction of travel*, a profile that doubles back gets
  inward-facing normals with no special case: the bowl's inner wall is the
  same expression as its outer wall, walked the other way.
- **`param(fn, …)`** — arbitrary parametric patch, normals by finite
  difference.

### Fixed during this pass

| # | Finding | Fix |
|---|---------|-----|
| 26 | **The Möbius strip rendered as a broken ring.** Modelled as a zero-thickness surface, it goes exactly edge-on twice per loop and disappears there. Geometrically correct, visually a bug — it read as a torus with two chunks missing. Caught by screenshotting it at three angles, not by any assertion. | Sweep a thin rectangle instead: two faces and two edges carried on the `{e, n}` frame that turns a half turn over the loop. That turn *is* the twist, which is why this is the one sweep that must not use the parallel-transport frame. |
| 27 | **The gear spent 15k of its 27k points on cap fill** at a fixed angular step, so the area near the bore was sampled ~4× denser than the rim needed. | Walk the radius outward with the existing `ringStep` and keep what falls inside a tooth: 27k → 15k points for the same picture. |
| 28 | **22 chips could only be reached by horizontal scrolling**, with "Hourglass" about four swipes off-screen. | The row wraps above 720px (two lines, everything visible at once) and stays a scroller below it, where wrapping would eat the stage. |

## Shading: one model → seven

Lambert · half-Lambert · toon · rim · specular · depth · normal, plus an
**Ambient** floor and a **Gloss** exponent. All seven are a branch and a few
flops in the existing inner loop — no second pass, no extra buffer. Depth mode
ignores the light entirely and shades purely by distance, which is the void
field's idea applied to the object itself.

## Charsets: five → nine, plus your own

Added the full **69-step gradient** (the one that makes an ASCII sphere look
genuinely smooth), **circles**, **hex** and **line art**. The select's last
entry is **custom…**, which reveals a text field: type any ramp, light to dark,
and it rides along in the share link. Guarded at both ends — an emptied field
falls back to a preset rather than dividing the luminance range by −1, and a
pasted 600-character ramp is cut to 254 because the index buffer is a
`Uint8Array`.

## Verification

Headless Chromium over 22 objects × 7 models × 10 charsets plus a hostile hash
(`#density=999&dist=0&ramp=99&shade=99&gloss=1e9&amb=5&obj=nope&charset=<600 chars>`):
no console or page errors, no ragged rows, void layer aligned to the object
grid in every frame, 59–65 fps throughout. Desktop and phone viewports both
clean, no horizontal page scroll.

## Still open (after the third pass)

1. Everything under *Still open (after the second pass)* stands — in
   particular the parallel-transport frame is still `play.html`-only, and the
   four files still duplicate each other.
2. **`hull()` is O(n⁴).** Fine for 20 vertices (~27k operations, once at load),
   useless past a few hundred. A real incremental hull would be the fix if
   anyone ever wants a shape with real vertex count.
3. **Möbius is the heaviest object at 18k points** — it needs dense sampling
   along `u` or the outer edge gaps. Auto-quality handles a weak device, but a
   radius-aware `u` step would be better than leaning on it.
4. **A custom charset containing a space punches holes in the object** — the
   blank glyph lands mid-ramp and the cell reads as empty even though the
   z-buffer holds a surface. It is arguably a nice thresholding effect, so it
   is left alone rather than filtered, but it is a footgun with no warning.
