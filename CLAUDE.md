# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-page HTML5 application. The game is self-contained in `index.html` with inline CSS and JavaScript—no external dependencies, no build process. It includes:

- Canvas-based 2D rendering with dynamic backgrounds
- WebAudio API for procedural sound generation
- Progressive difficulty scaling across stages
- Boss encounters with special patterns
- Multiple weapon systems and power-ups
- Full touch and mouse/keyboard support
- Accessibility features (reduced-motion preferences, screen-reader support)

## Game Architecture

### Core Game Loop & State

The entire game runs in a single IIFE that manages these top-level objects:

- **`player`**: `{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }`
  - Position and size define hitbox
  - `weapon` is either `'vulcan'` or `'laser'`; `level` affects fire rate and damage
  - `invuln` is a timer preventing damage spam after hits
  - Speed is in pixels/sec

- **`enemies`**: Array of enemy objects spawned via `spawnEnemy()`
  - Enemy types: `grunt` (basic), `interceptor` (fast), `tank` (tanky)
  - Each has `{ type, x, y, w, h, hp, angle, shootTimer, vx, vy }`
  - `type` determines visual rendering and collision/explosion SFX

- **`boss`**: Single boss object (or `null` when no boss active) via `spawnBoss()`
  - Complex multi-part rendering via `drawBossShip(b)`
  - Pattern-based AI firing controlled in `updateBoss(dt)`
  - Six distinct boss designs in `BOSSES` array with color schemes

- **`bullets`, `enemyBullets`**: Arrays of projectiles
  - Bullets: `{ x, y, vx, vy, w, h, bounceX, bounceY, homeTarget }`
  - Enemy bullets: simpler `{ x, y, vx, vy }`

- **`particles`**: Explosion/visual effects via `spawnExplosion(x, y, color, count)`
  - Dust, debris, hit flashes animated per-frame

- **`powerUps`**: Collectibles that spawn probabilistically on enemy death
  - Types: `health`, `bomb`, `weapon`
  - Applied via `applyPowerUp(type)`

### Difficulty & Scaling

`DIFFICULTIES` object keys (`easy`, `normal`, `hard`) control:
- Enemy spawn rates and min/max timings
- Enemy speed multiplier and shoot frequency
- Player starting lives and bombs
- Boss HP scaling factor

Current difficulty stored in `localStorage[STORAGE_DIFF]` and read on game start.

### Stage System

- Game progresses through stages automatically
- Each stage requires `KILLS_PER_STAGE` (18) kills to advance
- Boss appears after stage is complete
- `stage` global counter, incremented in `killBoss()`

### Weapon System

Two weapons (selected via power-up):
- **Vulcan**: Direct fire, higher rate, constant direction
  - Fire patterns: rapid spread, homing with target tracking
- **Laser**: Single beam, must line up shot, higher damage per hit
  - Fire patterns: column, spiral, cross formations

Weapon level (1–6) affects fire rate and pattern complexity. Levels incremented via weapon power-ups; level caps pattern availability.

## Input Handling

### Mouse/Keyboard
- **Movement**: WASD, arrow keys, or mouse position (when `mouseActive`)
- **Fire**: Mouse click, Space, or continuous while pressed
- **Bomb**: X, Shift, or mobile bomb button
- **Pause/Resume**: P, Esc, or pause button

### Touch
- **Movement**: Drag in left 55% of screen controls joystick (`joyX`, `joyY`)
- **Fire/Bomb**: Buttons in right 45% at screen bottom
- Joystick "base" is hidden by default; shown during drag
- Touch ID tracking prevents multi-touch conflicts

## Audio System

All sounds generated procedurally via WebAudio (no asset files):

- **`beep(freq, dur, type, vol, opts)`**: Single oscillator tone with optional frequency sweep, filter, and panning
- **`noiseBurst(dur, vol, opts)`**: White noise burst, used for explosions
- **`sfx` object**: Named effect templates (e.g., `sfx.explodeGrunt(x)`, `sfx.hit()`)
  - Panning based on screen position for spatial audio
  - Delay option allows staggered multi-voice sounds

Mute state persisted in `localStorage[STORAGE_MUTE]`.

## Rendering

### Canvas & Scaling
- Base resolution: 480×720 (game area)
- Scales responsively to window, capped at 1.6× to prevent excessive upscaling
- DPI-aware and touch-friendly

