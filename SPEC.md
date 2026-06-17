# SPEC.md — Blackjack Card Counting Trainer

A single-file, offline, browser-based blackjack trainer built to teach **Hi-Lo
card counting** and **perfect basic strategy** through real play. No ads, no
server, no tracking. Runs by opening `index.html` in any browser.

This document reflects the current state of the project and the direction it is
heading. It is a living reference, not a construction checklist.

---

## The Vision

The experience should feel like sitting across from a dealer at a proper table
in a 1950s jazz lounge — warm lamplight, a little smoky, unhurried. The player
faces a real dealer figure, a genuine felt table, and a room that exists behind
it all. The trainer aspect is woven in subtly: counts and hints are available
but never intrusive.

The aesthetic is **warm pixel art**: rich, readable, atmospheric. Not garish
Vegas neon, not cold sci-fi. Cards and chips should feel physically present.

---

## Architecture

- **Stack:** Vanilla HTML + CSS + JavaScript. No frameworks, no build step, no npm.
- **Single file:** `index.html` with embedded `<style>` and `<script>`. The background
  image is baked in as a base64 data-URI constant (`BG_SRC`).
- **Canvas:** 640×360 internal resolution, CSS-scaled 16:9. `image-rendering: pixelated`.
  The canvas renders everything except menus/panels (stats, settings).
- **Sound:** Web Audio API, synthesized procedurally — no external audio files.
- **Persistence:** `localStorage`. Keys: `bj_alltime_v1` (stats), `bj_settings_v1`
  (table config).
- **No network calls at runtime.**

---

## Current State — What's Built

### Layout
The canvas is divided into two zones:

- **Room zone (top ~50%):** The jazz lounge background image shows through here.
  The dealer figure and shoe widget live in this zone, immediately above the felt.
- **Table zone (bottom ~50%):** Felt oval with mahogany rail and brass trim.
  Dealer cards sit just inside the top of the felt; player cards in the lower
  half. The chip bar and bet display sit at the bottom of the felt. Action
  buttons are HTML elements below the canvas.

Constants that define this geometry:
```
CW = 640, CH = 360
FELT_TOP = 178, FELT_H = 216  (felt runs from y=178 to y=394, clipped at canvas bottom)
DEALER_Y = 156                 (dealer cards)
PLAYER_Y = 296                 (player cards)
SHOE_X = 348, SHOE_Y = 108    (shoe widget, adjacent to the dealer figure)
```

### Background
A pixel-art jazz lounge scene is hardcoded as a base64 webp in the `BG_SRC`
constant. To swap it: replace the string value of `BG_SRC`. The fallback (when
`BG_SRC` is empty) is a flat dark warm brown fill (`ROOM = '#1A1008'`).

### Dealer Figure
An animated brass automaton drawn procedurally with canvas 2D primitives.
Four poses: `idle`, `dealing`, `win`, `lose`. Arms and eye colour lerp smoothly
between poses via `dealerPoseAnim`.

The robot is a **placeholder**. The intended final state is a human pixel-art
dealer sprite. The pose state machine (`DEALER_POSE`, `setDealerPose`,
`setDealerPoseWith`) will remain; only the draw function changes.

Eye colours by pose:
- Idle: warm amber `#F0B83C`
- Dealing: focused warm white `#FFE9B0`
- Win (dealer won): warm red `#E0503A`
- Lose (dealer lost): green `#6FBF5A`

### Cards
Drawn procedurally: warm off-white face `#F5F0E8`, pixel diamond back pattern.
Suits as Unicode glyphs. Red suits use `CARD_RED = '#C41E3A'`.

Animations: slide from shoe, flip reveal (scale-X interpolation), bust shake,
blackjack gold flash.

### Chips
Denominations: 1 / 5 / 10 / 25 / 100.

Chips are drawn on the canvas felt (not HTML buttons). The HTML chip buttons
remain in the DOM (hidden) for their event-listener wiring; canvas click
hit-testing calls the same `addChip()` / `clearBet()` functions.

Win payouts animate as a chip-stack arc from dealer zone to bet area.

### Chip Bar (canvas)
Drawn at the bottom of the felt. Five coloured circles, a CLEAR button, and
a `BET: X` label above. Dims to 30% opacity when not in a betting phase.

Chip positions (center X): `[155, 220, 285, 350, 415]`, radius 16px.

### Shoe & Shuffle
The shoe widget sits at `SHOE_X=348, SHOE_Y=108` — adjacent to the dealer,
within the room zone. A **Shuffle** button in the controls triggers
`manualShuffle()`: rebuilds the shoe, resets the count, and plays a 1200ms
amber flicker animation on the shoe widget. Auto-shuffle still triggers at
the configured penetration threshold inside `deal()`.

