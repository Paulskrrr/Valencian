# Valencian Nights — Stage Identity & Shop Pitch

> **Status: PITCH ONLY — nothing here is implemented yet.** This proposes how to
> give each of the four Nights stages a distinct gameplay *feel* (beyond rule
> tweaks) and an unlockable shop that turns the carried-forward bankroll into
> meaningful choices.

## Where we are today
- Four stages already exist with fixed rule sets (decks, H17/S17, BJ payout,
  surrender, DAS, resplit), a hand limit, and a profit target.
- Bankroll now **carries forward only out of conquered stages** — a failed night
  replays from your entry bankroll. That makes the shop economy safe to build on:
  you can't grind yourself to zero and get stuck.
- Each stage has a persona who now reacts in speech bubbles (friendly guy →
  flirty challenger → alien → rich-egoist boss).

The rules differences are real, but they're *invisible math*. The pitch is to
layer one signature, legible mechanic on top of each persona, plus a shop that
spends bankroll on persistent edges.

---

## Part 1 — One signature mechanic per stage

Each stage keeps blackjack intact; the twist is a single, readable modifier that
matches the host's personality. Counting/strategy stay valid.

### Stage 1 — Jazz Highrise (Friendly Guy): **"Warm-up House"**
- **Theme:** the tutorial host. Generous, low-pressure.
- **Mechanic — Comp Chips:** every Nth cleared hand the dealer slides you a small
  free chip ("on the house"). Reinforces the friendly tone and eases new players
  into the bankroll loop.
- **Why:** teaches the carry-forward economy without punishing mistakes.

### Stage 2 — Paradise Bar (Flirty Challenger): **"Double or Dare"**
- **Theme:** she challenges you; risk is flirtation.
- **Mechanic — Dares:** occasionally before a hand she offers a wager modifier:
  "double the payout if you win this hand — but no surrender/insurance allowed."
  Player taps to accept or decline. Optional, high-variance, opt-in spice.
- **Why:** introduces player-driven risk decisions on top of basic strategy.

### Stage 3 — Moon Base (Alien): **"Unknown Variables"**
- **Theme:** cryptic, alien — the rules feel *off*.
- **Mechanic — Glyph Rounds:** every so often a hand is flagged with a glyph and
  one parameter is secretly shifted for that hand only (e.g. blackjack pays 2:1,
  or a push becomes a half-win), revealed only at settle via a symbol banner.
  Always player-favorable or neutral so it reads as "alien generosity," never a
  feel-bad. The HUD shows glyphs, not numbers.
- **Why:** novelty + tension; rewards players who stay flexible. Never breaks the
  underlying count.

### Stage 4 — Villa Betera (Rich-Egoist Boss): **"The House Always Wins"**
- **Theme:** the final boss who taunts you and stacks the deck.
- **Mechanic — House Pressure:** a visible "House Confidence" meter. While high,
  the boss takes small liberties (e.g. ties on 22 push instead of win — the
  signature villain rule, telegraphed). Beating hands drains the meter; when you
  empty it, the night's profit target drops or a payout bonus unlocks for the
  rest of the night. The taunts escalate as the meter falls.
- **Why:** turns the finale into a comeback arc with a clear win condition beyond
  raw bankroll, and pays off the "house always wins" voice when you finally break
  it.

**Design guardrails (carried from the existing plan):**
- Sandbox mode stays 100% untouched.
- No mechanic alters the validity of counting or basic strategy in a way the
  player can't see — every twist is telegraphed (banner, glyph, or meter).
- Each is *one* mechanic per stage, so each night has a single memorable hook.

---

## Part 2 — The Shop (spend carried-forward bankroll)

The bell already opens a shop sidebar with 8 locked placeholder slots and a
working whiskey/tip economy. Proposal: fill those slots with persistent,
bankroll-priced unlocks. Two tiers:

### A. Consumables (per-night, reset each night)
| Item | Effect | Notes |
|---|---|---|
| Whiskey *(exists)* | flavor + dealer reaction | keep as-is |
| Double Espresso | reveals optimal-play hint for N hands | disabled in counting? no — Nights has no count anyway |
| Lucky Cigar | one free re-deal (mulligan) per night | high value, high price |
| Marker (IOU) | small one-time bankroll advance, repaid from next win | risk tool |

### B. Persistent unlocks (bought once, owned forever)
| Item | Effect | Unlock gate |
|---|---|---|
| Velvet Bankroll | start each night with +X seed chips | cleared Stage 1 |
| Counter's Eye | re-enable a subtle running-count tint in Nights | cleared Stage 2 |
| Loaded Dice (cosmetic) | custom chip/card-back skins | bankroll milestone |
| House Key | replay any cleared stage at a higher target for bigger payout | cleared Stage 3 |
| Boss's Cufflinks | unlocked by beating Stage 4 — small permanent payout boost | cleared Stage 4 |

### Shop principles
- **Bankroll is the only currency** — ties directly into the carry-forward rule
  you just asked for. Winning nights funds permanent power; the loop is
  "conquer → bank → invest → push deeper."
- **No pay-to-win cliff:** persistent items give comfort/flavor, not broken edges.
  The biggest spends are cosmetic or convenience.
- **Each persona sells different things:** the friendly guy's shop is cheap
  comforts; the boss's shop is absurdly expensive flexes. Flavor through pricing.
- **Slots unlock by progression,** so the shop grows as you clear stages —
  reusing the existing 8-slot sidebar and lock states.

---

## Suggested build order (when greenlit)
1. Shop sidebar: wire the 8 slots to a real `items` state + bankroll spend.
2. Persistent unlocks first (Velvet Bankroll, cosmetics) — lowest risk, all reuse
   existing systems.
3. One stage mechanic at a time, starting with Stage 1 Comp Chips (simplest) and
   ending with Stage 4 House Pressure meter (most new UI).
4. Per-night consumables last (they need a "reset on night start" hook).

Everything above stays inside the single-file, no-dependency, no-network
constraints, and leaves Sandbox mode untouched.
