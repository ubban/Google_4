# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single self-contained HTML5 file. The entire game runs in the browser with no external assets, build tools, or dependencies — just vanilla JavaScript, Canvas rendering, and Web Audio API synthesis.

## Architecture

The game is organized as a single IIFE (Immediately Invoked Function Expression) within `index.html` with these conceptual layers:

### 1. **Configuration & Constants** (lines 473–523)
- `DIFFICULTIES`: Three difficulty presets (easy/normal/hard) controlling spawn rates, enemy speed, lives, etc.
- `THEMES`: Six colorful sky themes that cycle per stage
- `ENEMY_TYPES`: Three enemy variants (grunt, interceptor, tank) with stats like HP, speed, score value, and appearance
- `BOSSES`: Six boss designs with distinct visual colors and attack patterns
- `KILLS_PER_STAGE`: Number of enemies to defeat before a boss spawns (18)

### 2. **Web Audio Synthesis** (lines 542–676)
A custom synth with no external audio files. The `beep()` function creates tonal sounds via oscillators with optional frequency sweeps and filters. The `noiseBurst()` function synthesizes noise bursts. The `sfx` object maps game events to audio cues (shoot, explode, boss hit, etc.). Audio is muted if the master `muted` flag is set or if the audio context is unavailable.

**Key insight:** Audio is fully procedural — all sounds are generated on-the-fly using Web Audio oscillators and filters.

### 3. **Input Handling** (lines 767–861)
- **Keyboard input** stored in a `keys` object by event code
- **Touch joystick** on the left side of the screen; detected via `isTouchDevice`
- **Mouse input** provides aim point that the player glides toward; `mouseActive` tracks whether mouse is over the canvas
- **Button input** for firing and bombs on touch; firing is also controlled by Space key or mouse button
- Priority system: keyboard/joystick input takes priority over mouse when active

### 4. **Game State** (lines 863–911)
Main game objects:
- `player`: Position, weapon, level, invulnerability timer, cooldown
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Entity arrays
- `boss`: Active boss object (only one at a time)
- Scalars: `score`, `lives`, `stage`, `stageKills`, `chain`, `multiplier`, `frame`, etc.

State is reset by `resetGame()` when the game starts.

### 5. **Update Loop** (lines 1148–1302)
The `update(dt)` function runs every frame and handles:
- Player movement (keyboard priority > joystick > mouse)
- Player shooting (with weapon-specific behavior)
- Enemy spawning and movement (with weaving for interceptors)
- Collision detection between bullets and enemies, player and hazards, etc.
- Chain/multiplier decay (breaks when the player takes damage)
- Boss updates via `updateBoss(dt)`

The main game loop (`loop(now)`) calls `update()` and `draw()` via `requestAnimationFrame`.

### 6. **Rendering** (lines 1520–1637)
The `draw()` function renders:
- Sky gradient and sun glow (theme-based)
- Parallax clouds at two depths
- Player plane with flame when moving
- Bullets and enemy bullets
- Enemies and boss
- Power-ups (rotating squares)
- Particle explosions
- Screen shake effect applied via canvas translate

`drawPlane(x, y, w, h, color, darkColor, flame)` and `drawBossShip(b)` are specialized renderers for aircraft.

### 7. **Game Logic**
- **Scoring**: Killing enemies adds score via `addScore()`, which increments the kill chain and multiplier
- **Weapons**: Vulcan (spread shots) and Laser (piercing beams); switched by power-ups, with 3 levels per weapon
- **Boss Patterns**: Six boss designs each have three attack patterns (radial bursts, aimed spreads, etc.) — see `BOSS_PATTERNS`
- **Difficulty Scaling**: Enemy spawn rate increases over time; boss HP scales by difficulty preset

## Key Code Sections to Know

| Section | Lines | Purpose |
|---------|-------|---------|
| Difficulty/theme/enemy data | 485–515 | Game balance configuration |
| Audio synth | 542–676 | Sound generation |
| Input handling | 767–861 | Player control systems |
| Update logic | 1148–1302 | Physics and collision |
| Boss behavior | 1411–1454 | Boss movement and attack patterns |
| Drawing | 1520–1637 | Canvas rendering |
| Boss patterns array | 1372–1409 | Attack pattern definitions |

