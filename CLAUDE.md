# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application. It's a fully self-contained web game with no external dependencies or build process required.

## Quick Start

### Development
The game runs directly in a browser—no build step needed:
```bash
# Open index.html in any web browser
# Development: Make changes to index.html and refresh the browser
```

### Testing Changes
- Changes are live-reload-friendly: edit HTML, refresh the browser
- Use browser DevTools Console to debug JavaScript errors
- Test across desktop (keyboard/mouse) and mobile (touch) by resizing the browser or using device emulation

## Architecture

### Code Organization (all in index.html)

The file is structured in three parts:

1. **HTML + CSS** (lines 1–359)
   - Canvas element and HUD (heads-up display)
   - Overlay screens (start, pause, game over)
   - Touch controls for mobile (joystick + action buttons)
   - Button controls (mute, motion, pause)
   - CSS variables and responsive sizing using `clamp()` for scaling across devices

2. **JavaScript Game Engine** (lines 422–1666)
   - **State Management**: game variables (player, enemies, bullets, score, etc.)
   - **Input Handling**: keyboard (WASD/Arrows), mouse steering, touch joystick
   - **Rendering Loop**: `loop()` function drives `update()` then `draw()` at 60 FPS
   - **Game Logic**: collision detection, scoring, difficulty ramping

### Key Subsystems

#### 1. **Player & Movement**
- `player` object: position (x, y), dimensions, weapon, level, cooldown, invulnerability
- Input sources: keyboard arrows/WASD, mouse hover (glide toward pointer), touch joystick
- Keyboard always takes priority over mouse to prevent input fighting

#### 2. **Weapons System**
- Two weapon types: "vulcan" (spread bullets) and "laser" (piercing beams)
- Each has 3 levels (wider spread/more lanes at higher levels)
- Power-ups cycle through levels; at level 3, toggle to the other weapon
- Fire rate: laser 0.1s, vulcan 0.15s (controlled by `player.cooldown`)

#### 3. **Enemies**
- Three types defined in `ENEMY_TYPES`: grunt, interceptor, tank
  - Each has unique speed, HP, color, and score value
  - Tanks appear only at stage 2+
- Two fire patterns: "spread" (5-shot fan) and "aimed" (homing shot toward player)
- Weaving interceptors use sinusoidal movement for evasion
- Enemies spawn via `spawnEnemy()` with difficulty-based spawn rates

#### 4. **Boss Battles**
- Spawn after killing 18 enemies per stage (`KILLS_PER_STAGE`)
- Six boss designs in `BOSSES` array, cycling through stages
- Each boss has unique attack patterns (`BOSS_PATTERNS`): radial bursts, columns, homing, spirals, etc.
- Boss telegraph phase (22 frames) gives player reaction time before attack fires
- Damage: regular bullets deal 3 HP, bomb deals 24 HP

#### 5. **Scoring & Combo System**
- Base score per kill varies by enemy type
- **Multiplier**: increases by 1 for every 5 consecutive kills (max 10x)
- **Chain Timer**: 2.2 seconds—breaks if no enemy is killed within this window
- Combo pop animation shows at every 10 kills
- Boss kill awards 500 base points

#### 6. **Particle System**
- Explosions created by `spawnExplosion()` with random velocity particles
- Particles fade out over their lifetime; quality setting scales count
- Used for enemy death, player hit, bomb blast visual feedback

#### 7. **Audio (WebAudio Synth)**
- No external audio files—all sounds synthesized in real-time
- `ensureAudio()` initializes AudioContext on first user interaction
- Sound effects: shoot, laser, explosions (by type), hit, bomb, power-up, boss telegraph/death, stage clear, UI click, game over
- Master gain at 0.9, compressor for dynamic range control
- Pan support (left/right channels) for spatial audio feedback

#### 8. **Difficulty Scaling**
- Three levels: easy, normal, hard
- Control spawn rates, enemy speed, enemy fire chance, starting lives, bomb count, boss HP multiplier
- Stored in localStorage, selectable from start screen
- Optional adaptive quality system: tracks frame times and reduces particle/shadow quality under load

#### 9. **Mobile & Accessibility**
- **Touch Controls**: Joystick (left 55% of screen), Fire/Bomb buttons (right 45%)
- **Keyboard**: Arrows/WASD move, Space/X/Shift fire/bomb, P/Esc pause
- **Mouse**: Move to aim, click to fire (auto-reload on mouseup)
- **Accessibility**: Screen reader announcements via `#srLive` ARIA live region; reduced-motion support
- **Responsive**: Canvas scales with `clamp()` for any viewport; safe-area insets for notches

