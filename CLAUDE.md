# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single, self-contained HTML file with no external dependencies. The entire application (HTML, CSS, JavaScript, and procedurally-generated audio) is embedded in `index.html`.

## Running the Game

- **No build process required.** Simply open `index.html` in any modern web browser.
- The game runs entirely in the browser with no server setup needed.
- Works on desktop (mouse/keyboard) and mobile (touch with on-screen joystick and buttons).
- Supports responsive scaling from 480×720 base resolution up to a maximum 1.6× scale.

## Architecture & Key Systems

### Single-File Structure
All code resides in `index.html` (~1,670 lines):
- **Lines 1–359**: HTML + embedded CSS (viewport, canvas, UI overlays, touch controls)
- **Lines 360–420**: HTML structure (canvas, HUD elements, overlay cards, touch controls)
- **Lines 422–1,667**: JavaScript game engine (IIFE)

### Core Game Loop
The game uses `requestAnimationFrame` for consistent 60 FPS target:
- **`loop(now)`** (line 1640): Main animation frame callback
- **`update(dt)`** (line 1148): Physics, input, collision, entity updates (runs only if `running && !paused`)
- **`draw()`** (line 1520): Canvas rendering

### State Management

**Game state variables** (lines 864–869):
- `player`: Ship object with position, weapons, cooldown, invulnerability
- `bullets`: Array of player projectiles
- `enemies`: Array of active enemy ships
- `enemyBullets`: Array of enemy projectiles
- `particles`: Array of explosion particles
- `powerUps`: Array of drops (life, bomb, weapon)
- `boss`: Current boss object or null
- `score`, `lives`, `stage`, `multiplier`: Game metrics stored in canvas

**Game flow states**:
- `running`: Active gameplay (false during menus, pause, game over)
- `paused`: Game is paused (update/draw still called, but input ignored)
- `gameOver`: Game ended (shows end screen)

### Entity Systems

#### Player (`player` object)
- **Properties**: `x, y` (center), `w, h` (collision), `speed`, `weapon` (vulcan|laser), `level` (1–3), `invuln`, `cooldown`, `rapid` (unused)
- **Weapons**:
  - **Vulcan** (default): Straight bullets with spread at level 2+
  - **Laser**: Penetrating shots with angled lanes at level 2+
- **Movement**: Keyboard/arrows + joystick (takes priority) or mouse tracking (fallback)
- **Invulnerability**: Set to 1.5s after hit, player blinks while invuln > 0

#### Enemies (`enemies` array)
Three types defined in `ENEMY_TYPES` (line 502):
- **Grunt** (stage 1+): Red, 1 HP, normal speed, score 10
- **Interceptor** (stage 1+): Orange, 1 HP, fast with weaving, score 15
- **Tank** (stage 2+): Purple, 4 HP with visible health bar, slow, score 35

**Enemy behaviors**:
- Spawn at top, move down with difficulty-scaled speeds
- Shoot pattern: "aimed" (toward player) or "spread" (radial), random 30-90 frame intervals
- Interceptors weave side-to-side using sine wave + frame counter (line 1212-1214)
- Shot down → particle burst + score + possible power-up

#### Boss (`boss` object, nullable)
Spawned after `KILLS_PER_STAGE` (18) enemy kills per stage. Each boss has:
- **Design** (from `BOSSES` array, line 508): Name, hull/wing/core colors, HP multiplier
- **Attack patterns** (line 1372–1408): Boss-specific radial, aimed, spiral, column, and homing attacks
- **State**: `entering` (slide-in animation), `telegraph` (pre-attack flash), `attackTimer`
- **HP system**: Scale varies by difficulty (`bossHpMul`) and design (`hpMul`)

Boss patterns are indexed by stage modulo 6 (6 unique designs, patterns repeat). Attack telegraph (white flash) lasts 22 frames before pattern fires.

