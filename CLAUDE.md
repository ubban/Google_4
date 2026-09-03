# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. There is no build process, package manager, or external dependencies — all code (HTML, CSS, JavaScript) is in `index.html`.

## Architecture & Game Structure

The game is organized around a **single game loop** powered by `requestAnimationFrame`. The code structure within `index.html` follows this pattern:

1. **DOM Setup & Initialization** (~lines 360-470): HTML elements, canvas context, UI element references
2. **Configuration Constants** (~lines 485-515): Difficulty settings, enemy types, boss designs, themes
3. **State Variables**: Global mutable state tracking player, enemies, bullets, score, etc.
4. **Utility Systems**:
   - **Audio**: WebAudio API synth-based sound effects (`beep()`, `noiseBurst()`, and `sfx` object)
   - **Input Handling**: Keyboard, mouse, and touch/joystick control tracking
   - **Canvas & Rendering**: Adaptive performance scaling, camera/screen shake
5. **Game Objects**: Factories for spawning enemies (`spawnEnemy()`), bosses (`spawnBoss()`), power-ups
6. **Game Logic**: Core `update(dt)` function, collision detection, weapon/bomb mechanics
7. **Drawing**: Canvas 2D rendering (`draw()` function) with performance-adaptive quality
8. **Main Loop** (~line 1640): The `loop()` function called via `requestAnimationFrame`

### Key Game Entities

- **Player**: Single ship with position, weapon (Vulcan/Laser), level, health/invulnerability
- **Enemies**: Array of enemy objects with types (grunt/interceptor/tank), HP, movement patterns, shooting behavior
- **Boss**: One active boss per stage with health, attack patterns, telegraph (warning) animation
- **Bullets**: Player bullets (spread/pierce-capable) and enemy bullets (straight/homing/radial)
- **Power-ups**: Life/bomb/weapon drops from killed enemies
- **Particles**: Explosion effects, persisted in array and expired based on lifetime

### Game Flow

1. **Start Screen**: Overlay displays game info, difficulty selector, start button; stores difficulty in localStorage
2. **Wave Combat**: Enemies spawn on timer, scale with game difficulty and elapsed time
3. **Stage Progression**: After ~18 kills, a boss is spawned; boss defeat increments stage and resets enemy wave
4. **Boss Encounters**: Boss has health bar, enters the arena, uses one of 3+ attack patterns (different per boss)
5. **Game Over**: Triggered by loss of all lives; compares score to best score, stores in localStorage

## Key Mechanics & Parameters

### Difficulty Scaling

Three difficulty presets (Easy/Normal/Hard) adjust:
- `spawnBase` / `spawnMin`: Enemy spawn rate (frames between spawns)
- `enemySpeed`, `enemyShootChance`: AI aggression
- `lives`, `bombs`: Starting health/resources
- `bossHpMul`: Boss health multiplier

### Weapons & Upgrade System

- **Vulcan**: Spread shot; upgrades widen spread (levels 1–3)
- **Laser**: Narrow piercing beams; upgrades add lanes and pierce count
- Collecting 3 weapon power-ups cycles weapon (with level reset)

### Score Multiplier

- `chain` counter increments on each kill
- `multiplier = 1 + floor(chain / 5)` (max 10x)
- Chain breaks if `chainTimer` expires (2.2 sec without kills)

### Performance Tuning

- **Adaptive Quality**: Tracks frame time; scales between 0 (full particles/glow/shadow) and 2 (minimal)
- **Reduced Motion**: Disables shakes, flashes, animations if user prefers; respects `prefers-reduced-motion` media query

### Boss Patterns

Each boss has 3 distinct attack patterns accessed via `BOSS_PATTERNS[designIndex]`. Examples:
- `fireRadial()`: Circular burst around boss
- `fireAimedSpread()`: Fan directed at player
- `fireColumns()`, `fireCross()`, `fireHomingSeeded()`: Specialized attack shapes

## Canvas & Rendering

