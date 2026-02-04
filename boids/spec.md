# Boids Simulation Specification

## Overview
A high-performance flocking simulation implementing Craig Reynolds' boids algorithm with multi-threaded physics, spatial optimization, obstacle avoidance, and real-time analytics.

## Features
- Web Worker-based physics for decoupled rendering and simulation
- Uniform grid spatial partitioning for O(n) neighbor lookups
- Transferable Float32Array data transfer between threads
- Three core steering behaviors: Separation, Alignment, Cohesion
- Pink spiked obstacles with look-ahead whisker avoidance
- Wrap-around and bounce boundary modes
- Behavior presets (Schooling, Chaotic Swarm, Tight Cluster)
- Mouse interaction (click to attract, right-click to repel)
- Real-time HUD with FPS, boid count, avg speed, avg neighbors
- Live mini-charts for neighbor density, speed variance, and flock compactness

## Controls
| Control | Type | Range | Default | Description |
|---------|------|-------|---------|-------------|
| Separation | Slider | 0 - 5 | 1.5 | Desire to maintain personal space |
| Alignment | Slider | 0 - 5 | 1.0 | Desire to point in the same direction |
| Cohesion | Slider | 0 - 5 | 1.0 | Desire to stay near center of group |
| Neighbor Radius | Slider | 20 - 200 | 60 | Detection distance for other boids |
| Max Speed | Slider | 1 - 12 | 4.0 | Maximum velocity magnitude |
| Boid Count | Slider | 10 - 800 | 200 | Total active entities |
| Boundary Mode | Toggle | Wrap / Bounce | Wrap | Edge behavior for boids |
| Add Obstacle | Button | -- | -- | Click canvas to place a spiked obstacle |
| Clear All | Button | -- | -- | Remove all obstacles |
| Pause / Resume | Button | -- | -- | Freeze or resume the simulation |
| Reset | Button | -- | -- | Re-randomize all boid positions and velocities |

## Presets
| Preset | Separation | Alignment | Cohesion | Speed | Radius |
|--------|-----------|-----------|----------|-------|--------|
| Schooling | 1.5 | 3.0 | 1.5 | 4 | 80 |
| Chaotic Swarm | 3.0 | 0.0 | 0.3 | 8 | 50 |
| Tight Cluster | 1.0 | 0.5 | 4.0 | 3 | 40 |

## Mouse Interaction
- **Click**: Attract boids toward cursor
- **Right-click**: Repel boids from cursor
- **Shift+Click**: Place a spiked obstacle

## Obstacles
- Rendered as pink spiked circles
- Boids use look-ahead whiskers (4 forward probes at 10px intervals) to detect obstacles
- Exponential repulsive force: F = k / d^2
- Placed via sidebar button or Shift+Click on canvas

## Boundaries
- **Wrap**: Boids exiting one side reappear on the opposite side
- **Bounce**: Glowing blue boundary edges apply steering force to keep boids within the viewport

## Analytics HUD
- **FPS**: Current rendering frames per second
- **Boid Count**: Total active entities
- **Avg Speed**: Mean velocity magnitude across the flock
- **Avg Neighbors**: Mean number of boids within the detection radius

## Live Charts
- **Neighbor Density**: Average neighbor count over time
- **Speed Variance**: Standard deviation of boid speeds (measures flock stability)
- **Compactness**: Standard deviation of distances from flock centroid

## Visual Design
- Dark navy background (`#0a0a1a`) with subtle trail fading
- HSL color-shifting boids with directional arrow shapes and glow
- Pink spiked obstacles (`#ff6b9d`) with inner core (`#cc4477`)
- Blue glowing edges in bounce mode (`rgba(85, 136, 255)`)
- Sidebar with dark theme, blue accent color (`#5588ff`)

## Technical Details
- **File**: `index.html`
- **Framework**: Vanilla JavaScript with HTML5 Canvas
- **Threading**: Web Worker via Blob URL for physics computation
- **Data Transfer**: Transferable Float32Array (5 floats per boid: x, y, vx, vy, hue)
- **Spatial Partitioning**: Uniform grid with cell size matching perception radius
- **Canvas**: Dynamically resizes to fill available viewport

## Live URL
https://aiml-1870-2026.github.io/LightningPhoenix70/boids/
