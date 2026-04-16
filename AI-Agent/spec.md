# Blackjack AI Agent — Specification

## 1. Overview

A single-page static web application that plays Blackjack with assistance from an LLM-backed agent. The user supplies an API key via an uploaded `.env` file. The app deals hands, queries the LLM for structured action recommendations, visualizes those recommendations against the known-optimal Basic Strategy matrix, executes the chosen action, and tracks performance metrics over time. The user can select a risk profile that is passed to the LLM as part of its system prompt, letting them observe how recommendations shift between conservative, balanced, and aggressive play.

The entire app runs client-side. No server, no build step, no persistence of the API key.

---

## 2. Reference Material: the `temp/` Folder

The repository contains a `temp/` folder with a working example of a static webpage that interacts with an LLM via an uploaded `.env` file. Use it as a reference for:

- How to parse a `.env` file for the API key (in-memory only).
- The `fetch()` call structure for the LLM API.
- Error handling patterns for failed API requests.

**Do NOT include the `temp/` folder in the final build or deployment.** It exists solely as a pattern reference during development. Specifically:

- No files from `temp/` should be linked, imported, copied, or bundled into the deliverable.
- The `temp/` folder should be excluded from any deployment artifact (add it to `.gitignore` for deploy branches, or delete it before shipping).
- If a pattern from `temp/` is useful, reimplement it cleanly inside the new module structure described in §5 — do not `import` from `temp/` directly.

### Build requirements (verbatim from the brief)

Build a static Blackjack AI agent page that:

- Accepts an API key via `.env` file upload.
- Implements a full Blackjack game (deck, dealing, scoring).
- Calls the LLM to recommend hit/stand given the current hand.
- Returns a structured JSON response so the action can be extracted reliably (avoids keyword-search ambiguity).
- Logs key interactions to the console for debugging.
- Tracks the player's balance across hands.

Every item above is addressed in the sections that follow.

---

## 3. Goals & Non-Goals

**Goals**
- Playable Blackjack game with correct rules (hit, stand, double, split where reasonable).
- LLM recommends actions via a structured JSON response — no keyword-scraping.
- Side-by-side view of the LLM's recommendation and the Basic Strategy matrix cell that applies to the current hand.
- Risk profile selector that modifies LLM prompting.
- Running metrics: win rate, bankroll curve, decision quality vs. Basic Strategy.
- API key stays in memory for the session only.

**Non-Goals**
- Multiplayer, betting against other users, real money, or any persistence of user data across sessions.
- Full card-counting simulations or multi-deck shoe modeling beyond what's needed for Basic Strategy.
- Server-side anything.

---

## 4. Tech Stack

- **HTML / CSS / vanilla JavaScript (ES modules).** No framework required.
- **Chart.js** via CDN for the bankroll / win-rate charts.
- **No bundler, no build.** Open `index.html` directly or serve with any static file server.
- **LLM provider:** Anthropic Claude API (`https://api.anthropic.com/v1/messages`), model `claude-sonnet-4-5` by default. The code path is provider-agnostic enough that swapping to OpenAI's chat completions endpoint is a config change.

---

## 5. File Structure

```
/
├── index.html           # Single page, all UI mounted here
├── css/
│   └── styles.css
├── js/
│   ├── main.js          # Entry point, wires everything together
│   ├── env.js           # .env file parsing (in-memory only)
│   ├── deck.js          # Card, deck, shuffle, deal
│   ├── game.js          # Blackjack rules, hand scoring, game loop
│   ├── agent.js         # LLM call, structured response parsing
│   ├── strategy.js      # Basic Strategy matrix + lookup
│   ├── metrics.js       # Win rate, bankroll, decision-quality tracking
│   └── ui.js            # DOM rendering, event handlers
├── assets/
│   └── cards/           # Optional card face SVGs (or CSS-drawn cards)
└── spec.md
```

The `temp/` reference folder is **excluded** from the final deliverable.

---

## 6. API Key Handling

### 6.1 Upload flow
- A file input accepts a `.env` file. On change, the file is read with `FileReader.readAsText()`.
- `env.js` parses it line by line: skip blanks and lines starting with `#`, split on the first `=`, trim whitespace, strip matching surrounding quotes.
- The parsed key is held in a module-scoped variable in `env.js` and exposed only via a getter. It is **never**:
  - written to `localStorage`, `sessionStorage`, `IndexedDB`, or cookies;
  - logged to the console;
  - sent anywhere except the LLM provider's endpoint in the `Authorization` / `x-api-key` header.
- The UI displays only a masked confirmation (`sk-ant-…`, last 4 chars) once loaded.

