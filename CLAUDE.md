# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-file HTML5 game. The entire game—logic, rendering, audio synthesis, and UI—is contained in `index.html` with no external dependencies.

## Running the Game

**Local Development:**
```bash
# Simple HTTP server (any of these work)
python3 -m http.server 8000
# or
npx http-server
# or
php -S localhost:8000
```
Then open `http://localhost:8000` in a browser.

**Production:**
Deploy `index.html` to any static web host (GitHub Pages, Netlify, Vercel, etc.).

## Architecture Overview

The game follows a classic game loop pattern: update → render → repeat via `requestAnimationFrame`. Everything is scoped in a single IIFE (Immediately Invoked Function Expression) to avoid global pollution.

### Core Systems

1. **Game Loop** (line 1640–1653)
   - `loop(now)` runs every frame via `requestAnimationFrame`
   - Calculates delta time, calls `update(dt)`, then `draw()`
   - Performance tracking adapts rendering quality if frame rate drops

2. **Input Handling** (line 768–861)
   - **Keyboard:** Arrow keys, WASD for movement; Space/click to fire; X/Shift for bombs; P/Esc to pause
   - **Mouse:** Player glides toward cursor; click to fire
   - **Touch:** Left side = joystick movement; right side = FIRE and BOMB buttons
   - Input is normalized so keyboard/joystick take priority over mouse

3. **Physics & Collision** (line 1148–1302, 1068–1071)
   - `update(dt)` handles all object movement and collision detection
   - `rectsOverlap(a, b)` checks axis-aligned bounding box collision
   - Player stays within canvas bounds; bullets/enemies auto-remove when off-screen

4. **Rendering** (line 1520–1638)
   - `draw()` renders in this order: sky gradient + sun glow → clouds → player → bullets → enemies → boss → power-ups → particles
   - Uses canvas 2D API; adaptive quality reduces particle/shadow effects under load
   - Screen shake applied via canvas translation when taking damage

5. **Audio** (line 542–676)
   - Web Audio API synthesizes all sound effects (no audio files needed)
   - `beep(freq, dur, type, vol, opts)` generates pitched tones with optional sweep/filter/pan
   - `noiseBurst(dur, vol, opts)` generates noise bursts for explosions
   - Master gain and compressor for volume control; muted state stored in localStorage

6. **Game State** (line 863–912)
   - Player, enemies, bullets (player + enemy), particles, power-ups, and boss all tracked in arrays
   - Stage counter increments after each boss; kill count triggers boss spawn
   - Chain timer for combo multiplier (expires after 2.2 seconds of no kills)

### Key Game Values & Configs

**Difficulty Settings** (line 485–489)
- `DIFFICULTIES[difficulty]` controls spawn rates, enemy speed/accuracy, lives, bomb count, boss HP
- Stored in localStorage; user selects before game starts

**Enemy Types** (line 502–506)
- `grunt`, `interceptor` (fast, weaving), `tank` (slow, durable)
- Each has color, dimensions, HP, score value, min stage to appear

**Boss Patterns** (line 1372–1409)
- 6 bosses with distinct attack patterns using helper functions: `fireRadial()`, `fireColumns()`, `fireSpiralBurst()`, etc.
- Boss selects one pattern per telegraph phase; each has unique HP multiplier and design colors

**Themes** (line 493–500)
- 6 visual themes (Daylight Front, Sunset Corridor, Night Approach, etc.) that cycle per stage
- Each theme specifies sky gradient colors, cloud color, and sun color

**Weapon System** (line 871–1146)
- Two weapons: Vulcan (spread, fast) and Laser (pierce, slower)
- Weapon levels 1–3 increase bullet count; level 3+ switches weapons instead of upgrading
- Fire rates: Vulcan 0.15s, Laser 0.1s per shot

### Important Constants

- `BASE_W = 480, BASE_H = 720` — canvas resolution (scaled to fit window, max 1.6×)
- `KILLS_PER_STAGE = 18` — regular enemies killed before boss spawns
- Chain multiplier: `1 + min(9, floor(chain / 5))` capped at 10× for consecutive kills

### localStorage Keys

- `skyfighter.bestScore` — high score
- `skyfighter.muted` — mute state (0/1)
- `skyfighter.difficulty` — last difficulty chosen
- `skyfighter.reducedMotion` — reduced motion preference (respects system setting)

## Modifying the Game

**Add a New Enemy Type:**
1. Add entry to `ENEMY_TYPES` (line 502–506) with dimensions, HP, color, score, minStage
2. Update `spawnEnemy()` (line 1022–1041) probability checks if needed
3. Add explosion SFX mapping to `EXPLODE_SFX` if custom sound needed (line 678)

**Add a New Boss:**
1. Add entry to `BOSSES` (line 508–515) with colors and hpMul
2. Add attack pattern array to `BOSS_PATTERNS` (line 1372–1409)
3. Boss is selected by `(stage - 1) % BOSSES.length` so it cycles

**Adjust Difficulty/Balance:**
- Edit `DIFFICULTIES` (line 485–489) to tweak spawn rates or enemy behavior
- Adjust `fireWeapon()` (line 1124–1146) spread/count for player bullets
- Change `addScore(base)` multiplier calculation (line 1087–1094) to alter score scaling

**Add a Visual Theme:**
1. Add object to `THEMES` array (line 493–500) with sky gradient and cloud colors
2. Theme auto-cycles by stage: `(stage - 1) % THEMES.length`

**Modify Audio:**
- Adjust `beep()` frequency, duration, sweep, or filter in each `sfx.*` function (line 624–676)
- Change `panFor(x)` scaling (line 562–564) for stereo effects
- Adjust `masterGain.gain.value` (line 556) for overall volume

## Performance Notes

- Quality adapts automatically: `quality` ranges from 0 (full) to 2 (low)
- Frame times tracked; if average FPS drops below ~40, quality downgrades
- Low quality disables: particle count reduction, shadow blurs, glow effects
- Reduced motion mode (system preference or user toggle) disables animations and screen shake

## Accessibility

- Screen reader announcements via `srLive` element for stage transitions, combo counts, game over state
- Reduced motion toggle accessible via UI button; respects system `prefers-reduced-motion`
- Keyboard controls available for all actions
- Touch controls scaled for mobile with safe-area inset support

## Notes for Future Development

- The entire game state is procedural/deterministic (no random seeding), so replays would require recording input
- Adding music would require either web audio synthesis (complex) or an audio file asset
- Mobile performance is adaptive; consider keeping particle/effect counts reasonable
- Canvas 2D rendering; WebGL migration would be needed for significantly more complexity
