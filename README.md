# Generative-Lifeform-Generative-Art

A seed-based generative art catalogue for **Generative Lifeform**, a computational network system. Built as a single-file HTML page with p5.js, styled as a gallery-ready portfolio and licensing surface.

**Live:** `https://reyrove.github.io/Generative-Lifeform-Generative-Art/`

---

## Overview

**Generative Lifeform** renders glowing node-and-branch networks from numeric seeds. Each composition picks one of three modes — radial, network, cellular — then places nodes across a dark blue-violet gradient ground, connects them with luminous branches, and animates the field with 2–5 breathing energy pulses. Every node carries a layered glow and specular highlight; every branch is drawn three times for a soft outer bloom.

The page functions as a **catalogue and licensing surface** for fashion houses, textile studios, and surface designers. Every composition is reproducible by seed, meaning the same number always produces the identical organism.

---

## Features

### Generative Engine (p5.js)

- **Three modes** — each with its own topology and rhythm:
  - **Radial** — a central hub with 5–12 orbital nodes at 0.2–0.4× canvas radius
  - **Network** — 8–20 irregular nodes connected by proximity (distance < 0.3× canvas, 50% connection probability)
  - **Cellular** — 15–30 clustered nodes, each connecting to at most 3 neighbours within 0.2× canvas
- **Layered node glow** — every node is drawn as 5 concentric ellipses at 10% alpha, then a bright solid core, then a small specular highlight at 30% alpha offset to the upper-left
- **Double-stroked branches** — every branch is drawn 3 times with descending width and rising alpha, producing a soft outer bloom around a crisp inner line
- **Energy pulses** — 2–5 rings that oscillate through a sine wave, each wrapped in 4 concentric glow rings
- **Gradient ground** — a hue-walk from 230° at the top to 250° at the bottom of the canvas
- **Per-seed palette** — node hues cluster around:
  - Radial: `(rand × 60 + 200) % 360` — blue-violet band
  - Network: `rand(180, 300)` — teal to violet
  - Cellular: `rand(150, 270)` — green to violet
- **Full-spectrum branch hues** — every branch picks its own hue independently

### Same Seed → Many Surfaces

A single seed drives every instance on the page:

| Instance | Purpose | Tiling |
|----------|---------|--------|
| **Cover** | Hero composition | 1 × 1 |
| **Plate** | Main work (master) | 1 × 1 |
| **Frame** | Framed archival print | 1 × 1 |
| **Print** | Archival print surface | 1 × 1 |
| **Scarf** | Silk scarf (3:1) | 3 × 1 |
| **Textile** | Fabric yardage (4:3) | 2 × 2 |
| **Wall** | Wallpaper (2:3) | 1 × 2 |
| **Archive** | 8 unique seeds | 1 × 1 each |

### Controls

- **New Seed** button / `R` key — regenerate with a fresh seed
- **Download** button / `S` key — export the plate canvas as PNG
- **Click any archive plate** — load that seed into the main view

### Performance

- Debounced resize handling (250ms)
- `IntersectionObserver` pauses off-screen sketches
- `visibilitychange` pauses rendering when the tab is hidden
- `pixelDensity(1)` for a lighter canvas footprint
- Two-frame render (`maxFrames: 2`) so tiles draw once and stop
- Energy pulses frozen at a fixed phase (see note below)

---

## File Structure

```
Generative-Lifeform-Generative-Art/
├── index.html          # Single-file page (structure + styles + generative engine)
├── README.md
└── images/
    ├── fav.svg         # Favicon
    ├── tote.png        # Mockup: tote bag
    ├── tee.png         # Mockup: t-shirt
    └── cushion.png     # Mockup: cushion
```

No build step. No dependencies to install. p5.js is loaded from a CDN.

---

## Getting Started

1. Clone the repo:
   ```bash
   git clone https://github.com/reyrove/Generative-Lifeform-Generative-Art.git
   cd Generative-Lifeform-Generative-Art
   ```
2. Add the three mockup images to `images/` (optional — they appear in the Commission section).
3. Open `index.html` in a browser.

That's it. The page self-initialises on `DOMContentLoaded`.

---

## How the Generative System Works

### Seeded Random

A linear congruential generator replaces p5's native randomness inside each sketch:

```js
class SeededRandom {
  constructor(s) { this.state = (Math.abs(s) % 2147483647) || 1; }
  rand() { this.state = (this.state * 9301 + 49297) % 233280; return this.state / 233280; }
  range(min, max) { return this.rand() * (max - min) + min; }
  int(min, max) { return Math.floor(this.range(min, max + 1)); }
  pick(arr) { return arr[Math.floor(this.rand() * arr.length)]; }
}
```

