# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It's a complete, self-contained HTML5 game with no external dependencies.

### Key Features
- 3 difficulty levels (Easy, Normal, Hard)
- 6 progressively-fought boss encounters
- Dual weapon system (Vulcan spray, Laser piercing) with 3 upgrade levels
- Smart bomb mechanic for clearing the screen
- Score multiplier chain system
- 6 dynamic sky themes with parallax cloud layers
- Adaptive performance quality scaling
- Full touch control support for mobile
- Web Audio API procedural sound effects
- Accessibility features (reduced motion, screen reader announcements)
- Local storage persistence (high scores, settings)

### Technology Stack
- **Single HTML file** (index.html) containing all HTML, CSS, and JavaScript
- **Canvas 2D** for rendering game graphics
- **Web Audio API** for real-time sound synthesis
- **localStorage** for persistence
- Pure vanilla JavaScript—no build tools, frameworks, or external assets needed

## How to Run

1. **Open the game**: Simply open `index.html` in any modern web browser
2. **Desktop controls**: Mouse to aim, click or Space to fire, X/Shift for bombs, P/Esc to pause
3. **Mobile/touch**: Drag left zone to move, tap FIRE/BOMB buttons
4. **Adjust difficulty**: Select from the start screen

## Codebase Architecture

The entire game is implemented in a single `<script>` block in index.html. Here's the structure:

### Constants & Configuration (lines 485–523)
- **DIFFICULTIES**: Three preset configs (easy/normal/hard) controlling spawn rates, enemy speed, lives, bomb count, and boss HP multiplier
- **THEMES**: Six sky gradient/cloud color schemes that cycle per stage
- **ENEMY_TYPES**: Three enemy archetypes (grunt, interceptor, tank) with stats and behavior flags
- **BOSSES**: Six boss designs with unique attack pattern sets and visuals
- **STORAGE_***: localStorage keys for best score, mute state, difficulty, reduced motion

### Audio Engine (lines 542–676)
- **ensureAudio()**: Lazily initializes Web Audio context with a compressor chain
- **beep()**: Synthesizes tones with frequency sweeps, filters, and panning
- **noiseBurst()**: Creates noise bursts for percussive effects
- **sfx object**: Named sound effects used throughout (shoot, explode, bomb, hit, boss telegraph, etc.)
- All sounds are synthesized in real-time—no audio files required

### Game State & Initialization (lines 863–911)
- **resetGame()**: Clears all entities, resets player position, lives, bombs, and stage
- Game state arrays: `player`, `bullets`, `enemies`, `enemyBullets`, `powerUps`, `boss`, `particles`
- Difficulty, lives, score, kills, stage, chain multiplier tracked at module level

### Player & Input (lines 768–862)
- **Keyboard input**: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb
- **Touch input**: Joystick in left zone (moveZone) and two buttons (fireBtn, bombBtn)
- **Mouse input**: Pointer aim (mouseActive = true when over canvas), click to fire
- **fireWeapon()**: Generates Vulcan or Laser bullets based on weapon type and level
- **useBomb()**: Clears all enemy bullets, damages boss, removes enemies, awards score

### Collision & Physics (lines 1069–1302)
- **rectsOverlap()**: AABB collision detection by comparing center distances
- **update(dt)**: Main update loop handling:
  - Player movement (keyboard/joystick/mouse priority)
  - Bullet lifecycle (fired and filtered off-screen)
  - Enemy spawning (spawn timer affected by difficulty)
  - Boss spawning (after KILLS_PER_STAGE = 18 enemies killed)
  - Collision detection (bullets vs enemies, bullets vs boss, enemies vs player, enemy bullets vs player, boss vs player)
  - Power-up collection
  - Chain timer and multiplier decay
  - Performance tracking (quality = 0/1/2 based on frame time average)

### Enemy Behavior (lines 1022–1228)
- **spawnEnemy()**: Creates an enemy with random type, position, speed, and shoot pattern
- Enemies spawn off-top, move down, and can weave left-right (interceptors)
- Three shot patterns: "spread" (fan) or "aimed" (toward player)
- Enemies despawn when off-bottom or killed

### Boss System (lines 1043–1454)
- **spawnBoss()**: Creates boss with entering animation, HP scaled by stage & difficulty
- **updateBoss(dt)**: Handles boss horizontal movement, telegraph timing, and pattern execution
- **BOSS_PATTERNS**: Each boss has 3 unique attack patterns (radial, columns, homing, spirals, etc.)
- **Pattern firing functions**: fireRadial, fireAimedSpread, fireColumns, fireSpiralBurst, etc.
- Boss telegraphs 22 frames before firing, then enters 85-frame cooldown
- **killBoss()**: Awards 500 score, progresses stage, triggers stage banner

