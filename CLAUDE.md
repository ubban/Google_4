# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML application. The entire game—graphics, audio, game logic, and UI—is contained in `index.html` with embedded CSS and JavaScript.

## Architecture

### Core Components

The game uses a monolithic architecture with these major systems:

1. **Game Loop** (`loop()` function, line 1640)
   - Main `requestAnimationFrame` loop that updates game state and renders
   - Time delta (`dt`) is capped at 50ms to prevent runaway physics
   - Performance tracking with adaptive quality scaling

2. **Player & Input** (lines 768-861)
   - Keyboard input: Arrow keys/WASD for movement, Space to fire, X/Shift for bomb
   - Mouse steering: player glides toward cursor when hovering canvas
   - Touch joystick: left zone for movement, buttons for fire/bomb
   - All input types coexist; keyboard/joystick takes priority over mouse

3. **Rendering** (`draw()` function, line 1520)
   - Canvas 2D API with shader-like effects (gradients for sky, glow)
   - Draws in order: sky/clouds → player → bullets → enemies → boss → particles
   - Screen shake on impacts
   - Theme cycling every stage (6 themes in `THEMES` array, line 493)
   - Quality scaling reduces particle count and shadow blur at low FPS

4. **Game State** (lines 863-910)
   - `player`: position, weapon, level, cooldowns, invulnerability frames
   - `enemies`, `bullets`, `enemyBullets`, `powerUps`, `particles`
   - `boss`: spawns after 18 kills per stage; has health bar and telegraph animations
   - `score`, `lives`, `stage`, `chain` (for multiplier), `difficulty`

5. **Audio** (lines 542-676)
   - Synthesized via WebAudio (no external files)
   - `beep()`: oscillator with frequency sweep + optional filter/pan
   - `noiseBurst()`: white noise with envelope
   - `sfx` object has ~12 preset sounds (shoot, explode, bomb, boss hit, etc.)
   - Respects muted state and reduced-motion preference

6. **Physics & Collisions**
   - Movement: `update()` reads input, applies speed, clamps to bounds (line 1148)
   - Collision detection: `rectsOverlap()` (line 1069) for AABB checks
   - Bullet pierce: laser weapons have pierce levels; bullets removed on impact
   - Enemy patterns: "spread" (fixed angles) or "aimed" (toward player)

7. **Boss System** (lines 1411-1454)
   - Spawns after 18 enemy kills
   - 6 boss designs cycle with increasing HP multiplier per stage
   - Each boss has 3 distinct attack patterns (radial, spread, homing, etc.)
   - Telegraph animation before each pattern (player warning)
   - Damaged by bullets (3 HP per hit) and bomb (24 HP flat)

### Data Structures

- **Entities** (player, enemies, boss): `{ x, y, w, h, ... }`
- **Bullets**: `{ x, y, vx, vy, w, h, speed, pierce, laser }`
- **Enemy bullets**: `{ x, y, w, h, speed, vx, radial, dx, dy }`
- **PowerUps**: `{ x, y, w, h, speed, type: 'life'|'bomb'|'weapon' }`
- **Particles**: `{ x, y, vx, vy, life, maxLife, color }`

### Game Flow

```
Start Screen (difficulty picker) 
  → resetGame() → Game Loop
    → spawn enemies every ~50 frames
    → after 18 kills: warn & spawn boss
    → boss defeated: stage++, reset kills, cycle theme
    → player loses all lives → endGame() → Game Over screen
```

## Key Functions

| Function | Line | Purpose |
|----------|------|---------|
| `update(dt)` | 1148 | Update all entities (player, enemies, bullets, collisions) |
| `draw()` | 1520 | Render game state to canvas |
| `fireWeapon()` | 1124 | Create bullets based on weapon type/level |
| `spawnEnemy()` | 1022 | Create random enemy with weighted type selection |
| `spawnBoss()` | 1043 | Create stage boss with design and HP |
| `updateBoss(dt)` | 1411 | Boss movement, telegraph, attack patterns |
| `useBomb()` | 1103 | Clear enemy bullets, damage all enemies + boss |
| `applyPowerUp(type)` | 1304 | Handle life/bomb/weapon pickups |
| `endGame()` | 1469 | Handle game over, save best score |

