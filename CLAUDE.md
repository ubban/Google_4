# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The entire game (1669 lines) is contained in `index.html` with inline CSS and JavaScript, no external dependencies or asset files.

## Getting Started

**Run the game locally:**
```bash
# Open in a web browser (most platforms)
open index.html

# Or from the command line on Linux/macOS
xdg-open index.html  # Linux
open index.html      # macOS

# On Windows, or any OS with Python 3:
python3 -m http.server 8000
# Then visit http://localhost:8000/index.html
```

The game runs entirely in the browser with no build step or server required. However, using a local HTTP server is recommended for cross-origin testing of features like localStorage.

## Architecture

The game is structured in three sections within `index.html`:

### 1. HTML Markup (Lines 361-420)
- Game canvas container for rendering
- HUD elements (score, lives, bombs, multiplier, weapon indicator)
- Boss health bar with stage warning banner
- Combo popup and stage transition banner
- Control buttons (mute, motion toggle, pause)
- Overlay screens (start menu, pause, game over)
- Touch-enabled virtual joystick and action buttons
- Accessibility features (screen reader live region)

### 2. CSS Styling (Lines 9-359)
- Responsive canvas scaling (BASE_W: 480px, BASE_H: 720px)
- Dark theme with glassmorphic UI elements (backdrop-filter blur)
- Mobile-optimized layouts with safe-area insets
- Accessible color contrasts and animations
- Prefers-reduced-motion media query support for accessibility
- Emoji-based iconography (no external icon files)

### 3. JavaScript Game Logic (Lines 422-1669, IIFE pattern)

**Key Game Systems:**

#### Difficulty System (Lines 485-489)
Three preset profiles controlling enemy spawn rate, speed, shoot chance, starting lives/bombs, and boss HP multiplier. Stored in localStorage and can be selected from menu.

#### Core Game State (Line ~880+)
```javascript
player       // x, y, w, h, weapon, level, invuln timer, health
enemies      // array of active enemy entities
boss         // current boss object with hp, position, attack patterns
powerUps     // dropped weapon/life/bomb pickups
projectiles  // bullets from player and enemies
particles    // visual effects (explosions, etc)
stage        // current level (affects enemy types, themes, boss selection)
score        // cumulative points with multiplier
chain        // consecutive kills for combo multiplier
```

#### Enemy Spawning (Lines 1022-1041)
- Three enemy types: grunt, interceptor, tank
- Stage-based availability (tank only appears from stage 2+)
- Random shoot patterns: spread vs aimed
- Weaving interceptors follow sine-wave paths
- Spawn timer decreases as stage progresses (KILLS_PER_STAGE = 18)

#### Boss System (Lines 1043-1056, 508-515)
- 6 boss designs cycling through stages (BOSSES array)
- Attack telegraphing with warning effect
- HP scales by difficulty and boss design multiplier
- Stages beyond 6 repeat boss designs
- Boss health bar UI shows during boss encounters

#### Weapon System
- Vulcan: rapid-fire spread shots
- Laser: slower single-beam concentrated fire
- Weapons upgrade through pickup collection (sfx.powerup triggers level-up animation)
- Visual indicators in HUD show current weapon and level

#### Scoring & Multiplier (Lines 1087-1098)
- Chain mechanic: consecutive kills increase multiplier (1x to 10x)
- Multiplier decays over 2.2 seconds without kills (chainTimer)
- Each enemy kill adds base score × multiplier
- Kills per stage requirement triggers progression

#### Web Audio System (Lines 542-676)
No external audio files—all sound is synthesized using Web Audio API:
- `beep()`: frequency sweep with waveform (sine/square/sawtooth/triangle)
- `noiseBurst()`: filtered white noise for impact effects
- `sfx` object maps game events to audio: shoot, explode, hit, bomb, powerup, bossTelegraph, bossDown, stageClear, gameover
- `masterGain` node respects muted state
- Spatial audio: pan parameter positions sounds left/right based on entity.x

#### Input System (Lines 768-840)
**Keyboard:** WASD/Arrows (move), Space (fire), X/Shift (bomb), P/Esc (pause)
**Mouse:** Move to crosshair, click to fire
**Touch:** 
- Left 55% of screen: drag joystick to move
- Right 45% of screen: FIRE and BOMB buttons
- Joystick base appears dynamically under finger on first touch in move zone

