# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game (HTML, CSS, and JavaScript) lives in `index.html` (~1669 lines). The game features:

- A player-controlled ship with a multi-level weapon system
- Multiple enemy types (grunts, interceptors, tanks) that scale with difficulty
- Six unique boss encounters across stages
- Three difficulty levels: Easy, Normal, Hard
- A combo/multiplier system for consecutive kills
- Lives and smart bomb mechanics
- Six visual themes with dynamic sky gradients
- Touch controls (virtual joystick) and keyboard input
- Adaptive performance quality adjustment
- LocalStorage persistence (best score, settings)
- Audio synthesis for effects and feedback

## How to Run

1. **Local development**: Open `index.html` directly in a browser
   - Or: Start any local HTTP server and navigate to `index.html`
   - Touch controls are auto-enabled on mobile devices
   - Use mouse/keyboard on desktop (arrow keys or WASD to move, space to fire, E for bomb)

2. **No build process**: The file is self-contained; no compilation, bundling, or dependencies

## Code Architecture

The JavaScript (lines 422–1666) uses a single IIFE (Immediately Invoked Function Expression) that encapsulates the entire game. Key sections:

### State & Configuration (lines 463–520)
- Storage keys and localStorage helpers
- Difficulty settings (spawn rates, enemy stats, lives, bomb counts)
- Enemy type definitions (grunt, interceptor, tank) with colors and properties
- Boss definitions with unique colors and HP multipliers
- Visual themes (six different sky/cloud/sun gradients)
- Adaptive performance tracking

### Game Loop & Main Functions (line 1640+)
- `loop(now)`: Main requestAnimationFrame loop that calls `update()` then `draw()` when game is running
- `update(dt)`: Advances physics, spawns enemies, checks collisions, updates player/enemy/boss state, applies powerups
- `draw()`: Renders canvas background, player ship, enemies, boss, projectiles, particles, and UI

### Player & Controls
- `player` object: tracks position (`x`, `y`), velocity, weapon (type, level), health, bomb count
- `keys` object: tracks pressed keyboard keys
- Touch controls: virtual joystick (visual feedback) and two action buttons (fire, bomb)
- Mouse aim: player always faces mouse position on desktop

### Entities
- **Enemies**: Pool of objects with `x`, `y`, `type` (grunt/interceptor/tank), `hp`, `shootCooldown`, `shootDir`
- **Projectiles**: Player bullets and enemy shots with position, velocity, lifetime
- **Powerups**: Life/bomb/weapon drops that rotate on-screen
- **Particles**: Explosion/hit effect particles with fading opacity
- **Boss**: Single boss object with complex movement patterns and multi-phase HP bar

### Key Systems

**Weapon Leveling**: Player starts with level 1 spread shot; hits/kills increment `player.weaponExp`. At certain thresholds, weapon levels up (levels 1–5), changing spread angle and fire rate.

**Spawning**: Enemy spawn intervals decrease as combo multiplier increases, creating dynamic difficulty. Harder enemies appear only after certain stage thresholds (e.g., tanks at stage 2+).

**Stages**: After 18 kills (`KILLS_PER_STAGE`), the stage increments, theme changes, and a boss appears. Bosses block stage progression until defeated.

**Combo System**: Consecutive hits without taking damage build a multiplier (1–8x). Multiplier affects score and spawn rate. Vignette flash and "combo pop" announce milestones.

**Collision**: AABB (axis-aligned bounding box) detection for bullets vs. enemies, player vs. enemies, player vs. powerups.

**Audio**: WebAudio synthesis (beeps for effects, short clips for events). Audio context created on first user interaction (`ensureAudio()`).

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (e.g., `{ w: 40, h: 40, hp: 2, speedMul: 1.2, color: '#...', dark: '#...', score: 25, minStage: 3 }`)
2. In spawn logic (inside `update()`), add case for new type if special rules needed
3. In `drawPlane()` or add new draw logic if visual appearance differs

### Adding a New Boss
1. Add object to `BOSSES` array with `name`, hull colors, core color, and `hpMul`
2. In `update()`, add new boss behavior case in the boss AI section (movement patterns, shooting, phase changes)
3. Test by reaching that stage (18 kills × stage number)

### Adjusting Difficulty
- Modify `DIFFICULTIES` object: `spawnBase/Min` (lower = harder), `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
- Test with different difficulty selections from the title screen

### Changing Visual Theme
- Add entry to `THEMES` array with `sky` (gradient stops), `cloud`, `sun` color values, and `name`
- Themes cycle automatically as stages progress

### Tuning Game Feel
- Weapon spread angle: controlled in weapon fire code (search `Math.PI / 3` or spread constants)
- Player speed/accel: `player` constants in initialization
- Enemy speed multipliers: already in `speedMul` fields (easy 1.0 → hard 1.9)
- Projectile lifetime: `PROJ_LIFE` or similar constant
- Shake intensity on hit: `shakeTime` variable (search for damage application)

## Testing & Debugging

**In-browser testing**:
- Open DevTools console to catch errors
- LocalStorage can be cleared to reset best score and settings: `localStorage.clear()`
- Throttle CPU in DevTools to simulate lower performance and trigger adaptive quality downgrades
- Test touch controls on mobile device or DevTools device emulation

**Functional areas to test**:
1. Weapon leveling progression (kill enemies, verify spread widens at levels 2, 3, 4, 5)
2. Combo multiplier (rapid kills without hit, verify multiplier shows and resets on damage)
3. Boss fight per stage (reach stage 2+, defeat boss, verify next stage loads)
4. Difficulty scaling (select hard, verify faster spawning and tougher enemies)
5. Powerup pickup (drops on kill, auto-collect near player)
6. Bomb mechanics (start with count, use with E key or bomb button, refill on pickup)
7. Touch controls (on mobile: joystick movement, fire/bomb buttons, pause)

## Code Style Notes

- IIFE wraps all code to avoid global namespace pollution
- LocalStorage keys are prefixed with `skyfighter.` namespace
- Colors use hex strings (e.g., `'#e53935'`) for consistency
- Adaptive quality uses three levels: 0 (full), 1 (medium glow off), 2 (no particle glow)
- No external dependencies; pure vanilla JS + Canvas API
- Comments are sparse; variable names convey intent (e.g., `enemyShootChance`, `bossHpMul`)

## Performance Considerations

- **Particle limit**: Particles auto-fade and are pruned to prevent accumulation
- **Enemy pool**: Enemies are recycled; destroyed enemies are removed from the array
- **Canvas rendering**: Background layers (sky, clouds) are drawn each frame; optimization available by rendering to offscreen canvas if performance degrades
- **Adaptive quality**: Frame time tracking adjusts glow and particle rendering when FPS drops below ~40

## File Structure

```
Google_4/
├── index.html          # Single-file game (HTML + CSS + JS)
├── README.md           # Basic project timestamp
└── CLAUDE.md           # This file
```
