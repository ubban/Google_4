# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based vertical shoot-'em-up game (Raiden-style), built entirely in HTML5 Canvas with Web Audio API for procedurally generated sound effects. It's a single-file game with no external dependencies.

## How to Run

1. **Open the game**: Simply open `index.html` in any modern web browser (or serve it via HTTP)
2. **Controls**:
   - **Desktop**: Mouse to steer, click to fire, `Space` to fire, `X`/`Shift` to drop bombs, `P`/`Esc` to pause
   - **Mobile**: Drag left side to move, tap FIRE button (right), tap BOMB button (top-right)
   - **Keyboard**: Arrow keys or `WASD` to move

There's no build process, no npm dependencies, and no server required. The game works standalone.

## Code Architecture

The entire game logic is contained in `index.html` within a single IIFE (`(function () { ... })()`). The code is organized by functional areas:

### Core Game Systems

1. **Game State** (lines 863-869)
   - `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
   - Score tracking, lives, stage progression, multiplier system
   - `running`, `paused`, `gameOver` flags

2. **Input Handling** (lines 767-861)
   - **Keyboard**: tracks keys in `keys` object
   - **Mouse**: steering via `mouseTarget` (player glides toward cursor when keyboard/joystick inactive)
   - **Touch**: virtual joystick on left side, fire/bomb buttons on right
   - `firing` state managed by both touch buttons and keyboard/mouse

3. **Rendering Pipeline** (lines 690-1638)
   - `draw()`: clears canvas, draws background/theme, all game objects
   - Theme system: 6 different sky gradients + cloud rendering (lines 493-500)
   - Entity drawing: `drawPlane()` for player/enemies, `drawBossShip()` for bosses
   - Particle effects: explosion particles with velocity and fade

4. **Game Loop & Updates** (lines 1640-1652)
   - `requestAnimationFrame` loop; `dt`-based frame-independent updates
   - Adaptive performance tracking (lines 473-483): throttles to medium/low quality if averaging below 40 FPS
   - `update(dt)` handles all game logic; `draw()` renders

5. **Collision Detection** (line 1069)
   - `rectsOverlap()`: axis-aligned bounding-box collision for all entities
   - Checked per-frame: bullets vs enemies, bullets vs boss, enemies vs player, enemy bullets vs player, boss vs player, powerups vs player

### Enemy & Boss Systems

1. **Enemy Types** (lines 502-506, 1022-1041)
   - **grunt**: 1 HP, basic red enemy, score 10
   - **interceptor**: 1 HP, fast red-orange, can weave side-to-side, score 15
   - **tank**: 4 HP, tanky purple, slow, shows health bar, score 35
   - Spawn logic: random type selection by difficulty stage; some enemies spawn after a certain stage

2. **Boss System** (lines 508-515, 1043-1056, 1230, 1371-1408)
   - 6 named boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
   - One boss per stage; each has unique attack patterns stored in `BOSS_PATTERNS`
   - Boss enters from top, bounces left-right, telegraphs attacks, drops on defeat
   - Each boss design has unique colors and HP multiplier

### Weapon & Power-Up System

1. **Weapons** (lines 1124-1146)
   - **Vulcan**: spread increases at levels 2 & 3; fires 1, 2, or 3 shots
   - **Laser**: pierces enemies (2+ layers at high levels); fires 1, 2, or 3 lasers
   - Weapon swap/upgrade via power-ups

2. **Power-ups** (lines 1058-1067, 1304-1322)
   - **life**: restore 1 HP (up to max)
   - **bomb**: restore 1 bomb (up to max)
   - **weapon**: upgrade current weapon level, or swap weapon at max level
   - Spawn rate: 6% life, 8% bomb, 20% weapon

### Audio System (lines 542-676)

All sound effects are synthesized in real-time via Web Audio API—no audio files.

- `ensureAudio()`: lazily initializes `audioCtx` and `masterGain` on first user interaction
- `beep(freq, dur, type, vol, opts)`: generates synth tones with optional sweep, filter, panning
- `noiseBurst(dur, vol, opts)`: white/filtered noise for explosions
- Individual SFX functions in `sfx` object: `shoot`, `laser`, `explodeGrunt`, `explodeTank`, `bomb`, `hit`, `powerup`, `bossHit`, `bossTelegraph`, `bossDown`, `stageClear`, `gameover`, etc.

### Difficulty & Persistence

1. **Difficulty Levels** (lines 485-489)
   - **easy**: slower spawns, weaker enemies, 4 lives, 3 bombs
   - **normal**: medium difficulty (default), 3 lives, 2 bombs
   - **hard**: fast spawns, strong enemies, 3 lives, 2 bombs
2. **LocalStorage** (lines 463-466, 517-527):
   - Best score, mute state, difficulty choice, reduced-motion preference

### Scoring & Progression

- **Score Multiplier** (lines 1087-1101): increases by 1x for every 5 consecutive kills within 2.2 seconds
- **Stage Progression**: Complete 18 kills (`KILLS_PER_STAGE`) to spawn a boss; defeat boss → next stage
- **Difficulty Ramp**: spawn rate decreases over time (`Math.floor(frame / 300) * 4`)

### Visual & UX Features

1. **HUD Display** (lines 368-374): score, best, multiplier, weapon, hearts, bombs
2. **Effects**:
   - Screen shake: reduces to 15% intensity in reduced-motion mode
   - Hit flash, bomb flash animations
   - Combo popup ("10 KILLS!")
   - Stage banner
   - Boss health bar
   - Vignette red pulse when health critical (1 HP)
3. **Accessibility**:
   - Reduced-motion preference (respects `prefers-reduced-motion` media query)
   - Screen reader announcements via `srLive` element
   - Pause menu

## Key Implementation Details

### Performance Optimization

- Adaptive rendering quality (line 474): tracks last 45 frames; if avg > 25ms, downgrade to medium quality; if avg < ~18ms, upgrade to full quality
- Quality level affects:
  - Particle count: full → 60%, medium → 35% of specified count
  - Glow/shadow blur: disabled at low quality
  - Animation frame skips in reduced-motion mode

### Collision System

All entities are axis-aligned rectangles. Collisions are checked every frame with simple bounds overlap:
```
|a.x - b.x| < (a.w + b.w) / 2 AND |a.y - b.y| < (a.h + b.h) / 2
```

### Frame-Independent Movement

All movement uses `dt` (delta time in seconds):
```
position += velocity * speed * dt
```

### Canvas Scaling

- Logical size: 480×720 (lines 753)
- Physical size scales up to fit window (max 1.6× scale) while maintaining aspect ratio
- Touch and mouse input are scaled back to logical coordinates

## Common Development Tasks

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` object (around line 502)
2. Set properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
3. Optional: add `weave: true` for side-to-side movement
4. Enemy selection logic in `spawnEnemy()` filters by stage and random chance

### Adding a New Boss

1. Add to `BOSSES` array with `name`, `hullTop/Bottom`, `wing`, `core` colors, `hpMul`
2. Add corresponding attack pattern array to `BOSS_PATTERNS` (must have same index)
3. Each pattern array contains 2-3 attack functions that fire bullets via helper functions like `fireRadial()`, `fireAimedSpread()`, etc.

### Adding a New Weapon

Modify `fireWeapon()` (line 1124) and weapon-level logic in power-up handling.

### Tweaking Difficulty

Adjust values in `DIFFICULTIES` object (lines 485-489):
- `spawnBase`, `spawnMin`: enemy spawn rate
- `enemySpeed`, `enemyShootChance`: enemy behavior
- `lives`, `bombs`: starting resources
- `bossHpMul`: boss health multiplier

## Notes on Code Style

- Single IIFE to avoid global pollution
- ES6 syntax (arrow functions, `let`/`const`, destructuring)
- Immediate function execution; no import/export
- Extensive use of closures to maintain game state
- Animation via frame counter (`frame++`) rather than time
