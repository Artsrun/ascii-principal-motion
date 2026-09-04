# ASCII · Principal Motion → Hold

Interactive 3D surfaces rendered in pure ASCII using the a1k0n projection pipeline (surface samples → perspective → 1/z → luminance).

## Live demos
(Enable GitHub Pages: Settings → Pages → Deploy from branch `main` / root)

| Demo | URL | What it teaches |
|------|-----|-----------------|
| **Donut** | / | Classic principal motion (auto-spin + velocity steer) |
| **Trefoil** | /trefoil.html | Same principle, more complex path |
| **Hold** | /hold.html | Real-world-like free orbit + optional spin + everyday objects |

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
