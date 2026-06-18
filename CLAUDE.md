# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## The Project

Valencian is a single-file, offline, browser-based blackjack card-counting trainer. The entire game — HTML, CSS, JavaScript, all assets as base64 data URIs — lives in **`index.html`**. No build step, no dependencies, no network calls at runtime.

Open `index.html` in any browser to run it.

## Syntax Checking

There is no build tool. After editing `index.html`, validate the script block with:

```bash
node -e "const fs=require('fs');const src=fs.readFileSync('index.html','utf8');const m=src.match(/<script>([\s\S]*?)<\/script>/);try{new Function(m[1]);console.log('OK');}catch(e){console.log('ERROR:',e.message);}"
```

Run this after every set of changes before committing.

## Editing the File

`index.html` is 3 MB+ due to embedded base64 sprites. The `Read` tool will hit token limits — use `sed -n 'START,ENDp'` or `grep -n` to locate sections, then edit with targeted Python string replacement scripts. Never use `sed`/`awk` for multi-line replacements; always use Python `str.replace()`.

## Architecture

### Rendering
- **Canvas:** 640×360 logical pixels (`CW`/`CH`), supersampled at 2× (physical 1280×720). `image-rendering: pixelated`. Everything game-related draws on this canvas; stats/settings/insurance are HTML overlays.
- **Render loop:** `scheduleRender()` queues one `requestAnimationFrame`; `renderFrame()` draws everything. Animations re-call `scheduleRender()` while active. Use `render()` (immediate) for instant UI updates like bankroll changes.
- **Draw order (back to front):** static scene (background + felt/rail/rim, cached) → stage fade overlay → dealer sprite → table dynamic layer (bell → bankroll stacks → bet circle → shoe) → cards → banners/overlays.
- **Static scene cache:** `drawStaticScene()` caches background + felt/rail to an offscreen canvas keyed on `CONFIG.dealerChar`. Invalidate by setting `_staticLayer = null` before calling `scheduleRender()` when visual state changes (e.g. bg image loads).

### Key Constants
```
TABLE_CX=320, TABLE_TOP_Y=208, TABLE_BOT_Y=334, TABLE_HALF_W=248
BELL_X=148, BELL_Y=200
CW=640, CH=360
```

### State
- `state` object: game phase (`IDLE/PLAYER/INSURANCE/DEALER/DONE`), `bankroll`, `bet`, `pendingBet`, `betChipLog`, hands, cards.
- `CONFIG` object: all rule settings + `dealerChar` (0–3). Persisted to `localStorage` via `saveSettings()`/`loadSettings()`.
- Chip decomposition: `decomposeChips(amount)` → array of `{denom, count}`. `betChipLog` is rebuilt from `decomposeChips(state.pendingBet)` in `finishRound()`.

### Dealer Stages
Four stages, each with a background image, dealer sprite, and unlock threshold:
| `CONFIG.dealerChar` | Dealer | Background | Unlock |
|---|---|---|---|
| 0 | Default male (dealer 1) | Jazz lounge | default |
| 1 | Female beach dealer 2 | Beach bar | 1100 |
| 2 | Blonde male (dealer 3) | Backyard pool | 2000 |
| 3 | Alien (dealer 4) | Moon base | 2500 |

Stage transitions trigger `stageFadeAnim = {t0, dur:380}` (black fade) via `_switchDealer()`.

### Dealer Sprites
Each dealer has `DEALER_IMGS`, `DEALER2_IMGS`, `DEALER3_IMGS`, `DEALER4_IMGS` — each with `idle/dealing/win/lose/blink` canvases. Sprites are PNG base64 constants (`DEALER_IDLE_SRC`, etc.) with black backgrounds stripped (`alpha=0` for pixels < 15,15,15 in all channels). Blink canvases are generated at load by scanning for eye pixels and painting over them with skin/armor color:
- Humans (dealers 1–3): `_makeDealer1BlinkCanvas` / `_makeHumanBlinkCanvas` — detects dark pixel clusters.
- Alien (dealer 4): `_makeAlienBlinkCanvas` — detects bright (glowing) pixel clusters, covers with dark armor color.

Dealer 4 also has `DEALER4_THIRDEYE_SRC`/`DEALER4_THIRDEYE_IMGS` — a variant sprite with a forehead eye. `_thirdEyeActive` flips true every ~30s for ~600ms, shown via override in `_drawDealerSprite()`.

Idle micro-animation: breathing (`Math.sin(now * 0.0018) * 1.5` px vertical) and sway — applied as transforms in `_drawDealerSprite()`.

### Table Geometry
`_tablePath(xi, yi)` generates a `Path2D` for the table outline using **5-step** integer-snapped Bezier segments (deliberately low step count for pixel-art stair-step edges). `xi/yi` are inset offsets for nested paths (felt inside rail inside rim).

### Pixel Art Rules (Non-negotiable from SPEC)
- All drawn primitives must use `fillRect` where possible — no `ctx.arc`/`ctx.ellipse` for visible game shapes. Arcs are only acceptable for glow/pulse effects.
- `_tablePath` uses few steps deliberately for chunky stair-stepped edges.
- Bell (`drawBell()`) is drawn entirely with per-row `fillRect` calls.
- Chips use a 12-gon polygon style (drawn in `_drawBankrollStacks` / `_drawBetCircle`).

### Item Tray
5 positions along the right shoulder Bezier curve of the felt, offset ~25px inward: `ITEM_SLOT_POSITIONS`. No visible markers. `drawWhiskey()` draws at slot 0. No slot circles/markers should ever be drawn.

### Shop Sidebar
HTML overlay (`#shop-sidebar`), opens on bell click. Centered vertically (`top:50%; transform: translateY(-50%)`), max-height 320px. 8 locked placeholder slots.

### Non-Goals (never violate)
- No external libraries, frameworks, build tools, or npm.
- No server, network calls, accounts, or tracking.
- No 3D animations or physics.

## Git Workflow

Development branch: `claude/sweet-cray-kjo4zg`

Always push with:
```bash
git push -u origin claude/sweet-cray-kjo4zg
```
