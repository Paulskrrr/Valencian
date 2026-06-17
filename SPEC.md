# SPEC.md — Valencian: Blackjack Card Counting Trainer

A single-file, offline, browser-based blackjack trainer that teaches **Hi-Lo card
counting** and **perfect basic strategy** through real play. No ads, no server,
no tracking. Runs by opening `index.html` in any browser.

This document describes the **current state** of the project and the **immediate
next development step**. It is a living reference, not a build log. Prior phase
history is irrelevant; what matters is what exists now and where it goes next.

---

## The Vision

Sitting across from a dealer at a proper table in a 1950s jazz lounge — warm
lamplight, a little smoky, unhurried. A real dealer figure, a genuine felt table,
a room that exists behind it all. The trainer aspect is woven in subtly: counts
and hints are available but never intrusive. Aesthetic: **warm pixel art** —
rich, readable, atmospheric. Not Vegas neon, not cold sci-fi. Cards and chips
feel physically present.

---

## Non-goals (permanent — do not violate)

- No server, no accounts, no network calls at runtime.
- **No external libraries, frameworks, build tools, or npm.** Everything is
  hand-written vanilla JS in a single file.
- No monetisation, ads, or tracking of any kind.
- No 3D card animations or physics — pixel art, flat, readable.

---

## Architecture (as built)

- **Stack:** Vanilla HTML + CSS + JavaScript. Single `index.html` with embedded
  `<style>` and `<script>`. No dependencies.
- **Canvas:** 640x360 internal resolution (`CW`/`CH`), CSS-scaled to fill the
  viewport at 16:9. `image-rendering: pixelated`. The canvas renders everything
  except the stats and settings panels (which are HTML overlays).
- **Background:** baked in as a base64 data-URI constant `BG_SRC`. Swap the scene
  by replacing that string. Fallback when empty: flat `ROOM = '#1A1008'`.
- **Sound:** Web Audio API, synthesized procedurally. No audio files.
- **Persistence:** `localStorage` — `bj_alltime_v1` (stats), `bj_settings_v1`
  (config).
- **Render loop:** `scheduleRender()` drives draws; animations re-schedule
  themselves while active. `prefers-reduced-motion` collapses animations to
  instant cuts.

### Layout geometry (constants)
```
CW = 640, CH = 360
FELT_TOP = 178, FELT_H = 216   (felt y=178 to y=394, clipped at canvas bottom)
DEALER_Y = 156                  (dealer cards)
PLAYER_Y = 296                  (player cards)
SHOE_X = 348, SHOE_Y = 108      (shoe widget, beside the dealer figure)
```
Top ~50% is the **room zone** (background shows through; dealer figure + shoe
live here). Bottom ~50% is the **table zone** (felt oval, mahogany rail, brass
trim; dealer cards inside the felt top, player cards lower; chip bar + bet
display at the felt bottom). Action buttons are HTML below the canvas.

### Draw order (back to front, in the main render)
room background -> dealer figure -> felt/rail -> shoe -> cards -> chips ->
chip bar -> banners/overlays.

---

## Current State — What's Built

### Blackjack Engine
| Feature | Detail |
|---|---|
| Decks | Configurable 1-8 |
| Dealer rule | H17 or S17 |
| BJ payout | 3:2 / 6:5 / 1:1 |
| Surrender | Late / None |
| DAS | On/Off |
| Re-split | Up to 3x |
| Split aces | One card each (configurable) |
| Double | Any two cards, or 9-10-11 only |
| Penetration | 50-90% configurable |
| Dealer peek | US rules, on by default, configurable |
| Insurance | Offered on dealer Ace; settled correctly in peek & no-peek |

### Hi-Lo Counting
Running count updates as each card reveals. True count = RC / decks remaining
(rounded to nearest 0.5, min 0.5). Resets on shuffle. Overlay toggled with [C]
or the RC button.

