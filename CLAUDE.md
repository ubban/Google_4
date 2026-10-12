# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML file (~1670 lines). It features:
- Canvas-based 2D rendering with procedurally generated WebAudio effects
- 6 boss designs with multiple attack patterns each
- 3 difficulty levels with adaptive performance management
- Progressive stages with varying enemy types and visual themes
- Touch controls (mobile) and keyboard/mouse controls (desktop)
- Persistent high score storage

The entire application is self-contained in `index.html` with no external dependencies — CSS is embedded in `<style>` and JavaScript is in `<script>`.

## Game Architecture

The codebase is organized as a single IIFE (Immediately Invoked Function Expression) with the following major sections:

### Configuration Objects
- `DIFFICULTIES` — Three difficulty presets (easy/normal/hard) controlling enemy spawn rate, speed, shoot frequency, and boss HP multiplier
- `THEMES` — Six visual themes with different sky gradients and cloud colors (cycle per stage)
- `ENEMY_TYPES` — Three enemy classes: grunt, interceptor, tank (with HP, speed, score values)
- `BOSSES` — Six boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with unique color schemes
- `BOSS_PATTERNS` — Array of attack pattern functions, one set per boss design (e.g., radial bursts, aimed spreads, spiral bursts)

### Core Game Loop
```
requestAnimationFrame(loop) → update(dt) → draw()
```
- **`loop(now)`** (line ~1640): Main frame handler; manages deltaTime, calls update/draw, tracks performance
- **`update(dt)`** (line ~1148): Physics, collision, enemy/boss AI, powerup logic
- **`draw()`** (line ~1520): Canvas rendering; sky gradient, clouds, entities (player, enemies, bullets, boss, particles)

### State Variables
- Game state: `running`, `paused`, `gameOver`, `stage`, `score`, `lives`, `bombCount`, `chain`, `difficulty`, `quality` (performance level)
- Collections: `enemies`, `enemyBullets`, `bullets`, `powerUps`, `particles`, `boss`
- Player: `{ x, y, w, h, speed, weapon, level, invuln, cooldown }`
- Input: `keys` (object), `joyActive`, `joyVec`, `mouseActive`, `mouseTarget`, `firing`

### Major Functions

**Input & Control**
- `handleKeyDown/Up` — Keyboard input tracking
- Touch joystick logic (moveZone, actionZone) — Mobile movement and action buttons

**Rendering**
- `drawPlane()` — Renders player and enemy aircraft (line ~900)
- `drawBossShip()` — Renders boss with multi-part design and animations (line ~950)
- `drawClouds()` — Procedural cloud layers with parallax

**Game Logic**
- `spawnEnemy()` — Spawns enemy with type, speed, and attack pattern
- `spawnBoss()` — Begins a boss stage after 18 kills
- `fireWeapon()` — Creates player bullets (Vulcan spread or Laser pierce)
- `useBomb()` — Clears all enemy bullets and damages boss
- `updateBoss(dt)` — Boss movement, telegraph animation, attack pattern execution
- `applyPowerUp()` — Handles life/bomb/weapon pickups
- `loseLife()` / `endGame()` — Damage and game-over logic

**Audio (WebAudio Synthesis)**
- `beep()` — Tone synthesis with frequency sweep and filtering
- `noiseBurst()` — Procedural noise with envelope
- `sfx` object — SFX definitions (shoot, laser, explode variants, bomb, powerup, boss telegraph/defeat, game over)

**Collision**
- `rectsOverlap(a, b)` — AABB collision check
- Collision detection in update loop against player bullets/enemies, enemy bullets/player, powerups/player

## Common Development Tasks

### Running the Game
Open `index.html` in a browser. No build process is required.

### Testing Changes Locally
1. Edit `index.html`
2. Reload the browser tab (Ctrl+R / Cmd+R)
3. Test different difficulty levels via the start menu
4. Use browser DevTools console to inspect game state or temporarily log variables

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line ~502) with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, and optional `weave`
2. Update `spawnEnemy()` (line ~1022) to include it in the spawn pool if desired
3. Add explosion SFX variant to `EXPLODE_SFX` (line ~678) if needed and create corresponding sfx function

