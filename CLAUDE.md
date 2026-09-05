# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical bullet-hell game built entirely in vanilla JavaScript. It's a single HTML file with embedded CSS and JavaScript—no build process, no dependencies, no external assets. The game runs in any modern web browser.

## Architecture & Key Systems

### Single-File Design
The entire game lives in `index.html`. CSS is in a `<style>` block, game logic in a `<script>` block. This simplicity means:
- Open the file directly in a browser to play
- No build step or server required
- Modifications take effect on refresh
- All game state is self-contained (except localStorage for high scores/settings)

### Core Game Loop & Timing
- **Frame-based update**: `requestAnimationFrame(loop)` drives everything at ~60fps
- **Delta time**: Converts frame timing to real time (`dt` = seconds since last frame)
- **Performance adaptive**: Quality variable (0/1/2) scales particle effects and shadow blur based on average frame time to maintain 60fps on lower-end devices

### Separation of Concerns (within the IIFE)
1. **Canvas & DOM**: `canvas`, `ctx`, overlay elements, buttons
2. **Game State**: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, score, lives, stage, chain
3. **Input**: Keyboard keys object, touch/joystick tracking, mouse steering
4. **Audio**: Web Audio Context, synthesized SFX (no audio files)
5. **Rendering**: `draw()` creates full canvas scene each frame
6. **Update**: `update(dt)` advances all entities, handles collisions, spawning

### Key Game Systems

#### Difficulty System
Three presets (Easy/Normal/Hard) affect spawn rate, enemy speed, shot frequency, and lives. Stored in localStorage and can be changed mid-menu.

#### Weapon & Power-Up System
- **Vulcan**: Spread shot, faster fire rate
- **Laser**: Single/dual/triple streams, pierces enemies
- **Power-up types**: Life (heart), Bomb, Weapon upgrade
- Weapons level up; at level 3 you can switch between Vulcan and Laser

#### Enemy Types & Spawning
- **Grunt**: Basic fighter (1 HP, fast)
- **Interceptor**: Weaving pattern (1 HP, faster)
- **Tank**: Heavily armored (4 HP, slow, appears stage 2+)
- Spawning is **timer-based** (`spawnTimer`); as difficulty ramps, spawn rate increases every 5 seconds of gameplay
- Spawn attempts stop once boss battle begins (`if (!boss)`)

