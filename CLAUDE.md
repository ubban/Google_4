# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game written as a single-file HTML application. The game features player-controlled aircraft, multiple weapon types, enemy waves, and boss encounters. It uses Canvas for rendering and the WebAudio API for procedurally-generated sound effects (no external audio assets).

## Architecture & Key Systems

### Single-File Structure
The entire game is contained in `index.html` (~2800 lines):
- **CSS** (lines ~10-358): Responsive UI, animations, dark theme with safe-area support
- **HTML** (lines ~361-420): Game canvas, HUD elements, control zones, overlay UI
- **JavaScript** (lines ~422-end): Complete game engine

### Core Game Systems

#### Game State Management
- `running`: Whether active gameplay is happening
- `paused`: Pause state (toggle with P/Esc)
- `gameOver`: Game over state
- `stage`: Current stage number (increases after defeating boss)
- `stageKills`: Enemy kills needed to spawn boss (18 per stage)

#### Entity Management
Four main entity arrays drive the game:
- `enemies`: Spawned grunt/interceptor/tank enemies with HP, movement patterns, and shoot behavior
- `bullets`: Player bullets with pierce/laser flags for weapon behavior
- `enemyBullets`: Hostile fire (straight, radial, homing patterns)
- `powerUps`: Dropped on enemy death (life, bomb, weapon upgrades)
- `particles`: Explosion effects (color-based, physics-driven)

**Key collision detection:** `rectsOverlap(a, b)` tests all entity-entity and entity-player collisions per frame.

#### Weapon System
Two weapon types, each with 3 upgrade levels:
- **Vulcan**: Projectile-based; spreads cone at L2+ (spread angles stored in arrays)
- **Laser**: Beam-based with pierce count (ignores enemy HP cost unless pierces exhausted)

Weapon switching unlocks after 3 Vulcan upgrades, resetting to Laser level 1.

#### Difficulty System
`DIFFICULTIES[easy|normal|hard]` controls:
- Enemy spawn rates and minimum gaps (`spawnBase`, `spawnMin`)
- Enemy speed multiplier and shoot probability
- Lives/bombs at start
- Boss HP multiplier

Affects all spawning and behavior through difficulty-dependent calculations.

#### Boss System
- 6 boss designs cycle per stage (`BOSSES` array)
- Each boss has 3 attack patterns (simple → complex bullets)
- `BOSS_PATTERNS[designIndex]` holds pattern functions (radial, homing, aimed spreads, columns, cross, spiral)
- Boss telegraph phase (22 frames) before attack, then cooldown (85 frames)
- Damage is hardcoded: 1 HP per bullet hit, 3 HP from player bullets, 24 HP per bomb

#### Scoring & Combo
- Base enemy score: 10-35 points (type-dependent)
- Multiplier: `1 + Math.min(9, Math.floor(chain / 5))` (scales by 5-enemy chains)
- Chain breaks on miss or boss depletion
- Boss kill: flat 500 points
- Bombs grant 5 points per enemy cleared

#### Visual Themes
6 sky gradients and color schemes rotate per stage. Each theme affects:
- Sky gradient (3 colors for linear gradient)
- Cloud and sun particle colors
- Overall aesthetic without gameplay impact

#### Input Handling
- **Keyboard**: Arrow keys/WASD (move), Space (fire), X/Shift (bomb), P/Esc (pause)
- **Mouse**: Movement steers to cursor, click fires
- **Touch**: Left 55% is move zone (drag), right 45% has FIRE/BOMB buttons
- Joystick visualization drawn when touch-active

#### Audio
WebAudio synthesis (no samples):
- Oscillators with configurable sweep, filter, pan
- Dynamic compression on master gain
- SFX: shoot, laser, explode variants, powerup, hit, bomb, boss patterns
- No background music (could add with oscillator + envelope)

#### Performance Adaptation
`quality` (0=full, 1=medium, 2=low) adjusts:
- Particle emission counts (spawn fewer at lower quality)
- Shadow rendering (blur values drop)
- Frametime tracking (45-frame rolling average triggers quality shift)

### File Organization Patterns

**Constants at top (~lines 463-517):**
- Storage keys for persistence
- Difficulty presets
- Enemy types with spawn conditions
- Boss designs and attack patterns
- Visual themes

**Game loop (~lines 1500+):**
- `update(dt)` runs per frame, handles all physics and collision
- `render()` draws canvas, HUD, overlays
- `requestAnimationFrame` maintains smooth animation

**Render functions (scattered):**
- `drawPlayer()`, `drawEnemy()`, `drawBoss()`: Entity visuals
- `renderHearts()`, `renderBombs()`: HUD stat display
- `flashHit()`, `flashBomb()`: Screen flash effects
- `screenShake()`: Camera jitter logic

