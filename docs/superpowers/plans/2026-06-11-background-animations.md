# Background Animations Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a dramatic alien-space SVG background — layered starfield, glowing nebula washes, and shooting stars — to the full viewport using GSAP, without affecting gameplay or accessibility.

**Architecture:** A fixed `<svg id="bg-canvas">` sits behind all page content (`z-index: 0`). A new `bg.js` script uses the GSAP CDN global to generate SVG elements at runtime and drive all animations. No build step; no changes to game logic.

**Tech Stack:** SVG, GSAP 3 (CDN), vanilla JS (IIFE), CSS custom properties already defined in `style.css`.

---

### Task 1: CSS — Position the background SVG and lift the console

**Files:**
- Modify: `style.css`

- [ ] **Step 1: Add `#bg-canvas` fixed-positioning rule**

Open `style.css`. After the `body { ... }` block (around line 34), add:

```css
#bg-canvas {
  position: fixed;
  inset: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  pointer-events: none;
}
```

- [ ] **Step 2: Give `.console` a stacking context so it floats above the SVG**

Find the `.console` rule (around line 36). Add `position: relative; z-index: 1;` to it:

```css
.console {
  position: relative;
  z-index: 1;
  max-width: 1060px;
  margin: 0 auto;
  padding: 28px 20px 40px;
}
```

- [ ] **Step 3: Commit**

```bash
git add style.css
git commit -m "style: add bg-canvas fixed layer and lift console z-index"
```

---

### Task 2: HTML — Add the SVG element and script tags

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Insert the empty SVG as the first child of `<body>`**

Open `index.html`. Add the SVG on the line immediately after `<body>` (before `<div class="console">`):

```html
<body>

  <svg id="bg-canvas" aria-hidden="true"></svg>

  <div class="console">
```

- [ ] **Step 2: Add GSAP CDN and bg.js script tags before `</body>`**

Immediately before the existing `<script type="module" src="script.js"></script>` line, add:

```html
  <!-- Background animation (GSAP + bg.js must load before script.js reads the DOM) -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
  <script src="bg.js"></script>
```

The full bottom of `<body>` should now read:

```html
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
  <script src="bg.js"></script>
  <!-- Game logic + student customization area -->
  <script type="module" src="script.js"></script>
</body>
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add bg-canvas SVG and GSAP script tags"
```

---

### Task 3: Create bg.js — full animation logic

**Files:**
- Create: `bg.js`

- [ ] **Step 1: Create `bg.js` with the complete implementation**

Create `bg.js` in the project root with this content:

