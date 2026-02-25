# 🃏 Dreamy Blackjack — Game Specification

## Overview

A single-file HTML/CSS/Vanilla JS blackjack game with a soft, dreamy pastel aesthetic. The game features full casino-standard blackjack rules, rich animations, a progression system with unlockable table themes, achievement badges, and a streak counter — all wrapped in a pink and blue visual identity.

---

## Tech Stack

- **Single file:** `blackjack.html`
- **Languages:** HTML5, CSS3, Vanilla JavaScript (no frameworks, no dependencies)
- **Persistence:** `localStorage` for balance, unlocked themes, and achievements across sessions

---

## Visual Design

### Color Palette (Default Theme)
| Role | Color |
|---|---|
| Background | `#fce4ec` (soft blush pink) |
| Table felt | `#e8eaf6` (periwinkle blue) |
| Primary accent | `#f48fb1` (pastel pink) |
| Secondary accent | `#90caf9` (pastel blue) |
| Card background | `#ffffff` |
| Card suits (hearts/diamonds) | `#e91e63` |
| Card suits (spades/clubs) | `#1a237e` |
| Text primary | `#4a148c` (deep purple) |
| Button default | `#f48fb1` |
| Button hover | `#f06292` |
| Win glow | `#f48fb1` |
| Loss glow | `#90caf9` |

### Typography
- **Font:** Google Fonts — `Quicksand` (rounded, soft, friendly)
- **Headings:** `Quicksand Bold`
- **Body/UI:** `Quicksand Medium`
- **Card values:** `Georgia` or `serif` for classic feel

### Layout
- Centered single-column layout, max width `900px`
- Top bar: balance display + streak counter + theme selector button
- Middle: dealer hand area → table felt → player hand area
- Bottom: bet panel (chip UI) + action buttons
- Fixed footer: win/loss history tracker

---

## Unlockable Table Themes

Each theme changes the table felt color, background, and accent colors. Themes are unlocked by reaching balance milestones and persist via `localStorage`.

| Theme | Unlock Condition | Felt Color | Background |
|---|---|---|---|
| Default (Pink & Blue) | Always unlocked | `#e8eaf6` periwinkle | `#fce4ec` blush |
| Lavender | Reach $7,500 | `#e1bee7` lavender | `#f3e5f5` lilac |
| Rose Gold | Reach $10,000 | `#ffccbc` rose | `#fbe9e7` peach |
| Mint Green | Reach $15,000 | `#c8e6c9` mint | `#f1f8e9` soft green |
| Sunset Coral | Reach $20,000 | `#ffccbc` coral | `#fff3e0` warm cream |
| Midnight Blue | Reach $25,000 | `#1a237e` navy | `#0d0d2b` deep navy |

A **"Themes"** button in the top bar opens a theme picker modal showing locked/unlocked status for each theme with a soft lock icon on locked ones.

---

## Game Setup

### Starting State
- Player begins with **$5,000**
- A standard **52-card deck** is used (can be extended to multi-deck in future)
- Deck is **shuffled** at game start and **reshuffled automatically** when fewer than **15 cards** remain
- Player enters no nickname — game is anonymous

### Card Representation
Each card is rendered as a styled `div` with:
- Large rank in top-left and bottom-right corners
- Large suit symbol in the center
- Red for hearts/diamonds, dark navy for spades/clubs
- Rounded corners (`border-radius: 12px`)
- Subtle drop shadow
- White background with a faint pastel border matching the current theme accent

---

## Betting System

### Chip Denominations
Chips are displayed as circular buttons at the bottom of the screen:

| Chip | Color |
|---|---|
| $5 | Soft pink |
| $10 | Pastel blue |
| $25 | Lavender |
| $50 | Mint |
| $100 | Rose gold |
| $500 | Sunset coral |

### Bet Rules
- Clicking a chip **adds** that value to the current bet
- A **"Clear Bet"** button resets the current bet to $0
- A **"Deal"** button confirms the bet and starts the round
- Player **cannot bet more than their current balance** — chips exceeding the balance are visually greyed out and non-clickable
- Minimum bet is **$5**
- Current bet is displayed prominently in a styled "chip stack" display above the chip row, showing the total bet amount in large text

---

## Core Gameplay — Blackjack Rules

### Deal
- Player receives 2 cards face-up
- Dealer receives 1 card face-up, 1 card face-down (hole card)

### Player Actions (shown as buttons, contextually enabled/disabled)

