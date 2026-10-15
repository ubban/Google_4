# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single self-contained HTML5 file (index.html). No build tools, bundlers, or external dependencies are required.

## Running the Game

Simply open `index.html` in any modern web browser. The game is responsive and works on desktop (mouse/keyboard) and mobile (touch controls).

## Code Architecture

Everything is in a single 1669-line HTML file with three integrated sections:

### 1. HTML Structure (`<head>` and `<body>`)
- Canvas element for game rendering
- HUD overlays for score, health, weapons, etc.
- Pause/game-over screen overlay
- Touch control zones (joystick + fire/bomb buttons)
- Screen reader announcements for accessibility

### 2. CSS Styling
- Dark theme with glassmorphism effects
- Responsive sizing with `clamp()` for mobile-first design
- Animations for combo popups, boss warnings, screen flashes
- Safe area insets for notch-aware mobile devices
- Reduced-motion media query support for accessibility
- All colors and effects are inline

### 3. JavaScript Game Logic (lines 422–1667)
Wrapped in an IIFE (immediately invoked function expression) for scoping. The code is organized by feature:

**Core Systems:**
- **Canvas & Rendering** (lines 752–1638): 2D canvas setup, drawing functions for planes, bullets, enemies, particles
- **Audio System** (lines 542–676): WebAudio synth (no external assets), sound effects via oscillators and noise bursts
- **Input Handling** (lines 767–861): Keyboard, mouse steering, and touch/joystick control
- **Game State** (lines 863–912): Player, enemies, bullets, particles, bosses; reset and initialization
- **Game Loop** (lines 1640–1665): RAF-based main loop with update/draw cycle
- **Collision Detection** (lines 1069–1071): `rectsOverlap()` utility

**Game Mechanics:**
- **Spawning** (lines 1022–1056): Enemy and boss spawning with stage-based progression
- **Enemy Behavior** (lines 1210–1227): Movement, weaving patterns, shooting logic
- **Boss Patterns** (lines 1372–1435): 6 unique boss designs with distinct attack patterns (radial bursts, homing, spirals, etc.)
- **Weapon System** (lines 1124–1146): Vulcan (spread shot) and laser (piercing), with 3 upgrade levels each
- **Power-ups** (lines 1058–1322): Health, bombs, and weapon upgrades with collision pickup
- **Scoring & Chain** (lines 1087–1101): Score multiplier based on combo kills
- **Screen Shake & Visual Effects** (lines 726–730, 698–709): Vignette, flash, particle effects
- **Difficulty Balancing** (lines 485–489): Three preset difficulty curves with spawn rate, enemy speed, boss HP
- **Adaptive Quality** (lines 473–483): Performance tracking and rendering quality scaling
- **Pause & Game Over** (lines 1491–1489): Pause screen, game-over detection, best score tracking

## Key Data Structures

- **Player**: `{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }`
- **Enemies**: `{ x, y, w, h, speed, hp, type, color, pattern, shootTimer, ... }`
- **Bullets**: `{ x, y, w, h, speed, vx, pierce, laser }` (player) or `{ x, y, speed, vx, dy, radial, dx }` (enemy)
- **Boss**: `{ x, y, w, h, hp, maxHp, design, entering, phaseTimer, attackTimer, patternIndex, telegraph, ... }`
- **Power-ups**: `{ x, y, w, h, speed, type: 'life'|'bomb'|'weapon' }`
- **Particles**: `{ x, y, vx, vy, life, maxLife, color }`

## Common Development Tasks

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` (around line 502) with `w, h, hp, speedMul, color, dark, score, minStage`
2. Add explosion sound to `EXPLODE_SFX` (line 678) if unique
3. Add spawn logic in `spawnEnemy()` if it has special patterns

### Add a New Boss
1. Add to `BOSSES` array (line 508) with hull colors, core color, and HP multiplier
2. Add a new pattern array to `BOSS_PATTERNS` (line 1372) with firing functions
3. Boss selection is `BOSSES[(stage - 1) % BOSSES.length]`, so it cycles

### Add a New Weapon Type
1. Modify `weaponLabel()` (line 871) to include your weapon name
2. Update `fireWeapon()` (line 1124) with your weapon's firing logic
3. Update `applyPowerUp('weapon')` (line 1311) to cycle through weapons

### Add Visual Theme
1. Add entry to `THEMES` (line 493) with sky gradient colors, cloud color, sun color, and name
2. Theme is selected by `THEMES[(stage - 1) % THEMES.length]` in `draw()`
3. Themes cycle with stages

### Add Sound Effect
1. Add a function to the `sfx` object (line 624) using `beep()` or `noiseBurst()`
2. Call it where needed (e.g., `sfx.myEffect()`)
3. Audio is muted if `muted === true` or if audio context fails

### Modify Game Balance
- **Difficulty levels** in `DIFFICULTIES` (line 485): spawn rates, enemy speed, shoot chance, lives, bombs, boss HP
- **Performance quality** tiers: `quality` variable adapts rendering based on frame times
- **Weapon damage**: Boss takes 3 damage per hit; enemies take 1
- **Enemy spawning**: `spawnTimer` uses base spawn time minus difficulty ramp
- **Multiplier**: Chain increases by 1 every 5 kills; resets if 2.2 seconds pass without a kill

## Important Notes

- **No Build Step**: Changes to index.html are live on refresh
- **Canvas Size**: 480×720 (BASE_W/BASE_H); scaled on display via CSS
- **Frame-Rate Agnostic**: Uses delta time (dt) for all physics/movement
- **LocalStorage Keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- **Accessibility**: Screen reader announcements, reduced-motion support, color contrast tested
- **Mobile First**: Touch controls auto-enable on touch devices; mouse and keyboard always work
- **No External Assets**: All graphics are canvas drawing, all audio is synthesized

## Debugging Tips

- Open browser DevTools → Console for errors
- Modify `quality` variable (line 474) to force rendering quality: 0=full, 1=medium, 2=low
- Set `running = true` in console and modify game state directly (e.g., `score += 1000`)
- Check LocalStorage: `localStorage.getItem('skyfighter.bestScore')`
- Frame count is available as `frame` variable; used for animations and timing
