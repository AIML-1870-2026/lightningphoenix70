# Product Review Generator — Technical Specification

## Overview

A dynamic, single-page web application built with Node.js that enables users to generate
polished product reviews by interacting with a large language model. Users describe a
product, customize review parameters, and select both a provider (Anthropic or OpenAI)
and a specific model within that provider's family. The generated review is streamed back
to the page and rendered as formatted HTML.

---

## Goals

- Provide an intuitive UI for generating product reviews via LLM.
- Support two providers — **Anthropic (Claude)** and **OpenAI (GPT / o-series)** — with
  seamless switching between them.
- Allow users to select a model family and a specific model within it.
- Support customization of review tone, length, star rating, and reviewer persona.
- Stream output progressively so the review appears token-by-token.
- Render the model's markdown output as formatted HTML in the browser.
- Keep all API keys server-side; never expose them to the browser.

---

## Reference Implementation

The `temp/` folder contains a complete **LLM Switchboard** project (HTML, CSS, and JS
files). This is **NOT** part of the current project — do not include it in the final
build or deployment.

Use it as a reference for:

- How to parse a `.env` file for API keys (in-memory only, never written to disk or
  sent to the client)
- The `fetch()` call structure for OpenAI's chat completions API
- Error handling patterns for failed or timed-out API requests
- How code is organized across separate files (router, prompt builder, stream handler)
- The general approach to building a single-page LLM tool

Ignore these Switchboard features (not needed here):

- Structured output mode and JSON schema handling — this project uses **free-form
  (unstructured) responses only**
- Any provider-specific UI chrome that does not apply to the two providers above

---

## Architecture

```
Browser (HTML/CSS/JS)
        │
        │  HTTP POST + Server-Sent Events (SSE)
        ▼
Node.js Express Server
        │
        ├──▶ Anthropic Messages API  (provider = "anthropic")
        │
        └──▶ OpenAI Chat Completions API  (provider = "openai")
```

### Backend — Node.js + Express

- Serves the static frontend from `public/`.
- Exposes a single `POST /generate` endpoint.
- Reads `ANTHROPIC_API_KEY` and `OPENAI_API_KEY` from a `.env` file at startup
  (following the pattern in the Switchboard reference). Neither key is ever sent to
  the browser.
- Selects the correct upstream API based on the `provider` field in the request body.
- Streams the upstream response back to the browser as SSE (`text/event-stream`).
- Returns structured JSON error responses on failure (4xx/5xx).

### Frontend — Vanilla HTML / CSS / JS (no build step)

- Single HTML file served by Express; CSS and JS in separate files following the
  Switchboard file-organization pattern.
- Provider and model selection updates the UI dynamically (no page reload).
- Uses `fetch()` + `ReadableStream` to consume the SSE stream and append tokens to
  the output area as they arrive.
- Converts the completed markdown response to formatted HTML using **marked.js** (CDN)
  once streaming finishes, replacing the raw token buffer.

---

## Provider & Model Selection

The user first picks a **provider**, then a **model family** within that provider, then
a **specific model** within that family. Changing the provider resets the family and
model selections.

### Anthropic — Claude Models

| Family     | Display Name      | API Model String             | Tier     |
|------------|-------------------|------------------------------|----------|
| Claude 4.6 | Claude Opus 4.6   | `claude-opus-4-6`            | Flagship |
| Claude 4.6 | Claude Sonnet 4.6 | `claude-sonnet-4-6`          | Balanced |
| Claude 4.5 | Claude Sonnet 4.5 | `claude-sonnet-4-5-20251001` | Balanced |
| Claude 4.5 | Claude Haiku 4.5  | `claude-haiku-4-5-20251001`  | Fast     |

Default when Anthropic is selected: **Claude Sonnet 4.6**

### OpenAI — GPT & o-series Models

| Family   | Display Name | API Model String | Tier              |
|----------|--------------|------------------|-------------------|
| GPT-4o   | GPT-4o       | `gpt-4o`         | Flagship          |
| GPT-4o   | GPT-4o mini  | `gpt-4o-mini`    | Fast              |
| o-series | o3           | `o3`             | Reasoning         |
| o-series | o4-mini      | `o4-mini`        | Reasoning / Fast  |

