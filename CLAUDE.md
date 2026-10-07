# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a self-contained HTML5 vertical shoot-'em-up game (Raiden-style). The entire codebase is a single `index.html` file (~1670 lines) with embedded CSS and JavaScript. It requires no build step, no external dependencies, and no npm packages.

- **Type**: Browser-based game
- **No build process**: Run directly in a browser via `index.html`
- **Single file**: All code (HTML, CSS, JS) is in `index.html`
- **No external assets**: All graphics drawn with Canvas 2D API; all sound synthesized with WebAudio API

## Running the Game

**Development**: Open `index.html` directly in a browser (Firefox, Chrome, Safari all support it). The game auto-detects touch devices and adjusts controls accordingly.

**Testing locally**: Use a simple HTTP server to avoid CORS issues if testing from a file:// URL:
```bash
python3 -m http.server 8000
# Then visit http://localhost:8000
```

## Game Architecture

### Core Game Loop

The engine runs at `requestAnimationFrame` (~60fps target with adaptive quality). Each frame:
1. **Input** — read keyboard/mouse/touch state (lines 768–861)
2. **Update** — move entities, check collisions, spawn enemies (lines 1148–1302)
3. **Draw** — render canvas with theme-based sky, entities, particles (lines 1520–1638)

Frame timing is clamped to max 0.05s to prevent large dt jumps (line 1644).

### Entity System

Entities are plain objects with position (x, y), dimensions (w, h), and type-specific properties:
- **Player** (line 877): `{ x, y, w, h, speed, cooldown, weapon, level, invuln }`
- **Bullets** (line 1129): `{ x, y, w, h, speed, vx, pierce, laser? }`
- **Enemies** (line 1032): `{ x, y, w, h, hp, speed, type, pattern, weave?, ... }`
- **Enemy Bullets** (line 1219): simple radial physics with `dx, dy` per frame
- **Boss** (line 1047): large entity with HP bar, telegraph timer, attack patterns
- **Particles** (line 1076): ephemeral explosions with velocity and alpha fade
- **Power-ups** (line 1061): small pickups that spawn on enemy death

All entities stored in top-level arrays (`bullets`, `enemies`, `boss`, etc.) and updated/culled each frame.

### Collision Detection

Simple AABB (axis-aligned bounding box) check via `rectsOverlap()` (line 1069):
```javascript
Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2
```
Used for: player–bullet, player–enemy, bullet–enemy, bomb–all, bullet–boss.

### Difficulty System

Three difficulty tiers (lines 485–489) control spawn rates, enemy speed, shoot chance, HP multipliers:
- **Easy**: `spawnBase: 75`, `spawnMin: 40`, lives: 4
- **Normal**: `spawnBase: 58`, `spawnMin: 26`, lives: 3 (default)
- **Hard**: `spawnBase: 40`, `spawnMin: 18`, lives: 2 (plus increased boss HP)

Difficulty is persisted to localStorage and selectable from the start menu (lines 741–750).

### Weapon System

Player has two weapons with 3 levels each (line 877):
- **Vulcan**: spread pattern increases with level (0→1→2 lanes)
- **Laser**: multi-lane shots; pierce through enemies

Power-up system (line 1304): Level up, or swap weapons and reset to level 1. Weapon state persists across frames.

### Scoring & Chain Multiplier

Score multiplier (lines 1087–1101) increases with consecutive kills in a 2.2-second window (`chainTimer`):
- Multiplier = 1 + floor(chain / 5), capped at 10x
- Breaks when player takes damage or timer expires

### Boss Patterns

Six unique boss designs (lines 508–514), each with 3 attack patterns (lines 1372–1409):
- **VANGUARD**: radial bursts + aimed fan + V-fan
- **BEHEMOTH**: heavy columns + homing + dense radial
- **SPECTER**: teleport dash + spiral + double aimed
- **DREADNOUGHT**: 8-way cross + homing + radial wall
- **WRAITH**: teleport barrage + double ring + cross-spread
- **TITAN**: columns + full ring + relentless homing

Boss selection cycles through the list based on stage (line 1045). Each pattern fires from the boss center with specific bullet counts and speeds.

### Visual Themes

Six color themes (lines 493–500) cycle per stage: sky gradient, cloud color, sun color. Used for procedural background gradient (lines 1529–1544), not external images.

### Audio

