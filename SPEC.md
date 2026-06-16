# SPEC.md — Blackjack Card Counting Trainer

A single-file, offline, browser-based blackjack game with an 8-bit / retro pixel
aesthetic, built to teach **Hi-Lo card counting** and **perfect basic strategy**
through real play. No ads, no monetization, no server, no tracking. Runs by
opening `index.html` in any browser.

This document is the source of truth. Build it in the phases below. **Pause after
each phase** so the human can test in the browser before continuing.

---

## 0. Tech & architecture decisions (locked)

- **Stack:** Vanilla HTML + CSS + JavaScript. No frameworks, no build step, no npm.
- **Rendering:** A single `<canvas>` for the table, cards, chips, and animations.
  DOM/HTML only for menus, toggles, the settings panel, and stats overlays.
- **Pixel look:** `image-rendering: pixelated;` on the canvas. Render at a low
  internal resolution (e.g. 480×270 or 320×180) and scale up with CSS to keep
  crisp pixels. Cards are **drawn procedurally on the canvas** (rects, pips,
  rank glyphs in a bitmap-style font) — do NOT depend on external image files.
- **Fonts:** Use a pixel/bitmap webfont loaded via a `<style>` `@font-face` from a
  CC0/OFL source embedded as a data-URI if possible, OR fall back to a CSS pixel
  font stack. If no font available, draw rank/suit glyphs procedurally. The game
  must work with zero network access.
- **Sound:** Web Audio API. Synthesize the three sounds procedurally (square/
  triangle wave blips) so there are **no external audio files**:
  - card deal/flip: short click/whoosh
  - win: rising arpeggio
  - lose: falling tone
  A master mute toggle is required.
- **Persistence:** `localStorage`. Two scopes:
  - `session` stats (reset when page reloads or via a "New session" button)
  - `allTime` stats (persist across reloads)
  Plus an **Export stats** button that downloads a JSON file, and an **Import**
  that reads one back (this is the "save to my PC" the human wanted, done safely).
- **File layout:** Prefer a single `index.html` with embedded `<style>` and
  `<script>`. If it grows unwieldy, split into `index.html` + `game.js` +
  `style.css` in the same folder — still no build step. Decide based on size.
- **No external network calls of any kind at runtime.**

---

## 1. Core blackjack engine (Phase 1)

Build the rules engine first, headless-ish (logic separated from rendering so it's
testable).

- Shoe of N decks (configurable, see settings). 52 cards per deck, standard ranks.
- Deal flow: player gets 2 cards up, dealer 1 up + 1 hole card.
- Player actions: **Hit, Stand, Double, Split, Surrender** (each respecting the
  configured rules — e.g. Surrender only if enabled).
- Splitting: allow re-splits up to configured max; split aces get one card each
  (configurable); Double After Split (DAS) respected.
