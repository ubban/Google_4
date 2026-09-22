# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game implemented as a single-file HTML/JavaScript application. The game features a player-controlled fighter jet, procedurally spawned enemy waves, boss encounters across multiple stages, dynamic weapon systems, and synthesized audio using the Web Audio API. This is a self-contained browser game with no external dependencies.

## Quick Start

- **Open/run the game**: Open `index.html` directly in any modern browser (Chrome, Firefox, Safari, Edge)
- **Development**: All code is in `index.html` (1669 lines); edit and refresh the browser to see changes
- **No build step**: The game is production-ready as-is; just deploy the single HTML file

## Architecture Overview

The game operates on a single-threaded animation loop with three main phases:

1. **Update phase** (`update(dt)` at ~1148): Processes player input, moves entities, handles collisions, spawns enemies/bosses, applies scoring logic
2. **Render phase** (`draw()` at ~1520): Draws sky gradient, clouds, all entities (player, bullets, enemies, bosses, effects), HUD overlays
3. **Game loop** (`loop(now)` at ~1640): RequestAnimationFrame-driven, computes delta time, calls update/draw, handles pause state

### Core Game Systems

**Game State Objects** (initialized in `resetGame()` at ~875):
- `player`: Fighter jet with position (x, y), dimensions (w, h), weapon/ammo level, cooldown timers, invulnerability frames
- `bullets`: Array of player projectiles (vulcan bullets or lasers); each has position, velocity, damage properties
- `enemies`: Array of spawned hostile aircraft with position, movement pattern (weave), shoot timers, health
- `enemyBullets`: Array of incoming fire to dodge
- `boss`: Null or current boss object with multi-phase behavior, complex attack patterns, health bar
- `particles`: Visual effects (explosions) for collision feedback
- `powerUps`: Temporary pickups (weapon upgrades, extra lives, bombs)

**Gameplay State**:
- `score`, `multiplier`, `chain`: Scoring system with kill-streak bonuses
- `lives`, `bombCount`: Player resources; bombs trigger screen-clearing effect
- `stage`, `stageKills`, `bossWarned`: Progression; boss appears after 18 kills (KILLS_PER_STAGE)
- `difficulty`: Affects enemy spawn rates, speed, accuracy, health; stored in localStorage

**Timing**:
- `frame`: Game tick counter (incremented each update)
- `spawnTimer`, `shootTimer`: Countdown timers for entity actions (discrete ticks, not wall-clock)
- `chainTimer`: Tracks kill-streak window; expired chain resets multiplier

### Audio System

Web Audio API synthesizer with no audio files. All sounds are generated procedurally:
- `beep(freq, dur, type, vol, opts)`: Sine/square/sawtooth oscillators with optional frequency sweep, filter, stereo pan
- `noiseBurst(dur, vol, opts)`: Filtered white noise for percussion
- Global `sfx` object with named effects (shoot, laser, explode, hit, bomb, powerup, etc.)
- Master gain + compressor chain for mixing
- Mutable; audio context resumption on first user interaction

### Rendering Pipeline

**Canvas-based 2D graphics** (no WebGL):
- Sky gradient (3-color theme) with parallax clouds
- Entities drawn as composite shapes (planes via `drawPlane()`, bosses via `drawBossShip()`)
- Particle system for explosions (circles with fade)
- Glow effects (canvas shadow) on high-quality setting; disabled on low-quality for performance
- HUD: Score, multiplier, weapon, lives/bombs display; boss health bar

**Adaptive Quality** (`trackPerf(dt)` at ~476):
- Monitors frame time over 45 frames
- Auto-scales to 3 quality levels (full → medium → low glow)
- Aims for 40+ FPS; downgrades if frame time exceeds threshold

### Difficulty & Enemy Spawning

**DIFFICULTIES** (at ~485): Easy, Normal, Hard presets control:
- Enemy spawn rate (base, min)
- Speed multiplier
- Shoot accuracy
- Life/bomb counts
- Boss health scaling

**ENEMY_TYPES** (at ~502): Grunt (fast, weak), Interceptor (very fast, weaving), Tank (slow, tanky):
- Each has sprite properties, HP, speed, score value, minimum stage availability

**BOSSES** (at ~508): Six named boss types with:
- Hull color scheme (top/bottom)
- Wing/core colors
- HP multiplier (difficulty adjustment)
- Attack patterns triggered by phase (e.g., radial burst, spiral, homing)