### Visual Effects
- **`drawPlane(x, y, w, h, color, darkColor, flame)`**: Player/enemy ship with optional flame
- **`drawBossShip(b)`**: Multi-part boss with wings, hull, core; animates via `frame * 0.15` oscillation
- **`drawClouds(speed, fillColor, rx, ry)`**: Parallax scrolling background
- Screen-shake: `screenShake(mag, dur)` applies canvas transform; disabled in reduced-motion mode
- Flash effects: `flashHit()`, `flashBomb()` trigger CSS animations
- HUD overlays: Score, health (hearts), bombs, weapon, multiplier, boss health bar

### Dynamic Theming
`THEMES` array: Six sky gradient schemes, each with cloud and sun colors. Theme cycles per stage.

## Game Physics & Collision

- **Movement**: Clamped to canvas bounds (player), no wrapping
- **Collision**: `rectsOverlap(a, b)` axis-aligned box check for hitboxes
  - Player vs. enemy bullets → damage/death
  - Player bullets vs. enemies → kill/damage
  - Player vs. power-ups → apply bonus
  - Explosions apply damage to nearby enemies/boss via proximity

## Scoring & Chains

- Base enemy score determined by enemy type
- Kill chains: Consecutive hits within `chainTimer` (auto-decays) increase `multiplier` (caps at 6×)
- `breakChain()` resets multiplier when chain breaks
- Best score persisted in `localStorage[STORAGE_BEST]`

## Accessibility

- **Reduced Motion**: Disables animations, screen-shake, flashes when OS/user preference set
  - Toggle via button (top-right); persisted in `localStorage[STORAGE_REDUCED]`
- **Screen Reader**: `srLive` element announces game state changes via `announce(text)`
- **Touch/Mouse Flexibility**: Both input methods fully supported; auto-detected

## Common Development Tasks

### Run the Game
Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge).

### Test Difficulty Levels
Use the difficulty buttons on the start screen to toggle between Easy, Normal, Hard. State persists via localStorage.

### Add a New Boss
1. Add entry to `BOSSES` array with `{ name, hullTop, hullBottom, wing, core, hpMul }`
2. `drawBossShip()` auto-renders from design colors
3. Boss patterns defined in `updateBoss(dt)` switch statement via `b.pattern` (0–5 for current set)

### Add an Enemy Type
1. Add to `ENEMY_TYPES` with `{ w, h, hp, speedMul, color, dark, score, minStage, weave? }`
2. Rendering: `drawPlane()` handles color/flame
3. Update `spawnEnemy()` pool to include type if stage allows (`minStage` check)
4. Add explosion SFX key to `EXPLODE_SFX`

### Tweak Game Balance
- Enemy spawn rates: Modify `DIFFICULTIES`
- Weapon fire patterns: See weapon-fire functions (`fireRadial`, `fireAimedSpread`, etc.)
- Enemy behavior: Adjust angle/shoot timers in `update(dt)` enemy loop
- Boss patterns: Modify fire sequences in `updateBoss(dt)`
- Multiplier caps/timers: `chain`, `chainTimer`, `multiplier` globals

### Debug Game State
- Use browser DevTools console:
  - Inspect `player`, `enemies`, `bullets`, `score`, `stage` etc.
  - Call `resetGame()` to restart, `endGame()` to force game-over
  - Set `running = false` to pause physics (but not rendering)

## Notable Code Patterns

- **Monolithic Architecture**: All state and logic in single IIFE; no modules or classes (intentional for single-file deployment)
- **Adaptive Quality**: `trackPerf(dt)` monitors frame times and adjusts particle/effect density via `quality` variable (0=full, 2=low)
- **Lazy Audio Context**: `ensureAudio()` defers WebAudio init until first user interaction (required by browsers)
- **Canvas Transform**: Resets before each render; screen-shake applied via temporary translate
- **Event Delegation**: Mouse/keyboard/touch handlers attached at page load; state flags (`firing`, `keys`) drive per-frame polling

## Browser Requirements

- **ES6 Support**: Arrow functions, template literals, const/let
- **Canvas 2D**: Standard CanvasRenderingContext2D API
- **WebAudio**: AudioContext or webkit variant
- **LocalStorage**: For best score, settings persistence
- **Touch Events** (optional): For mobile support

No polyfills included; doesn't run on IE11 or older.

## File Size & Performance

- Single HTML: ~59 KB uncompressed
- No external assets; all graphics rendered procedurally
- Target: 60 FPS on modern mobile and desktop
- Adaptive quality system prevents frame drops under load

