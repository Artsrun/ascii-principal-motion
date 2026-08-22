# ASCII · Principal Motion

Interactive 3D surfaces rendered in pure ASCII using the classic a1k0n projection pipeline (surface sampling → perspective → 1/z → luminance).

## Live demos
(Enable GitHub Pages from Settings → Pages → main / root if not already live)

| Demo | URL | Shape |
|------|-----|-------|
| **Donut** | https://artsrun.github.io/ascii-principal-motion/ | Classic torus |
| **Trefoil** | https://artsrun.github.io/ascii-principal-motion/trefoil.html | Tubular trefoil knot |

## Interaction (identical on both)
- **Tap** — pause / resume
- **Drag** — steer angular velocity
- **Double-tap** — reset
- **Long-press** — math overlay

Same principle, different center path. The trefoil is the natural next step after the torus: a circle of radius R₁ swept along a more interesting curve.

## Skills
- `ascii-donut-web` — owns the interactive ASCII / point-cloud torus + mobile grammar
- `4dgs-web` — next step when you leave pure samples for real Gaussians / deformation fields

Math origin: https://www.a1k0n.net/2011/07/20/donut-math.html
