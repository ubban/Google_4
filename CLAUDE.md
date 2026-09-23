# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML5 file with embedded CSS and JavaScript. It features player aircraft combat, enemy waves, boss battles, weapon upgrades, and a scoring system with combo multipliers.

- **No build system** — this is a standalone `index.html` file
- **No external dependencies** — all sounds are synthesized with Web Audio API
- **No tests or linting** — development changes are directly testable in a browser

## Running the Game

Open `index.html` in any modern web browser. The game is fully playable on desktop (keyboard/mouse) and mobile (touch controls).

**Test locally:**
- Use a simple HTTP server: `python3 -m http.server 8000` then visit `http://localhost:8000`
- Or open the file directly with `file://` (works fine for this project)

## Architecture

### High-Level Structure

The game uses a standard game loop pattern with three phases per frame:
1. **Update** (`update(dt)`) — process input, move entities, detect collisions, update game state
2. **Draw** (`draw()`) — render all sprites and backgrounds to canvas
3. **Loop** (`requestAnimationFrame(loop)`) — runs at ~60fps

### Key Subsystems

**Game State**
- Located around line 863-869: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Tracked: `score`, `lives`, `running`, `paused`, `gameOver`, `stage`, `chain` (combo multiplier)

**Input Handling** (lines 767-862)
- Keyboard: Arrow keys / WASD for movement, Space to fire, X/Shift for bombs, P/Esc to pause
- Mouse: Movement steers player, click fires (when hovering canvas)
- Touch: Joystick on left half for movement, FIRE/BOMB buttons on right half
- Implementation: `keys` object tracks held keys; `mouseActive`, `joyActive` track input modes
- Priority: keyboard/joystick > mouse (never fight each other)

**Entity Systems**
- Bullets (line 1188): fast travel, despawn off-screen; weapon type (vulcan vs laser) determines spread/pierce
- Enemies (lines 1022-1040, 1210-1227): spawn based on difficulty, move down, shoot at/near player, weave patterns
- Enemy Bullets (line 1191): update position, detect player collision
- Particles (lines 1296-1297): decorative explosion fragments, life-based lifespan
- Power-ups (lines 1058-1067, 1194-1195): spawn from dead enemies, types are life/bomb/weapon

**Difficulty & Balance** (lines 485-489)
- Three difficulty levels (easy/normal/hard) modify spawn rates, enemy speed, shoot chance, boss HP
- `KILLS_PER_STAGE` = 18 enemy kills to trigger boss battle
- `THEMES` array (line 493): 6 different visual themes with sky gradients and cloud rendering

**Enemy Types** (lines 502-506)
- `grunt`: slow, 1 HP, score 10, starts stage 1
- `interceptor`: fast, 1 HP, weaves, score 15, starts stage 1
- `tank`: slow, 4 HP, score 35, starts stage 2

**Weapon System** (lines 871-1145)
- Vulcan: horizontal spread, multiple lanes at higher levels (1-3 lanes)
- Laser: vertical lanes, pierce through enemies, higher fire rate
- Weapon levels 1-3, then swap to other weapon at level 3
- Each weapon has different visual indicators in HUD and canvas glow

**Boss System** (lines 1043-1055, 1372-1408, 1411-1434)
- 6 unique boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each boss has 3 attack patterns (rotates through them)
- Pattern types: radial bursts, aimed spreads, columns, spirals, crosses, homing, teleport
- Entering animation (y += 60px until y >= 120)
- Telegraph phase before attack (22 frames)
- HP scales with stage and difficulty

**Collision Detection** (line 1069-1070)
- `rectsOverlap(a, b)`: simple AABB using distance thresholds
- Checked: bullet-enemy, bullet-boss, enemy-player, enemyBullet-player, boss-player, powerUp-player

