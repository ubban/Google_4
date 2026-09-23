# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 game — a Raiden-style vertical shoot-'em-up where the player controls a fighter plane, collects weapon upgrades, uses smart bombs, and faces boss encounters across multiple stages.

The entire game (HTML, CSS, JavaScript) is contained in `index.html` (~1669 lines). No external build tools, dependencies, or asset files are required. The game runs entirely in the browser using Canvas 2D and Web Audio APIs.

## Running & Development

**To run:** Open `index.html` in any modern web browser. The game is immediately playable.

**To test changes locally:** Since there's no build step, simply save changes to `index.html` and refresh the browser. Use the browser's Developer Tools (F12) to debug; the entire game runs in a strict IIFE at the bottom of the script block.

**Difficulty modes:** Easy, Normal, Hard — configurable from the start screen; persisted to localStorage.

**Key controls:**
- **Mouse:** Move to steer, click to fire
- **Keyboard:** Arrow keys or WASD to move, Space to fire, X/Shift to drop bomb, P/Esc to pause
- **Mobile:** Drag left half for movement, tap FIRE/BOMB buttons

## Architecture & Key Systems

All logic is organized within a single IIFE (`(function () { ... })()`). The file structure is roughly:

### 1. DOM Setup & Configuration (lines 1–421)
- Canvas and overlay HTML structure
- CSS styling (responsive design, animations, UI elements)
- Configuration constants: `THEMES`, `ENEMY_TYPES`, `BOSSES`, `DIFFICULTIES`, `KILLS_PER_STAGE`
- Storage keys for persistence: best score, mute state, difficulty, reduced motion

### 2. Audio System (lines 542–676)
**Procedural WebAudio synthesis — no audio files.** Uses Web Audio API to generate all sounds on the fly:
- `ensureAudio()`: Initializes audio context (handles browser auto-play policy)
- `beep(freq, dur, type, vol, opts)`: Sine/square/sawtooth oscillators with optional frequency sweep, filtering, and panning
- `noiseBurst(dur, vol, opts)`: White noise with envelope
- `sfx` object: Pre-configured sound effects (shoot, laser, explosions, bomb, powerup, boss hits, stage clear, game over)

**To add a new sound effect:** Add a function to the `sfx` object that calls `beep()` or `noiseBurst()` with appropriate parameters.

### 3. Input Handling (lines 767–861)
Three input methods with priority: keyboard > joystick > mouse (so keyboard always wins if active).
- **Keyboard:** `keys` object tracks ArrowUp/Down/Left/Right, A/W/D/S, Space, X, P, Esc
- **Touch joystick:** Appears on mobile; drag left half of screen to move
- **Mouse:** Plane glides toward pointer when hovering canvas
- Touch controls use `touchstart`/`touchmove`/`touchend` with `preventDefault()` to avoid default mobile behavior

### 4. Game State (lines 863–910)
Central state variables:
- `player`: Position, velocity, weapon, level, invulnerability timer, cooldown
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Entity arrays
- `boss`: Current boss (if any)
- `score`, `lives`, `multiplier`, `chain`, `stage`, `bombCount`: Gameplay metrics
- `running`, `paused`, `gameOver`: State flags

`resetGame()` initializes/resets all state for a fresh run.

### 5. Enemy & Boss Systems (lines 1022–1453)

**Enemy Types** (`ENEMY_TYPES`):
- **grunt:** Basic enemy, 1 HP, score 10
- **interceptor:** Fast, 1 HP, weaves, score 15
- **tank:** Tanky (4 HP), slower, score 35, appears stage 2+

`spawnEnemy()` spawns enemies with random type (weighted by stage), pattern (spread or aimed shots), and weave behavior.