### Game Loop Flow

```
loop(timestamp)
  ├─ trackPerf(dt)          // Monitor frame time for quality scaling
  ├─ update(dt)             // Physics, spawn, collision, scoring
  │  ├─ Player movement (keyboard/mouse/joystick priority)
  │  ├─ Weapon firing & cooldown
  │  ├─ Enemy spawning & movement
  │  ├─ Bullet movement
  │  ├─ Collision detection (player-enemy, bullet-enemy, bullet-boss, powerup-player)
  │  ├─ Boss AI & attack patterns
  │  └─ Particle updates
  └─ draw()                 // Canvas rendering
     ├─ Sky gradient & clouds (theme-based)
     ├─ Player & enemies
     ├─ Bullets & explosions
     ├─ HUD text (score, lives, weapon, boss bar)
     └─ Post-processing (screen shake, vignette)
```

### State Variables (Key Ones)

| Variable | Purpose |
|----------|---------|
| `player` | Player object with position, weapon, level, invulnerability |
| `enemies` | Array of active enemy objects |
| `boss` | Current boss object (null if no boss) |
| `bullets` | Player bullets |
| `enemyBullets` | Enemy projectiles |
| `particles` | Explosion particles |
| `powerUps` | Collectible power-ups |
| `score`, `multiplier`, `chain` | Scoring system |
| `lives`, `maxLives` | Health system |
| `stage`, `stageKills`, `kills` | Progression |
| `running`, `paused`, `gameOver` | Game states |

## Common Tasks

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (define w, h, hp, speedMul, color, dark, score, minStage, weave)
2. Add sound effect to `sfx` object if needed (e.g., `sfx.explodeCustom`)
3. Add key to `EXPLODE_SFX` map to map enemy type to SFX key
4. Update `spawnEnemy()` weights if desired (e.g., change `r < 0.4` thresholds)

### Add a New Boss Attack Pattern
1. Add pattern function to `BOSS_PATTERNS[bosIndex]` array
2. Use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`
3. Pattern receives boss object `b`, fires into `enemyBullets` array
4. Control speed (typically 130–300) and bullet count

### Add a New Stage Theme
1. Add object to `THEMES` array with `{ sky: [top, mid, bottom], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }`
2. Themes cycle by `(stage - 1) % THEMES.length` in `draw()`
3. Sky colors are CSS color strings; cloud/sun are RGB triplets (no '#')

### Adjust Difficulty Parameters
Edit `DIFFICULTIES` object:
- `spawnBase` / `spawnMin`: control spawn rate (lower = faster)
- `enemySpeed`: multiply enemy velocity
- `enemyShootChance`: probability enemy fires (0–1)
- `lives` / `bombs`: starting resources
- `bossHpMul`: scale boss health

### Add/Change Audio
1. Add function to `sfx` object (use `beep()` for tones, `noiseBurst()` for noise)
2. `beep(freq, duration, type, volume, opts)`: type is 'sine'/'square'/'sawtooth'/'triangle'
3. `opts` can include: `sweepTo`, `filterType`, `filterFreq`, `pan`, `delay`
4. Call `ensureAudio()` before playing to initialize Web Audio

## Performance Considerations

- **Adaptive Quality**: Tracks frame times; if FPS drops below ~40, reduces particle count and shadow blur
- **Collision**: O(n²) enemy-bullet checks; acceptable for ~30 enemies + ~10 bullets
- **Rendering**: Canvas clears every frame; particles and explosions scale with quality level
- **Audio**: Synth is CPU-efficient; mutable via UI

## Key Constants & Formulas

| Constant | Value | Purpose |
|----------|-------|---------|
| `KILLS_PER_STAGE` | 18 | Enemies to kill before boss |
| `BASE_W` / `BASE_H` | 480 / 720 | Canvas native resolution |
| `chainTimer` | 2.2s | Combo timeout |
| Multiplier | 1 + floor(chain / 5) | Score multiplier (max 10x) |
| Boss HP | 70 + stage × 35 | Base health before multipliers |

## Browser Support

- **Modern browsers**: Chrome, Firefox, Safari, Edge
- **Features used**: Canvas 2D, Web Audio API, localStorage, Pointer/Touch events, requestAnimationFrame
- **Mobile**: Full touch support; test with device emulation in DevTools

## Notes for Future Work

- All state is ephemeral (no server)—no persistence except localStorage for best score, mute, difficulty
- Accessibility: Screen reader announcements in `announce()` function; reduced-motion respected
- The game is intentionally self-contained (no build, no dependencies) for simplicity and instant play
