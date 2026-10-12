# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 canvas game — a Raiden-style vertical shoot-'em-up with mouse/keyboard/touch controls, procedurally-generated waves of enemies, boss encounters, weapon upgrades, and smart bombs. The entire game (HTML, CSS, JavaScript) is in `index.html` (1600+ lines).

## Getting Started

- **Play locally**: Open `index.html` in a web browser (no build step needed)
- **Edit**: Modify `index.html` directly — changes are live after refresh
- **Test**: Use browser DevTools to check console for errors

## Code Architecture

The game is structured as a single IIFE (Immediately Invoked Function Expression) with these main sections:

### Game State
- `player`: Position, health, weapon level, active weapon type, bomb count
- `enemies[]`: Array of enemies with position, type, HP, velocity
- `bullets[]`: Array of projectiles (player, enemies, bosses)
- `explosions[]`: Particle effects
- `powerUps[]`: Dropped items (weapon upgrades, health, bombs)
- `bosses[]`: Active boss encounters
- `score`, `multiplier`, `waves`: Game progression counters

### Configuration & Difficulty
- `DIFFICULTIES`: Three difficulty presets (easy/normal/hard) tuning spawn rates, enemy speed, lives, bomb count, boss HP
- `THEMES`: Six visual themes with color gradients (Daylight Front, Sunset Corridor, Night Approach, etc.)
- `ENEMY_TYPES`: Grunt, Interceptor, Tank with health, speed, scoring values
- `BOSSES`: Six boss variants (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan) with unique color schemes and HP multipliers
- `KILLS_PER_STAGE = 18`: Enemy kills required to advance stage and spawn boss

### Audio System
WebAudio API synthesis (no external audio files):
- `beep()`: Sine/square wave with optional frequency sweep, filtering, panning
- `noiseBurst()`: Noise-based sounds with filtering
- Audio is controlled by mute button and stored in `localStorage`

### Input Handling
- **Mouse**: Move to steer, click to fire
- **Keyboard**: Arrows/WASD move, Space fires, X or Shift drops bomb, P/Esc pauses
- **Touch**: Left zone for movement (with visual joystick), right zone for Fire/Bomb buttons
- Touch controls only enabled on mobile (detected by `touchstart` events)

### Rendering Pipeline
Main game loop: `update(dt)` → collision/logic → `draw()`

**Key draw functions:**
- `drawPlane()`: Player aircraft with flame effect
- `drawBossShip()`: Boss with hull, wings, core
- `drawClouds()`: Parallax background
- `renderHearts()`, `renderBombs()`: HUD elements
- Canvas is resized on window resize to maintain full-screen

### Game Mechanics
- **Weapon Levels**: 4 levels (VULCAN → LASER → MISSILE → PLASMA) with different fire patterns
- **Boss Fire Patterns**: 6 unique attack types (radial, aimed spread, columns, spiral, cross, homing) triggered in sequence
- **Power-ups**: Weapon upgrades (1/5 chance per enemy kill), health (1/15), bombs (1/20)
- **Combo System**: Multiplier increases on consecutive kills without being hit; resets on damage
- **Adaptive Difficulty**: Frame time tracking auto-reduces particle quality if FPS drops below 40

### Accessibility
- Screen reader announcements for game state changes
- Reduced motion mode (respects `prefers-reduced-motion` OS setting, disables animations/shake/flash)
- Touch keyboard buttons with visual feedback
- High-contrast HUD with text shadows

### Storage
LocalStorage keys for persistence:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Mute state
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference

## Common Tasks

**Add a new enemy type**: Edit `ENEMY_TYPES` object, update `minStage` if stage-gated. Enemy spawn logic is in `spawnEnemy()`.

**Add a new boss**: Add entry to `BOSSES` array. Boss AI is in `updateBoss()` which cycles through fire patterns. Add corresponding fire pattern function if needed.

**Adjust difficulty balance**: Modify `DIFFICULTIES` object tuning values. Test each difficulty to verify wave pacing and difficulty curve.

**Change visual theme**: Edit `THEMES` array. Theme is selected randomly per stage in `resetGame()`.

**Add a visual effect**: Create a new draw function and call it in the main `draw()` loop. Use `flashHit()`, `screenShake()`, `showComboPop()` as examples.

**Tweak weapon balance**: Modify weapon parameters in `fireWeapon()` or individual fire pattern functions (e.g., `fireRadial()`, `fireAimedSpread()`). Adjust fire rate by changing the `weaponCooldown` threshold.

## Performance Considerations

- **Particle count**: Adaptive quality reduces explosion particles when frame rate drops below 40 FPS
- **Render culling**: Enemies/bullets off-screen are skipped in draw step (check in loop bounds)
- **Audio**: Only 2–3 sounds play simultaneously; older notes are stopped to prevent audio saturation
- **Canvas size**: Clamped to `min(800, 90vw)` × `min(600, 90vh)` to balance quality on mobile

## Tips

- Game state is completely reset on `resetGame()` — new stage, new enemy wave, cleared bullets/explosions
- Boss spawns when `kills % KILLS_PER_STAGE === 0`
- Collision detection uses `rectsOverlap()` — simple AABB, no rotation
- Player can be hit only once per frame; damages are queued in `loseLife()`
- Paused state freezes all entity updates but continues to render (allows pause-screen screenshots)
