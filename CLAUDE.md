# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Sky Fighter: Project Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-'em-up game built entirely with HTML5 Canvas and vanilla JavaScript. The entire game logic, rendering, and UI are contained in `index.html` (~1670 lines).

## Running the Game

- **Local**: Open `index.html` directly in a browser, or serve with any HTTP server (`python -m http.server`, `npx http-server`, etc.)
- **No build step required** — the game runs as-is
- **No external dependencies** — all libraries are built-in (WebAudio API for synth SFX, Canvas API for rendering)

## Architecture & Core Systems

### Game Loop
The main loop runs via `requestAnimationFrame` and executes 60 FPS:
1. `loop()` calculates delta time and gates the update/draw cycle
2. `update(dt)` handles all game logic: player movement, shooting, collision detection, enemy spawning
3. `draw()` renders the scene to canvas with layering: sky gradient → clouds → entities → particles

**Key state variables** (line ~865):
- `player`: position, weapon, health, invulnerability timer
- `bullets`, `enemies`, `enemyBullets`, `powerUps`: entity arrays
- `boss`: active boss object (null when stage is normal)
- `score`, `lives`, `stage`, `chain`, `multiplier`: game state
- `running`, `paused`, `gameOver`: flow control

### Player & Input
- **Desktop**: mouse steering (glides toward pointer) + Space/arrow keys/WASD for movement + X/Shift for bombs
- **Mobile**: joystick on left half of screen + FIRE/BOMB buttons
- Mouse movement has lower priority than keyboard/joystick input to prevent fighting between input modes

### Weapons & Upgrades
Two weapon types:
1. **Vulcan** (default): spread shot that widens at levels 2-3
2. **Laser**: piercing beams that split into lanes at higher levels; faster fire rate (0.1s vs 0.15s)

Power-ups drop randomly from killed enemies:
- Life: restore up to max lives
- Bomb: restore up to max bombs (difficulty-dependent: 2-3 bombs)
- Weapon: upgrade level or switch weapon type

### Enemy Types
Three enemy types with different HP/behavior (line ~502):
- **Grunt**: basic red enemy, 1 HP
- **Interceptor**: faster, weaves side-to-side, 1 HP
- **Tank**: slow tanky purple type, 4 HP, shows health bar; gated to stage 2+

Each has distinct death particles and sound effects.

### Boss System
Six boss designs cycle per stage (line ~508). Each boss:
- Has unique visual design and color scheme
- Uses one of six attack pattern sets (line ~1372)
- Enters via smooth animation from top
- Telegraphs (white flash core) before each attack wave
- Fires 3 distinct pattern types per encounter

Boss patterns:
- Radial bursts, aimed spreads, homing missiles, cross-fire, teleport, double rings, columns, etc.
- Each boss has 3 patterns in rotation at difficulty-scaled HP

### Difficulty System
Three presets (line ~485) affect:
- Enemy spawn rate (higher = more frequent)
- Enemy speed and shoot accuracy
- Starting lives and bombs
- Boss HP multiplier

Settings persist in localStorage.

### Chain Multiplier & Score
- **Kills without gap**: each consecutive kill increments `chain`
- **Gap of 2.2s**: chain resets to 1x multiplier
- **Formula**: `multiplier = 1 + Math.floor(chain / 5)` → max 10x at 45+ chain
- Score = base points × multiplier
- Milestone animations at 10, 20, 30+ kills

### Audio (WebAudio Synth)
No audio files — all SFX generated via oscillators and noise:
- `beep()`: pitched tones with optional sweep and filter
- `noiseBurst()`: noise-based effects (explosions, bomb)
- Spatial panning via `panFor()` maps X position to left/right channel
- Master gain + compressor for consistent levels
- Mute state persists in localStorage

### Adaptive Quality
Performance tracking (line ~474):
- Monitors frame times over 45 frames
- Drops quality (0 = full, 1 = medium, 2 = low) if avg frame time > 1/40ms
- Reduces particle counts and disables shadow rendering at lower quality

### Accessibility
- Reduced-motion preference honored: disables animations (vignette pulse, heart beat, flash)
- `aria-live` region announces game state for screen readers
- High contrast UI: dark backgrounds with white text and shadows
- Touch-friendly buttons on mobile with tap feedback

## Key Constants & Configuration

- **Canvas**: 480×720 base (scales up to screen size, max 1.6× scale)
- **Player speed**: 260 px/s (gliding) / 440 px/s (max with boost)
- **Kills per stage**: 18 to trigger boss entrance
- **Boss enter timeout**: 1.2s after warning before spawn
- **Weapon cooldown**: 0.15s (Vulcan) / 0.1s (Laser)

## Debugging & Development Tips

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line ~502) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add render path to `drawPlane()` call in the enemies loop (line ~1603) if custom visuals needed
3. Add explosion SFX entry to `EXPLODE_SFX` (line ~678) if custom death sound
4. Adjust spawn probabilities in `spawnEnemy()` (line ~1028)

### Adding a New Boss Pattern
1. Create a pattern function in `BOSS_PATTERNS` (line ~1372) using helper functions like `fireRadial()`, `fireAimedSpread()`, etc.
2. Patterns fire via `enemyBullets` array with `radial` flag for free-moving projectiles
3. Boss telegraph (white glow) shows 22 frames before pattern fires
4. Access player position via `player.x` / `player.y` for aimed attacks

### Tweaking Game Feel
- **Player speed**: line 877
- **Weapon spread**: line 1127-1143
- **Enemy spawn**: line 1198-1203, difficulty object
- **Boss attacks**: pattern functions + timing in `updateBoss()` (line ~1430)
- **Screen shake**: `screenShake(magnitude, duration)` called in collision/bomb events

### Visual Tweaks
- **Theme colors**: `THEMES` array (line ~493) — each stage has sky gradient, cloud, and sun colors
- **UI styling**: inline `<style>` block (line ~10) uses CSS variables on `:root` for easy recoloring
- **Canvas resolution**: `BASE_W` / `BASE_H` (line 753)
- **Scale up on larger screens**: resize function (line 754)

## Storage Keys
All persistence uses `localStorage`:
- `skyfighter.bestScore`: high score
- `skyfighter.muted`: audio mute state
- `skyfighter.difficulty`: selected difficulty
- `skyfighter.reducedMotion`: accessibility preference

## Performance Considerations
- Particle count scales with quality setting (full = 14, medium = 8, low = 5 per explosion)
- Shadow rendering disabled at low quality
- Enemies remove off-screen with 30px buffer
- Bullets remove off-screen with 20px buffer
- Delta time capped at 0.05s to prevent spiral on frame drops

## Common Modifications

**Change game balance**: Edit `DIFFICULTIES` object and enemy `ENEMY_TYPES` HP/speed multipliers

**Add new weapon**: Add weapon branch in `fireWeapon()` (line ~1124), update `weaponLabel()` (line ~871)

**Add new visual effect**: Use `spawnExplosion(x, y, color, count)` or `screenShake(mag, dur)` at collision points

**Customize controls**: Modify key-checking in `update()` (line ~1153) or touch handlers (line ~814)
