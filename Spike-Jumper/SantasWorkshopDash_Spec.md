# 🎅 Santa's Workshop Dash
### Game Design Specification Document

---

## 1. Game Overview

Santa's Workshop Dash is a desktop-only, endless runner game in the style of Geometry Dash. The player controls Santa Claus as he sprints through his North Pole workshop on Christmas Eve, jumping over obstacles while a Gifts Delivered counter ticks up. The game gets progressively faster over time, and obstacles are synced to the beat of the background music. One hit ends the run.

---

## 2. Core Mechanic

### 2.1 Movement
- Santa runs automatically from left to right at a constant horizontal speed.
- The player presses and holds **SPACEBAR** to jump. The longer SPACE is held, the higher and longer Santa stays in the air.
- Releasing SPACE causes Santa to descend back to the ground at a natural gravity rate.
- This hold-to-jump mechanic requires the player to judge obstacle height and width, deciding how long to hold for each unique obstacle.
- No double-jump or mid-air control beyond releasing the spacebar early.

### 2.2 Speed Progression
- The game starts at a moderate run speed.
- Speed gradually increases the longer the player survives, creating escalating difficulty.
- The background music also intensifies as speed increases, reinforcing the urgency.

### 2.3 Collision & Death
- One hit from any obstacle ends the run immediately — no lives, no second chances.
- On death, the Death Screen is triggered (see Section 10).

---

## 3. Scoring
- The primary score metric is **Gifts Delivered** — a counter displayed prominently on-screen during gameplay.
- The counter ticks up continuously while the player is alive. As the game speeds up, the counter ticks faster.
- A **Best Score** (highest Gifts Delivered ever achieved) is tracked and persisted across sessions.
- The current score and best score are both displayed on the Death Screen.

---

## 4. World Layout & Sections

The world is divided into three repeating sections that cycle infinitely in order. Each section has a unique background, floor, obstacles, and atmosphere. A decorative sign appears at the entrance of each new section announcing its name.

### 4.1 Section 1 — The Workshop Floor
- **Floor:** Wooden workshop floorboards.
- **Entry Sign:** *"Santa's Workshop"*
- **Background Details:** Conveyor belts moving toys, elves working at benches, toy soldiers marching, candy canes lining the walls, giant decorated Christmas trees, shelves stacked with colorful gifts, workbenches with tools and half-built toys.
- **Obstacles:** Wrapped presents (short), stacks of toy boxes (tall), toys scattered on the floor (medium), elves running across the path (variable height).

### 4.2 Section 2 — Mrs. Claus's Kitchen
- **Floor:** Warm kitchen tile floor.
- **Entry Sign:** *"Mrs. Claus's Kitchen"*
- **Background Details:** Mrs. Claus waving cheerfully at Santa and holding a tray of cookies, ovens, baking counters, hanging stockings, jars of candy, and festive decorations.
- **Obstacles:** Cookie trays dropped on the floor (short/wide), stacked cookie jars (tall), rolling pins (low and fast), pies cooling on the floor (short and wide).

### 4.3 Section 3 — The Reindeer Stables & Sleigh Bay
- **Floor:** Snowy path with light snow on the ground.
- **Entry Sign:** *"Reindeer Stables"*
- **Background Details:** Reindeers lined up in their stalls (Dasher, Dancer, Prancer, etc.), the gleaming red sleigh loaded with gifts, snowy exterior walls, icicles hanging from above, a starry night sky visible through windows.
- **Obstacles:** Hay bales (medium/tall), reindeer feed buckets (short), reindeer saddle equipment (variable height), small snowdrifts (short and wide).

---

## 5. Obstacles
- Obstacles vary in both **height** (short, medium, tall) and **width** (narrow, wide) to make full use of the hold-to-jump mechanic.
- Obstacles are section-specific (see Section 4).
- Obstacles are spawned at precise timing intervals synced to the background music beat (see Section 6.2).
- Spawn patterns get denser and faster as the game speed increases.
- Obstacles scroll in from the right side of the screen and move leftward at the game's current speed.

