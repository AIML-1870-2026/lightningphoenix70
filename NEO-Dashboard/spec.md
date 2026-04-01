# NEO Dashboard — Product Specification

## Overview

A single-page, public-facing web dashboard that pulls live near-Earth object (NEO) data from NASA's JPL Close Approach Data API. The goal is to make asteroid data approachable, visually engaging, and educational for a general audience — using plain English throughout, with a threat-aware framing to communicate danger levels clearly.

---

## Tech Stack

- **Format:** Single-file (`index.html`) — plain HTML, CSS, and vanilla JavaScript
- **Data source:** `https://ssd-api.jpl.nasa.gov/cad.api`
- **Default date range:** ~1 year back to ~1 year ahead of the current date (dynamically calculated on load)
- **No build tools, no frameworks, no dependencies** beyond what can be loaded via CDN (e.g. Chart.js for charts, Leaflet.js for the map)

---

## Visual Design

- **Theme:** Dark space aesthetic — deep navy/black backgrounds, glowing accent colors (amber for warning, red for danger, green for safe), monospace or NASA-style sans-serif fonts
- **Tone:** Dramatic but informative — this is space, after all
- **Language:** Plain English throughout; all technical terms (AU, km/s, absolute magnitude, etc.) must be accompanied by a plain-English tooltip or inline explanation

---

## API Details

**Endpoint:** `https://ssd-api.jpl.nasa.gov/cad.api`

Key query parameters used:
- `date-min` / `date-max` — sets the date window (~1 year back, ~1 year forward)
- `dist-max` — filter by maximum close approach distance (in AU)
- `diameter` — request diameter estimates where available
- `vel-inf` / `vel-rel` — relative velocity fields
- `sort` — sort by date, distance, or size
- `fullname` — full object name in results

All API calls happen client-side on page load, with a loading state shown while data fetches.

---

## Tab Structure

### Tab 1 — Overview

The landing experience. Designed to orient and hook the general public immediately.

**Contents:**
- **Hero stat bar** — 4 glanceable KPI cards at the top:
  - Total NEOs in the current date window
  - Closest approach on record (within the window)
  - Fastest object (km/s, plain-English label like "faster than a bullet")
  - Next upcoming close approach with a **live countdown timer** (days, hours, minutes, seconds)
- **Danger level summary** — a simple visual breakdown of objects by threat tier (Safe / Watch / Danger), using color-coded counts and icons. Tiers defined by close approach distance:
  - 🟢 Safe: > 0.05 AU
  - 🟡 Watch: 0.01–0.05 AU
  - 🔴 Danger: < 0.01 AU (within ~1.5 million km)
- **Did You Know? panel** — a rotating card showing fun asteroid facts, refreshed every 10 seconds. Examples:
  - "The asteroid Bennu is about as tall as the Empire State Building."
  - "Most NEOs burn up in the atmosphere before reaching the ground."
  - "NASA tracks over 30,000 near-Earth asteroids."
- **Next 5 upcoming approaches** — a compact preview list with name, date, distance, and danger badge; clicking one deep-links to the Data Explorer tab filtered to that object

---

### Tab 2 — Deep Dive

Richer visualizations and educational breakdowns for users who want to explore further.

**Contents:**
- **Historical vs. Upcoming toggle** — switch between past approaches (last ~1 year) and upcoming ones (next ~1 year); all charts below respond to this toggle
- **Close Approach Distance Chart** — a line or scatter chart (Chart.js) plotting approach distance over time, with a horizontal threshold line marking the "Danger" zone. X-axis = date, Y-axis = distance in lunar distances (LD) and AU. Lunar distances used as the primary label since they're more relatable.
- **Velocity Distribution Chart** — a bar/histogram chart showing how fast objects are moving (km/s buckets), with a plain-English callout like "Most objects travel between 10–20 km/s — that's 25x faster than a rifle bullet"
- **Size Comparator** — for each object with a known diameter estimate, show a side-by-side visual comparison against a familiar landmark. Tiers:
  - < 50m → compare to a city block or football field
  - 50–200m → compare to the Eiffel Tower or Statue of Liberty
  - 200m–1km → compare to a skyscraper or cruise ship
  - > 1km → compare to a mountain
- **World Map — Flyby Trajectory Viewer** (Leaflet.js) — a dark-themed world map showing approach vectors as animated arcs. Each arc originates from a point representing the object's approach direction relative to Earth. Clicking an arc shows a popup with object name, distance, velocity, and danger level. *(Note: true orbital trajectory data is not in the CAD API; arcs will be illustrative/approximate, based on approach direction metadata where available, and clearly labeled as approximate.)*

---

### Tab 3 — Data Explorer

A full sortable, filterable data table for users who want to dig into the raw numbers.

**Contents:**
- **Filter bar** at the top with controls for:
  - Date range (pre-set to default window, but user-adjustable via date pickers)
  - Danger tier (All / Safe / Watch / Danger)
  - Minimum diameter (slider or input)
  - Max close approach distance (AU or LD toggle)
- **Sortable data table** with columns:
  - Object Name (links to NASA's JPL small body database page for that object)
  - Close Approach Date
  - Distance (shown in both LD and AU)
  - Velocity (km/s, with plain-English descriptor on hover)
  - Est. Diameter (meters, with size comparator icon)
  - Danger Badge (color-coded pill: Safe / Watch / Danger)
- **Pagination** — 25 rows per page, with total count shown
- **CSV Export button** — downloads the currently filtered dataset as a `.csv` file
- **Glossary drawer** — a collapsible panel at the bottom explaining all column terms in plain English (AU, LD, velocity types, absolute magnitude, etc.)

---

## Global Features

- **Loading state** — a full-screen animated starfield or spinner while the API call resolves, with text like "Scanning the solar neighborhood…"
- **Error state** — if the API fails, show a friendly message with a retry button
- **Responsive design** — functional on desktop and tablet; charts/table reflow gracefully. Mobile is secondary but should not be broken.
- **No login, no data storage** — fully stateless; all data fetched fresh on each page load

---

## Danger Tier Definitions (Reference)

| Tier   | Distance Threshold     | Color  | Label  |
|--------|------------------------|--------|--------|
| Safe   | > 0.05 AU (~7.5M km)  | Green  | Safe   |
| Watch  | 0.01–0.05 AU           | Amber  | Watch  |
| Danger | < 0.01 AU (~1.5M km)  | Red    | Danger |

> These thresholds are simplified for public communication and do not reflect official NASA Planetary Defense criteria (e.g. Torino Scale). A disclaimer to this effect should appear in the UI.

---

## Out of Scope (v1)

- User accounts or saved searches
- Push notifications or alerts
- True orbital mechanics / trajectory simulation
- Mobile-native app
- Data from additional APIs (e.g. NeoWs, Sentry risk tables) — potential v2 additions