| Action | Condition |
|---|---|
| **Hit** | Always available during player turn |
| **Stand** | Always available during player turn |
| **Double Down** | Only on first action (2 cards); player doubles the bet and receives exactly 1 more card |
| **Split** | Only if both cards share the same rank; splits into 2 hands, each gets a new card; both hands are played independently |
| **Insurance** | Only offered when dealer's face-up card is an Ace; costs half the original bet; pays 2:1 if dealer has blackjack |

### Double Down on Split
- Allowed on either or both split hands
- Player doubles the bet on that specific split hand and receives 1 card

### Dealer Rules
- Dealer reveals hole card after player stands/busts
- Dealer **hits on soft 17** (Ace + 6)
- Dealer **stands on hard 17 and above**

### Payouts
| Outcome | Payout |
|---|---|
| Win | 1:1 (even money) |
| Blackjack (natural) | 3:2 |
| Push (tie) | Bet returned |
| Insurance win | 2:1 on insurance bet |
| Lucky 7-7-7 | 2x total payout (see below) |

### Bust
- Player busts over 21 → round ends, player loses bet immediately
- Dealer busts → player wins

### Split Blackjack
- A blackjack achieved on a split hand counts as **21**, not a natural blackjack — pays 1:1, not 3:2

---

## Lucky 7-7-7 Bonus

- If the player's first 3 cards are all **7s** (any suit), a **Lucky 7-7-7** event triggers
- The normal win payout is **doubled (2x)**
- A special full-screen animation plays: golden 7s rain down over the table with sparkle effects in pastel gold
- An achievement badge is awarded (see Achievements)
- This applies even on split hands if the condition is met

---

## Animations & Transitions

All animations should feel smooth, soft, and satisfying — never jarring.

### Card Deal — Slide & Flip
- Cards animate from the top-center "deck" position, sliding to their target position
- As they arrive, they perform a CSS `rotateY` flip from face-down to face-up
- Duration: `0.4s` ease-out per card, staggered `0.2s` between cards

### Dealer Hole Card Reveal
- When the dealer reveals their hidden card, it performs a dramatic `rotateY` flip
- A subtle pulse/glow effect in the accent color plays on the card as it flips
- Duration: `0.6s`

### Chip Click Feedback
- Clicking a chip makes it **bounce** (scale up then down) with a spring-like feel
- The bet total counter animates (number ticks up) using a CSS counter animation
- Duration: `0.2s`

### Win/Loss Table Glow
- On win: the table felt pulses with a soft **pink glow** (`box-shadow`)
- On loss: the table felt pulses with a soft **blue glow**
- On push: a neutral **white/silver glow**
- Duration: `0.8s` fade in and out

### Balance Odometer
- When balance changes (win or loss), the number **ticks up or down** digit by digit
- Uses a CSS animation on individual digit spans
- Duration: `0.5s`

### Button Hover
- All action buttons have a smooth `0.2s` color transition on hover
- Slightly increase in scale (`transform: scale(1.05)`) on hover
- Active/pressed state scales down slightly (`scale(0.97)`)

### Card Shimmer on Deal
- A subtle shimmer/gloss sweep passes across each card as it's dealt
- Implemented as a pseudo-element `::after` with a linear gradient animation
- Duration: `0.6s` per card

### Confetti Burst on Win
- On any win, pastel pink and blue confetti particles burst from the top of the screen
- On blackjack, confetti is more intense and lasts longer
- On Lucky 7-7-7, gold and pastel confetti with sparkle icons
- Implemented in vanilla JS with canvas or animated `div` particles

### Achievement Badge Pop-up
- When an achievement unlocks, a badge animates in from the top-right corner
- Slides in, bounces slightly, holds for `3s`, then slides back out
- Soft pastel card design with an icon and badge name

### Streak Counter Update
- When streak increments, the streak number does a quick scale bounce
- Flame/star emoji pulses beside the number

### Game Over Screen
- Full-screen overlay fades in with a soft blur backdrop
- Shows final balance, achievements earned, highest streak
- A **"Play Again"** button resets balance to $5,000 and clears the session (but keeps unlocked themes and achievements)

---

## Edge Cases & Rules

