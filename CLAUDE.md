# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. It features stage-based progression, multiple enemy types, boss encounters, weapon upgrades, and smart bombs—all with synthesized audio and responsive touch controls.

The entire game (HTML, CSS, JavaScript) is contained in `index.html` (~1670 lines). There are no build tools, dependencies, or external assets—open the file directly in a browser to play.

## Running the Game

- **Local play**: Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge)
- **File permissions**: Works from `file://` URLs; no server required
- **Responsive**: Adapts to desktop, tablet, and mobile screens; works in portrait and landscape

Game state (best score, mute, difficulty, reduced-motion preference) is stored in `localStorage` under keys prefixed with `skyfighter.`.

## Architecture Overview

### Game Loop & Frame Timing

The game runs via a single `requestAnimationFrame` loop that:
1. Calculates delta time since last frame
2. Updates all game entities (player, enemies, bullets, particles, physics)
3. Detects collisions
4. Renders the canvas with all sprites
5. Tracks performance and adaptively lowers quality if FPS drops

Adaptive quality reduces sprite counts and visual effects without pausing the game, maintaining 60 FPS on lower-end devices.

### Core Entity Systems

- **Player**: Single ship at canvas bottom; position, weapon state, lives, invulnerability timer
- **Enemies**: Pool of dynamically spawned `grunt`, `interceptor`, and `tank` types (stage-gated)
  - Each type has distinct speed, AI, and visual appearance
  - Shoot at random intervals; movement is simple vertical + occasional horizontal weave
  - Grant score on death; drop weapon/bomb powerups
- **Bullets**: Player and enemy projectiles (separate arrays); simple linear movement, destroy on screen exit or hit
- **Particles**: Transient visual effects (explosions, score popups); fade out and remove
- **Boss**: Single large enemy per stage; health bar, telegraphed attacks, boss-specific explosion SFX

### Stage & Difficulty Progression

- **Stages**: Kill 18 enemies per stage to trigger boss spawn; 6 bosses in sequence (then restart)
- **Difficulty tiers** (stored in `localStorage`):
  - `easy`: Slower spawning, weaker enemies, 4 lives, lower boss HP
  - `normal`: Balanced difficulty (default); 3 lives
  - `hard`: Fast spawning, aggressive enemies, higher boss HP multiplier
- **Difficulty object shape**: `{ spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul }`

### Rendering Pipeline

1. **Canvas setup**: Fixed logical size (480×720), scaled to fit window with `Math.min()` to keep aspect ratio
2. **Sky gradient**: Drawn once per frame using one of 6 color themes that rotate per stage
3. **Entities**: Planes and projectiles drawn via 2D canvas API; boss via `drawBossShip()`
4. **HUD**: Lives (hearts), bombs (dynamite), score, multiplier, weapon level—all DOM elements overlaying canvas
5. **Effects**: Flash (hit/bomb), vignette (low health), combo popups, stage banners, boss health bar

### Input Handling

- **Keyboard**: Arrow keys / WASD move; Space / mouse click / touch fire button fires; X / Shift bombs; P / Esc pauses
- **Mouse**: Plane glides toward pointer; left-click to fire
- **Touch**: Left 55% of screen is movement (joystick visual); right 45% has FIRE and BOMB buttons
- **Joystick**: Virtual joystick appears only on touch; tracks touch ID to prevent interference with fire/bomb buttons

Input priority: keyboard/joystick > mouse (mouse only activates if no keys held).

### Audio System

All sound is procedural WebAudio synthesis—no external files. Key functions:
- `beep(freq, dur, type, vol, opts)`: Oscillator with optional frequency sweep, filter, panning
- `noiseBurst(dur, vol, opts)`: Filtered white noise (for explosions, bombs)
- `sfx` object: Map of sound effects (shoot, explode variants, hit, powerup, etc.)
- **Master gain**: 0.9 with dynamic range compressor to prevent clipping
- **Panning**: SFX pan left/right based on enemy X position for spatial audio

Mute state is respected globally; audio context auto-resumes on first input.

### Collision Detection

- **Bullet-Enemy**: Iterate bullets and enemies; check AABB overlap; credit player and add particles
- **Enemy-Player**: Damage player on contact; trigger invulnerability (5-frame flash); screen shake
- **Bomb blast**: On detonation, kill all onscreen enemies/enemy bullets within radius; massive SFX and visual feedback
- **Score chain**: Consecutive kills within ~0.5s without taking damage increase multiplier; combo popup shows chain

### Performance & Motion Preferences

- **Adaptive quality**: Tracks last 45 frame times; if avg > 1/40s, reduce quality; if < 1/56s, increase
- **Reduced-motion mode**: Disables screen shake, flashes, animations for accessibility; toggleable via 🎆 button
- **Respects `prefers-reduced-motion` media query** for system-level preference