#### Collision System
- **`rectsOverlap(a, b)`** (line 1069): AABB collision for all entities
- **Player bullets vs. enemies** (line 1232): Damage enemy, spawn particles; piercing lasers persist
- **Enemy bullets vs. player** (line 1276): Trigger hit flash, lose life
- **Boss collision** (line 1284): Treated like enemy contact
- **Power-up pickup** (line 1289): Apply immediately

### Game Progression

**Difficulty Scaling** (`DIFFICULTIES` object, line 485):
- **Easy**: 75 base spawn rate, 4 lives, 3 bombs, 0.8× boss HP
- **Normal** (default): 58 spawn rate, 3 lives, 2 bombs, 1.0× boss HP
- **Hard**: 40 spawn rate, 3 lives, 2 bombs, 1.3× boss HP

**Stage Progression**:
- Each stage requires 18 enemy kills → boss spawns
- 6 visual themes rotate (line 493) with changing sky, clouds, sun colors
- Boss defeated → stage increments, theme rotates, spawn timer resets
- Enemy type availability increases per stage (tanks at stage 2+)

**Scoring & Multipliers**:
- Chain mechanic (line 1087–1101): Each kill within 2.2 seconds increases combo
- Multiplier = 1 + floor(chain / 5), capped at 10×
- Boss kills grant 500 base score
- Bomb use grants 5 points per enemy killed

### Audio System (Web Audio API)

**No audio files** — all sounds are synthesized using the Web Audio API:
- **`beep(freq, dur, type, vol, opts)`** (line 566): Tonal synthesis with optional sweep, filter, pan
- **`noiseBurst(dur, vol, opts)`** (line 597): White noise with envelope
- **Master chain** (line 549–557): Dynamics compressor on master gain → speaker

**Sound effects** (`sfx` object, line 624):
- `shoot`, `laser`, `explodeGrunt`, `explodeInterceptor`, `explodeTank`
- `hit`, `bomb`, `powerup`, `bossHit`, `bossTelegraph`, `bossDown`, `stageClear`, `uiClick`, `gameover`

Spatial audio: `panFor(x)` (line 562) maps enemy X-position to stereo pan (-1 to 1).

### Rendering System

**Canvas**: 480×720 base, scales responsively (line 752–764)

**Drawing layers** (line 1520–1637):
1. **Background**: Gradient sky (theme-based), radial sun glow, parallax clouds
2. **Player ship**: `drawPlane()` with flame when weapon active
3. **Bullets**: Glowing rectangles (yellow vulcan, cyan laser) with optional shadow
4. **Enemy bullets**: Red circles
5. **Enemies**: Plane sprites with health bar for tanks
6. **Boss**: `drawBossShip()` with animated wings and pulsing core
7. **Power-ups**: Rotating colored squares
8. **Particles**: Fading circles from explosions

**Visual effects**:
- Screen shake (line 727–730): Reduces intensity if reduced-motion enabled
- Flash overlay (line 705–709): White flash on hit or bomb
- Vignette (line 687): Red pulsing edge when at critical HP (1 life)
- Stage banner & combo popups: Animated text overlays

### Input Handling

**Keyboard** (lines 768–776):
- Arrow keys / WASD: Movement
- Space: Fire
- X / Shift: Bomb
- P / Escape: Pause

**Mouse** (lines 836–861):
- Hover movement: Player glides toward cursor (lower priority than keyboard)
- Click to fire

**Touch** (lines 779–834):
- Left 55% of screen: Joystick for movement
- Right side: FIRE and BOMB buttons
- Visual joystick base and stick elements appear on touch

### UI & Overlays

**HUD** (top center): Score, best, multiplier, weapon level, hearts, bombs

**Boss bar** (top center when boss active): Boss name and health bar

**Overlay card** (center, semi-transparent backdrop):
- Main menu: Difficulty picker, start button, controls hint
- Pause menu: Resume button
- Game over: Final stats, best score highlight, play again button

**Accessibility**:
- Reduced motion support (line 521–540): Disables animations on `prefers-reduced-motion`
- Screen reader announcements (line 468–471): Updates live region on kills, stage changes, game over
- Keyboard navigation: All controls have keyboard equivalents

