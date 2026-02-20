# Readability Explorer — Specification

## Overview

**Readability Explorer** is an interactive, single-file web application (`index.html`) that lets users experiment with background color, text color, and font size combinations to understand how those choices affect legibility and WCAG contrast compliance on digital screens.

The aesthetic direction is **"Lab Terminal meets Retro Oscilloscope"** — dark matte UI panels with phosphor-green accent glows, monospaced instrument-style typography for controls, and a central "viewport" that feels like looking through a calibrated display monitor. Think: scientific instrument crossed with a darkroom enlarger. Controls are laid out like hardware sliders on a mixing board.

---

## Tech Stack

- **Single HTML file** — no build tools, no dependencies, no framework
- Vanilla HTML5 + CSS3 + JavaScript (ES6+)
- Google Fonts: `Space Mono` (monospaced, instrument feel) + `DM Serif Display` (for sample text display)
- No external libraries required

---

## File Structure

```
index.html   ← entire application lives here
```

---

## Visual Design

### Color Palette (UI chrome, not the demo area)

| Token              | Value       | Use                             |
|--------------------|-------------|----------------------------------|
| `--bg-deep`        | `#0d0f0e`   | Outermost page background        |
| `--panel`          | `#141918`   | Control panel surfaces           |
| `--panel-raised`   | `#1c2220`   | Inset / raised panel variants    |
| `--border`         | `#2a3530`   | Subtle panel borders             |
| `--phosphor`       | `#39ff88`   | Primary accent (phosphor green)  |
| `--phosphor-dim`   | `#1a7a44`   | Dimmed accent                    |
| `--amber`          | `#ffb347`   | Warning / fail state             |
| `--text-primary`   | `#c8d8d0`   | Main UI text                     |
| `--text-muted`     | `#5a7065`   | Labels, secondary UI text        |
| `--red-fail`       | `#ff4455`   | WCAG fail indicator              |
| `--green-pass`     | `#39ff88`   | WCAG pass indicator              |

### Typography

- **UI Labels & Controls**: `Space Mono`, 11–13px, letter-spacing: 0.08em, uppercase
- **Readout Numbers**: `Space Mono`, bold, phosphor-green color
- **Sample Text Area**: `DM Serif Display`, size controlled by user slider

### Layout

```
┌──────────────────────────────────────────────────────────────┐
│  ░░ READABILITY EXPLORER  •  WCAG Contrast Lab  ░░           │
│  ────────────────────────────────────────────────────────     │
│                                                              │
│  ┌──────────────────────────┐  ┌───────────────────────────┐ │
│  │  BACKGROUND COLOR        │  │  TEXT COLOR               │ │
│  │  R ══════════════ [255]  │  │  R ══════════════ [  0]  │ │
│  │  G ══════════════ [255]  │  │  G ══════════════ [  0]  │ │
│  │  B ══════════════ [255]  │  │  B ══════════════ [  0]  │ │
│  │  ┌──────────────────┐   │  │  ┌──────────────────┐    │ │
│  │  │   COLOR SWATCH   │   │  │  │   COLOR SWATCH   │    │ │
│  │  └──────────────────┘   │  │  └──────────────────┘    │ │
│  └──────────────────────────┘  └───────────────────────────┘ │
│                                                              │
│  ┌─────────────────────┐   ┌──────────────────────────────┐  │
│  │  FONT SIZE          │   │  CONTRAST RATIO              │  │
│  │  ══════════ [16]px  │   │  ┌──────────────────────┐   │  │
│  └─────────────────────┘   │  │   4.52 : 1           │   │  │
│                             │  │  ██ AA PASS  ░░ AAA  │   │  │
│                             │  └──────────────────────┘   │  │
│                             │  BG LUM:  0.9505            │  │
│                             │  FG LUM:  0.0000            │  │
│                             └──────────────────────────────┘  │
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  PRESET COMBINATIONS                                    │ │
│  │  [Black on White] [White on Black] [Red Flag] ...       │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
│  ╔═════════════════════════════════════════════════════════╗ │
│  ║  T E X T   D I S P L A Y   A R E A                     ║ │
│  ║                                                         ║ │
│  ║  The quick brown fox jumps over the lazy dog.           ║ │
│  ║  Pack my box with five dozen liquor jugs.               ║ │
│  ║  Sphinx of black quartz, judge my vow.                  ║ │
│  ║                                                         ║ │
│  ║  Aa Bb Cc Dd Ee Ff Gg 0123456789 !@#$%                  ║ │
│  ╚═════════════════════════════════════════════════════════╝ │
└──────────────────────────────────────────────────────────────┘
```

