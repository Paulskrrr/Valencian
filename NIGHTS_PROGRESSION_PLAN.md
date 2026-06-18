# Valencian Blackjack – Nights & Progression Plan

## Vision

Create a compelling, atmospheric progression layer on top of the existing sharp Blackjack engine without compromising the core gameplay, optimal strategy, or rule customization.

The game will have two clearly separated modes:
- **Sandbox Mode** (current game) — for free play, practice, counting, and experimentation.
- **Valencian Nights Mode** — a guided, story-tinged, progression-based experience with fixed rules per stage, light gamification, and replayability.

## Core Principles
- Never alter core Blackjack rules, probabilities, strategy engine, or counting in any way.
- Sandbox Mode remains completely untouched.
- Nights Mode uses fixed rule sets per stage.
- Progression is relaxed and chill-friendly.
- Focus on atmosphere, whiskey, dealer personality, and "one more night" feeling.

## Main Menu & Overworld

**Main Menu**
- Title screen with two primary buttons:
  - **Sandbox Mode** → launches current game (unchanged)
  - **Valencian Nights** → enters the new mode

**Overworld Map**
- One beautiful AI-generated image serving as a clickable map (casino floor plan, rooftop view, or stylized city map with locations).
- Locations represent different "stages" / nights.
- Initially only **Stage 1** is unlocked and highlighted.
- Locked stages are grayed out with tooltip showing unlock requirement.
- Persistent UI bar showing: Bankroll (Nights), Reputation/Level, Total Whiskey, Achievements.
- Clickable areas lead into specific Nights.

## Nights System

A **Night** is a structured session with:
- Fixed rule set (defined per stage)
- Hand limit (e.g. 80–150 hands depending on stage)
- Profit target to "clear" the night and unlock the next stage
- Persistent bankroll across all Nights

**Night Flow**:
1. Player enters a stage from the Overworld.
2. Night begins with fixed rules and fresh shoe.
3. Play until hand limit is reached.
4. At end of night → Summary screen (profit, hands played, accuracy, whiskey consumed, etc.).
5. If profit target met → Stage cleared → Next stage unlocked on Overworld.
6. If profit target not met → Must replay the same stage.

## Progression

- Start with **Stage 1** only.
- Each subsequent stage requires beating the previous stage's profit target in one night.
- Beating a stage unlocks the next on the map.
- Previously cleared stages can be replayed freely.
- Global progression:
  - Reputation / Level (gained from hands + profit + strategy accuracy in Nights)
  - Cosmetic unlocks (felts, card backs, dealer variants, map decorations)

## Achievements (Light)

Track across Nights mode:
- First Night Completed
- Clear Stage 1
- Reach +$X profit in one night (multiple tiers)
- High strategy accuracy over a full night
- Whiskey milestones
- Big win streaks
- High true count achievements

Achievements grant cosmetic rewards and satisfaction.

## Flavor & Atmosphere

- **Whiskey** has visible effects (dealer animation, table lighting, dialogue).
- Random light flavor events during nights (cosmetic only):
  - Dealer commentary
  - Temporary visual changes (lighting, particles)
  - Ambient sound or music shifts
- Later stages can have unique themes, dealer personalities, and minor story flavor.

## Technical Requirements

- Single HTML file only.
- Use existing canvas rendering, engine, and logic wherever possible.
- Separate localStorage keys for Nights progress (`nights_progress`, `nights_bankroll`, etc.).
- New states: `MODE_MAIN_MENU`, `MODE_OVERWORLD`, `MODE_NIGHT_PLAY`, `MODE_NIGHT_SUMMARY`.
- Overworld can use an `<img>` with click detection or canvas-based hit areas.

## Recommended Implementation Phases

1. Main Menu + state switching (Sandbox vs Nights)
2. Overworld map screen with Stage 1 clickable
3. Basic Night session (fixed rules + hand limit + end summary)
4. Profit target logic + stage unlocking
5. Persistent storage for Nights progress
6. Light achievements system
7. Whiskey integration in Nights
8. First random flavor events
9. Polish (summary screens, map interactions, cosmetics)

## Future Expansion Ideas
- More stages with unique themes
- Story beats between major stages
- More dealer variants and animations
- High-score / best night leaderboards per stage

---

**Approved by Grok – June 2026**

This document preserves the sharp design of the original game while adding meaningful, chill progression. Ready for implementation.