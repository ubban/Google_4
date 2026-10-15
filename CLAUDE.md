# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-'em-up web game. The entire game (1669 lines) is contained in `index.html` with embedded CSS and vanilla JavaScript using the Canvas API for rendering.

## Architecture

### Single-File Structure

All code is in `index.html`:
- Lines 1-359: HTML markup and CSS styles
- Lines 422-1669: JavaScript game logic (wrapped in IIFE for scope isolation)

**Key architectural constraint:** This is deliberately a monolithic design for portability—the entire game is a self-contained unit that can be opened directly in any browser without external dependencies or build processes.

### Game Systems

#### 1. **Game Loop & Timing** (lines 1640–1665)
- `requestAnimationFrame(loop)` drives the game at browser refresh rate
- Delta time (`dt`) passed to `update()` ensures frame-rate independent physics
- Performance tracking: `trackPerf()` monitors frame times and adjusts quality (full → medium → low)
- Three game states: `running`, `paused`, `gameOver`

#### 2. **Input Handling** (lines 767–861)
- Keyboard: Arrow keys + WASD for movement, Space/click to fire, X/Shift for bomb, P/Esc to pause
- Mouse: Player glides toward cursor on desktop
- Touch: Two-zone interface—left 55% for joystick movement (drag), right 45% for FIRE/BOMB buttons
- Input priority: Keyboard/joystick > mouse (mouse is a fallback for desktop steering)

#### 3. **Player Ship** (lines 874–1172)
- Entity object with position, velocity, dimensions, cooldown, weapon state, level, invulnerability timer
- Movement: keyboard/joystick input with `player.speed = 260` px/s; mouse input smoothly glides toward target
- Weapons: **Vulcan** (spread, rapid-fire) vs **Laser** (pierce enemies, fewer lanes)—weapon levels 1–3
- Firing: Cooldown controlled by `fireDelay` (laser 0.1s, vulcan 0.15s)

#### 4. **Enemies & Spawning** (lines 1022–1040)
- Three enemy types in `ENEMY_TYPES`: **grunt** (fast, weak), **interceptor** (weaver, evasive), **tank** (slow, durable)
- Each has unique color, dark variant (for shadow), speed, HP, score value, and `minStage` unlock
- Enemies spawn via `spawnEnemy()` with adaptive timing based on difficulty
- **Weaving:** Interceptors follow a sine-wave pattern `baseX + sin(...) * 60`
- Enemy bullets: spread pattern (fixed angles) or aimed pattern (toward player)

#### 5. **Boss System** (lines 1043–1439)
- Six boss designs (`BOSSES`), cycled by stage: VANGUARD → BEHEMOTH → SPECTER → DREADNOUGHT → WRAITH → TITAN
- Each boss has unique attack patterns in `BOSS_PATTERNS` (array of 3 attack functions per boss)
- Boss state: entering (descending), attacking, telegraph phase (telegraph timer = 22 frames before attack fires)
- Attack patterns include: radial bursts, aimed spreads, homing projectiles, spirals, columns, cross-fire, teleport+barrage
- HP scales with difficulty and boss design (`hpMul`); bomb damage = 24 HP

#### 6. **Weapon & Enemy Bullets** (lines 1124–1145, 1219–1358)
- Player bullets: **Vulcan** spreads (angles) with no pierce initially; **Laser** has `pierce` count (goes through enemies)
- Enemy bullets: marked as `radial: true` if dx/dy velocity; otherwise use `speed` and `vx` (aimed)
- Collision detection: `rectsOverlap(a, b)` checks AABB bounding boxes

#### 7. **Scoring & Combo Chain** (lines 1087–1101)
- Base score per enemy type (`score` in `ENEMY_TYPES`)
- **Combo chain:** Each kill increments `chain` counter, resets timer to 2.2s; breaks if timer expires
- **Multiplier:** `1 + floor(chain / 5)`, max 10× from 50+ consecutive kills
- Combo milestone: "N KILLS!" pop-up every 10 kills

#### 8. **Difficulty System** (lines 485–489)
- Three presets: easy, normal, hard
- Tuned parameters: spawn rate, enemy speed, shoot chance, starting lives/bombs, boss HP multiplier
- Persisted in localStorage; can be changed mid-menu (not during play)

#### 9. **Audio** (lines 542–676)
- Web Audio API synth-based sounds (no external audio files)
- `beep(freq, dur, type, vol, opts)`: oscillator with optional pitch sweep and filter
- `noiseBurst(dur, vol, opts)`: procedural noise (white noise → tailored with filter)
- Master gain + dynamics compressor; all audio respects `muted` flag
- Spatial audio: `panFor(x)` maps screen position to stereo pan (-1 to +1)
- Example: `sfx.explodeGrunt()` uses sawtooth + noise; `sfx.laser()` uses sawtooth sweep

