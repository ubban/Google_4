# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game—rendering, game logic, audio synthesis, input handling, and UI—is contained in `index.html` (1670 lines).

This is a **static HTML file** with no build system, dependencies, or server required. It runs directly in any modern browser.

## Running the Game

Open `index.html` directly in a web browser:
```bash
# On macOS/Linux:
open index.html

# On Windows:
start index.html

# Or use any local server:
python3 -m http.server 8000
# Then visit http://localhost:8000/index.html
```

The game is fully responsive and works on desktop (keyboard/mouse), tablet (touch), and mobile devices.

## High-Level Architecture

The codebase is organized as a single large IIFE (Immediately Invoked Function Expression) that manages:

1. **Rendering** — Canvas-based 2D graphics with particle effects
2. **Game State** — Player, enemies, bullets, power-ups, boss tracking
3. **Game Loop** — `requestAnimationFrame`-based update/draw cycle
4. **Input Handling** — Keyboard, mouse, touch/joystick on mobile
5. **Audio Synthesis** — Web Audio API with procedural sound effects
6. **Difficulty System** — Three selectable difficulties affecting spawn rates and enemy behavior
7. **UI Overlays** — Start screen, pause menu, game-over screen with score tracking

## Key Systems

### Game State Management (Lines 863–910)
Player, enemies, bullets, power-ups, boss, and global counters live here. `resetGame()` initializes a fresh run; `running`, `paused`, and `gameOver` flags control state transitions.

### Update Loop (Lines 1148–1302)
`update(dt)` runs each frame and handles:
- Player movement (keyboard, mouse, joystick)
- Firing weapons
- Enemy spawning and movement
- Collision detection (player vs enemies/bullets, bullets vs enemies, boss)
- Power-up collection
- Chain/multiplier tracking for scoring

### Rendering (Lines 1520–1638)
`draw()` renders the background (theme-based sky gradient with clouds), all game objects (player, enemies, bullets, power-ups, particles), and debug visuals (mouse target reticle).

Key drawing functions:
- `drawPlane()` — Player and enemy ships
- `drawBossShip()` — Boss with animated core
- `drawClouds()` — Parallax cloud background

### Input Handling (Lines 768–861)
- **Keyboard** — Arrow keys + WASD for movement, Space/mouse button to fire, X/Shift/P for bomb/pause
- **Mouse** — Pointer movement steers the ship; click to fire
- **Touch** — Virtual joystick in movement zone (left side), fire button (right side), bomb button
- Mouse and keyboard/joystick input don't conflict — keyboard takes priority when active

### Audio System (Lines 542–676)
- Uses **Web Audio API** with synthesized sounds (no external files)
- `beep()` generates tones with frequency sweep, envelope, and optional filter/panning
- `noiseBurst()` creates percussive/explosion sounds
- `sfx` object maps game events to their audio cues (shoot, explode, powerup, bossTelegraph, etc.)
- Master gain and compressor for clean output
- Mute state persisted in localStorage

### Difficulty System (Lines 485–489)
Three presets (Easy, Normal, Hard) affecting:
- Enemy spawn rates and minimum spawn timer
- Enemy movement speed and firing frequency
- Player lives and bomb count
- Boss HP multiplier

Selected difficulty is persisted in localStorage and applies immediately.

### Boss System (Lines 1043–1054, 1372–1434)
Boss spawns after 18 enemy kills per stage. Each boss (VANGUARD through TITAN, cycling) has:
- **Design** — Visual appearance (colors, shape)
- **Attack Patterns** — Procedural bullet formations (radial bursts, aimed spreads, spirals, homing, etc.)
- **Telegraph Phase** — Warning period before attack (visual + audio cue)
- **Attack Sequence** — 3-pattern rotation with 85-frame intervals between attacks
- Boss is vulnerable during telegraph and after pattern execution

Boss patterns are defined in `BOSS_PATTERNS[designIndex]` (lines 1372–1409). Adding a new boss:
1. Add entry to `BOSSES` array with colors and hpMul
2. Add 3 pattern functions to `BOSS_PATTERNS`
3. Update the cycling logic if desired (currently: `(stage - 1) % BOSSES.length`)