Every aesthetic decision — mode, node count, node positions, branch connections, pulse count, background hue, per-node colour — is drawn from this generator in a fixed order. Same seed, same sequence, same organism.

### Parameter Space (per seed)

| Parameter | Range |
|-----------|-------|
| Mode | 3 options (radial → cellular) |
| Node count | 6 (radial) / 8–20 (network) / 15–30 (cellular) |
| Node size | 0.02 – 0.10 × canvas |
| Node hue | mode-dependent band |
| Node saturation | 70 – 90 |
| Node brightness | 70 – 90 |
| Branch thickness | 0.003 – 0.015 × canvas |
| Branch hue | 0 – 360 (full spectrum) |
| Pulse count | 2 – 5 |
| Pulse size | 0.1 – 0.3 × canvas |
| Pulse speed | 0.02 – 0.05 |
| Background hue | 230 – 250 (gradient) |

### The Three Modes

**Radial** — a hub-and-spoke topology:

```js
// central node at (cs/2, cs/2)
// 5–12 orbital nodes at radius 0.2–0.4 × cs
// every orbital node connects to the hub
// ~30% of neighbouring orbitals also connect to each other
```

Reads as a starburst, a cell, or a compass rose. The central hub anchors the composition.

**Network** — a proximity graph:

```js
// 8–20 nodes scattered in the 0.1–0.9 × cs range
// for every pair within 0.3 × cs, connect with 50% probability
// edge thickness scales inversely with distance
```

Reads as a constellation, a social graph, or a neural tangle. No anchor; the graph is distributed.

**Cellular** — a limited-degree graph:

```js
// 15–30 nodes scattered across the full canvas
// each node connects to at most 3 neighbours within 0.2 × cs
// connection probability 60%
```

Reads as a tissue sample, a Voronoi approximation, or a scatter of biological cells.

### The Glow

Every visual element is drawn with a layered glow. Nodes are stacked 5 deep at 10% alpha, then a solid core, then a specular highlight. Branches are stroked 3 times with descending width. Pulses are wrapped in 4 concentric rings at 15% alpha. This gives the composition its "bioluminescent" quality — everything reads as if it is emitting light from a dark ground.

### Energy Pulses

Each pulse is a ring whose radius oscillates through a sine wave:

```js
const pulseSize = pulse.size * (0.5 + 0.5 * Math.sin(phase * Math.PI * 2));
```

In the original sketch, `phase` runs continuously with `frameCount`. In this catalogue, all sketches render for **two frames and stop**, so the animation cannot run. To preserve the visual rhythm, **the pulses are drawn at a fixed phase (0.5)** — a mid-breath moment where the rings sit at half amplitude.

If you want the animation back for the **Plate** only (leaving cover and surfaces frozen for performance), see the Customisation section below.

### Tiling Logic

Rectangular surfaces (scarf, textile, wall) tile the square composition N×M. Each tile is drawn into its own translated coordinate space, centred within its cell:

```js
const tileW = canvasW / tileCols;
const tileH = canvasH / tileRows;
const tileSize = Math.min(tileW, tileH);
// draw square of side tileSize at (ox, oy) for each cell
```

This preserves the square composition's proportions regardless of surface aspect ratio.

---

## Customisation

### Change the default seed

Edit the init function:

```js
function init() {
  seed = 123456789;  // fixed seed instead of random
  // ...
}
```

### Restore the pulse animation on the Plate

Change `maxFrames` in `drawPlate()`:

```js
createLiveP5({
  slotId: 'plate-slot',
  seedValue: seed,
  isMaster: true,
  maxFrames: Infinity,  // was 2 — animates the plate
  tile: { cols: 1, rows: 1 },
  onCreated: (inst) => { plateP5 = inst; }
});
```

Then change the pulse size calculation in `drawComposition` back to use `p.frameCount`:

```js
// Replace:
const pulseSize = pulse.size * (0.5 + 0.5 * Math.sin(frozenPhase * Math.PI * 2));

// With:
const pulseSize = pulse.size * (0.5 + 0.5 * Math.sin(p.frameCount * pulse.speed));
```

The pulses will then breathe at their natural rates.

### Adjust the mode distribution

Inside `drawComposition`:

```js
const mode = sr.int(0, 2);
```

Currently balanced across the three modes. To favour one, replace with a weighted pick:

```js
const mode = sr.rand() < 0.6 ? 0 : sr.int(1, 2);   // 60% radial
```

### Adjust the node count ranges

- **Radial:** `sr.int(5, 11)`
- **Network:** `sr.int(8, 19)`
- **Cellular:** `sr.int(15, 29)`

