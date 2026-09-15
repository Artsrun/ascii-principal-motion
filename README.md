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

### Objects — 12, in three families
| Family | Objects |
|--------|---------|
| **Solids** | sphere · cube · cylinder · cone · pyramid · capsule · octahedron · icosahedron |
| **Rings** | torus · trefoil · spring |
| **Things** | mug |

Polyhedra come from a shared `mesh(verts, faces, R)` sampler that lays a
barycentric lattice over each triangle and shades it flat from the face normal,
so adding another solid is a vertex list and a face list. Curved solids are
parametric, with a ring step that keeps arc spacing constant as the radius
shrinks — cone tips and cylinder caps don't pile up into a hot spot.

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
- **Live**: density · zoom · distance (K2) · depth fog · spin · light azimuth/elevation · charset · Y-stretch · invert
- **Shareable**: all state lives in the URL hash — `play.html#obj=icosa&ramp=2&fog=0.8&vfield=0.7`. Values are clamped on the way in, so a hand-edited link can't hang the tab.
- **Copy frame**: current ASCII straight to the clipboard
- **Auto quality**: thins the point cloud when fps drops below 28, recovers above 55
- **Keyboard**: `←→↑↓` turn · `space` spin · `R` reset · `C` copy · `S` share · `[` `]` density · `-` `+` zoom · `1`–`9` object · `,` `.` cycle · `V` void · `H` panel
- **Touch**: drag to turn · pinch or wheel to zoom · double-tap to reset

See [`UX-REVIEW.md`](./UX-REVIEW.md) for the full review of the three original demos —
what was fixed in this branch and what is still open.

## Hold demo — key improvements
- **Hold mode** (default): drag turns the object like holding it in your hand (absolute orientation)
- **Spin mode** (toggle): gentle continuous rotation; drag still steers
- Object switcher: Torus · Trefoil · Sphere · Mug
- Double-tap resets orientation
- Same mobile-first interaction grammar

## Roadmap toward cheap 3D
1. **ASCII** (now) — surface samples + z-buffer + luminance chars
2. **Canvas points** — same samples drawn as small discs/rects (bridge to “splat” intuition)
3. **WebGL points / low-poly** — same data, GPU projected, still no heavy assets
4. **Simple meshes** — very low poly real-world objects, same orbit controls
5. **4DGS / deformation** — load `4dgs-web` skill when you need real Gaussians, time, or compression

Keep the interaction language stable across every stage so the user never relearns how to “hold” an object.

Math origin: https://www.a1k0n.net/2011/07/20/donut-math.html