- **Base Resolution**: 480×720 (logical canvas size)
- **Scaling**: Responsive; scales up to ~1.6× on large displays, maintains aspect ratio
- **Drawing Pipeline**:
  1. Clear canvas with sky gradient (theme-specific)
  2. Draw background clouds/sun
  3. Draw all game objects (player, enemies, boss, bullets, power-ups, particles)
  4. Apply screen shake offset to camera transform
  5. Render UI (HUD, health/bombs, boss bar, combo pop-ups)

All drawing happens in the `draw()` function; rendering is conditional on `quality` level for performance.

## Input & Control

- **Keyboard**: Arrow keys or WASD for movement; Space/X/Shift for bomb; P/Esc to pause
- **Mouse**: Move to aim (if mouse is active); click to fire
- **Touch/Mobile**: 
  - Left zone (55% width, 55% height from bottom): movement via joystick
  - Right zone: Fire and Bomb buttons
  - Joystick appears on first touch; base and stick are drawn and positioned in realtime

## Local Storage

Persisted player preferences:
- `skyfighter.bestScore`: Best score achieved (integer)
- `skyfighter.muted`: Mute flag (boolean as '0'/'1')
- `skyfighter.difficulty`: Selected difficulty ('easy'/'normal'/'hard')
- `skyfighter.reducedMotion`: Reduced motion toggle ('0'/'1'), overrides system preference

## Audio System

**No external audio files** — all sound is synthesized via WebAudio API:
- `beep(freq, dur, type, vol, opts)`: Sine/square/sawtooth oscillator with optional sweep and filter
- `noiseBurst(dur, vol, opts)`: Noise with filter envelope
- Audio context is created on-demand (`ensureAudio()`) and only if not muted

Sound effects are keyed in the `sfx` object (e.g., `sfx.shoot()`, `sfx.bomb()`, `sfx.bossDown()`).

## Accessibility

- **Screen Reader Support**: SR-only live region (`#srLive`) for announcements (kills, boss defeat, pause state)
- **Reduced Motion**: Fully respected; animations disabled, shake/flash muted
- **Semantic HTML**: ARIA labels, roles, and keyboard navigation where applicable
- **Safe Area Inset**: Positions HUD and buttons aware of notches/rounded corners on mobile

## Collision Detection

`rectsOverlap(a, b)`: Simple axis-aligned bounding box collision based on center positions and half-widths.

## Common Development Tasks

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` constant with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add drawing logic in the `draw()` function's enemy loop, or use the generic `drawPlane()`
3. Optionally add a sound effect key to `EXPLODE_SFX` and define it in the `sfx` object

### Adding a Boss Attack Pattern

1. Create a new pattern function (e.g., `fireCustom(b)`) that calls `enemyBullets.push(...)` to spawn bullets
2. Add the pattern to the boss's entry in `BOSS_PATTERNS[bossIndex]`
3. Test via `updateBoss()` which cycles through patterns

### Adjusting Difficulty

Modify the `DIFFICULTIES` object constants or add a new difficulty preset, then wire it into the difficulty selector.

### Tweaking Visual Feedback

- Screen shake: `screenShake(magnitude, duration)` function
- Flash effects: `flashHit()` (red flash) / `flashBomb()` (white flash)
- Animations: Added/removed via CSS in `<style>` block; reduced-motion respected via `body.reduced-motion` selector
- Glow/Shadow: Controlled by `quality` level in the `draw()` function

### Testing a Single Mechanic

- Modify `resetGame()` to set `stage`, `lives`, `bombCount`, `player.level` to desired values
- Or add a cheat key (e.g., `if (keys['KeyG']) spawnBoss();`) to the input handling
- Use localStorage debugging in browser console to override `STORAGE_*` keys

## Performance Notes

- **Frame Budget**: Target 60 FPS; `dt` is clamped to 50ms to avoid spiral-of-death on lag spikes
- **Particle Count**: Adaptive; `spawnExplosion()` scales particle count by quality level
- **Enemy Bullets**: Not pooled; recreated each frame (acceptable at typical counts; ~50–200 bullets)
- **Rendering Optimization**: Canvas size fixed at 480×720; scaling via CSS `transform` for smooth mobile performance

## File Size & Assets

- **Single HTML file**: ~70KB with all code, CSS, inline SVG favicon, no external dependencies
- **No image/audio files**: Everything procedurally generated or synthesized
