# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML file (1669 lines). It features a player-controlled fighter jet that shoots enemies, collects power-ups, faces boss encounters, and accumulates score across multiple stages.

**Key Game Features:**
- Weapon system (Vulcan, Laser, Missiles) with upgradeable levels
- 3 enemy types with distinct behaviors and bullet patterns
- 6 unique boss designs with complex attack patterns
- Smart bomb mechanic for clearing bullets
- Difficulty levels: Easy, Normal, Hard
- Particle effects, screen shake, and visual feedback
- Touch and mouse input support with joystick for mobile
- Web Audio API for procedurally generated sound effects

## Getting Started

### Running the Game
The game is a standalone HTML file—no build process or server required:
- **Local file**: Open `index.html` directly in a browser
- **Local server** (optional): `python3 -m http.server 8000` then visit `http://localhost:8000`
- **Mobile testing**: Use a local server and access from a phone on your network to test touch input

### Testing & Development
- **No build step**: All code changes are live upon page reload
- **Testing a change**: Edit the HTML, save, reload the browser to see the effect immediately
- **Debugging**: Use browser DevTools console; the game prints performance metrics and has a `trackPerf()` function for frame timing
- **Local storage**: Best score, mute state, difficulty, and reduced-motion preference are saved to `localStorage` with keys prefixed `skyfighter.*`

## Code Architecture

### High-Level Structure
The entire game runs inside an IIFE (Immediately Invoked Function Expression) at the bottom of the file (`<script>` tag). All game state is stored in module-scope variables, with the main loop driven by `requestAnimationFrame`.

### Core Game Loop (`loop()` function, ~line 1640)
1. Calculate delta time (`dt`)
2. Call `update(dt)` to simulate physics, movement, collisions, spawning
3. Call `draw()` to render the frame
4. Continue with `requestAnimationFrame`

### State Management
**Game state variables** (module scope):
- `player`: Object with `{ x, y, w, h, weapon, weaponLevel, hp, invuln, maxHp }`
- `enemies`: Array of active enemy objects
- `bullets`: Array of player projectiles
- `enemyBullets`: Array of enemy/boss projectiles
- `powerUps`: Array of droppable power-ups (life, bomb, weapon)
- `particles`: Array of explosion particles
- `boss`: Current boss object or null
- `lives`, `score`, `stage`, `combo`, `multiplier`: Game metrics
- `running`, `gameOver`, `paused`: Game state flags

**Difficulty configuration** (`DIFFICULTIES` constant, ~line 485):
- Defines spawn rate, enemy speeds, player speed, health scaling per difficulty
- Affects balance significantly—changes here have cascading effects on game feel

### Key Subsystems

#### 1. **Rendering System**
- Canvas-based 2D rendering with theme-aware gradient backgrounds
- **Theme system** (`THEMES` constant): 6 color themes (sky gradients, sun color, cloud color) that cycle per stage
- **Player rendering** (`drawPlane()`): Draws player ship sprite with adjustable colors
- **Boss rendering** (`drawBossShip()`): Draws boss ships with visual indicators (telegraph effect, health bar)
- **Screen shake**: Applied as `ctx.translate()` offset when `shakeTime > 0`
- **Quality setting** (`quality` variable): Controls glow effects; lower quality adds shadows

#### 2. **Input System**
- **Mouse**: Moves player directly to cursor; click/touch fires weapon
- **Keyboard**: WASD or arrow keys move; Space fires; Q for bomb
- **Mobile joystick**: Custom joystick created via touch events (`joystickStart/Move/End`)
- **Gamepad**: Analog stick support with threshold detection
- **Target visualization**: Crosshair shown on screen when using mouse/touch

#### 3. **Weapon System** (`fireWeapon()` function, ~line 1124)
- **Vulcan**: Fires bullets straight up, auto-fires while held
- **Laser**: Beam weapon with continuous fire and lower fire rate, larger hitbox
- **Missiles**: Slower projectiles with homing behavior toward nearest enemy
- **Weapon levels**: Increase spread, fire rate, or projectile count per level
- Weapons picked up as power-ups; fire frequency tied to difficulty settings in `DIFFICULTIES`

#### 4. **Enemy System**
- **Enemy types** (`ENEMY_TYPES` constant, ~line 502): Grunt (weak), Interceptor (agile), Tank (armored)
- **Spawning** (`spawnEnemy()`): Random type, position, and velocity based on difficulty and progression
- **Kill threshold** (`KILLS_PER_STAGE`): 18 enemies must be defeated before boss appears
- **Collision**: `rectsOverlap()` checks bullet-enemy and player-enemy collisions
- **Score on kill**: Base points adjusted by combo multiplier

#### 5. **Boss System**
**Bosses** (`BOSSES` array, ~line 508): 6 designs with distinct visual styles and health pools
**Attack patterns** (`BOSS_PATTERNS` array, ~line 1372): Each boss has 3 attack patterns that cycle
- **Pattern types**: Radial bursts, aimed spreads, columns, spirals, homing, teleport dashes
- **Pattern selection**: Boss moves through its 3 patterns repeatedly, each pattern holds for ~85 frames with a ~22 frame telegraph (warning flash)
- **Boss states**: `entering` (scroll in), `telegraph` (charge-up), `attackTimer` (cooldown between patterns)
- **Boss spawn**: Appears after `KILLS_PER_STAGE` enemies killed on a stage

