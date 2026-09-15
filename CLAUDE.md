# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a vertical shoot-'em-up game built entirely in a single HTML file with no external dependencies. It's a fully self-contained game featuring procedurally challenging enemies, multiple boss encounters, difficulty levels, and WebAudio synthesized sound effects.

## Architecture

The game uses a classic game loop architecture implemented in a single `<script>` block within `index.html` (starting around line 422):

### Core Game State (line 864)
```
player, bullets, enemies, enemyBullets, particles, powerUps, boss
score, lives, maxLives, running, paused, gameOver
bombCount, maxBombs, multiplier, chain, stage
```

### Main Systems

#### 1. Game Loop (line 1640)
- **`loop(now)`** - Main RAF-driven loop that:
  - Calculates delta time (capped at 50ms)
  - Tracks performance and adapts quality
  - Calls `update(dt)` when running and not paused
  - Calls `draw()` every frame
  - Schedules next frame via requestAnimationFrame

#### 2. Update System (line 1128)
- **`update(dt)`** - Handles all game logic:
  - Player movement and weapon firing
  - Enemy spawning (difficulty ramps over time via frame counter)
  - Enemy behavior (movement, shooting patterns)
  - Boss logic via `updateBoss(dt)`
  - Collision detection (player vs enemies, bullets vs enemies, bullets vs boss, pickups)
  - Particle system updates
  - Chain/multiplier logic

#### 3. Rendering System (line 1520)
- **`draw()`** - Canvas-based 2D rendering:
  - Screen shake effect applied via canvas transform
  - Theme-based sky gradient (6 themes, one per stage)
  - Cloud parallax layers with opacity
  - Player (with invulnerability blink, laser aura when equipped)
  - Bullets (yellow/lime with glow on high quality, shadow blur)
  - Enemy bullets (red circles)
  - Enemies (using shared `drawPlane()` function)
  - Health bars for tank enemies
  - Boss ship via `drawBossShip()` (not shown in excerpt)
  - Power-ups (rotating squares, color-coded)
  - Particle explosions
  - Quality setting affects shadow/glow rendering

#### 4. Input System (line 768-862)
Three input methods, with keyboard taking priority over mouse when active:
- **Keyboard**: Arrow keys / WASD to move, Space to fire, X / Shift to bomb, P / Esc to pause
- **Mouse**: Move to steer, click to fire (priority: keyboard > mouse > nothing)
- **Touch**: Virtual joystick (left zone), FIRE/BOMB buttons (right zone)

#### 5. Audio System (line 542-676)
- **WebAudio synthesizer** - no external assets
- Dynamically creates oscillator-based SFX with frequency sweeps, filters, and panning
- Separate sound effects for each action (shoot, laser, explode variants, bomb, powerup, boss telegraph, etc.)
- Master gain with compressor for consistent loudness
- Can be muted via button; state persists in localStorage

#### 6. Difficulty System (line 485-489)
Three preset difficulty levels (`DIFFICULTIES` object):
- **Easy**: Slower spawn rate, slower enemies, higher player HP/bombs, easier boss
- **Normal**: Balanced (default)
- **Hard**: Faster spawn rate, faster enemies, lower HP/bombs, tougher boss

Each difficulty modifies spawn timers, enemy speed, shoot chance, and boss multiplier.

#### 7. Enemy & Boss System
- **Enemy Types** (line 502-506):
  - `grunt` (34x34, 1 HP, red) - minions
  - `interceptor` (26x26, 1 HP, orange, weaving) - faster threats
  - `tank` (46x46, 4 HP, purple) - armored, slower
  
- **Boss Types** (line 508-515):
  - 6 boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
  - Each has unique visual colors (hull top/bottom, wing, core) and HP multiplier
  - Boss appears after 18 enemy kills per stage (`KILLS_PER_STAGE`)

- **Boss Attack Patterns** (line 1372-1409):
  - Each boss has 3 distinct attack patterns (radial, aimed, homing, columns, crosses, etc.)
  - Patterns are queued and cycle; telegraph phase before each attack
  - Boss movement: bounces left/right at y=120

#### 8. Weapon System
- **Vulcan** (default): Rapid-fire bullets, 3 levels
  - Level 1: Single shot
  - Level 2: Dual shot
  - Level 3: Triple spread
- **Laser**: Piercing beam weapon, 3 levels, same level progression
- Weapons switch on level 4 pickup; can toggle between them