### 6.2 Expected .env contents
```
ANTHROPIC_API_KEY=sk-ant-...
# Optional overrides
LLM_MODEL=claude-sonnet-4-5
```

### 6.3 Session lifecycle
- Key is discarded on page reload/close (it only lives in a JS variable).
- A "Clear key" button in the UI zeroes the variable and re-locks the agent controls.

---

## 7. Blackjack Game Engine (`game.js`, `deck.js`)

### 7.1 Rules implemented
- Standard 6-deck shoe, reshuffled when the cut card (~75% penetration) is reached.
- Dealer stands on all 17s (S17).
- Blackjack pays 3:2.
- Double allowed on any first two cards.
- Split allowed on any pair; one split per hand (keeps UI and LLM context manageable). No resplit, no surrender in v1.
- No insurance in v1 (can be added later; out of scope for the first build).

### 7.2 Scoring
- Cards 2–10 at face value, J/Q/K = 10, A = 11 or 1 (soft/hard logic in a helper).
- `scoreHand(cards)` returns `{ total, isSoft, isBlackjack, isBust }`.

### 7.3 Game loop (per hand)
1. Player places bet (default 10 units, adjustable).
2. Deal two cards to player, two to dealer (one up, one down).
3. Check for naturals. Settle immediately if either side has blackjack.
4. Player turn loop:
   - Build current game state object.
   - Call agent for recommendation (see §8).
   - Show recommendation in the UI. User can **accept** (one click) or **override** (hit/stand/double/split buttons).
   - Record the user's final action and whether it matched the agent and whether it matched Basic Strategy.
   - Apply the action. Loop until stand, bust, double-then-deal-one, or split-resolved.
5. Dealer reveals hole card, draws to 17+.
6. Resolve payout, update bankroll, append result to metrics.

### 7.4 Console logging
Log one line per significant event (deal, recommendation received, action applied, round resolved, shuffle) with a consistent prefix like `[BJ]`. No PII, no API key.

---

## 8. LLM Agent (`agent.js`)

### 8.1 Request shape
`POST https://api.anthropic.com/v1/messages` with:
- `model`: from env or default `claude-sonnet-4-5`.
- `max_tokens`: 300 (responses are small JSON objects).
- `system`: composed from a base prompt plus the selected risk profile (§10).
- `messages`: a single user message containing a JSON-serialized game state:
  ```json
  {
    "player_hand": ["Ah", "7c"],
    "player_total": 18,
    "is_soft": true,
    "dealer_up": "9d",
    "can_double": true,
    "can_split": false,
    "bankroll": 980,
    "bet": 10,
    "hands_played": 14
  }
  ```
- The system prompt instructs the model to respond **only** with a JSON object matching a fixed schema. No prose, no markdown fences.

### 8.2 Response schema
```json
{
  "action": "hit" | "stand" | "double" | "split",
  "confidence": 0.0,
  "reasoning": "short string, 1-2 sentences",
  "estimated_win_probability": 0.0
}
```

### 8.3 Parsing & validation
- `JSON.parse` the response body.
- Validate `action` is in the allowed set and is legal given the current state (e.g., no double on three cards). If invalid, fall back to the Basic Strategy matrix lookup and surface a warning in the UI.
- Strip any accidental markdown fences before parsing (defensive — the prompt forbids them, but check anyway).

### 8.4 Error handling (pattern mirrored from `temp/`)
- Network failure, non-2xx status, or JSON parse failure → show a non-blocking error toast, fall back to Basic Strategy, and log the full error to console.
- Rate-limit (429) → exponential backoff up to 3 retries, then fall back.
- Invalid key (401) → prompt user to reload `.env`.

---

## 9. Basic Strategy Matrix (`strategy.js`)

### 9.1 Data
Three lookup tables keyed by `[player_hand_descriptor][dealer_up_rank]`:
- Hard totals (5–21)
- Soft totals (A,2 through A,9)
- Pairs (2,2 through A,A)

Values: `"H"` hit, `"S"` stand, `"D"` double (or hit if not allowed), `"P"` split, `"Ds"` double (or stand if not allowed).

This is the standard S17 Basic Strategy chart; values are encoded as constants.

### 9.2 Lookup
`getBasicStrategyAction(playerHand, dealerUpCard, { canDouble, canSplit })` returns a normalized action string matching the agent's schema (`"hit" | "stand" | "double" | "split"`).

### 9.3 Visualization
- Render the matrix as a 3-tab view (Hard / Soft / Pairs), each a color-coded grid.
- The cell that corresponds to the current hand is highlighted.
- The LLM's chosen action is overlaid on that cell with a badge; if it differs from Basic Strategy, the cell is marked with a subtle warning stripe.
- Legend explains colors (green = stand, red = hit, blue = double, yellow = split).