| Scenario | Behavior |
|---|---|
| Player balance hits $0 | Game Over screen appears immediately after the round ends |
| Player tries to bet more than balance | Chips over remaining balance are greyed out and unclickable |
| Bet would exceed balance mid-click | Bet is capped at current balance |
| Push/tie | Player's bet is returned in full |
| Dealer has Ace showing | Insurance prompt appears before dealer checks for blackjack |
| Dealer has blackjack + player has blackjack | Push — both bets returned, insurance pays if taken |
| Player splits and gets blackjack on split hand | Counts as 21, not natural blackjack (1:1 payout) |
| Double down on split hand | Allowed — doubles that hand's bet, player receives exactly 1 more card |
| Deck runs below 15 cards | Deck is automatically reshuffled before the next round with a subtle shuffle animation |
| Player hits 21 exactly | Hand is auto-stood (no further hit option shown) |
| Player busts on split hand | That hand loses; other hand continues as normal |

---

## Streak Counter

- Displayed in the top bar with a flame or star emoji
- Increments by 1 on every **win** (including blackjack and Lucky 7-7-7)
- Resets to 0 on any **loss** or **push**
- On a streak of 5+, the counter glows and pulses with a warm pink color
- All-time best streak is saved to `localStorage` and shown on the Game Over screen

---

## Achievement Badges

Badges are displayed in a small panel accessible from a button in the top bar. Locked badges show a silhouette with a lock icon.

| Badge | Name | Unlock Condition | Icon |
|---|---|---|---|
| 🃏 | First Blackjack | Get your first natural blackjack | Card icon |
| 🔥 | On Fire | Win 5 hands in a row | Flame |
| 💸 | Big Spender | Place a single bet of $1,000 or more | Money bag |
| 💎 | High Roller | Reach a balance of $10,000 | Diamond |
| 🦋 | Comeback Kid | Win a hand with a balance under $100 | Butterfly |
| 7️⃣ | Lucky Sevens | Hit Lucky 7-7-7 | Seven |

- Badge unlock triggers the badge pop-up animation (slides in from top-right)
- Earned badges are saved to `localStorage` permanently (persist across sessions)
- On the Game Over screen, all earned badges are displayed

---

## Win/Loss History Tracker

- A collapsible panel fixed at the bottom of the screen
- Shows the last **20 rounds** in a scrollable row
- Each round is represented by a small chip icon:
  - 🟢 Green chip = Win
  - 🔴 Red chip = Loss
  - 🟡 Yellow chip = Push
- Hovering over a chip shows a tooltip with: round result, amount won/lost, hand summary
- Panel can be toggled open/closed with a smooth slide animation

---

## Game States & Button Visibility

| State | Visible Buttons |
|---|---|
| Betting phase | Chip denominations, Clear Bet, Deal |
| Player's turn (first action) | Hit, Stand, Double Down, Split (if eligible) |
| Player's turn (after hit) | Hit, Stand |
| Split hand active | Hit, Stand, Double Down (if eligible) |
| Insurance prompt | Take Insurance, Decline Insurance |
| Round over | New Round |
| Game over | Play Again |

---

## localStorage Persistence

The following data persists across browser sessions:

| Key | Value |
|---|---|
| `bj_balance` | Player's current balance |
| `bj_themes_unlocked` | Array of unlocked theme names |
| `bj_active_theme` | Currently selected theme |
| `bj_achievements` | Array of earned achievement IDs |
| `bj_best_streak` | All-time highest win streak |

---

## File Structure

Since this is a single HTML file, the internal structure should be organized as follows:

```
blackjack.html
├── <head>
│   ├── Google Fonts (Quicksand)
│   └── <style> — all CSS including themes, animations, responsive styles
└── <body>
    ├── Top bar (balance, streak, themes button, achievements button)
    ├── Dealer area (dealer hand, dealer score)
    ├── Table felt (result message, deck indicator)
    ├── Player area (player hand(s), player score(s))
    ├── Bet panel (chip UI, current bet display)
    ├── Action buttons
    ├── History tracker (collapsible)
    ├── Modals (theme picker, achievements, game over)
    └── <script> — all JavaScript game logic
```

---

## Responsive Design

- Fully playable on desktop (primary target)
- Cards and chips scale down gracefully on tablet
- Mobile: stack layout vertically, chips wrap to 2 rows, buttons become full-width

---

## Future Enhancements (Out of Scope for v1)

- Multi-deck shoe (4 or 6 decks)
- Dealer avatar/character with speech bubbles
- Sound effects
- Multiplayer or leaderboard
- Time-of-day ambient background shifts
- Mystery chip mechanic