### Visual Rendering (lines 1520–1638)
- **draw()**: Clears canvas, applies screen shake, draws sky gradient, clouds, player, bullets, enemies, boss, power-ups, particles
- **drawPlane()**: Procedurally draws player or enemy ships (wings, hull, cockpit, exhaust flame)
- **drawBossShip()**: Procedurally draws boss with design colors, telegraphing glow effect
- **drawClouds()**: Parallax animated clouds at different speeds
- Quality-based rendering (glow shadows disabled on low quality)

### Game Flow (lines 1469–1505)
- **endGame()**: Triggers when lives hit 0; compares score to best, shows game-over overlay
- **togglePause()**: Freezes update loop, shows pause overlay
- **Main loop (loop)**: requestAnimationFrame-driven, updates at capped dt, calls update() then draw()

### HTML & UI Elements
- **HUD**: Score, best score, multiplier, weapon info, hearts, bombs (top-center)
- **Boss bar**: Shows boss health during boss encounters (hidden otherwise)
- **Overlay**: Start screen, pause screen, game-over screen (centered card layout)
- **Top buttons**: Mute, reduced-motion toggle, pause
- **Touch controls**: Joystick base + fire/bomb buttons (hidden on desktop)
- **Accessibility**: ARIA live region (#srLive) for screen reader announcements

## Common Tasks

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` constant (around line 502) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage` properties
2. Add to `choices.includes()` filter in spawnEnemy() if limiting by stage
3. Add probability check in spawnEnemy() to decide when it spawns
4. Optionally add unique AI pattern (weave, behavior flags)
5. Add sound effect to `EXPLODE_SFX` mapping (line 678)

### Add a New Boss
1. Add entry to `BOSSES` constant (around line 508) with name, hull colors, wing color, core glow color, hpMul
2. Add 3 firing pattern functions to `BOSS_PATTERNS` array in the same order as bosses
3. Pattern functions receive the boss object and fire using `fireRadial()`, `fireAimedSpread()`, etc.
4. Boss design is auto-selected by `(stage - 1) % BOSSES.length`

### Adjust Difficulty
1. Modify entries in `DIFFICULTIES` constant (line 485): spawnBase/spawnMin control spawn rates; enemySpeed/enemyShootChance control behavior; lives/bombs/bossHpMul control player resources and boss HP
2. Changes persist to localStorage and apply on next game start

### Add a Visual Effect
1. **Screen shake**: Call `screenShake(magnitude, duration)` (e.g., on bomb use or player hit)
2. **Flash overlay**: Add/remove class on `#flash` element (`hit` or `bomb` class triggers animation)
3. **Particles**: Push to `particles` array with x, y, vx, vy, life, maxLife, color
4. Reduced motion respects prefers-reduced-motion and toggles via UI button

### Optimize Performance
- Game tracks average frame time (last 45 frames) and adjusts `quality` variable (0 = full, 1 = medium, 2 = low)
- Quality affects: particle count, shadow/glow rendering, cloud detail
- No manual FPS cap; runs at display refresh (60/120Hz native)

## Key Invariants

- **Canvas size**: Fixed at 480×720 logical pixels, scaled to fit window (up to 1.6x)
- **Movement priority**: Keyboard > joystick > mouse aim
- **Firing priority**: Keyboard Space > mouse click > touch fire button
- **Chain multiplier**: Decays if > 2.2 seconds without a kill; rewards combo gameplay
- **Invincibility frames**: 1.5 seconds after being hit
- **Boss entrance**: Animates from top for ~1.2 seconds before engaging

## Testing in Browser

1. Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge)
2. Inspect console for any errors
3. Test controls: keyboard, mouse, and (on mobile) touch
4. Verify sounds play (check browser autoplay policy; user must click before audio plays)
5. Check that scores persist across refresh (localStorage)
6. Test all difficulties and verify enemy/boss scaling
7. Verify reduced-motion toggle disables animations (body class toggle)

## Future Enhancement Ideas

- Additional enemy types (mines, laser turrets)
- Power-up variations (shield, rapid-fire, targeting assist)
- Stage-specific gimmicks (obstacles, conveyor belts)
- Leaderboard (cloud sync or local IndexedDB)
- Mobile app distribution (PWA manifest, offline support)
- Sound settings per effect (individual mute toggles)
- Replay save/playback system