---

## 6. Audio

### 6.1 Background Music
- A jolly, upbeat Christmas track plays throughout the game.
- The music intensifies (tempo or arrangement layers up) as the game speed increases, building urgency.
- Music loops seamlessly.
- Music plays from the main menu and continues into gameplay.

### 6.2 Music-Synced Obstacle Spawning
- Obstacles spawn at precise moments tied to the music beat using the **Web Audio API**.
- The beat timeline is pre-mapped so that obstacles arrive at the player's position on beat hits.
- As music tempo increases with game speed, obstacle spawn timing tightens accordingly.
- This creates a rhythm-game feel layered on top of the endless runner.

### 6.3 Sound Effects
- **Jump:** A light whoosh when Santa leaves the ground.
- **Land:** A soft thud when Santa lands.
- **Collision/Death:** A festive crash or "oof" sound.
- **Section Transition:** A brief jingle as the section sign appears.

---

## 7. Santa — Player Character

### 7.1 Animation
- Santa has a running animation while on the ground.
- Santa's **hat bounces** with his running movement — a small idle animation detail.
- Santa has a jump pose while airborne.
- Santa has a death/crash animation on collision.

### 7.2 Costume System
The player can select a Santa costume from the Main Menu before starting. All costumes are purely cosmetic — no gameplay differences.

| Costume | Description |
|---|---|
| 🎅 Classic Santa | Traditional red suit, black belt, white trim |
| 🌺 Hawaiian Santa | Floral shirt, lei around neck, Santa hat |
| 🏖️ Swimsuit Santa | Board shorts, sunglasses, Santa hat |
| 🤠 Cowboy Santa | Cowboy hat, vest, boots, sheriff star |
| 🥷 Ninja Santa | Black ninja outfit, Santa hat |
| 👔 Business Santa | Suit and tie, briefcase, Santa hat |

---

## 8. Main Menu
- Title: **"Santa's Workshop Dash"** displayed prominently in festive typography.
- An animated preview of Santa running plays in the background behind the menu.
- Best Score is displayed on the main menu.
- A **"Change Costume"** button opens a costume selector panel showing all 6 Santa variants. The selected costume is highlighted and the player's selection persists.
- A **"Play"** button starts the game.
- Christmas music plays on the main menu.

---

## 9. In-Game HUD
- **Gifts Delivered** counter — displayed prominently (top-center or top-right). Ticks up continuously, speeds up with game speed.
- **Best Score** — displayed smaller beneath or beside the current score for reference.
- **Current Section** — small label showing which section Santa is currently in.
- HUD should be minimal and non-intrusive to keep focus on the action.

---

## 10. Death Screen
- Triggered immediately on collision with an obstacle.
- **Death Animation:** A burst of presents fly in from all sides of the screen, collide in the center, and explode in a festive particle burst (wrapping paper, ribbons, and bows scattering everywhere).
- After the animation resolves, the Death Screen displays:
  - **"Santa Crashed!"** headline.
  - Gifts Delivered this run.
  - Best Score (updated if new record — with a **"New Record! 🎉"** callout if so).
  - **"Try Again"** button — restarts the run immediately.
  - **"Main Menu"** button — returns to the main menu.

---

## 11. Technical Notes
- **Platform:** Desktop browser only (keyboard input, no touch/mobile support).
- **Input:** Spacebar only for jump mechanic.
- **Rendering:** HTML5 Canvas or a lightweight game framework (e.g. [Phaser.js](https://phaser.io)) recommended.
- **Audio:** Web Audio API for music playback and beat-synced obstacle timing.
- **Persistence:** `localStorage` to save Best Score and selected costume across sessions.
- **Scrolling Background:** Parallax layers recommended — background elements scroll slower than foreground to create depth.
- **Obstacle System:** Each obstacle type should have defined `height` and `width` properties used for both rendering and collision detection.

---

*— End of Specification —*
