# Turning Patterns Specification

## Overview
A generative art application using dragon curve turn sequences to create intricate patterns. Features dual side-by-side simulations running simultaneously with different parameters, freehand drawing, and twinkling sparkle effects.

## Features
- Dual canvas layout with two independent simulations
- Dragon curve algorithm using binary representation for turn decisions
- Freehand drawing by clicking and dragging
- Twinkling 4-pointed star sparkle particles (toggle on every other click)
- Multiple walker support via Shift+Click
- Six pattern presets with different characteristics
- Real-time parameter adjustment via sliders
- Pink-only color palette with adjustable saturation and lightness
- Optional glow effects and edge wrapping
- HUD displaying iteration count, angle, and sparkle status

## Controls

| Control | Action |
|---------|--------|
| Click & Drag | Draw freehand lines on canvas |
| Click | Set new walker origin (every 2nd click toggles sparkles) |
| Shift+Click | Add a new walker at cursor position |
| Scroll Wheel | Adjust turn angle for that canvas |
| Spacebar | Pause/Resume both simulations |
| R Key | Reset both simulations |
| Left/Right Buttons | Select which simulation to control |

## Parameters

| Parameter | Range | Default (Dragon) | Default (Spiral) | Description |
|-----------|-------|------------------|------------------|-------------|
| Turn Angle | 1 - 180 | 90° | 91° | Angle of each turn in degrees |
| Step Length | 1 - 40 | 8 | 6 | Distance traveled per step |
| Line Width | 0.5 - 6 | 1.5 | 1.0 | Stroke thickness |
| Speed | 1 - 100 | 15 | 20 | Steps per animation frame |
| Pink Variation | 0 - 10 | 0.8 | 1.5 | Rate of hue shift within pink spectrum |
| Saturation | 0 - 100% | 70% | 80% | Color saturation |
| Lightness | 20 - 80% | 60% | 55% | Color lightness |
| Glow Effect | On/Off | Off | Off | Adds shadow blur to lines |
| Wrap Edges | On/Off | On | On | Walkers wrap around canvas edges |

## Presets

| Preset | Turn Angle | Step | Width | Speed | Description |
|--------|------------|------|-------|-------|-------------|
| Dragon | 90° | 8 | 1.5 | 15 | Classic dragon curve fractal |
| Spiral | 91° | 6 | 1.0 | 20 | Slowly spiraling pattern |
| Hexagon | 60° | 10 | 2.0 | 12 | Hexagonal tiling pattern |
| Triangle | 120° | 12 | 1.5 | 10 | Triangular structures |
| Chaos | 137° | 5 | 0.5 | 50 | Golden angle chaos pattern |
| Fine Web | 89° | 3 | 0.5 | 80 | Dense web-like structure |

## Sparkle System
- 4-pointed twinkling star particles
- White fill with pink glow (`#ffccdd`)
- Spawns along drawn lines when sparkle mode is active
- Animated twinkle effect with fade-out
- Maximum 300 sparkles per simulation to maintain performance

## Visual Design
- **Background**: White (`#ffffff`) canvas, light pink (`#fff5f7`) page
- **Sidebar**: Soft pink (`#fff0f3`) with pink accent colors
- **Lines**: Pink spectrum only (HSL hue 330-350)
- **Sparkles**: White with pink glow
- **UI Elements**: Various pink shades for text, buttons, and borders
- **Canvas Labels**: Rounded pink badges showing pattern name

## Technical Details
- **File**: `index.html`
- **Framework**: Vanilla JavaScript with HTML5 Canvas
- **Architecture**: Object-oriented with Simulation, Walker, and Sparkle classes
- **Rendering**: Dual canvas with independent requestAnimationFrame loop
- **Algorithm**: Dragon curve using binary representation (bit manipulation)
- **HUD Updates**: Throttled to 200ms for readability

## Dragon Curve Algorithm
The turn direction at each step is determined by:
1. Find the lowest set bit position in the iteration number
2. Check the bit above it
3. If set: turn right (+angle), otherwise: turn left (-angle)

```javascript
let n = iteration;
while (n > 0 && (n & 1) === 0) n >>= 1;
if (n & 1) angle += turnAngle;
else angle -= turnAngle;
```

## Live URL
https://aiml-1870-2026.github.io/LightningPhoenix70/turning-patterns/
