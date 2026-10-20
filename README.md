# ASCII · Principal Motion → Hold

Interactive 3D surfaces rendered in pure ASCII using the a1k0n projection pipeline (surface samples → perspective → 1/z → luminance).

## Live demos
(Enable GitHub Pages: Settings → Pages → Deploy from branch `prod` / root)

| Demo | URL | What it teaches |
|------|-----|-----------------|
| **Donut** | / | Classic principal motion (auto-spin + velocity steer) |
| **Trefoil** | /trefoil.html | Same principle, more complex path |
| **Hold** | /hold.html | Real-world-like free orbit + optional spin + everyday objects |
| **Playtool** | /play.html | Every parameter live — and shareable as a URL |

Every page links to every other one in the header.

## Playtool — `play.html`
Turns the hard-coded pipeline into something you can drag.

### Objects — 22, in three families
| Family | Objects |
|--------|---------|
| **Solids** | sphere · cube · cylinder · cone · pyramid · capsule · tetrahedron · octahedron · dodecahedron · icosahedron · prism · gem |
| **Rings** | torus · trefoil · knot · spring · Möbius |
| **Things** | mug · gear · vase · bowl · hourglass |

Four generic samplers do the work, so most objects are one line:

- **`solid(verts, R)`** — every polyhedron is *just a list of points*.
  `hull()` finds the faces itself: for each triple of vertices it tests whether
  every other vertex lies on one side, gathers the coplanar vertices per face
  plane, sorts them around the centroid and fan-triangulates. There is no
  hand-written face table left to get the winding wrong, and a dodecahedron's
  pentagons come out as three triangles each rather than ten overlapping ones.
- **`lathe(prof, steps)`** — surface of revolution. `prof(u)` returns
  `[radius, y]` walking bottom to top; the outward normal is the profile
  tangent turned a quarter turn, so a profile that doubles back (a bowl's inner
  wall) gets inward-facing normals for free. Vase, bowl and hourglass are one
  expression each.
- **`tube(path, R, …)`** — sweeps a circle along any parametric curve on a
  parallel-transport frame. Trefoil, spring and the (2,5) torus knot.
- **`param(fn, …)`** — any parametric patch, normals by finite difference.

Curved primitives are parametric with a `ringStep` that keeps arc spacing
constant as the radius shrinks, so cone tips and cylinder caps don't pile up
into a hot spot.

### Shading — seven models
| Model | What it shows |
|-------|---------------|
| **Lambert** | the classic `n · l`, hard terminator |
| **Half-Lambert** | wrapped to `0.5 + 0.5(n · l)` — lit everywhere, no terminator |
| **Toon** | Lambert quantised to four bands |
| **Rim** | fresnel — bright where the surface turns away from the camera |
| **Specular** | Blinn-Phong highlight over Lambert, **Gloss** sets the exponent |
| **Depth** | light off entirely; brightness *is* distance |
| **Normal** | flat north light, `0.5 + 0.5·nᵧ` |

Plus an **Ambient** floor so the dark side keeps some structure. `M` cycles the
model, `shift+M` goes back.

### Charsets — nine presets and your own
`a1k0n classic` · `minimal` · **`long`** (the full 69-step gradient, for smooth
shading) · `blocks` · `dots` · `circles` · `hex` · `line art` · `binary` —
or pick **custom…** and type your own ramp, light to dark. It travels in the
share link with everything else. `N` cycles.

### Distance and the void
Depth is the thing ASCII normally throws away. Two controls put it back:

- **Depth fog** — attenuates each point's luminance by its depth inside the
  object's own bounding range, so the far side of a shape falls down the
  charset ramp. Normalised, so it looks the same at any K2, zoom or scale.
- **Void field** — a fixed point cloud the camera sits *inside*, projected
  through the same camera as the object, so it parallaxes against it instead
  of sitting on the glass. Glyph weight encodes distance (`+` near → `.` far).
  **Field** sets density, **Reach** how far the cloud extends, **Drift** a slow
  rotation of its own. The count is solved from the view frustum's solid angle
  each frame, so coverage stays ~3.5% of cells at any grid shape or zoom.

It renders as a second `<pre>` sharing the object's grid cell — same COLS/ROWS,
same font size, so the two character grids line up exactly and the void gets its
own colour and atmospheric mask without a span per glyph.

### Everything else
- **Live**: density · zoom · distance (K2) · depth fog · spin · shading model · light azimuth/elevation · ambient · gloss · charset · Y-stretch · invert
- **Shareable**: all state lives in the URL hash — `play.html#obj=dodeca&shade=3&ramp=2&fog=0.8&vfield=0.7`. Values are clamped on the way in, so a hand-edited link can't hang the tab.
- **Copy frame**: current ASCII straight to the clipboard
- **Auto quality**: thins the point cloud when fps drops below 28, recovers above 55
- **Keyboard**: `←→↑↓` turn · `space` spin · `R` reset · `C` copy · `S` share · `[` `]` density · `-` `+` zoom · `1`–`9` object · `,` `.` cycle object · `M` shading model · `N` charset · `V` void · `H` panel
- **Touch**: drag to turn · pinch or wheel to zoom · double-tap to reset

See [`UX-REVIEW.md`](./UX-REVIEW.md) for the full review of the three original demos —
what was fixed in this branch and what is still open.

## Hold demo — key improvements
- **Hold mode** (default): drag turns the object like holding it in your hand (absolute orientation)
- **Spin mode** (toggle): gentle continuous rotation; drag still steers
- Object switcher: Torus · Trefoil · Sphere · Mug
- Double-tap resets orientation
- Same mobile-first interaction grammar
# ASCII · Principal Motion

Interactive 3D surfaces on a phone: surface samples → perspective → 1/z → luminance.
No build step. Open a file, or the Pages site.

**Live:** https://artsrun.github.io/ascii-principal-motion/

GitHub Pages is served from branch `prod` (root). Merge into `prod` to publish.

## Demos

| Demo | Live | What it teaches |
|------|------|-----------------|
| **Donut** | [index](https://artsrun.github.io/ascii-principal-motion/) | Principal motion: auto-spin + velocity steer |
| **Trefoil** | [trefoil.html](https://artsrun.github.io/ascii-principal-motion/trefoil.html) | Same tube, knot path |
| **Hold** | [hold.html](https://artsrun.github.io/ascii-principal-motion/hold.html) | Hold in the hand + optional spin |
| **Fast Hold** | [fast-hold.html](https://artsrun.github.io/ascii-principal-motion/fast-hold.html) | Choose → load samples → show. Points first |

On device: tap / drag / double-tap. Fast Hold starts on a shape sheet.

## Package

Static HTML only.

```
index.html       donut
trefoil.html     knot tube
hold.html        orbit + chips
fast-hold.html   choose → load → show
LICENSE          MIT
.nojekyll        Pages: serve files as-is
```

Open any `.html` in a browser. No npm, no bundler.

## Fast Hold

- Shape sheet: Torus · Trefoil · Sphere · Mug
- Load gate samples the surface, then the object appears
- Default draw: canvas points (roadmap step 2); ASCII is a dock toggle
- Drag turns the object. Spin is optional.

## Roadmap toward cheap 3D

1. ASCII — samples + z-buffer + luminance chars
2. Canvas points — same samples as discs (Fast Hold default)
3. WebGL / WebGPU points — same data, GPU projected
4. Simple meshes — low-poly objects, same hold grammar
5. 4DGS / deformation — only when time or Gaussians are required

Keep the interaction language stable so the user never relearns how to hold an object.

Math: https://www.a1k0n.net/2011/07/20/donut-math.html
