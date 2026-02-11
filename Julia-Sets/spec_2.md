# Julia Set Explorer — Implementation Spec

## Overview

A single self-contained `index.html` file implementing an interactive Julia Set explorer
targeting college students with some math background. The app should be visually striking,
educationally informative, and smooth to interact with.

---

## Technology Stack

- **Single file:** Everything (HTML, CSS, JS, GLSL shaders) in one `index.html`
- **Rendering:** WebGL via a full-screen `<canvas>` element with GLSL fragment shaders
- **No external dependencies:** No libraries, no CDN imports — fully offline-capable
- **Target browsers:** Modern Chrome, Firefox, Safari, Edge

---

## Layout & UI Structure

The canvas fills the entire viewport. All UI elements float as overlay panels on top of
the fractal. The fractal is always visible behind the controls.

### Overlay Panels

**Top-left: App title + dark/light mode toggle**
- App name: "Julia Set Explorer"
- A toggle button/icon to switch between dark and light UI themes
- The fractal itself remains dark (black background) regardless of UI theme

**Top-right: Preset Gallery**
- A collapsible panel showing ~8 named presets (see Presets section below)
- Each preset is a clickable thumbnail or labeled button that loads a specific `c` value
- Panel can be opened/closed with a toggle button

**Bottom-left: Math Panel**
- A collapsible panel showing the current equation and parameter values
- Displays: `f(z) = z² + c` with the current value of `c` shown as `c = [real] + [imag]i`
- Brief one-sentence description of what a Julia Set is
- Shows current zoom level and center coordinates

**Bottom-right: Controls Panel**
- Always visible, compact
- Contains all interactive controls (see Controls section below)

---

## Controls

### Parameter Tweaking (c value)
- Two sliders: **Real part** of c (range: −2.0 to 2.0) and **Imaginary part** of c (range: −2.0 to 2.0)
- Numeric readouts next to each slider showing current value to 4 decimal places
- Changes re-render in real time as sliders are dragged

### Color Theme
- A dropdown or row of swatches to select a color palette. Include at least 5 options:
  1. **Classic** — smooth blue-to-white gradient
  2. **Fire** — black → red → orange → yellow
  3. **Electric** — dark purple → cyan → white
  4. **Monochrome** — black and white
  5. **Pastel** — soft pink, lavender, mint
- Color mapping should use smooth/continuous coloring (not banded iteration counts)

### Iteration Depth
- A slider for **Max Iterations** (range: 50–1000, default: 200)
- Higher values = more detail at deep zooms but slower render

### Reset Button
- Resets zoom, pan, and c value to defaults

---

## Navigation (Zoom & Pan)

- **Scroll wheel / pinch:** Zoom in and out, centered on cursor position
- **Click and drag:** Pan the view
- **Double-click:** Zoom in 2× centered on click point
- Zoom should be smooth (animated, not instant jumps)
- Display current zoom level in the Math Panel

---

## Preset Gallery

Eight named presets with their `c` values and short descriptions:

| Name | c value | Description |
|---|---|---|
| Dendrite | c = 0 + 1i | Branching tree-like structure |
| Douady Rabbit | c = −0.1226 + 0.7449i | Three-fold rabbit ears |
| San Marco | c = −0.7269 + 0.1889i | Interlocking circles |
| Siegel Disk | c = −0.3905 − 0.5867i | Smooth spiral disk |
| Airplane | c = −1.7549 + 0i | Abstract airplane shape |
| Lightning | c = 0.285 + 0.01i | Dense lightning bolt fractal |
| Spiral | c = −0.7269 − 0.1889i | Spiral arms |
| Dragon | c = −0.8 + 0.156i | Dragon curve-like tendrils |

Clicking a preset should:
1. Set the c value sliders to match
2. Reset zoom and pan to default view
3. Smoothly transition to the new fractal (brief animated fade or redraw)

---

## Math Panel — Educational Content

The math panel should explain the concept briefly and dynamically:

- **Equation display:** `f(z) = z² + c` rendered in a clean monospace or math font
- **Current c:** Updates live as sliders move, e.g. `c = -0.7269 + 0.1889i`
- **What is a Julia Set?** — Static one-liner: *"The Julia Set is the boundary between points that escape to infinity and points that remain bounded under repeated application of f(z)."*
- **Zoom level:** e.g. `Zoom: 1.00×` or `Zoom: 142.5×`
- **Center:** e.g. `Center: (−0.012, 0.334)`

---

## WebGL Rendering Details

### Fragment Shader Requirements
- Compute Julia Set iteration in GLSL
- Use **smooth coloring** (continuous, not integer iteration count):
  - `smoothed = iter - log2(log2(|z|))` formula
- Map smoothed value through the selected color palette as a GLSL gradient
- Escape radius: 4.0
- Support high-resolution canvas (account for `devicePixelRatio`)

### Uniforms needed
- `u_resolution` — canvas size in pixels
- `u_c` — the complex parameter c (vec2)
- `u_zoom` — current zoom level (float)
- `u_center` — pan offset (vec2)
- `u_maxIter` — max iterations (int)
- `u_colorScheme` — active palette index (int, 0–4)

### Vertex Shader
Simple passthrough — render a full-screen quad covering the canvas.

---

## Dark / Light UI Theme

The toggle switches the **overlay panels** only (not the fractal background).

| Element | Dark mode | Light mode |
|---|---|---|
| Panel background | `rgba(0,0,0,0.75)` | `rgba(255,255,255,0.85)` |
| Panel text | `#f0f0f0` | `#111111` |
| Slider accent | `#60a5fa` (blue) | `#2563eb` (blue) |
| Borders | `rgba(255,255,255,0.15)` | `rgba(0,0,0,0.15)` |

Default: Dark mode.

---

## Responsive Behavior

- Canvas always fills the full viewport (no scrollbars)
- On narrow screens (< 600px width), overlay panels should collapse to icon-only buttons
  that expand on tap
- Touch support: pinch-to-zoom and drag-to-pan on mobile

---

## Default State on Load

- `c = −0.7269 + 0.1889i` (San Marco preset — visually impressive on first load)
- Zoom: 1.0× (standard view, complex plane from roughly −2 to 2 on both axes)
- Center: (0, 0)
- Max iterations: 200
- Color scheme: Classic
- UI theme: Dark
- Math panel: open
- Preset gallery: closed

---

## File Structure

Everything lives in a single `index.html`:

```
index.html
├── <style> block — all CSS
├── <body> — canvas + overlay HTML
└── <script> block
    ├── WebGL setup (context, shaders, buffers)
    ├── GLSL shader source strings (vertex + fragment)
    ├── Render loop
    ├── Input handlers (mouse, touch, wheel)
    ├── UI handlers (sliders, presets, theme toggle)
    └── Preset data
```

No external fonts, no CDN, no build step required. Open `index.html` in a browser and it works.

---

## Out of Scope (Future Enhancements)

- Save / export image (PNG download)
- Mandelbrot Set companion view
- Animated c-value tours
- Deep zoom beyond ~10⁶× (would require arbitrary precision arithmetic)
