# Starfield-Assignment Specification

## Overview
A 3D starfield animation with perspective projection, motion trails, and falling panda emojis.

## Features
- **3D Starfield**: Stars flying toward viewer with depth perspective
- **Motion Trails**: Configurable trail length and fade effect
- **Falling Pandas**: 10 rotating panda emojis drifting down with wobble motion
- **Interactive Controls**: Real-time parameter adjustment

## Controls Panel
| Control | Range | Default | Description |
|---------|-------|---------|-------------|
| Star Count | 100-2000 | 500 | Number of stars |
| Speed | 0.5-10 | 2 | Star movement speed |
| Trail Length | 0-50 | 15 | Length of star trails |
| Trail Fade | 0.01-0.3 | 0.1 | Fade alpha for ghost effect |
| Star Size | 0.5-5 | 3 | Base star size |
| Field of View | 100-500 | 300 | Perspective FOV |

## Visual Design
- **Background**: Pure black (`#000`)
- **Stars**: Purple/lavender (`rgba(200, 150, 255)`)
- **Control Panel**: Semi-transparent dark with blur backdrop

## Technical Details
- **File**: `index.html`
- **Framework**: Vanilla HTML/CSS/JavaScript
- **Rendering**: HTML5 Canvas 2D
- **Classes**: `Star`, `Panda`

## Animation Logic
1. Stars move along Z-axis toward viewer
2. Perspective projection maps 3D to 2D screen coordinates
3. Trail points stored and drawn with decreasing opacity
4. Pandas fall with rotation and sinusoidal wobble

## Live URL
https://aiml-1870-2026.github.io/lightningphoenix70/Starfield-Assignment/
