# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It's a single-file HTML5 game (1669 lines) with embedded CSS and JavaScript, requiring no external assets or build process. Open `index.html` directly in a browser to play.

## Architecture

The game uses a traditional **game loop pattern** with three main phases:

### 1. Input Handling
- **Keyboard**: Arrow keys or WASD for movement, Space/X/Shift for fire/bomb
- **Mouse**: pointer position controls aim, click to fire
- **Touch**: left zone = joystick movement, right zone = fire/bomb buttons
- Input state is polled in the `update()` function; no event-driven actions except for pause/mute

### 2. Update Logic (`update(dt)` function)
Runs game physics, collision detection, and entity lifecycle:

- **Player movement**: WASD/arrows or mouse tracking (whichever is active)
- **Weapon firing**: cooldown-based, spreads based on weapon type and level
- **Enemy spawning**: difficulty-adjusted spawn timer
- **Enemy behavior**: movement patterns, weaving, shooting with aim/spread patterns
- **Boss logic**: handled in `updateBoss(dt)`, manages attack patterns and telegraphing
- **Collision detection**: AABBs via `rectsOverlap()`, checks bullets→enemies, bullets→boss, enemies→player, bombs clear screen
- **Score/chain system**: kills extend combo timer, multiplier increases per 5 kills
- **Particle lifetime decay**: explosions fade and disappear

### 3. Render Loop (`draw()` and RAF loop)
- Theme-based layered background: gradient sky → sun glow → two parallax cloud layers
- Entity rendering with optional glow effects (quality-dependent)
- HUD: score, best, weapon, lives, bombs, boss health bar
- Screen shake offset applied before all draws
- Flash effect (white screen flash) for damage/bombs

## Key Data Structures

### Entity Objects
- **player**: `{x, y, w, h, speed, cooldown, rapid, weapon, level, invuln}`
- **bullets**: `[{x, y, w, h, speed, vx, pierce, laser}]`
- **enemies**: `[{x, y, w, h, speed, hp, color, dark, type, pattern, canShoot, ...}]`
- **enemyBullets**: `[{x, y, w, h, speed, vx}]` (or `{...radial, dx, dy}` for boss patterns)
- **boss**: `{x, y, w, h, hp, maxHp, design, telegraph, entering, vx, patternIndex, attackTimer, ...}`
- **powerUps**: `[{x, y, w, h, speed, type}]` (type: 'life', 'bomb', 'weapon')
- **particles**: `[{x, y, vx, vy, life, maxLife, color}]`

### Configuration Constants
- **DIFFICULTIES**: spawn rates, enemy speed/shoot chance, lives, bombs, boss HP multiplier
- **ENEMY_TYPES**: grunt, interceptor, tank — each with w, h, hp, speed multiplier, color, score
- **BOSSES**: 6 designs with name, hull colors, core color, HP multiplier
- **BOSS_PATTERNS**: array of pattern arrays, one per boss; patterns are functions that fire attacks
- **THEMES**: 6 visual themes cycling with stage (sky gradients, cloud/sun colors)
- **KILLS_PER_STAGE**: 18 kills triggers boss spawn

## Extending the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (e.g., 'scout'): `{w, h, hp, speedMul, color, dark, score, minStage, weave?}`
2. Update `spawnEnemy()` random selection to include new type
3. If custom behavior: modify weaving or pattern logic in `update()`

### Adding a New Weapon
1. Add case to `fireWeapon()` function for new weapon type
2. Define bullet spread, speed, pierce behavior
3. Add firing sound in SFX section
4. Update `weaponLabel()` to name it

### Adding a New Boss Pattern
1. Add pattern function at bottom of BOSS_PATTERNS section (e.g., `fireHelicalWave()`)
2. It receives boss `b` as parameter and pushes to `enemyBullets`
3. Use helper functions like `fireRadial()`, `fireColumns()`, `fireAimedSpread()`, `fireHomingSeeded()`
4. Attack pattern arrays are indexed by boss design; add new pattern array or extend existing

### Adding Visual Effects
- **Explosions**: call `spawnExplosion(x, y, color, count)` to create particle burst
- **Screen effects**: `screenShake(magnitude, duration)` or `flashHit()`/`flashBomb()` for white flash
- **Announcements**: `announce(text)` for screen reader accessibility

## Performance & Optimization

- **Adaptive quality**: `quality` variable adjusts particle counts and glow effects (0=full, 1=medium, 2=low)
- **Entity pooling**: arrays are spliced/filtered each frame; no object pooling (acceptable for this scale)
- **Canvas layers**: single 2D context; no WebGL (performance cost is minimal)
- **Audio**: WebAudio synth system; "beep" and "noiseBurst" are synthesized, no file I/O

## Audio System

All sounds are synthesized using WebAudio API oscillators and noise:
- **beep(freq, dur, type, vol, opts)**: generates square/sine wave, supports frequency sweep and filter
- **noiseBurst(dur, vol, opts)**: white noise with envelope
- SFX object maps gameplay events to audio functions (shoot, laser, explode, boss attacks, etc.)
- **Mute state** persisted in localStorage; respects system audio preferences

## Browser & Platform Notes

- **Canvas sizing**: responsive, scales to fit viewport while maintaining aspect ratio
- **Touch**: full joystick UI, gesture support for pause/menus
- **Accessibility**: reduced-motion support (disables animations), screen reader announcements via ARIA live regions
- **Storage**: localStorage for best score, muted state, difficulty, reduced-motion preference
- **No external dependencies**: pure HTML5 Canvas + WebAudio

## Testing & Debugging

To debug in browser console:
- `player`: current player object
- `enemies`, `bullets`, `enemyBullets`, `boss`: current entity arrays
- `score`, `lives`, `stage`: game state
- Pause the game (P) to inspect canvas state without updates

## Modification Examples

**Change enemy spawn rate**: adjust `DIFFICULTIES.normal.spawnBase` (lower = faster)

**Adjust boss health**: modify `baseHp` calculation in `spawnBoss()` or adjust `BOSSES[].hpMul`

**Add weapon level 4**: in `fireWeapon()`, add case `level >= 4` for new spread or behavior

**Increase combo multiplier scaling**: change `multiplier = 1 + Math.min(9, Math.floor(chain / 5))` formula
