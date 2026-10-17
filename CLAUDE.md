# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-file HTML5 web application. The entire game engine, UI, and game logic are contained in `index.html` with embedded CSS and JavaScript. There are no build tools, dependencies, or external assets—it runs standalone in any modern browser.

## Running the Game

1. **Direct**: Open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
2. **Local server** (for consistent behavior): Use `python3 -m http.server 8000` or `npx http-server` in the repository directory, then visit `http://localhost:8000`

No build step is required. The game is fully functional immediately.

## Code Structure (Single File)

The `index.html` file is organized in logical sections within a single `<script>` tag (lines 422–1667):

### Core Systems

- **Initialization & Storage** (lines 468–527): LocalStorage for best score, mute state, difficulty, and reduced-motion preference
- **Audio System** (lines 542–676): WebAudio synth-based sound effects (`beep`, `noiseBurst`, and `sfx` object with named effects like `shoot`, `bomb`, `explode`)
- **UI Elements** (lines 680–750): Rendering HUD elements (hearts, bombs, weapon info) and difficulty selection
- **Canvas Setup** (lines 752–765): Responsive canvas sizing for desktop and mobile
- **Input Handling** (lines 767–861): Keyboard, mouse steering, and touch/joystick controls
- **Game State** (lines 863–911): Player, bullets, enemies, powerups, boss, score, lives, stage, chain multiplier; `resetGame()` initializes a new game
- **Game Loop** (lines 1640–1652): `loop(now)` runs at 60 FPS; calls `update(dt)` then `draw()`

### Game Mechanics

- **Enemy System** (lines 1022–1041): `spawnEnemy()` creates enemies based on stage difficulty
- **Boss System** (lines 1043–1055, 1324–1434): Boss spawns after 18 kills per stage; `updateBoss(dt)` handles boss movement and attack patterns (line 1372 has 6 distinct boss patterns)
- **Collision & Damage** (lines 1232–1302): Collision detection between player/enemies, bullets/enemies, bullets/boss; scoring on kills with chain multiplier
- **Weapon & PowerUp System** (lines 1124–1322): Two weapons (Vulcan, Laser) with 3 upgrade levels; powerups for health, bombs, and weapons
- **Rendering** (lines 1507–1638): Background gradient/clouds, plane sprites (player and enemies), bullets, particles, HUD overlay
- **Game States**: `running`, `paused`, `gameOver`; overlay shows at start/pause/end

### Key Data Structures

- **Player**: `{x, y, w, h, speed, cooldown, rapid, weapon, level, invuln}`
- **Enemy**: `{x, y, w, h, speed, hp, type, canShoot, pattern, ...}`
- **Boss**: `{x, y, w, h, hp, maxHp, entering, phaseTimer, attackTimer, design, ...}`
- **Bullet**: `{x, y, w, h, speed, vx, pierce, laser?}` (player) or `{x, y, w, h, speed, vx}` (enemy)
- **Difficulty**: `{spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul}`

## Common Development Tasks

### Adding/Modifying Enemy Types

Define new types in `ENEMY_TYPES` (line 502), then add selection logic in `spawnEnemy()` (line 1026). Enemy rendering uses `drawPlane()` (line 913).

### Adding/Modifying Bosses

Add entries to `BOSSES` (line 508) with color scheme. Create a new pattern array in `BOSS_PATTERNS` (line 1372) with functions like `fireRadial()`, `fireAimedSpread()`, etc. Pattern logic triggers after telegraph phase (line 1421).

### Tweaking Difficulty

Modify `DIFFICULTIES` object (line 485). Affects spawn rate, enemy speed, boss HP, and available lives/bombs.

### Sound Effects

All audio is synthesized in the `sfx` object (line 624). Each effect uses `beep()` or `noiseBurst()` with frequency, duration, type, and filter params. To add a new sound, create a function in `sfx` that calls these primitives.

### Visual Changes

- **Theme/colors**: Modify `THEMES` array (line 493) for sky gradients and cloud colors
- **Player drawing**: Edit `drawPlane()` function (line 913) for ship shape or `drawBossShip()` (line 970) for bosses
- **Particles**: Adjust `spawnExplosion()` (line 1073) spawn count and color for hit effects
- **Screen shake**: Tweak `screenShake()` calls (e.g., line 1108) with magnitude and duration

### Game Balance

- Enemy spawn rate & difficulty ramp: `DIFFICULTIES` and `spawnTimer` logic (line 1198)
- Weapon balance: Fire rates (line 1177), spread patterns (line 1127), and damage to enemies/bosses (lines 1237, 1259)
- Player movement speed: `player.speed` (line 877)
- Scoring multiplier: Chain reset timer & thresholds (lines 1089, 1090)

## Performance & Optimization

The game includes **adaptive quality** (lines 474–483) that reduces particle count and glow effects if frame rate drops. `quality` ranges from 0 (full effects) to 2 (minimal). Frame time is tracked and quality adjusts automatically.

## Browser Compatibility

- Requires: HTML5 Canvas, ES6 (arrow functions, template literals), Web Audio API
- Tested: Modern browsers (Chrome, Firefox, Safari 13+, Edge)
- Mobile: Touch controls are enabled automatically if `ontouchstart` is detected
- Reduced-motion support: Respects `prefers-reduced-motion` media query and allows user toggle

## Asset Storage

The game uses **no external assets**. Everything is rendered procedurally:
- Favicon: Embedded SVG data URI (line 9)
- Graphics: Canvas drawing API
- Audio: WebAudio synthesis
- No network requests (except for display fonts, none required)

## Notes for Modification

- The single-file approach keeps the game portable but can make the file large. Keep related logic together (e.g., all enemy spawning logic near line 1022).
- Input handling is split into keyboard (line 769), mouse (line 847), and touch (line 814) sections; test changes on all three paths.
- Boss patterns use relative positioning and scaling—verify pattern projectiles don't go offscreen on all resolutions before committing.
- The localStorage keys (line 463) are namespaced with `skyfighter.` prefix to avoid conflicts if the HTML is hosted alongside other apps.