## Common Development Tasks

### Tweaking Difficulty
Edit `DIFFICULTIES` object. Parameters:
- `spawnBase`: Initial spawn timer (lower = faster enemies)
- `spawnMin`: Minimum spawn timer as frame count increases
- `enemySpeed`: Multiplier on enemy speed
- `enemyShootChance`: Probability (0–1) that an enemy shoots
- `lives`, `bombs`: Starting counts
- `bossHpMul`: Boss health multiplier

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, and optional `weave`
2. Update the `spawnEnemy()` function to include the new type in the probability logic
3. Add explosion sprite handler in `EXPLODE_SFX` (e.g., `'newtype': 'explodeNewtype'`)
4. Define the corresponding explosion sound in `sfx` object if needed

### Adding a New Boss Pattern
1. Create a pattern function (e.g., `fireRadial(b, count, speed)`) that pushes enemy bullets
2. Add the function to `BOSS_PATTERNS[bossIndex]` array as a third attack pattern (arrays have 3 patterns each)
3. Boss index corresponds to `BOSSES` array index

### Adjusting Game Balance
- **Enemy spawning**: Modify `spawnTimer` calculation in `update()`
- **Weapon damage**: Edit hard-coded damage in collision logic (e.g., line 1259: `boss.hp -= 3`)
- **Power-up frequency**: Adjust probabilities in `maybeSpawnPowerUp()` (line 1058–1067)
- **Chain multiplier**: Adjust `multiplier` calculation in `addScore()` (line 1090)

### Adding a New Sound Effect
1. Define it in the `sfx` object using `beep()` or `noiseBurst()`
2. Call it at the appropriate game event (e.g., `sfx.mySound()`)
3. Audio only plays if `!muted` and `audioCtx` is initialized

### Changing Visual Style
- **Colors**: Theme colors are in `THEMES` (sky, cloud, sun), enemy colors in `ENEMY_TYPES`, boss colors in `BOSSES`
- **Screen shake**: Adjust magnitude/duration in `screenShake()` calls
- **Particle effects**: Tune spawn count in `spawnExplosion()` based on quality mode

## Performance Considerations

The game includes **adaptive performance** (lines 473–482). A rolling average of frame times is tracked; if average frame time exceeds 1/40ms (25 fps), `quality` is downgraded (less particles, less glow/shadows). This ensures the game remains playable on lower-end devices.

- **Quality mode 0**: Full effects (glow, shadows, all particles)
- **Quality mode 1**: Reduced particle count (60% of original)
- **Quality mode 2**: Heavy reduction (35% of original particles, no shadow/glow)

When modifying particle counts or rendering effects, check the `quality` variable.

## LocalStorage Keys

The game persists player preferences using these keys:
- `skyfighter.bestScore`: Best score ever
- `skyfighter.muted`: Audio mute state (1 or 0)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced motion preference (1 or 0)

## Browser Compatibility Notes

- **Web Audio API**: Required for sound; gracefully degrades if unavailable
- **Touch support**: Detected via `ontouchstart` or `navigator.maxTouchPoints`; enables virtual joystick and buttons
- **Prefers-reduced-motion**: Respects system motion preferences for accessibility; can be toggled in-game
- **Canvas**: Standard 2D context; no WebGL
- **Responsive**: Canvas scales to fit window; uses `clamp()` CSS for responsive UI sizing

## Debugging Tips

- **Frame counter**: The global `frame` variable increments every update; useful for timing effects
- **Quality mode**: Check `quality` to understand performance state
- **Game state**: `running`, `paused`, `gameOver` flags control overlay and update logic
- **Screen shake**: `shakeTime` and `shakeMag` track active screen shake; useful for verifying impact feedback
- **Chain system**: `chain` (kill count) and `multiplier` (1 + floor(chain/5)) drive score scaling

## Testing in Browser

Simply open `index.html` in a modern browser. The game:
1. Shows the title overlay with difficulty selection
2. Persists settings to localStorage
3. Works on mobile with touch controls (drag left to move, tap buttons to fire/bomb)
4. Works on desktop with mouse aim and keyboard/mouse input
5. Auto-adjusts performance based on frame rate

No build step needed.