No external sound files. All sounds synthesized in real-time using WebAudio API (lines 545–676):
- **Oscillator-based**: `beep()` with frequency sweep, filter, pan, volume envelope
- **Noise-based**: `noiseBurst()` for explosions

SFX library (lines 624–676) covers: shoot, laser, explosions (grunt/interceptor/tank), hit, bomb, power-up, boss sounds, game-over.

Audio context is lazy-initialized on first input (line 774, `ensureAudio()`). Mute state persists to localStorage (line 738).

### Input Handling

**Keyboard/Gamepad**: Arrow keys + WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause.
**Mouse**: Move to steer (smooth glide toward cursor), click to fire. Mouse input has lower priority than keyboard (line 1157).
**Touch**: Left half of screen is joystick (lines 785–806), right half has FIRE and BOMB buttons. Joystick visualization appears on touch (lines 789–791).

Input never "fights": if keyboard input is active, mouse is ignored (line 1164).

### Adaptive Quality

Performance tracker (lines 474–483) monitors frame time over 45 frames:
- If avg > 1/40s (25ms), reduce quality (step 0→1→2)
- If avg < 1/56s (18ms), increase quality
- Quality affects particle count (line 1074), glow effects (line 1579), shadow blur (line 1002)

### Game State Management

Top-level variables track:
- `running`, `paused`, `gameOver` — game state flags
- `score`, `lives`, `stage`, `kills` — progression counters
- `bombCount` — consumed/reset by bomb usage (line 1105)
- `chain`, `chainTimer`, `multiplier` — scoring streak
- `frame` — global frame counter used for animations and RNG seeding

## Extending the Game

### Adding an Enemy Type

1. Add to `ENEMY_TYPES` (line 502) with shape, HP, speed, score, minStage
2. Update spawn logic in `spawnEnemy()` if needed (line 1022)
3. Add explosion SFX to `EXPLODE_SFX` (line 678) if unique sound desired

### Adding a Boss Pattern

1. Add pattern function to `BOSS_PATTERNS[<index>]` array
2. Function receives boss object, calls firing helpers: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
3. Boss selection is automatic via `(stage - 1) % BOSSES.length`

### Adding a Weapon

1. Modify `fireWeapon()` (line 1124) to handle new weapon type
2. Update `weaponLabel()` (line 871) for HUD display
3. Add power-up logic in `applyPowerUp()` (line 1304)

### Tuning Game Feel

Key knobs (no constants file; search for values):
- **Spawn rate**: `DIFFICULTIES[difficulty].spawnBase` and `.spawnMin` (line 486)
- **Enemy bullet speed**: `enemyBullets.push({ ... speed: 200 ... })` (scattered throughout)
- **Screen shake**: `screenShake(magnitude, duration)` (line 727); reduced by half in reduced-motion mode
- **Enemy HP**: `ENEMY_TYPES[type].hp` (line 502)
- **Bullet pierce**: `pierce: 0` or `pierce: 1` etc. on bullet creation (line 1131, 1141)

### Accessibility

Respects `prefers-reduced-motion` media query (line 520) and provides UI toggle (line 535). Reduces animation duration and magnitude, disables glow effects, skips certain CSS animations (line 349–358).

Screen reader announcements via `announce()` (line 468) update a live region (line 390).

## Storage

Game persists to browser localStorage (keys prefixed `skyfighter.`):
- `bestScore` — high score
- `muted` — sound on/off
- `difficulty` — last selected difficulty
- `reducedMotion` — accessibility preference

## Common Tasks

**Change difficulty multipliers**: Edit `DIFFICULTIES` object (line 485). Each change affects spawn rate, enemy behavior, and HP scaling.

**Adjust visual theme**: Add to `THEMES` array (line 493). Each theme has sky gradient stops, cloud/sun colors, and a display name.

**Tweak physics**: Search for numeric values in `update()` (line 1148) and helper functions. E.g., `player.speed = 260` (line 877).

**Add UI feedback**: Use `showComboPop()`, `showStageBanner()`, `screenShake()`, or `flashHit()`/`flashBomb()` for visual/audio cues.

## Notes

- Canvas is 480×720px logical size; scaled to fit window (line 757). Responsive, works on mobile.
- No external dependencies means no npm, no build tool, no transpilation. Pure ES5-ish vanilla JavaScript.
- All collisions are AABB; no pixel-perfect or rotated hitboxes.
- Boss telegraph (line 1421) is a visual delay before attack patterns fire; tuned to 22 frames (~367ms at 60fps).