---

## Component Specifications

### 1. Header / Title Bar

- App name: **READABILITY EXPLORER** in `Space Mono` uppercase, phosphor-green, with a subtle text-shadow glow (`0 0 12px #39ff88aa`)
- Subtitle: `WCAG 2.1 · Contrast Analysis Lab` in muted text
- A decorative animated scanline or blinking cursor element to reinforce the terminal aesthetic
- A thin phosphor-green top border stripe (4px)

---

### 2. Background Color Control Panel

**Container**: Labeled `BACKGROUND COLOR`, panel-raised surface with inset border.

**Three slider rows — R, G, B:**

Each row contains:
- A 3-character channel label (`RED`, `GRN`, `BLU`) in muted monospace
- A color-tinted range slider:
  - R slider: thumb and track fill tinted red
  - G slider: thumb and track fill tinted green
  - B slider: thumb and track fill tinted blue
- An integer input field:
  - 3 characters wide, monospaced
  - Range: 0–255
  - Displays current value
  - Editable — typing a value updates the slider
- Range `[0, 255]`, step `1`

**Color Swatch:**
- A rectangle (~100% width, ~60px tall) below the sliders showing the current background color in real time
- Rounded corners, subtle inset shadow
- Hex code displayed inside the swatch in small text, auto-contrasted (white or black text chosen automatically for readability)

---

### 3. Text Color Control Panel

Identical structure to the Background Color panel, but:
- Label: `TEXT COLOR`
- Swatch shows the selected text color

---

### 4. Font Size Control Panel

**Container**: Labeled `FONT SIZE`, narrower panel.

Contains:
- A single horizontal slider, range `8–96`, step `1`, default `18`
- An integer input field displaying the current font size in px
- A small label below: `px`

The slider track uses the phosphor accent gradient.

---

### 5. Contrast Ratio Display

**Container**: Labeled `CONTRAST ANALYSIS`, same height as Font Size panel.

Elements:
- **Ratio readout**: Large monospaced display of the ratio, e.g. `4.52 : 1`
  - Font: `Space Mono`, 28–32px, phosphor-green glow when passing, amber/red when failing
- **WCAG Badge Row**: Two pill badges:
  - `AA NORMAL` — filled green if ratio ≥ 4.5, else red outline
  - `AA LARGE` — filled green if ratio ≥ 3.0, else red outline
  - `AAA NORMAL` — filled green if ratio ≥ 7.0, else red outline
  - `AAA LARGE` — filled green if ratio ≥ 4.5, else red outline
  - ("Large text" = 18pt/24px+ normal weight, or 14pt/~19px+ bold)
- **Luminance Readouts**:
  - `BG LUM` label + value, e.g. `0.9505`, in muted monospace
  - `FG LUM` label + value, e.g. `0.0000`, in muted monospace
  - Small annotation: `(relative luminance per WCAG 2.1)`

---

### 6. Preset Combinations

**Container**: Full-width panel below the controls, labeled `PRESET COMBINATIONS`.

A horizontal scrollable row of clickable preset tiles. Each tile:
- Shows a mini preview (small colored rectangle, ~60×40px) with a few sample characters rendered in the combination's colors
- Below the preview: a short label in `Space Mono`
- On click: instantly sets all six RGB values (BG R/G/B and FG R/G/B) and updates everything

**Required presets (minimum):**