#### 10. **Visual Themes** (lines 493–500)
- Six rotating theme palettes, one per stage: Daylight Front → Sunset Corridor → Night Approach → Storm Front → Arctic Wastes → Volcanic Ridge
- Each defines sky gradient (3-color), cloud color, and sun glow color
- Sky rendered as linear gradient; clouds drawn via `drawClouds()` (overlapping ellipses, parallax-scrolling)

#### 11. **Screen Effects** (lines 726–730, 698–709)
- Screen shake: `screenShake(mag, dur)` sets random offsets in canvas context
- Flash vignette: red gradient that pulses when at 1 HP
- Hit flash: white full-screen fade (0.25s for hit, 0.5s for bomb)
- Reduced motion: all animations disabled if `prefers-reduced-motion` or user toggles setting

#### 12. **Persistence** (lines 463–523)
- localStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Best score saved on game-over if beaten
- Settings (mute, difficulty, motion preference) persist across sessions

#### 13. **Canvas Sizing & Responsive Layout** (lines 752–765)
- Fixed internal resolution: 480×720 (portrait aspect ratio)
- CSS handles scaling: `max-width: 100%; max-height: 100%` with scale calculation to fit viewport
- Touch controls enabled on touch devices; hidden on desktop

### Key Variables & State

**Game State** (line 864–869):
- `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- `score`, `lives`, `maxLives`, `bombCount`, `maxBombs`
- `stage`, `stageKills`, `chain`, `multiplier`
- `running`, `paused`, `gameOver`

**Frame Counter** (line 889):
- `frame` increments each update; used for animations (e.g., boss core glow, cloud parallax, sprite blinking)

## Common Tasks

### Test the Game
Open `index.html` directly in a web browser (no server required). Play a quick round to verify:
- Player movement and firing work
- Enemies spawn and attack
- Bosses appear after 18 kills per stage
- Power-ups (weapon, bomb, life) drop and apply correctly
- Audio (if not muted) plays for shooting, explosions, boss attacks
- HUD updates: score, best score, multiplier, weapon level, hearts, bombs

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optionally `weave`
2. Update `spawnEnemy()` (line 1026) probability ranges if needed (e.g., `r < 0.15` for tank)
3. Add corresponding explosion SFX to `sfx` object (line 624) and add type key to `EXPLODE_SFX` (line 678)
4. Update `drawPlane()` or add custom draw function if visual differs from the generic plane

### Add a New Boss Design
1. Add entry to `BOSSES` (line 508) with properties: `name`, `hullTop`, `hullBottom`, `wing`, `core`, `hpMul`
2. Add corresponding pattern array to `BOSS_PATTERNS` (line 1372)—array of 3 attack functions
3. Patterns fire enemy bullets via helper functions like `fireRadial()`, `fireAimedSpread()`, etc.
4. Cycle will automatically use it when `(stage - 1) % BOSSES.length` matches its index

### Adjust Difficulty Curve
Modify `DIFFICULTIES` object (line 485):
- `spawnBase` / `spawnMin`: lower = faster spawning
- `enemySpeed`: multiplier on base enemy speeds
- `enemyShootChance`: 0–1 probability that enemy fires
- `lives` / `bombs`: starting counts
- `bossHpMul`: multiplier on boss base HP (70 + stage × 35)

### Modify Visual Rendering
All drawing happens in `draw()` (lines 1520–1638) and helper functions like `drawPlane()`, `drawBossShip()`, `drawClouds()`. The Canvas 2D API is used throughout:
- Background: sky gradient + sun glow + cloud layers
- Entities: player, enemies (via `drawPlane()`), bosses (via `drawBossShip()`), bullets, particles
- HUD overlaid via HTML elements, not Canvas

## Performance Considerations

- **Quality levels:** Variable `quality` (0/1/2) adjusts particle count, shadow blur, and glow effects based on frame time
- **Particle count:** Capped based on quality tier in `spawnExplosion()` (line 1074)
- **Collision detection:** O(n²) rect overlap checks; acceptable given typical enemy/bullet counts
- **Canvas scaling:** Fixed 480×720 internal resolution scaled via CSS; avoids expensive dynamic resizing

## Files

- `index.html`: Entire game (HTML + embedded CSS + JS)
- `README.md`: Project metadata
- `CLAUDE.md`: This file

No external dependencies, build tools, or server required.
