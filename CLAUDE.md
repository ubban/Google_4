# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sky Fighter is a Raiden-style vertical shoot-'em-up game implemented as a single HTML file with embedded CSS and JavaScript. The game features multiple difficulty levels, boss encounters, weapon upgrades, and dynamic difficulty based on frame rate performance.

## Getting Started

**No build or installation required.** Simply open `index.html` in a web browser to play the game.

For development:
- The entire game is in a single `index.html` file (~2700 lines)
- CSS styling spans lines 10–420
- JavaScript game logic begins at line 422 and uses an IIFE (immediately-invoked function expression) for scope isolation
- All audio is synthesized using Web Audio API; no external assets are required

## Architecture

The game follows a single-file monolithic architecture with these core systems:

### Game Loop & Canvas Rendering
- 60 FPS animation loop via `requestAnimationFrame`
- Canvas-based 2D rendering with adaptive quality levels (full / medium / low) based on frame time performance
- Quality downgrades if avg frame time > 25ms; upgrades if < 17.9ms

### Game State & Progression
- Three difficulty modes: **easy** (4 lives, 3 bombs), **normal** (3 lives, 2 bombs), **hard** (3 lives, 2 bombs)
- Six playable stages, each requiring 18 enemy kills to progress
- Boss encounters after each stage (six unique boss designs: Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan)
- Boss HP multiplier varies per difficulty and boss type
- Stage progression tied to `KILLS_PER_STAGE` constant (line 491)

### Entity Systems
- **Player**: single sprite; position tracked, health/bomb count managed, weapon level
- **Enemies**: three types with distinct behaviors
  - Grunt: basic red enemy, hp=1
  - Interceptor: fast orange enemy with weaving AI, hp=1
  - Tank: slow purple armored enemy, hp=4
  - Each type spawns after a minimum stage and has configurable speed/shoot behavior
- **Projectiles**: player shots and enemy fire, collision detection via AABB
- **Drops**: weapon pickups, health/bomb pickups spawn on enemy death
- **Particle system**: visual feedback for explosions, combos, hits

### Weapon & Power-Up System
- Four weapon types: Cannon (base), Laser, Spread, Missile (unlocked progressively)
- Weapon level ranges 1–5 per stage, reset on new stage
- Pickup drops determine weapon type; same weapon pickup increases level
- Combo system: consecutive hits within time window boost multiplier (×1 to ×3+)

### Audio System
- Web Audio API synthesis only (no external sound files)
- `beep(freq, dur, type, vol, opts)` for tonal sounds with sweep, filter, pan, delay
- `noiseBurst(dur, vol, opts)` for percussion/explosion effects
- Global mute toggle persisted in `localStorage`
- Spatial audio: panning based on X position for stereo effects

### UI/HUD
- Score, best score, weapon name/level, multiplier, remaining lives/bombs displayed top-center
- Boss health bar appears when boss active
- Stage banner and combo popup animations
- Screen shake and flash effects on hits (respects reduced-motion preference)
- Overlay card system for start menu, game-over screen, new best score notification

### Input Handling
- **Keyboard**: arrow keys or WASD to move, Space/Z/Ctrl to fire/bomb
- **Mouse**: move to player X, click to fire
- **Touch/Mobile**: dual-zone layout with joystick on left, fire + bomb buttons on right
- Touch controls enabled on mobile devices; adapted to safe area insets (notches, home bars)

### Storage & Preferences
- Best score, mute state, difficulty, and motion-reduction preference saved to `localStorage` with keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Respects system preference for `prefers-reduced-motion: reduce` on first visit

### Rendering & Themes
- Six visual themes with different sky gradients, cloud colors, and atmospheres (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- Theme selected randomly per run
- Canvas scales responsively; HUD and UI use clamp() for fluid scaling

## Key Constants & Configuration

Located near the start of the JavaScript (lines 463–515):

- `DIFFICULTIES`: spawn rates, enemy speed multipliers, lives, bombs, boss HP scaling per difficulty level
- `KILLS_PER_STAGE`: enemies needed to complete a stage (18)
- `THEMES`: six sky themes with color arrays, cloud/sun RGB values, names
- `ENEMY_TYPES`: object definitions for grunt, interceptor, tank (width, height, HP, speed, score value, minimum stage)
- `BOSSES`: six boss definitions with hull colors, core color, HP multiplier

## Common Development Tasks

### Adding a new weapon type
1. Define the weapon in the weapon-spawning logic (search `weaponNames`)
2. Add fire logic to the player-fire section
3. Update weapon cycling if applicable
4. Adjust drop spawn rates if needed

### Adding a new enemy type
1. Add entry to `ENEMY_TYPES` object with w, h, hp, speedMul, color, dark, score, minStage
2. Add spawn logic in the enemy-spawn section (respects minStage)
3. Add visual rendering in the enemy draw loop
4. Optionally add unique behavior (e.g., weave pattern for Interceptor)

### Adding a new boss
1. Add entry to `BOSSES` array with name, hull colors, core color, hpMul
2. Add rendering code in the boss-draw section
3. Add unique attack patterns in the boss-AI section if desired

### Adjusting difficulty balance
- Modify `DIFFICULTIES` object: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, lives, bombs, `bossHpMul`
- Adjust projectile speeds or player speed in the move/fire sections
- Tweak enemy spawn frequency via spawn-rate logic

### Adding visual feedback
- Use `flashEl.classList.add('hit')` or `.classList.add('bomb')` for screen flashes
- Use `vignetteEl.classList.add('on')` for red vignette on player hit
- Add animations via CSS keyframes (already defined for shake, flash, pulse effects)

## Code Organization Notes

The JavaScript is organized in logical sections via comments (search for `// ---------- Section Name ----------`):
- Adaptive performance tracking
- Difficulty/gameplay constants
- Audio setup and synthesis
- Player initialization and controls
- Enemy spawning and behavior
- Projectile management
- Collision detection and damage
- Rendering (background, enemies, player, projectiles, HUD)
- Game state management (win/lose/stage transitions)
- Event listeners and start/reset logic
- Touch control setup

Within this linear flow, related logic is grouped together; entity updates typically follow entity rendering.

## Testing & Debugging

- Open browser DevTools (F12) and check the Console for any errors
- Set `quality = 2` in console to force low quality (for stress testing) or `quality = 0` for full quality
- Modify `DIFFICULTIES[difficulty]` in console to test balance changes in real-time
- Use `localStorage.setItem('skyfighter.bestScore', 0)` to reset best score for testing
- Adjust `KILLS_PER_STAGE` to speed up stage progression for testing boss encounters

## Accessibility

- Screen reader live region (`#srLive`) announces important state changes (mute, motion, difficulty)
- Reduced-motion support: opacity changes and timing adjustments when enabled
- High contrast UI with text shadows for readability
- Keyboard-fully-playable (no mouse required)
- ARIA attributes on controls and overlay elements

## Performance Considerations

- Adaptive quality system automatically reduces particle count, enemy AI complexity, or render detail if frame rate drops
- Canvas is scaled to fit viewport while maintaining aspect ratio
- Particle pooling can be added if needed (currently particles destroyed after animation)
- Enemy spawn throttling prevents frame-rate spikes during intense battles
