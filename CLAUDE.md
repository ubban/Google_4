# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a vertical shoot-'em-up (shmup) game built as a single-file HTML5 application. The game features:
- A player-controlled aircraft defending against waves of enemies
- Multiple weapon types (Vulcan, Laser) with upgradeable levels
- Boss fights at stage transitions
- Combo/multiplier system for scoring
- Three difficulty levels (Easy, Normal, Hard)
- Procedurally generated enemy formations
- Responsive design for desktop, tablet, and mobile with touch controls
- Synthesized audio (no external sound assets)

## Development

### Running the Game

The game is self-contained in `index.html`. To play:

**Desktop/Web:**
- Open `index.html` directly in a modern web browser
- Use mouse (move to steer, click to fire), Arrow Keys / WASD (move), Space (fire), X / Shift (bomb)

**Mobile:**
- Drag left side to move, tap right-side FIRE button, tap BOMB button
- Supports touch event handling and responsive sizing

### Building / Deployment

No build process required. The file is ready to deploy as-is to any static hosting.

### Testing Changes

For rapid iteration:
1. Edit `index.html` directly
2. Reload the browser (or use live-reload tools if available)
3. Test across different difficulty levels, screen sizes, and input methods

## Architecture Overview

The entire game logic (~1,700 lines) is in a single `<script>` block within `index.html`, organized as an IIFE (Immediately Invoked Function Expression) for encapsulation.

### Core Systems

#### 1. **Rendering Pipeline** (Canvas 2D)
- `canvas` and `ctx` (2D context) handle all visuals
- Resolution: 480×720 base (scaled to fit viewport, max 1.6× upscale)
- Frame-based rendering: player, enemies, bullets, particles, UI elements
- All shapes drawn procedurally (no sprite sheets)

**Key rendering functions:**
- `drawPlane(x, y, w, h, color, darkColor, flame)` — draws player and enemy aircraft
- `drawBossShip(b)` — renders the boss with animated core glow and telegraph effects
- Particles, explosions, and visual feedback drawn each frame

#### 2. **Audio System** (WebAudio API)
- Synthesized audio using oscillators, noise buffers, and filters
- Two main functions: `beep(freq, dur, type, vol, opts)` and `noiseBurst(dur, vol, opts)`
- `sfx` object maps 10+ sound effects (shoot, explode, bomb, powerup, etc.)
- Master gain and compressor normalize levels
- Spatial panning based on entity X position (horizontal stereo)

**Important:** Audio context must be resumed via user gesture (`ensureAudio()`).

#### 3. **Game State**
Central mutable state:
```javascript
player, bullets, enemies, enemyBullets, particles, powerUps, boss
score, lives, maxLives, running, paused, gameOver
bombCount, maxBombs, spawnTimer, frame, kills, stage, stageKills, bossWarned
chain, chainTimer, multiplier
```

State is reset each game via `resetGame()`.

#### 4. **Entity Systems**
Each entity type is a plain object with position, dimensions, and behavior properties:

- **Player:** `{x, y, w, h, speed, cooldown, rapid, weapon, level, invuln}`
  - Moves via keyboard/mouse/joystick input
  - Fires bullets when `firing` is true (cooldown-gated)
  - Weapon upgrades persist per stage; reset to level 1 between stages

- **Enemies:** Spawned based on stage and difficulty; stored in `enemies` array
  - Types: `grunt`, `interceptor` (fast), `tank` (tanky)
  - Each has `hp`, `speed`, `shootChance`, enemies descend and fire at the player
  - Defeated enemies spawn particles and power-ups

- **Bullets:** Player bullets tracked in `bullets`, enemy bullets in `enemyBullets`
  - Collision checked each frame against their respective targets

- **Particles:** `particles` array for explosions, trails, visual effects
  - Simple physics: velocity + gravity, fade-out over time

- **Power-ups:** Spawn on enemy death; collectable for weapon/bomb upgrades

- **Boss:** Single boss entity with multiple attack patterns and phases

#### 5. **Game Loop & Update Logic**
The main update is driven by `requestAnimationFrame` in the unnamed game loop (~line 1450–1650):

