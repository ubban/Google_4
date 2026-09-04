# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game built as a single-file HTML5 application. It features:
- Multiple enemy types with varying behaviors
- 6 unique boss battles with distinct attack patterns
- Weapon upgrade system (Vulcan gun vs. Laser)
- Smart bomb mechanic for crowd control
- Score chain/multiplier system
- Three difficulty levels (Easy, Normal, Hard)
- Mobile touch controls + mouse/keyboard support
- Adaptive performance scaling
- WebAudio API synthesized sound effects (no external audio files)

## Architecture & Code Organization

The `index.html` file contains everything: HTML structure, CSS styling, and JavaScript game logic (1670 lines).

### JavaScript Structure (in execution order):

1. **Global DOM References** (lines 426-461): Canvas, UI elements, HUD components
2. **Local Storage Keys** (lines 463-466): Constants for persisting player preferences
3. **Audio System** (lines 542-676):
   - `ensureAudio()`: Lazy-initializes WebAudio context on first user interaction
   - `beep()`: Generates square/sawtooth/sine tones with frequency sweeps and filtering
   - `noiseBurst()`: White noise for percussion effects
   - `sfx` object: All sound effects (shoot, laser, explode, bomb, powerup, boss sounds, etc.)
4. **Difficulty & Theme Configuration** (lines 485-500):
   - `DIFFICULTIES`: Enemy spawn rates, speeds, shoot chances, HP multipliers per difficulty
   - `THEMES`: 6 sky gradient themes for each stage
   - `ENEMY_TYPES`: Grunt (fast, weak), Interceptor (very fast, weaving), Tank (slow, durable)
   - `BOSSES`: 6 unique boss designs with different HP multipliers