#### 9. UI & Feedback
- **HUD** (always visible): Score, best score, multiplier, weapon, hearts, bombs
- **Boss bar** (appears when boss active): Shows boss HP as percentage
- **Overlays**: Game start screen, pause screen, game over screen with new high score
- **Visual feedback**: Combo popups (kill counts), stage banners, flash effects (white screen on hit), screen shake
- **Accessibility**: Screen reader announcements via ARIA live region (`srLive`)

#### 10. Adaptive Quality (line 473-483)
- Tracks frame times (45-frame rolling average)
- If avg > 25ms (40 FPS), increase quality level (reduce visual effects)
- If avg < ~17.9ms (56 FPS), decrease quality level (add effects like shadow blur)
- Quality setting disables/enables shadow blur and glow effects during rendering

#### 11. Theme System (line 493-500)
- 6 unique sky gradients (backgrounds)
- Associated cloud colors and sun colors
- Themed stage names (Daylight Front, Sunset Corridor, etc.)
- Cycles per stage: `(stage - 1) % THEMES.length`

## Key Constants & Configuration

| Constant | Value | Purpose |
|----------|-------|---------|
| `BASE_W`, `BASE_H` | 480, 720 | Canvas logical resolution |
| `KILLS_PER_STAGE` | 18 | Enemy kills required before boss spawns |
| `STORAGE_*` | localStorage keys | Persistent state (best score, mute, difficulty, reduced motion) |

## How to Run

1. Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
2. Click "Start Game"
3. Use keyboard/mouse/touch controls as indicated

No build step, no dependencies, no local server required.

## Common Development Tasks

### Adding a New Boss
1. Add entry to `BOSSES` array (line 508-515) with design colors and HP multiplier
2. Add corresponding attack pattern array to `BOSS_PATTERNS` (line 1372-1409)
3. Each pattern is a function that calls firing helper functions like `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
4. Boss is spawned by `spawnBoss()` after stage kill threshold; uses `BOSSES[stage % BOSSES.length]`

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502-506) with sprite size, HP, speed multiplier, score, minimum stage
2. Enemies use `drawPlane()` for rendering (shared with player and boss)
3. Adjust enemy-specific behavior in `update()` (e.g., weave pattern, shooting behavior)

### Modifying Difficulty Balance
1. Edit `DIFFICULTIES` object (line 485-489) to adjust spawn rates, speeds, HP pools
2. Test by cycling difficulty on start screen and observing spawn frequency/difficulty
3. `spawnBase` and `spawnMin` control spawn timer; lower = faster spawns
4. `bossHpMul` scales boss health

### Tweaking Visual Effects
- **Screen shake**: `screenShake(magnitude, duration)` function (line 727); adjust `shakeMag` calculation
- **Flash effects**: `flashHit()` and `flashBomb()` toggle CSS classes on `flashEl`
- **Particle explosions**: `spawnExplosion(x, y, color, count)` (not shown in excerpt)
- **Quality/glow**: Controlled by adaptive quality system; high quality disables shadow blur

### Adding Sound Effects
1. Add new function to `sfx` object (line 624-676) using `beep()` or `noiseBurst()`
2. `beep(freq, duration, type, volume, opts)` - oscillator-based sound
3. `noiseBurst(duration, volume, opts)` - noise-based impact
4. Options include: `sweepTo` (frequency ramp), `filterType`/`filterFreq`, `pan`, `delay`
5. Call the SFX when appropriate (e.g., on collision, powerup collected)

### Performance Tuning
- Game automatically adapts quality based on frame time
- Quality level 0 (full): Shows shadow blur and glow on bullets/particles
- Quality levels 1-2: Reduce visual effects for low-end devices
- Modify thresholds in `trackPerf()` function if needed

## Stage Progression

1. Game spawns enemies continuously with accelerating spawn rate (difficulty ramps every 5 seconds)
2. After 18 enemy kills, boss telegraph appears
3. Boss enters from top (entering animation), then attacks from y=120
4. After boss defeat: stage counter increments, theme changes, new enemies with potentially harder properties spawn
5. Process repeats indefinitely until player loses all lives

## localStorage Keys

| Key | Value | Type |
|-----|-------|------|
| `skyfighter.bestScore` | Best score | Number |
| `skyfighter.muted` | Audio mute state | '0' or '1' |
| `skyfighter.difficulty` | Selected difficulty | 'easy', 'normal', 'hard' |
| `skyfighter.reducedMotion` | Accessibility setting | '0' or '1' |

## Performance Notes

- Uses adaptive quality system to maintain ~60 FPS on variable hardware
- Single canvas rendering (no off-screen canvases for now)
- Dynamic array filtering for bullets/enemies/particles (splice in reverse loop to avoid index issues)
- Frame-based timing (not time-based) for some patterns to ensure determinism
- No game state serialization; progress resets on refresh
