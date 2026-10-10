# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 web application. The game features:
- Responsive canvas-based rendering (480×720 native resolution, scales up to 1.6x)
- Procedurally generated WebAudio sound effects (no external audio assets)
- Touch controls for mobile, keyboard/mouse controls for desktop
- Progressive difficulty with multiple difficulty levels (Easy, Normal, Hard)
- Multi-stage boss battles with distinct attack patterns
- Weapon upgrade system (Vulcan spreading gun, Laser with piercing shots)
- Score multiplier chain mechanics and adaptive performance scaling

## Codebase Structure

The entire game is a **single `index.html` file** (~1670 lines) containing:

1. **HTML structure** (362-420): Game canvas, HUD elements, overlay screens (start menu, pause, game over)
2. **CSS styling** (10-358): Responsive design, animations, mobile UI controls, accessibility features
3. **JavaScript game engine** (422-1667):
   - **Initialization & state** (863-869): Player, game objects, score tracking
   - **Input handling** (767-861): Keyboard, mouse steering, touch controls with virtual joystick
   - **Game loop** (1640-1665): `requestAnimationFrame` with delta-time updates
   - **Update function** (1148-1302): Physics, spawning, collisions, state management
   - **Draw function** (1520-1638): Canvas rendering with quality scaling
   - **Game mechanics**: Boss AI (1411-1439), weapon firing (1124-1146), bomb (1103-1122), powerups (1304-1322)
   - **Audio system** (542-676): WebAudio synthesis with oscillators, noise bursts, filters, panning

## How to Work with This Codebase

### View the Game
- Open `index.html` directly in a modern web browser (Chrome, Firefox, Safari, Edge)
- The game runs at 480×720 internally and scales responsively
- No server or build process required

### Edit & Test
- Make changes directly to `index.html` (HTML, CSS, and JavaScript in one file)
- Refresh the browser to see changes
- Game state is preserved in localStorage (best score, difficulty, settings)

### Key Constants & Structures

**Difficulty settings** (line 485-489):
```javascript
DIFFICULTIES = {
  easy: { spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul },
  normal: { ... },
  hard: { ... }
}
```

**Enemies** (line 502-506): Types are `grunt`, `interceptor`, `tank` with speed, HP, and spawn stage
**Bosses** (line 508-515): 6 unique boss designs with different attack patterns indexed by stage
**Boss attack patterns** (line 1372-1409): Each boss has 3 attack patterns that rotate

**Game state variables** (line 864-869):
- `player`: Position, weapon, level, lives, invulnerability
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`: Entity arrays
- `score`, `lives`, `stage`, `multiplier`, `chain`: Game metrics
- `running`, `paused`, `gameOver`: Control flags

### Canvas & Rendering

- **Canvas dimensions**: 480×720 (BASE_W / BASE_H)
- **Responsive scaling**: Up to 1.6x, maintains aspect ratio
- **Frame-based animation**: Uses `frame++` counter for timed events
- **Delta-time**: All physics use `dt` (delta time in seconds) for frame-rate independence
- **Quality levels** (0-2): Automatically adapts based on frame time to reduce visual effects on slow devices

### Input System

**Keyboard**: Arrow keys / WASD for movement, Space/click for fire, X/Shift for bomb, P/Esc to pause
**Mouse**: Move over canvas to steer (auto-glide), click to fire
**Touch**: Virtual joystick on left, Fire/Bomb buttons on right, tap to fire

Keyboard and joystick take priority over mouse to prevent conflicts.

### Audio System

Uses WebAudio API for synthesized sound:
- **beep()**: Oscillator-based tones with frequency sweep and filters
- **noiseBurst()**: White noise with envelope and filtering
- Each sound (shoot, laser, explosions, powerup, etc.) has a unique signature

Audio is muted by default (requires user interaction) due to browser autoplay policies.

### Collision & Physics

- **Rectangle overlap**: `rectsOverlap(a, b)` checks AABB (axis-aligned bounding box)
- **Movement**: Clamped to canvas bounds, velocity-based with delta-time
- **Bullets**: Persist off-screen briefly, remove when off-bounds
- **Enemies**: Pathing includes optional weave pattern (sine wave movement)

### Game Progression

1. **Stages**: Increment after each boss defeat; theme and enemy difficulty increase
2. **Spawning**: Enemies spawn at top, ramp up in frequency as frame count increases
3. **Boss threshold**: After 18 kills (`KILLS_PER_STAGE`), warning shows and boss spawns after 1.2s
4. **Boss patterns**: Each boss has 3 attack patterns; telegraph phase precedes each pattern
5. **Difficulty scaling**: Affects spawn rate, enemy speed, boss HP, starting lives/bombs

### LocalStorage Keys

- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Audio mute state (0 or 1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Motion preference (0 or 1); defaults to OS preference

### Accessibility

- ARIA live region (`#srLive`) announces game state changes
- Respects `prefers-reduced-motion` media query; disables screen shake, flash, animations
- Color contrast meets WCAG AA standards
- Touch controls scale based on viewport

## Common Development Tasks

### Add a new enemy type
1. Add entry to `ENEMY_TYPES` (line 502-506) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add explosion SFX mapping to `EXPLODE_SFX` (line 678)
3. Update `spawnEnemy()` (line 1022-1041) weight logic if needed

### Modify boss attack patterns
1. Add attack function(s) to use existing helpers: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`
2. Add new 3-pattern array to `BOSS_PATTERNS` (line 1372-1409) in stage order
3. Patterns are called during boss telegraph phase (line 1427)

### Add a new weapon or upgrade
1. Modify `fireWeapon()` (line 1124-1146) to add weapon type or spread pattern
2. Update `weaponLabel()` (line 871-873) if adding weapon names
3. Modify `applyPowerUp()` (line 1304-1322) weapon logic if changing progression

### Adjust difficulty
Modify `DIFFICULTIES` object (line 485-489) constants:
- `spawnBase` / `spawnMin`: Control enemy spawn frequency (lower = faster)
- `enemySpeed`: Enemy movement speed multiplier
- `enemyShootChance`: Probability enemy fires at player
- `lives` / `bombs`: Starting values
- `bossHpMul`: Boss health scaling

### Add visual or audio effects
- **Sound**: Use `beep()`, `noiseBurst()` in the `sfx` object (line 624-676)
- **Particles**: Push to `particles` array with `{ x, y, vx, vy, life, maxLife, color }`
- **Screen shake**: Call `screenShake(magnitude, duration)`
- **Flash effect**: Call `flashHit()` or `flashBomb()`

## Testing

1. **Desktop**: Open `index.html` in browser, test keyboard/mouse controls
2. **Mobile**: Test on phone/tablet; ensure touch controls are responsive and buttons are reachable
3. **Performance**: Check frame time in browser DevTools; game adapts quality if FPS drops
4. **Accessibility**: Use keyboard only (Tab to navigate menus, no mouse), test with screen reader
5. **LocalStorage**: Verify settings persist across sessions (clear browser storage to reset)

## Git Workflow

Currently on branch `claude/inspiring-planck-hangh6` for development. Push changes with descriptive commits.
