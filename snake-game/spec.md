# Snake Game Specification

## Overview
A classic Snake game with a cowboy theme, sparkly color-switching visuals, and support for both single-player and two-player modes on one screen.

## Features
- Classic snake gameplay with wall and self-collision
- 1-Player and 2-Player modes, toggled via a button
- Snake changes color between sparkly gold and sparkly blue each time it turns
- Animated sparkle effects on each snake segment
- Brown cowboy hat on every snake's head
- Snake speeds up slightly each time it eats food
- High score saved to local storage
- Game over screen with flashing yellow/blue overlay, floating sparkle particles, and a slanted cowboy hat on the "G" in "GAME OVER"
- Winner announcement in 2-player mode

## Controls
| Control | Player | Description |
|---------|--------|-------------|
| W / A / S / D | P1 | Move up / left / down / right |
| Arrow Keys | P2 (2P mode) / P1 (1P mode) | Move up / left / down / right |
| R | Both | Restart the game |
| Mode Button | -- | Toggle between 1 Player and 2 Players |

## Visual Design
- **Background**: Light pink (`#ffccd5`) body, soft pink (`#ffe8ec`) canvas
- **Snake (yellow mode)**: Gold/yellow with animated shimmer and white sparkle dots
- **Snake (blue mode)**: Blue with animated shimmer and white sparkle dots
- **Food**: Light pink circle (`#ffb3c1`) with a soft glow
- **Cowboy hat**: Brown brim, tapered crown, dark hat band on each snake head
- **Game Over**: Light pink text, slanted cowboy hat on the "G", flashing yellow/blue overlay with floating sparkle particles
- **Grid**: Subtle pink grid lines (`#f5d0d6`)
- **UI**: Pink-themed border, button, and score text

## Technical Details
- **File**: `index.html`
- **Framework**: Vanilla JavaScript with HTML5 Canvas
- **Rendering**: 2D canvas with `requestAnimationFrame` game loop
- **Grid**: 30x30 cells (20px each) on a 600x600 canvas
- **Speed**: Starts at 100ms per tick, decreases by 3ms per food eaten, minimum 40ms
- **High Score**: Persisted via `localStorage`

## Live URL
https://aiml-1870-2026.github.io/LightningPhoenix70/snake-game/
