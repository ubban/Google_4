# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It's a single-file, self-contained browser game implemented in HTML/CSS/JavaScript with no build system, dependencies, or external assets. All game logic, rendering, audio synthesis, and styling are contained in `index.html`.

### How to Run

Open `index.html` in any modern web browser. The game supports:
- **Desktop controls**: Mouse/keyboard for movement and firing
- **Touch controls**: Virtual joystick on mobile devices
- **Difficulty modes**: Easy, Normal, Hard (affects enemy spawn rates, speeds, player lives, and boss HP)
- **Accessibility**: Reduced motion mode, mute toggle, screen reader support

## Code Architecture

The entire game is implemented within a single IIFE (Immediately Invoked Function Expression) starting at line 423. The structure follows a classic game loop pattern:

### Game Loop (`loop()`, line 1640)
- Runs via `requestAnimationFrame`
- Calls `update(dt)` for game state changes
- Calls `draw()` for rendering
- Tracks frame timing for performance monitoring

### Core Game State
- **Player** (tracked via keyboard input, mouse position, and touch controls)
- **Enemies** (`enemies[]`): spawned based on stage progress and difficulty
- **Projectiles** (`playerBullets[]`, `enemyBullets[]`): tracked separately for efficient collision
- **Explosions** (`explosions[]`): particle effects spawned on hits
- **Boss** (optional): one active boss per stage with complex behavior and multi-phase attacks
- **Power-ups** (`powerups[]`): randomly spawned on enemy death
- **Score/Progress**: tracked via `score`, `combo`, `stage`, `lives`, `bombs`, and `weaponLevel`

All game state is scoped within the IIFE; localStorage is used only for persistence (best score, difficulty, mute state, reduced motion preference).

### Key Functions by Category

**Entity Management & Spawning:**
- `spawnEnemy()` (line 1022): spawns grunt/interceptor/tank enemies based on stage and difficulty
- `spawnBoss()` (line 1043): spawns the stage boss with themed appearance and AI
- `maybeSpawnPowerUp()` (line 1058): randomly drops weapon, health, or bomb power-ups
- `spawnExplosion()` (line 1073): creates particle explosion effects with color variance

**Collision & Damage:**
- `rectsOverlap()` (line 1069): simple AABB collision detection for all entities
- `loseLife()` (line 1456): handles player damage, bomb triggers, and game-over logic
- `killBoss()` (line 1441): boss destruction with stage progression and score bonuses
- `applyPowerUp()` (line 1304): weapon/health/bomb pickup logic

**Weapon System:**
- `fireWeapon()` (line 1124): determines fire pattern based on current `weaponLevel` and fires accordingly
- Six distinct weapon fire patterns (lines 1324–1367):
  - `fireRadial()`: spread in all directions
  - `fireAimedSpread()`: spread aimed at nearest enemy
  - `fireColumns()`: vertical columns with spread
  - `fireSpiralBurst()`: rotating spiral pattern
  - `fireCross()`: four-directional cross
  - `fireHomingSeeded()`: homing projectiles toward spawned enemies
  - `fireDoubleRing()`: dual-ring concentric pattern
  Each weapon level (1–6) maps to one pattern; advancing unlocks new patterns.

**Boss AI & Attacks:**
- `updateBoss()` (line 1411): boss movement, targeting, health check, and attack pattern cycling
- Boss attack patterns are hardcoded cycles; boss fires bullets that split on hit or time-decay

**Rendering:**
- `draw()` (line 1520): master render function; clears canvas, draws sky gradient, entities, and HUD
- `drawPlane()` (line 913): player ship sprite (triangle with flame)
- `drawBossShip()` (line 970): multi-part boss sprite with hull, wings, and core
- `drawClouds()` (line 1507): parallax cloud layer

**Audio (WebAudio Synthesis):**
- `ensureAudio()` (line 545): initializes Web Audio API context with compressor/limiter
- `beep()` (line 566): oscillator-based tones with optional frequency sweep and filtering
- `noiseBurst()` (line 597): synthesized noise for impact sounds
- `sfx` object (line 624): pre-configured sound effects (shoot, hit, bomb, wave clear, game over, etc.)
- No external audio files; all sounds are generated in real-time

**UI & Input:**
- `togglePause()` (line 1491): pause/resume with keyboard and button support
- `joystickStart()`, `joystickMove()`, `joystickEnd()` (lines 785–837): virtual mobile joystick
- Keyboard input captured in `keydown`/`keyup` handlers (lines 768–783)
- Mouse tracking for desktop aim
- Canvas is respawn-tracked via `resize()` (line 754)