Default when OpenAI is selected: **GPT-4o**

> **Note:** o-series models do not support a `system` role message. When an o-series
> model is selected, merge the system prompt content into the first `user` message
> instead of sending it as a separate `system` turn.

Each model option in the UI should display its tier label and a one-line description
to help users choose.

---

## User Inputs

| Field               | Type             | Options / Constraints                                                                      | Required |
|---------------------|------------------|--------------------------------------------------------------------------------------------|----------|
| Provider            | Segmented toggle | Anthropic, OpenAI                                                                          | Yes      |
| Model Family        | Segmented toggle | Populated from the selected provider's family list                                         | Yes      |
| Specific Model      | Radio group      | Populated from the selected family                                                         | Yes      |
| Product Name        | Text input       | 1–120 characters                                                                           | Yes      |
| Product Category    | Dropdown         | Electronics, Apparel, Home & Kitchen, Beauty, Books, Sports, Toys, Food & Beverage, Other | Yes      |
| Product Description | Textarea         | Up to 500 characters; brief summary of features                                            | Yes      |
| Star Rating         | Star picker      | 1–5 stars (integer)                                                                        | Yes      |
| Review Tone         | Radio / toggle   | Enthusiastic, Balanced, Critical                                                           | Yes      |
| Review Length       | Radio / slider   | Short (~100 words), Medium (~250 words), Long (~500 words)                                 | Yes      |
| Reviewer Persona    | Dropdown         | Casual Buyer, Tech Enthusiast, Professional Critic, Parent, Gift Buyer                     | No — defaults to Casual Buyer |

---

## Prompt Construction

The server builds prompts from the submitted fields. Both providers receive the same
logical content; only the message structure differs.

### System Prompt (all models except o-series)

```
You are an expert product reviewer. Write authentic, helpful product reviews in the
voice of the specified reviewer persona. Match the requested tone and length precisely.
Use markdown formatting — paragraphs, bold key phrases, and a bullet list of pros/cons
where appropriate. Do not include a headline or title — output the review body only.
```

### User Prompt Template

```
Write a {length} product review for the following item.

Product Name: {productName}
Category: {category}
Description: {description}
Star Rating: {rating}/5
Tone: {tone}
Reviewer Persona: {persona}

Write the review now.
```

### o-series Adjustment

o-series models do not accept a `system` role. Prepend the system prompt inline into
the user message instead:

```
[You are an expert product reviewer. Write authentic, helpful product reviews in the
voice of the specified reviewer persona. Match the requested tone and length precisely.
Use markdown formatting — paragraphs, bold key phrases, and a bullet list of pros/cons
where appropriate. Do not include a headline or title — output the review body only.]

Write a {length} product review for the following item.
...
```

---

## API Integration

### `/generate` Endpoint

**Method:** `POST`
**Content-Type:** `application/json`
**Response:** `text/event-stream` (SSE)

#### Request Body

```json
{
  "provider":    "anthropic | openai",
  "model":       "<api model string>",
  "productName": "string",
  "category":    "string",
  "description": "string",
  "rating":      3,
  "tone":        "Enthusiastic | Balanced | Critical",
  "length":      "Short | Medium | Long",
  "persona":     "string"
}
```

#### SSE Event Format

Each streamed chunk is emitted as:

```
data: <token text>\n\n
```

A special termination event signals the end of the stream:

```
data: [DONE]\n\n
```

On error, emit a single event before closing:

```
data: [ERROR] <human-readable message>\n\n
```

### Anthropic API Call (server-side)

```js
// POST https://api.anthropic.com/v1/messages
{
  model:      req.body.model,
  max_tokens: 1024,
  stream:     true,
  system:     systemPrompt,
  messages: [
    { role: "user", content: userPrompt }
  ]
}
// Headers:
//   x-api-key: <ANTHROPIC_API_KEY>
//   anthropic-version: "2023-06-01"
//   content-type: application/json
```

Consume `content_block_delta` events from the Anthropic stream and forward each
`delta.text` value as an SSE chunk to the browser.

### OpenAI API Call (server-side)