## Key Game Mechanics

### Weapons

- **Vulcan**: Rapid-fire normal bullets; default weapon
- **Laser**: Slower fire rate, tighter spread, higher damage per hit
- Weapon level progresses 1–3; higher levels increase fire rate and spread
- Enemies randomly drop weapon powerups; picking up same weapon increases level (capped at 3)

### Bombs

- Smart bombs detonate all visible enemies + enemy bullets in one area
- Limited supply (2–3 per difficulty); refilled on stage clear
- Visual feedback: white flash, massive combo pop (999+ bonus), screen shake
- SFX: Complex multi-tone explosion with layered noise and bass sweep

### Scoring & Multiplier

- Base score per enemy kill: 10–35 points depending on type
- Multiplier starts at 1×; increases by 1 for each kill within chain window
- Chain resets if you take damage or too much time passes
- Score is boosted by multiplier on each kill; "CHAIN ×5" etc. displayed as combo pop
- New best score triggers celebration overlay and SFX

### Enemy Types

| Type | HP | Speed | Score | Introduced | Notes |
|------|----|----- -|-------|------------|-------|
| Grunt | 1 | normal | 10 | Stage 1 | Red, basic fighter; slow, low threat |
| Interceptor | 1 | fast | 15 | Stage 1 | Orange, aggressive; faster, weaves side-to-side |
| Tank | 4 | slow | 35 | Stage 2 | Purple, durable; heavy fire rate |

### Bosses

Six boss designs with escalating difficulty:
1. **Vanguard**: Balanced; 1.0× HP multiplier
2. **Behemoth**: Tanky; 1.35× HP
3. **Specter**: Glass cannon; 0.8× HP
4. **Dreadnought**: Very tanky; 1.55× HP
5. **Wraith**: Balanced; 0.95× HP
6. **Titan**: Hardest; 1.7× HP

Each boss has unique color scheme (hull, wings, core). Boss spawn triggers after 18 stage kills; telegraphs before attacking; low-health final explosion has specialized SFX.

## Development Notes

### State Management

All game state lives in function scope variables (`player`, `enemies`, `score`, etc.) and is reset via `resetGame()`. No external state management needed.

### Important Constants

- `BASE_W = 480, BASE_H = 720`: Logical canvas dimensions (standard mobile portrait)
- `KILLS_PER_STAGE = 18`: Enemies to kill before boss
- `DIFFICULTIES` object: Tweakable per-difficulty spawn rates, speeds, HP multipliers
- `ENEMY_TYPES`, `BOSSES`, `THEMES`: Parallel objects for defining game content

### Canvas Coordinate System

- Origin (0, 0) is top-left
- Positive X is right; positive Y is down
- Player spawns near bottom; enemies fall from top
- Collisions checked via AABB (rectangular bounding boxes)

### Common Edits

- **Change spawn rates**: Edit `DIFFICULTIES[difficulty].spawnBase` / `spawnMin`
- **Tweak enemy speed**: Modify `ENEMY_TYPES[type].speedMul` or base speed in update loop
- **Add new weapon**: Add entry to `sfx` object and fork weapon logic in `updatePlayer()`
- **New theme/boss colors**: Extend `THEMES` or `BOSSES` arrays
- **Adjust difficulty balance**: Tune `bossHpMul`, lives, bomb count in `DIFFICULTIES`

### Performance Considerations

- Rendering is 2D canvas with no offscreen buffer; each draw is direct to screen
- Particle effects (explosions) are pooled and reused; old particles simply set to invisible
- Enemy spawn is throttled via `spawnTimer` to avoid FPS spikes
- Audio uses `audioCtx.currentTime` for precise timing; no setTimeout needed for SFX

### Accessibility

- Screen-reader live region (`#srLive`) announces game events (mute, difficulty change, stage clear)
- Keyboard-only playable (no mouse required)
- Touch controls with large buttons (78×78px minimum)
- Reduced-motion toggle disables all animations; respects system preference
- Focus management not emphasized (single-player game); but no focus traps

## Testing & Debugging

- **No unit tests or test runner**; game is tested by playing in browser
- **Browser DevTools**: Use console to inspect state (`player`, `score`, `enemies` length, etc.)
- **Performance profiling**: Open DevTools → Performance tab; record gameplay; look for dropped frames
- **Audio debugging**: Check `audioCtx.state` and `masterGain.gain.value` in console
- **Canvas scaling**: Temporarily set `canvas.style.width = 'auto'` to debug rendering

## Browser Compatibility

- Requires `requestAnimationFrame`, `localStorage`, Canvas 2D API, WebAudio API
- Works in Chrome, Firefox, Safari 14+, Edge
- Mobile: iOS Safari, Chrome Android, Samsung Internet
- Audio context requires user interaction to start (first click/tap)
