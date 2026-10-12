# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 web application. The game features:
- Multiple enemy types with distinct behaviors
- Boss squadrons with complex attack patterns
- Weapon upgrade system (Vulcan, Spread, Laser, Homing)
- Difficulty settings (Easy, Normal, Hard)
- Score multiplier / combo system
- Touch and keyboard controls
- Procedurally generated stages with visual themes

**Repository Structure:** Everything is contained in a single `index.html` file (~1669 lines). No build tools, dependencies, or external files are required.

## High-Level Architecture

### Game Loop & Core Flow

The game operates with a standard game loop:
```
requestAnimationFrame(frame) →
  update(deltaTime) →
    update player movement/firing
    update enemies & boss AI
    update projectiles and collisions
    spawn enemies/bosses based on stage progression
  draw() →
    render background (with theme-specific sky gradient)
    render all entities (player, enemies, projectiles, explosions)
    update HUD
```

**Key State Variables:**
- `running` — game loop active
- `gameOver` — terminal game end state
- `paused` — pause state (overlay shown, game logic paused)
- `stage` — current stage number (increments on boss defeat)
- `score`, `mult` — score and current multiplier
- `player` — player entity with position, health, weapon, invulnerability
- `enemies`, `enemyBullets`, `playerBullets` — lists of active entities
- `boss` — currently active boss (null if none)

### Entity Model

Entities are plain objects with properties like `x`, `y`, `w` (width), `h` (height), `vx/vy` (velocity), type-specific properties. No class system — all entities managed via arrays:
- `enemies` — ENEMY_TYPES (Grunt, Interceptor, Tank) with different firing patterns
- `boss` — single boss at a time; has telegraph delay before attack patterns fire
- `playerBullets`, `enemyBullets` — projectile tracking
- `powerups` — weapon upgrades, extra lives, smart bombs
- `particles` — explosions and visual effects

### Weapon System

Four weapons available; upgrading cycles through them:
1. **Vulcan** — standard rapid fire
2. **Spread** — 5-way spread shot
3. **Laser** — continuous beam (visually distinct rendering)
4. **Homing** — self-seeking missiles

Weapon state stored in `player.weapon` (string name) and `player.weaponLvl` (upgrade count). Firing is driven by `fireWeapon()` which calls the weapon's fire logic.

### Boss Patterns

Bosses have multiple attack patterns per design. Each pattern is a function that fires bullets in a specific arrangement:
- `fireRadial()` — circular spread in all directions
- `fireAimedSpread()` — aimed cone of bullets
- `fireColumns()` — vertical columns down the screen
- `fireSpiralBurst()` — outward spiral
- `fireCross()` — cross pattern (horizontal + vertical)
- `fireHomingSeeded()` — homing missiles to player position
- `fireDoubleRing()` — two concentric rings at different speeds

Bosses cycle through patterns; telegraph delay (red indicator) precedes each attack. Boss designs:
1. **CUTTER** — fast, light attacks
2. **CRUISER** — medium difficulty
3. **DREADNOUGHT** — heavy, complex patterns
4. **TITAN** — extreme difficulty

### Difficulty System

`DIFFICULTIES` object defines enemy spawn rates and boss health multiplier for Easy/Normal/Hard. Affects:
- How often enemies spawn (`spawnRate`, `spawnTick`)
- Boss health scaling (`bossHealthMult`)
- Controls difficulty by adjusting game speed (not visual).

### Collision & Scoring

- `rectsOverlap()` — AABB collision check used for all hit detection
- Score multiplier increments on enemy kills (`addScore()`) and resets on player hit (`breakChain()`)
- Bosses award 500 points flat; enemies scale by multiplier
- Best score persisted in localStorage

### Audio System

Web Audio API synthesis (no external files):
- `beep(freq, dur, type, vol, opts)` — pure sine/square/triangle tones
- `noiseBurst(dur, vol, opts)` — white noise with envelope
- `sfx` object collects named sound effects (e.g., `sfx.fire()`, `sfx.bossDown()`, `sfx.gameover()`)
- Mute state persisted in localStorage; respects prefers-reduced-motion

### Input Handling

Three input modes:
1. **Keyboard** — WASD/Arrows move, Space/X fire, Shift bomb, P/Esc pause
2. **Mouse** — move to cursor position, click to fire
3. **Touch** — joystick in lower-left (drag to move), fire/bomb buttons in lower-right

Joystick logic in `joystickStart/Move/End()`; mouse position tracked via `canvasPointFromEvent()`.

