# Science Experiment Generator — Project Specification

## Overview

A dynamic, single-page web application that allows users to generate grade-appropriate science experiments based on a list of available materials. The app sends user inputs to an OpenAI model and renders the free-form response as formatted HTML. Previously generated experiments are saved in-memory and displayed in a history panel for the duration of the session.

---

## Deployment Target

- **Single-file deployment:** The entire application is contained in one `index.html` file.
- Compatible with static hosting (e.g., GitHub Pages).
- No build step, no bundler, no backend server required.

---

## User Inputs

### 1. Grade Level (Dropdown)
- A `<select>` dropdown allowing the user to choose the target grade level.
- Options:
  - Kindergarten
  - 1st Grade
  - 2nd Grade
  - 3rd Grade
  - 4th Grade
  - 5th Grade
  - 6th Grade
  - 7th Grade
  - 8th Grade
  - 9th Grade
  - 10th Grade
  - 11th Grade
  - 12th Grade

### 2. Available Supplies (Text Input)
- A multi-line `<textarea>` where the user enters a comma-separated or line-separated list of materials they have on hand or can easily acquire.
- Placeholder text should guide the user (e.g., `"e.g., baking soda, vinegar, balloons, string..."`).

### 3. OpenAI Model Selection
- A `<select>` dropdown listing available OpenAI models.
- Suggested default models to include (update as needed):
  - `gpt-4o`
  - `gpt-4o-mini`
  - `gpt-4-turbo`
  - `gpt-3.5-turbo`
- The selected model is used in the API request.

---

## LLM Integration

### Provider
- **OpenAI only.** No Anthropic or Google provider switching.
- Uses the OpenAI Chat Completions API endpoint:
  `https://api.openai.com/v1/chat/completions`

### API Key Handling
- The API key is loaded from a `.env` file at runtime using the same in-memory-only pattern as the Reference Implementation (see below).
- The key is never stored, logged, persisted to localStorage, or embedded in the HTML.
- If no API key is found, display a clear error message to the user.

### Prompt Construction
The system prompt should instruct the model to:
- Generate a science experiment appropriate for the specified grade level.
- Use only the materials listed by the user.
- Automatically assign a **difficulty rating** (e.g., Easy, Medium, Hard) based on the complexity of the experiment relative to the grade level.
- Structure the response with clear sections: experiment title, difficulty rating, objective, materials needed, step-by-step instructions, and expected results.
- Format the response using Markdown (headings, bold text, numbered lists, etc.).

The user message should include:
- The selected grade level.
- The list of available supplies.

### Response Handling
- Responses are **unstructured free-form text** — no JSON parsing required.
- The model's Markdown-formatted response is rendered as properly formatted HTML using a Markdown-to-HTML library (e.g., [marked.js](https://cdn.jsdelivr.net/npm/marked/marked.min.js) via CDN).
- Raw Markdown text must never be displayed directly to the user.

---

## Difficulty Rating

- The difficulty rating is **assigned automatically by the LLM** as part of its response.
- The prompt instructs the model to include the rating prominently (e.g., as a labeled section near the top of the response).
- Suggested scale: `Easy`, `Medium`, `Hard` — though the model may elaborate if appropriate.
- The difficulty rating should be visually highlighted in the rendered output (e.g., styled badge or bold label).

---

## Experiment History (In-Memory)

- Each successfully generated experiment is saved to an in-memory array (JavaScript variable).
- History is **cleared on page refresh** — nothing is persisted to localStorage or any external storage.
- A history panel is displayed on the page showing previously generated experiments for the current session.
- Each history entry displays:
  - Experiment title (extracted from the response or auto-labeled by index)
  - Grade level used
  - Difficulty rating
  - A button or toggle to expand and view the full formatted experiment
- The most recent experiment appears at the top of the history list.
- A "Clear History" button allows the user to wipe the in-memory list and reset the panel.

---

## Output Display

- The generated experiment is rendered in a dedicated output panel below the input form.
- Markdown is converted to HTML and injected into the panel using `innerHTML` (after sanitization or trusted content — no user-supplied HTML is rendered).
- The difficulty rating should be visually distinguished (e.g., colored badge: green = Easy, yellow = Medium, red = Hard).

---

## Error Handling

- If the API key is missing or invalid, display a user-friendly error message.
- If the API request fails (network error, rate limit, etc.), display the error reason without crashing the app.
- Follow the error handling patterns established in the Reference Implementation.

---

## Reference Implementation

The `temp/` folder contains a complete **LLM Switchboard** project (HTML, CSS, and JS files). This is **NOT** part of the current project — do not include it in the final build or deployment.

Use it as a reference for:
- How to parse a `.env` file for API keys (in-memory only)
- The `fetch()` call structure for OpenAI's Chat Completions API
- Error handling patterns for failed API requests
- How the code is organized across separate files
- The general approach to building a single-page LLM tool

Ignore these Switchboard features (not needed here):
- Anthropic integration (this project is OpenAI-only)
- The model selection dropdown / provider switching
- Structured output mode and JSON schema handling

This project uses **unstructured (free-form) responses only.**
Render the model's Markdown output as formatted HTML.

---

## Tech Stack

| Concern | Approach |
|---|---|
| Structure | Single `index.html` file |
| Styling | Inline `<style>` block within `index.html` |
| Logic | Inline `<script>` block within `index.html` |
| Markdown rendering | [marked.js](https://cdn.jsdelivr.net/npm/marked/marked.min.js) via CDN |
| API | OpenAI Chat Completions (`fetch`) |
| Persistence | None — in-memory only |
| Hosting | GitHub Pages (static) |

---

## File Structure

```
project-root/
├── index.html        # Complete application (HTML + CSS + JS)
├── .env              # API key (gitignored, never committed)
├── .gitignore        # Must include .env
└── temp/             # Reference implementation only — excluded from build
```

---

## Out of Scope

- No backend server or Node.js runtime in production.
- No database or persistent storage.
- No user authentication.
- No Anthropic or Google model support.
- No structured JSON output mode.
- No multi-experiment comparison view.
