# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-file HTML5 web application. The game features:
- Player aircraft with weapon upgrades
- Multiple enemy types with varying behaviors
- Six unique boss encounters with distinct attack patterns
- Scoring system with multiplier chains
- Touch and keyboard controls with adaptive quality rendering
- Procedurally themed stages with different color schemes
- Web Audio API synthesis for all sound effects (no external audio assets)

## Architecture

### Single-File Structure
The entire game (HTML, CSS, JavaScript) is in `index.html`. This monolithic approach keeps dependencies minimal and enables easy distribution.

### Game Loop Architecture
The game follows a traditional game engine loop pattern:

1. **Input Processing** (keyboard, mouse, touch): Updates movement vector and fire state
2. **Update Phase**: 
   - Move entities (player, bullets, enemies)
   - Spawn new enemies based on difficulty and progression
   - Check collision detection (bullets vs enemies, boss, player)
   - Manage spawning of bosses when stage thresholds are met
3. **Draw Phase**: Render everything to canvas with screen shake effects

Key state variables:
- `player`: Aircraft position, health, weapon state
- `enemies[]`, `enemyBullets[]`: Active entities
- `bullets[]`: Player projectiles
- `boss`: Current boss ship (or null)
- `stage`, `stageKills`, `kills`: Progression tracking
- `score`, `multiplier`, `chain`: Scoring system

### Three Tiers of Difficulty

Each tier scales:
- Enemy spawn rate (spawnBase, spawnMin)
- Enemy speed and shoot chance
- Available lives and bombs
- Boss HP multiplier

See `DIFFICULTIES` constant (~line 485) for tuning values.

### Enemy AI

- **Grunt** (red): Basic straight-down movement, slower shooting
- **Interceptor** (orange): Fast with weaving side-to-side pattern
- **Tank** (purple): Slow, high HP (4 hits), high score reward

Each enemy chooses between two shoot patterns:
- `spread`: Three bullets at angles
- `aimed`: Single bullet tracking player position

### Boss System

Six boss designs cycle by stage. Each has:
- Unique visual appearance (colors, shape)
- Distinct attack pattern set (3 phases per boss)
- Telegraphing animation before attack
- Entering animation when spawned
- Variable HP based on difficulty and boss multiplier

Attack patterns use helper functions:
- `fireRadial()`: Circular burst
- `fireAimedSpread()`: Fan toward player
- `fireColumns()`: Vertical streams
- `fireSpiralBurst()`: Rotating ring
- `fireDoubleRing()`: Two concentric rings
- Other specialized patterns

Boss patterns live in `BOSS_PATTERNS` array (~line 1372).

### Scoring System

- Base score for each kill (10-35 points depending on enemy type)
- Boss kill: 500 points
- Bomb use: 5 points per enemy cleared
- Multiplier increases by 1x for every 5 consecutive kills
- Chain breaks if player takes damage or too much time passes (2.2s)

### Input Handling

