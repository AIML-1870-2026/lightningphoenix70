# Weather Dashboard — Project Spec

## Overview

A browser-based weather dashboard built with plain HTML, CSS, and JavaScript. Users can search any city and instantly see current conditions plus a 5-day forecast. The visual style is **bright, colorful, and energetic** — bold gradients, vivid accent colors, and lively typography that shifts feel based on weather conditions.

---

## Tech Stack

| Layer | Choice |
|---|---|
| Markup | HTML5 |
| Styling | CSS3 (custom properties, flexbox, grid, animations) |
| Logic | Vanilla JavaScript (ES6+) |
| Weather Data | OpenWeatherMap API |
| Fonts | Google Fonts (display + body pairing — avoid Inter/Roboto/Arial) |

---

## API Details

- **Provider:** OpenWeatherMap
- **API Key:** `3b06274dec1590f9160c801ed296f68f`
- **Current Weather endpoint:**
  ```
  https://api.openweathermap.org/data/2.5/weather?q={city}&appid={API_KEY}&units={units}
  ```
- **5-Day Forecast endpoint:**
  ```
  https://api.openweathermap.org/data/2.5/forecast?q={city}&appid={API_KEY}&units={units}
  ```
- **Units param:** `metric` (Celsius) or `imperial` (Fahrenheit)
- **Note:** The forecast endpoint returns data in 3-hour intervals. Group by day and display the midday reading (or average) for each of the 5 days.

---

## Features

### 1. City Search
- Text input field for entering a city name
- Submit on Enter key or clicking a Search button
- Show a user-friendly error message if the city is not found (e.g. "City not found — try again")
- Retain the last searched city in `localStorage` and auto-load it on page refresh

### 2. Temperature Unit Toggle
- Toggle switch or button group: **°C / °F**
- Switching units re-fetches data (or recalculates from cached data) and updates all displayed temperatures immediately
- Default unit: **Fahrenheit (°F)**

### 3. Current Weather Card
Display the following for the searched city:
- City name + country code
- Current temperature (large, prominent)
- "Feels like" temperature
- Weather condition description (e.g. "Partly Cloudy")
- Weather condition icon (from OpenWeatherMap icon CDN: `https://openweathermap.org/img/wn/{icon}@2x.png`)
- Humidity (%)
- Wind speed (mph or km/h depending on unit)

### 4. 5-Day Forecast Strip
A horizontal row of 5 day cards, each showing:
- Day of the week (e.g. "Mon", "Tue")
- Weather icon
- High / Low temperature

---

## UI & Visual Design

### Style Direction: Vivid & Atmospheric
- **Color palette:** Saturated, high-energy colors. Use CSS variables so the palette can later shift dynamically based on weather (e.g. sunny = warm yellows/oranges, rainy = cool blues, night = deep indigos).
- **Typography:** Pick a bold, characterful display font for temperatures and headings. Pair with a clean but distinctive body font. No Inter, no Roboto, no Arial.
- **Background:** Gradient mesh or layered gradient — not a flat solid color. Should feel atmospheric.
- **Cards:** Frosted glass effect (`backdrop-filter: blur`) over the gradient background. Rounded corners, soft shadows.
- **Animations:** Subtle fade-in on load, smooth transition when new city data loads. Hover states on forecast cards.
- **Layout:** Centered, single-column on mobile. On wider screens, current weather card above, forecast strip below.

### Responsive Breakpoints
| Breakpoint | Layout |
|---|---|
| < 480px | Stacked, full-width cards |
| 480px – 768px | Slightly wider cards, same single-column |
| > 768px | Max-width container centered, forecast cards in a row |

---

## File Structure

```
weather-dashboard/
├── index.html
├── style.css
└── app.js
```

Keep everything in three flat files — no build tools, no bundler.

---

## JavaScript Architecture (`app.js`)

Organize into clear sections:

```
// 1. CONFIG — API key, base URLs, default settings
// 2. STATE  — current city, current units
// 3. API    — fetchCurrentWeather(), fetchForecast()
// 4. RENDER — renderCurrentWeather(), renderForecast(), renderError()
// 5. UI     — event listeners (search input, unit toggle)
// 6. INIT   — load from localStorage, kick off initial fetch
```

- Use `async/await` with `try/catch` for all API calls
- Keep API key in a `const` at the top of `app.js` (not hardcoded inline)
- No external JS libraries or frameworks

---

## Error States

| Scenario | Behavior |
|---|---|
| City not found (404) | Show inline error message below search bar |
| Network failure | Show "Could not reach weather service — check your connection" |
| Empty search submitted | Show "Please enter a city name" |

---

## Out of Scope (for now)

- Geolocation / auto-detect city
- Hourly forecast view
- Dynamic background theme changes based on weather condition
- Animated weather icons
- Dark mode toggle

These are planned enhancements for a future iteration.

---

## Future Enhancements (Phase 2)

- **Dynamic theming:** Background gradient and accent colors shift based on weather condition and time of day
- **Weather animations:** CSS-animated rain, clouds, or sun rays in the background
- **Geolocation:** "Use my location" button
- **Hourly breakdown:** Expand a day in the forecast to see hourly data
- **Feels-like index card:** More detailed "comfort" metrics panel