**Scoring & Difficulty:**
- `addScore()` (line 1087): adds base + combo multiplier, updates best score if exceeded
- `breakChain()` (line 1096): resets combo when player is hit
- `DIFFICULTIES` object (line 485): defines spawn rates, enemy speed, shoot chance, lives, bombs, and boss HP scaling per difficulty
- `THEMES` object (line 493): 6 visual themes (sky gradient, cloud color, sun tint) rotating per stage

### Constants & Configuration

- **`KILLS_PER_STAGE`** (line 491): number of enemy kills required to spawn boss and advance stage
- **`DIFFICULTIES`** (line 485): Easy/Normal/Hard configuration matrix
- **`THEMES`** (line 493): visual theme rotation indexed by stage
- **`ENEMY_TYPES`** (line 502): grunt, interceptor, tank properties (size, HP, speed multiplier, score value, unlock stage)
- **`BOSSES`** (line 508): boss names, colors, and HP multipliers (6 bosses × 6 themes = 36 unique encounters)

### Collision Detection & Physics

- All collision uses axis-aligned bounding box (AABB) via `rectsOverlap()` — no rotation/angle needed since entities are axis-aligned
- Player projectiles vs. enemies: checked every frame; hit clears bullet and applies damage
- Enemy projectiles vs. player: checked every frame; hit damages player health
- Explosions are visual only; they don't collide (particles fall off-screen)
- No gravity or drag; all movement is constant velocity

### Game Flow

1. **Start Screen**: difficulty selection, best score display
2. **Stage**: spawn enemies in waves, player shoots and avoids fire, combo multiplier increases on consecutive hits
3. **Boss Spawn**: after `KILLS_PER_STAGE` enemies, boss appears with health bar and complex attack pattern
4. **Boss Death**: score multiplier bonus, advance to next stage (new theme + more difficult enemies)
5. **Player Defeat**: lose a life (max 3 on Normal), trigger game-over on last life
6. **Pause**: freezes game state; resume or restart from overlay
7. **End Screen**: final score, best score comparison, restart button

## Development Notes

### No Build System
There is no build, bundle, or compilation step. Edit `index.html` directly; changes are live on browser refresh. This means:
- CSS and JS are inline; there's no external stylesheet or script file
- String constants and configuration are hardcoded (no JSON config file)
- All asset generation is embedded (inline SVG favicon, synthesized audio, procedural sprites)

### Key Implementation Details

- **Canvas Scaling**: responsive via `resize()` (line 754) and CSS `max-width: 100%; max-height: 100%`
- **Safe Area**: respects viewport notches via `env(safe-area-inset-*)` in CSS and mobile/joystick positioning
- **Reduced Motion**: toggleable via `reducedMotion` flag; disables screen shake, vignette pulse, and combo animations when ON
- **Accessibility**: screen reader announcements via `#srLive` element; semantic buttons and links; high contrast UI
- **Storage**: `localStorage` keys (`STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`) persist user settings across sessions

### Common Edits

- **Difficulty Balancing**: adjust `DIFFICULTIES` object spawn rates, speeds, or boss HP multiplier
- **Visual Themes**: add/edit entries in `THEMES` array (sky colors, cloud color, sun hue)
- **Enemy Types**: add new type in `ENEMY_TYPES` or modify existing (HP, speed, color)
- **Boss Encounters**: add boss in `BOSSES` array or tweak `updateBoss()` attack patterns
- **Weapon Levels**: edit fire patterns in `fireWeapon()` switch cases or modify fire functions (e.g., `fireRadial()` bullet count)
- **Audio Tuning**: adjust frequencies, durations, or filtering in `beep()` and `noiseBurst()` calls within `sfx` object
- **Score/Combo**: modify `addScore()` multiplier or `combo` threshold in `applyPowerUp()` health pickup logic

### Performance Considerations

- Frame time is tracked in `frameTimes[]` array; average is logged periodically (useful for detecting lag spikes)
- Canvas rendering uses 2D context; no WebGL or GPU acceleration
- Enemy and projectile counts scale with stage but are capped implicitly (small canvas, limited spawn rate)
- Audio context uses a dynamics compressor to prevent clipping on burst attacks
- Reduced motion mode disables expensive animations (vignette pulse, combo text scale-bounce)

## Testing

There is no automated test suite. Verification is manual:
- Open `index.html` in a desktop browser and desktop device (verify mouse/keyboard)
- Test on mobile device or mobile device emulation (verify touch controls and responsive layout)
- Play through multiple stages to check enemy/boss difficulty scaling
- Verify audio playback (especially on first click, since browsers require user gesture to start audio context)
- Check that localStorage persists settings (mute, difficulty, best score) across page reloads
- Toggle reduced motion and verify animations disable