## Difficulty & Content Tuning

### Difficulty Settings (line 485)
- `easy`: slower spawns (75 → 40 frame min), 1.0x enemy speed, lower shoot chance
- `normal`: medium spawns (58 → 26), 1.4x speed, 0.7 shoot chance
- `hard`: fast spawns (40 → 18), 1.9x speed, 0.95 shoot chance
- Affects lives (3-4), bombs (2-3), boss HP multiplier

### Enemy Types (line 502)
- `grunt`: base weak unit, red, 1 HP, score 10
- `interceptor`: fast & agile, orange, 1 HP, weaves side-to-side, score 15
- `tank`: heavy, purple, 4 HP, slow, score 35 (available from stage 2)

### Boss Patterns (line 1372)
Six distinct boss designs (VANGUARD → TITAN) each with 3 attack patterns. Patterns use helper functions:
- `fireRadial()`: 360° ring of bullets
- `fireAimedSpread()`: fan toward player with spread
- `fireColumns()`: vertical columns
- `fireSpiralBurst()`: rotating ring
- `fireDoubleRing()`: two concentric rings
- `fireHomingSeeded()`: seeded homing with spread

## Development Notes

### Rendering Quality
- `quality` variable (line 474) auto-adjusts: 0 (full) → 1 (medium) → 2 (low)
- Tracks average frame time; increases quality if dropping below ~40 FPS
- At low quality: fewer particles, no shadow blur, no glow

### Accessibility
- Reduced motion preference respected (prefers-reduced-motion media query, line 520)
- Screen reader announcements via `announce()` (line 468)
- Semantic HTML with ARIA labels

### LocalStorage Keys
- `skyfighter.bestScore`: high score
- `skyfighter.muted`: audio mute state (0/1)
- `skyfighter.difficulty`: last selected difficulty
- `skyfighter.reducedMotion`: motion preference (0/1)

### Running & Testing
To run the game: open `index.html` in a modern browser. No build, no dependencies, no server needed.

**Test checklist for changes:**
- Player movement is smooth and bounded
- Bullets fire at correct rate and don't overlap visually
- Enemies spawn at increasing difficulty (frame counter should ramp)
- Collisions (bullets hit enemies, bombs work, player damage) are precise
- Boss patterns are distinct and readable
- Audio works (at least on desktop; mobile may need user gesture)
- Touch controls work on mobile (joystick, fire, bomb)
- High score persists across refreshes

## Common Edits

### Add a new enemy type
1. Add entry to `ENEMY_TYPES` (line 502) with `w`, `h`, `hp`, `speedMul`, `color`, `score`, `minStage`
2. Update `spawnEnemy()` filter (line 1026) if stage-gated
3. Add explosion SFX to `EXPLODE_SFX` (line 678) if needed

### Add a new boss
1. Add to `BOSSES` array (line 508): `{ name, hullTop, hullBottom, wing, core, hpMul }`
2. Add pattern functions (line 1372) to `BOSS_PATTERNS` at matching index
3. Adjust `BOSS_PATTERNS[index].length` if pattern count differs

### Adjust difficulty
- Tune `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `bossHpMul` in `DIFFICULTIES` (line 485)
- Adjust `KILLS_PER_STAGE` (line 491) to shorten/lengthen stages

### Add audio feedback
- Call existing `sfx.*()` from relevant locations, or
- Define new sound in `sfx` object using `beep()` and/or `noiseBurst()` with frequency/filter/pan options

### Change visual style
- Edit `THEMES` (line 493) colors: `sky` (gradient stops), `cloud`, `sun` RGB values
- Adjust entity colors in `ENEMY_TYPES` and `BOSSES`
- Modify CSS (lines 10-358) for HUD, overlay, buttons, animations

## Notes for Future Work

- **Single file design**: simplicity & no build required, but making this modular would help with large feature adds
- **Performance**: Canvas 2D is not optimized for 20+ enemies; quality scaling mitigates but consider OffscreenCanvas if needed
- **Weapon balance**: laser is strong at high levels; could add cooldown scaling or recoil
- **Boss difficulty**: patterns scale with stage HP, not attack speed; late-stage bosses may feel repetitive
