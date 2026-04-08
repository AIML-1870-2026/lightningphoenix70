# LLM Switchboard Explorer — Product Specification

**Version:** 1.0  
**Status:** Draft  
**Delivery Format:** Single HTML file (no build step)

---

## 1. Overview

The LLM Switchboard Explorer is a single-file web application that lets a user interact with large language models from multiple providers through their APIs. It serves as reference infrastructure for future assignments where specialized tools will interact with models in specific ways.

The app supports provider and model selection, dual output modes (unstructured and structured JSON), preloaded example prompts and schema templates, and a side-by-side model comparison view. All API keys are handled in-memory only and are never persisted.

---

## 2. Goals

- Provide a clean, low-friction interface for testing prompts across providers and models
- Demonstrate unstructured vs. structured (JSON schema) output patterns
- Enable simultaneous comparison of two models on the same prompt
- Be fully self-contained: one `.html` file, no server, no build step
- Serve as reusable reference infrastructure — future assignments will fork or extend this file

---

## 3. Non-Goals

- No user accounts, no saved history, no backend
- No support for streaming responses in v1
- No mobile-optimized layout (desktop browser assumed)
- No provider beyond OpenAI for live API calls in v1 (see Section 9 for Anthropic constraint)

---

## 4. API Key Handling

### 4.1 Input Methods

Users may provide API keys in two ways:

- **Manual paste:** A password-type text input field per provider. Keys are masked by default with a show/hide toggle.
- **File upload:** A file input that accepts `.env`, `.json`, or plain `.txt` files. The app parses the file client-side and extracts known key patterns (e.g., `OPENAI_API_KEY=sk-...`, `ANTHROPIC_API_KEY=sk-ant-...`).

### 4.2 Storage and Privacy

- Keys are stored exclusively in JavaScript memory (a module-scoped variable).
- Keys are **never** written to `localStorage`, `sessionStorage`, `IndexedDB`, cookies, or any form of persistent storage.
- Keys are **never** sent anywhere except directly to the relevant provider's API endpoint.
- On page close or refresh, all keys are lost. This is by design.

### 4.3 Privacy Guarantee UI

A persistent banner is displayed at the top of the page at all times:

> 🔒 **Your API keys are never stored.** They live in memory only and are cleared when you close or refresh this page.

This banner uses a neutral, non-alarming color (e.g., muted blue or gray) so it reads as informational rather than a warning.

---

## 5. Provider and Model Selection

### 5.1 Supported Providers

| Provider | Live API Calls | Notes |
|---|---|---|
| OpenAI | ✅ Yes | Full support |
| Anthropic | ⚠️ Limited | CORS constraint; see Section 9 |

### 5.2 Model List

Model lists are **hardcoded** in v1. Dynamic discovery via the API is not used because it requires additional authenticated calls, can fail silently, and may expose keys in unexpected request patterns.

A `lastVerified` date is shown next to the model selector (e.g., *"Models last verified April 2025"*) so users understand the list may not reflect the latest releases.

**OpenAI models (initial list):**
- gpt-4o
- gpt-4o-mini
- gpt-4-turbo
- gpt-3.5-turbo

**Anthropic models (initial list):**
- claude-opus-4-5
- claude-sonnet-4-5
- claude-haiku-4-5

### 5.3 Model Selector UI

A two-step selector: choose provider first (segmented button or tabs), then choose model from a dropdown filtered to that provider. The selected model and provider are always visible in the header of the prompt panel and the comparison columns.

---

## 6. Dual Output Modes

### 6.1 Mode Toggle

A clearly labeled toggle switch in the prompt panel header switches between **Unstructured** and **Structured** modes. The toggle persists its state across prompts within the same session.

### 6.2 Unstructured Mode

- Prompt: full-width textarea
- Response: full-width scrollable text panel with monospace font
- No schema editor visible

### 6.3 Structured Mode

When structured mode is active, the layout changes:

