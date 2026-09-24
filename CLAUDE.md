# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML5 file. It features procedural enemy spawning, boss battles, multiple weapon types, and adaptive performance optimization.

## Development

### Quick Start
- The entire game is contained in `index.html` (single file, ~1.6k lines)
- Open `index.html` directly in a browser to play
- No build process, server, or dependencies required
- Works on desktop (keyboard controls) and mobile (touch joystick)

### Testing the Game
To test changes:
1. Open `index.html` in a browser
2. Test gameplay mechanics by playing through stages
3. Press ⏸ to pause and verify pause state works
4. Toggle 🔊 mute button to test audio
5. Click difficulty buttons to verify game resets properly
6. Test on mobile devices to verify touch controls

### Development Tips
- **Performance tracking**: The game monitors frame times and automatically reduces visual quality (glows, shadows) if FPS drops
- **Browser DevTools**: Use Console to monitor performance; game logs to `console` via `trackPerf(dt)` callback
- **Accessibility**: Game includes ARIA announcements for screen readers; verify announcements work after changes
- **LocalStorage**: Game persists best score, mute state, difficulty, and reduced motion preference under keys like `skyfighter.bestScore`

## Architecture

### Game Loop & Rendering
- **Main loop**: `requestAnimationFrame(loop)` continuously runs at ~60 FPS
- **Update**: `update(dt)` processes all game logic (dt = delta time in seconds, clamped to 0.05s max)
- **Render**: `draw()` renders canvas (only called if game is running and not paused)
- **Delta time handling**: dt is calculated as elapsed milliseconds divided by 1000, with protection against frame drops

### Game State
Game state is maintained as module-level variables:
- `player`: Position, velocity, health, weapon state
- `enemies[]`: Array of active enemies with type, health, position, velocity
- `boss`: Current boss object (null when not in a fight)
- `bullets[]`: Player bullets and projectiles
- `enemyBullets[]`: Shots fired by enemies/bosses
- `powerUps[]`: Drop items from defeated enemies
- `score`, `multiplier`, `stage`: Current game progress
- `shakeTime`, `vignetteTime`: Visual effect timers
- `quality`: Adaptive performance level (0=full, 1=medium, 2=low)

### Difficulty & Progression
Three difficulty levels defined in `DIFFICULTIES` object:
- **easy**: Lower spawn rates, slower enemies, 4 lives, 3 bombs
- **normal**: Standard spawn rates, moderate difficulty, 3 lives, 2 bombs
- **hard**: High spawn rates, fast enemies, 3 lives, 2 bombs, stronger bosses

Progression: Each stage requires 18 enemy kills (`KILLS_PER_STAGE`), then a boss fight. Stages 2+ unlock harder enemy types.

### Enemy System
- **ENEMY_TYPES**: grunt (red), interceptor (fast orange), tank (purple, high HP)
- **Spawning**: `spawnEnemy()` picks random enemy types and positions; spawn rate throttled by difficulty
- **Behavior**: Enemies move downward, aim at player, and shoot randomly based on difficulty
- **Collision**: `rectsOverlap(a, b)` checks rectangle overlap for hit detection

