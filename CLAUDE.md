# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML file. The game features:
- Vertically scrolling gameplay with player aircraft battling enemies and bosses
- Multiple difficulty levels (Easy, Normal, Hard)
- Weapon upgrade system (Vulcan gun with spread, Laser with pierce)
- Smart bombs and lives system
- Six boss encounters with unique attack patterns
- Web Audio API synthesized sound effects
- LocalStorage for persistent high scores and settings
- Touch controls for mobile devices
- Responsive canvas scaling for different screen sizes

## Running the Game

The entire game is in `index.html` — simply open it in a modern web browser. No build step, server, or dependencies required.

### Local Development

- **Browser preview**: Open `index.html` directly in your browser (file:// protocol works fine)
- **With a local server**: For live-reload and clean asset serving, use any HTTP server:
  ```bash
  # Python 3
  python3 -m http.server 8000
  # Python 2
  python -m SimpleHTTPServer 8000
  # Node.js
  npx http-server
  ```
  Then visit `http://localhost:8000/index.html`

## Code Architecture

All code is in a single `<script>` tag within `index.html`, organized as an IIFE (Immediately Invoked Function Expression) to avoid global scope pollution.

### High-Level Structure

The monolithic game is organized conceptually into these sections:

#### Game Configuration & Data
- **Difficulty levels** (`DIFFICULTIES`): Controls spawn rates, enemy speed, lives, bombs, boss HP
- **Enemy types** (`ENEMY_TYPES`): Defines grunt, interceptor, tank with stats (HP, speed, color, score)
- **Boss definitions** (`BOSSES`): Six boss templates with visual themes and HP multipliers
- **Visual themes** (`THEMES`): Six sky/cloud/sun color schemes, one per stage
- **Boss attack patterns** (`BOSS_PATTERNS`): 18 distinct attack functions (6 bosses × 3 patterns each)

#### Rendering Systems
- **Canvas setup**: 480×720 base resolution, scales with `max-width: 100%; max-height: 100%`
- **Drawing functions**: `drawPlane()`, `drawBossShip()`, `drawClouds()`
- **Particle rendering**: Explosion particles with fade and shrink
- **HUD rendering**: `renderHearts()`, `renderBombs()` update health/bomb display

#### Game Loop
- **`update(dt)`**: Called every frame; handles input, movement, collision, spawning
- **`draw()`**: Renders background, entities, effects; applies screen shake and theme colors
- **`loop(now)`**: RAF-driven game loop; tracks delta time, calls update/draw, throttles to ~50ms per frame

#### Input Systems
- **Keyboard**: Arrow keys, WASD for movement; Space to fire; X, Shift, P, Esc for special actions
- **Mouse**: Hovers on canvas; plane glides toward mouse cursor (lower priority than keyboard/joystick)
- **Touch**: Joystick in left 55% of screen; Fire/Bomb buttons in right 45%; dragging the left zone controls movement
- **Priority**: Direct input (keyboard/joystick) overrides mouse auto-steer

#### Game State
- **Player**: Position, health (lives), weapon, level, invulnerability timer
- **Entities**: Bullets, enemies, enemy bullets, particles, power-ups, boss
- **Progression**: Stage, kills, score, combo chain, difficulty setting
- **Persistent**: Best score and settings stored in `localStorage` with keys like `skyfighter.bestScore`

#### Key Mechanics
- **Weapon system**: Vulcan (spread shot, level 1–3) ↔ Laser (piercing, level 1–3)
- **Combo chain**: Resets if player takes damage; multiplies score (1+ up to 10× at 50+ kills)
- **Smart bombs**: Clear all enemies on screen, deal fixed damage to boss; limited by difficulty
- **Boss lifecycle**: Entrance animation → patterns (telegraph + attack) → defeat triggers stage transition
- **Difficulty scaling**: Affects spawn rate, enemy behavior, HP, lives, bombs

#### Audio System
- **Web Audio API synthesizer**: No external audio files; generates tones on-the-fly
- **SFX functions**: `beep()` (oscillator + envelope), `noiseBurst()` (white noise filtered)
- **Mute toggle**: Persisted in `localStorage`
- **Panning**: Enemy explosions pan left/right based on screen position

#### Accessibility
- **Screen reader support**: `#srLive` ARIA live region announces game events
- **Reduced motion**: Preference detection and toggle; disables animations if enabled
- **Keyboard controls**: Full game playable without mouse

## Key Functions for Extension

### Boss Attack Patterns
Boss patterns are pure functions that push bullets into `enemyBullets`:
- `fireRadial(boss, bulletCount, speed)`: Bullets in all directions
- `fireAimedSpread(boss, count, speed, spreadStep)`: Spread aimed at player
- `fireColumns(boss, speed)`: Vertical columns
- `fireDoubleRing(speedA, speedB)`: Two concentric rings
- `fireHomingSeeded(boss, count, speed)`: Homing projectiles with slight randomness

To add a new boss pattern, add a function to `BOSS_PATTERNS` at the appropriate boss index.

### Collision Detection
`rectsOverlap(a, b)` uses axis-aligned bounding box collision. Checks center distance against half-widths.

### Score and Progression
- `addScore(basePoints)`: Adds to score, increments combo chain, updates multiplier display
- `breakChain()`: Resets multiplier to 1× (called on damage)
- High score persisted in `localStorage[STORAGE_BEST]`

### Power-Up System
`applyPowerUp(type)` handles 'life', 'bomb', 'weapon'. Weapon upgrades cycle: Vulcan 1→2→3 → Laser 1→2→3 → repeat.

### Visual Effects
- `screenShake(magnitude, duration)`: Applies frame offset; reduced motion mode dampens
- `spawnExplosion(x, y, color, count)`: Creates particle burst
- `flashHit()` / `flashBomb()`: Flashes white overlay for impact feedback

## Performance Tuning

**Adaptive quality** (variable `quality`):
- Tracks frame times over 45 frames; scales from 0 (full) → 1 (medium) → 2 (low)
- Reduces particles, shadow blur, and particle count at lower quality
- Triggered automatically if average frame time exceeds 25ms

**Optimization opportunities**:
- Particle cap: Tunable in `spawnExplosion()`
- Shadow blur: Disabled at quality > 0
- Enemy bullet optimization: Uses simple circular rendering; could batch

## Development Workflow

### Testing Game Balance
Difficulty settings are in the `DIFFICULTIES` object. Adjust:
- `spawnBase` / `spawnMin`: Time (frames) between enemy spawns
- `enemySpeed`: Global multiplier for enemy velocity
- `enemyShootChance`: Probability an enemy shoots (0–1)
- `lives` / `bombs`: Starting counts
- `bossHpMul`: Boss health multiplier

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add matching explosion SFX to `EXPLODE_SFX` mapping
3. Create explosion function in `sfx` object (e.g., `explodeNewType`)
4. Adjust spawn logic in `spawnEnemy()` to choose the new type

### Adding a New Boss
1. Add to `BOSSES` array with `name`, hull colors, core color, `hpMul`
2. Add 3 attack pattern functions to `BOSS_PATTERNS` at the same index
3. Boss index derived from `(stage - 1) % BOSSES.length` — scales across infinite stages

### Tweaking Visuals
- **Background**: `THEMES` array controls sky gradient, cloud colors, sun glow
- **Entity colors**: Hardcoded in `ENEMY_TYPES`, `BOSSES`, and drawing functions
- **Canvas size**: Change `BASE_W` / `BASE_H` constants; affects all scaling

## Notes on the Monolithic Structure

The entire game runs in a single function scope. Globals are intentionally minimized (only input state and rare toggles like muted). Entity lists (`bullets`, `enemies`, etc.) are array-based with frequent filtering — suitable for indie game scale (~100–200 entities max).

### No Build System
- No Node modules, no transpilation, no minification
- Works in any browser with ES6 support (all modern browsers since ~2015)
- LocalStorage requires HTTPS or localhost (file:// may block access in some browsers)

### Responsive Design
Canvas auto-scales; overlay and HUD elements use `clamp()` and viewport-relative sizing for mobile. Safe-area insets account for notches on iOS.

## Common Modifications

**Change starting lives**: Edit `DIFFICULTIES.normal.lives` (and easy/hard)

**Disable a boss**: Comment out entries in `BOSSES` or adjust `(stage - 1) % BOSSES.length` modulo

**Tweak spawn difficulty**: Adjust `DIFFICULTIES[difficulty].spawnBase` — lower = harder

**Change combo multiplier scaling**: Edit `addScore()` — currently 1 + floor(chain / 5)

**Add music**: Replace `audioCtx.resume()` with a `<audio>` tag and `play()` call (separate from SFX)