**Boss System:**
- `BOSSES` array: 6 unique boss designs with color schemes and HP multipliers
- `BOSS_PATTERNS` array: 6 pattern sets, each with 3 attack patterns
- Each boss enters from top, bounces horizontally, shows attack telegraph (visual effect), then fires
- `updateBoss(dt)`: Handles movement, telegraph state, attack cooldown, and pattern execution
- Patterns use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`

Stage progression: After 18 enemy kills, boss spawns after 1.2s delay. Defeating boss advances stage, increases enemy spawn rate, cycles theme.

### 6. Collision & Damage (lines 1232–1295)
`rectsOverlap(a, b)`: Simple AABB collision using half-widths.

Collision checks (in `update()`):
- Enemy bullets vs player (triggers `loseLife()`)
- Player bullets vs enemies (pierce mechanic for lasers, hp decrement)
- Bombs (called via `useBomb()`) clear all enemy bullets and damage/kill enemies
- Power-ups collected automatically
- Boss collision dealt separately (only applies damage from bullets, not contact)

### 7. Rendering (lines 1520–1637)
`draw()` called every frame:
- **Background:** Parallax clouds with theme-based sky gradient and sun glow
- **Entities:** Player plane, enemies, bullets, boss, power-ups, particles
- **HUD:** Score, best, multiplier, weapon, hearts (lives), bombs — updated in DOM
- **Screen shake:** Applies random offset to canvas when active

**Drawing functions:**
- `drawPlane(x, y, w, h, color, darkColor, flame)`: Draws player/enemy ships with optional thrust flame
- `drawBossShip(b)`: Draws boss with color-coded design, animated core glow, rotating engine pods

### 8. Game Loop (lines 1640–1665)
`loop(now)` — called via `requestAnimationFrame()`:
1. Calculate delta time (capped at 50ms to prevent large jumps)
2. Track performance; adaptively reduce quality if frame time exceeds threshold
3. If running and not paused: call `update(dt)`, apply screen shake decay, call `draw()`
4. Schedule next frame

### 9. Weapon System (lines 1124–1146)
Two weapons, each with 3 levels:

**Vulcan:**
- Level 1: Single center bullet
- Level 2: Dual spread bullets
- Level 3: Triple spread bullets

**Laser:**
- Level 1: Single center beam (pierce: 1)
- Level 2: Dual offset beams
- Level 3: Triple beams (pierce: 2)

Power-ups cycle: weapon level up → level 3 switches weapon and resets to level 1.

### 10. Scoring & Multiplier (lines 1087–1102, 1304–1322)
- Killing enemies starts a chain; chain resets after 2.2s of no kills
- Multiplier = 1 + floor(chain / 5), capped at 10x
- Score = base enemy points × multiplier
- Bombs award flat 5 points per enemy; bosses award 500 points

Power-up application: life (restore heart), bomb (restore bomb), weapon (upgrade level or switch weapon).

### 11. Difficulty Settings (lines 485–489, 741–750)
Three difficulty presets affecting:
- Spawn rate: base interval and minimum interval
- Enemy speed and shoot probability
- Starting lives and bombs
- Boss HP multiplier

Persisted to `localStorage` under key `skyfighter.difficulty`.

### 12. UI & Overlay (lines 200–358)
- **Start screen:** Difficulty buttons, start button, instructions
- **Game over screen:** Final score, best score, new best message, play again button
- **Pause screen:** Resume button
- **In-game HUD:** Score, best, multiplier, weapon info, health/bomb indicators (updated live)

All controlled via overlay div; game only renders canvas when overlay is hidden.

## Performance Optimization

**Adaptive quality** (`quality` variable, tracked via `trackPerf()`):
- **Quality 0:** Full effects (glow, shadows, many particles)
- **Quality 1:** Reduced particle count (~60% of normal)
- **Quality 2:** Minimal particles (~35% of normal), no shadow blur

Adjusts based on rolling average frame time. Particle count dynamically scales; boss glow and shadow effects reduce or disable at lower quality.

**Reduced motion preference:**
- Respects `prefers-reduced-motion` media query
- Can be toggled via UI button
- Disables screen shake (reduced to 15% magnitude), animations, flash effects
- Persisted to localStorage

## Extending the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add shooting pattern logic in `spawnEnemy()` if desired (modify pattern selection)
3. Add explosion sound in `sfx` (e.g., `explodeNewType`) and reference in `EXPLODE_SFX` map

### Adding a New Boss
1. Add boss design to `BOSSES` array with colors and `hpMul`
2. Add pattern set to `BOSS_PATTERNS` (3 attack patterns using helper functions)
3. Boss will automatically cycle through designs as stages progress

### Adding a New Weapon
1. Add weapon name to weapon selection logic (currently: `vulcan` or `laser`)
2. Modify `fireWeapon()` to handle new weapon type; define projectile properties (speed, pierce, etc.)
3. Add sound effect to `sfx`
4. Adjust `weaponLabel()` to display name

### Adding/Modifying Themes
1. Add entry to `THEMES` array with sky gradient colors, cloud color, sun color, and name
2. Themes cycle per stage; modify the modulo operator in `draw()` and `killBoss()` to adjust frequency

### Tuning Difficulty
Adjust values in `DIFFICULTIES` object. Key parameters:
- `spawnBase`/`spawnMin`: Enemy spawn interval in frames (lower = more enemies)
- `enemySpeed`: Speed multiplier for enemies
- `enemyShootChance`: Probability enemy shoots (0–1)
- `bossHpMul`: Boss health multiplier

## Storage & Persistence

Uses `localStorage` with these keys (all prefixed with `skyfighter.`):
- `bestScore`: Best score achieved (JSON number)
- `muted`: Mute state (1 or 0)
- `difficulty`: Last selected difficulty (easy/normal/hard)
- `reducedMotion`: Motion preference (1 or 0)

## Browser Compatibility

Requires:
- **Canvas 2D context** (`getContext('2d')`)
- **Web Audio API** (`AudioContext` or `webkitAudioContext`)
- **localStorage** (for persistence)
- **Touch API** (for mobile)
- ES6+ (arrow functions, destructuring, const/let)

Tested on modern browsers (Chrome, Firefox, Safari, Edge). Older IE not supported.

## Code Style & Patterns

- **Single IIFE:** All code wrapped in strict-mode IIFE to avoid global pollution
- **Inline assets:** All graphics drawn procedurally; all sounds synthesized; no external files
- **Functional over OOP:** Uses plain objects for entities (player, enemy, bullet) rather than classes; logic is procedural
- **Canvas rendering:** 2D context; no WebGL, no libraries
- **Adaptive logic:** Frame rate detection; quality degradation; responsive canvas sizing

## Common Debugging

**Game not running:**
- Check browser console for errors (F12)
- Verify audio context initialized (autoplay policy may block)
- Check if `running` flag set after clicking Start

**Janky performance:**
- Lower quality settings auto-engages if FPS drops; check `quality` variable in DevTools console
- Disable glow/shadow effects by reducing particle counts or disabling shadow blur in `draw()`

**Enemies not spawning:**
- Check `DIFFICULTIES[difficulty]` spawn parameters
- Verify `stage` and `stageKills` are incrementing correctly

**Boss not appearing:**
- Ensure `stageKills >= KILLS_PER_STAGE` (18 kills) and `spawnBoss()` is called
- Boss only appears after brief delay; check timeout in `update()` logic
