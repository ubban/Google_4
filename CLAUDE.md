# Sky Fighter

A Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application.

## Project Overview

**Type:** Single-page HTML5 game  
**Main File:** `index.html` (1669 lines)  
**Audience:** Web browsers (desktop, mobile, tablet)

Sky Fighter is a fully-featured arcade shooter with:
- Progressive stage-based gameplay
- Multiple enemy types with distinct behaviors
- Boss encounters with unique attack patterns
- Weapon upgrade system (Vulcan/Laser)
- Smart bomb mechanic
- Persistence (localStorage for best score, settings)
- Accessibility features
- Adaptive performance management
- Synthesized audio engine (no external assets)

## Architecture

### Code Organization

The entire game runs in a single `<script>` block within `index.html`:

```html
<script>
(function () {
  'use strict';
  // All game logic here
})();
</script>
```

This closure-based structure encapsulates game state and prevents global namespace pollution.

### Key Components

#### 1. **Rendering (Canvas)**
- Base resolution: 480x720 (responsive scaling)
- Rendered via `canvas.getContext('2d')`
- Adaptive quality system adjusts particle counts and effects based on frame time

**Drawing Functions:**
- `drawPlane(x, y, w, h, color, darkColor, flame)` – player and enemy aircraft with optional flame
- `drawBoss(boss, x, y, sz)` – boss designs with hull, wings, core
- `drawEnemy(type, x, y)` – enemy sprites based on type (grunt/interceptor/tank)
- `drawParticle(p)` – explosion particles with velocity falloff
- `drawBullet(bullet)` – player and enemy bullets

#### 2. **Game Loop**
- Runs at 60 FPS via `requestAnimationFrame`
- Delta-time based physics (`dt`)
- Handles: spawn, update, collision, rendering

**Key Functions:**
- `update(dt)` – updates all game entities and state
- `render()` – draws current frame

#### 3. **Audio Engine**
- WebAudio API-based synthesizer (no external files)
- Dynamics compressor to prevent clipping
- Per-sound biquad filtering and pitch sweeps
- Spatial panning based on event x-position

**Audio Functions:**
- `beep(freq, dur, type, vol, opts)` – oscillator with optional sweep/filter/pan
- `noiseBurst(dur, vol, opts)` – noise generator with filter
- `sfx` object – high-level sound effects (shoot, explode, powerup, etc.)

#### 4. **Input Handling**
- **Keyboard:** Arrow keys / WASD for movement, Space to fire, X/Shift for bombs, P/Esc to pause
- **Touch:** Joystick (left zone) and Fire/Bomb buttons (right zone)
- **Mouse:** Move pointer to steer, click to fire, crosshair cursor

Input priority: Keyboard > Joystick > Mouse (mouse only active when neither keyboard nor joystick input)

#### 5. **Entity System**
Game entities are plain objects with properties:

```js
player = { x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }
bullets = [{ x, y, vx, vy, w, h, damage, life }]
enemies = [{ x, y, vx, vy, w, h, type, hp, shootTimer, color, dark, score }]
enemyBullets = [{ x, y, vx, vy, w, h }]
particles = [{ x, y, vx, vy, life, color }]
powerUps = [{ x, y, vx, vy, type }]
boss = { x, y, phase, hp, maxHp, patternTimer, patterns, design, pattern }
```

No class hierarchy – all objects are plain data with update logic in procedural functions.

### Game Mechanics

#### Difficulty Levels
```js
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
}
```

#### Weapon System
- **Vulcan:** rapid-fire spread shot (default)
- **Laser:** slower, single concentrated beam
- Levels 1–3; pickups increase weapon level and grant bonus score

#### Stages & Bosses
- 3 background themes rotate: Daylight, Sunset, Night (repeats across 6 total themes with variants)
- 6 boss designs cycle through stages: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each boss has unique attack patterns (radial burst, aimed fan, homing spread, spiral, etc.)
- Kill count per stage: 18 regular enemies trigger boss spawn
- Stage clears award multiplier boost

#### Enemy Types
```js
ENEMY_TYPES = {
  grunt:       { w: 34, h: 34, hp: 1, speedMul: 1,    score: 10 },
  interceptor: { w: 26, h: 26, hp: 1, speedMul: 1.7,  score: 15, weave: true },
  tank:        { w: 46, h: 46, hp: 4, speedMul: 0.55, score: 35 }
}
```

Tanks appear from stage 2 onward. Interceptors weave while moving.