### Strategy Engine
Tables: `PAIRS`, `SOFT_H17`, `SOFT_S17`, `HARD`, `SURRENDER_H17`,
`SURRENDER_S17`. Hint engine highlights the recommended action button (star
toggle, [I]); wrong decisions show a one-line reason. Strategy accuracy tracked
per category (hard / soft / pairs / surrender). Bet-spread coaching:
`suggestedBet = betUnit * max(1, TC-1)`, shown alongside the bet when hints on.

### Dealer Figure (current = placeholder)
An animated **brass automaton** drawn procedurally with canvas 2D primitives in
`drawDealer()`. A pose state machine drives it:
- `DEALER_POSE = { IDLE, DEALING, WIN, LOSE }`, `dealerPose` holds current.
- `setDealerPose(to, dur)` / `setDealerPoseWith(to, returnAfter, dur)` transition
  and (optionally) auto-return to idle.
- `dealerPoseAnim` holds the active tween; `_getPoseArms()` lerps arm angles and
  `_getPoseEye()` switches eye colour at the tween midpoint.
- Pose tables: `POSE_ARMS` (arm angles L/R per pose), `POSE_EYE` (eye colour per
  pose). Eye colours: idle amber `#F0B83C`, dealing warm white `#FFE9B0`, win
  red `#E0503A`, lose green `#6FBF5A`.
- `drawDealer()` seats the figure via a save/translate/scale(1.7)/translate so
  its base meets the felt top, centred at x~290.

The robot is explicitly a placeholder. **The pose state machine is the stable
contract; only the rendering of the figure changes.**

### Cards
Procedural: warm off-white face `#F5F0E8`, pixel diamond back, Unicode suit
glyphs, red suits `#C41E3A`. Animations: slide from shoe, flip reveal (scale-X),
bust shake, blackjack gold flash.

### Chips
Denominations 1 / 5 / 10 / 25 / 100. Drawn on the felt (hidden HTML buttons
retained only for event wiring; canvas hit-testing calls `addChip()`/`clearBet()`).
Win payouts animate as a chip-stack arc. Chip bar at felt bottom: five circles
(centres `[155,220,285,350,415]`, r=16), CLEAR button, `BET: X` label; dims to
30% when not betting.

### Shoe & Shuffle
Shoe widget at `SHOE_X=348, SHOE_Y=108`. **Shuffle** control runs
`manualShuffle()` (rebuild shoe, reset count, 1200ms amber flicker). Auto-shuffle
fires at the penetration threshold inside `deal()`.

### Sound (procedural)
Deal (swept noise burst + sine thump, staggered), win (rising warm arpeggio),
lose (falling minor tone), blackjack (bright distinct win). Mute [M], persisted.

### Stats / Settings / Insurance UI
Stats: session + all-time side by side — hands, win/loss/push, blackjacks,
strategy accuracy by category, bankroll delta, sparkline; export/import JSON;
new-session and reset-all. Settings: all table rules + feature toggles, persisted
to `bj_settings_v1`, applied immediately or on next deal/shuffle. Insurance:
plain text `Insurance? Yes / No`, keys [Y]/[N].

### Controls
Space/Enter deal/advance . H hit . S stand . W double . P split . R surrender .
Y/N insurance . C count . I hints . M mute . T stats . G settings.

### Palette
Felt `#2D5A3D` . rail `#8B4513` . brass trim `#D4A017` . room `#1A1008` . card
face `#F5F0E8` . card back/red `#C41E3A` . gold `#D4A017`.

---

## NEXT DEVELOPMENT STEP — Human dealer sprite + procedural idle

Replace the placeholder brass robot with a **human pixel-art dealer sprite** (a
1950s lounge croupier, warm tones, consistent with the background), and give it
subtle life through **code-driven micro-animation** — no new drawn frames beyond
a small set of pose images.

This is a **rendering-only change.** Do NOT touch the engine, counting, strategy,
stats, settings, sound, or the pose state machine. The contract is: `drawDealer()`
and its supporting draw helpers are rewritten; everything that *calls into* the
pose system (`setDealerPose`, `setDealerPoseWith`, `DEALER_POSE`, the
win/lose/dealing triggers already wired in `deal()` and settlement) stays exactly
as is.