### Boss System
- **6 bosses** with unique names and color schemes (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- **Boss patterns**: `BOSS_PATTERNS` array contains 7 different attack patterns (radial, waves, rings, etc.)
- **Spawning**: `spawnBoss()` creates boss when stage kill quota is met
- **AI**: `updateBoss(dt)` handles movement, pattern selection, and attack cooldowns
- **Health bar**: Visual display at top of screen shows boss HP
- **Defeat**: `killBoss()` handles boss death (explosions, score, stage advance)

### Weapon System
- **Multiple firing patterns**: radial bursts, aimed spreads, columns, spiral bursts, cross patterns, homing, double rings
- **Fire functions**: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc. create bullets in specific patterns
- **Weapon upgrades**: Pickups increase `player.weaponLvl` and change weapon behavior
- **Bullet lifecycle**: Bullets removed when off-screen; collision handled in `update(dt)`

### Visual Effects & Quality Modes
- **Performance adapting**: `trackPerf(dt)` samples frame times; if average > 25ms (40 FPS), reduce quality; if < 17.8ms (56 FPS), improve quality
- **Quality levels**:
  - `quality=0`: Full glow effects (shadows, blur on projectiles)
  - `quality=1`: Medium (some glows removed)
  - `quality=2`: Low (minimal visual effects)
- **Screen shake**: `screenShake(magnitude, duration)` creates camera shake effect (disabled if reduced motion enabled)
- **Flash effects**: `flashHit()` and `flashBomb()` create screen flash on damage/bomb (animated via CSS)
- **Vignette**: Red pulse around screen edges when player takes damage (controlled by `vignetteTime`)

### Controls
- **Desktop**: Arrow keys (move), Space (shoot), B key (bomb)
- **Mobile**: Virtual joystick in bottom-left, fire button (right), bomb button (top-right)
- **Joystick code**: `joystickStart()`, `joystickMove()`, `joystickEnd()` handle touch events on moveZone
- **Mouse aiming**: Crosshair draws at mouse position; player shoots toward cursor

### Audio
- **Web Audio API**: Uses `AudioContext` for synthesis (no audio files)
- **Sound effects**:
  - `beep(freq, dur, type, vol, opts)`: Generates tones (sine, square, sawtooth)
  - `noiseBurst(dur, vol, opts)`: Creates noise-based explosion sounds
  - `panFor(x)`: Pans audio left/right based on x position
- **Mute state**: Persists to localStorage; checked before playing sounds
- **Muted audio still plays**: Web Audio nodes are created but gain set to 0 (allows smooth unmute)

### Input Handling
- **Keyboard**: Checked in `update(dt)` via `keyState` object (arrow keys, space, B)
- **Mouse**: `mousemove` event updates `mouseTarget` for aiming/shooting
- **Touch**: Joystick and button elements on mobile; tracked via `pointerdown`, `pointermove`, `pointerup`
- **Resize**: `resize()` function scales canvas and game dimensions based on window size; called on window resize and at start

### Themes & Visual Customization
- **THEMES array**: 6 themes with sky gradient colors, cloud/sun colors, and theme names
- **Theme selection**: Random theme chosen per game session in `resetGame()`
- **Sky gradient**: Canvas background uses 3-color gradient (changes per theme)

### Accessibility
- **Screen reader support**: `srLive` element with `aria-live="polite"` announces events
- **announce(text)` function**: Queues text for screen reader announcement
- **Reduced motion**: Respects `prefers-reduced-motion` media query; disables screen shake, vignette, flash
- **Mobile touch support**: Full game playable via touch controls

### Persisted State (localStorage)
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Mute toggle (0 or 1)
- `skyfighter.difficulty`: Selected difficulty (easy, normal, hard)
- `skyfighter.reducedMotion`: Reduced motion preference (0 or 1)

## Collision Detection & Physics
- **Rectangle overlap**: `rectsOverlap(a, b)` checks if two rectangles (with x, y, w, h) overlap
- **Player collision**: Checked against enemy bullets and enemies; loses life on hit
- **Enemy collision**: Player bullets remove enemies (HP decremented)
- **Boss collision**: Player bullets damage boss (HP decremented, health bar updated)
- **Power-up collision**: Player collects power-ups when overlapping
- **Damage handling**: `loseLife()` handles player damage; `addScore(base)` applies multiplier to score
- **Score multiplier**: Increases per consecutive hits; resets on miss or death via `breakChain()`

## Code Style Notes
- Single large IIFE wrapping all code (module pattern)
- 'use strict' mode enabled
- No external dependencies
- Minimal comments (only on non-obvious logic)
- All rendering via 2D canvas context
- Game state mutation (no immutability)

## Common Modifications
- **Adjust difficulty**: Edit `DIFFICULTIES` object values (spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul)
- **Add new weapon pattern**: Create new function like `firePattern()` with bullet creation logic; call from `fireWeapon()`
- **Add new enemy type**: Add entry to `ENEMY_TYPES` object; update `spawnEnemy()` to consider `minStage`
- **Change colors/themes**: Modify `THEMES` array or colors in `ENEMY_TYPES`, `BOSSES`
- **Adjust performance threshold**: Modify comparison in `trackPerf()` (avg > 1/40 or avg < 1/56)
- **Add visual effects**: Use `spawnExplosion()` as template; particles added to `particles[]` array in `update()`