#### Rendering (Lines ~1200+)
- Canvas context manipulation (translate, rotate, globalAlpha for transparency)
- Procedural sprite drawing for player, enemies, and boss using fillPath()
- Parallax cloud layers with sine-wave movement
- Sky gradient theme rotation every 6 stages (THEMES array)
- Dynamic shadow/glow effects on boss core when telegraphing (quality-dependent)
- Particle effects for explosions scaled by performance quality setting

#### Performance Adaptation (Lines 473-483)
- Automatic quality scaling: 0 (full), 1 (medium), 2 (low)
- Tracks last 45 frame times and adjusts quality if avg > 25ms or < 17.8ms
- Affects particle count, shadow blur, and animation polish
- Users cannot manually select quality (automatic only)

#### Accessibility Features
- Screen reader announcements via `announce()` function and srLive region
- Reduced motion toggle (button in top-right) respects prefers-reduced-motion
- Reduced motion: eliminates shake, flash, pulse animations; caps animation duration
- Color contrast: text uses text-shadow for readability on varied backgrounds
- No emoji-only buttons: all buttons have title attributes and text labels

#### State Persistence (localStorage)
```javascript
skyfighter.bestScore      // all-time best score
skyfighter.muted          // audio mute state
skyfighter.difficulty     // selected difficulty (easy/normal/hard)
skyfighter.reducedMotion  // accessibility toggle
```

#### Game Loop (Lines ~1500+)
```
requestAnimationFrame(update)
  ├─ Update player input/position
  ├─ Update enemies/boss/projectiles
  ├─ Collision detection (player↔enemies, projectiles↔enemies, player↔powerups)
  ├─ Spawn new enemies if timer ready
  ├─ Stage progression (kill count → boss spawn)
  ├─ Check win/loss conditions
  └─ render() → draw() all entities
```

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line ~502) with properties: w, h, hp, speedMul, color, dark, score, minStage, optional weave
2. Add explosion sound to `EXPLODE_SFX` mapping (line 678) if unique
3. Optional: add new `sfx.explodeXxx()` function in sfx object (lines 624-676)

### Adding a New Boss
1. Add entry to `BOSSES` array (line 508) with name, colors (hullTop/Bottom, wing, core), and hpMul
2. Boss drawing logic is generic in `drawBossShip()` and cycles through this array automatically

### Adding a New Weapon Type
1. Modify `weaponLabel()` function to return new weapon name
2. Update weapon firing logic in the main update loop (around line ~1400+) where `player.weapon` is checked
3. Add new `sfx.xxx()` for weapon fire sound
4. Update weapon type cycling when pickup is collected

### Tweaking Difficulty
Adjust constants in `DIFFICULTIES` object (lines 485-489):
- `spawnBase` / `spawnMin`: spawn timer thresholds (lower = more enemies)
- `enemySpeed`, `enemyShootChance`: combat difficulty
- `lives`, `bombs`: starting resources
- `bossHpMul`: boss health scaling

### Modifying Themes
Edit the `THEMES` array (lines 493-500): each entry defines sky gradient colors, cloud color, sun color, and name. Accessed sequentially by `(stage - 1) % THEMES.length`.

## Code Style Notes

- **IIFE pattern**: entire game runs in a self-executing function to avoid global scope pollution
- **Entity objects**: player, enemies, boss, powerUps, projectiles are plain objects (no classes)
- **Canvas context**: often saved/restored for transformation safety
- **No external dependencies**: HTML5 Canvas, Web Audio API, localStorage only
- **Math precision**: collisions use distance/overlap; no physics engine
- **Animation timing**: uses frame counter (incremented each render), not timestamps

## Testing & Debugging

**No automated tests exist.** Manual testing checklist:

- [ ] Game starts, difficulty selection works
- [ ] Player movement (keyboard & mouse) is responsive
- [ ] Weapons fire and hit enemies correctly
- [ ] Bombs clear screen and reset combo
- [ ] Enemy spawn rate increases per stage
- [ ] Boss appears after KILLS_PER_STAGE enemies
- [ ] Score and multiplier display updates
- [ ] Best score persists across sessions (localStorage)
- [ ] Mute toggle silences audio
- [ ] Pause/Resume works and resets overlay state
- [ ] Touch controls appear on mobile, hidden on desktop
- [ ] Reduced motion toggle disables animations without breaking gameplay
- [ ] Stage transitions and boss intros display correctly
- [ ] Game Over shows final stats and allows restart

**Browser DevTools:**
- Open Console to catch any JS errors
- Check Network tab for any unexpected requests (there should be none)
- Use Performance tab to profile frame rendering and adjust quality logic if needed
- Inspect localStorage to verify state persistence
