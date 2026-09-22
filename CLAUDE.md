# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a vertical shoot-'em-up game (Raiden-style) built entirely with:
- Vanilla JavaScript (ES6+, no frameworks)
- HTML5 Canvas 2D API for rendering
- WebAudio API for synthesized sound effects
- Local Storage for persistent data (high scores, settings)

The entire game is contained in a single `index.html` file (~1670 lines) with embedded CSS and JavaScript.

## How to Run / Develop

Since this is a single-file HTML game with no build step:

1. **Open locally**: Simply open `index.html` in any modern browser:
   - Double-click the file, or
   - Drag it into a browser window, or
   - Use a local server: `python3 -m http.server 8000` then visit `http://localhost:8000/index.html`

2. **Deploy**: The game works as a static file—deploy the `index.html` file directly to any web host.

3. **No build required**: All code is self-contained. Modifications to the game can be tested immediately by refreshing the browser.

## Code Architecture

The game is structured as a single large IIFE (Immediately Invoked Function Expression) that encapsulates all game state and logic. Key sections:

### Audio System (Lines 542–676)
- **`ensureAudio()`**: Initializes AudioContext lazily (triggered on first user interaction per browser security)
- **`beep()`** and **`noiseBurst()`**: Synthesize individual sound effects using oscillators and noise buffers
- **`sfx` object**: Predefined sound functions for gameplay events (shoot, explosion, powerup, boss telegraph, etc.)
- No external audio assets—all sounds are procedurally generated in real-time

### Rendering (Lines 913–1638)
- **`drawPlane()`**: Renders player and enemy ships as geometric shapes (triangles, ellipses)
- **`drawBossShip()`**: Boss ship with animated parts and core glow
- **`drawClouds()`**: Parallax scrolling cloud background layers
- **`draw()`**: Main render function; clears canvas, draws background, all entities, particles, and UI overlays
- No spritesheets—all visuals are canvas path/shape drawing

### Game Loop (Lines 1640–1652)
- **`loop(now)`**: RequestAnimationFrame-based game loop
- Tracks delta time (`dt`), caps frame updates at `0.05s` to prevent large jumps
- Calls `update(dt)` for game logic, then `draw()` for rendering
- Performance tracking adjusts visual quality (`quality` variable) based on frame time

### Game State & Logic (Lines 863–1505)
- **Player object**: Position, size, weapon type/level, cooldown, invulnerability timer
- **Bullets, enemies, enemy bullets, power-ups, particles**: Arrays of active entities
- **`update(dt)`** (lines 1148–1302): Core game update loop
  - Handles player movement (keyboard, mouse steering, joystick/touch)
  - Updates all entity positions
  - Spawns enemies on timers; triggers boss spawns after kill thresholds
  - **Collision detection**: `rectsOverlap()` checks AABB collisions for bullets → enemies/boss, enemies → player, etc.
  - Applies power-ups, manages lives and bombs
  - Updates scoring chain/multiplier
- **`fireWeapon()`**: Creates bullet objects based on weapon type (Vulcan vs. Laser)
- **`useBomb()`**: Clears all enemy bullets, damages all enemies and boss, adds score