| Label | BG Color | FG Color | Category |
|---|---|---|---|
| Classic | `#ffffff` | `#000000` | ✅ High Contrast |
| Inverted | `#000000` | `#ffffff` | ✅ High Contrast |
| Newspaper | `#f5f0e8` | `#1a1a1a` | ✅ High Contrast |
| Navy + Gold | `#1b2a4a` | `#ffd700` | ✅ High Contrast |
| Terminal | `#0d1117` | `#39ff88` | ✅ High Contrast |
| Solarized | `#fdf6e3` | `#657b83` | 🟡 Medium Contrast |
| Ocean | `#0077b6` | `#90e0ef` | 🟡 Medium Contrast |
| Muted Moss | `#8a9a7a` | `#6b7a5c` | ❌ Low Contrast / Fail |
| Gray Haze | `#aaaaaa` | `#999999` | ❌ Low Contrast / Fail |
| Red Alert | `#ff0000` | `#ff6666` | ❌ Problematic / Fail |
| Lavender | `#e6e6fa` | `#d8bfd8` | ❌ Problematic / Fail |
| Parchment | `#f5deb3` | `#daa520` | ❌ Problematic / Fail |

Add visual category indicators (small colored dot or icon) on each tile to show pass/fail at a glance.

---

### 7. Text Display Area

**Container**: Prominent full-width panel at the bottom, labeled `DISPLAY VIEWPORT`.

- Double-border frame effect (outer dark border, inner 1px panel border) to evoke a monitor bezel
- Corner screws or bracket decorations (ASCII/CSS art) for the oscilloscope aesthetic
- Background: set by user's BG color selection
- Text color: set by user's FG color selection
- Font size: set by user's font size slider
- Font family: `DM Serif Display`

**Sample text blocks (all rendered in chosen colors/size):**

```
The quick brown fox jumps over the lazy dog.
Pack my box with five dozen liquor jugs.
Sphinx of black quartz, judge my vow.

How vexingly quick daft zebras jump!

Aa Bb Cc Dd Ee Ff Gg Hh Ii Jj Kk Ll Mm
Nn Oo Pp Qq Rr Ss Tt Uu Vv Ww Xx Yy Zz

0 1 2 3 4 5 6 7 8 9
! @ # $ % ^ & * ( ) - + = [ ] { } | ; : , . < > ? /
```

Text should be selectable. Padding inside the display area should be generous (24–32px).

---

## Behavior / Interactivity

### Slider ↔ Input Synchronization

- Each slider and its corresponding integer input field are **two-way synchronized**:
  - Moving the slider immediately updates the input field value
  - Typing in the input field (on `input` event) immediately updates the slider
  - Input field validates range: clamp to [0, 255] for color fields, [8, 96] for font size
  - Invalid / empty input: revert to current valid value on `blur`

### Real-Time Color & Size Updates

- Any change to a color slider or input → immediately update:
  1. The relevant color swatch
  2. The text display area background or text color
  3. Recompute contrast ratio and luminance values
- Any change to the font size → immediately update the font size in the text display area

### Contrast Ratio Calculation (WCAG 2.1)

```
Step 1: For each channel value V in {R, G, B}:
  sRGB = V / 255
  if sRGB <= 0.04045:
    linear = sRGB / 12.92
  else:
    linear = ((sRGB + 0.055) / 1.055) ^ 2.4

Step 2: Relative luminance L:
  L = 0.2126 * R_linear + 0.7152 * G_linear + 0.0722 * B_linear

Step 3: Contrast ratio:
  L1 = max(L_bg, L_fg)
  L2 = min(L_bg, L_fg)
  ratio = (L1 + 0.05) / (L2 + 0.05)

Step 4: Display as "X.XX : 1" (2 decimal places)
```

Luminance values displayed to 4 decimal places.

### Preset Loading

Clicking a preset tile:
1. Instantly sets BG R, G, B sliders and inputs to preset background color values
2. Instantly sets FG R, G, B sliders and inputs to preset text color values
3. Triggers the full update pipeline (swatches, display area, contrast recalculation)

---

## Animations & Polish