### Modifying a Boss Pattern
1. Boss patterns are stored in `BOSS_PATTERNS` (line ~1372); each boss (0–5) has 3 attack functions
2. Edit the pattern array entry for the desired boss (e.g., BOSS_PATTERNS[0] for VANGUARD)
3. Pattern functions receive the boss object and should call bullet-spawn helpers: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc. (lines ~1324–1370)
4. Bullets are added to `enemyBullets` array; collision is handled automatically

### Adjusting Difficulty Parameters
Edit the `DIFFICULTIES` object (line ~485):
- `spawnBase`/`spawnMin` — Enemy spawn rate (frames between spawns); lower = more frequent
- `enemySpeed` — Multiplier applied to all enemy velocities
- `enemyShootChance` — Probability (0–1) enemies open fire
- `lives` / `bombs` — Starting counts
- `bossHpMul` — Boss health multiplier

### Tweaking Visual Themes
Themes are in `THEMES` array (line ~493). Each has:
- `sky`: [top color, middle, bottom] — Used in a linear gradient
- `cloud`, `sun` — RGB color strings for cloud and glow overlay

### Audio Adjustments
All SFX are **procedurally generated** with no external audio files. Edit `sfx` object (line ~624) or call `beep()`/`noiseBurst()` directly. Frequency, duration, and envelopes control the sound; use the DevTools console to experiment or trace the parameter flow in the function definitions.

### Performance Tuning
- `quality` variable (line ~474) adjusts particle count, shadow blur, and visual complexity
- `trackPerf(dt)` monitors frame times and auto-adjusts `quality` (0=full, 1=medium, 2=low)
- Modify thresholds in the `trackPerf` function to be more/less aggressive about downgrading

## Important Implementation Details

**Physics & Collision**
- No gravity; all motion is vector-based
- Entities use axis-aligned bounding boxes (AABB) for collision
- Player movement is clamped to canvas bounds
- Enemy weave pattern (for interceptors) modifies X position based on sine wave

**Boss Telegraph System**
- Boss attack patterns are delayed by `telegraph` countdown (~22 frames)
- Visual feedback: boss flickers with reduced alpha during telegraph
- This gives players time to react before attack fires

**Weapon System**
- Vulcan: tight spread increasing with level (1→3 bullets) with velocity spread
- Laser: pierce mechanic (pass through enemies), more bullets at level 3, faster fire rate
- Switching weapons resets to level 1 and resets the level counter

**Chain/Multiplier Mechanic**
- Kill counter increments on every enemy death; resets if no kill for 2.2 seconds (`chainTimer`)
- Multiplier = 1 + floor(chain / 5), capped at 10x
- All score rewards are multiplied before adding to total

**Touch Controls**
- moveZone (left 55% of screen) for joystick input
- actionZone (right 45%) for FIRE and BOMB buttons
- joystick is shown only when touch is active
- fallback to keyboard/mouse on desktop

**Accessibility Features**
- Reduced motion toggle (defaults to system preference)
- Screen reader announcements via `announce()` function
- All button interactions fire SFX and UI click sound

## Debugging Tips

1. **Inspect game state** in console: Log `stage`, `score`, `enemies.length`, `boss`, `player`
2. **Monitor performance**: Set `quality = 0` to see max effects or `quality = 2` for low-end stress test
3. **Test edge cases**: Pause during boss attack, take damage during invulnerability, use bomb with no enemies
4. **Audio issues**: Call `ensureAudio()` in console if sound doesn't play; WebAudio requires user gesture first
5. **Visual debugging**: Temporarily add `ctx.strokeRect()` calls to `draw()` to visualize collision boxes

## Notes for Future Development

- The codebase prioritizes performance and self-containment over modularity
- No external libraries; all rendering and audio synthesis is native Web APIs
- LocalStorage keys are prefixed with `skyfighter.` to avoid collisions
- Canvas resolution is `800x600` (hardcoded); update `canvas.width/height` assignments if resizing
- The game is touch-friendly and responsive; test on mobile devices when making UI changes
