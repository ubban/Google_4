# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file, browser-based Raiden-style vertical shoot-'em-up game built entirely in vanilla HTML5/CSS/JavaScript. No external libraries or build tools—just `index.html` with embedded styles and game logic.

## How to Run & Develop

- **Play the game**: Open `index.html` directly in a browser, or serve it via any HTTP server (e.g., `python -m http.server 8000`).
- **Development**: Edit `index.html` directly. Changes take effect on page reload.
- **No build step needed**—CSS and JavaScript are inline.

## Architecture

The game is a self-contained IIFE (Immediately Invoked Function Expression) that manages all state and rendering. Key sections:

### 1. **State Management** (lines 863–869)
- `player`: Position, speed, weapon, level, invulnerability timer
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Collision and movement tracked each frame
- `boss`: Current boss (null if not spawned)
- Game flags: `running`, `paused`, `gameOver`, `stage`, `score`, `lives`, `chain`/`multiplier`

### 2. **Canvas & Rendering** (lines 520–1638)
- Fixed 480×720 logical canvas, scaled to fit the viewport
- **draw()**: Clears and renders each frame—sky gradient, clouds, player, enemies, bullets, particles, boss
- **draw functions**: `drawPlane()`, `drawBossShip()`, `drawClouds()`
- Particle effects for explosions; screen shake applied via canvas translation

### 3. **Input Handling** (lines 767–861)
- **Keyboard**: Arrow keys or WASD to move, Space to fire, X/Shift to bomb, P/Esc to pause
- **Mouse**: Pointer position controls player target (smooth steering)
- **Touch**: Left zone (move) with virtual joystick, right zone (fire/bomb buttons)
- `firing` flag drives weapon cooldown

### 4. **Game Update Loop** (lines 1148–1302)
- **update()**: Called each frame to move entities, handle collisions, and spawn enemies/bosses
- **Collision detection**: `rectsOverlap()` checks all bullet-enemy, bullet-boss, enemy-player, bomb-all interactions
- **Player movement**: Priority: keyboard/joystick > mouse steering (keyboard never conflicts with mouse)
- **Scoring**: Kill combo chain (`chain`) resets after 2.2 seconds idle; multiplier scales with chain (1–10x)

### 5. **Enemy Spawning & AI** (lines 1022–1227)
- `spawnEnemy()`: Picks type (grunt, interceptor, tank) based on stage; adds random weaving and shoot patterns
- Enemy types have fixed stats (color, size, HP, speed, score value)
- Shoot patterns: "spread" (3 bullets in a fan) or "aimed" (track player)
- Enemies spawn until 18 kills (`KILLS_PER_STAGE`), then boss warning triggers

### 6. **Boss System** (lines 1043–1439)
- **spawnBoss()**: Creates boss with stage-based design; 6 boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) cycle by stage
- **Boss patterns**: Each design has 3 attack patterns (radial bursts, aimed spreads, homing, columns, spirals, etc.). See `BOSS_PATTERNS` array
- **Boss telegraph**: Red core flash and sound before attack; safe window to dodge
- **updateBoss()**: Horizontal movement, pattern triggering, HP tracking
- Boss defeated → stage advances, combo reset, new boss preview

### 7. **Audio (WebAudio Synth)** (lines 542–676)
- No external audio files; all sound is generated in real-time using Web Audio API
- `beep()`: Tone synthesis with optional frequency sweep, filter, panning
- `noiseBurst()`: Noise-based explosion/bomb sounds
- `sfx` object: Named presets (shoot, laser, explode, bomb, powerup, boss sounds, UI click, game over)
- Master gain and compressor for audio balancing

### 8. **Weapon System** (lines 871–1146)
- **Vulcan**: Straight bullets (up to 3 wide at level 3); rapid fire, medium speed
- **Laser**: Piercing beams (up to 3 lanes at level 3); slower fire rate, higher damage per hit, pierce multiple enemies
- Weapons cycle on level overflow; 3 levels max per weapon type
- Power-ups drop from enemies: life (+1 HP), bomb (+1 bomb), weapon (+1 level or swap)

### 9. **Difficulty Settings** (lines 485–489)
- Easy: Slower spawns, slower enemies, lower boss HP
- Normal: Balanced (default)
- Hard: Aggressive spawns, fast enemies, high boss HP
- Stored in localStorage; difficulty affects spawn rates, enemy behavior, and boss multipliers

### 10. **Persistence & UI** (localStorage keys line 463–466, UI elements 428–461)
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Sound on/off
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility toggle (respects `prefers-reduced-motion` OS setting; disables screen shake/flash/animations)

### 11. **Quality Adaptation** (lines 473–483)
- Monitors frame times; if avg > 25ms, reduces particle count and glow effects
- Quality levels: 0 (full effects), 1 (medium, 60% particles), 2 (low, 35% particles)
- Prevents lag by scaling visual complexity

## Key Code Locations

| Feature | Location |
|---------|----------|
| Game loop & update | `update()` (line 1148), `loop()` (line 1640) |
| Rendering | `draw()` (line 1520), `drawPlane()` (line 913), `drawBossShip()` (line 970) |
| Collision & damage | `update()` lines 1232–1295 (bullet-enemy, bullet-boss, collision checks) |
| Enemy spawning | `spawnEnemy()` (line 1022) |
| Boss patterns | `BOSS_PATTERNS` (line 1372), `updateBoss()` (line 1411) |
| Input handling | Canvas event listeners (lines 814–861), keyboard (lines 769–776), touch (lines 814–834) |
| Audio | `beep()`, `noiseBurst()`, `sfx` object (lines 542–676) |
| Weapon firing | `fireWeapon()` (line 1124) |
| Power-up logic | `applyPowerUp()` (line 1304) |
| Game state reset | `resetGame()` (line 875) |

## Gameplay Mechanics to Know

1. **Combo Chain**: Killing enemies without getting hit extends a chain. Every 5 kills = +1x multiplier (max 10x). Chain resets 2.2 seconds after last kill or on damage.
2. **Boss Entry**: After 18 stage kills, boss warning appears; 1.2 seconds later boss enters with attack pattern loop.
3. **Invulnerability**: 1.5 seconds after hit; player blinks while invuln.
4. **Screen Shake & Flash**: Reduced motion toggle disables these (affects accessibility).
5. **Touch Controls**: Joystick on left, fire/bomb buttons on right (different UX than mouse/keyboard).

## Common Tasks

- **Add a new weapon type**: Add to weapon switching logic in `applyPowerUp()` and `fireWeapon()`.
- **Adjust difficulty balance**: Modify `DIFFICULTIES` object (line 485).
- **Add a new enemy type**: Define in `ENEMY_TYPES` (line 502), add to `EXPLODE_SFX` (line 678).
- **Create a new boss**: Add design to `BOSSES` (line 508), add patterns to `BOSS_PATTERNS` (line 1372).
- **Tweak graphics**: Modify canvas drawing functions (`draw`, `drawPlane`, `drawBossShip`) and CSS gradient/shadow values.
- **Change audio**: Modify `sfx` presets or `beep`/`noiseBurst` calls; adjust `masterGain` for overall volume.

## Performance Notes

- Particle count scales with quality setting to maintain 60 FPS on slower devices.
- Boss patterns use array lookups and pattern indices; adding too many patterns per boss is safe (no re-rendering cost).
- Canvas is fixed 480×720; scaling is CSS-only (efficient).
- localStorage reads happen once on startup; writes are infrequent (at end of game or UI toggle).