### Boss System (Lines 1043–1439)
- **6 boss designs**: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- **Unique attack patterns**: Each boss has 3 bullet-fire patterns in `BOSS_PATTERNS`
  - Radial bursts, aimed spreads, columns, spirals, homing bullets, cross-fire, double rings
  - `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc. (lines 1324–1370)
- **Telegraph mechanic**: Boss shows telegraph warning (22-frame delay) before firing
- **Boss HP scaling**: Base HP scales with stage and difficulty; each boss design has a `hpMul` multiplier

### Difficulty System (Lines 485–489)
Three preset difficulty levels (easy/normal/hard) control:
- Enemy spawn rates (`spawnBase`, `spawnMin`)
- Enemy movement speed and shoot frequency
- Starting lives and bombs
- Boss health multiplier

### Input Handling (Lines 767–861)
- **Keyboard**: Arrow keys or WASD for movement, Space to fire, X/Shift/P for bomb/pause
- **Mouse**: Move cursor to steer (smooth interpolation toward mouse position)
- **Touch**: Left side of screen is movement joystick, right side has Fire and Bomb buttons
- Input priority: keyboard/joystick > mouse steering

### UI & Settings (Lines 463–750)
- **HUD**: Score, best score, multiplier, weapon name/level, hearts (lives), bombs
- **Pause overlay**: Shows pause screen with resume button
- **Game over overlay**: Displays final stats and difficulty selector
- **Settings toggles** (stored in localStorage):
  - Mute/unmute audio
  - Reduced motion (disables animations/screen shake)
  - Difficulty selection

## Key Systems to Understand

### Enemy Spawning
- `spawnTimer` decreases each frame; when ≤ 0, a random enemy spawns
- Three enemy types: grunt (weak), interceptor (fast), tank (tanky, high-value)
  - Tank only appears after stage 2
- Enemies weave and shoot; shoot patterns are "spread" (fan) or "aimed" (toward player)
- After 18 enemy kills per stage, a boss spawns after a brief delay

### Scoring & Multiplier Chain
- Killing enemies adds base score × multiplier
- **Chain timer**: Resetting on each kill; stays alive for 2.2 seconds
- **Multiplier**: 1 + (kills / 5), capped at 10×
- Chain breaks if player takes damage or time expires

### Collision System
- All collisions use AABB (axis-aligned bounding box) via `rectsOverlap()`
- Damage and hits are applied immediately on collision detection
- Player has invulnerability frames after being hit (1.5 seconds)

### Quality Scaling
- `quality` variable (0 = full, 1 = medium, 2 = low)
- Tracks average frame time over 45 frames
- If avg > 25ms, quality degrades; if < 17.86ms, quality improves
- Particle counts and shadow/glow effects scale with quality

## Theme & Stage System

Six themes (sky gradients, cloud colors, sun colors) cycle with stages:
- Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge

Themes are cosmetic and repeat. Each stage increments difficulty by spawning enemies more frequently.

## Performance Considerations

1. **Canvas quality scaling**: Dynamically reduces particle effects and visual quality on low-frame-rate devices
2. **Lazy audio init**: AudioContext only created on first user interaction (browser requirement)
3. **Entity array filtering**: Dead bullets, off-screen enemies, and finished particles are filtered out each frame
4. **No asset loading**: Everything is procedural; no network latency
5. **Reduced motion**: Users can disable animations/screen shake for accessibility

## Common Development Tasks

### Add a new enemy type
1. Add an entry to `ENEMY_TYPES` object with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update the `spawnEnemy()` function's logic to include the new type
3. If it has a unique death animation, add it to `EXPLODE_SFX`

### Add a new boss
1. Add boss design to `BOSSES` array: `{ name, hullTop, hullBottom, wing, core, hpMul }`
2. Define attack patterns as a 3-element array in `BOSS_PATTERNS`
3. Boss index in `BOSS_PATTERNS` corresponds to index in `BOSSES` (e.g., 6th boss uses `BOSS_PATTERNS[5]`)

### Add a new weapon type
1. In `fireWeapon()`, add a condition for `player.weapon === 'yourWeapon'`
2. Create bullet objects with appropriate speed, spread, and visual properties
3. Update weapon-switching logic in `applyPowerUp()` if desired

### Adjust difficulty values
- Modify constants in `DIFFICULTIES` object (spawn rates, enemy speeds, health pools)
- Difficulty multipliers are applied in `spawnEnemy()`, `spawnBoss()`, and `update()` logic

### Modify visual appearance
- **Player/enemy colors & shapes**: Edit `drawPlane()` and sprite-drawing logic
- **Boss design**: Adjust `drawBossShip()` rendering or colors in `BOSSES`
- **Particle/effect colors**: Modify color strings in `spawnExplosion()` or SFX functions
- **Background**: Adjust `THEMES` array gradients and cloud parameters

### Change game balance
- **Scoring**: Modify base enemy scores in `ENEMY_TYPES` or the `addScore()` logic
- **Player speed**: Change `player.speed` in `resetGame()`
- **Weapon fire rate**: Adjust `fireDelay` in the weapon logic (lines 1177–1180)
- **Bomb damage**: Modify the damage value in `useBomb()` (currently 24 HP to boss)
- **Player health**: Change `maxLives` based on difficulty

## Browser Compatibility

- **Requires**: Modern browsers with ES6 support, Canvas 2D API, WebAudio API
- **Tested on**: Chrome, Firefox, Safari, Edge
- **Mobile**: Touch controls adapt to touch-capable devices
- **Accessibility**: Respects `prefers-reduced-motion` media query

## Storage Keys

The game uses localStorage for persistence:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state (0/1)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced motion preference (0/1)

## Notes for Future Work

- **Audio**: All sounds are real-time synthesis. To add new SFX, use `beep()` (for tonal sounds) or `noiseBurst()` (for percussive sounds)
- **Responsiveness**: Canvas scales to fit viewport while maintaining aspect ratio (BASE_W=480, BASE_H=720)
- **Frame-based logic**: Many timers use frame counts; be aware that performance scaling can affect timing slightly
- **Boss randomization**: Boss designs cycle predictably by stage; randomizing via `Math.random()` if desired
- **Particle effects**: Scaled by quality; heavily used on screen shake/explosion effects