**Audio Synthesis** (lines 542-676)
- `ensureAudio()`: lazy-init Web Audio context (respects user gesture requirement)
- `beep(freq, dur, type, vol, opts)`: oscillator-based tones with optional sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: white noise with decay
- `sfx` object: pre-composed sounds (shoot, laser, explosions, powerup, boss telegraph, etc.)
- Muted state persisted to localStorage

**Screen Shake & Effects** (lines 726-730, 698-709)
- `screenShake(mag, dur)`: applies random translation to canvas each frame
- Respects `reducedMotion` preference (reduced amplitude and duration)
- Flash effects (hit/bomb) via CSS animation triggers
- Vignette pulse on critical health

**Scoring & Chain Multiplier** (lines 1087-1100)
- `addScore(base)`: increments chain, updates multiplier (1 + Math.floor(chain/5), capped at 10x)
- `chainTimer`: 2.2 second window to keep chain alive
- `breakChain()`: resets chain and multiplier on damage or timeout

**Adaptive Performance** (lines 473-483)
- `quality` levels: 0 (full) → 1 (medium) → 2 (low)
- Tracks last 45 frame times, adjusts based on average
- Affects: particle count, shadow/glow effects, explosion particle density

**Rendering Optimization**
- Canvas clear + context save/restore for transformations
- Conditional shadow/glow based on quality level
- Cloud parallax at two speeds
- Theme-based sky gradient at runtime

### Game Flow

1. **Title/Menu** (overlay shown): Select difficulty, review high score
2. **Game Loop** (overlay hidden): 
   - Spawn enemies every N frames (decreases over time)
   - Player fires continuously when key/button held
   - Collision detection updates score, health, boss state
3. **Boss Phase**: After 18 enemy kills, spawn boss with entrance animation
4. **Stage Clear**: Boss dies → score spike (500 pts) → advance stage, reset enemy counter
5. **Game Over**: No lives remaining → show final score, high score, restart option

## Storage & Persistence

Uses `localStorage` for:
- `skyfighter.bestScore` — high score across sessions
- `skyfighter.muted` — audio preference
- `skyfighter.difficulty` — last selected difficulty
- `skyfighter.reducedMotion` — motion sickness preference (merged with OS preference)

## Important Constants & Tweaks

- **Canvas base size**: 480×720 (scales to fit viewport up to 1.6× actual size)
- **Player speed**: 260 px/sec
- **Fire cooldown**: 0.15s (vulcan), 0.1s (laser)
- **Bomb cooldown**: None (1 per use, max stored depends on difficulty)
- **Invulnerability duration**: 1.5s after hit
- **Max multiplier**: 10× (requires 50 kills in 2.2s window)
- **Boss entrance position**: y = -80, slides to y = 120
- **Boss attack telegraph duration**: 22 frames

## Common Edits

**Adjust Difficulty**
- Modify `DIFFICULTIES` object (lines 485-489): `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `bossHpMul`

**Add New Boss Pattern**
- Add function to pattern function definitions (lines 1324-1370)
- Reference in `BOSS_PATTERNS` array (lines 1372-1409)
- Boss designs in `BOSSES` array (lines 508-515) — up to 6 (cycles)

**Add New Enemy Type**
- Add entry to `ENEMY_TYPES` (lines 502-506): define `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
- Update spawn logic in `spawnEnemy()` if needed (lines 1022-1040)

**Add New Weapon**
- Modify `fireWeapon()` (lines 1124-1145) to add weapon case
- Update weapon swap logic in `applyPowerUp()` (lines 1304-1322)
- Update HUD label in `weaponLabel()` (lines 871-872)

**Tweak Visuals**
- CSS styles (lines 10-358) control all UI, animations, button styling
- Canvas drawing functions: `drawPlane()` (913-968), `drawBossShip()` (970-1019), `drawClouds()` (1507-1518)

**Add/Remove Themes**
- Modify `THEMES` array (lines 493-500): sky gradients, cloud color, sun color, name
- Theme cycles based on `stage % THEMES.length`
