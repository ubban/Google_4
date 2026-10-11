# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application (index.html, ~1669 lines). The game runs entirely in the browser with no external dependencies, featuring synthesized audio via Web Audio API and responsive controls supporting keyboard, mouse, touch, and gamepad input.

## Running and Testing

- **Play the game**: Open `index.html` directly in a modern web browser. The game is fully playable without a build step or server.
- **Test on different devices**: Use browser dev tools device emulation or open on actual mobile/tablet to test touch controls and responsive layout.

## Architecture & Key Game Systems

### Game Entity Hierarchy
The game manages several entity types defined in configuration objects at the top of the script:

1. **Player**: Single player-controlled fighter with movement (cardinal + mouse), weapon firing, and bomb capacity. Stored in `player` object with properties: `x`, `y`, `w` (width), `h` (height), `speed`, `weapon`, `level`, `invuln` (invulnerability timer), `cooldown` (fire cooldown).

2. **Enemies** (`enemies` array): Three spawnable types defined in `ENEMY_TYPES`:
   - `grunt`: Basic red fighter (hp:1, score:10)
   - `interceptor`: Fast orange fighter with weaving pattern (hp:1, score:15)
   - `tank`: Slow, heavily armored purple fighter (hp:4, score:35)
   
   Enemies have per-stage spawn caps (controlled by difficulty settings) and weaving/aimed shooting patterns. Spawn type and stage availability controlled by `minStage` property.

3. **Bosses** (`boss` object): Six unique designs in `BOSSES` array, rotating per stage. Each has:
   - Visual design (color scheme: hullTop, hullBottom, wing, core)
   - HP multiplier (`hpMul`) for difficulty scaling
   - Attack patterns from `BOSS_PATTERNS` array — each boss has 2-3 pattern functions that fire different projectile formations (columns, radial rings, homing, aimed spreads)

4. **Projectiles**: Two separate arrays:
   - `bullets`: Player-fired projectiles (moving upward, pierce-capable)
   - `enemyBullets`: Enemy and boss fire (moving downward, some tracked/homing)

5. **Power-ups** (`powerUps` array): Weapons (`vulcan`, `laser`), bomb stock (`bomb`), and health (`health`). Spawn on enemy kill with random selection.

### Difficulty & Progression

Difficulty levels (`DIFFICULTIES` object) control:
- Enemy spawn rate and minimum spawn interval
- Enemy speed and shooting probability
- Player lives and bomb count
- Boss HP multiplier

Progression is measured in **stages**: player must kill `KILLS_PER_STAGE` (18) enemies to advance. Bosses spawn after each stage clear. Boss type rotates through `BOSSES` by `(stage - 1) % 6`.

### Weapons & Combat

Two weapon types:
- **Vulcan**: Spreads outward; higher levels add spread angles (`level >= 3`: [-0.24, 0, 0.24])
- **Laser**: Vertical lane weapon; higher levels split into multiple lanes and gain pierce (`level >= 3`: 3 lanes + pierce:2)

Firing is frame-rate independent via cooldown timers. Collision detection is axis-aligned bounding box (AABB).

### Scoring & Combo System

- **Base score**: Enemy score value + boss score (500)
- **Combo multiplier**: Increases by 0.2× per kill in quick succession; resets after `chainTimer` expires (5 seconds)
- **Score display**: Updates HUD and stored best score on game over

### Visual Themes

Six theme presets in `THEMES` array control sky gradient, cloud color, and sun color. Theme cycles per stage progression.

## Key Code Patterns

### Main Game Loop
The `loop` function (called via `requestAnimationFrame`) executes:
1. Delta-time calculation (`dt`)
2. Performance tracking (`trackPerf`)
3. Game state update (`update(dt)`)
4. Render pass (`draw()`)

### Update Function Flow
1. Player input polling (keyboard, joystick, mouse)
2. Player movement with clamping to canvas bounds
3. Weapon firing (cooldown-based)
4. Bullet/enemy/power-up physics
5. Collision detection (player-bullet, enemy-bullet, boss-bullet)
6. Enemy spawning/despawning
7. Boss behavior and attack patterns
8. State transitions (stage clear, game over, boss down)

### Drawing & Rendering
- **Custom draws**: `drawPlane()` for player and enemies (procedural triangle with colors), `drawBossShip()` for bosses
- **Projectiles**: Simple rectangles
- **Effects**: Explosions via `spawnExplosion()`, screen shake via `screenShake()`, flash overlays
- **HUD**: Static canvas position for score/health/weapon; separate DOM updates for live stats

### Audio
Web Audio API synth with single oscillator-based beep generation:
- `beep(freq, dur, type, vol, opts)` creates tones with optional pitch sweep, filtering, panning, and delay
- Compressor and master gain for volume management
- SFX presets: `sfx.shoot()`, `sfx.laser()`, `sfx.bomb()`, `sfx.hit()`, `sfx.bossTelegraph()`, etc.
- All audio context creation deferred to first user interaction

## Performance Considerations

- **Adaptive quality**: `quality` variable (0-2) adjusts rendering details based on average frame time. Low quality disables shadow blurs and glow effects.
- **Array filtering**: Bullets and enemies are pruned each frame (`filter()`) to remove off-screen entities.
- **LocalStorage**: Best score, difficulty, mute state, and motion preference persisted across sessions.

## Controls & Input Handling

- **Keyboard**: WASD or Arrow keys for movement; Space for fire
- **Mouse**: Move to aim, click to fire
- **Touch**: Virtual joystick for movement (left zone), fire button (bottom-right)
- **Gamepad**: Dual analog joystick with deadzone; dual buttons for fire and bomb
- **Screen readers**: `announce()` sends status updates to `srLive` div for accessibility

## Game States

- **Menu**: Overlay shown; difficulty selector active
- **Playing**: Game running; HUD and in-game effects visible
- **Paused**: Game frozen; overlay dims play area
- **Game Over**: Overlay shows final stats (score, stage, best); restart prompts difficulty selection

## Common Modifications

### Adding an Enemy Type
1. Define new entry in `ENEMY_TYPES` with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add to the random selection in `spawnEnemy()` check via `Math.random()` threshold
3. Handle any new visual or behavioral code in `drawPlane()` or collision detection

### Adding a Boss Attack Pattern
1. Create new pattern function in `BOSS_PATTERNS` sub-array: `(boss) => { fireXxx(boss, ...args); }`
2. Use firing helpers: `fireColumns()`, `fireRadial()`, `fireDoubleRing()`, `fireAimedSpread()`, `fireHomingSeeded()`
3. Pattern will auto-rotate per boss design when `updateBoss()` triggers attack cycle

### Tweaking Difficulty
Edit `DIFFICULTIES.normal` (or other presets) to adjust spawn timers, enemy speed, shoot chance, lives, or boss HP multiplier.

### Visual Theme Customization
Modify `THEMES` array entries: `sky` (gradient stops), `cloud`, and `sun` use RGB strings (e.g., `'255,100,0'`).

## Notes for Future Development

- **Single file structure**: Entire game is one IIFE in HTML. Consider modular extraction only if game grows significantly.
- **No build step**: Keep dependencies zero for easy deployment and testing.
- **Canvas-based rendering**: Direct 2D context use; no framework overhead. Performance-critical on low-end devices.
- **Accessibility**: Keyboard, touch, gamepad, and screen reader support already in place; maintain on new features.