### Theming & Visuals

`THEMES` array holds per-stage visual styles (sky gradient colors, sun color, cloud color). Cycles through themes as stages progress. Each theme painted fresh each frame with gradient and procedural cloud rendering.

## Development Workflow

### Running the Game

**No build needed.** Open `index.html` directly in a browser:
```bash
open index.html  # macOS
xdg-open index.html  # Linux
start index.html  # Windows
```

Or serve locally:
```bash
# Python 3
python3 -m http.server 8000
# Then visit http://localhost:8000/index.html

# Node.js
npx http-server
```

### Testing Changes

1. **Modify code** in the `<script>` section of `index.html`
2. **Reload the browser** (Ctrl+Shift+R / Cmd+Shift+R to bypass cache)
3. **Test manually:**
   - All three input modes (keyboard, mouse, touch)
   - All difficulties
   - Weapon cycles (Vulcan → Spread → Laser → Homing)
   - Boss encounters (at least one full stage)
   - Pause, mute, reduced-motion toggles
   - Best score persistence (reload page, verify high score saved)

### Common Editing Tasks

**Add a new weapon:**
1. Define firing logic function (e.g., `fireQuad()`)
2. Add weapon name to the cycle in `fireWeapon()` switch statement
3. Update weapon label in `weaponLabel()`
4. Test weapon upgrade progression

**Adjust enemy difficulty:**
- Edit spawn rates / health in `DIFFICULTIES` object
- Modify ENEMY_TYPES firing behavior or bullet speeds
- Test by playing through stages on each difficulty

**Add a new boss pattern:**
1. Create a pattern function (e.g., `fireWave()`)
2. Add to the boss's pattern array in `BOSS_PATTERNS`
3. Adjust telegraph delay (`b.telegraph = XX`) to telegraph length
4. Test timing so attack neither too fast nor telegraphed too long

**Tweak game balance:**
- Enemy spawn rate: `DIFFICULTIES[diff].spawnRate` (lower = more enemies)
- Boss health: `DIFFICULTIES[diff].bossHealthMult` (higher = tankier)
- Score multiplier decay: `breakChain()` logic
- Player speed: `const playerSpeed = 200` in update

### Performance Notes

- Game loop uses `requestAnimationFrame` for smooth 60fps
- Canvas is 2D context; all rendering is immediate-mode (no retained scene graph)
- Particle effects (explosions) are simple circles; count capped implicitly by spawn rate
- No physics engine; collision is simple AABB
- Audio synthesis is real-time; beeps are synchronous (blocking briefly) — keep durations short
- Frame skip detection via `trackPerf(dt)` logs frame timing but doesn't throttle (always 60fps target)

### Key Files & Sections

All in `index.html`:
- **Lines 1–420:** HTML structure & CSS styles
- **Line 422:** `<script>` tag opens
- **Lines 468–640:** Utility functions (announce, perftrack, audio setup, SFX definitions)
- **Lines 680–730:** UI helper functions (renderHearts, flashHit, screenShake, etc.)
- **Lines 768–840:** Input handling (keys, joystick, mouse)
- **Lines 875–1148:** Game state functions (resetGame, fireWeapon, spawnEnemy/Boss, update)
- **Lines 1304–1367:** Weapon/boss firing patterns
- **Lines 1411–1454:** Boss logic (updateBoss, killBoss)
- **Lines 1469–1550:** End-game & pause logic
- **Lines 1550+:** Main render loop (draw, animation frame setup, event listeners)

## Development Tips

1. **Collision Testing:** Add a debug draw of bounding boxes by rendering `rectsOverlap()` calls with `strokeRect()`
2. **Boss Pattern Debugging:** Reduce boss health or remove telegraph delay temporarily to test patterns more quickly
3. **Audio Muting:** Game respects Web Audio Context state; mute via `audioCtx.destination.gain.value`
4. **Screen Shake Intensity:** Controlled by `shakeMag` parameter in `screenShake()`; increase for heavier screen feedback
5. **Invulnerability Frames:** `player.invuln` countdown in update loop; set high to test collision safety window

## Storage Keys

Game state persisted in localStorage:
- `skyfighter.bestScore` — high score (numeric string)
- `skyfighter.muted` — audio mute flag (boolean JSON)
- `skyfighter.difficulty` — last selected difficulty (string: easy/normal/hard)
- `skyfighter.reducedMotion` — accessibility flag (boolean JSON)

Clear all: `localStorage.clear()` in console.
