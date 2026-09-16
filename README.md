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
