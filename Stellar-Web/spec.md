# Stellar-Web Specification

## Overview
An interactive particle network visualization demonstrating emergent network structures through proximity-based connections. Nodes drift through 3D space and connect when within a configurable connectivity radius.

## Features
- **Particle Network**: Nodes connected by edges when within proximity
- **3D Depth Effect**: Z-position affects node size and opacity
- **Alternating Colors**: Nodes alternate between yellow and blue
- **Panda Obstacle**: Giant panda emoji in center that nodes bounce off
- **Network Statistics**: Real-time display of edges, connections, and density
- **Interactive Controls**: Adjust all parameters in real-time

## Controls Panel (Bottom Right)
| Control | Range | Default | Description |
|---------|-------|---------|-------------|
| Nodes | 20-300 | 100 | Number of particles |
| Connectivity Radius | 50-300 | 150 | Distance threshold for edges |
| Node Size | 1-12 | 6 | Base node size |
| Edge Opacity | 0.1-1.0 | 0.5 | Transparency of connection lines |
| Speed | 0.1-3.0 | 1.0 | Node movement speed |
| Depth Effect | 0-1 | 0.5 | Influence of Z-position on appearance |

## Network Statistics
- **Total Edges**: Count of current connections
- **Avg Connections**: Mean edges per node
- **Network Density**: Percentage of possible connections active

## Visual Design
- **Background**: Dark blue (`#0a0a1a`)
- **Yellow Nodes**: Golden glow (`rgba(255, 230, 100)`)
- **Blue Nodes**: Cyan glow (`rgba(100, 180, 255)`)
- **Edges**: Gold to yellow gradient (`hue 45-60`)
- **Control Panel**: Semi-transparent with yellow/gold theme

## Technical Details
- **File**: `index.html`
- **Framework**: Vanilla HTML/CSS/JavaScript
- **Rendering**: HTML5 Canvas 2D
- **Main Class**: `Node`

## Physics
- Nodes drift with random velocity in X, Y, Z
- Screen wrapping on X/Y boundaries
- Z-position bounces between 0 and 1
- Collision detection with panda (circular obstacle)
- Reflection physics for bouncing off panda

## Panda Obstacle
- **Size**: 200px emoji
- **Position**: Canvas center
- **Collision Radius**: 90px (size * 0.45)

## Live URL
https://aiml-1870-2026.github.io/lightningphoenix70/Stellar-Web/