#### 6. **Audio System** (`ensureAudio()`, `beep()`, `noiseBurst()`)
- **Web Audio context**: Created on-demand (first user interaction triggers it)
- **Procedural SFX**: No audio files—all sounds generated with oscillators and noise bursts
- **Sound effects**: Boss telegraph, explosion types, UI clicks, damage, stage clear, game over
- **Panning**: Sound effects pan based on event location (explosions near screen edges pan left/right)
- **Mute state**: Stored in localStorage; can be toggled via UI button

#### 7. **Particle & Visual Effects**
- **Explosions** (`spawnExplosion()`): Creates colored particles that fade out over time
- **Screen shake** (`screenShake()`): Temporary random canvas translation, duration-based
- **Flash effects**: White flash on hit, bomb flash on bomb use
- **Vignette**: Red pulsing effect during damage invulnerability
- **Combo popup**: Floating text showing multiplier increases
- **Stage banner**: Text banner announcing stage transitions

#### 8. **Power-Up System** (`maybeSpawnPowerUp()`, `applyPowerUp()`)
- **Types**: `life` (heal), `bomb` (refill bomb), `weapon` (random or level-up current)
- **Drop chance**: Randomized on enemy kill; some spawn rates tied to difficulty
- **Pickup behavior**: Automatic on collision; rotating sprite animation
- **Weapon logic**: Picking up the current weapon levels it up; picking up a different weapon switches to it

### Important Constants & Tuning

| Constant | Value | Impact |
|----------|-------|--------|
| `BASE_W`, `BASE_H` | 480, 720 | Canvas aspect ratio—all game coords are in these units |
| `KILLS_PER_STAGE` | 18 | Enemy count before boss appears |
| `DIFFICULTIES` | Easy/Normal/Hard | Spawn rates, speeds, enemy health (normal is balanced) |
| `THEMES` | 6 color sets | Visual themes cycle by stage mod 6 |
| `BOSS_PATTERNS` | ~18 attack patterns | Each boss draws 3; edit to change boss behavior |
| Weapon levels | 1–4 (Vulcan), 1–3 (others) | Upgrade caps; exceeding caps wraps or stays maxed |

### Rendering Canvas Coordinates
- Origin (0, 0) is top-left
- X increases right, Y increases down
- Player moves in a bounded area within the canvas
- Enemies and bullets wrap or despawn at screen edges

### Collision Detection
- Uses `rectsOverlap()` for axis-aligned bounding boxes
- Player collision box: centered at `player.x, player.y` with size `player.w × player.h`
- Hitbox is fairly forgiving (encourages engagement)

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` constant with `hp`, `color`, `dark` (darker shade for 3D effect), and optionally `speed`
2. Update `spawnEnemy()` to include the new type in the spawn logic
3. Add unique bullet patterns in `update()` enemy movement section if needed
4. Add explosion color mapping in `EXPLODE_SFX` if a custom death sound is desired

### Adding a New Boss
1. Add entry to `BOSSES` array with `{ name, hp, color, dark, w, h }`
2. Add a 3-pattern array to `BOSS_PATTERNS` at the matching index
3. Each pattern is a function `(b) => {...}` that fires bullets via `enemyBullets.push(...)`
4. Test by editing `KILLS_PER_STAGE` to 0 to skip enemies and spawn the boss immediately

### Tuning Difficulty
Edit the `DIFFICULTIES` object to adjust spawning, speeds, and enemy behavior. Key parameters:
- `spawnRate`: Frames between spawns (higher = slower)
- `bulletSpeed`, `enemySpeed`, `playerSpeed`: Self-explanatory multipliers
- `enemyHp`: Health scaling factor for enemy types

### Customizing Audio
- Weapon fire, hits, and boss attacks are tied to specific `sfx.*()` calls
- Edit `beep()` and `noiseBurst()` function parameters to change sound character
- Add new sound generation functions following the same pattern

### Testing a Specific Scenario
Use browser console to set game state directly:
```javascript
stage = 5;           // Jump to stage 5
lives = 10;          // Set lives
player.weapon = 'laser';
player.weaponLevel = 4;
```
Then manually trigger boss spawn or enemy spawn in `spawnEnemy()`.

## Accessibility & UX Features

- **Reduced motion mode** (`reducedMotion` flag): Disables screen shake and particle effects
- **Screen reader support**: ARIA live region (`#srLive`) announces score, life changes, stage transitions
- **High contrast**: Glow effects can be toggled; colors chosen to be distinct
- **Responsive canvas scaling**: Maintains 16:9 aspect ratio across devices; responsive font sizing with `clamp()`

## Browser Compatibility

- Requires `requestAnimationFrame` and `Web Audio API`
- Canvas 2D context with basic 2D transforms and text rendering
- `localStorage` for persistent high score and settings
- Works on modern browsers (Chrome, Firefox, Safari, Edge)
- Mobile support via touch events and gamepad API

## Performance Notes

- **Frame-rate independent physics**: All movement scaled by `dt` (delta time)
- **No frame-skipping**: If performance drops, gameplay slows proportionally
- **Particle limit**: Explosions create `count` particles; no particle pooling, so many explosions may cause slowdown
- `trackPerf(dt)` logs frame timing; check browser console for FPS
- Quality setting (`quality = 0`) controls shadow/glow rendering for lower-end devices

## Future Enhancement Ideas (Not Implemented)

- Leaderboard persistence across sessions
- Additional weapon types (spread shot, bouncing bullets)
- Enemy formations and scripted waves
- Unlockable difficulty modes or modifiers
- Boss health bar visible on screen
- Combo visual effects (damage multiplier feedback)
- Achievements/badges system
