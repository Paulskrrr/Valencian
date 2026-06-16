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

**Logic only. No new visual design in this phase — use whatever UI exists.**

Track and display:

- Hands played, win/loss/push rate, blackjacks hit.
- **Strategy accuracy:** % of decisions matching basic strategy, broken down by
  category (hard totals / soft totals / pairs / surrender).
- **Bankroll delta** for the session (simple number, not a chart — chart comes in
  Phase 7 if at all).
- Two scopes: **This session** and **All time**, shown side by side.
- Buttons: **New session** (resets session stats only), **Reset all** (clears
  all-time with a confirm dialog), **Export stats** (downloads JSON file),
  **Import stats** (reads a JSON file back in).
- All persisted via `localStorage`. Export/import is the user's PC-save mechanism
  — no server, no cookies, no other storage.
- Correct/incorrect decisions feed into accuracy from Phase 3's hint engine.

Do NOT add counting-guess prompts or any game-mode logic here. Keep it a passive
tracker only.

Deliverable: a stats panel showing session + all-time numbers with working
export/import. Functional, not pretty.

---

## 6. Settings panel — table conditions (Phase 6)

**Logic only. No new visual design in this phase — use whatever UI exists.**

All settings persisted in `localStorage`, applied on next shuffle/round:

- Number of decks (1–8).
- Dealer rule: H17 (hits soft 17) / S17 (stands soft 17). **Must switch which
  strategy table the hint engine uses.**
- Double After Split (DAS): on/off. **Affects pair-splitting hints (Y/N cells).**
- Blackjack payout: 3:2 / 6:5 / 1:1.
- Surrender: none / late.
- Re-split limit (1–3×), split aces re-splittable: yes/no.
- Double rules: any two cards / 9-10-11 only.
- Penetration: % of shoe dealt before reshuffle (50–90%).
- Starting bankroll.
- Feature toggles: count overlay on/off, hint engine on/off, bet-coaching on/off,
  sound on/off, index deviations on/off.

Deliverable: a settings panel where every option actually changes game and trainer
behaviour. Functional, not pretty.

---

## 7. Full design pass — pixel art, layout, animations, sound (Phase 7)

**This is the only phase that owns visual decisions. Phases 1–6 build no
permanent UI — Phase 7 redesigns the whole thing from scratch if needed.**

### Visual direction: Jazz Lounge

The target aesthetic is a **gentlemen's jazz lounge, pixel art, circa 1950s**.
Relaxed, warm, a little smoky. Think: a proper felt table under warm lamplight,
not a garish Vegas floor. The pixel art should be readable and charming, not
microscopic noise.

Specific requirements:

- **Full-screen layout.** The game fills the browser window — no boxed island in
  the middle of a dark void like Phase 1. The table IS the page.
- **Resolution:** render at a resolution where cards are clearly legible — at
  minimum 640×360 internal canvas resolution scaled up, or higher. 320×180 was
  too low. Pixel art yes; unreadable smudge no.
- **Palette (locked, 6 colours):**
  - Table felt: `#2D5A3D` (deep forest green)
  - Table trim/rail: `#8B4513` (warm wood brown)
  - Card face: `#F5F0E8` (warm off-white)
  - Card back / UI accent: `#C41E3A` (deep red)
  - Chip gold: `#D4A017`
  - Background (room/lounge): `#1A1008` (very dark warm brown — the "room"
    behind the table)
- **Cards:** drawn procedurally on canvas. Large enough to read rank and suit
  clearly. Suits drawn as pixel glyphs (♠ ♥ ♦ ♣). Card back has a simple
  pixel diamond/cross pattern in the deep red.
- **Chips:** stacked pixel chips in denominations 1/5/25/100. Warm colours
  (white/red/green/black with gold edge). Arc animation onto bet spot on click;
  slide back on win.
- **Table felt:** a simple pixel-art oval or rounded rectangle felt area. Dealer
  zone at top, player zone at bottom, bet spot in the middle. Subtle pixel
  stitching on the rail edge.
- **Background:** the "room" behind the table — very dark warm brown, optionally
  a hint of pixel art detail (a lamp, a curtain edge, whatever fits) but keep it
  subtle. The table is the hero.
- **Count overlay:** a small warm-toned readout (think: brass nameplate). Colour
  shifts: cool blue tint when count is negative, warm amber when positive — helps
  build intuition without text.
- **Animations:**
  - Cards slide in from a shoe in the top-right corner, ease in, then flip
    (scale-x interpolation) to reveal face. Synced with deal sound.
  - Bust: brief screen-shake.
  - Blackjack: brief gold flash on the cards.
  - Chip arc on bet placement.
  - All animations respect `prefers-reduced-motion` (instant cuts instead).
- **Sound (Web Audio API, procedural — no external files):**
  - Deal/flip: short crisp click + soft whoosh
  - Win: short rising major arpeggio (warm tone)
  - Lose: short falling minor tone
  - Master mute toggle always visible.
- **HUD / controls:**
  - Action buttons (Hit / Stand / Double / Split / Surrender) sit below the table,
    clean and readable. Keyboard shortcuts labelled.
  - Count toggle (C), hint toggle, settings — small, unobtrusive, top corner.
  - Stats panel accessible via a button, slides in as an overlay (doesn't
    replace the table view).
- **Typography:** pixel/bitmap font for all in-game text. Sentence case. Active
  voice. No decorative copy.
- **Accessibility floor:** full keyboard control, visible focus ring, mute
  toggle, reduced-motion support. Usable down to a 1280px wide window.

Deliverable: the finished, satisfying version. This phase may rewrite large parts
of the rendering code — that is expected and fine.

---

## Build protocol for the agent

1. Read this whole SPEC and STRATEGY.md fully before writing any code.
2. Keep logic (engine, counting, strategy, stats) strictly separated from
   rendering. The engine should never care what the UI looks like.
3. Implement **one phase at a time, in order**. After finishing a phase:
   - Verify `index.html` opens and the phase deliverable works correctly.
   - Write a 2–3 line note of what was built and how to test it.
   - **STOP and wait for human feedback before the next phase.**
4. Phases 1–6 are logic phases. Do not make permanent visual design decisions
   in these phases. Placeholder UI is fine; anything that will be redesigned in
   Phase 7 should be minimal.
5. Phase 7 owns all visual decisions. It may rewrite rendering entirely.
6. Never add network calls, analytics, ads, or external runtime dependencies.
7. STRATEGY.md tables are authoritative — match cell-for-cell, never assume.
8. Correctness of blackjack rules and strategy always takes priority over visual
   flourish. The teaching purpose is non-negotiable.