### Blackjack Engine

| Feature | Detail |
|---|---|
| Decks | Configurable 1–8 |
| Dealer rule | H17 or S17 (configurable) |
| BJ payout | 3:2 / 6:5 / 1:1 |
| Surrender | Late / None |
| DAS | On/Off |
| Re-split | Up to 3× |
| Split aces | One card each (configurable) |
| Double | Any two cards (configurable) or 9-10-11 only |
| Penetration | 50–90% configurable |
| Dealer peek | US rules — dealer checks for BJ on 10/A; configurable, on by default |
| Insurance | Offered on dealer Ace; settled correctly in peek and no-peek modes |

### Hi-Lo Counting
Running count updates as each card is revealed. True count = RC ÷ decks
remaining (rounded to nearest 0.5, minimum 0.5). Count resets on shuffle.
Toggle overlay with [C] or the RC button.

### Strategy Engine
Tables: `PAIRS`, `SOFT_H17`, `SOFT_S17`, `HARD`, `SURRENDER_H17`, `SURRENDER_S17`.
Hint engine highlights the recommended action button (★ toggle, hotkey [I]).
Wrong decisions show a one-line reason. Strategy accuracy tracked per category
(hard / soft / pairs / surrender).

Bet-spread coaching: `suggestedBet = betUnit × max(1, TC−1)`. Shown alongside
bet amount when hints are on, always (not separately gated).

### Animations
| Animation | Trigger |
|---|---|
| Card slide | Deal, hit, double, split, dealer draw |
| Card flip | Hole card reveal |
| Screen shake | Bust |
| Gold flash | Player blackjack |
| Chip arc | Win payout |
| Result banner | End of round (WIN / LOSE / PUSH / BLACKJACK / SURRENDER + amount) |
| Dealer pose lerp | Dealing, win, lose, idle transitions |
| Shuffle flicker | Manual or auto-shuffle |

All animations respect `prefers-reduced-motion` (instant cuts).

### Sound (procedural, Web Audio API)
- **Deal:** swept low-pass noise burst + sine thump, staggered per card
- **Win:** rising warm arpeggio
- **Lose:** falling minor tone
- **Blackjack:** distinct bright win sound
- Mute toggle [M], persistent via settings.

### Insurance UI
Plain text: `Insurance? Yes / No` with underlined Y and N.
Keyboard: [Y] takes insurance, [N] declines. No button styling.

### Stats
Session and all-time scopes, side by side. Tracks: hands, win/loss/push,
blackjacks, strategy accuracy by category, bankroll delta, sparkline.
Export/import JSON. New session and Reset all buttons.

### Settings
All settings persisted to `bj_settings_v1`. Applied immediately (feature
toggles) or on next deal/shuffle (table rules).

### Controls & Keyboard
| Key | Action |
|---|---|
| Space / Enter | Deal / advance round |
| H | Hit |
| S | Stand |
| W | Double down |
| P | Split |
| R | Surrender |
| Y / N | Insurance yes / no |
| C | Count overlay |
| I | Hints toggle |
| M | Mute |
| T | Stats panel |
| G | Settings panel |

---

## Colour Palette

| Role | Value |
|---|---|
| Felt | `#2D5A3D` |
| Rail (mahogany) | `#8B4513` |
| Rail trim (brass) | `#D4A017` |
| Room / background fallback | `#1A1008` |
| Card face | `#F5F0E8` |
| Card back / red suits | `#C41E3A` |
| Gold accent | `#D4A017` |

---

## What's Next — Open Items

1. **Human dealer sprite.** Replace the procedural brass robot with a pixel-art
   human dealer. The pose state machine stays; only `drawDealer()` is swapped out.
   Art direction: 1950s lounge croupier, warm tones, consistent with the background.

2. **Background polish.** The current BG is a single static image. Could eventually
   animate subtle details (smoke curl, lamp flicker) if it doesn't hurt performance.

3. **Index deviations.** The setting exists and is toggleable, but deviation
   logic is not yet wired into the hint engine. Illustrative deviations:
   Fab 4 surrenders, Insurance at TC≥3, 16 vs 10 stand at TC≥0, etc.

4. **Mobile / touch.** The canvas scales responsively but touch targets on the
   chip bar are 32px circles — acceptable but could be larger on phones.

5. **Sparkline in canvas.** The session bankroll sparkline is a separate HTML
   canvas in the stats panel. Could be moved to a persistent corner of the main
   canvas if the stats panel ever goes away.

---

## Non-goals (permanent)

- No server, no accounts, no network.
- No external libraries or build tools.
- No monetisation, ads, or tracking of any kind.
- No fancy 3D card animations or physics — pixel art, flat, readable.