1. Calculate delta time (`dt`)
2. Update player (movement, shooting, invulnerability)
3. Spawn enemies (based on `spawnTimer` and difficulty)
4. Update and collide bullets, enemies, particles
5. Check stage transitions and boss spawns
6. Update UI (HUD, combo pop, stage banner, boss bar)
7. Render everything
8. Handle screen shake, flash effects, audio cues

**Difficulty config:** `DIFFICULTIES` object (line ~486) adjusts spawn rates, enemy speed, lives, bomb count, and boss HP multipliers.

#### 6. **Input Handling**
Three input methods with priority logic (keyboard/joystick > mouse):

- **Keyboard:** Arrow keys, WASD for movement; Space to fire; X/Shift to bomb
- **Mouse:** Move to aim, click to fire (only active when cursor hovers canvas)
- **Touch:** Left zone for joystick movement, right zone for FIRE/BOMB buttons (mobile only)

`mouseActive` flag prevents mouse and keyboard from fighting when both are used.

#### 7. **UI & Menus**
Overlay system manages title screen, game-over screen, and pause menu:
- `overlay` div shown/hidden to display state
- HUD displays score, best, multiplier, weapon, lives, bombs
- Boss bar shows when boss HP > 0
- Difficulty buttons update `difficulty` (persisted in localStorage)

#### 8. **Persistent State**
Four values saved to `localStorage`:
- `skyfighter.bestScore` — best score ever
- `skyfighter.muted` — audio mute toggle
- `skyfighter.difficulty` — selected difficulty
- `skyfighter.reducedMotion` — accessibility flag (reduced animations/shake)

## Key Constants & Configuration

### Configuration Objects (lines ~485–515)

**`DIFFICULTIES`** — controls game difficulty:
- `spawnBase` / `spawnMin`: spawn interval range (frames)
- `enemySpeed`: multiplier on enemy base speed
- `enemyShootChance`: probability enemy fires each frame
- `lives`, `bombs`: starting resources
- `bossHpMul`: health multiplier for boss fights

**`ENEMY_TYPES`** — properties for each enemy class
- `GRUNT`, `INTERCEPTOR`, `TANK`: HP, speed, score, color, etc.

**`BOSSES`** — list of 6 boss variants with:
- `name`: displayed during fight
- `hullTop` / `hullBottom` / `wing` / `core`: colors for rendering
- `hpMul`: health scaling (relative to player level)

**`THEMES`** — 6 sky/environment color schemes (sky gradient, cloud color, sun color)

**Game constants:**
- `BASE_W = 480`, `BASE_H = 720`: base canvas resolution
- `KILLS_PER_STAGE = 18`: enemies to defeat before boss spawns

## Common Edits

### Adding a new enemy type:
1. Add entry to `ENEMY_TYPES` object
2. Update spawn logic in game loop (search for `spawnType`)

### Adjusting difficulty:
Edit `DIFFICULTIES` constants or modify spawn/enemy speed parameters in the game loop.

### Adding a new weapon:
1. Define weapon behavior in `player.weapon` check during update
2. Add SFX to `sfx` object for firing sound
3. Update weapon label in `weaponLabel()`

### Tweaking visual feedback:
- Screen shake: modify `screenShake(mag, dur)` calls
- Flash effects: adjust CSS animations in `#flash` styles
- Colors: edit `THEMES`, `ENEMY_TYPES`, or `BOSSES` color properties

### Accessibility:
- Reduced motion flag toggles animations; check `reducedMotion` before applying effects
- Audio mute flag; check `muted` in `sfx` and `beep()` calls
- Screen reader announcements use `announce(text)` function

## Notes for Future Work

- **Single-file constraint:** Keep the game in one HTML file for simplicity and deployment ease.
- **No external assets:** All graphics are Canvas 2D primitives; all audio is synthesized.
- **Mobile responsiveness:** The game scales to fit viewport (max 1.6×) and adjusts UI elements with viewport-relative sizing (clamp, vw, vh units).
- **Performance:** Adaptive quality system (`quality` variable) throttles rendering on low-end devices; monitor `frameTimes` to detect frame drops.
- **Touch controls:** Only enabled on touch devices; always test both touch and keyboard/mouse paths.