**Keyboard**: Arrow keys or WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause
**Mouse**: Move to steer, click to fire (mouse input doesn't fight keyboard/joystick)
**Touch**: Left half is movement joystick, FIRE/BOMB buttons on right

### Audio

All sound effects are synthesized via Web Audio API (no external audio files):
- Oscillators for tonal sounds (beep frequencies, sweeps)
- Noise bursts for percussion/explosions
- Dynamic compression + master gain for polished mix
- Spatial panning based on source position
- Optional muting (persisted in localStorage)

See `sfx` object (~line 624) for sound definitions.

### Visual Effects

- **Screen shake**: Magnitude and duration based on event severity
- **Vignette pulse**: Red pulsing overlay when health critical (1 heart)
- **Flash effects**: White flash on hit, brighter on bomb
- **Combo popup**: Large text that pulses when reaching 10-kill milestones
- **Stage banner**: Animated text when entering new stage
- **Boss health bar**: Animated fill tracking remaining boss HP
- **Particle explosions**: Radial burst scaled by quality setting
- **Animated backgrounds**: Multi-layer scrolling clouds with stage-specific sky gradients

Reduced-motion mode disables animations for accessibility (respects `prefers-reduced-motion`).

### Adaptive Quality

Frame time tracking (~line 474) automatically adjusts visual quality:
- Quality 0: Full effects (glows, shadows, max particle count)
- Quality 1: Medium (fewer particles, some glow reduction)
- Quality 2: Low (minimal effects, sparse particles)

This keeps 60 FPS stable on lower-end devices.

## Development & Testing

### Running the Game

Open `index.html` in any modern browser. No build step or server required.

**Local testing:**
```bash
# Simple HTTP server (Python 3)
python -m http.server 8000
# Then visit http://localhost:8000/index.html

# Or with Node.js
npx http-server
```

### Testing Specific Features

**Test difficulty balance:**
- Use developer console: `difficulty = 'easy'/'normal'/'hard'` then reload
- Check `DIFFICULTIES` object to tune spawn rates and HP multipliers

**Test boss patterns:**
- Set `stage = X` in console before game starts to spawn specific boss
- Boss attack patterns are in `BOSS_PATTERNS[designIndex]`

**Test scoring/multiplier:**
- Quick way: modify `addScore()` or `chain` logic to test thresholds

**Accessibility:**
- Toggle reduced motion: Click icon or localStorage: `localStorage.setItem('skyfighter.reducedMotion', '1')`
- Screen reader announces lives, stage progression via `srLive` live region
- Test with browser DevTools reducing motion settings

### Canvas Sizing

Canvas is 480x720 (portrait), scaled up to 1.6x max or fit to window. Responsive layout uses CSS transforms rather than changing canvas resolution.

### State Persistence

Uses `localStorage` for:
- Best score: `skyfighter.bestScore`
- Mute state: `skyfighter.muted`
- Difficulty preference: `skyfighter.difficulty`
- Reduced motion preference: `skyfighter.reducedMotion`

## Key Constants & Configuration

| Constant | Location | Purpose |
|----------|----------|---------|
| `BASE_W`, `BASE_H` | ~753 | Canvas logical dimensions |
| `DIFFICULTIES` | ~485 | Spawn rate, speed, lives per difficulty |
| `THEMES` | ~493 | Sky gradients, cloud colors, theme names |
| `ENEMY_TYPES` | ~502 | Enemy stats (HP, speed, score) |
| `BOSSES` | ~508 | Boss names, colors, HP multipliers |
| `BOSS_PATTERNS` | ~1372 | Boss attack pattern sequences |
| `KILLS_PER_STAGE` | ~491 | Kills needed to trigger boss (18) |

## Important Code Sections

| Function | Purpose | Notes |
|----------|---------|-------|
| `update(dt)` | Main game logic per frame | Collision, entity movement, spawning |
| `draw()` | Render all game elements | Sky gradient, entities, HUD updates |
| `fireWeapon()` | Player shoots based on weapon | Vulcan spreads, laser pierces |
| `updateBoss(dt)` | Boss movement and attack timing | Telegraph delay before patterns |
| `spawnEnemy()` | Create new enemy with random type | Weighted by stage and difficulty |
| `rectsOverlap()` | Simple AABB collision detection | Used for all hit detection |

## When Modifying the Game

**Adding new enemy types:**
1. Add entry to `ENEMY_TYPES` with stats (w, h, hp, score, etc.)
2. Add minStage to control when they appear
3. Update `spawnEnemy()` weight checks if needed
4. Add explosion sound reference to `EXPLODE_SFX` if different

**Adding new boss patterns:**
1. Define pattern function (takes boss `b`, manipulates `enemyBullets`)
2. Add to appropriate boss's pattern array in `BOSS_PATTERNS`
3. Pattern is called when `telegraph` animation ends

**Tweaking difficulty:**
- Modify `DIFFICULTIES` object spawn rates and speeds
- Adjust `d.bossHpMul` to make bosses harder/easier
- Change `KILLS_PER_STAGE` to extend/shorten stage length

**Changing visual themes:**
- Add new entry to `THEMES` array with sky color stops and cloud colors
- Themes cycle by `(stage - 1) % THEMES.length`

## Known Constraints & Patterns

- **No external assets**: All sounds synthesized, all graphics drawn via Canvas 2D
- **Single IIFE**: Entire game in one IIFE to avoid global pollution
- **LocalStorage only**: No server backend; high scores stored in browser
- **Collision via AABB**: No pixel-perfect or rotation-aware collision
- **Particle system is simple**: Only color, position, velocity—no rotation or scale
- **Boss telegraphing**: Always 22 frames (~366ms) to telegraph before attack
- **Stage themes**: Cycle in order, visuals only—mechanics identical between stages

## Performance Notes

- Adaptive quality system keeps frame rate stable on varied hardware
- Enemy and bullet arrays cleaned up each frame (filter out off-canvas)
- Canvas is fixed 480x720; CSS scaling handles responsive sizing
- No image assets or external requests—game runs offline after page load
- Audio context created on first user interaction (Web Audio API requirement)
