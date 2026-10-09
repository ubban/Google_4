# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built entirely in HTML5. The entire game (HTML, CSS, JavaScript) is contained in a single `index.html` file (1669 lines).

The game features:
- Canvas-based 2D rendering
- Web Audio API synthesized sound effects (no external assets)
- Multiple difficulty levels and visual themes
- Touch and keyboard controls
- Boss encounters and stage progression
- Score multiplier system and power-ups
- Adaptive performance quality scaling

## Project Structure

- **index.html** — Single-file game containing all HTML, CSS, and JavaScript

No build tools, external dependencies, or tests. The game runs directly in a browser by opening `index.html`.

## Running the Game

To run locally:
```bash
# Option 1: Simple HTTP server (Python 3)
python3 -m http.server 8000

# Option 2: Node.js
npx http-server

# Option 3: Any static file server
# Then open http://localhost:PORT/index.html
```

Open `index.html` directly in a browser (file:// protocol works).

## Game Architecture

### Architecture Overview

The game uses a **game loop** driven by `requestAnimationFrame` that updates game state and renders each frame. Key systems:

1. **Input Handling** (lines ~768–862)
   - Keyboard (ArrowKeys, WASD, Space, X, Shift for bomb)
   - Mouse steering and click-to-fire
   - Touch/joystick for mobile (left zone = movement, right zone = fire/bomb buttons)
   - Multiplexed input: keyboard/joystick take priority over mouse

2. **Audio System** (lines ~542–676)
   - Web Audio API OscillatorNodes and BiquadFilters create all sounds
   - `beep()` and `noiseBurst()` are the synthesis primitives
   - Spatial audio: `panFor(x)` positions explosions left/right based on screen position
   - All SFX memoized in `sfx` object for easy lookup
   - Master gain node for mute control

3. **Game State** (lines ~863–915)
   - Player, bullets, enemies, enemyBullets, particles, powerUps, boss arrays
   - Score, lives, stage, kill counters, weapon state, combo chain
   - Persistent: best score, mute state, difficulty, reduced-motion preference (localStorage)

4. **Entity System** (no classes; arrays of objects)
   - **Player** — position, velocity, weapon, cooldown, invuln timer
   - **Enemies** — type (grunt/interceptor/tank), position, velocity, hp, shoot cooldown
   - **Bullets** — position, velocity, damage, owner (player or enemy)
   - **Particles** — position, velocity, lifetime, visual (trail/spark/etc)
   - **PowerUps** — position, velocity, type (weapon upgrade or bomb), collision size

5. **Boss System**
   - Boss spawns after KILLS_PER_STAGE (18) kills
   - Each stage has a different boss from BOSSES array (6 total)
   - Boss health scales with difficulty and by its hpMul
   - Boss displays health bar and name
   - Boss shoots 4 bullet patterns in sequence

6. **Difficulty Scaling** (DIFFICULTIES object, line ~485)
   - enemy spawn frequency, speed, shoot chance
   - Lives and bomb counts
   - Boss health multiplier
   - Settings are applied at game start; changing difficulty while playing queues for next game

7. **Combo/Multiplier System**
   - Kill consecutive enemies without taking damage → chain increases
   - Multiplier is 1 + floor(chain / 3), capped by difficulty
   - Chain resets on hit; chainTimer extends the reset window slightly
   - showComboPop() animates the display

### Rendering Pipeline (lines ~920–1350+)

1. **Clear & background** — gradient sky, layered clouds
2. **Draw entities** — bullets, enemies, particles, player, boss
3. **HUD overlay** — score, weapon level, lives, bombs (updated via DOM)
4. **Boss bar** — health bar + name (shown when boss active)
5. **Flash/vignette** — screen effects (hit vignette, bomb flash)
6. **Touch controls** — joystick and buttons on mobile

### Update Loop (rough structure; see around line ~1550)

Each frame:
1. Handle keyboard/joystick/mouse input → set player velocity
2. Update player (cooldown, invuln, position, collision)
3. Spawn enemies (based on spawn timer and stage)
4. Update all entities (position, cooldown, lifetime)
5. Collision detection (player ↔ enemies, bullets ↔ enemies, etc.)
6. Handle kills, powerups, combo, stage advance
7. Update audio, HUD, effects
8. Render frame

## Key Variables & Constants

| Constant | Purpose |
|----------|---------|
| `BASE_W = 480, BASE_H = 720` | Game canvas size (fixed logical size) |
| `KILLS_PER_STAGE = 18` | Enemy kills required to spawn boss and advance stage |
| `DIFFICULTIES` | Difficulty presets (easy/normal/hard) |
| `ENEMY_TYPES` | grunt, interceptor, tank — stats, colors, scoring |
| `BOSSES` | 6 boss types with color schemes and health multipliers |
| `THEMES` | 6 sky gradients, cloud colors, sun colors |
| `EXPLODE_SFX` | Map enemy type → explosion sound effect |

## Modifying the Game

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` object (line ~502):
   ```javascript
   sharpshooter: { w: 32, h: 32, hp: 2, speedMul: 0.9, color: '#2196f3', dark: '#0d47a1', score: 25, minStage: 4, ... }
   ```
   - `w, h` — visual size
   - `hp` — health (bullets needed to kill)
   - `speedMul` — relative speed vs base enemy speed
   - `color, dark` — hull colors for drawing
   - `score` — points on kill
   - `minStage` — first stage it appears
   - `weave` — optional; true = enemy weaves side-to-side while descending

2. Add explosion SFX to `EXPLODE_SFX` map (line ~678) and define in `sfx` object if unique.

3. Add drawing code in render loop (find the loop that draws enemies, around line ~1100).

### Adjusting Difficulty Parameters

Edit the `DIFFICULTIES` object (line ~485):
- `spawnBase` — base frames between spawns
- `spawnMin` — minimum spawn interval (as difficulty increases)
- `enemySpeed` — multiplier on all enemy velocities
- `enemyShootChance` — probability enemy fires each frame
- `lives, bombs` — starting player lives and bombs
- `bossHpMul` — boss health multiplier

### Adding a New Boss

1. Add to `BOSSES` array (line ~508):
   ```javascript
   { name: 'CUSTOM', hullTop: '#rrggbb', hullBottom: '...', wing: '...', core: '...', hpMul: 1.2 }
   ```

2. The boss drawing code uses these colors; no additional code needed unless you want custom attack patterns.

### Changing Audio Pitch/Duration

Sounds are in the `sfx` object (line ~624). Each uses `beep(freq, duration, type, volume, opts)` or `noiseBurst(duration, volume, opts)`.

- `beep(freq, dur, type, vol, opts)` — type = 'sine'|'square'|'sawtooth'|'triangle'
  - `opts.sweepTo` — target freq for exponential ramp
  - `opts.filterFreq, filterType` — lowpass|highpass|bandpass filter
  - `opts.pan` — left/right panning (-1 to 1)
- `noiseBurst(dur, vol, opts)` — white noise with optional filter

Example: make the shoot sound higher-pitched:
```javascript
shoot: () => beep(1100, 0.07, 'square', 0.045, { sweepTo: 700, filterFreq: 2600 })
```

### Adding a New Visual Theme

1. Add entry to `THEMES` array (line ~493):
   ```javascript
   { sky: ['#start', '#mid', '#end'], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }
   ```
   - `sky` — 3 colors for gradient (top to bottom)
   - `cloud, sun` — RGB values as comma-separated strings

### Modifying the Game Mechanic (Boss Health, Bomb Damage, Etc.)

Search for these constants in the code:
- **Boss HP** — `boss.hp = boss.maxHp = ...` (around line ~1150)
- **Bomb radius/damage** — look for bomb explosion handling (line ~1200+)
- **Weapon damage** — bullet damage is per-weapon in spawn code
- **Combo multiplier cap** — search for `multiplier =` (around line ~1300)

## Testing & Debugging

No automated tests. To test changes:

1. Open DevTools (F12)
2. Open index.html locally (see Running the Game section)
3. Check Console for errors
4. Use localStorage commands to reset state:
   ```javascript
   localStorage.clear()  // Reset all game data
   ```

### Performance Monitoring

The adaptive quality system (line ~473) tracks frame time and reduces quality if FPS drops. Check `quality` variable (0=full, 1=medium, 2=low) in DevTools console during gameplay.

## Known Limitations & Future Work

- Single file limits code organization at scale
- No persistent data beyond localStorage (scores, replays)
- Boss patterns are predetermined; no algorithmic challenge scaling
- Mobile controls are touch-only (no hardware gamepad support)

## Standards & Style

- **Strict mode** — file runs in `'use strict'`
- **No external dependencies** — vanilla HTML5
- **No comments** — code is self-documenting; inline SFX definitions are the exception (synthesized audio is complex)
- **Canvas scaling** — logical size is 480×720; rendered size scales to fit viewport
- **Accessibility** — screen reader announcements for game events, reduced-motion support