#### Kill Chain Multiplier
- Kill chain (no gaps) increases multiplier up to 8x
- Resets on miss/player hit; score multiplied by current chain

#### Power-ups
- Weapon level (W)
- Extra bomb (B)
- Extra life (L)

### UI Components

#### HUD (Top)
- Score, Best Score, Kill Chain Multiplier
- Current weapon and level
- Heart lives and bomb count
- Updated live during gameplay

#### Overlays
- **Start Screen:** Title, difficulty buttons, controls hint, high score
- **Boss Health Bar:** Shows boss name and HP when boss is active
- **Combo Pop:** Announces multiplier milestones above player
- **Stage Banner:** "STAGE N" on stage advance

#### Accessibility
- Reduced motion toggle (respects OS `prefers-reduced-motion`)
  - Disables screen shake, hit flash, and health vignette pulse
- Screen reader announcements (`aria-live` region)
  - Announces life loss, boss defeats, stage clear, game over

#### Top Control Bar
- Mute/unmute (toggles audio)
- Motion reduction (toggles screen shake and animations)
- Pause (freezes game state)

### Performance & Quality

**Adaptive Quality System:**
- Tracks frame times (rolling 45-frame window)
- 3 tiers: full (0), medium (1), low (2)
- Reduces particle counts and disables shadow-blur glow in lower tiers
- Automatically steps up when frame rate recovers

**Optimizations:**
- Frame-time based updates (delta-time)
- Collision checks use bounding boxes
- Drawing only visible entities

### Data Persistence

Uses `localStorage` with namespaced keys:
- `skyfighter.bestScore` – high score (number)
- `skyfighter.difficulty` – selected difficulty (string)
- `skyfighter.muted` – audio mute state (0/1)
- `skyfighter.reducedMotion` – motion reduction toggle (0/1)

## Development Guidelines

### Adding Features

1. **New Enemy Type:** Add entry to `ENEMY_TYPES`, update `drawEnemy()`, consider stage requirement in `minStage`
2. **New Boss:** Extend `BOSSES` array with new design, define pattern functions (e.g., `bossShootPattern_<name>`), reference in boss spawn logic
3. **New Weapon:** Add to weapon logic in `update()`, create bullet rendering, define shot pattern
4. **New Background Theme:** Add to `THEMES` array with sky gradient colors, cloud color, sun color, and name

### Modifying Game Balance

- Difficulty multipliers are in `DIFFICULTIES`
- Enemy spawn rates: `spawnBase` (starting interval) and `spawnMin` (fastest)
- Boss health: scale via `bossHpMul`
- Player speed, cooldown, lives: adjust in `resetGame()`

### Testing

1. **Manual Testing:** Open `index.html` in a browser
2. **Controls:** Test keyboard, touch (if on mobile), mouse
3. **Difficulty:** Verify spawn rates and enemy behavior scale correctly
4. **Audio:** Ensure no console errors, overlapping sounds don't clip
5. **Accessibility:** Test with reduced motion enabled, check screen reader announcements
6. **Performance:** Monitor frame times on slower devices; adaptive quality should activate

### Code Style

- No linting or formatting enforced; the codebase uses:
  - ES6+ (arrow functions, const/let)
  - Single IIFE to avoid globals
  - Plain objects (no classes)
  - Procedural update and render loops
  - Simple variable naming (e.g., `dt`, `vx`, `vy`, `px`, `py`)

### Known Limitations

- Single file (1669 lines) – future refactoring could split into modules
- No build step – served as-is
- Audio context requires user gesture to start (browser security)
- Touch controls disabled on non-touch devices
- Frame-rate dependent physics (mitigated by delta-time)

## How to Run

Open `index.html` in a modern web browser. No build, server, or dependencies required.

### Browser Requirements
- ES6+ support
- Canvas 2D context
- Web Audio API (fallback silently if unavailable)
- LocalStorage (for persistence)
- Touch events (for mobile support)

## Commits

1. **4efdad1** – Initial commit
2. **c4dc8f6** – Add Sky Fighter: Raiden-style vertical shoot-em-up game
   - Full feature set: enemies, bosses, weapons, bombs, stages, UI, audio, accessibility, performance tuning

## Future Enhancements

- Module/bundle setup for code organization
- Level editor for custom stages
- Leaderboard integration
- Replay system
- More enemy/boss variety
- Story/dialogue elements
- Seasonal themes/cosmetics

---

**Author:** ubban  
**Last Updated:** 2026-10-08
