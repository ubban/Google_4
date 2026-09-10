# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and WebAudio. It's a single-file application (index.html) with no external dependencies, designed to run directly in browsers on desktop and mobile devices.

## Running the Game

- **Local Development**: Open `index.html` directly in a web browser, or use any local HTTP server (e.g., `python -m http.server 8000`)
- **No Build Step**: The game runs as-is; there's no compilation, bundling, or build process
- **Browser Support**: Requires ES6+ JavaScript support and Web Audio API (works on all modern browsers)

## Code Architecture

The entire game logic (~1670 lines) lives in a single `<script>` block in index.html, organized into these sections:

### 1. **Initialization & DOM Setup** (lines 423–462)
- Caches all DOM elements (canvas, HUD, buttons, etc.)
- Defines storage keys for persistence (best score, settings)
- Sets up event listeners for buttons and game modes

### 2. **Audio System** (lines 542–676)
- **WebAudio Synth**: Generates all sounds procedurally (no asset files)
- **Functions**: `beep()`, `noiseBurst()`, and a `sfx` object with named effects
- **Design Pattern**: Sound effects are stateless; stereo panning uses canvas x-position via `panFor()`
- **Key Insight**: All sfx are kept short (0.05–0.55s); dynamic compressor manages levels

### 3. **Game Configuration** (lines 473–524)
- **DIFFICULTIES**: Settings for enemy spawning, speed, bullet patterns, lives, bombs by difficulty
- **THEMES**: 6 themed sky gradients with color palettes
- **ENEMY_TYPES**: Definition of grunt, interceptor, tank with HP, speed, scoring
- **BOSSES**: 6 boss designs with color schemes and stat multipliers
- **BOSS_PATTERNS**: Per-boss attack patterns (radial, spirals, homing, cross-fire, etc.)

### 4. **Input System** (lines 768–862)
- **Keyboard**: Arrow keys or WASD to move, Space to fire, X/Shift for bombs, P/Esc to pause
- **Mouse**: Move to steer (non-competitive with keyboard), click to fire
- **Touch**: Dual joystick on mobile—left zone for movement, right zone for fire/bomb buttons
- **Priority**: Keyboard/joystick input overrides mouse steering when active

### 5. **Rendering System** (lines 1507–1638)
- **Canvas Sizing**: Responsive scaling to fit window; base 480×720px
- **Theme System**: Sky gradient, sun glow, and parallax clouds per stage
- **Drawing Functions**: `drawPlane()`, `drawBossShip()`, `drawClouds()`
- **Screen Effects**: Vignette flash on low health, screen shake on hits/bombs, flash on bomb

### 6. **Game Loop & Update** (lines 1640–1302)
- **`loop(dt)`**: RequestAnimationFrame-based loop; tracks performance and throttles dt to 50ms max
- **`update(dt)`**: Handles movement, firing, collision detection, scoring, spawning
- **Performance Adaptation**: `quality` variable scales particle counts and shadow blur (3 levels: 0=full, 1=medium, 2=low)
- **Key Mechanics**:
  - **Collision**: Simple AABB via `rectsOverlap()`
  - **Scoring**: Chain multiplier (x1–x10) resets after 2.2s without kills
  - **Spawning**: Enemy spawn timer decreases with frame count; boss spawns after 18 kills/stage
  - **Powerups**: 6% life, 8% bomb, 20% weapon (random drop on enemy kill)

### 7. **Boss System** (lines 1043–1439)
- **Boss Lifecycle**: Entering phase → attack phase → telegraph → fire → repeat
- **Attack Patterns**: Each boss has 3 patterns (cycled each attack); patterns use helper functions like `fireRadial()`, `fireAimedSpread()`, `fireCross()`
- **Telegraph**: 22-frame delay before attack fires; sfx and visual cues player
- **HP Calculation**: `baseHp * difficultyMul * bossMul` where bossMul varies per design

### 8. **Game State Management** (lines 863–911)
- **Global State**: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, `chain`, `multiplier`
- **Persistence**: localStorage for best score, mute state, difficulty, reduced motion preference
- **Lifecycle**: `resetGame()` → `running=true` → `loop()` → `endGame()` / `togglePause()`

## Key Patterns & Conventions

### Difficulty Scaling
- Spawning rate, enemy speed, shoot chance all scale by difficulty
- Boss HP scales differently per design (0.8x–1.7x multiplier)
- Lives and bomb counts vary (easy: 4 lives/3 bombs; normal/hard: 3 lives/2 bombs)

### Collision & Damage
- All hitbox checks use AABB (`rectsOverlap`)
- Bullets pierce armor (laser only); enemy HP decrements per bullet
- Player invulnerability after hit (1.5s)

### Scoring & Chain System
- `addScore()` increments chain counter and score; resets if no kills for 2.2s
- Multiplier ranges x1–x10 based on chain (`1 + floor(chain/5)`)
- Milestone popups appear at 10, 20, 30... kills

### Weapon System
- **Vulcan**: Spreads based on level (level 1: 1 bullet, level 2: 2, level 3: 3)
- **Laser**: Lane-based (level 1: 1 lane, level 2: 2 lanes, level 3: 3 lanes); pierce=2 at level 3
- Weapon swaps reset level to 1; powerup upgrades level or swaps weapon

### Mobile Adaptation
- Touch controls enabled on devices with `ontouchstart` or `maxTouchPoints > 0`
- Joystick appears only when moving in left zone; constrained to 50px radius
- Button firing is via touchstart (not touch end) for responsiveness

### Accessibility
- Live region (`#srLive`) announces score milestones, deaths, stage transitions, game over
- Reduced motion preference respected (can toggle via button)
- Keyboard shortcuts: P/Esc pause, X/Shift bomb, arrow/WASD move

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with properties (w, h, hp, speedMul, score, minStage)
2. Add explosion sfx to `sfx` object (e.g., `explodeNewType`)
3. Add key to `EXPLODE_SFX` mapping
4. Update enemy spawn logic in `spawnEnemy()` to include in choices

### Adding a New Boss Pattern
1. Create a pattern function in the boss patterns section (e.g., `fireWaveSpread()`)
2. Add it to the appropriate boss's pattern array in `BOSS_PATTERNS`
3. Pattern receives `boss` object and fires into `enemyBullets` array

### Balancing Difficulty
- Adjust `DIFFICULTIES` spawn timers, enemy speed multipliers, and shoot chances
- Modify `BOSS_PATTERNS` firing rates and patterns for individual bosses
- Test via localStorage persistence (difficulty selector saves to `STORAGE_DIFF`)

### Tuning Performance
- Quality levels (0=full, 1=medium, 2=low) automatically adapt based on frame time
- Adjust particle spawn counts in `spawnExplosion()` to reduce GPU load
- Disable shadow blur in rendering section for very low-end devices

## Storage & Persistence

Game saves these keys to localStorage:
- `skyfighter.bestScore`: Best score across all sessions
- `skyfighter.difficulty`: Last selected difficulty (easy/normal/hard)
- `skyfighter.muted`: Whether audio is muted (0/1)
- `skyfighter.reducedMotion`: Reduced motion preference toggle (0/1)

All data is cleared when player opens game fresh; no server-side saves.