5. **State Management** (lines 864-869): Game state, player, entities (bullets, enemies, particles)
6. **Input Handling** (lines 767-861):
   - Keyboard input tracking
   - Touch joystick implementation (left side = movement, right side = fire/bomb buttons)
   - Mouse steering system (player glides toward cursor when mouse is over canvas)
   - Priority: keyboard/joystick > mouse (they don't fight for control)
7. **Game Core Loop**:
   - `loop()` (lines 1640-1653): RequestAnimationFrame loop, 60fps target
   - `update(dt)` (lines 1148-1302): Physics, collision detection, entity lifecycle
   - `draw()` (lines 1520-1638): Canvas rendering with layered drawing
8. **Game Systems**:
   - **Spawn System** (lines 1022-1041): `spawnEnemy()` - spawns increasing variety as stages progress
   - **Boss System** (lines 1043-1056, 1411-1439): `spawnBoss()`, `updateBoss()`, `killBoss()`
     - Boss patterns defined in `BOSS_PATTERNS` (lines 1372-1409)
     - 3 pattern functions per boss with distinct attack types (radial, aimed spread, columns, spirals, homing, teleport, etc.)
   - **Weapon System** (lines 1124-1146, 1304-1322): Vulcan (spread shot) vs Laser (piercing rays)
   - **Power-up System** (lines 1058-1067, 1304-1322): Life/bomb/weapon pickups
   - **Collision System** (lines 1069-1071, 1232-1295): `rectsOverlap()` checks for bullet-enemy, bullet-boss, enemy-player, powerup-player
   - **Particle System** (lines 1073-1085): Explosion particles with velocity and fade
   - **Chain/Multiplier System** (lines 1087-1101): Kill streak mechanics
9. **Rendering**:
   - **Canvas sizing** (lines 752-765): Responsive scaling, maintains 480x720 aspect ratio
   - **Drawing functions**: `drawPlane()`, `drawBossShip()`, `drawClouds()`
   - **Effect rendering**: Screen shake, flash, vignette pulse for low health
10. **UI & Accessibility**:
    - HUD updates via DOM manipulation (score, lives, bombs, weapon level)
    - Screen reader announcements via `announce()` (ARIA live region)
    - Reduced motion support (respects `prefers-reduced-motion` media query)
11. **Performance Adaptation** (lines 473-483):
    - `trackPerf()` monitors frame times over 45-frame window
    - Adjusts `quality` level (0=full, 1=medium, 2=low) to disable shadow/glow effects when needed

### Key Constants:
- `BASE_W = 480, BASE_H = 720`: Canvas resolution
- `KILLS_PER_STAGE = 18`: Enemies to defeat before boss appears
- 6 `THEMES`: Visual variety across stages
- `ENEMY_TYPES`: 3 types with 7 stage-gated
- `BOSSES`: 6 boss variants, cycled across stages
- `BOSS_PATTERNS`: Attack pattern library per boss

## Gameplay Flow

1. **Main Menu**: Difficulty selection, show previous best score
2. **Game Start**: `resetGame()` initializes player, enemies, score
3. **Stage Loop**:
   - Enemies spawn at increasing rates as difficulty ramps
   - Kill 18 enemies → boss spawned after 1.2s delay
   - Boss defeated → stage increments, theme changes, enemy variety increases
4. **Game Over**: Score recorded if it's a new best, retry prompt shown
5. **Pause**: Toggled via P/Esc or pause button

## Running the Game

Since this is a single HTML file with no build process:

```bash
# Open in any modern browser (desktop or mobile)
# File-based: just open index.html in a browser
# HTTP server (recommended for development):
python3 -m http.server 8000
# Then visit http://localhost:8000/index.html
```

The game will work in any browser supporting:
- HTML5 Canvas 2D
- WebAudio API (fallback: game runs silent if unavailable)
- LocalStorage (for persistence)
- ES6 JavaScript

## Development Notes

### Adding New Features

**New Boss Pattern**: Add to `BOSS_PATTERNS` array (1372-1409). Pattern functions should call firing helpers like `fireRadial()`, `fireAimedSpread()`, etc.

**New Enemy Type**: Add to `ENEMY_TYPES` object (502-506). Set `minStage` to gate appearance, provide `speedMul` and `weave` for movement behavior.

**New Weapon**: Modify `fireWeapon()` (1124-1146) and `applyPowerUp()` (1304-1322). Update HUD in `weaponLabel()`.

**New Difficulty**: Add to `DIFFICULTIES` object (485-489), then add button + handler.

**New Theme**: Add to `THEMES` array (493-500). Use RGB tuples for sky gradient stops and cloud/sun colors.

### Modifying Game Balance

- Enemy spawn rate: `DIFFICULTIES[difficulty].spawnBase` and `spawnMin` (485-489)
- Enemy behavior: `enemySpeed` multiplier and `enemyShootChance` (485-489)
- Boss HP: Base formula at line 1046 (`baseHp = 70 + stage * 35`) then multiplied by difficulty and boss-specific `hpMul`
- Player speed: `player.speed` (line 877, currently 260)
- Weapon fire rate: `fireDelay` constants (line 1177)
- Score values: `en.score` per enemy type (502-506), boss kill score (1446)

### Audio

WebAudio synth approach — no external files:
- `beep(freq, dur, type, vol, opts)`: Base oscillator with optional sweep, filter, panning, envelope
- `noiseBurst(dur, vol, opts)`: Brown noise for percussion
- All sounds in `sfx` object are composed from these primitives
- Muting via localStorage + UI toggle

### Mobile Considerations

- Touch joystick on left side (movement zone), buttons on right (fire/bomb)
- Viewport locked to prevent scroll on mobile
- Touch-action disabled for game canvas
- Respects device safe areas (notches, rounded corners)
- Reduced motion preference auto-detected at startup

### Performance

- Adaptive quality scaling in `trackPerf()` disables shadows/glows under load
- Particle count scales with quality level
- No external libraries — pure Canvas + WebAudio

## Common Tasks

**Run locally for testing:**
```bash
python3 -m http.server 8000
# Open http://localhost:8000/index.html
```

**Test a specific difficulty:**
Open DevTools console and run:
```javascript
setDifficulty('hard'); // or 'normal', 'easy'
```

**Debug enemy spawn rates:**
Look for `spawnTimer` logic (lines 1197-1207). Reduce `spawnBase` to spawn faster.

**Check collision detection:**
All collision checks use `rectsOverlap()` (lines 1069-1071). Modify AABBs to adjust hitbox sizes.

**Adjust screen shake:**
Call `screenShake(magnitude, duration)` — tweak these numbers for game feel.

**Add a new sound effect:**
1. Create a function in the `sfx` object (624-676)
2. Compose it from `beep()` and `noiseBurst()` calls
3. Call via `sfx.mySound()` in game code

## Browser Compatibility

- Chrome/Edge: Full support
- Firefox: Full support
- Safari: Full support (WebAudio works on iOS after first touch)
- Mobile browsers: Touch controls activate automatically

## Testing & Verification

Since this is a visual game with no test suite:
- **Manual testing**: Run through each difficulty, beat bosses, verify score calculation
- **Cross-browser**: Test on multiple browsers and devices
- **Mobile**: Verify touch controls (joystick + buttons) on small screens
- **Accessibility**: Verify reduced-motion mode disables animations, screen reader announcements work