---

## 10. Risk Profiles

Selectable from a dropdown:

| Profile | System prompt delta |
|---|---|
| **Conservative** | Avoid double and split unless heavily favored. Prefer standing on marginal totals (12–16 vs. dealer 7+ → lean stand only on 16). Prioritize bankroll preservation. |
| **Balanced** | Follow Basic Strategy. This is the default and should closely mirror the matrix. |
| **Aggressive** | Take doubles and splits whenever the edge is plausibly positive. Accept higher variance for higher expected value. |
| **Card-aware (experimental)** | In addition to balanced strategy, consider that the shoe has been dealt from and tilt marginally based on `hands_played`. Explicitly disclaimed as heuristic. |

The profile string is injected into the system prompt. The UI shows, under the recommendation, a small note like *"Recommendation adjusted for: Aggressive profile"*.

Switching profiles mid-hand is allowed; the next `agent.recommend()` call uses the new profile.

---

## 11. Metrics (`metrics.js`)

All metrics are computed from an in-memory array of round records. No persistence.

### 11.1 Per-round record
```js
{
  roundId, timestamp,
  bet, payout, bankrollAfter,
  result: 'win' | 'loss' | 'push' | 'blackjack',
  decisions: [
    { state, agentAction, basicStrategyAction, userAction, matchedAgent, matchedBasic }
  ]
}
```

### 11.2 Displayed metrics
- **Win rate** (wins / (wins + losses), pushes excluded) with running count.
- **Bankroll over time** — line chart, x = hand number, y = bankroll.
- **Decision quality** — % of decisions matching Basic Strategy; separately, % matching the agent. Shown as two gauges.
- **Agent vs. Basic agreement rate** — how often the LLM's pick equals the Basic Strategy pick. Useful for spotting drift between profiles.
- **Hands played**, **current streak**, **biggest win**, **biggest loss**.

### 11.3 Reset
A "Reset session" button clears metrics, reshuffles, and restores bankroll to the starting value (default 1000 units).

---

## 12. UI Layout (`ui.js`, `index.html`, `styles.css`)

Single-page, three-column layout on desktop; stacked on mobile.

```
┌─────────────────────────────────────────────────────────────────┐
│ Header: title · .env status · risk profile dropdown · reset    │
├──────────────────┬──────────────────────────┬───────────────────┤
│ Game table       │ Agent panel              │ Strategy matrix   │
│ - dealer hand    │ - recommended action     │ - Hard/Soft/Pairs │
│ - player hand(s) │ - confidence             │ - current cell    │
│ - bet controls   │ - reasoning              │   highlighted     │
│ - action buttons │ - est. win probability   │ - agent's pick    │
│ - bankroll       │ - "Accept" button        │   overlaid        │
├──────────────────┴──────────────────────────┴───────────────────┤
│ Metrics strip: win rate · bankroll chart · decision quality     │
└─────────────────────────────────────────────────────────────────┘
```

### Action buttons
Hit, Stand, Double, Split. Each is enabled/disabled based on game legality. Each has two visual states: "agent suggests this" (pulsing outline) and "not the agent's pick" (normal). One-click **Accept recommendation** triggers the suggested action.

### Accessibility
- All interactive elements reachable by keyboard.
- Card suits have text labels in addition to color.
- ARIA live region announces "Dealer shows X, you have Y, agent recommends Z."

---

## 13. Flow Summary

1. User uploads `.env` → key parsed into memory → agent controls unlock.
2. User picks risk profile and clicks **Deal**.
3. Game deals; state object built; agent called; recommendation + matrix cell rendered.
4. User accepts or overrides; action applied; if more player decisions remain, loop back to step 3.
5. Dealer plays out; result settled; metrics updated; charts re-render.
6. **Deal next** restarts at step 3 with the same shoe until the cut card triggers a reshuffle.

---

## 14. Testing Checklist

- `.env` with quoted values, comments, blank lines, CRLF line endings → all parse correctly.
- No key loaded → agent call button is disabled with a clear tooltip.
- LLM returns malformed JSON → fallback to Basic Strategy, warning surfaced, game continues.
- LLM returns illegal action (e.g., split on non-pair) → same fallback path.
- Split hand → each sub-hand gets its own recommendation call.
- Reshuffle mid-session → round record not corrupted, metrics unaffected.
- Risk profile change between hands → next recommendation reflects the new profile (verifiable by reading the outgoing request in devtools).
- Reset session → metrics zeroed, bankroll restored, shoe reshuffled.

---

## 15. Out of Scope (v1)

Insurance, surrender, resplits, side bets, multi-hand play, true-count card counting, server-side logging, login, persistence of any kind, mobile-native packaging.