### Asset model
- The dealer is supplied as **pose images with transparent backgrounds**, loaded
  the same way `BG_SRC` is — as base64 data-URI constants (e.g. `DEALER_IDLE_SRC`,
  `DEALER_DEALING_SRC`, `DEALER_WIN_SRC`, `DEALER_LOSE_SRC`). The human will
  provide these. Until they exist, fall back to the current procedural robot so
  the game never breaks (guard on whether the sprite constant is non-empty,
  mirroring the `BG_SRC` fallback pattern).
- Optionally a separate **eyes-closed** variant of the idle image
  (`DEALER_BLINK_SRC`) for blinking. If absent, skip blinking gracefully.
- Images are drawn with `ctx.drawImage` seated using the **same transform the
  current `drawDealer()` uses** (centre x~290, scaled so the figure's base meets
  `FELT_TOP`). Keep pixel crispness — no smoothing
  (`ctx.imageSmoothingEnabled = false` around the draw).

### Pose -> image mapping
Map the existing `DEALER_POSE` states to images:
- `IDLE` -> `DEALER_IDLE_SRC`
- `DEALING` -> `DEALER_DEALING_SRC`
- `WIN` -> `DEALER_WIN_SRC`
- `LOSE` -> `DEALER_LOSE_SRC`
During a `dealerPoseAnim` transition, cross-fade or hard-swap at the tween
midpoint (reuse the existing midpoint logic from `_getPoseEye()` — same timing,
so transitions stay consistent with the rest of the figure). Arm-angle lerping
(`_getPoseArms`) is no longer needed for a sprite; it can be ignored or removed,
but the pose *timing* and triggers must be preserved.

### Procedural idle animation (the "life")
Applied as transforms on the sprite each frame inside the draw — no extra art:
- **Breathing:** a slow vertical sine on the sprite's y (or a tiny scaleY around
  the torso). Suggested: `Math.sin(now * 0.0018) * 1.5` px amplitude. Subtle.
- **Sway:** a second, slower sine on x or a tiny rotation, offset in frequency so
  it doesn't look mechanical, e.g. `Math.sin(now * 0.0011) * 0.8` px.
- **Blink:** if `DEALER_BLINK_SRC` exists, swap to it for ~110ms on a random
  interval of 2.5-6s, only while in `IDLE`. If absent, skip.
- All idle motion must **pause/instant-settle under `prefers-reduced-motion`**,
  consistent with the rest of the animation system.
- Idle motion applies during `IDLE`; during action poses (`DEALING`/`WIN`/`LOSE`)
  it can continue subtly or pause — pick whatever reads calmest, but never let it
  fight the pose.

### Acceptance
- With sprite constants empty: game runs identically to now (robot fallback).
- With sprite constants populated: the human dealer renders seated correctly at
  the felt edge, crisp pixels, breathing + swaying gently when idle, blinking if
  a blink frame is supplied, and changing pose on deal/win/lose exactly when the
  robot did — because the triggers are untouched.
- No change to any game logic, timing of rounds, or other animations.

### Build protocol
1. Read this SPEC and `STRATEGY.md` fully before editing.
2. Confirm understanding of the current `drawDealer()` / pose system and state
   the plan **before** writing code; wait for go-ahead.
3. Make the rendering change only. Keep the robot fallback intact.
4. After implementing: note what changed and how to test it (including how to
   drop in the dealer image constants), then stop for feedback.
5. Never add libraries, network calls, or build steps. Single file stays single.

---

## Later / Open Items (not now)

- **Multiple dealers** (robot, human variants) selectable — the sprite model
  above is designed to extend to this: more constant sets + a picker. Defer until
  the single human dealer works.
- **Background polish:** subtle animated detail (smoke curl, lamp flicker) if
  performance allows.
- **Index deviations:** setting is toggleable but not yet wired into the hint
  engine (Fab 4 surrenders, Insurance at TC>=3, 16v10 stand at TC>=0, etc.).
- **Mobile/touch:** chip-bar targets are 32px circles — usable, could enlarge on
  phones.
- **Canvas sparkline:** currently a separate HTML canvas in the stats panel.