### Performance Adaptation (Lines 473–483)
Quality level (0=full, 1=medium, 2=low) auto-scales based on average frame time over 45 frames. Affects:
- Particle count (spawnExplosion uses quality multiplier)
- Shadow/glow effects in rendering
- Reduced motion preference is also respected

### Stage Progression
Stages increment after each boss defeat. Each stage:
- Cycles to next theme (visual environment)
- Difficulty ramping: spawn timer tightens every 5 seconds of play
- Boss stats increase (HP scales by stage × 35)

### Theme System (Lines 493–500)
Six visual themes that cycle per stage, each with sky gradient, cloud, and sun colors. Easy to add new themes or swap for custom art.

## Data Structures

**Player**: `{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }`
**Enemy**: `{ x, y, w, h, speed, hp, type, color, dark, canShoot, pattern, weave, ... }`
**Bullet**: `{ x, y, w, h, speed, vx, pierce, laser }`
**Enemy Bullet**: `{ x, y, w, h, speed, vx, radial, dx, dy }`
**Power-up**: `{ x, y, w, h, speed, type: 'life'|'bomb'|'weapon' }`
**Particle**: `{ x, y, vx, vy, life, maxLife, color }`
**Boss**: `{ x, y, w, h, hp, maxHp, entering, design, telegraph, attackTimer, patternIndex, vx }`

## Common Modifications

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with dimensions, HP, speed, color, and minimum stage
2. Update `spawnEnemy()` to include it in the spawn pool and its spawn chance
3. Define explosion color/type in `EXPLODE_SFX` if needed (line 678)
4. Add audio in `sfx` if different sound desired

### Tweaking Difficulty
Edit `DIFFICULTIES` object (lines 485–489). Key levers:
- `spawnBase` / `spawnMin` — Control how fast enemies spawn
- `enemySpeed` — Raw movement speed multiplier
- `enemyShootChance` — Probability enemy will shoot at player
- `lives` / `bombs` — Starting resources
- `bossHpMul` — Boss health scaling

### Changing Weapons
- **Vulcan**: spread pattern that widens at level 2–3 (lines 1126–1133)
- **Laser**: pierce through enemies, gains additional lanes at higher levels (lines 1136–1144)

Add new weapon by:
1. Extending the weapon name check in `fireWeapon()`
2. Defining its bullet properties and firing pattern
3. Update `weaponLabel()` to display its name

### Adding Visual Effects
Particles are spawned in `spawnExplosion()` (line 1073). Adjust:
- `count` — Number of particles (respects quality)
- `color` — Particle tint
- Initial velocity range and lifetime in the particle object

Screen shake is triggered by `screenShake(magnitude, duration)` (line 727) and scaled by reduced-motion preference.

## UI Persistence

All user preferences are stored in `localStorage`:
- `skyfighter.bestScore` — High score
- `skyfighter.muted` — Audio mute state
- `skyfighter.difficulty` — Selected difficulty
- `skyfighter.reducedMotion` — Reduced motion preference

These are loaded on startup and applied automatically.

## Canvas and Responsive Sizing

Base canvas size is **480×720** (portrait). Responsive scaling maintains aspect ratio up to 1.6× native size (lines 752–765). Touch/mobile gets appropriate canvas scaling and fullscreen vignette on low health.

## Accessibility

- **Screen reader announcements** — Key game events announced via `announce(text)` (line 468)
- **Reduced motion** — Respects OS preference and allows toggle; disables camera shake and flash effects
- **Keyboard controls** — Full keyboard support (arrows, Space, X, P, Escape)
- **Touch controls** — Virtual joystick + on-screen buttons with haptic feedback

## Notes for Extension

- **No external assets** — Entire game is self-contained HTML. All audio is synthesized.
- **Deterministic frame rate** — Game uses `requestAnimationFrame`, not fixed timestep. Rendering is tied to display refresh.
- **Collision detection** — Simple AABB (axis-aligned bounding box) overlap via `rectsOverlap()`.
- **Canvas context** — Uses 2D context; no WebGL or advanced rendering features.