#### Boss System
- **6 boss designs** cycle through stages (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- **Attack patterns**: Each boss has 3 different attack patterns (radial bursts, aimed spreads, teleport barrage, etc.)
- **Telegraph mechanic**: Boss shows a visual warning (core flashes white) for ~22 frames before firing
- **HP scales** with difficulty (`bossHpMul`) and boss design (`hpMul`)
- Bosses enter from top, move side-to-side, fire repeatedly until defeated

#### Collision Detection
Simple **AABB (axis-aligned bounding box)** `rectsOverlap(a, b)` checks:
- Player vs bullets (both friendly and enemy)
- Player vs enemies & boss
- Bullets vs enemies & boss
- Player vs power-ups

#### Screen Shake & Visual Feedback
- `screenShake(magnitude, duration)` sets `shakeTime` and `shakeMag`
- Reduces in reduced-motion mode (`shakeTime = Math.min(dur, 0.1)`)
- Flash overlay (`#flash`) animates on hit or bomb detonation
- Vignette effect pulses when health is critical (1 HP)

#### Score & Combo System
- **Chain multiplier**: Each kill increments `chain`; multiplier = 1 + floor(chain / 5), max 10x
- **Chain timer**: Resets to 0 on gap > 2.2 seconds, multiplier drops back to 1x
- Combo pop shows every 10 kills

#### Touch Controls
- **Left zone** (55% of width): Virtual joystick with drag-to-steer
- **Right zone** (45% of width): FIRE and BOMB buttons
- Joystick base and stick visually appear when dragging

#### Audio (Web Audio Synthesis)
- **No external audio files**—all SFX synthesized with oscillators and noise bursts
- Master compressor and gain control
- Panning for positional audio (left/right based on collision X position)
- `beep(freq, dur, type, vol, opts)` and `noiseBurst(dur, vol, opts)` are the primitives

#### Rendering & Themes
- **6 themes** (Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge)
- Themes cycle per stage and customize sky gradient, cloud color, sun glow
- **Particle system**: Explosions spawn short-lived particles with velocity and fade
- **Drawing order**: Sky → clouds → player → bullets → enemies → boss → particles → UI overlays

## Code Walkthrough

### Global State Variables (within IIFE)
```
player, bullets, enemies, enemyBullets, particles, powerUps, boss
score, lives, maxLives, running, paused, gameOver
bombCount, maxBombs
spawnTimer, frame, kills, stage, stageKills, bossWarned
chain, chainTimer, multiplier
```

### Main Functions

- **`resetGame()`**: Initializes all state for a new game
- **`update(dt)`**: Advances all entities; called once per frame if not paused/game over
- **`draw()`**: Renders everything to canvas; called once per frame
- **`loop(now)`**: requestAnimationFrame callback; computes dt, calls update/draw, checks pause/game-over
- **`fireWeapon()`**: Spawns bullets based on current weapon type and level
- **`spawnEnemy()`**: Creates a random enemy at top of screen
- **`spawnBoss()`**: Creates boss entity; transitions game to boss-battle mode
- **`updateBoss(dt)`**: Boss movement, telegraph timing, pattern firing
- **`killBoss()`**: Handles boss defeat, advances stage, plays fanfare
- **`useBomb()`**: Clears enemy bullets, damages/kills all enemies and boss, screen shake
- **`loseLife()`**: Decrements lives, breaks chain, checks game over
- **`endGame()`**: Shows overlay, saves high score, triggers game-over SFX
- **`togglePause()`**: Freezes game loop, shows pause overlay
- **`resize()`**: Responsive canvas sizing; maintains BASE_W × BASE_H aspect ratio

### Key Constants

- `BASE_W` (480px) × `BASE_H` (720px): Internal canvas resolution
- `DIFFICULTIES`: Spawn timers, enemy speed, boss HP multiplier per difficulty
- `THEMES`: Sky colors, cloud/sun colors, stage names
- `ENEMY_TYPES`: Stats for grunt, interceptor, tank
- `BOSSES`: Name, hull/wing/core colors, HP multiplier per boss
- `BOSS_PATTERNS`: 2D array; patterns[boss_index][pattern_num] is a function that fires a burst
- `KILLS_PER_STAGE`: 18 kills trigger boss arrival

## How to Modify the Game

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add new SFX to `EXPLODE_SFX` map if explosion sound differs
3. Update `spawnEnemy()` logic to choose your type based on stage/probability

### Add a New Boss
1. Add entry to `BOSSES` array with colors and `hpMul`
2. Add a new pattern array to `BOSS_PATTERNS` (index must match BOSSES index)
3. Each pattern is a function `(boss) => { ...fire bullets... }`
4. Patterns can use `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.

### Add a New Weapon
1. Modify `fireWeapon()` to handle new weapon type
2. Add bullet properties (speed, spread, pierce, etc.)
3. Update weapon-switching logic in `applyPowerUp()`

### Adjust Difficulty Balancing
- Edit `DIFFICULTIES` object for spawn rate, enemy speed, lives
- Tweak `KILLS_PER_STAGE` to speed up / slow down boss arrival
- Boss HP multipliers in `spawnBoss()` depend on `d.bossHpMul`

### Change Visual Style
- Modify `THEMES` for different sky gradients and cloud colors
- Adjust `drawPlane()` and `drawBossShip()` functions to alter ship designs
- Particle colors in `spawnExplosion()` calls

### Adjust Audio
- Modify `beep()` and `noiseBurst()` calls in the `sfx` object
- Tweak frequencies, durations, filters, and volumes

## Testing & Running

### Quick Start
1. Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge)
2. Game starts on page load with intro overlay
3. Click "Start Game" to play

### Testing Different Scenarios
- **Difficulty**: Click difficulty buttons before starting to test spawn rate, enemy AI
- **Reduced motion**: Toggle 🎆 button to verify animations work with/without CSS animations
- **Touch**: Open on mobile or use Chrome DevTools device emulation to test joystick
- **High score**: Play a game, close tab, reopen—high score persists in localStorage
- **Boss patterns**: Advance to stage 2+ quickly to test boss behavior (or set `stage = 2` in console, trigger boss spawn via kills)

### Browser DevTools Console
```javascript
// Set stage for quick testing
stage = 5;
showStageBanner('STAGE 5');

// Trigger instant boss spawn
spawnBoss();

// Modify difficulty mid-game
difficulty = 'hard';

// Check localStorage
console.log(localStorage.getItem('skyfighter.bestScore'));
```

## Performance & Optimization

- **Particle count** scales with quality level to maintain 60fps
- **Shadow blur** disabled on low-quality; shadow rendering is expensive on canvas
- **Frame time tracking**: Every frame's delta is logged; if average > 1/40s (25ms), quality increases
- **Collision checks**: O(enemies × bullets) + enemies vs boss; acceptable for typical spawn counts
- **Canvas size**: Rendered at 480×720; CSS scaling up to 1.6x means no loss of visual quality on large displays

## Known Constraints & Future Considerations

- **Single file**: Assets must be data URIs or synthesized (favicon is inline SVG)
- **No persistence beyond localStorage**: State is lost on page refresh unless high score is saved
- **Touch controls** are simple drag-to-steer; could be enhanced with tap-and-aim variants
- **Boss variety**: Pattern count per boss is fixed; could be randomized or blended
- **Enemy AI**: Weaving and aimed shots are deterministic; no advanced pathfinding

## External Dependencies

**None.** Game uses only:
- Canvas 2D API (drawing)
- Web Audio API (synthesis)
- localStorage (persistence)
- requestAnimationFrame (timing)
- Standard DOM APIs (buttons, events)

All fully supported in modern browsers (IE 11+, all current versions).