Enemies spawn on rhythm; interceptors weave via sine motion tied to global frame counter.

### Weapon System

**Weapon states** (vulcan vs. laser):
- **Vulcan** (default): Single-shot bullets; fire rate 0.15s; upgradeable via powerups (up to level 5)
- **Laser** (switchable): Continuous line projectile; fire rate 0.1s; pierces enemies up to pierce count
- Firing held via spacebar or touch fire button

**Weapon upgrades** (`applyPowerUp()` at ~1304):
- Increases player.level; affects bullet count, damage, or spread
- Boss patterns scale with weapon level (e.g., homing seeded by level)

### Collision System

`rectsOverlap(a, b)` (at ~1069): AABB collision for all entity interactions:
- Enemy hit detection (player bullets vs. enemies/boss)
- Player hit detection (enemy bullets vs. player; skip if invulnerable)
- Power-up pickup (player vs. powerups)

### Boss Encounters

Triggered after 18 stage kills. Boss behavior:
- Entry animation; health bar displays
- Multi-phase (attacks rotate: radial, aimed, homing, etc.)
- Boss fires every ~120-150 frames (varies by pattern)
- Defeat spawns max-score explosion, stage clear SFX, advances to next stage/theme

### Input Handling

**Keyboard**: Arrow keys or WASD for movement; Space to fire
**Mouse**: Click and drag to set target position (if no stick input active)
**Touch**: Virtual joystick (move zone) for movement; fire/bomb buttons
- Joystick input (`joyVec`) blended with keyboard movement

**UI controls**:
- Mute button (toggles `muted` flag, persists to localStorage)
- Motion button (reduces shake/flash if `reducedMotion` set)
- Pause button (freezes `running`; shows overlay)
- Difficulty buttons (set before game start)

## Code Organization

```
<script>
  Game constants & configuration (DIFFICULTIES, ENEMY_TYPES, BOSSES, THEMES)
  DOM element refs (canvas, UI elements)
  Audio system (beep, noiseBurst, sfx definitions)
  Helper functions (collision, spawn, rendering primitives)
  Game state variables (player, bullets, enemies, score, etc.)
  Event listeners (keyboard, mouse, touch, UI buttons)
  Main game loop (update, draw, requestAnimationFrame loop)
  Initialization (resetGame, setDifficulty, startBtn click)
</script>
```

No classes; all state is module-scoped variables, all logic is functions. Game loop runs at ~60 FPS via `requestAnimationFrame`.

## Common Editing Tasks

**Add a new enemy type**: Add entry to `ENEMY_TYPES` (at ~502), add sprite colors, update `spawnEnemy()` (at ~1022) to select it based on stage/probability

**Tune difficulty balance**: Edit `DIFFICULTIES` presets (at ~485) or adjust spawn timers in `update()` (at ~1198)

**Add weapon pattern**: Create new function like `fireRadial()` (at ~1324), wire into `updateBoss()` (at ~1411) pattern rotation

**Create new SFX**: Add function to `sfx` object (at ~624), use beep/noiseBurst with frequency/pan/filter parameters; test with muted=false

**Adjust visuals**: Edit canvas styles in `<style>` (at ~10), tweak theme colors in `THEMES` array (at ~493), adjust glow/shadow in `draw()` (at ~1520)

**Modify controls**: Update key mappings in `update()` (at ~1153), touch handlers in `joystickStart/Move/End` (at ~785)

## Storage & Persistence

LocalStorage keys (at ~463):
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference

Read on init, write on change. User can manually clear via DevTools or browser settings.

## Performance Considerations

- **Particle cleanup**: Particles are pruned each frame; uncontrolled explosions can spike CPU
- **Enemy count**: 20-30 simultaneous enemies is typical; large waves might tank on low-end devices
- **Canvas size**: Scales with viewport; huge viewports = large render area
- **Audio context**: Lazy-initialized; first sound triggers setup; can fail silently if AudioContext denied
- **Quality downgrade**: Automatic; glow effects disabled at quality=2 to recover frames

For optimization: reduce particle counts in `spawnExplosion()`, cap bullet/enemy arrays, or lower canvas resolution.

## Testing

- Open in browser → manual play-test (verify controls, audio, bosses, scoring)
- Check console for errors (F12 → Console tab)
- Test on mobile (touch controls, safe-area insets)
- Verify localStorage persistence (refresh, check scores persist)
- Stress-test: trigger many explosions, check frame rate dips

No automated test suite; all testing is manual interaction.
