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

- **Objects**: torus · trefoil · sphere · mug · cube · spring
- **Live**: density · zoom · distance (K2) · spin · light azimuth/elevation · charset · Y-stretch · invert
- **Shareable**: all state lives in the URL hash — `play.html#obj=cube&ramp=2&rotX=0.5&rotY=0.9`
- **Copy frame**: current ASCII straight to the clipboard
- **Auto quality**: thins the point cloud when fps drops below 28, recovers above 55
- **Keyboard**: `←→↑↓` turn · `space` spin · `R` reset · `C` copy · `S` share · `[` `]` density · `-` `+` zoom · `1`–`6` object · `H` panel
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