## Development Workflow

### Testing Changes
1. Open `index.html` in a browser (works offline, no build needed)
2. Use browser DevTools console to inspect/modify game state (e.g., `lives = 10`)
3. Reload to reset to title screen
4. Game saves best score and difficulty preference to `localStorage`

### Common Modifications

**Add new enemy type:**
1. Add entry to `ENEMY_TYPES` with `w`, `h`, `hp`, `color`, `score`, `minStage`
2. Reference in `spawnEnemy()` — enemies pick from types matching current stage
3. Optional: Add `weave: true` for serpentine movement
4. Add explosion SFX in `EXPLODE_SFX` map if unique sound needed

**Add new boss attack pattern:**
1. Add function to `BOSS_PATTERNS[bossIndex]` array
2. Pattern function receives boss object `b`, fires via `enemyBullets.push()`
3. Radial bullets use `{x, y, w, h, radial: true, dx, dy}` format
4. Straight bullets use `{x, y, w, h, speed, vx}` (legacy format)
5. Boss cycles patterns via `b.patternIndex % patterns.length`

**Adjust difficulty balance:**
- Edit `DIFFICULTIES` spawn/speed values
- Tweak `ENEMY_TYPES` HP/score (higher minStage = later unlock)
- Modify boss `hpMul` per design or difficulty multiplier in `spawnBoss()`
- Weapon pierce/spread in `fireWeapon()` (Vulcan spread angles, Laser lane offsets)

**Extend audio:**
- New SFX: call `beep(freq, duration, type, vol, opts)` or `noiseBurst()`
- Background music: use oscillator + envelope in update loop
- Panning: use `panFor(x)` helper to convert screen X to stereo pan

### Performance Considerations

- **Canvas size:** Fixed to window dimensions (high-DPI scales smoothly)
- **Particle count:** Capped per quality level; always prune dead particles
- **Collision**: Uses bounding-box only (no pixel-perfect, intentionally)
- **Adaptive quality**: Triggered by 45-frame FPS average; lowers shadow blur, particle spawn
- **Reduced motion:** CSS animations disabled if user prefers; no screen shake, flash, pulse

### Accessibility Features

- **Screen reader support:** `aria-live` region + `announce()` function narrates key events
- **Keyboard-first:** All actions mappable to keys; no mouse-only mechanics
- **Reduced motion toggle:** Button in-game; respects `prefers-reduced-motion` system setting
- **Color contrast:** HUD uses white text on dark semi-transparent backgrounds
- **Mobile safe areas:** `env(safe-area-inset-*)` for notch/dynamic island safety

## Key Variables & Functions Reference

### Game State
- `score`, `bestScore`: Current and personal best score
- `lives`, `maxLives`: Health system
- `bombCount`, `maxBombs`: Special ability ammunition
- `multiplier`, `chain`, `chainTimer`: Combo system
- `difficulty`: Current preset (easy/normal/hard)
- `stage`: Level counter
- `frame`: Global animation frame counter

### Player Object
- `player.x, player.y`: Position on canvas
- `player.w, player.h`: Bounding box (28×30)
- `player.weapon`: 'vulcan' or 'laser'
- `player.level`: Upgrade level (1-3)
- `player.invuln`: Invulnerability timer after hit
- `player.cooldown`: Fire rate limiter

### Key Functions
- `update(dt)`: Physics, collision, enemy AI each frame
- `fireWeapon()`: Spawn bullets based on weapon/level
- `useBomb()`: Clear all enemy bullets, damage boss, clear enemies
- `spawnEnemy()`: Add random enemy with type/pattern
- `spawnBoss()`: Transition to boss phase after 18 kills
- `addScore(base)`: Increment score and chain multiplier
- `rectsOverlap(a, b)`: Bounding-box collision check
- `applyPowerUp(type)`: Handle life/bomb/weapon pickups

## Browser Compatibility

- Modern browsers with Canvas, WebAudio, `requestAnimationFrame`
- Requires ES6 (arrow functions, const/let, template literals)
- Works offline (no external dependencies or network calls)
- Mobile: iOS Safari, Chrome Android (touch controls + safe-area support)

## Notes for Future Work

- **No build step required**: Edit HTML directly, reload browser
- **Persistence via localStorage**: Best score, mute state, difficulty preference survive reload
- **No external assets**: All audio synthesis, all visuals procedural
- **Tight collision**: Bounding-box only; hitboxes are deliberately forgiving
- **Frame-independent physics**: All movement scaled by `dt` (delta time in seconds)
