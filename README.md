# ASCII · Principal Motion

Interactive spinning torus rendered in pure ASCII using the classic a1k0n (2011) math.

**Live demo (enable Pages first):**  
https://artsrun.github.io/ascii-principal-motion/

## Real-device interaction
- **Tap** — pause / resume principal motion
- **Drag** — steer angular velocity
- **Double-tap** — reset angles & speeds
- **Long-press** — show the math overlay

Targets 30–60 fps on modern phones. No libraries. Surface sampling + perspective + 1/z depth + simple diffuse luminance.

## Skill set created
- New skill: `ascii-donut-web` (interactive ASCII / lightweight point torus for mobile web)
- Composes with existing `4dgs-web` skill for progression toward real Gaussian / 4D deformation fields on the web.

ASCII principal motion is the pedagogical “training wheels” before point-cloud or full splat viewers.

## Enable GitHub Pages
1. Open **Settings → Pages**
2. Source: **Deploy from a branch**
3. Branch: `main` / `(root)`
4. Save — the site is live in about a minute at the URL above.

Math source: https://www.a1k0n.net/2011/07/20/donut-math.html