Higher values produce denser fields; lower values produce sparser, more legible topologies.

### Adjust the connection distances

- **Network:** `distance < cs * 0.3`
- **Cellular:** `distance < cs * 0.2`

Larger distances produce more connections per node; smaller distances produce more isolated clusters.

### Adjust the node sizes

```js
size: sr.range(0.02, 0.06) * cs   // network
```

Increase for larger, more prominent nodes. The current ranges are calibrated so nodes stay distinguishable from branches.

### Change the glow intensity

Every glow layer is drawn at a fixed alpha:

- Node glow: `p.fill(..., 0.1)`
- Branch outer glow: `p.stroke(..., 0.2)`
- Pulse glow: `p.stroke(..., 0.15)`

Increase these for a stronger, more luminous effect; decrease for a subtler one.

### Change the background gradient

```js
const hue = 230 + (y / cs) * 20 + sr.range(-cs * 0.01, cs * 0.01);
```

- Change `230` for a different starting hue.
- Change `20` for a wider or narrower hue walk.
- Change the jitter range for more or less noise in the background.

### Change the pulse count

```js
const pulseCount = sr.int(2, 5);
```

Increase for a busier composition, decrease for a more restrained one.

### Change surface tiling

Edit `surfaceDefs`:

```js
const surfaceDefs = [
  { id: 'surfacePrint',   wrapId: 'wrap-surfacePrint',   tile: { cols: 1, rows: 1 } },
  { id: 'surfaceScarf',   wrapId: 'wrap-surfaceScarf',   tile: { cols: 3, rows: 1 } },
  // ...
];
```

The `tile` object accepts any `{ cols, rows }` pair.

### Replace mockup images

Swap the files in `images/` or update the `src` attributes in the Commission section:

```html
<img src="images/your-mockup.png" alt="..." loading="lazy" />
```

---

## Design System

| Token | Value | Use |
|-------|-------|-----|
| `--bg` | `#0a0a0a` | Page background |
| `--paper` | `#f4f1ea` | Light surface (Surfaces section) |
| `--gold` | `#b8935a` | Primary accent |
| `--gold-soft` | `#d4b483` | Italic emphasis |
| `--serif` | Cormorant Garamond | Headings, titles |
| `--mono` | DM Mono | Labels, metadata, UI |

Type scale, spacing, and section rhythm match the Skyline, Portrait, Girih, Gothic Grid, Order to Chaos, Wallpaper Groups, Wandering Tiles, Food Grid, Framed Line, Deconstructed Stage, Layered Diagram, Japanese Patterns, Matrix Rain, Organic Shapes, and Recursive Grid templates. Responsive breakpoints at 900px, 720px, 560px, and 400px.

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `R` | Regenerate with a new seed |
| `S` | Download plate as PNG |
| `Enter` / `Space` | Activate focused archive item |
| `Esc` | Close legal modal |

---

## Browser Support

Modern evergreen browsers with:

- Canvas 2D (universal)
- `IntersectionObserver` (universal in modern browsers)
- `backdrop-filter` (Safari 9+, Chrome 76+, Firefox 103+)
- CSS custom properties (universal)

Fallback: if `IntersectionObserver` is unavailable, sketches render continuously.

---

## Accessibility

- Archive items are keyboard-focusable with `role="button"` and `aria-label`
- Legal modal traps focus, closes on `Esc` and overlay click
- `prefers-reduced-motion` disables transitions and smooth scroll
- All interactive elements have visible focus states (via browser default)

---

## Section Map

| Section | ID | Purpose |
|---------|-----|---------|
| Cover | — | Hero title + live organism |
| Statement | `#statement` | Artist statement |
| Plate | `#work` | Main work + metadata + controls |
| Surfaces | `#application` | Framed print + 4 tiled surfaces |
| Process | `#process` | Three process cards |
| Archive | `#archive` | 8 selectable seeds |
| Commission | `#commission` | Services + mockups + CTA |
| Colophon | — | Footer, links, legal |
| Legal modal | `#legalModal` | Licensing / Terms / Credits |

---

## Deployment (GitHub Pages)

1. Push the repo to GitHub.
2. Go to **Settings → Pages**.
3. Under **Source**, select `Deploy from a branch`.
4. Choose branch `main` and folder `/ (root)`.
5. Save — the site goes live at `https://reyrove.github.io/Generative-Lifeform-Generative-Art/`.

---

## Notes on the Source Sketch

The generative engine is adapted almost verbatim from the original p5 sketch. The adaptations that turn it into a catalogue page:

1. **Seeded random replaces p5's `random()`** — a local `SeededRandom` class drives every choice. The original sketch calls `setSeeds()` for fxhash compatibility; that's replaced with the seeded generator directly.
2. **Sketch factory** — `makeSketch({ slotId, seedValue, ... })` returns a p5 sketch that can be instantiated into any DOM slot, enabling cover / plate / frame / surfaces / archive from one seed.
3. **DOM-driven canvas sizing** — `cs = min(windowHeight, windowWidth)` becomes slot-rect dimensions.
4. **Tiling support** — the `tile: { cols, rows }` parameter repeats the square composition across rectangular surfaces.
5. **Drawing logic takes `p` explicitly** — the original used p5's global mode; here `p` is passed so the factory closure works.
6. **`TWO_PI` replaced with `Math.PI * 2`** — for clarity inside the factory closure.
7. **`map(distance, 0, cs * 0.3, 0.01, 0.005)` inlined** — replaced with `(0.01 - t * 0.005) * cs` where `t = distance / (cs * 0.3)`. Mathematically identical.
8. **Pulse animation frozen at phase 0.5** — since the catalogue renders each sketch for two frames then stops, the `frameCount`-driven pulse animation would not be visible. The pulses are drawn at a fixed mid-breath phase so the composition still reads as a moment of a living system. To restore the animation on the Plate only, change `maxFrames` from `2` to `Infinity` (see Customisation).
9. **`windowResized` replaced by a debounced `handleResize`** — all sketches are destroyed and recreated on resize (250ms debounce), since their canvases are locked to slot dimensions.
10. **`triggerPreview` removed** — the original calls it for fxhash thumbnail generation; not needed in the plain-browser catalogue.

---

## Related Repositories

| Repo | Theme |
|------|-------|
| [`Skyline-Series`](https://github.com/reyrove/Skyline-Series) | Themed cityscapes |
| [`Portrait-Series-Generative-Art`](https://github.com/reyrove/Portrait-Series-Generative-Art) | Generative faces |
| [`Girih-2-Generative-Art`](https://github.com/reyrove/Girih-2-Generative-Art) | Islamic geometric ornament |
| [`Gothic-Grid-Generative-Art`](https://github.com/reyrove/Gothic-Grid-Generative-Art) | Mosaic grid composition |
| [`Order-to-Chaos-Generative-Art`](https://github.com/reyrove/Order-to-Chaos-Generative-Art) | Gradient grid composition |
| [`Wallpaper-Groups-Generative-Art`](https://github.com/reyrove/Wallpaper-Groups-Generative-Art) | Crystallographic symmetry |
| [`Wandering-Tiles-Generative-Art`](https://github.com/reyrove/Wandering-Tiles-Generative-Art) | Line-drawing composition |
| [`Food-Grid-Generative-Art`](https://github.com/reyrove/Food-Grid-Generative-Art) | Illustrated food grid |
| [`Framed-Line-Generative-Art`](https://github.com/reyrove/Framed-Line-Generative-Art) | Gesture line composition |
| [`Deconstructed-Stage-Generative-Art`](https://github.com/reyrove/Deconstructed-Stage-Generative-Art) | Suprematist composition |
| [`Layered-Diagram-Generative-Art`](https://github.com/reyrove/Layered-Diagram-Generative-Art) | Layered linework composition |
| [`Japanese-Patterns-Generative-Art`](https://github.com/reyrove/Japanese-Patterns-Generative-Art) | Traditional motif composition |
| [`Matrix-Rain-Generative-Art`](https://github.com/reyrove/Matrix-Rain-Generative-Art) | Digital rain composition |
| [`Organic-Shapes-Generative-Art`](https://github.com/reyrove/Organic-Shapes-Generative-Art) | Layered petal-form composition |
| [`Recursive-Grid-Generative-Art`](https://github.com/reyrove/Recursive-Grid-Generative-Art) | Recursive subdivision composition |
| **`Generative-Lifeform-Generative-Art`** | Network / node-graph composition *(this repo)* |

All sixteen share the same catalogue template, design tokens, and seed-based reproducibility model.

---

## Credits

- **Design & Generative System** — Reyhaneh Daneshdoost
- **Typefaces** — Cormorant Garamond, DM Mono
- **Engine** — p5.js 1.9.0
- **Modes** — Radial · Network · Cellular
- **Edition** — Generative Lifeform, Autumn 2026

---

## License

All artwork, code, and generative systems are the intellectual property of the artist. Compositions may not be reproduced, redistributed, resold, or adapted without a written license.

For licensing, commissions, or private systems: **reyhanehdaneshdoost@gmail.com**