### Persistence

**LocalStorage keys** (line 463–466):
- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Audio mute state (1 or 0)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced motion preference override

All settings load at startup and persist across sessions.

### Performance Optimization

**Adaptive quality** (lines 473–483):
- Tracks average frame time over 45 frames
- Increases quality if avg > 25ms (drops below 40 FPS)
- Decreases quality if avg < 17.9ms (sustains >56 FPS)
- Quality affects:
  - Glow effects (shadow blur) in rendering
  - Particle count (line 1074): Full at 0, 60% at 1, 35% at 2

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with properties: `w, h, hp, speedMul, color, dark, score, minStage`
2. Optionally add `weave: true` for side-to-side movement
3. Update spawn logic (line 1022–1031) to filter by stage if needed
4. Add explosion sound to `EXPLODE_SFX` (line 678) if unique behavior needed

### Adjusting Difficulty
Edit `DIFFICULTIES` object (line 485):
- `spawnBase` / `spawnMin`: Lower = more frequent enemy spawning
- `enemySpeed`: Multiplier on all enemy velocity
- `enemyShootChance`: Probability (0–1) that enemy can shoot
- `lives` / `bombs`: Starting resources
- `bossHpMul`: Scales boss HP

### Adding a New Boss Pattern
1. Create pattern function using helper: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc. (lines 1324–1370)
2. Add 3 patterns to `BOSS_PATTERNS` array (line 1372) at the stage index
3. Pattern fires when `telegraph` timer reaches 0, `patternIndex` cycles through the array

### Tweaking Game Feel
- **Player movement speed**: Change `player.speed` in `resetGame()` (line 877, currently 260)
- **Fire rate**: Adjust `fireDelay` (line 1177, 0.15 for vulcan, 0.1 for laser)
- **Bomb damage**: Damage vs. boss is hardcoded to 24 (line 1117)
- **Stage length**: `KILLS_PER_STAGE` constant (line 491, currently 18)
- **Weapon switch cost**: Check `applyPowerUp()` logic (line 1304–1321)

## Important Patterns

### Entity-Centric Update Loop
Each entity type (bullets, enemies, etc.) is updated in a single pass, then filtered for removal. This keeps entity lifetimes simple and collision checks localized:
```js
bullets.forEach(b => { b.y -= b.speed * dt; ... });
bullets = bullets.filter(b => b.y > -20);
```

### Temporal Game Events
Screen shake, flash effects, particle bursts, and boss telegraphs are driven by frame counters or timers that decay each frame. No external timers or event queues—everything is state-driven.

### Collision Resolution Order
Collisions are checked in this order:
1. Player bullets vs. enemies (damage enemies, delete bullets)
2. Player bullets vs. boss (damage boss, delete bullets)
3. Enemies vs. player (lose life)
4. Enemy bullets vs. player (lose life)
5. Boss vs. player (lose life)
6. Power-ups vs. player (apply effect)

This order prevents the player from "dying" and picking up a life in the same frame.

### Rendering Independence
The draw function is called regardless of game state (running, paused, game over). The overlay layer manages UI visibility via `hidden` attribute. This keeps rendering logic clean.

## Notes for Future Development

- **No framework dependencies**: Avoid adding external libraries. Web Audio, Canvas 2D, and localStorage are all native APIs.
- **Responsive design**: The 480×720 resolution is the design target; CSS handles scaling via CSS clamp() functions.
- **Touch-first**: Mobile controls are critical to the game's appeal. Test changes on actual mobile devices or mobile emulation.
- **Audio is synthesized**: Adding new sounds means writing new synthesis functions, not importing files. See `sfx` object patterns.
- **Theme system**: Visual themes cycle by stage; adding more requires editing the `THEMES` array with sky gradient stops and cloud/sun colors.
- **Accessibility first**: The `reducedMotion` flag, screen reader live region, and keyboard-only play are essential features—don't remove or disable them.

