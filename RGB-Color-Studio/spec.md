# 🎨 RGB Color Studio — Product Specification

**Version:** 1.0  
**Target Audience:** Kids (ages 6–14) and color-curious learners  
**Platform:** Web (React + Canvas/WebGL)  
**Aesthetic:** Playful, bubbly, vibrant — think crayon box meets science lab

---

## Table of Contents

1. [Overview](#overview)
2. [Global UI & Aesthetic](#global-ui--aesthetic)
3. [Layout Architecture](#layout-architecture)
4. [Feature 1: Animated Color Explorer](#feature-1-animated-color-explorer)
5. [Feature 2: Palette Generator](#feature-2-palette-generator)
6. [Tab: Color Vision Simulator](#tab-color-vision-simulator)
7. [Shared Controls & State](#shared-controls--state)
8. [Interactions & Animations](#interactions--animations)
9. [Accessibility](#accessibility)
10. [Technical Stack](#technical-stack)

---

## Overview

RGB Color Studio is a browser-based interactive app that teaches color theory through play. It presents two main side-by-side features — a live animated color mixing playground and a palette generator — plus a dedicated tab for exploring how color-deficient viewers perceive the same palettes. The app is designed to feel like a toy: tactile, rewarding, and full of "wow" moments.

---

## Global UI & Aesthetic

### Visual Language

- **Font:** Nunito or Fredoka One (rounded, friendly)
- **Background:** Deep space dark (`#0D0D1A`) with subtle animated star-field or floating orb effect — this makes colors pop
- **Panel containers:** Rounded rectangles (`border-radius: 24px`) with a soft neon glow border that matches the currently selected color
- **Color scheme of chrome:** Desaturated purples/navy so that user-chosen colors always feel like the "loudest" thing on screen
- **Buttons:** Chunky, pill-shaped, with a satisfying "boing" scale animation on press (`transform: scale(0.92)` then spring back)
- **Icons:** Bold, outlined icon style (e.g., Phosphor Icons)
- **Micro-copy:** Casual, exclamatory — "Mix it up!", "Whoa, look at that!", "Save your fave! 💾"

### Playful Mode (Always On)

The app ships with one mode: **Playful Mode**. This means:

- Every color selection triggers a burst particle effect at the cursor/touch point
- Background elements (stars, blobs) subtly shift hue toward the selected color over ~2 seconds
- Sound design (optional, toggled via 🔊 button): soft pops, whooshes, and chimes on interactions
- A **mascot** — a small floating paint blob character named **"Blobby"** — lives in the bottom-right corner. Blobby reacts to color choices with a speech bubble ("Ooh, that's a warm one! 🔥" or "So cool and icy! ❄️") and blinks/wiggles on interactions.

---

## Layout Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│  🎨 RGB Color Studio         [🔊] [?]              Blobby →  (ᵔ ᴥ ᵔ) │
│  ─────────────────────────────────────────────────────────────────── │
│                                                                      │
│  [ Color Explorer ]        [ Palette Generator ]                     │
│  ┌─────────────────────┐  ┌──────────────────────────────────────┐   │
│  │                     │  │                                      │   │
│  │  ANIMATED CANVAS    │  │  HARMONY WHEEL + PALETTE CARDS       │   │
│  │  (spotlight/beams/  │  │                                      │   │
│  │   particles)        │  │                                      │   │
│  │                     │  │                                      │   │
│  │  ─────────────────  │  │  ─────────────────────────────────   │   │
│  │  RGB SLIDERS        │  │  Harmony Type Selector               │   │
│  │  R [■■■■░░░░] 180   │  │  [Complementary] [Analogous] [...]   │   │
│  │  G [■░░░░░░░]  40   │  │                                      │   │
│  │  B [■■■■■■░░] 200   │  │                                      │   │
│  └─────────────────────┘  └──────────────────────────────────────┘   │
│                                                                      │
│  ════════════════════════════════════════════════════════════════    │
│  [ 🌈 My Saved Palettes ]   [ 👁️ Color Vision Simulator ]            │
│  ─────────────────────────────────────────────────────────────────── │
│  (Tab content area)                                                  │
└──────────────────────────────────────────────────────────────────────┘
```

### Panel Structure

| Zone | Description |
|---|---|
| **App Header** | Logo, sound toggle, help button, Blobby mascot |
| **Left Panel** | Animated Color Explorer (canvas + RGB controls) |
| **Right Panel** | Palette Generator (harmony wheel + palette display) |
| **Bottom Tab Bar** | Tabs: "My Saved Palettes" and "Color Vision Simulator" |
| **Bottom Tab Content** | Expands below the two main panels; collapsible |

The two main panels are equal-width (roughly 50/50) on desktop. On tablet, they stack vertically. The bottom tab section is collapsible with a drag handle.

---

## Feature 1: Animated Color Explorer

**Panel Title:** "Mix Your Color! 🎨"

### 1.1 Canvas Animation Area

A full-bleed canvas (approximately `500×320px` on desktop) at the top of the left panel. The canvas renders one of three interactive **lighting effects** that the user can switch between. All effects react in real time to the current RGB color.

#### Effect A: Spotlight
- A soft circular gradient light (radial gradient from the selected color to transparent) on a dark background
- The spotlight follows the mouse/touch in real time
- When idle, the spotlight drifts slowly around the canvas in a looping figure-eight path
- Rim lighting: a thin halo at the edge of the spotlight shifts complementarily

#### Effect B: Color Beams
- Three to five diagonal light beams shoot across the canvas
- Each beam is tinted: the center beam uses the selected color, flanking beams use slightly rotated hues (±15°, ±30° in HSL)
- Beams pulse gently (opacity oscillates between 0.6–1.0 on a sine wave, different phase per beam)
- Clicking the canvas spawns a temporary "flash" beam from that point

#### Effect C: Particle Burst
- Hundreds of small circular particles drift upward from the bottom of the canvas
- Each particle's color is a slight variation of the selected RGB (random ±20 per channel)
- On mouse click/tap, a burst of 40 particles explodes outward from the cursor
- Particles fade out with easing over ~2 seconds

#### Effect Switcher
Three icon buttons above the canvas: `☀️ Spotlight`, `✨ Beams`, `🫧 Particles`. Active effect is highlighted with a glow border. Switching effects has a brief crossfade (~300ms).

#### Canvas Interactivity
- All effects respond to mouse move and touch move
- Click/tap triggers a "reaction" specific to each effect (spotlight flare, beam flash, particle burst)
- A subtle color name label floats in the top-left corner of the canvas (e.g., "Medium Purple", "Coral Red") — derived from nearest CSS named color or a fun descriptive name lookup table

### 1.2 RGB Sliders

Below the canvas, three clearly labeled sliders for Red, Green, and Blue channels (0–255).

**Slider Design:**
- Track is a gradient from black to the full channel color (e.g., Red slider track: `#000000` → `#FF0000`)
- Thumb is a large, round, draggable knob (28px diameter) with a drop shadow
- Current numeric value displayed to the right, editable by clicking to type
- Each slider label is colored in its own channel color (R label is red, G is green, B is blue)

**Hex Input:**
- A hex code input field (e.g., `#B428C8`) sits below the sliders
- Editing the hex updates all three sliders simultaneously with a satisfying animation
- A small copy-to-clipboard icon button sits beside it

**Color Preview Swatch:**
- A large rounded rectangle swatch (`120×60px`) shows the current mixed color
- On change, the swatch briefly "bounces" (scale keyframe animation)

**Random Color Button:**
- A `🎲 Surprise Me!` button randomizes all three channels with a "slot machine" animation (sliders rapidly cycle before landing on new values over ~600ms)

---

## Feature 2: Palette Generator

**Panel Title:** "Build a Palette! 🌈"

### 2.1 Harmony Type Selector

A row of pill-shaped toggle buttons at the top of the right panel. Only one is active at a time. Options:

| Button | Harmony Type | Colors Generated |
|---|---|---|
| Complementary | 180° opposite on wheel | 2 colors + base |
| Analogous | ±30° neighbors | 4 colors + base |
| Triadic | 3 equidistant (120°) | 3 colors |
| Split-Comp | ±150° from base | 3 colors |
| Tetradic | 4 equidistant (90°) | 4 colors |
| Monochromatic | Same hue, varied lightness | 5 shades |

Active button glows with a neon border in the base color.

### 2.2 Harmony Wheel

A circular color wheel (`240×240px`, generated via canvas) displayed in the center of the right panel.

- The selected base color is marked by a large dot with a white ring
- Generated harmony colors are marked by smaller colored dots connected to the base by dotted lines
- Clicking anywhere on the wheel sets the base color (also updates RGB sliders in the left panel)
- The wheel slowly rotates its saturation gradient on idle (very subtle, ~0.5 RPM)
- Dots for harmony colors "travel" to their new positions with a spring animation when harmony type changes

### 2.3 Palette Display Cards

Below the wheel, the generated palette is shown as a horizontal row of **Color Cards**.

#### Card Design

Each card is a rounded rectangle (`80×140px`) containing:
- **Top ~60%:** Solid fill in the card's color
- **Bottom ~40%:** Dark panel with:
  - Hex code (e.g., `#4A90D9`) in monospace font
  - RGB values in small text (`rgb(74, 144, 217)`)
  - A small ❤️ / bookmark icon to save to "My Palettes"
  - A copy icon

#### Card Animations
- Cards slide in from below (staggered, 80ms delay per card) when palette regenerates
- Hovering a card: it grows slightly (`scale(1.06)`) and displays the color's "personality label" in a tooltip bubble — e.g., "Ocean Breeze 🌊", "Forest Canopy 🌿", "Sunrise Glow 🌅" (mapped from hue/saturation/lightness ranges)
- Cards can be dragged to reorder within the palette (drag-and-drop)

#### Palette Name Generator
Above the cards, a fun auto-generated palette name is displayed in large playful font (e.g., "Cosmic Berry Splash", "Meadow at Noon"). A 🔄 button regenerates just the name. Names are constructed from hue-based adjective/noun lookup tables.

### 2.4 Export Options

Below the palette cards:
- `📋 Copy All Hex Codes` — copies comma-separated hex values
- `🖼️ Download as PNG` — generates a horizontal swatch image (800×200px) with hex labels
- `💾 Save Palette` — adds to "My Saved Palettes" tab

---

## Tab: Color Vision Simulator

**Tab Label:** `👁️ See Through Their Eyes`

This tab expands in the bottom content area and shows how the currently generated palette appears to people with three types of color vision deficiency, plus normal vision.

### Layout

A 2×2 grid of **Simulation Panels**, each showing the full current palette:

```
┌─────────────────────────┬─────────────────────────┐
│  👁️ Normal Vision        │  🔴 Protanopia           │
│  [swatch] [swatch] ...  │  [swatch] [swatch] ...  │
│  "How most people see it"│  "Red-blind: reds appear │
│                         │   dark or missing"       │
├─────────────────────────┼─────────────────────────┤
│  🟢 Deuteranopia         │  🔵 Tritanopia            │
│  [swatch] [swatch] ...  │  [swatch] [swatch] ...  │
│  "Green-blind: greens   │  "Blue-blind: blues and  │
│   and reds merge"       │   yellows look similar"  │
└─────────────────────────┴─────────────────────────┘
```

### Simulation Panels

Each panel contains:
- A **header badge** with the condition name and a short kid-friendly description
- The palette rendered as color swatches, transformed through the appropriate color matrix simulation
- A small "What's happening?" expandable section — a 1–2 sentence plain-language explanation (e.g., "People with protanopia have trouble seeing red light, so bright reds look very dark or brownish.")
- An **"Is it distinct enough?"** indicator: a simple ✅/⚠️/❌ badge that checks whether the simulated palette colors are still visually distinct from one another (based on ΔE color distance thresholds)
  - ✅ Great contrast even in this mode
  - ⚠️ Some colors may look similar
  - ❌ Colors are hard to tell apart — consider revising

### Color Transformation Matrices

Simulations use established Brettel/Viénot matrices applied per-pixel on a canvas offscreen:

- **Protanopia:** Reduced red cone response
- **Deuteranopia:** Reduced green cone response  
- **Tritanopia:** Reduced blue cone response

The transformation is computed client-side using Canvas 2D `getImageData` / `putImageData` on the palette swatch colors.

### Educational Callout

At the bottom of the tab, a cheerful callout box explains:

> "About 1 in 12 boys and 1 in 200 girls have some form of color vision difference. Designing with everyone in mind makes your palettes more beautiful for all! 🌍"

A **"Make It Accessible" wand button** (✨) auto-adjusts the current palette's lightness values to maximize contrast across all four simulated views, then updates both the Palette Generator and the Explorer simultaneously.

---

## Shared Controls & State

### Global Color State

The app maintains a single source of truth for the **base color** (`rgb(R, G, B)`):
- Changing RGB sliders in the Explorer updates the base color
- Clicking the harmony wheel in the Generator updates the base color
- Both panels react simultaneously to any base color change

### My Saved Palettes Tab

- **Tab Label:** `💜 My Palettes`
- Palettes saved via the ❤️ or 💾 buttons appear as compact rows
- Each row shows the palette name, a mini swatch strip, and a trash icon to delete
- Clicking a saved palette loads it back into both panels
- Palettes are persisted in `localStorage`
- Maximum 20 saved palettes; if exceeded, oldest is nudged out with a gentle warning toast

### Toast Notifications

Lightweight toasts appear bottom-center:
- "Palette saved! 💾" (green)
- "Copied to clipboard! 📋" (blue)
- "Palette full — oldest removed 🗑️" (amber)

---

## Interactions & Animations

| Interaction | Animation |
|---|---|
| Slider drag | Canvas effect updates at 60fps; swatch bounces on release |
| Hex input confirm | Sliders animate to new positions (spring easing) |
| Harmony type switch | Palette cards slide out left, new cards slide in from right; wheel dots travel |
| Effect switcher | Canvas crossfades between effects (300ms opacity blend) |
| Card hover | Scale up, personality tooltip floats in |
| Save palette | Card briefly turns gold with ✨ sparkle burst |
| "Surprise Me!" | Slot-machine slider animation (600ms), burst particles on canvas |
| Tab switch | Tab content slides up/down from bottom |
| Blobby mascot | Wiggles on any color change; speech bubble appears for 2s |

---

## Accessibility

- All interactive elements have clear focus rings (2px solid white offset)
- Color information is never conveyed by color alone (hex/RGB values always shown)
- All sliders have ARIA labels (`aria-label="Red channel, 0 to 255"`)
- Canvas animations respect `prefers-reduced-motion`: if enabled, all animations are disabled and canvas shows a static preview instead
- Blobby speech bubbles are announced via `aria-live="polite"`
- Sound is off by default; the toggle remembers preference in localStorage
- Minimum touch target size: 44×44px

---

## Technical Stack

| Concern | Recommended Technology |
|---|---|
| Framework | React 18 + TypeScript |
| Canvas Rendering | HTML5 Canvas 2D API (effects), requestAnimationFrame loop |
| Color Math | Custom utilities (RGB↔HSL↔HSV) + chroma.js for ΔE |
| Color Deficiency Simulation | Canvas `getImageData` + Brettel matrix transforms |
| Animations | Framer Motion (UI) + manual rAF (canvas) |
| State Management | Zustand (lightweight, single color store) |
| Persistence | Browser `localStorage` |
| Icons | Phosphor Icons (React) |
| Styling | Tailwind CSS + CSS custom properties for dynamic color theming |
| Sound (optional) | Tone.js or Web Audio API |
| Build Tool | Vite |

---

## File Structure (Suggested)

```
src/
├── components/
│   ├── layout/
│   │   ├── AppHeader.tsx
│   │   ├── BottomTabBar.tsx
│   │   └── Blobby.tsx              # Mascot component
│   ├── explorer/
│   │   ├── ColorExplorerPanel.tsx
│   │   ├── AnimatedCanvas.tsx
│   │   ├── effects/
│   │   │   ├── SpotlightEffect.ts
│   │   │   ├── BeamsEffect.ts
│   │   │   └── ParticlesEffect.ts
│   │   └── RGBSliders.tsx
│   ├── palette/
│   │   ├── PaletteGeneratorPanel.tsx
│   │   ├── HarmonyWheel.tsx
│   │   ├── PaletteCards.tsx
│   │   └── ColorCard.tsx
│   ├── simulator/
│   │   ├── ColorVisionSimulator.tsx
│   │   ├── SimulationPanel.tsx
│   │   └── colorBlindnessMatrices.ts
│   └── shared/
│       ├── Toast.tsx
│       └── SavedPalettes.tsx
├── store/
│   └── colorStore.ts               # Zustand store
├── utils/
│   ├── colorMath.ts                # RGB/HSL conversions, harmony calculations
│   ├── colorNames.ts               # Hue → "personality label" lookup
│   └── paletteNameGen.ts           # Auto palette name generator
└── App.tsx
```

---

## Open Questions / Future Considerations

- Should the canvas effects support WebGL for better performance on low-end devices?
- Should palette export include Figma/Adobe Swatch formats (.ase)?
- Consider a "Color Quiz" minigame where Blobby asks the player to mix a target color
- Consider a "Print My Palette" option that generates a printable coloring-sheet style page

---

*Spec authored for RGB Color Studio v1.0. Ready for handoff to VS Code / development.*