- Dealer plays out by rule: hits/stands on soft 17 per setting (H17/S17).
- Blackjack payout configurable (3:2 or 6:5 or 1:1).
- Correct handling of soft/hard totals, busts, pushes, blackjacks.
- Insurance offered when dealer shows an Ace (player can take/decline; default
  advice is don't take — tie this into the hint engine later).
- **The cut card / shuffle:** the shoe reshuffles when penetration threshold is
  reached (configurable). A visible cut card marker is a nice-to-have.

Deliverable at end of Phase 1: a playable but ugly version (placeholder rendering
is fine) where a full round can be played start to finish with correct rules.

---

## 2. Hi-Lo counting core + show/hide overlay (Phase 2)

- **Running count** updated as each card is revealed: 2–6 = +1, 7–9 = 0,
  10/J/Q/K/A = −1.
- **True count** = running count ÷ decks remaining (estimate decks remaining from
  cards dealt vs shoe size; round to nearest half-deck for the estimate).
- **Show/Hide toggle**: a button (and a hotkey, e.g. `C`) that reveals an overlay
  showing the current **running count** and **true count**. Hidden by default so
  the player practices keeping the count in their head, then peeks to check.
- The count must update live as cards animate in.
- Reset count correctly on shuffle.

Deliverable: player can play and toggle the count overlay on/off to self-check.

---

## 3. Basic strategy hint engine (Phase 3)

Encode the basic strategy tables EXACTLY as provided by the human (see
`STRATEGY.md` which the human will supply, or the embedded tables below). Two
table sets exist: a baseline set and an H17/deviation set — use the set that
matches the current table rules (specifically H17 vs S17). For now implement the
**H17 + Late Surrender** version as primary, since that matches the provided chart.

Tables to encode (dealer upcard 2–A across the top):

- **Pair splitting** (A,A / T,T / 9,9 / 8,8 / 7,7 / 6,6 / 5,5 / 4,4 / 3,3 / 2,2),
  with Y / Y-if-DAS / N semantics.
- **Soft totals** (A,9 down to A,2): H / S / D (double else hit) / Ds (double else
  stand).
- **Hard totals** (17 down to 8): H / S / D.
- **Late surrender**: 16 vs 9/10/A, 15 vs 10 (baseline); plus the H17 additions
  shown in the chart (e.g. 17 vs A, 15 vs A, etc. as per the provided image).
- **Insurance:** don't take (baseline). (Counting deviation handled in Phase 4.)

Hint behaviour (toggleable, OFF by default):

- When ON, for the player's current hand vs dealer upcard, show the correct
  action recommendation BEFORE the player acts (a subtle highlight or label on the
  recommended button).
- After the player acts, if their action differed from correct play, show a
  **"Wrong" indicator + a one-line reason** explaining why, e.g.
  *"Stand: hard 16 vs dealer 10 — high bust risk, basic strategy stands."*
  The reason should reference the relevant factor (total, dealer upcard, and where
  relevant the true count once Phase 4 deviations are in).
- When hint is OFF, the game says nothing and just lets the player play.
- Correct/incorrect decisions are recorded for stats (Phase 5).

Deliverable: with hints on, the game tells you right/wrong + why; with hints off,
silent play.

---

## 4. Betting, chips & count-based bet spread (Phase 4)

- A **bankroll** (starting amount configurable, e.g. 1000 units).
- **Chips**: clickable pixel chip denominations (e.g. 1 / 5 / 25 / 100) to build a
  bet before each round; satisfying stack animation when placing/winning.
- Bet is locked when the round deals; payouts settle to bankroll afterward.
- **Bet-spread coaching (optional, ties to hints):** since real counting profit
  comes from betting more at high true counts, optionally show a suggested bet
  size based on a simple ramp (e.g. bet = base × max(1, trueCount − 1)). When
  bet-coaching is on and the player's bet deviates a lot from the suggestion at a
  given count, note it (this is part of becoming a good counter, not just basic
  strategy).

Deliverable: full betting loop with chips, bankroll tracking, and optional
count-based bet suggestions.

---

## 5. Stats tracking — session + all-time + export (Phase 5)

Track and display:

- Hands played, win/loss/push rate, blackjacks.
- Bankroll over time (simple sparkline on canvas is a nice-to-have).
- **Strategy accuracy:** % of decisions that matched basic strategy, broken down
  optionally by category (hard/soft/pairs/surrender).
- **Counting accuracy:** when the player peeks at the count via the overlay,
  optionally first let them guess and compare (lightweight — only if it doesn't
  add a separate game mode; could be a small "I think the count is __" input that
  appears when they open the overlay, scored silently).
- Two scopes shown side by side: **This session** and **All time**.
- Buttons: **New session** (resets session stats), **Reset all** (clears all-time,
  with confirm), **Export stats** (JSON download), **Import stats** (file read).
- All persisted via `localStorage`; export/import is the PC-save mechanism.

Deliverable: a stats panel with session + all-time numbers and working export/
import.

---

## 6. Settings panel — table conditions (Phase 6)

All configurable, persisted in `localStorage`, applied on next shuffle/round where
relevant:

- Number of decks (1–8).
- Dealer hits or stands on soft 17 (H17 / S17). **Switches which strategy table
  the hint engine uses.**
- Double After Split (DAS) on/off. **Affects pair-splitting hints (the Y-if-DAS
  cells).**
- Blackjack payout: 3:2 / 6:5 / 1:1.
- Surrender: none / late. (Early surrender optional, low priority.)
- Re-split limit, and whether aces can be re-split.
- Double rules: any two cards / 9-10-11 only.
- **Shuffle / penetration:** at what fraction of the shoe the cut card triggers a
  reshuffle (e.g. 50%–90%).
- Starting bankroll.
- Toggles: count overlay availability, hint engine on/off, bet-coaching on/off,
  sound on/off.

Deliverable: a settings panel that actually changes game behaviour and the strategy
the trainer teaches.

---

## 7. Polish pass — pixel art, animations, sound, layout (Phase 7)

This is where it becomes satisfying. Design direction:

- **Aesthetic:** 8-bit casino, but NOT garish. Think a focused limited palette —
  deep felt green/teal table, warm chip colours, off-white card faces with a
  single bold accent for the count/hint UI. Avoid the generic "neon on black"
  default. Pick a small named palette (4–6 hex values) and stick to it.
- **Card animations:** cards slide in from the shoe with an ease, then flip
  (scale-x interpolation) to reveal — synced with the deal sound. Bust/blackjack
  gets a small screen-shake or flash.
- **Chip animations:** chips arc onto the bet spot; on win they slide back doubled.
- **Count overlay:** when shown, it's a clean readout; consider a subtle colour
  shift (cool when count is negative, warm when positive) so the player builds
  intuition.
- **Microcopy:** active-voice, sentence case, in the game's voice. Errors/empty
  states give direction, not mood. (E.g. hint reasons are instructive, not cute.)
- **Accessibility floor:** keyboard controls for all actions (H/S/D/P/R, C for
  count, etc.), visible focus, `prefers-reduced-motion` respected (instant deals
  instead of animated when set), and the master mute.
- Responsive down to a small window; usable on a laptop screen.

Deliverable: the finished, satisfying version.

---

## Build protocol for the agent

1. Read this whole SPEC before writing any code.
2. Keep logic (engine, counting, strategy) separated from rendering so it stays
   testable and editable.
3. Implement **one phase at a time, in order**. After finishing a phase:
   - Make sure `index.html` opens and the phase's deliverable works.
   - Write a 2–3 line note of what was done and how to test it.
   - **STOP and wait** for the human to test and give feedback before the next
     phase. Do not steamroll all 7 phases without check-ins.
4. Never add network calls, analytics, ads, or external runtime dependencies.
5. If the human supplies a `STRATEGY.md` with their exact tables, that overrides
   any strategy values you'd otherwise assume. Match it cell-for-cell.
6. Prefer correctness of blackjack rules and strategy over visual flourish; the
   teaching is the point.