```javascript
(function () {
  'use strict';

  const NS  = 'http://www.w3.org/2000/svg';
  const svg = document.getElementById('bg-canvas');
  const vw  = window.innerWidth;
  const vh  = window.innerHeight;

  function r(min, max) { return min + Math.random() * (max - min); }
  function pick(arr)   { return arr[Math.floor(Math.random() * arr.length)]; }

  function mkEl(tag, attrs, parent) {
    const el = document.createElementNS(NS, tag);
    Object.entries(attrs).forEach(([k, v]) => el.setAttribute(k, v));
    if (parent) parent.appendChild(el);
    return el;
  }

  // Gradient defs container
  const defs = mkEl('defs', {}, svg);

  // ── Stars ──────────────────────────────────────────────────────────────────
  const STAR_COLORS = ['#ffffff', '#e8f8ff', '#5eead4', '#5eead4', '#a78bfa', '#ffb347'];

  const LAYERS = [
    { id: 'stars-slow', count: 40, rMin: 0.5, rMax: 1.2, drift: 5,  durMin: 20, durMax: 30 },
    { id: 'stars-mid',  count: 50, rMin: 0.8, rMax: 1.8, drift: 10, durMin: 10, durMax: 15 },
    { id: 'stars-fast', count: 30, rMin: 1.2, rMax: 2.5, drift: 20, durMin: 5,  durMax: 8  },
  ];

  const starsG = mkEl('g', { id: 'stars' }, svg);
  LAYERS.forEach(layer => {
    const g = mkEl('g', { id: layer.id }, starsG);
    for (let i = 0; i < layer.count; i++) {
      mkEl('circle', {
        cx:      r(0, vw).toFixed(1),
        cy:      r(0, vh).toFixed(1),
        r:       r(layer.rMin, layer.rMax).toFixed(2),
        fill:    pick(STAR_COLORS),
        opacity: r(0.4, 1.0).toFixed(2),
      }, g);
    }
  });

  // ── Nebulas ────────────────────────────────────────────────────────────────
  const NEBULA_DEFS = [
    { x: vw * 0.15, y: vh * 0.20, rx: 280, ry: 180, color: '#5eead4', rot: -20 },
    { x: vw * 0.80, y: vh * 0.10, rx: 320, ry: 200, color: '#7c3aed', rot:  15 },
    { x: vw * 0.60, y: vh * 0.70, rx: 260, ry: 160, color: '#ec4899', rot:  30 },
    { x: vw * 0.10, y: vh * 0.75, rx: 300, ry: 190, color: '#6366f1', rot: -10 },
    { x: vw * 0.90, y: vh * 0.50, rx: 240, ry: 150, color: '#0ea5e9', rot:  25 },
    { x: vw * 0.45, y: vh * 0.40, rx: 350, ry: 220, color: '#8b5cf6', rot:   5 },
  ];

  const nebulaG = mkEl('g', { id: 'nebulas' }, svg);
  NEBULA_DEFS.forEach((cfg, i) => {
    const gid  = `ng${i}`;
    const grad = mkEl('radialGradient', { id: gid, cx: '50%', cy: '50%', r: '50%' }, defs);
    mkEl('stop', { offset: '0%',   'stop-color': cfg.color, 'stop-opacity': '0.18' }, grad);
    mkEl('stop', { offset: '100%', 'stop-color': cfg.color, 'stop-opacity': '0'    }, grad);
    mkEl('ellipse', {
      cx:      cfg.x.toFixed(1),
      cy:      cfg.y.toFixed(1),
      rx:      cfg.rx,
      ry:      cfg.ry,
      fill:    `url(#${gid})`,
      opacity: r(0.06, 0.15).toFixed(3),
    }, nebulaG);
  });

  // ── Shooting star pool ─────────────────────────────────────────────────────
  const shootersG = mkEl('g', { id: 'shooters' }, svg);
  const POOL = Array.from({ length: 4 }, () =>
    mkEl('line', {
      x1: -200, y1: -200, x2: -100, y2: -100,
      stroke: '#5eead4',
      'stroke-width':   r(1, 2).toFixed(1),
      'stroke-linecap': 'round',
      'stroke-dasharray': '80 300',
      opacity: 0,
    }, shootersG)
  );

  // ── Reduced motion guard ───────────────────────────────────────────────────
  if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return;

  // ── GSAP: star drift + twinkle ─────────────────────────────────────────────
  LAYERS.forEach(layer => {
    document.querySelectorAll(`#${layer.id} circle`).forEach(star => {
      gsap.to(star, {
        y: r(layer.drift * 0.5, layer.drift),
        x: r(-5, 5),
        duration: r(layer.durMin, layer.durMax),
        repeat: -1, yoyo: true,
        ease: 'sine.inOut',
        delay: r(0, layer.durMax),
      });
      gsap.to(star, {
        opacity: r(0.3, 1.0),
        duration: r(2, 5),
        repeat: -1, yoyo: true,
        ease: 'sine.inOut',
        delay: r(0, 5),
      });
    });
  });

  // ── GSAP: nebula setup + pulse ─────────────────────────────────────────────
  document.querySelectorAll('#nebulas ellipse').forEach((el, i) => {
    gsap.set(el, { rotation: NEBULA_DEFS[i].rot, transformOrigin: 'center center' });
    gsap.to(el, {
      opacity:         r(0.12, 0.18),
      scale:           r(1.05, 1.15),
      rotation:        `+=${r(-15, 15).toFixed(1)}`,
      transformOrigin: 'center center',
      duration:        r(8, 14),
      repeat: -1, yoyo: true,
      ease: 'sine.inOut',
      delay: r(0, 8),
    });
  });

  // ── GSAP: shooting stars ───────────────────────────────────────────────────
  let idx = 0;

  function shoot() {
    const line = POOL[idx % POOL.length];
    idx++;

    const x1  = r(vw * 0.3, vw * 1.1);
    const y1  = r(-80, vh * 0.25);
    const len = r(120, 220);
    const rad = r(30, 60) * Math.PI / 180;
    const x2  = x1 - len * Math.cos(rad);
    const y2  = y1 + len * Math.sin(rad);

    gsap.set(line, { attr: { x1, y1, x2: x1, y2: y1 }, opacity: 0 });
    gsap.timeline({ onComplete: () => gsap.delayedCall(r(3, 6), shoot) })
      .to(line, { opacity: 1, duration: 0.05 })
      .to(line, { attr: { x1, y1, x2, y2 }, duration: 0.55, ease: 'power2.in' }, '<')
      .to(line, { opacity: 0, duration: 0.25 }, '-=0.2');
  }

  gsap.delayedCall(r(1, 4), shoot);
})();
```

- [ ] **Step 2: Open the page via Live Server and verify visually**

Open `index.html` with VS Code Live Server (`http://127.0.0.1:5500`).

Expected results:
- Dark starfield visible behind all panels — stars of varying sizes and colors (white, cyan, purple, amber)
- Soft glowing blobs (cyan, violet, magenta) visible in the background corners and center
- Stars slowly drifting and twinkling
- A shooting star streaks diagonally across the screen within the first 4 seconds, then repeats every 3–6 seconds
- Game canvas, webcam panel, buttons, and all text remain fully interactive and readable

- [ ] **Step 3: Verify `pointer-events: none` — click the Launch mission button**

Click the "Launch mission" button on the start screen. It must respond normally. If it doesn't, check that `pointer-events: none` is present on `#bg-canvas` in `style.css`.

- [ ] **Step 4: Commit**

```bash
git add bg.js
git commit -m "feat: add alien space background animations with GSAP"
```

---

### Task 4: Verify reduced-motion accessibility

**Files:**
- No code changes — verification only

- [ ] **Step 1: Simulate `prefers-reduced-motion: reduce` in Chrome DevTools**

1. Open DevTools → Rendering tab (via three-dot menu → More tools → Rendering)
2. Set "Emulate CSS media feature prefers-reduced-motion" to `reduce`
3. Reload the page

- [ ] **Step 2: Confirm all GSAP animations are paused**

Expected: stars, nebulas, and shooting stars are all static. The SVG elements still render (starfield and nebula washes visible) but nothing moves.

- [ ] **Step 3: Restore normal motion and confirm animations resume**

Set "Emulate CSS media feature prefers-reduced-motion" back to "No emulation". Reload. Animations should play normally.

- [ ] **Step 4: Final commit**

```bash
git add .
git commit -m "chore: verify reduced-motion accessibility for background animations"
```