- **Slider thumb**: Custom styled with a circular phosphor-green glowing thumb on `:hover`/`:active`
- **Contrast ratio number**: CSS transition on color (green ↔ amber ↔ red) as ratio changes
- **Preset tile hover**: Brief scale-up transform + border glow
- **WCAG badges**: Smooth background-color transition when switching pass/fail states
- **Scanline overlay**: Very subtle repeating-linear-gradient scanline effect on the text display viewport (thin semi-transparent dark lines) to reinforce the screen-within-a-screen feel — but make it dismissable or very subtle so it doesn't obscure content
- **Header**: Blinking block cursor `█` character after the title, using CSS animation (`opacity` keyframes at 1s interval)
- **Panel borders**: Left border accent in phosphor-green on active/focused control panels

---

## Accessibility of the Tool Itself

- All controls have `<label>` elements with `for` attributes
- Slider `aria-label` attributes describe the channel and range
- Contrast ratio region has `aria-live="polite"` so screen readers announce changes
- Preset tiles are `<button>` elements, not `<div>`s
- Focus styles visible (phosphor-green outline)

---

## Implementation Notes

### HTML Structure Sketch

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Readability Explorer</title>
  <link href="https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=DM+Serif+Display&display=swap" rel="stylesheet" />
  <style>
    /* All styles here — CSS variables, panel layouts, slider overrides, animations */
  </style>
</head>
<body>
  <header>...</header>
  <main>
    <div class="controls-grid">
      <section class="panel" id="bg-panel">...</section>
      <section class="panel" id="fg-panel">...</section>
      <section class="panel" id="size-panel">...</section>
      <section class="panel" id="contrast-panel">...</section>
    </div>
    <section class="panel" id="presets-panel">...</section>
    <section id="display-viewport">...</section>
  </main>
  <script>
    /* All JavaScript here */
  </script>
</body>
</html>
```

### JavaScript Architecture

Keep everything in a single script block. Suggested structure:

```js
// --- State ---
const state = {
  bg: { r: 255, g: 255, b: 255 },
  fg: { r: 0,   g: 0,   b: 0   },
  fontSize: 18
};

// --- DOM references ---
// (gathered once on DOMContentLoaded)

// --- Core functions ---
function sRGBtoLinear(v) { ... }       // channel linearization
function relativeLuminance(r, g, b) { ... }  // WCAG luminance
function contrastRatio(L1, L2) { ... }  // WCAG ratio
function wcagBadgeStatus(ratio, fontSize, bold) { ... }  // AA/AAA check

// --- Update pipeline ---
function updateAll() {
  updateSwatches();
  updateDisplayViewport();
  updateContrastPanel();
}

// --- Event wiring ---
// For each channel slider and input: bidirectional sync
// For preset buttons: set state then call updateAll()

// --- Init ---
updateAll();
```

---

## Edge Cases

| Scenario | Behavior |
|---|---|
| BG color = FG color (ratio = 1.00:1) | All WCAG badges show fail state; ratio displays `1.00 : 1` |
| User types value > 255 | Clamp to 255, update slider |
| User types value < 0 | Clamp to 0, update slider |
| User types non-numeric | Ignore / revert on blur |
| Very small font (e.g. 8px) | AA Large / AAA Large badge criteria don't apply; display accordingly |

---

## Deliverable Checklist

- [ ] Single `index.html` file, opens in any modern browser without a server
- [ ] All 6 color channel sliders with synchronized inputs
- [ ] Font size slider with synchronized input
- [ ] Live-updating text display viewport
- [ ] Contrast ratio calculated per WCAG 2.1 and displayed as `X.XX : 1`
- [ ] Relative luminance values for BG and FG displayed
- [ ] WCAG AA / AAA pass/fail badges (normal and large text thresholds)
- [ ] ≥ 12 presets covering high contrast, medium contrast, fail cases, and themed combos
- [ ] Phosphor-green terminal aesthetic, `Space Mono` UI font, `DM Serif Display` sample text
- [ ] Animations: blinking cursor, slider glow, badge color transitions, preset hover
- [ ] Responsive layout (panels stack gracefully on narrower screens)
- [ ] Accessible markup (labels, aria attributes, keyboard navigable)
