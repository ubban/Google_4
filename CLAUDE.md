# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game runs entirely in the browser with canvas-based 2D rendering and synthesized WebAudio sound effects. No external assets or build process required.

## Quick Start

- **Run**: Open `index.html` in any modern web browser
- **Deploy**: The single `index.html` file is fully self-contained and ready to host
- **No build process**: The game runs directly; no compilation, bundling, or dependencies

## Architecture Overview

The game uses a classic arcade-style architecture with distinct systems:

### Core Game Loop
- `loop()`: Main frame update function (60 FPS target, with adaptive quality scaling)
- `update(dt)`: Game state updates (player movement, collisions, spawning)
- `draw()`: Canvas rendering (clearing, drawing all entities, visual effects)
- Timestep-based: Delta time (`dt`) ensures frame-rate independence

### Adaptive Quality System
- Monitors frame times (`trackPerf()`) and dynamically adjusts visual quality
- `quality` levels: 0 (full) → 1 (medium) → 2 (low) based on FPS
- Reduces particle counts and glow effects on slower devices

### Game State Management
- Player object: position, speed, weapon, health, cooldowns
- Enemies array: multiple types with varying behaviors
- Bullets/enemy bullets: separate tracking for collision detection
- Boss object: single boss per stage with complex AI patterns
- PowerUps: weapon upgrades, extra lives, bombs
- Particles: explosion effects

### Input Handling
Three input methods coexist (keyboard takes priority, then joystick, then mouse):
- **Keyboard**: Arrow keys or WASD for movement, Space to fire, X/Shift for bomb
- **Mouse**: Move to steer (smooth gliding), click to fire
- **Touch**: Left 55% for joystick movement, right 45% for Fire/Bomb buttons

### Difficulty System
Three presets in `DIFFICULTIES` object define spawn rates, enemy behavior, lives, bombs, and boss health multipliers. Settings persist in `localStorage`.

### Enemy Types and Boss AI
- **ENEMY_TYPES**: grunt, interceptor, tank with varying speed, HP, and shooting patterns
- **BOSSES**: 6 unique boss designs, each with 3 attack patterns in `BOSS_PATTERNS`
- Bosses use a telegraph system (visual wind-up) before each attack
- Attack patterns include radial bursts, aimed spreads, homing missiles, teleport barrages

### WebAudio Synthesizer (no external audio files)
- `beep()`: Generates tones with frequency sweeps, filters, and panning
- `noiseBurst()`: Creates noise-based sound effects (explosions, impacts)
- `sfx` object defines all game sounds: shoot, laser, explosions (per-enemy type), boss hit, bomb, powerup, etc.
- Master gain and compressor for consistent audio levels

## Key Systems

### Collision Detection
- `rectsOverlap(a, b)`: Simple AABB overlap check used for all collisions
- Player-enemy, player-bullet, bullet-enemy, bullet-boss, powerup-player

### Scoring and Combo System
- `addScore()`: Increments chain counter and multiplier (1x to 10x based on kill count)
- `breakChain()`: Resets on damage or when timer expires
- Boss kills grant 500 points + explosions

### Visual Effects
- Screen shake: `screenShake(magnitude, duration)` (reduced in reduced-motion mode)
- Flash effects: `flashHit()` and `flashBomb()` with CSS animations
- Combo popup: Displays at 10-kill intervals
- Vignette pulse: Appears when health is critical (1 life remaining)

### Theme System
- 6 rotating sky themes per stage, each with gradient background, cloud color, and sun glow
- All rendered programmatically (no images)

### Accessibility
- Reduced motion preference respected (localStorage + system `prefers-reduced-motion`)
- Screen reader live region for status announcements
- All buttons have accessible titles

### Persistence
Four localStorage keys track player preferences:
- `skyfighter.bestScore`: Highest score achieved
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Visual effect preference

## Important Patterns

### Canvas Rendering
- Always `ctx.save()` / `ctx.restore()` around transforms and property changes
- Quality-dependent rendering: `if (quality === 0)` enables glow/shadow effects
- Frame-based animations: Use `frame` counter (incremented each update) for smooth motion

### Enemy Spawning
- Difficulty affects spawn rate (`spawnBase`, `spawnMin`) and ramps with time
- Weaving interceptors use sine wave with individual `weaveSeed`
- 18 enemy kills trigger boss warning and spawn

### Boss Lifecycle
1. Boss "enters" from top (y < 120) before becoming shootable
2. Telegraph phase (22 frames) with visual wind-up before attack
3. Attack patterns index cycles through `BOSS_PATTERNS[designIndex]`
4. Killing resets stage, increments counter, and ramps difficulty

### Pause Mechanism
- Pauses game state (`running && !paused` guards all updates)
- Overlay shows pause screen; pressing P or Escape again resumes
- Different button text for pause vs. resume

## Editing Considerations

### Game Balance
- Enemy spawn tuning: `DIFFICULTIES` object controls difficulty curves
- Boss HP: `baseHp = 70 + stage * 35` scaled by difficulty and boss design multiplier
- Weapon progression: Vulcan upgrades from 1 → 3 levels; laser has 1 → 3 pierce levels
- Powerup drop rates: Adjust percentages in `maybeSpawnPowerUp()`

### New Features
- **New boss**: Add to `BOSSES` array, assign unique `design` properties (colors), add 3 attack patterns to `BOSS_PATTERNS`
- **New enemy type**: Add to `ENEMY_TYPES` with stats; optional `weave` flag for sine-wave movement
- **New weapon**: Modify `fireWeapon()` branch; track in `player.weapon` string

### Performance Considerations
- Particle count scaled by quality level in `spawnExplosion()`
- Enemy bullet rendering: Circles render faster than rects; no glow on low quality
- Boss glow only on quality 0 (full quality)
- Monitor with browser DevTools Performance tab during high-intensity boss patterns

### Audio
- `ensureAudio()` lazily initializes AudioContext (required for user interaction first)
- All frequencies in Hz; sweep effects use `exponentialRampToValueAtTime()`
- Panning based on screen position: left side negative, right side positive

## Testing

No automated tests; manually verify:
1. All three difficulties spawn correct enemy types and rates
2. Boss patterns fire correctly (check visually for telltale shapes)
3. Collision detection with various spawn positions
4. Audio works (enable speakers, verify mute toggle)
5. Touch controls on mobile/tablet (joystick and buttons responsive)
6. Reduced motion mode disables shake/flash/animations
7. LocalStorage persists across page reloads (best score, difficulty, mute state)