Follow the `fetch()` pattern from the Switchboard reference for the chat completions
endpoint (`POST https://api.openai.com/v1/chat/completions`).

```js
// Standard models (GPT-4o family)
{
  model:    req.body.model,
  stream:   true,
  messages: [
    { role: "system", content: systemPrompt },
    { role: "user",   content: userPrompt }
  ]
}

// o-series models — no system role
{
  model:    req.body.model,
  stream:   true,
  messages: [
    { role: "user", content: mergedPrompt }   // system content prepended inline
  ]
}

// Header: Authorization: Bearer <OPENAI_API_KEY>
```

Consume `choices[0].delta.content` from each streamed chunk and forward to the browser.

---

## Markdown Rendering

- While streaming, append raw token text to a `<pre>` element so the user sees output
  immediately.
- When the `[DONE]` event is received, pass the full accumulated text through
  **marked.js** (`marked.parse()`) and replace the `<pre>` with the resulting HTML
  inside a styled `<div class="review-output">`.
- Sanitize the HTML with **DOMPurify** (CDN) before injecting it into the DOM.

---

## Error Handling

Follow the error handling patterns from the Switchboard reference. Additionally:

| Scenario                           | Behavior                                                          |
|------------------------------------|-------------------------------------------------------------------|
| Missing or invalid API key         | 500 response: `"API key not configured for <provider>"`           |
| Upstream API returns 4xx           | Forward status + upstream message; emit `[ERROR]` SSE event       |
| Upstream API returns 5xx / timeout | Retry once after 1 s; if still failing, emit `[ERROR]` SSE event  |
| Required field missing in request  | 400 response before any stream is opened                          |
| Stream interrupted mid-response    | Display partial review with a visible "⚠ Stream interrupted" notice |

If only one API key is configured, the UI should disable the corresponding provider
toggle and show a tooltip explaining why it is unavailable, rather than allowing the
user to select it and encounter a runtime error.

---

## File Structure

```
product-review-generator/
├── .env                    # ANTHROPIC_API_KEY, OPENAI_API_KEY (never committed)
├── .gitignore              # includes .env and node_modules
├── package.json
├── server.js               # Express entry point; mounts routes; loads .env
├── routes/
│   └── generate.js         # POST /generate — provider routing + SSE streaming
├── lib/
│   ├── promptBuilder.js    # Builds system + user prompts from request fields;
│   │                       #   handles o-series merge logic
│   ├── anthropicClient.js  # Streaming call to Anthropic Messages API
│   └── openaiClient.js     # Streaming call to OpenAI Chat Completions API
│                           #   (based on Switchboard fetch() pattern)
└── public/
    ├── index.html          # Single-page UI markup
    ├── style.css           # All styles
    └── app.js              # Provider/model selection logic, fetch + SSE consumer,
                            #   marked.js rendering, DOMPurify sanitization
```

> `temp/` (the Switchboard reference) lives outside this directory and is never
> referenced by any `require()` or `import` in the project source.

---

## Environment Variables

| Variable            | Required | Description                      |
|---------------------|----------|----------------------------------|
| `ANTHROPIC_API_KEY` | Yes*     | Anthropic API key                |
| `OPENAI_API_KEY`    | Yes*     | OpenAI API key                   |
| `PORT`              | No       | HTTP port; defaults to `3000`    |

\* At least one must be present at startup. If a key is absent, requests to that
provider return a 500 error with a clear message; the other provider remains usable.

---

## Dependencies

| Package      | Purpose                                        |
|--------------|------------------------------------------------|
| `express`    | HTTP server and routing                        |
| `dotenv`     | `.env` parsing at startup                      |
| `node-fetch` | `fetch()` polyfill for Node < 18               |
| *(CDN only)* |                                                |
| `marked.js`  | Markdown → HTML conversion (browser-side)      |
| `DOMPurify`  | HTML sanitization before DOM injection         |

No frontend build tool or bundler is required.

---

## Out of Scope

- User authentication or session management
- Saving or exporting generated reviews
- Structured / JSON output mode
- Image input or multimodal features
- The Switchboard's model-selection dropdown pattern (replaced by the hierarchical
  provider → family → model selection described above)