- **Schema panel** appears below the prompt textarea (or in a collapsible side panel — implementer's choice). It contains a JSON editor (see Section 8.2) pre-populated with the selected schema template.
- The system prompt is automatically augmented with an instruction to return only valid JSON matching the provided schema. This augmentation is shown to the user in a read-only collapsed callout so they can see what is being sent.
- **Response panel** switches to a JSON viewer with syntax highlighting, collapsible tree nodes, and a validation badge. If the response is valid JSON matching the schema, a green ✓ badge appears. If it is valid JSON but does not match the schema, a yellow ⚠ badge appears with a diff. If it is not valid JSON at all, a red ✗ badge appears with the raw text shown below the viewer.

---

## 7. Example Prompts

Prompts are organized into three categories. Users select from a dropdown; selecting a prompt populates the textarea (replacing any existing content after a confirmation if the textarea is non-empty).

### 7.1 General Reasoning / Logic

| Label | Prompt |
|---|---|
| River crossing | "A farmer needs to cross a river with a fox, a chicken, and a bag of grain. He can only take one at a time in his boat. The fox will eat the chicken if left alone, and the chicken will eat the grain. How does he get everything across safely?" |
| Coin on table | "A table has 52 cards face down. 13 are heads-up. You cannot see or feel which are heads-up. How do you create two groups with the same number of heads-up cards?" |
| Counterfeit coin | "You have 12 coins, one of which is counterfeit and either heavier or lighter than the rest. Using a balance scale with exactly 3 weighings, how do you identify the counterfeit coin and whether it is heavier or lighter?" |

### 7.2 Code Generation

| Label | Prompt |
|---|---|
| Debounce function | "Write a JavaScript `debounce` function that delays invoking a callback until after a given number of milliseconds have elapsed since the last invocation. Include a `cancel` method on the returned function. Add JSDoc comments and at least two usage examples." |
| SQL window function | "Write a SQL query using window functions that, for each employee in a `salaries` table (columns: employee_id, department, salary, hire_date), returns their salary rank within their department and their salary as a percentage of the department's maximum salary." |
| Retry with backoff | "Implement an async Python function `fetch_with_retry(url, max_retries=3)` that retries a failed HTTP GET request using exponential backoff with jitter. Handle timeout and HTTP 429 errors specifically. Include type hints." |

### 7.3 Data Analysis

| Label | Prompt |
|---|---|
| Explain outlier | "You are analyzing monthly revenue data: [120, 118, 125, 122, 119, 121, 84, 123, 120]. Identify any outliers, hypothesize three possible causes for each, and describe what additional data you would want to confirm or rule out each hypothesis." |
| Metric selection | "A product team wants to measure whether a new onboarding flow is better than the old one. They have access to: sign-up completion rate, time-to-first-action, 7-day retention, and support ticket volume. Recommend the single most important metric to use as the primary success indicator and explain why the others are secondary." |
| Correlation vs causation | "A dataset shows a strong positive correlation between ice cream sales and drowning incidents. Explain why this correlation does not imply causation, identify the likely confounding variable, and describe how you would design a study to isolate a true causal relationship." |

---

## 8. Schema Templates and Editor

### 8.1 Preloaded Templates

Three templates are available in a dropdown in the schema panel. Selecting a template populates the schema editor.

**Sentiment + Reasoning**
```json
{
  "type": "object",
  "properties": {
    "sentiment": { "type": "string", "enum": ["positive", "neutral", "negative"] },
    "confidence": { "type": "number", "minimum": 0, "maximum": 1 },
    "reasoning": { "type": "string" },
    "key_phrases": { "type": "array", "items": { "type": "string" } }
  },
  "required": ["sentiment", "confidence", "reasoning"]
}
```

**Named Entity Recognition**
```json
{
  "type": "object",
  "properties": {
    "entities": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "text": { "type": "string" },
          "type": { "type": "string", "enum": ["PERSON", "ORG", "LOCATION", "DATE", "OTHER"] },
          "start_index": { "type": "integer" }
        },
        "required": ["text", "type"]
      }
    }
  },
  "required": ["entities"]
}
```

**Simple Key-Value Extraction**
```json
{
  "type": "object",
  "properties": {
    "extracted_fields": {
      "type": "object",
      "additionalProperties": { "type": "string" }
    },
    "unextracted_keys": {
      "type": "array",
      "items": { "type": "string" }
    }
  },
  "required": ["extracted_fields"]
}
```

### 8.2 Schema Editor

The schema editor is a syntax-highlighted `<textarea>` with:

- Real-time JSON validation (red border + inline error message on invalid JSON)
- A **Format** button that pretty-prints the JSON in place
- A **Reset** button that restores the last-selected template
- A **Save as Custom** button that stores the current schema in memory under a user-provided name, which then appears in the template dropdown under a "Custom" group for the remainder of the session

Users may freely edit any template. Edits do not overwrite the original template unless saved as custom.

---

## 9. Anthropic CORS Constraint

### 9.1 The Problem

Anthropic's API does not include CORS headers that permit direct browser-to-API calls. Any attempt to call `api.anthropic.com` from a browser will be blocked by the browser's same-origin policy. This is not a bug in the app — it is a deliberate server-side policy.

### 9.2 In-App Behavior (v1)

When a user selects an Anthropic model, the app:

1. Displays an **inline informational callout** in the prompt panel (yellow/amber, non-blocking):

   > ⚠️ **Anthropic models can't be called directly from a browser.** Anthropic's API does not allow cross-origin requests. To use Claude models, see the proxy setup guide below or use [Claude.ai](https://claude.ai) directly.

2. **Disables the Send button** for Anthropic models and replaces it with a grayed-out "Requires proxy" button.
3. Anthropic models remain **fully selectable in the comparison view** for layout preview purposes, but both columns will show the callout and disabled state if an Anthropic model is selected.

The Anthropic models remain visible in the UI (not hidden) because the app is reference infrastructure and the user should understand the full model landscape even where live calls are not possible.

### 9.3 Recommended Resolution Path (Future Iteration)

The recommended fix is a lightweight local proxy. A future version of this spec should include:

- A minimal Node.js or Python proxy (10–20 lines) that the user runs locally, which forwards requests from `localhost` to `api.anthropic.com` and adds the required CORS headers
- A **proxy status indicator** in the app header (e.g., a colored dot: green = proxy detected, gray = not running) that pings `localhost:3001/health` on load and periodically thereafter
- Instructions for starting the proxy embedded in the app as a collapsible "Setup" panel, shown automatically when an Anthropic model is selected
- Once the proxy is detected, the Send button re-enables for Anthropic models and calls `localhost:3001/proxy` instead of `api.anthropic.com` directly

This approach keeps the app as a single HTML file while unblocking Anthropic support without requiring any cloud infrastructure.

---

## 10. Side-by-Side Comparison View

### 10.1 Layout

The comparison view is a separate tab or section accessible from the main navigation. It renders as two equal-width columns, each containing:

- A **model selector** (provider + model, independent per column)
- A **response panel** (shared scroll position is not required)
- A **response time badge** (shown after response arrives, e.g., "3.2s")
- A **status indicator** (idle / loading / error / complete)

A single **shared prompt textarea** sits above both columns. The output mode toggle (unstructured / structured) also applies globally to both columns. If structured mode is active, a single schema editor is shown and used for both requests.

### 10.2 Sending Requests

A **"Send to Both"** button fires both API requests simultaneously using `Promise.allSettled`. Each column updates independently as its response arrives. One column completing does not block or affect the other.

A **"Clear Both"** button resets both response panels without clearing the prompt.

### 10.3 Comparison with Anthropic Models

If either column has an Anthropic model selected, that column shows the CORS callout and disabled state (see Section 9.2). The other column may still fire its request normally. This makes the constraint visible without breaking the entire comparison view.

---

## 11. Error Handling

All error states are displayed inline within the response panel of the affected column (or the main response panel in single-model mode). Errors do not use browser `alert()`.

| Error Type | Detection | UI Treatment |
|---|---|---|
| Invalid API key | HTTP 401 | Red banner: "Invalid API key. Please check your key and try again." Key field is highlighted. |
| Rate limit | HTTP 429 | Amber banner: "Rate limit reached. Retry after [X] seconds." A countdown timer is shown if a `Retry-After` header is present. |
| Request timeout | No response after 30s | Amber banner: "Request timed out after 30 seconds." A **Retry** button re-fires the same request. |
| Network error | `fetch` throws | Red banner: "Network error. Check your connection and try again." |
| Invalid JSON (structured mode) | Response received but not valid JSON | Yellow badge in response panel with raw text shown below (see Section 6.3). |
| CORS block (Anthropic) | Browser blocks request | Prevented by disabling send; not an runtime error state (see Section 9.2). |

---

## 12. Response Display

- Responses appear in a panel with a maximum height of approximately 60% of the viewport.
- Content exceeding the panel height is scrollable within the panel (the page itself does not scroll to accommodate responses).
- A **Copy to Clipboard** button appears in the panel header once a response is present.
- An **Expand Full** button opens the response in a modal overlay for comfortable reading of long outputs.
- In structured mode, the JSON viewer supports collapsible tree nodes for nested objects and arrays.
- Response panels are cleared when a new request is sent.

---

## 13. Technical Implementation Notes

### 13.1 Dependencies (CDN-loaded)

All dependencies are loaded from `cdnjs.cloudflare.com` or equivalent trusted CDNs. No npm, no build step.

| Purpose | Library |
|---|---|
| JSON syntax highlighting | highlight.js (json theme) |
| JSON schema validation | Ajv (Another JSON Validator) |
| General UI | Vanilla JS + CSS custom properties (no framework) |

### 13.2 File Structure

The entire app lives in one `.html` file with three clearly delimited sections:

```
<!-- === STYLES === -->
<style> ... </style>

<!-- === HTML === -->
<body> ... </body>

<!-- === SCRIPTS === -->
<script> ... </script>
```

Internal JS is organized into named modules using IIFE or ES module patterns to keep the single file maintainable.

### 13.3 State Management

All app state (keys, current model selections, current prompt, mode, schema, response history for the session) lives in a single top-level `AppState` object. No global variable sprawl.

### 13.4 API Call Pattern

All provider API calls go through a single `callModel(provider, model, messages, schema?)` function that abstracts provider-specific request shapes and returns a normalized `{ text, latencyMs, error }` result object.

---

## 14. Future Iterations

Beyond the Anthropic proxy path described in Section 9.3, the following capabilities are out of scope for v1 but should be considered for future versions:

- **Streaming responses:** Display tokens as they arrive rather than waiting for the full response.
- **Session history:** An in-memory log of recent prompt/response pairs, exportable as JSON.
- **Token counting:** Show estimated token usage per request and running session totals.
- **Additional providers:** Gemini (Google), Mistral, and local model support via Ollama's OpenAI-compatible API.
- **Prompt diffing:** In comparison mode, highlight semantic differences between two model responses.
- **Schema inference:** Given a sample response, suggest a JSON schema that would capture it.
